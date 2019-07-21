# Eliminating Large JVM GC Pauses Caused by Background IO Traffic

_Coauthor: [Cuong Tran](https://www.linkedin.com/in/cuong-tran-30666), Systems Architect_

In our production environments, we have repeatedly seen that applications running in JVM  (Java Virtual Machine) occasionally experience large STW (Stop-The-World) application pauses due to JVM’s GC logging being blocked by background IO traffic (e.g., OS page cache writeback).  During such STW pauses, JVM pause all application threads, and applications stop responding to user requests hence incurring unacceptable delays to latency-sensitive use cases.

Our investigations show that the pauses are induced by the JVM GC (Garbage Collection)’s write() system calls during GC log writing. Such log writes, even in an asynchronous (i.e., buffered IO or non-blocking IO) write mode, can still be blocked for considerable time by OS mechanisms including page cache writeback.

We discuss various approaches to mitigate the problem. For latency-sensitive Java applications, we suggest moving Java log files to a separate or high-performing disk drive (e.g., SSD, tmpfs).

# Production issue

When Java heap space that the JVM manages is garbage collected, the JVM could be stopped, which introduces STW pauses to the applications. Depending on JVM options supplied when starting the Java instance, various types of GC and JVM activities are logged into GC log files.

Though some GC-induced STW pauses that scan/mark/compact heap objects are well known, we found out that there are some large STW pauses caused by background IO traffic.  In our production environments, we have seen unexplainable large STW pauses ( > 5 seconds) in our mission-critical Java applications. Such pauses cannot be explained by application-level logic and JVM GC activities. As shown below, we show a large STW pause of more than 4 seconds and some GC information. The garbage collector is G1\. With a mere 8GB heap size and parallel Young Garbage Collection in G1, the garbage collection typically takes less than a second to complete, and the simple GC log options incur little overhead. However the application threads stopped for more than 4 seconds. The amount of work done by GC (e.g., collected heap size) is not able to explain the large pause value of 4.17 seconds.

A GC STW pause of 4.17 seconds with G1 collector Raw

As another example, the following GC log snapshot shows another STW pause of 11.45 seconds. The garbage collector is CMS (Concurrent Mode Sweep). The “user”/”sys” time is negligible, however the “real” GC time is more than 11 seconds. The last line confirms the 11.45-second application stop time.

A GC STW pause of 11.45 seconds with CMS collector Raw

Since the applications are very latency-sensitive, we spent considerable effort investigating the problem. In the end, we successfully reproduced the problem, found the root cause, and came up with solutions to address it.

## Reproducing the problem in a lab environment

We started by reproducing the problem of unexplainable large JVM pauses in a lab environment. For controllability and repeatability, we designed a simple workload which removed the complexity of the production applications.

We ran the workload in two scenarios: with and without background IO activities. The scenario where no background IO existed was treated as the “baseline,” whereas the other scenario with introduced background IO was to reproduce the problem.

### Java Workload

The Java workload we used simply kept allocating objects of 10KB to a queue. Whenever the number of objects reaches 100,000, half of the objects will be removed from the queue. So the maximum number of objects in the heap is 100,000 objects, occupying about 1GB raw size. This process continued for a fixed amount of time (e.g., 5 minutes).

The Java source code, as well as the background IO generation script, is at [https://github.com/zhenyun/JavaGCworkload](https://github.com/zhenyun/JavaGCworkload). The main performance metric we considered was the number of  big JVM GC pauses.

### Background IO

The background IO was introduced by a bash script which repeatedly copied big files. The background workload is generating 150MB/s writing load, sufficient to saturate a single hard drive. To understand how heavy the generated IO load was, we used “sar -d -p 2” to gather the statistics of await (The average time (in milliseconds) for I/O requests issued to the device to be served), tps (Total number of transfers per second that were issued to physical devices), and wr_sec-per-s (Number of sectors written to the device). The average values of them are: await=421 ms, tps=305, wr_sec-per-s=302K.

### System Setup

*   ![System Setup](https://content.linkedin.com/content/dam/engineering/site-assets/images/blog/posts/2016/02/JVM-SystemSetup.jpg)

### Scenario I (Without background IO load)

The baseline run does not have background IO load. The time series data of all JVM GC pauses are shown in the figure below. No single larger-than-250ms pause is observed.

*   ![All JVM GC pauses in Scenario I (without background IO load)](https://content.linkedin.com/content/dam/engineering/site-assets/images/blog/posts/2016/02/JVM-GC1.jpg)

All JVM GC pauses in Scenario I (without background IO load)

### Scenario II (With background IO load)

When the background IO ran, the same Java workload saw 1 STW pause exceeding 3.6 second, and 3 pauses exceeding 0.5 seconds during a mere 5-minute run!

*   ![All JVM GC pauses in SCenario-II (With background IO load)](https://content.linkedin.com/content/dam/engineering/site-assets/images/blog/posts/2016/02/JVM-GC2.jpg)

All JVM GC pauses in SCenario-II (With background IO load)

## Investigation

Trying to understand what system calls caused the STW pauses, we used [strace](http://linux.die.net/man/1/strace) to profile the system calls issued by the JVM instance.

We first verified that the JVM logs GC information to files using asynchronous IO. We traced all system calls issued by the JVM since startup. The GC log file is opened in asynchronous mode and no fsync() calls are observed. 

Captured JVM’s open() system call of opening GC log file Raw

However, traces show that several asynchronous write() system calls issued by JVM have unusually large execution time. Examining the timestamps of the system calls and the JVM pauses, we found that they correlate well. In the below figures, we plot the time series of the two latencies for two minutes.

*   ![Time series correlation (JVM STW pauses)](https://content.linkedin.com/content/dam/engineering/site-assets/images/blog/posts/2016/02/TimeSeries-JVM1.jpg)

Time series correlation (JVM STW pauses)

*   ![Time series correlation (write() system call latencies)](https://content.linkedin.com/content/dam/engineering/site-assets/images/blog/posts/2016/02/TimeSeries-JVM2.jpg)

Time series correlation (write() system call latencies)

We then zoomed in to focus on the biggest pause of 1.59 seconds happening at 13:32:35\. The relevant GC logs and strace output are displayed here:

*   ![JVM GC Log](https://content.linkedin.com/content/dam/engineering/site-assets/images/blog/posts/2016/02/JVM-GC-Log.png)

GC logs and strace output

Let’s try to understand what’s going on.

1.  At time 35.04 (line 2), a young GC starts and takes 0.12 seconds to complete.
2.  The young GC finishes at time 35.17 and JVM tries to output the young GC statistics to gc log file by issuing a write() system call (line 4).
3.  The write() call is blocked for 1.47 seconds and finally finishes at time 36.64 (line 5), taking 1.47 seconds.
4.  When write() call returns at 36.64 to JVM, JVM records this STW pause of 1.59 seconds (i.e., 0.12 + 1.47) (line 3).

In other words, the actual STW pause time consists of two parts: (1) GC time (e.g., young GC) and (2) GC logging time (e.g., write() time).

These data suggest that the GC logging process is on the JVM’s STW pausing path, and the time taken for logging is part of STW pause. Specifically, the entire application pause mainly consists of two parts: pause due to JVM GC activity and pause due to OS blocking write() system call corresponding to JVM GC logging. The following diagram shows the relationship between them.

*   ![JVM - OS interaction](https://content.linkedin.com/content/dam/engineering/site-assets/images/blog/posts/2016/02/JVM-schematic-revised.jpg)

The interaction between JVM and OS during GC logging

If the GC logging (i.e., write() calls) is blocked by OS, the blocking time contributes to the STW pause. The new question is why buffered writes are blocked? Digging into various resources including the kernel source code, we realized that buffered writes could be stuck in kernel code. There are multiple reasons including: (1) stable page write; and (2) journal committing.

Stable page write: JVM writing to GC log files firstly “dirties” the corresponding file cache pages. Even though the cache pages are later persisted to disk files via OS’s writeback mechanism, dirtying the cache pages in memory is still subject to a page contention caused by “stable page write.” With “stable page write,” if a page is under OS writeback, a write() to this page has to wait for the writeback to complete. The page is locked to ensure data consistency by avoiding a partially fresh page from being persisted to disk. 

Journal committing: For the journaling file system, appropriate journals are generated during file writing. When appending to the GC log file results in new blocks being allocated, the file system needs to commit the journal data to disk first. During journal committing, if the OS has other IO activities, the commitment might need to wait. If the background IO activities are heavy, the waiting time can be noticeably long. Note that [EXT4](https://en.wikipedia.org/wiki/Ext4) file system has a feature of “delayed allocation” which postpones certain journal data to OS writeback time, which alleviates this problem. Note also that changing EXT4’s data mode from the default “ordered” mode to “writeback” does not really address this cause, as the journal needs to be persisted before write-to-extend call returns.

## Background IO activities

From the standpoint of a particular JVM garbage collection, background IO activities are inevitable in typical production environments. There are several sources of such IO activities: (1) OS activity; (2) administration and housekeeping software; (3) other co-located applications; (4) IO of the same JVM instance. First, OS contains many mechanisms (e.g., “/proc” file system) that incur data writing to underlying disks. Second, system-level software such as [CFEngine](https://cfengine.com/) also perform disk IO. Third, if the node has co-located applications that share the disk drives, then other applications contend on IO. Fourth, the particular JVM instance may use disk IO in ways other than GC logging.

# Solutions

Since in current HotSpot JVM implementation (as well as in some others) GC logging can be blocked by background IO activities, there are various solutions that can help mitigate this problem when writing to GC log file.

First, the JVM implementation could be enhanced to completely address this issue. Particularly, if the GC logging activities are separated from the critical JVM GC processes that cause STW pauses, then this problem will go away. For instance, JVM can put the GC logging function into a different thread which handles the log file writing independently, hence not contributing to the STW pauses. Taking the separate-thread approach however risks losing last GC log information during JVM crash. It might make sense to expose a JVM flag allowing users to specify their preference.

Since the extent of STW pauses caused by background IO depends on how heavy the latter is, various ways to reduce the background IO intensity can be applied. For instance, de-allocating other IO-intensive applications on the same node, reducing other types of logging, improving on the log rotation, etc.

For latency-sensitive applications such as online ones serving interactive users, large STW pauses (e.g., >0.25 seconds) are intolerable. Hence, special treatments need to be applied.  The bottom line of ensuring no big STW pauses induced by OS is to avoid GC logging being blocked by OS IO activities.

One solution is to put GC log files on tmpfs (i.e., -Xloggc:/tmpfs/gc.log). Since tmpfs does not have disk file backup, writing to tmpfs files does not incur disk activities, hence is not blocked by disk IO. There are two problem with this approach: (1) the GC log file will be lost after system crashes; and (2) it consumes physical memory. A remedy to this is to periodically backup the log file to persistent storage to reduce the amount of the loss.1

Another approach is to put GC log files on SSD (Solid-State Drives), which typically has much better  IO performance. Depending on the IO load, SSD can be adopted as a dedicated drive for GC logging, or shared with other IO loads. However, the cost of SSD needs to be taken into consideration.

Cost-wise, rather than using SSD, a more cost-effective approach is to put GC log file on a dedicated HDD. With only the IO activity being the GC logging, the dedicated HDD likely can meet the low-pause JVM performance goal. In fact, the Scenario I we showed above can mimic such a setup, since in that setup no other IO activities exist on the GC-logging drive.

# Evaluation of putting GC log on SSD and tmpfs

We take the dedicated-file-system approach by putting the GC log file on SSD and tmpfs drives. We run the same Java workload and the background IO load as in Scenario II.

For both SSD and tmpfs, we observe similar results, and the following figure shows the results of putting GC log file on a SSD disk. We notice that the JVM pausing performance are on-par with Scenario I, and all pauses are under 0.25 seconds. The results indicate the background IO load does not impact the application performance.  

*   ![All JVM STW pauses when GC logging to SSD](https://content.linkedin.com/content/dam/engineering/site-assets/images/blog/posts/2016/02/JVM-Pauses-2-revised.jpg)

All JVM STW pauses when GC logging to SSD

# Conclusion

Latency-sensitive Java applications require small JVM GC pauses. However, the JVM can be blocked for substantial time periods when disk IO is heavy.

We investigated this issue and found out that:

1.  JVM GC needs to log GC activities by issuing write() system calls;
2.  Such write() calls can be blocked due to background disk IO;
3.  GC logging is on the JVM pausing path, hence the time taken by write() calls contribute to JVM STW pauses.

We propose a set of solutions to mitigate this problem. In particular, the findings can be used to enhance the JVM implementation to avoid this issue. For latency-sensitive applications, an immediate solution should be avoiding the IO contention by putting the GC log file on a separate HDD or high-performing disk such as SSD.

# [Getting information about a process' memory usage from /proc/pid/smaps](https://unix.stackexchange.com/questions/33381/getting-information-about-a-process-memory-usage-from-proc-pid-smaps)


Clean pages are pages that have not been modified since they were mapped (typically, text sections from shared libraries are only read from disk (when necessary), never modified, so they'll be in shared, clean pages).
Dirty pages are pages that are not clean (i.e. have been modified).

Private pages are available only to that process, shared pages are mapped by other processes*.

RSS is the total number of pages, shared or not, currently mapped into the process. So `Shared_Clean` + `Shared_Dirty` would be the shared part of the RSS (i.e. the part of RSS that is also mapped into other processes), and `Private_Clean` + `Private_Dirty` the private part of RSS (i.e. only mapped in this process).

PSS (proportional share size) is as you describe. Private pages are summed up as is, and each shared mapping's size is divided by the number of processes that share it.
So if a process had 100k private pages, 500k pages shared with one other process, and 500k shared with four other processes, the PSS would be:

```
100k + (500k / 2) + (500k / 5) = 450k

```

Further readings:

*   [ELC: How much memory are applications really using?](http://lwn.net/Articles/230975/)
*   [`Documentation/filesystems/proc.txt`](http://www.kernel.org/doc/Documentation/filesystems/proc.txt) in the kernel source
*   [`man proc(5)`](http://www.kernel.org/doc/man-pages/online/pages/man5/proc.5.html)
*   [Linux Memory Management Overview](http://tldp.org/LDP/khg/HyperNews/get/memory/linuxmm.html)
*   [Memory Management](http://tldp.org/LDP/tlk/mm/memory.html) at TLDP.org
*   [LinuxMM](http://linux-mm.org/)

Regarding process-wide sums:

*   `RSS` can be (approximately+) obtained by summing the `Rss:` entries in `smaps` (you don't need to add up the shared/private shared/dirty entries).

    ```
    awk '/Rss:/{ sum += $2 } END { print sum }' /proc/$$/smaps

    ```

*   You can sum up `Pss:` values the same way, to get process-global `PSS`.
*   `USS` isn't reported in `smaps`, but indeed, it is the sum of private mappings, so you can obtain it the same way too

*Note that a "share-able" page is counted as a private mapping until it is _actually_ shared. i.e. if there is only one process currently using `libfoo`, that library's text section will appear in the process's _private_ mappings. It will be accounted in the shared mappings (and removed from the private ones) only if/when another process starts using that library.
+The values don't add up exactly for all processes. Not exactly sure why... sorry.

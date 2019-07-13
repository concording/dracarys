
2019-07-13T14:36:27.775+0800: 686353.638: [GC (CMS Initial Mark) [1 CMS-initial-mark: 63K(64K)] 1571374K(1835072K), 0.3458093 secs] [Times: user=1.34 sys=0.02, real=0.35 secs]
2019-07-13T14:36:28.121+0800: 686353.984: [CMS-concurrent-mark-start]
2019-07-13T14:36:28.121+0800: 686353.984: [CMS-concurrent-mark: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
2019-07-13T14:36:28.121+0800: 686353.984: [CMS-concurrent-preclean-start]
2019-07-13T14:36:28.122+0800: 686353.985: [CMS-concurrent-preclean: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
2019-07-13T14:36:28.122+0800: 686353.985: [CMS-concurrent-abortable-preclean-start]
2019-07-13T14:36:28.122+0800: 686353.985: [CMS-concurrent-abortable-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
2019-07-13T14:36:28.123+0800: 686353.986: [GC (CMS Final Remark) [YG occupancy: 1571310 K (1835008 K)]2019-07-13T14:36:28.123+0800: 686353.986: [Rescan (parallel) , 0.3780544 secs]2019-07-13T14:36:28.501+0800: 686354.364: [weak refs processing, 0.0000327 secs]2019-07-13T14:36:28.501+0800: 686354.364: [class unloading, 0.0158953 secs]2019-07-13T14:36:28.517+0800: 686354.380: [scrub symbol table, 0.0124408 secs]2019-07-13T14:36:28.529+0800: 686354.392: [scrub string table, 0.0011111 secs][1 CMS-remark: 63K(64K)] 1571374K(1835072K), 0.4111989 secs] [Times: user=1.53 sys=0.00, real=0.41 secs]
2019-07-13T14:36:28.534+0800: 686354.397: [CMS-concurrent-sweep-start]
2019-07-13T14:36:28.534+0800: 686354.397: [CMS-concurrent-sweep: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
2019-07-13T14:36:28.534+0800: 686354.397: [CMS-concurrent-reset-start]
2019-07-13T14:36:28.534+0800: 686354.397: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]


 /opt/app/jdk1.8.0_91/bin/java -Djava.util.logging.config.file=/home/apple/insurance-admin-tomcat-1/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xmx2048m -Xms2048m -Xmn2048m -XX:SurvivorRatio=6 -XX:PermSize=256m -XX:MaxPermSize=256m -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Xloggc:/data/logs/insurance/admin/gc.log -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/data/logs/insurance/admin/ -Dtomcatlogdir=/data/logs/insurance/admin -Djava.endorsed.dirs=/home/apple/insurance-admin-tomcat-1/endorsed -classpath /home/apple/insurance-admin-tomcat-1/bin/bootstrap.jar:/home/apple/insurance-admin-tomcat-1/bin/tomcat-juli.jar -Dcatalina.base=/home/apple/insurance-admin-tomcat-1 -Dcatalina.home=/home/apple/insurance-admin-tomcat-1 -Djava.io.tmpdir=/home/apple/insurance-admin-tomcat-1/temp org.apache.catalina.startup.Bootstrap start


```
-Xmn : the size of the heap for the young generation

Young generation represents all the objects which have a short life of time. Young generation objects are in a specific location into the heap, where the garbage collector will pass often. All new objects are created into the young generation region (called "eden"). When an object survive is still "alive" after more than 2-3 gc cleaning, then it will be swap has an "old generation" : they are "survivor".
```



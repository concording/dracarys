# Improving cache consistency

December 02, 2016

A typically web application introduces an in-memory cache like `memcache` or `redis` to reduce load on the primary database for reads requesting hot data. The most primitive design looks something like `Figure 1`.

```
+--------------------------------+        +------------+        +----------------+
|            database            <--------+ web server +-------->     cache      |
| mssql, mysql, oracle, postgres |        +------------+        | memcache/redis |
+--------------------------------+                              +----------------+

```

_Figure 1_

Unfortunately this design is really common despite the many issues it introduces. I’ve seen some organizations with large scale applications still using this design and they maintain a bunch of hacks to overcome these issues which increases the systems operational complexity and sometimes surfaces as inconsistent data to end users.

## Issue 1\. Pool of connections to the cache services per web server instance

In a large application sometimes thousands of web server instances (especially in slower languages like Ruby) are hosting the web application. Each one has to maintain connections to the infrastructure the web application code communicates with directly. This can include primary databases like MSSQL, MySQL, Oracle, Postgres and cache services like Memcache or Redis. Each web server instance would for example have a pool of connections for each database or cache service instance it communicates with.

```
         --------------------------------------------------------------------------
         |                database (mssql, mysql, oracle, postgres)               |
         +----^--^-----------^--^-----------^--^-----------^--^-----------^--^----+
              |  |           |  |           |  |           |  |           |  |
N connections |  |           |  |           |  |           |  |           |  |
              |  |           |  |           |  |           |  |           |  |
         +------------+ +------------+ +------------+ +------------+ +------------+
         | web server | | web server | | web server | | web server | | web server |
         +------------+ +------------+ +------------+ +------------+ +------------+
              |  |           |  |           |  |           |  |           |  |
N connections |  |           |  |           |  |           |  |           |  |
              |  |           |  |           |  |           |  |           |  |
         -----v--v-----------v--v-----------v--v-----------v--v-----------v--v-----
         |                         cache (memcache, redis)                        |
         +------------------------------------------------------------------------+

```

_Figure 2_

This can be a strain on resources both on the web server but more importantly the database or cache service as shown in `Figure 2`. This is why I included a `16,384` connection benchmark in my [benchmarks of Redis server libraries for Go](http://simongui.github.io/2016/10/24/benchmarking-go-redis-server-libraries.html) to see how they scaled. It’s not uncommon to see `10,000` or `20,000` connections to a Memcache or Redis server in a large system designed like this.

## Issue 2\. Many web app requests have to execute cache set operations

Similar to how a HTTP request may issue multiple SQL INSERT or UPDATE statements, multiple SET operations may be issued against the cache service. Even though these can be done asynchronously, they still consume resources on the web server and it would be great if the web servers only had to be concerned with updating the primary database.

## Issue 3\. No fault tolerance. Data loss if cache set operations fail

The typical sequence of operations of how `Figure 2` in a web application would be designed would be as follows.

*   Update the primary database (MSSQL, MySQL, Oracle, Postgres, etc).
*   If the transaction fails return a HTTP error.
*   If the transaction succeeds send SET operations to the cache server(s) (memcache, redis, etc).

Any SET operation could fail even after retrying which puts the cache service(s) inconsistent with the primary database which could result in users seeing incorrect information. Even worse depending how the application is designed you could experience _partial failures_ which results in users seeing partially correct and partially incorrect information after a change and a cache hit.

Some cache service protocols support sending multiple SET operations in one command but some do not. Not all web applications are smart enough to group SET operations that happen in different areas of the code into a single command either. If this is the case you could have _partial failures_where some of the SET operations succeeded and some failed.

Outside of retrying there’s not much the web application can do to eventually correct the missing cache SET operations. It has to retry and give up at some point. The cache will be serving cache hits that are inconsistent with the primary database until the cache key(s) invalidate via a TTL or some other process.

### Messaging middleware

Sometimes this gets solved by messaging middleware like Kafka where the web applications push SET operations into Kafka and consumers pull changes from Kafka and execute the SET operations on the cache service(s). This greatly increases the cache consistency and allows the caches survive failures and catch up after short or long failures.

This introduces latency in the system. Changes may not be seen right away to users. Some web applications solve this by doing sticky sessions and caching in-memory in the web application to hide that data is inconsistent. Stale results are still possible if the web server fails and requests route to a different web server instance. This introduces complexity in the request routing tier of the system.

```
         +------------------------------------------------------------------------+
         |                database (mssql, mysql, oracle, postgres)               |
         +----^--^-----------^--^-----------^--^-----------^--^-----------^--^----+
              |  |           |  |           |  |           |  |           |  |
N connections |  |           |  |           |  |           |  |           |  |
              |  |           |  |           |  |           |  |           |  |
         +----+--+----+ +----+--+----+ +----+--+----+ +----+--+----+ +----+--+----+
         | web server | | web server | | web server | | web server | | web server |
         +----+--+----+ +----+--+----+ +----+--+----+ +----+--+----+ +----+--+----+
              |  |           |  |           |  |           |  |           |  |
N connections |  |           |  |           |  |           |  |           |  |
              |  |           |  |           |  |           |  |           |  |
         +----v--v-----------v--v-----------v--v-----------v--v-----------v--v----+
         |                    message queue (kafka, rabbitmq)                     |
         +----------------------------------^--^----------------------------------+
                                            |  |
                              N connections |  |
                                            |  |
                                     +------+--+------+
                                     | kafka consumer |
                                     +------+--+------+
                                            |  |
                              N connections |  |
                                            |  |
         +----------------------------------v--v----------------------------------+
         |                         cache (memcache, redis)                        |
         +------------------------------------------------------------------------+

```

_Figure 3_

As shown in `Figure 3` this greatly reduces the connection load on the cache service but introduces a lot of operational complexity such as the following.

*   Deploy and operate a high throughput messaging system like Kafka with multiple brokers to survive broker failures.
*   Deploy and operate multiple consumer processes that consume messages in Kafka and execute SET operations to the cache service(s) to survive consumer failures.

## Issue 4\. No sequential consistency with the primary database

Leslie Lamport describes [sequential consistency](http://research.microsoft.com/en-us/um/people/lamport/pubs/lamport-how-to-make.pdf) as follows.

> The result of any execution is the same as if the operations of all the processors were executed in some sequential order, and the operations of each individual processor appear in this sequence in the order specified by its program.

`Figure 3` greatly improves fault tolerance and reduces the chances of losing an update however does not address order. `Issue 3` describes the possibility of complete and partial failures and explains how a user could see partially up-to-date and partially stale results. Diving deeper operations could fail before following operations succeed. The order of visible changes could be out-of-order. Some applications may be more sensitive to this kind of inconsistency. Some applications may require strict partial order. Even if order isn’t critical, providing sequential consistency is a better experience for users and less confusing.

# Solution: MySQL binlog replication

`Figure 3` shows the benefits of a shared message queue however deploying one with fault tolerance is not trivial and operating one smoothly isn’t trivial either. If you use a database with replication there’s already a queue in your system and you may not need to deploy yet another queue and new piece of infrastructure like Kafka to solve some of these problems.

```
+----------+---+---+---+---+---+   binlog replication   +--------------------------+
| MySQL    | 1 | 2 | 3 | 4 | 5 <------------------------+ MySQL replication client |
+----------+---+---+---+---+---+                        +--------------------------+
              MySQL binlog
            binlog positions

```

_Figure 4_

MySQL has a binlog replication protocol which is used for primary/secondary replication. This is essentially a replicated queue that has all the transactions recorded in-order as shown in `Figure 4`.

This isn’t a popular solution but I say, why not? It works very well. You can write an application that can speak the MySQL binlog replication protocol that consumes the binlog entries and execute SET operations against the cache service(s). There are two ways you could consume the binlog data.

*   Interpret the raw SQL syntax and issue SET operations.
*   The web application embeds cache keys as a comment in the SQL.

Both of these options are good because you can even get the transaction scope of each transaction in the binlog statements if you need to and if the target system supports atomic multi-set operations. I prefer the 2nd option because it’s easier to parse and the application already has this information in most cases.

```
         +------------+ +------------+ +------------+ +------------+ +------------+
         | web server | | web server | | web server | | web server | | web server |
         +------------+ +------------+ +------------+ +------------+ +------------+
              |  |           |  |           |  |           |  |           |  |
N connections |  |           |  |           |  |           |  |           |  |
              |  |           |  |           |  |           |  |           |  |
         +----v--v-----------v--v-----------v--v-----------v--v-----------v--v----+
         |                 database (mssql, mysql,,oracle, postgres)              |
         +------------------------------------^-----------------------------------+
                                              |
                                 1 connection |
                                              |
                               +---------------------------+
                               | binlog replication client |
                               +---------------------------+
                                            |  |
                              N connections |  |
                                            |  |
         +----------------------------------v--v----------------------------------+
         |                         cache (memcache, redis)                        |
         +------------------------------------------------------------------------+

```

_Figure 5_

`Figure 5` shows the overall architecture with the binlog replication in place.

### Benefits

*   Drastically reduces connection load on the cache service(s). Web servers only connect to the database.
*   Sequential consistency because we are reading the databases commit log into the cache service(s).
*   Possible to connect to _any_ MySQL replica in the replication chain since they are all sequentially consistent.

I love Kafka and have nothing against it, I use it myself. Reducing infrastructure simplified the architecture and reduced operational complexity. By replicating the MySQL commit log to the cache service(s) we have increased the consistency as well as gained strict partial order between the database and the cache service(s).

I’m currently working on a project in Go that provides this proposed functionality that I’ll announce at a later date. Contact me if you want to know more about it.
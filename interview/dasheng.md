# 实战

### 太阳58

1. HashMap底层原理，hash碰撞

2. 事务隔离性分别是什么，采用不同的隔离级别会有什么不同结果。

3. Mq，如何防止消息重复

4. 什么是最终一致性。

5. RPC框架用过哪些。

6. 
```
String s1 = 'abc';
String s2 = 'abc';
s1==s2 ?
s1.equals(s2) ?
String s3 = new String("abc"); 创建了几个对象，都分别存在哪里？
s3==s1 ?
```
7. 
```Ingeger i1 = 100;

Integer i2 = 100;
i1 == i2  ?
i1.equals(i2) ?
```
8. MQ如果不能使用，有哪几种方案实现mq的功能？(redis订阅, 数据库轮询)

9. Redis数据结构，都应用在什么场景下。

10. Setnx（redis命令）什么意思。

11. 分布式事务（难，后面研究）

[https://www.cnblogs.com/savorboard/p/distributed-system-transaction-consistency.html](https://www.cnblogs.com/savorboard/p/distributed-system-transaction-consistency.html)

比如很多人都知道数据库事务的几个特性：原子性(Atomicity )、一致性( Consistency )、隔离性或独立性( Isolation)和持久性(Durabilily)，简称就是ACID。

12. 分布式锁实现原理，有几种实现方案 (redis, zookeeper)

13. Redis有什么特点，都在哪些场景下体现了？

14. Redis和数据库怎么保证数据同步(rdb, aof)，有几种方式，哪种最好(aof)。

15. 如何保证消息的执行顺序??

设置每次只消费一个消息, 手动应答

16. 如果不使用MQ怎么实现MQ的功能。

17. 多台服务器修改库存数据，怎么保证库存的数据正确，有几种方案??

18. java实现servlet每次一个请求，执行业务代码需要5秒钟，怎么进行优化，可以从哪些方面入手。

执行业务使用多线程, 数据库优化, 业务逻辑算法优化,

19. TCP

20. dubbo的实现方式，什么是长连接，什么是短连接。在哪用到了

dubbo协议就是长连接, Tcp连接, 适用于多次请求, 但是数据量都很小的

hessian协议, http协议, 短连接, 适合文件传输,大数据量的

21. dubbo容错机制，failOver和failfast什么区别，应用在什么场景下。

failfast失败立马报错,适用于非幂等性的写操作

失败自动切换，当出现失败，重试其它服务器 1。通常用于读操作，但重试会带来更长延迟。可通过 retries="2" 来设置重试次数(不含第一次)。

22. 响应端服务超时没有响应，如何才能保证消息送达。如果消息重复了怎么办。

23. 订单表2亿的数据。怎么能提高效率。

分表分库

24. 分表分库原则是什么。

25. wait（），sleep（），join都怎么使用。

26. 有一个主线程，等待其他三个线程响应结果后在进行数据处理。怎么实现，有几种方式。

27. 线程池怎么实现？核心线程数，最大线程数，作用是什么

28. Array有100个元素，只有每次从数组中取出一个后，才能在添加进去一个元素。使用阻塞式线程怎么实现。说一说实现步骤。

29. 有两个表存10G容量，放的都是String类型数据，现在有1G内存，怎么获取两个表中的相同数据。说一下实现思路。

30. 做一个秒杀系统需要考虑哪些因素，怎么解决。100件商品，如何保证不超买。怎么分担数据库的压力。怎么分担前端压力。怎么防止恶意刷单

31. dubbo、nginx负载均衡实现方式

32. 一致性hash原理，什么是虚拟节点。

33. cap理论

34. 线程锁都用过哪些，优缺点。数据库锁都有哪些，优缺点。

35. 订单有5000万数据，实现用户查看最近100条订单，现在查询结果是30秒，有几种方案进行优化。

36. Volitle，countdownLunch

37. 数据库索引的b-tree和hash优缺点是什么。

### 太阳京东0503

1. 手写SQL语句

2. Redis穿透问题。

3. 介绍自己的项目相关问题

提供的功能和服务有哪些，

访问量多少，

负载多少，

并发数是多少，

集群-几个服务器，

压测峰值，

是否有性能监控

QPS是多少，

在各个环节多访问量突然增多进行了什么样的处理

在哪个环节使用的是RPC，哪个环节使用的是MQ

项目的容灾机制

项目的数据可靠性如何保证

项目是如果进行部署的，如何查看服务器性能

怎么保证数据一致性的（缓存和数据库同步问题）

主备如何进行切换（Redis、mysql）, mysql的读写分离,集群需要补!!

4. 哪个技术了解的最深（看过源码）

5. spring AOP在什么时候切面会失效

6. spring的事务怎么使用，原理是什么，在什么时候会存在失效问题

7. mysql事务，特征（4点），事务隔离级别，每种隔离级别会产生什么效果

8. mysql分布式事务如果保证。一阶段提交和二阶段提交，哪里是阻塞的，哪里是异步的。

9. mysql怎么实现分布式锁

10. mysql索引有哪几种，特点分别是什么。

11. 当QPS达到100%的时候如何定位问题

12. mysql如何定位慢查询，优化步骤，需要注意哪些问题

13. Redis的数据结构，各自特点

14. Redis，如何保证和数据库数据一致的

15. 什么是最终一致性，在项目中哪个环节采用的最终一致性来保证数据的（难）

16. 有多台服务器，如何查询项目适合部署到哪一台，都用到什么命令

17. 多线程实现方式，核心线程数，最大线程数，队列，拒绝策略分别都是什么意思。

18. 多线程的核心线程数，最大线程数，队列，这三者的开启顺序是什么

19. 多线程，拒绝策略有哪几种（4种）

20. NIO，BIO区别，各自在什么场景下使用

21. 线程停止有几种方式，有什么区别

22. 多个线程获取线程结果后在进行运算，怎么实现

23. 多个线程同时执行到某一位置后，再统一继续向下执行，怎么实现

24. concurrent包下，都了解哪些。

25. concurrentHashMap的特点

26. java1.8的lambda表达式特点是什么，什么时候使用?

27. 设计模式都了解哪些，什么是策略模式，什么是桥接模式，适配器模式和装饰着模式有什么区别，单例模式有几种，都在什么情况下使用，在你的项目中哪些地方用到了

28. 多个线程如何保证执行顺序

29. Junit如何使用

30. Ajax特点，怎么实现的

31. RPC都了解哪些，dubbo的原理

32. 长连接和短链接的区别

33. 注册中心挂了，通信是否会受到影响

34. zookeeper了解多少

35. netty是否了解

36. TCP，UDP区别，TCP怎么实现的

37. MQ的消息确认机制，防重，回退，重发，死信队列，如何保证消息传递的可靠性

38. 消息中间件存储数据的结构是什么样的

39. 分布式事务，分布式锁

### 网信6.7

1. spring的核心类有哪些

这个参考<JavaWeb>书中的类图,

Bean组件中主要有：

DefaultListableBeanFactory, 父类有configurableListableBeanFactory, AbstractAutowiredCapableBeanFactory, Bean定义方面的RootBeanDefinition, Bean解析方面的xmlBeanDefinitionReader.

context组件有：

AbstractXmlApplicationContext, AbstractRefreshableWebApplicationContext, AbstractRefreshableApplicationContext,

core组件有：Resource, DefaultResourceLoader, ResourcePatternResolver

2. controller是否线程安全？相关问题: spring单例，service和dao确能保证线程安全吗?

答：都不安全，如果配置为多例，则安全!!

[https://jingyan.baidu.com/article/bea41d439460c0b4c51be6e3.html](https://jingyan.baidu.com/article/bea41d439460c0b4c51be6e3.html) (controller-service-dao都不是线程安全的, 都是单例, 如果有共享变量都会有问题, 案例很好)

在使用spring开发web 时要注意，默认Controller、Dao、Service都是单例的, 都不是线程安全的, 如果想线程安全, 有几种解决方法：

1、在Controller中使用ThreadLocal变量

2、在spring配置文件Controller中声明 scope="prototype"，每次都创建新的controller

[https://www.cnblogs.com/chanshuyi/p/5052426.html](https://www.cnblogs.com/chanshuyi/p/5052426.html) (servlet是线程安全吗, 不是!!)

[https://blog.csdn.net/c289054531/article/details/9196053](https://blog.csdn.net/c289054531/article/details/9196053) (这里只是针对dao使用的connection为什么是线程安全, 并且用多线程进行验证)

3. 主机远程复制命令？查看端口号状态命令？查看cup命令

[https://www.cnblogs.com/tocy/p/linux_scp_remote-file-transfer.html](https://www.cnblogs.com/tocy/p/linux_scp_remote-file-transfer.html)  远程复制命令scp

[https://www.cnblogs.com/Archmage/p/7570716.html](https://www.cnblogs.com/Archmage/p/7570716.html)  查看端口状态命令netstat

[https://blog.csdn.net/albenxie/article/details/72885951](https://blog.csdn.net/albenxie/article/details/72885951)  查看cpu方面的命令

(top -bn 1 -i -c) (netstat -ntlp)

[http://man.linuxde.net/top](http://man.linuxde.net/top) top命令详解

4. 查出每个班级第一名？或第五名？（学号，班级，成绩）

答：第一名简单，第五名mysql很难实现。

5. rabbitMq相比其他mq有什么优势？是否能防止丢消息？别的mq可以吗？

[https://blog.csdn.net/dj2008/article/details/78872889](https://blog.csdn.net/dj2008/article/details/78872889)  （关于消息队列的应用场景很好）

[https://yq.aliyun.com/articles/475265](https://yq.aliyun.com/articles/475265)  （Kafka、RabbitMQ、RocketMQ等消息中间件的对比  ——  消息发送性能和优势）

[https://www.jianshu.com/p/ea04ee9504c3](https://www.jianshu.com/p/ea04ee9504c3)  （RabbitMQ消息可靠性分析，从四个方面好，太帅了！）

[http://www.likecs.com/default/index/show?id=14248](http://www.likecs.com/default/index/show?id=14248)  （关于消息中间件的选型，太长了，有空再看）

6. 一个类的加载会经历几个类加载器

答：三个，根据双亲委派机制，所有加载请求最终都会传送到顶层的启动类加载器。所以在没有自定义类加载器的情况下，都会经历至少三个，Application ClassLoader, Extension ClassLoader, Bootstrap ClassLoader。

### 58coin 会分期 6.8

1. 线程池原理，重要参数，有几种阻塞队列？

[http://www.cnblogs.com/dolphin0520/p/3932921.html](http://www.cnblogs.com/dolphin0520/p/3932921.html)  （这篇全面，但是源码不是最新版）

[https://blog.csdn.net/cjh94520/article/details/70545202](https://blog.csdn.net/cjh94520/article/details/70545202)  （这篇分析的简单，以最新版源码分析的）

默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；

默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；

重点解释一下corePoolSize、maximumPoolSize、largestPoolSize三个变量：（文章中举得例子很好！！）

corePoolSize在很多地方被翻译成核心池大小，其实我的理解这个就是线程池的大小。也就是说corePoolSize就是线程池大小，maximumPoolSize在我看来是线程池的一种补救措施，即任务量突然过大时的一种补救措施。largestPoolSize只是一个用来起记录作用的变量，用来记录线程池中曾经有过的最大线程数目，跟线程池的容量没有任何关系。

这里有一个非常巧妙的设计方式，假如我们来设计线程池，可能会有一个任务分派线程，当发现有线程空闲时，就从任务缓存队列中取一个任务交给空闲线程执行。但是在这里，并没有采用这样的方式，因为这样会要额外地对任务分派线程进行管理，无形地会增加难度和复杂度，这里直接让执行完任务的线程去任务缓存队列里面取任务来执行。

到这里，大部分朋友应该对任务提交给线程池之后到被执行的整个过程有了一个基本的了解，下面总结一下：

　　1）首先，要清楚corePoolSize和maximumPoolSize的含义；

　　2）其次，要知道Worker是用来起到什么作用的；

　　3）要知道任务提交给线程池之后的处理策略，这里总结一下主要有4点：

·如果当前线程池中的线程数目小于corePoolSize，则每来一个任务，就会创建一个线程去执行这个任务；

·如果当前线程池中的线程数目>=corePoolSize，则每来一个任务，会尝试将其添加到任务缓存队列当中，若添加成功，则该任务会等待空闲线程将其取出去执行；若添加失败（一般来说是任务缓存队列已满），则会尝试创建新的线程去执行这个任务；

·如果当前线程池中的线程数目达到maximumPoolSize，则会采取任务拒绝策略进行处理；

· 如果线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止，直至线程池中的线程数目不大于corePoolSize；如果允许为核心池中的线程设置存活时间，那么核心池中的线程空闲时间超过keepAliveTime，线程也会被终止。

**_线程池的线程初始化：_**

默认情况下，创建线程池之后，线程池中是没有线程的，需要提交任务之后才会创建线程。在实际中如果需要线程池创建之后立即创建线程，可以通过以下两个方法办到：

prestartCoreThread()：初始化一个核心线程；

prestartAllCoreThreads()：初始化所有核心线程

**_任务缓存队列及排队策略：_**

workQueue的类型为BlockingQueue<Runnable>，通常可以取下面三种类型：

1）ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小；

2）LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；

3）synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。

**_任务拒绝策略：_**

当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略，通常有以下四种策略：

ThreadPoolExecutor.AbortPolicy: 丢弃任务并抛出RejectedExecutionException异常。

ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。

ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）

ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

[http://www.cnblogs.com/dolphin0520/p/3932906.html](http://www.cnblogs.com/dolphin0520/p/3932906.html)  **_关于阻塞队列_**

阻塞队列包括了非阻塞队列中的大部分方法，上面列举的5个方法在阻塞队列中都存在，但是要注意这5个方法在阻塞队列中都进行了同步措施。除此之外，阻塞队列提供了另外4个非常有用的方法：

put(E e); take();offer(E e,long timeout, TimeUnit unit);

poll(long timeout, TimeUnit unit);

put方法用来向队尾存入元素，如果队列满，则等待；

take方法用来从队首取元素，如果队列为空，则等待；

offer方法用来向队尾存入元素，如果队列满，则等待一定的时间，当时间期限达到时，如果还没有插入成功，则返回false；否则返回true；

poll方法用来从队首取元素，如果队列空，则等待一定的时间，当时间期限达到时，如果取到，则返回null；否则返回取得的元素；

理解interrupt()方法:  [https://blog.csdn.net/u013238950/article/details/53893676](https://blog.csdn.net/u013238950/article/details/53893676)

**_顺带资料，_****_java_****_面试题_****_50_****_道，这里头的回答比较权威，_**基本都是官方解释，值得信赖。里头有些链接需要翻墙。

[https://www.cnblogs.com/dolphin0520/p/3958019.html](https://www.cnblogs.com/dolphin0520/p/3958019.html)

Java中如何停止一个线程？

Java提供了很丰富的API但没有为停止线程提供API。JDK 1.0本来有一些像stop(), suspend() 和 resume()的控制方法但是由于潜在的死锁威胁因此在后续的JDK版本中他们被弃用了，之后Java API的设计者就没有提供一个兼容且线程安全的方法来停止一个线程。当run() 或者 call() 方法执行完的时候线程会自动结束,如果要手动结束一个线程，你可以用volatile 布尔变量来退出run()方法的循环或者是取消任务来中断线程。点击这里查看示例代码。Using a flag to stop Thread is a very popular way of stopping the thread and it's also safe because it doesn't do anything special rather than helping run() method to finish itself.

Java中notify 和  notifyAll有什么区别？

为什么wait, notify 和 notifyAll这些方法不在thread类里面？（这个问题刚好和58问的sleep和wait有什么区别类似）

一个很明显的原因是JAVA提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。如果线程需要等待某些锁那么调用对象中的wait()方法就有意义了。如果wait()方法定义在Thread类中，线程正在等待的是哪个锁就不明显了。

由于wait，notify和notifyAll都是锁级别的操作，所以把他们定义在Object类中因为锁属于对象。

为什么wait和notify方法要在同步块中调用？

为什么你应该在循环中检查等待条件?

Java中堆和栈有什么不同？Java中活锁和死锁有什么区别？怎么检测一个线程是否拥有锁？JVM中哪个参数是用来控制线程的栈堆大小的？有三个线程T1，T2，T3，怎么确保它们按顺序执行？如果你提交任务时，线程池队列已满。会时发会生什么？（这个解释不太合理）什么是阻塞式方法？如何在Java中创建Immutable对象？

多线程中的忙循环是什么?  volatile 变量和 atomic 变量有什么不同？如果同步块内的线程抛出异常会发生什么？单例模式的双检锁是什么？（这个问题就是58面试问的，主要参考链接，链接中的方法很好，最好的方法在于使用内部类）

写出3条你遵循的多线程最佳实践？（这个问题回答的好，相当于多线程实践经验）

Java多线程中调用wait() 和 sleep()方法有什么不同？（卧槽，这里竟然有）

Java程序中wait 和 sleep都会造成某种形式的暂停，它们可以满足不同的需要。wait()方法用于线程间通信，如果等待条件为真且其它线程被唤醒时它会释放锁，而sleep()方法仅仅释放CPU资源或者让当前线程停止执行一段时间，但不会释放锁。

[https://javarevisited.blogspot.com/2011/12/difference-between-wait-sleep-yield.html](https://javarevisited.blogspot.com/2011/12/difference-between-wait-sleep-yield.html)  这个解释更牛逼

2. Threadlocal原理

[http://www.cnblogs.com/dolphin0520/p/3920407.html](http://www.cnblogs.com/dolphin0520/p/3920407.html)

这个对使用情景的解释很到位，因为ThreadLocal在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间互不影响，这样一来就不存在线程安全问题，也不会严重影响程序执行性能。

但是要注意，虽然ThreadLocal能够解决上面说的问题，但是由于在每个线程中都创建了副本，所以要考虑它对资源的消耗，比如内存的占用会比不使用ThreadLocal要大。（这是肯定的）

ThreadLocal是如何为每个线程创建变量的副本的：

首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。

初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。

然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

实际的使用中，通过重写initialValue()，代码更加合理，省去判断和防止空指针。最常见的ThreadLocal使用场景为用来解决  数据库连接（connection）、操作数据库的Session管理等。

3. spring的bean能否相互依赖？

结论：1\. field属性的循环依赖是没有问题的，spring可以解决；2\. 构造中的循环依赖是没法解决的，直接抛异常。

[https://blog.csdn.net/u010853261/article/details/77940767](https://blog.csdn.net/u010853261/article/details/77940767)  （具体分析过程，写得好，并且对spring创建bean的过程描述清晰，解决疑惑）

Spring的循环依赖的理论依据其实是基于Java的引用传递，当我们获取到对象的引用时，对象的field或则属性是可以延后设置的(但是构造器必须是在获取引用之前)。

三级缓存的解释非常好，就是三个大Map，singletonObjects、singletonFactories、earlySingletonObjects。

4. jvm内存模型

线程私有的：程序计数器、java虚拟机栈、本地方法栈

线程共享区域：java堆、方法区、运行时常量池（方法区的一部分）、直接内存（不是jvm一部分，但是这部分会频繁使用）

5. 保证事务上下文是一个，多线程是咋用的，完整介绍一个项目，

这个回头研究，尤其是涉及本地service调用，远程service调用，分布式如何保证！！

6. 对百万级表修改（比如金额增加10元），并保证如果一个商户的金额有多条记录大于10000元的，只有一条更新，如何设计？

这个考虑多线程，每个线程里面按商户查找记录，然后做修改，在修改过程中判断，是否已有一条记录做了修改，可以根据更新时间来过滤。

7. 说下spring的aop和ioc

8. 如何查找生产上执行慢的问题？

top, ps –aux | grep 进程, netstat –antl | grep 端口号, jstat –gcutil pid, jstack dump pid,

9. 怎么实现自旋锁，还有别的锁？

10. volatile的作用？

11. mq有哪些应用场景？缓存一致性协议？

12. 微服务的优缺点，springboot的特性，框架认识，

13. 大数据量处理，比方说对账，可以利用交易时间天然拆分，对于T+1的通道，可以一天多个时间段分开执行，要考虑一个业务线分布在不同通道。

14. 实际项目中的高并发问题，怎么处理，用什么技术，从架构的不同层次解决，比方说支付系统。常用解决方案，谈一些细节？

15. mysql主从复制，读写分离，搭过集群吗？

16. mybatis和hibernate区别，

17. springboot跟spring区别，

[https://blog.csdn.net/xiaobing_122613/article/details/54943448](https://blog.csdn.net/xiaobing_122613/article/details/54943448) springboot核心原理-自动配置

18. mvc和struts区别

### 58金融 6.9

1. 二叉树遍历（前序、中序、后序），有序数组查找元素，string赋值问题

2. jdk和cglib动态代理的原理和区别？

3. wait()，sleep()区别

4. 线程池执行原理，参数，阻塞队列有哪些？

5. 线程有哪些状态？（容易错，书中P21很经典）

6. countDownLatch？

[http://www.cnblogs.com/dolphin0520/p/3920397.html](http://www.cnblogs.com/dolphin0520/p/3920397.html)  这篇文章介绍了CountDownLatch，CyclicBarrier和Semaphore辅助类, 帮助我们进行并发编程.

CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行（类似于一个栅栏，只不过可以在多个线程中设置这个栅栏，让它们在指定的位置等待）。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

另外CyclicBarrier是可以重用的，

Semaphore翻译成字面意思为  信号量，Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而release() 释放一个许可。

以上工具类的使用，一般都是通过构造将这个工具类传入到线程任务中，然后在任务的执行代码中进行设置。

7. volatile作用？

[https://www.cnblogs.com/dolphin0520/p/3920373.html](https://www.cnblogs.com/dolphin0520/p/3920373.html)  这篇很强大

Java内存模型具备一些先天的“有序性”，即不需要通过任何手段就能够得到保证的有序性，这个通常也称为 happens-before 原则。如果两个操作的执行次序无法从happens-before原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。

8. Jvm的内存模型

9. 堆内存结构？

[https://www.cnblogs.com/langtianya/p/4441206.html](https://www.cnblogs.com/langtianya/p/4441206.html)  （堆和栈讨论）

10. Object类有哪些方法

11. 内存溢出和内存泄漏的区别？

[https://blog.csdn.net/u012813201/article/details/73793668](https://blog.csdn.net/u012813201/article/details/73793668)  （案例以及导致原因）

12. ConCurrentHashMap的底层结构？

13. 主键和唯一索引的区别？

[https://blog.csdn.net/baoqiangwang/article/details/4832814](https://blog.csdn.net/baoqiangwang/article/details/4832814)

14. redis和memcache的区别？

15. 说出知道的设计模式？

16. 跨域问题怎么解决？

17. 各种排序算法？考虑边界问题

18. interrupt()方法？（见源码demo）

[https://blog.csdn.net/u013238950/article/details/53893676](https://blog.csdn.net/u013238950/article/details/53893676)

19. 有几种类加载器？

20. 有几种锁？

[https://www.cnblogs.com/lxmyhappy/p/7380073.html](https://www.cnblogs.com/lxmyhappy/p/7380073.html)  这篇解释不错

21. 有关静态变量存在哪的问题？类初始化的触发条件？（这个看jvm部分）

[https://blog.csdn.net/stevendbaguo/article/details/51848228](https://blog.csdn.net/stevendbaguo/article/details/51848228)  （测试）

[https://blog.csdn.net/u012031380/article/details/54981472](https://blog.csdn.net/u012031380/article/details/54981472)  （java的堆，栈，静态代码区（常量区）详解）有点复杂

### 搜狐

1. mybatis如何能直接使用接口，底层如何实现？

2. dependencies和dependencyManagement的区别？

[https://blog.csdn.net/OiteBody/article/details/70882940](https://blog.csdn.net/OiteBody/article/details/70882940)  （解释的很好）

3. 定时任务的执行原理？quartz是分布式的吗？如何配置？如何知道触发器该执行了？

[https://www.cnblogs.com/zhangchengzhangtuo/p/5705672.html](https://www.cnblogs.com/zhangchengzhangtuo/p/5705672.html)

[https://blog.csdn.net/xlxxcc/article/details/52104463](https://blog.csdn.net/xlxxcc/article/details/52104463)

[https://www.jianshu.com/p/bab8e4e32952](https://www.jianshu.com/p/bab8e4e32952)  （比较给力）

Quartz 简单总结：

1、scheduler.shutdown(true); // true表示等到本次执行结束

2、为了配置强大时间调度策略，可以研究专门的CronTrigger。

3、整合Spring时，Quartz容器关闭方式有两种方法，一种是关闭- Spring容器，一种是获取到SchedulerFactoryBean实例，然后调用一个shutdown就搞定了。

4、Quartz的JobDetail、Trigger都可以在运行时重新设置，并且在下次调用时候起作用。这就为动态作业的实现提供了依据。你可以将调度时间策略存放到数据库，然后通过数据库数据来设定Trigger，这样就能产生动态的调度。

5、JobDetail不存储具体的实例，但它允许你定义一个实例，JobDetail 又指向JobDataMap。JobDetail持有Job的详细信息，如它所属的组，名称等信息

6、JobDataMap保存着任务实例的对象，并保持着他们状态信息，它是Ｍap接口的实现，即你可以往里面put和get一些你想存储和获取的信息．

7、scheduler容器包含多个JobDetail和Trigger。scheduler是个容器，容器中有一个线程池，用来并行调度执行每个作业，这样可以提高容器效率。

Quartz 运行过程

quartz运行时由QuartzSchedulerThread类作为主体，循环执行调度流程。JobStore作为中间层，按照quartz的并发策略执行数据库操作，完成主要的调度逻辑。JobRunShellFactory负责实例化JobDetail对象，将其放入线程池运行。LockHandler负责获取LOCKS表中的数据库锁。

整个quartz对任务调度的时序大致如下:

 0.调度器线程run()

 1.获取待触发trigger

 1.1读取JobDetail信息

 1.2读取trigger表中触发器信息并标记为”已获取”

 2.触发trigger

 2.1确认trigger的状态

 2.2读取trigger的JobDetail信息

 2.3读取trigger的Calendar信息

 2.4更新trigger信息

 3实例化并执行Job

 3.1从线程池获取线程执行JobRunShell的run方法

Quartz原理分析系列文章：

[http://royliu.me/2017/04/13/quartz2-x-code-analyse-startprocess/](http://royliu.me/2017/04/13/quartz2-x-code-analyse-startprocess/)

4. spring怎么找到bean的？

5. dubbo消费者怎么发现新启用的服务？

6. zk实现分布式锁？自旋锁？

7. jdk和cglib动态代理的原理？静态代理？

8. netty的执行原理（大概），主要的类有哪些？

9. 字符串比较，判断是大写还是小写？

10. hashCode()和equals()方法的关系，使用意义，比方说在hashMap中，

11. concurrentHashMap中size()方法加锁范围？

12. 可重入锁？重量级锁？轻量级锁？自旋锁？等

[https://blog.csdn.net/u013174217/article/details/78270248?locationnum=7&fps=1](https://blog.csdn.net/u013174217/article/details/78270248?locationnum=7&fps=1)  （可重入）

13. 对象锁，类锁分别存在哪里？（这个题有点变态）

14. 显式锁的原理？（读写锁的原理？）

15. 索引种类有哪些？聚合索引？何时失效？索引失效的原因？

16. 堆排序？快速排序？

17. jvm加载类过程涉及到内存操作？对象创建到回收销毁涉及到的内存操作过程？

18. spring框架是线程安全的吗？

19. group by 后的having过滤原理？

20. 线程池队列有界吗？线程池工作原理

21. linux命令，筛选日志，根据多个字段？vim中查找替换，翻页到末尾，删除连续几行（前几行），查询某句话前后几行，grep –n含义，查看端口连接线程数？

22. volatile作用？单例中使用内部静态类如何实现懒加载？

23. 使用countDownLatch和仅仅使用callable的Future区别，好处是什么？

### 每日优鲜

1. 内部类的初始化静态字段顺序问题（联想到servlet的初始化过程）

[https://www.cnblogs.com/heben/p/5750079.html](https://www.cnblogs.com/heben/p/5750079.html)

[https://blog.csdn.net/u013257679/article/details/52053567](https://blog.csdn.net/u013257679/article/details/52053567)

2. redis从一个亿key中找十万个前缀相同的key，（哥们说有个scan命令可以）

[https://blog.csdn.net/zhang197093/article/details/74615717](https://blog.csdn.net/zhang197093/article/details/74615717)

[https://www.jb51.net/article/138394.htm](https://www.jb51.net/article/138394.htm)

[https://www.cnblogs.com/Eivll0m/p/4553377.html](https://www.cnblogs.com/Eivll0m/p/4553377.html)  （使用redis的monitor命令排除故障）

redis的monitor命令可以实时打印出 redis 服务器接收到的命令，我们就通过这些数据进行分析。

3. mvc的启动原理

4. redis数据类型

5. 请求排队机制

6. 分表，有没有用过中间件

7. 如何查看线上问题，服务响应慢，查看端口连接数，查看哪种连接？这个涉及连接种类，定位耗时？数据库连接慢，系统上怎么试一下？

[https://blog.csdn.net/u012151556/article/details/45539665](https://blog.csdn.net/u012151556/article/details/45539665)  （查找线上问题常用命令）

1） 查看cup最多的进程（两种方法）

2） 在1）的基础上查对应的服务名是什么（两种方法）

3） 查看某个端口的连接情况，因为上面已经发现是哪个服务，我们可以根据服务配置的端口号，再查看连接。（也有两种方法）

[https://blog.csdn.net/u010827436/article/details/46564673](https://blog.csdn.net/u010827436/article/details/46564673) （线上命令）

[https://blog.csdn.net/zxp_cpinfo/article/details/54971115](https://blog.csdn.net/zxp_cpinfo/article/details/54971115) （通过jstack日志分析和问题排查）

[https://www.aliyun.com/jiaocheng/126984.html](https://www.aliyun.com/jiaocheng/126984.html)

[https://blog.csdn.net/u010827436/article/details/46564673](https://blog.csdn.net/u010827436/article/details/46564673)

通过redis的monitor命令排除故障：  [https://www.cnblogs.com/Eivll0m/p/4553377.html](https://www.cnblogs.com/Eivll0m/p/4553377.html)

8. 怎么部署项目，https的原理和区别（协议的问题），简单找相同算法

那个图还是很好理解的，主要是获取到公钥之后，产生一个随机的（对称）秘钥，然后用公钥对这个随机秘钥加密，同时使用随机对称秘钥对报文加密，然后先将使用公钥加密后的随机秘钥发送过去，然后将加密后的报文发过去。

9. mybatis的二级缓存什么时候释放？如何配置二级缓存？

[https://blog.csdn.net/zmx729618/article/details/77991916](https://blog.csdn.net/zmx729618/article/details/77991916)

10. 各种异常、错误的区别？？

### 猎聘

1. Long值的比较

2. 如何实现在10min内高频访问拦截黑名单

3. 如何查找大数据量表的数据

4. 请求转发和重定向的区别？拦截器对于这两种，还会拦截吗？

5. 知道哪些设计模式

6. 手写一个单例模式

7. git如何检出不同版本，使用不同环境配置部署项目，git只要是push了，是不是都能会退到之前的版本

### 美团

1. 定时任务怎么分布式实现，多台机器，防止单点问题？quartz自己就能实现吗？

2. 线上排查问题如何定位（使用什么命令，思路），如何准确地找到业务处理耗时地方（jstack，jmap了解吗，线上怎么用？），jvm性能调优？

3. redis怎么用的，解决了什么问题

4. 使用的哪种线程池，线程池原理，阻塞队列？有哪些拒绝策略？（回答的不好）

5. 虚拟机内存，堆内存结构？

6. 垃圾回收过程，MinorGC，FullGC，你说的这种属于哪种垃圾回收器？

7. HashMap原理，哪些线程安全的map，concurrentHash，hashTable原理

8. 两种锁的区别，synchronized同步的原理，reentranLock的原理？

9. synchronized()同步块传入不同的值，同步范围有啥区别？

10. AtomicInteger原理（或是同包别的），怎么实现线程安全的？怎么实现i++的，CAS原理？

11. mysql索引数据结构（B+tree），索引原理，如何查看sql是否使用了索引？

12. 数据库做了哪些优化？对于数据量大的表做了什么优化？

13. 如果是两个或三个字段的联合索引，我只用最左边或右边的字段查，索引失效吗?

14. volatile有哪些使用场景？怎么用？作用是啥？volatile能保证i++安全吗？

15. 项目上遇到哪些难解决的问题？哪个项目用到了比较新的解决方案？

16. 如何实现sso单点登录，token存在哪，前端存在哪（cookie？），sso实现原理？

17. 请求的响应码都有哪些？

18. 分布式事务是什么？应用场景？保证一致性的解决方案？

19. dubbo如何实现远程调用的？（原理）如果有一台服务挂掉，消费者怎么知道？消费者怎么请求服务者的？dubbo如何实现注册中心的？dubbo协议是什么？

[https://www.cnblogs.com/gotodsp/p/6532856.html](https://www.cnblogs.com/gotodsp/p/6532856.html) 这篇关于调用过程原理不错

20. 怎么自己实现一个服务治理的注册中心？如何防止单点问题？zk的心跳检测机制是什么？服务注册如何实现？

21. 对zk了解多吗？

22. 自旋锁怎么实现？

23. 如何查看服务load过大？load过大怎么解决？

24. 聚集索引和非聚集索引，索引和表的锁之间关系，行锁、表锁、间隙锁！！

如果没加索引，写操作会加表锁；如果加了索引，只是加了间隙锁或行锁。

25. 探讨题：分布式事务问题，是不是因操作不同的库造成的，还是操作不同的表也会存在这个问题？导致原因？解决方案和原理？

### 猫眼电影

1. static关键字的作用，static修饰的什么时候初始化？包括静态内部类

2. 反射机制是什么

3. gc过程，有没有查看gc和调优方面的经验

4. jdk8对hashMap做了什么改进

5. hashMap为何不是线程安全，会造成什么后果

6. 显式锁的区别

7. 什么是可重入锁？

8. springMVC如何处理一个请求？

9. 内存角度理解多线程并发

10. spring的两大特性

11. 说一下springboot框架执行原理

12. dubbo调用是不是有状态的？

13. Btree和B+tree的区别

14. 使用top命令，有三个数代表啥

最近5、10、15分钟的平均负载

15. sso单点登录怎么实现的？知道CAS框架吗

16. rabbitmq有什么优点

17. 线程池的原理？你用的是哪种线程池？有哪几种？为什么选这种？关键参数？（存活时间）过期时间是什么过期？线程池为什么要有存活时间这样的设计？

18. xml解析模版怎么用的？项目还有哪些点可以优化？

19. mysql怎么存数据的？索引结构了解吗？

20. 写一个二叉树前序遍历？对二叉树每层统计值存入List<>中，最终都存入List<List<>>算法？实现字符串”123”转int123？

21. 算法，将一个数组的0都移动到末尾？单项链表翻转？将两个有序数组合并，求中间值？（利用折半查找如何实现）

22. 最大的表数据量有多大，如何考虑分表，跨表查询如何解决？

23. 设计题，对于数据量很大的订单表，考虑以后会有商户和日期两个维度的查询，如何分表，如何对不同维度查询？尤其是比方说查询一个商户所有订单这种跨表的实现？

24. 设计题，一个生产者生产消息，多个消费者消费消息，如何保证这些消费者都能消费每个消息，并且这些消费者更新数组能近实时地同步？考虑某台消费者宕机了重启的情况，主要是近实时？（提示：考虑push和get方式）

### 国美

1. 讲下spring的aop，aspectj的优势？切面编程的原理是啥

2. spring的事务隔离级别有哪些？传播机制呢？

3. 用的哪种线程池，主要参数，原理

4. 为什么单线程的redis可以高并发？redis事务？redis事务与数据库事务区别？

5. 什么是索引最左前缀原则？聚集索引和非聚集索引？还有覆盖索引？where b=? and a=? 索引有效吗？订单表的status有必要加索引吗？

6. 表和sql的优化手段？

7. 后台系统，A,B用户同时修改一篇新闻，A用户提交修改的内容，更新数据库数据，过几分钟后，B用户提交修改后的内容会覆盖A用户修改的内容，请问怎么避免A用户修改的内容被B用户覆盖？

8. 远程通信有几种方式，分别有什么区别和联系，分别用什么框架实现

9. 列出3种计算字符串相似度的方法

10. sql优化需要考虑哪几点？

11. 提高性能的方案？

12. 如何解决表的死锁？

13. 目前一些系统大量用到了BTree树，请写出BTree树的节点定义的结构，并说明他的时间复杂度由哪些因子决定的？

14. 集群服务器抽奖功能：0-9999之间的数字，每个用户随机抽取不能重复的数字（算法）？

[https://www.cnblogs.com/savorboard/p/distributed-system-transaction-consistency.html](https://www.cnblogs.com/savorboard/p/distributed-system-transaction-consistency.html)（分布式事务）

[https://www.cnblogs.com/felixzh/p/5869212.html](https://www.cnblogs.com/felixzh/p/5869212.html) （Zookeeper的功能以及工作原理）

这里头关于leader的选举算法很有意思，具体可以参照书上7.6章节的讲解

[https://www.jianshu.com/p/1ff25f65587c](https://www.jianshu.com/p/1ff25f65587c) （dubbo服务注册与发现、服务调用过程）

[https://blog.csdn.net/aqzwss/article/details/53074186](https://blog.csdn.net/aqzwss/article/details/53074186) （B-树，B+树，B*树详解）

[https://www.cnblogs.com/dolphin0520/p/3920373.html](https://www.cnblogs.com/dolphin0520/p/3920373.html) （Java并发编程：volatile关键字解析）

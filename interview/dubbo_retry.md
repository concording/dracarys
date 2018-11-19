# 集群容错

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover ，即失败重试。可通过接口`com.alibaba.dubbo.rpc.cluster.Cluster`的SPI注解可知：

```
/**
 * Cluster. (SPI, Singleton, ThreadSafe)
 * 
 * <a href="http://en.wikipedia.org/wiki/Computer_cluster">Cluster</a>
 * <a href="http://en.wikipedia.org/wiki/Fault-tolerant_system">Fault-Tolerant</a>
 * 
 * @author william.liangf
 */
@SPI(FailoverCluster.NAME)
public interface Cluster {
    ...
}

```

接下来通过对dubbo源码的分析，一一讲解这些集群容错模式的具体实现；

## 集群调用关系

![](http://upload-images.jianshu.io/upload_images/6918995-7a7812889711d777.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600/format/webp)

cluster.jpg

图片来源: [https://dubbo.gitbooks.io/dubbo-user-book/demos/fault-tolerent-strategy.html](https://dubbo.gitbooks.io/dubbo-user-book/demos/fault-tolerent-strategy.html)
由图可知，通过Cluster的调用过程如下：

1.  调用list()从Directory中取得可用Invoker集合；
2.  根据路由规则过滤一些Invoker，得到可用Invoker集合；
3.  根据负载均衡机制得到一个合适的Invoker，[负载均衡机制参考](https://www.jianshu.com/p/10c30d7b8b6a)
4.  调用最终选出来的这个Invoker。

## 集群模式配置

按照以下示例在服务提供方和消费方配置集群模式

> `<dubbo:service cluster="failover" />`
> 或
> `<dubbo:reference cluster="failsafe" />`

## 集群模式概览

dubbo支持的集群模式如下图所示，由于dubbo通过SPI实现微内核，集群模式也不例外，所以想扩展自己对集群容错的处理方式，非常简单；

![](http://upload-images.jianshu.io/upload_images/6918995-6c2e3fc015266e5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

dubbo集群容错总览

接下来通过对源码的阅读，一一分析各个集群容错模式的实现；

### Failover Cluster

dubbo默认集群模式，失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟，且使集群的压力更大。可通过 retries="2" 来设置重试次数（默认为2，这个值是重试次数，所以不包括第一次调用，而是第一次调用失败后最大可重试次数）。重试次数配置示例如下：

> `<dubbo:service retries="2" />`
> 或

> `<dubbo:reference retries="2" />`
> 或
> `<dubbo:reference><dubbo:method name="findFoo" retries="2" /></dubbo:reference>`

核心实现源码：

```
public Result doInvoke(Invocation invocation, final List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    List<Invoker<T>> copyinvokers = invokers;
    // 检查copyinvokers即可用Invoker集合是否为空，如果为空，那么抛出异常
    checkInvokers(copyinvokers, invocation);
    // 得到最大可调用次数：最大可重试次数+1，默认最大可重试次数Constants.DEFAULT_RETRIES=2
    int len = getUrl().getMethodParameter(invocation.getMethodName(), Constants.RETRIES_KEY, Constants.DEFAULT_RETRIES) + 1;
    // 如果用户设置reties为负数，那么也要调用至少1次
    if (len <= 0) {
        len = 1;
    }
    // 保存最后一次调用的异常
    RpcException le = null;
    // 保存已经调用过的Invoker
    List<Invoker<T>> invoked = new ArrayList<Invoker<T>>(copyinvokers.size()); // invoked invokers.
    Set<String> providers = new HashSet<String>(len);
    // failover机制核心实现：如果出现调用失败，那么重试其他服务器
    for (int i = 0; i < len; i++) {
        //重试时，进行重新选择，避免重试时invoker列表已发生变化.
        //注意：如果列表发生了变化，那么invoked判断会失效，因为invoker示例已经改变
        if (i > 0) {
            checkWheatherDestoried();
            // 根据Invocation调用信息从Directory中获取所有可用Invoker
            copyinvokers = list(invocation);
            //重新检查一下
            checkInvokers(copyinvokers, invocation);
        }
        // 根据负载均衡机制从copyinvokers中选择一个Invoker
        Invoker<T> invoker = select(loadbalance, invocation, copyinvokers, invoked);
        // 保存每次调用的Invoker
        invoked.add(invoker);
        RpcContext.getContext().setInvokers((List)invoked);
        try {
            // RPC调用得到Result
            Result result = invoker.invoke(invocation);
            // 重试过程中，将最后一次调用的异常信息以warn级别日志输出
            if (le != null && logger.isWarnEnabled()) {
                logger.warn("Although retry the method " + invocation.getMethodName()
                        + " in the service " + getInterface().getName()
                        + " was successful by the provider " + invoker.getUrl().getAddress()
                        + ", but there have been failed providers " + providers 
                        + " (" + providers.size() + "/" + copyinvokers.size()
                        + ") from the registry " + directory.getUrl().getAddress()
                        + " on the consumer " + NetUtils.getLocalHost()
                        + " using the dubbo version " + Version.getVersion() + ". Last error is: "
                        + le.getMessage(), le);
            }
            return result;
        } catch (RpcException e) {
            // 如果是业务性质的异常，不再重试，直接抛出
            if (e.isBiz()) { // biz exception.
                throw e;
            }
            le = e;
        } catch (Throwable e) {
            // 其他性质的异常统一封装成RpcException
            le = new RpcException(e.getMessage(), e);
        } finally {
            providers.add(invoker.getUrl().getAddress());
        }
    }
    // 最大可调用次数用完还得到Result的话，抛出RpcException异常：重试了N次还是失败，并输出最后一次异常信息
    throw new RpcException(le != null ? le.getCode() : 0, "Failed to invoke the method "
            + invocation.getMethodName() + " in the service " + getInterface().getName() 
            + ". Tried " + len + " times of the providers " + providers 
            + " (" + providers.size() + "/" + copyinvokers.size() 
            + ") from the registry " + directory.getUrl().getAddress()
            + " on the consumer " + NetUtils.getLocalHost() + " using the dubbo version "
            + Version.getVersion() + ". Last error is: "
            + (le != null ? le.getMessage() : ""), le != null && le.getCause() != null ? le.getCause() : le);
}

```

### Failfast Cluster

快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。
核心实现源码：

```
public Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    checkInvokers(invokers, invocation);
    Invoker<T> invoker = select(loadbalance, invocation, invokers, null);
    try {
        return invoker.invoke(invocation);
    } catch (Throwable e) {
        if (e instanceof RpcException && ((RpcException)e).isBiz()) { // biz exception.
            throw (RpcException) e;
        }
        throw new RpcException(e instanceof RpcException ? ((RpcException)e).getCode() : 0, "Failfast invoke providers " + invoker.getUrl() + " " + loadbalance.getClass().getSimpleName() + " select from all providers " + invokers + " for service " + getInterface().getName() + " method " + invocation.getMethodName() + " on consumer " + NetUtils.getLocalHost() + " use dubbo version " + Version.getVersion() + ", but no luck to perform the invocation. Last error is: " + e.getMessage(), e.getCause() != null ? e.getCause() : e);
    }
}

```

> FailfastCluster实现比较简单，根据负载均衡机制选择一个Invoker后只调用1次，不管结果如何，不再进行任何重试：如果调用正常就返回Result，否则返回<u>记录了详细异常信息的**RpcException**</u>；

### Failsafe Cluster

失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。
核心实现源码：

```
public Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    try {
        checkInvokers(invokers, invocation);
        Invoker<T> invoker = select(loadbalance, invocation, invokers, null);
        return invoker.invoke(invocation);
    } catch (Throwable e) {
        logger.error("Failsafe ignore exception: " + e.getMessage(), e);
        return new RpcResult(); // ignore
    }
}

```

> FailsafeCluster实现比较简单，根据负载均衡机制选择一个Invoker后只调用1次，不管结果如何，不再进行任何重试：如果调用正常就返回Result，否则返回<u>一个空的**RpcResult**</u>，这是和**FailfastCluster**的唯一区别，不会把任何异常信息返回给consumer；

### Failback Cluster

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。
核心实现源码：

```
protected Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    try {
        checkInvokers(invokers, invocation);
        Invoker<T> invoker = select(loadbalance, invocation, invokers, null);
        return invoker.invoke(invocation);
    } catch (Throwable e) {
        logger.error("Failback to invoke method " + invocation.getMethodName() + ", wait for retry in background. Ignored exception: "
                             + e.getMessage() + ", ", e);
        // failback实现的核心，如果调用失败，后台记录失败请求，并定时重发
        addFailed(invocation, this);
        return new RpcResult(); // ignore
    }
}

```

`定时重发`核心实现源码：

```
// 处理重试任务的线程池
private final ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(2, new NamedThreadFactory("failback-cluster-timer", true));

private void addFailed(Invocation invocation, AbstractClusterInvoker<?> router) {
    if (retryFuture == null) {
        // double-check保证线程安全
        synchronized (this) {
            if (retryFuture == null) {
                // 一个独立的线程池处理，执行周期是5s
                retryFuture = scheduledExecutorService.scheduleWithFixedDelay(new Runnable() {
                    public void run() {
                        // 收集统计信息
                        try {
                            // 重试失败的请求，如果重试成功，把请求从remove掉；
                            retryFailed();
                        } catch (Throwable t) { // 防御性容错
                            logger.error("Unexpected error occur at collect statistic", t);
                        }
                    }
                }, RETRY_FAILED_PERIOD, RETRY_FAILED_PERIOD, TimeUnit.MILLISECONDS);
            }
        }
    }
    failed.put(invocation, router);
}

```

### Forking Cluster

并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks="2" 来设置最大并行数。
核心实现源码：

```
public Result doInvoke(final Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    checkInvokers(invokers, invocation);
    final List<Invoker<T>> selected;
    // forks数，默认为2
    final int forks = getUrl().getParameter(Constants.FORKS_KEY, Constants.DEFAULT_FORKS);
    // 请求超时
    final int timeout = getUrl().getParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
    // 如果设置的forks值为负数，或者超过了可用Invoker数，那么选择所有可用Invoker，即invokers
    if (forks <= 0 || forks >= invokers.size()) {
        selected = invokers;
    } else {
        selected = new ArrayList<Invoker<T>>();
        // 只选择forks值指定的Invoker数量
        for (int i = 0; i < forks; i++) {
            //在invoker列表(排除selected)后,如果没有选够,则存在重复循环问题.见select实现.
            Invoker<T> invoker = select(loadbalance, invocation, invokers, selected);
            if(!selected.contains(invoker)){//防止重复添加invoker
                selected.add(invoker);
            }
        }
    }
    RpcContext.getContext().setInvokers((List)selected);
    final AtomicInteger count = new AtomicInteger();
    final BlockingQueue<Object> ref = new LinkedBlockingQueue<Object>();
    for (final Invoker<T> invoker : selected) {
        // ForkingCluster核心实现，多线程并行调用
        executor.execute(new Runnable() {
            public void run() {
                try {
                    Result result = invoker.invoke(invocation);
                    // 把结果放到BlockingQueue中
                    ref.offer(result);
                } catch(Throwable e) {
                    int value = count.incrementAndGet();
                    if (value >= selected.size()) {
                        ref.offer(e);
                    }
                }
            }
        });
    }
    try {
        // 从BlockingQueue中取结果：即并行调用最先返回的结果
        Object ret = ref.poll(timeout, TimeUnit.MILLISECONDS);
        // 如果取得的是异常，那么将异常封装成RpcException并抛给Consumer
        if (ret instanceof Throwable) {
            Throwable e = (Throwable) ret;
            throw new RpcException(e instanceof RpcException ? ((RpcException)e).getCode() : 0, "Failed to forking invoke provider " + selected + ", but no luck to perform the invocation. Last error is: " + e.getMessage(), e.getCause() != null ? e.getCause() : e);
        }
        return (Result) ret;
    } catch (InterruptedException e) {
        // 如果timeout指定超时时间内还没有返回结果，那么将异常封装成RpcException并抛给Consumer
        throw new RpcException("Failed to forking invoke provider " + selected + ", but no luck to perform the invocation. Last error is: " + e.getMessage(), e);
    }
}

```

### Broadcast Cluster

广播调用所有提供者，逐个调用，任意一台报错则报错 。通常用于通知所有提供者更新缓存或日志等本地资源信息。
核心实现源码：

```
public Result doInvoke(final Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    checkInvokers(invokers, invocation);
    RpcContext.getContext().setInvokers((List)invokers);
    // 保存最后一个调用的异常
    RpcException exception = null;
    Result result = null;
    for (Invoker<T> invoker: invokers) {
        try {
            // 遍历所有Invoker，每个Invoker都会被调用（不管某个Invoker是否抛出异常）
            result = invoker.invoke(invocation);
        } catch (RpcException e) {
            exception = e;
            logger.warn(e.getMessage(), e);
        } catch (Throwable e) {
            exception = new RpcException(e.getMessage(), e);
            logger.warn(e.getMessage(), e);
        }
    }
    // 如果调用过程有异常，那么抛出该异常
    if (exception != null) {
        throw exception;
    }
    return result;
}

```

### Available Cluster

遍历所有从**Directory**中list出来的**Invoker**集合，调用第一个`isAvailable()`的**Invoker**，只发起一次调用，失败立即报错。
`isAvailable()`判断逻辑如下--Client处理连接状态，且不是READONLY：

```
@Override
public boolean isAvailable() {
    if (!super.isAvailable())
        return false;
    for (ExchangeClient client : clients){
        if (client.isConnected() && !client.hasAttribute(Constants.CHANNEL_ATTRIBUTE_READONLY_KEY)){
            //cannot write == not Available ?
            return true ;
        }
    }
    return false;
}

```

### Mergeable Cluster

请戳链接[14\. dubbo源码-集群容错之MergeableCluster](https://www.jianshu.com/p/512e2211f84c)

作者：阿飞的博客
链接：https://www.jianshu.com/p/454019601f9f
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
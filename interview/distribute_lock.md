# 1 分布式锁的疑问

谈到分布式锁，有很多实现方式，如数据库、redis、ZooKeeper等。提个问题：

*   实现分布式锁需要满足哪些条件呢？

# 2 数据库实现分布式锁

## 2.1 实现案例

如使用数据库事务中的锁如record lock来实现，如下所示

1 获取锁

```
public void lock(){
    connection.setAutoCommit(false)
    int count = 0;
    while(count < 4){
        try{
            select * from lock where lock_name=xxx for update;
            if(结果不为空){
                //代表获取到锁
                return;
            }
        }catch(Exception e){

        }
        //为空或者抛异常的话都表示没有获取到锁
        sleep(1000);
        count++;
    }
    throw new LockException();
}

```

2 释放锁

```
public void release(){
    connection.commit();
}

```

数据库的lock表，lock_name是主键,通过for update操作，数据库就会对该行记录加上record lock，从而阻塞其他人对该记录的操作。

一旦获取到了锁，就可以开始执行业务逻辑，最后通过connection.commit()操作来释放锁。

其他没有获取到锁的就会阻塞在上述select语句上，可能的结果有2种，在超时之前获取到了锁，在超时之前仍未获取到锁（这时候会抛出超时异常，然后进行重试）

数据库当然还有其他方式，如插入一个有唯一约束的数据。成功插入则表示获取到了锁，释放锁就是删除该记录。该方案也有很多问题要解决

## 2.2 存在的问题

首先性能不是特别高。

通过数据库的锁来实现多进程之间的互斥，但是这貌似也有一个问题：就是sql超时异常的问题

jdbc超时具体有3种超时，具体见[深入理解JDBC的超时设置](http://mp.weixin.qq.com/s?__biz=MjM5NzMyMjAwMA==&mid=2651477438&idx=1&sn=53c26a85d0af8c18a6f9d15d242c01be&scene=0#wechat_redirect)

*   框架层的事务超时
*   jdbc的查询超时
*   Socket的读超时

这里只涉及到后2种的超时，jdbc的查询超时还好（mysql的jdbc驱动会向服务器发送kill query命令来取消查询），如果一旦出现Socket的读超时，对于如果是同步通信的Socket连接来说(底层实现Connection的可能是同步通信也可能是异步通信)，该连接基本上不能使用了，需要关闭该连接，从新换用新的连接，因为会出现请求和响应错乱的情况，比如jedis出现的类型转换异常，详见[Jedis的类型转换异常深究](https://my.oschina.net/pingpangkuangmo/blog/737122)

# 3 redis实现分布式锁

而redis通常可以使用setnx来实现分布式锁

## 3.1 基本版

1 获取锁

```
public void lock(){
    for(){
        ret = setnx lock_ley (current_time + lock_timeout)
        if(ret){
            //获取到了锁
            break;
        }
        //没有获取到锁
        sleep(100);
    }
}

```

2 释放锁

```
public void release(){
    del lock_ley
}

```

setnx来创建一个key，如果key不存在则创建成功返回1，如果key已经存在则返回0。依照上述来判定是否获取到了锁

获取到锁的执行业务逻辑，完毕后删除lock_key，来实现释放锁

其他未获取到锁的则进行不断重试，直到自己获取到了锁

## 3.2 改进版

上述逻辑在正常情况下是OK的，但是一旦获取到锁的客户端挂了，没有执行上述释放锁的操作，则其他客户端就无法获取到锁了，所以在这种情况下有2种方式来解决：

*   为lock_key设置一个过期时间
*   对lock_key的value进行判断是否过期

以第一种为例，在set键值的时候带上过期时间，即使挂了，也会在过期时间之后，其他客户端能够重新竞争获取锁

```
public void lock(){
    while(true){
        ret = set lock_key identify_value nx ex lock_timeout
        if(ret){
            //获取到了锁
            return;
        }
        sleep(100);
    }
}

public void release(){
    value = get lock_key
    if(identify_value == value){
        del lock_key
    }
}

```

以第二种为例，一旦发现lock_key的值已经小于当前时间了，说明该key过期了，然后对该key进行getset设置，一旦getset返回值是原来的过期值，说明当前客户端是第一个来操作的，代表获取到了锁，一旦getset返回值不是原来过期时间则说明前面已经有人修改了，则代表没有获取到锁，详细见[用Redis实现分布式锁](http://www.jeffkit.info/2011/07/1000/)，改正如下：

```
# get lock
lock = 0
while lock != 1:
    timestamp = current_unix_time + lock_timeout
    lock = SETNX lock.foo timestamp
    if lock == 1 or (now() > (GET lock.foo) and now() > (GETSET lock.foo timestamp)):
        break;
    else:
        sleep(10ms)

# do your job
do_job()

# release
if now() < GET lock.foo:
    DEL lock.foo

```

这里看来第二种其实没有第一种比较好。

## 3.3 问题依旧

问题1： lock timeout的存在也使得失去了锁的意义，即存在并发的现象。一旦出现锁的租约时间，就意味着获取到锁的客户端必须在租约之内执行完毕业务逻辑，一旦业务逻辑执行时间过长，租约到期，就会引发并发问题。所以有lock timeout的可靠性并不是那么的高。

问题2： 上述方式仅仅是redis单机情况下，还存在redis单点故障的问题。如果为了解决单点故障而使用redis的sentinel或者cluster方案，则更加复杂，引入的问题更多。

# 4 ZooKeeper实现分布式锁

## 4.1 案例

这也是ZooKeeper客户端curator的分布式锁实现。

1 获取锁

```
public void lock(){
    path = 在父节点下创建临时顺序节点
    while(true){
        children = 获取父节点的所有节点
        if(path是children中的最小的){
            代表获取了节点
            return;
        }else{
            添加监控前一个节点是否存在的watcher
            wait();
        }
    }
}

watcher中的内容{
    notifyAll();
}

```

2 释放锁

```
public void release(){
    删除上述创建的节点
}

```

## 4.2 总结

ZooKeeper版本的分布式锁问题相对比较来说少。

*   **锁的占用时间限制**：redis就有占用时间限制，而ZooKeeper则没有，最主要的原因是redis目前没有办法知道已经获取锁的客户端的状态，是已经挂了呢还是正在执行耗时较长的业务逻辑。而ZooKeeper通过临时节点就能清晰知道，如果临时节点存在说明还在执行业务逻辑，如果临时节点不存在说明已经执行完毕释放锁或者是挂了。由此看来redis如果能像ZooKeeper一样添加一些与客户端绑定的临时键，也是一大好事。
*   是否单点故障：redis本身有很多中玩法，如客户端一致性hash，服务器端sentinel方案或者cluster方案，**很难做到一种分布式锁方式能应对所有这些方案**。而ZooKeeper只有一种玩法，多台机器的节点数据是一致的，没有redis的那么多的麻烦因素要考虑。

总体上来说ZooKeeper实现分布式锁更加的简单，可靠性更高。

# 5 分布式锁实现原理总结

从上面我们经历了3种实现方式，可以从中总结下，该怎么去回答最初提出的问题。

## 5.1 分布式锁的实现

在我自己看来有如下3个方面：

*   怎么获取锁
*   怎么释放锁
*   怎么得知锁被释放了

## 5.1.1 怎么获取锁

**能够提供一种方式，多个客户端并发操作，只能有一个客户端能满足相应的要求**

如数据库的for update的sql语句、或者插入一个含有唯一约束的数据等

如redis的setnx等

如ZooKeeper的求最小节点的方式

这些都可以保证只能有一个客户端获取到了锁

## 5.1.2 怎么释放锁

场景一般有2种情况：

*   1 正常情况下的释放锁
*   2 异常情况下如何释放锁（即释放锁的操作没有被执行，如挂掉、没执行成功等原因）

如redis正常情况下释放锁是删除lock_key，异常情况下，只能通过lock_key的超时时间了

如ZooKeeper正常情况下释放锁是删除临时节点，异常情况下，服务器也会主动删除临时节点（这种机制就简单多了）

## 5.1.3 怎么得知锁被释放了

实现方式一般有2种情况：

*   1 没有获取到锁的客户端不断尝试获取锁
*   2 服务器端通知客户端锁被释放了

当然第二种情况是最优的（客户端所做的无用功最少），如ZooKeeper通过注册watcher来得到锁释放的通知。而数据库、redis没有办法来通知客户端锁释放了，那客户端就只能傻傻的不断尝试获取锁了。
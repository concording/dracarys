## 蚂蚁金服通信框架SOFABolt解析|超时控制机制及心跳机制

原创： SOFALab 胡萝卜 [金融级分布式架构](javascript:void(0);) _今天_

> **SOFA** 
> 
> **S**calable **O**pen **F**inancial **A**rchitecture
> 
> 是蚂蚁金服自主研发的金融级分布式中间件，包含了构建金融级云原生架构所需的各个组件，是在金融场景里锤炼出来的最佳实践。

###   前言

SOFABolt是一个基于 Netty 最佳实践的轻量、易用、高性能、易扩展的通信框架。目前已经运用在了蚂蚁中间件的微服务，消息中心，分布式事务，分布式开关，配置中心等众多产品上。

本文将分析SOFABolt的超时控制和心跳机制。

###   超时

在程序中，超时一般指的是程序在特定的等待时间内没有得到响应，网络通信问题、程序BUG等等都会引起超时。系统引入超时机制往往是为了解决资源的问题，比如一个同步RPC请求，在网络不稳定的情况下可能一直无法得到响应，那么请求线程将一直等待结果而无法执行其它任务，最终导致所有线程资源耗尽。超时机制正是为了解决这样的问题，在特定的等待时间之后触发一个“超时事件”来释放资源。

在一个网络通信框架中，超时问题无处不在，连接的建立、数据的读写都可能遇到超时问题。并且网络通信框架作为分布式系统的底层组件，需要管理大量的连接，如何建立一个高效的超时处理机制就成为了一个问题。

###   时间轮（TimeWheel）

在网络通信框架中动辄管理上万的连接，每个连接上都有很多的超时任务，如果每个超时任务都启动一个java.util.Timer，不仅低效而且会占用大量的资源。George Varghese 和 Tony Lauck在1996年发表了一篇论文：《Hashed and Hierarchical Timing Wheels: EfficientData Structures for Implementing a Timer Facility》来高效的管理和维护大量的定时任务。

![](https://mmbiz.qpic.cn/mmbiz_jpg/nibOZpaQKw0icbdNVkDpX1ya2yrc5tCsMXSFpsM3hocVv6X9QKib8MSGlPOJGSG3WPRCGd0ylUZ4L8jqvic9xX7TlQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "timewheel.jpg | left | 412x227")

时间轮其实就是一种环形的数据结构，可以理解为时钟，每个格子代表一段时间，每次指针跳动一格就表示一段时间的流逝（就像时钟分为60格，秒针没跳动一格代表一秒钟）。时间轮每一格上都是一个链表，表示对应时间对应的超时任务，每次指针跳动到对应的格子上则执行链表中的超时任务。时间轮只需要一个线程执行指针的“跳动”来触发超时任务，且超时任务的插入和取消都是O(1)的操作，显然比java.util.Timer的方式要高效的多。

###   SOFABolt的超时控制机制

![](https://mmbiz.qpic.cn/mmbiz_png/nibOZpaQKw0icbdNVkDpX1ya2yrc5tCsMXruafhjo75lWzZaU3SwcgA93aic7Urib0nhYZL8ibcco5y84acbAiakb9GQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png | left | 631x583")

如上图所示，SOFABolt中支持四中调用方式：

*   oneway：不关心调用结果，所以不需要等待响应，那么就没有超时

*   sync：同步调用，在调用线程中等待响应

*   future：异步调用，返回future，由用户从future中获取结果

*   callback：异步调用，异步执行用户的callback
    在oneway调用中，因为并不关心响应结果，所以没有超时的概念。下面具体介绍SOFABolt中同步调用（sync）和异步调用（future\callback）的超时机制实现。

### 同步调用的超时控制实现

同步调用中，每一次调用都会阻塞调用线程等待服务端的响应，这种场景下同一时刻产生最大的超时任务取决于调用线程的数量。线程资源是非常昂贵的，用户的线程数是相对可控的，所以这种场景下，SOFABolt使用简单的java.util.concurrent.CountDownLatch来实现超时任务的触发。

![](https://mmbiz.qpic.cn/mmbiz_png/nibOZpaQKw0icbdNVkDpX1ya2yrc5tCsMXBztiavLeRiaQTz6KRib6AH7Wjrsl2NbD32ko71MRFxTiac7M36Zl9bIHzg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png | left | 738x722")

SOFABolt同步调用的代码如上，核心逻辑是：

1.  创建 InvokeFuture

2.  在 Netty 的 ChannelFuture 中添加 Listener，在写入操作失败的情况下通过 future.putResponse方法修改Future状态（正常服务端响应也是通过 future.putResponse来改变InvokeFuture的状态的，这个流程不展开说明）

3.  写入出现异常的情况下也是通过future.putResponse方法修改Future状态

4.  通过future.waitResponse来执行等待响应
    其中和超时相关的是future.waitResponse的调用，InvokeFuture内部通过java.util.concurrent.CountDownLatch来实现超时触发。

![](https://mmbiz.qpic.cn/mmbiz_png/nibOZpaQKw0icbdNVkDpX1ya2yrc5tCsMXdXmLdMzFDsKpetG8XjvtmzoTkYC5FKZj0edtUljdnSpXw3Cy6dodicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png | left | 666x178")

java.util.concurrent.CountDownLatch#await(timeout, timeoutUnit) 方法实现了等待一段时间的逻辑，并且通过countDown方法来提前中断等待，SOFABolt 中 InvokeFuture 通过构建 new CountDownLatch(1)的实例，并将 await 和 countDown 方法包装为 awaitResponse 和 putResponse 来实现同步调用的超时控制。

### 异步调用的超时控制实现

相对于同步调用，异步调用并不会阻塞调用线程，那么超时任务的数量并不受限于线程对的数量，用户可能通过一个线程来触发量大的请求，从而产生大量的定时任务。那么我们需要一个机制来管理大量的定时任务，并且作为系统底层的通信框架，需要保证这个机制尽量少的占用资源。上文已经提到 TimeWheel 是一个非常适合于这种场景的数据结构。
Netty 中实现了 TimeWheel 数据结构：io.netty.util.HashedWheelTimer，SOFABolt 异步调用的超时控制直接依赖于 Netty 的 io.netty.util.HashedWheelTimer 实现。
Future 模式和 Callback 模式在超时控制机制上一致的，下面以 Callback为 例分析异步调用的超时控制机制。

![](https://mmbiz.qpic.cn/mmbiz_png/nibOZpaQKw0icbdNVkDpX1ya2yrc5tCsMXUNUQXValpwSPAoqOsxGcOcBQtIJpjWfEK0iajQPUBN75aTlewWPJcQw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png | left | 747x901")

SOFABolt 异步调用的代码如上，核心逻辑是：

1.  创建 InvokeFuture

2.  创建 Timeout 实例，Timeout 实例的 run 方法中通过 future.putResponse 来修改 InvokeFuture 的状态

3.  在 Netty 的 ChannelFuture 中添加 Listener，在写入操作失败的情况下通过 future.cancelTimeout 来取消超时任务，通过 future.putResponse 来修改 InvokeFuture的状态

4.  在写入异常的情况下同样通过 future.cancelTimeout 来取消超时任务，通过 future.putResponse 来修改 InvokeFuture 的状态
    在异步调用的实现中，通过 Timeout 来触发超时任务，相当于同步调用中的java.util.concurrent.CountDownLatch#await(timeout, timeoutUnit)。Future#cancelTimeout()方法则是调用了 Timeout 的 cancel 来取消超时任务，相当于同步调用中通过java.util.concurrent.CountDownLatch#countDown()来提前结束超时任务。具体超时任务的管理则全部委托给了 Netty 的 Timer 实现。
    另外值得注意的一点是 SOFABolt 在使用 Netty 的 Timer 时采用了单例的模式，因为一般情况下使用一个 Timer 管理所有的超时任务即可，这样可以节省系统的开销。

### Fail-Fast机制

以上关于 SOFABolt 的超时机制介绍都是关于 SOFABolt 客户端如何完成高效的超时任务管理的，其实在 SOFABolt 的服务端同样针对超时的场景做了优化。
客户端为了应对没有响应的情况，增加了超时机制，那么就可能存在服务端返回一个响应但是客户端在收到这个响应之前已经认为请求超时了，移除了相关的请求上下文，那么这个响应对客户端来说就没有意义了。既然这个响应对客户端来说是没有意义的，那么服务端其实可以进一步优化：在确认请求已经超时的情况下，服务端可以直接丢弃请求来减轻服务端的处理负担，SOFABolt 把这个机制称为 Fail-Fast。

![](https://mmbiz.qpic.cn/mmbiz_png/nibOZpaQKw0icbdNVkDpX1ya2yrc5tCsMXCEbtlRokBBye4Bp2QoPl63LdbaYgUc3k2LdN14NV0dVbLTicU5Whc5A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png | left | 747x282")

如上图所示，请求可能在服务端积压了一段时间，此时这些请求在客户端看来已经超时了，如果服务端继续处理这些超时的请求，第一请求的响应最终会被客户端丢弃；第二可能加剧服务端的压力导致后续更多请求超时。通过 Fail-Fast 机制直接丢弃掉这批请求能减轻服务端的负担使服务端尽快恢复并提供正常的服务能力。
Fail-Fast 机制是一个明显的优化手段，唯一面临的问题是如何确定一个请求已经超时。注意，一定不要依赖跨系统的时钟，因为时钟可能不一致，从而导致未超时的请求被误认为超时而被服务端丢弃。
SOFABolt 采用了请求被处理时的时间和请求到达服务端的时间来判定请求是否已经超时，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/nibOZpaQKw0icbdNVkDpX1ya2yrc5tCsMXo6uxWnW2hibLlhOsNCVicYkNXhP9Xib3jP8KVuicSMUTxr4gaE0NL941vQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png | left | 747x242")

这样会有一小部分客户端认为已经超时的请求服务端还会处理（因为网络传输是需要时间的），但是不会出现误判的情况。

###   SOFABolt 的心跳机制

除了上文提供的超时机制外，在通信框架中往往还有另一类超时，那就是连接的超时。
我们知道，一次 tcp 请求大致分为三个步骤：建立连接、通信、关闭连接。每次建立新连接都会经历三次握手，中间包含三次网络传输，对于高并发的系统，这是一笔不小的负担。所以在通信框架中我们都会维护一定数量的连接，其中一个手段就是通过心跳来维持连接，避免连接因为空闲而被回收。
Netty 提供了 IdleStateHandler，如果连接空闲时间过长，则会触发 IdleStateEvent。SOFABolt 基于 IdleStateHandler 的 IdleStateEvent 来触发心跳，一来这样可以通过心跳维护连接，二来基于 IdleStateEvent 可以减少不必要的心跳。
SOFABolt 心跳相关的处理有两部分：客户端发送心跳，服务端接收心跳处理并返回响应。

![](https://mmbiz.qpic.cn/mmbiz_png/nibOZpaQKw0icbdNVkDpX1ya2yrc5tCsMX2J8or3ICwmYKQ2zB6x1df8cngAhp3sDhZl6PibibRNcayfBcJYGmu0yw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png | left | 747x1612")

上面是客户端触发心跳后的代码，当客户端接收到 IdleStateEvent 时会调用上面的heartbeatTriggered 方法。
在 Connection 对象上会维护心跳失败的次数，当心跳失败的次数超过系统的最大次时，主动关闭 Connection。如果心跳成功则清除心跳失败的计数。同样的，在心跳的超时处理中同样使用 Netty 的 Timer 实现来管理超时任务（和请求的超时管理使用的是同一个 Timer 实例）。

![](https://mmbiz.qpic.cn/mmbiz_png/nibOZpaQKw0icbdNVkDpX1ya2yrc5tCsMXzmmfwArd3vDAsWZR0DibRSJibTwgVLZv1D8Vkj7buibn16SghEk83w0gA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png | left | 747x974")

RpcHeartbeatProcessor 是 SOFABolt 对心跳处理的实现，包含对心跳请求的处理和心跳响应的处理（服务端和客户端复用这个类，通过请求的数据类型来判断是心跳请求还是心跳响应）。
如果接收到的是一个心跳请求，则直接写回一个 HeartbeatAckCommand（心跳响应）。如果接收到的是来自服务端的心跳响应，则从 Connection 取出 InvokeFuture对象并做对应的状态变更和其他逻辑的处理：取消超时任务、执行Callback。如果无法从 Connection 获取 InvokeFuture 对象，则说明客户端已经判定心跳请求超时。
另外值得注意的一点是，SOFABolt 中心跳请求和心跳响应对象都只包含 RequestCommand 和 ResponseCommand 的必要字段，没有额外增加任何属性，这也是为了减少不必要的网络带宽的开销。

###   总结

本文简单的介绍了 TimeWheel 的原理，SOFABolt 的超时控制机制和心跳机制的实现。SOFABolt 基于高效的 TimeWheel 实现了自己的超时控制机制，同时增加 Fail-Fast 策略优化服务端对超时请求的处理。另外 SOFABolt 默认实现了连接的心跳机制，以保持系统空闲时连接的可用性，这些都为 SOFABolt 的高性能打下了坚实的基础。

###   **《蚂蚁金服通信框架SOFABolt解析》系列历史文章**

*   [蚂蚁金服通信框架SOFABolt解析 | 编解码机制](http://mp.weixin.qq.com/s?__biz=MzUzMzU5Mjc1Nw==&mid=2247484406&idx=1&sn=da3e3364efc313d0014958f6f71aca17&chksm=faa0ec2ccdd7653a2ec0758c9339c0cee8c0380c7fe29f0c5d000e70bbb978cb39f3342a936f&scene=21#wechat_redirect)

*   [蚂蚁金服通信框架SOFABolt解析 |序列化机制(Serializer)](http://mp.weixin.qq.com/s?__biz=MzUzMzU5Mjc1Nw==&mid=2247484425&idx=1&sn=a7162c88139e8faf25e7c321613a58be&chksm=faa0ebd3cdd762c59a02bff3f392a213452fde17b9b4317e37cab24470740bbc1adacfc12091&scene=21#wechat_redirect)

*   [蚂蚁金服通信框架SOFABolt解析 | 协议框架解析](http://mp.weixin.qq.com/s?__biz=MzUzMzU5Mjc1Nw==&mid=2247484442&idx=1&sn=10141f9a20199e608ce5fd7f11ee4e29&chksm=faa0ebc0cdd762d60cedbfb079444e3e6d35383f063e060947bbcb622f86db39fded94f306f4&scene=21#wechat_redirect)

*   [蚂蚁金服通信框架SOFABolt解析 | 连接管理剖析](http://mp.weixin.qq.com/s?__biz=MzUzMzU5Mjc1Nw==&mid=2247484457&idx=1&sn=151334a84ace245a04189735743c154a&chksm=faa0ebf3cdd762e566e1736f4dd958c23f1d48bfcd33ab7064715441426d9bc9c492eadb1b61&scene=21#wechat_redirect)
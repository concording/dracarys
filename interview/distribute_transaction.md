

在[支付宝架构与技术](https://link.zhihu.com/?target=http%3A//wenku.baidu.com/view/d1bbd25877232f60ddcca1d9.html) 中对柔性事务有大致的描述：

![](https://pic1.zhimg.com/26b0179625de3dddfc4ca993070b2134_b.jpg)

![](https://pic1.zhimg.com/80/26b0179625de3dddfc4ca993070b2134_hd.jpg)

![](https://pic4.zhimg.com/6834e16809e7660a6437eb91afab71a3_b.jpg)

![](https://pic4.zhimg.com/80/6834e16809e7660a6437eb91afab71a3_hd.jpg)

![](https://pic2.zhimg.com/b50af08415528f55eeda6f9c1ecf1ed5_b.jpg)

![](https://pic2.zhimg.com/80/b50af08415528f55eeda6f9c1ecf1ed5_hd.jpg)

可以看出，柔性事务（遵循BASE理论）是指相对于ACID刚性事务而言的。
支付宝所说的柔性事务分为：两阶段型、补偿型、异步确保型、最大努力通知型几种。
由于支付宝整个架构是SOA架构，因此传统单机环境下数据库的ACID事务满足了分布式环境下的业务需要，以上几种事务类似就是针对分布式环境下业务需要设定的。
其中：
**1、两阶段型：就是分布式事务两阶段提交，对应技术上的XA、JTA/JTS。**
这是分布式环境下事务处理的典型模式。

**2、补偿型：** TCC型事务（Try/Confirm/Cancel）可以归为补偿型。
补偿型的例子，在一个长事务（ long-running ）中 ，一个由两台服务器一起参与的事务，服务器A发起事务，服务器B参与事务，B的事务需要人工参与，所以处理时间可能很长。如果按照ACID的原则，要保持事务的隔离性、一致性，服务器A中发起的事务中使用到的事务资源将会被锁定，不允许其他应用访问到事务过程中的中间结果，直到整个事务被提交或者回滚。这就造成事务A中的资源被长时间锁定，系统的可用性将不可接受。
WS-BusinessActivity提供了一种基于补偿的long-running的事务处理模型。还是上面的例子，服务器A的事务如果执行顺利，那么事务A就先行提交，如果事务B也执行顺利，则事务B也提交，整个事务就算完成。但是如果事务B执行失败，事务B本身回滚，这时事务A已经被提交，所以需要执行一个补偿操作，将已经提交的事务A执行的操作作反操作，恢复到未执行前事务A的状态。这样的SAGA事务模型，是牺牲了一定的隔离性和一致性的，但是提高了long-running事务的可用性。
例子来源：OASIS的WS-BusinessActivity文档

**3、异步确保型**
将一些同步阻塞的事务操作变为异步的操作，避免对数据库事务的争用，典型例子是热点账户异步记账、批量记账的处理。

**4、最大努力型**
PPT中提到的例子交易的消息通知（例如商户交易结果通知重试、补单重试）

如果有技术背景，可以参考另外一个文档 [大规模SOA系统中的分布事务处事](https://link.zhihu.com/?target=http%3A//wenku.baidu.com/view/be946bec0975f46527d3e104.html) ，对支付宝分布式事务处理机制有较为详细描述。
更详细的也可以参考OASIS的相关资料。
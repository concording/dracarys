[高并发下怎么做余额扣减](https://www.zhihu.com/question/61484424)

作为一个参与了多次某宝双十一账务，交易系统的改进，升级，并与分布式一致性服务的主要设计者亲切沟通过细节的码农，来深入浅出的说说一些业界的通用思路和想法。首先，就业务量而言，我把余额业务的高并发难点分成以下几个层次：

1. 单应用，单数据库，这是最基本的层次，在这个层次，只需要选择合适的单机数据库选型，做好业务场景就OK了，推荐的TPS 在5w以下，最好的优化 + 32核心64GB + io打满也不会大于30w
2. 单应用，多数据库，使用分库来解决数据库横向扩容的问题来增加TPS，此时，业务开发会成为最大瓶颈，因为太多业务往同一个代码库中堆积代码，最终造成整体代码质量下滑，bug频现，拖垮整个体系，这个体系一般不会持续太长时间，甚至有的公司不会有这个过渡，而是会直接进入层次3，这个层次的工具有一些中间件和一些proxy，举个例子，比如 [dangdangdotcom/sharding-jdbc](https://github.com/dangdangdotcom/sharding-jdbc) 也有基于proxy的：[MyCATApache/Mycat-Server](https://github.com/MyCATApache/Mycat-Server)， [flike/kingshard](https://github.com/flike/kingshard) ， 当然也有一些类spanner类的产品比如：[pingcap/tidb](https://github.com/pingcap/tidb)  或者OB，也可以解决扩容问题（实际上是在KV这里scale out了，而不是业务层来做数据的scale out），而且更进一步，让你看起来像一个数据库，当然这类产品其实用起来真的做起分布式事务来latency是很高的，还是需要业务指定分库键来把事务退化为单partition事务，否则线上根本扛不住。
3. 多应用，多数据库，进入了多应用，多数据库的层次后，突然会发现多了一个新的问题，分布式一致性事务，虽然在层次2中，有的业务已经发现了单应用跨库事务的问题，但一般来说，好的业务都能通过一些关系字段来很好规避分布式事务的问题，而且一个设计中也应该杜绝大量的跨库事务，所以层次2中问题并不明显（个别的完全可以走XA事务或google vitess之类的方案：[youtube/vitess](https://github.com/youtube/vitess/blob/master/doc/TwoPhaseCommitDesign.md) ，总之就是commit的二阶段递交，再加点乐观锁）

好了，接下来我们要进入到下一节，怎么解决好多个应用间的分布式一致性事务问题

解决多应用之间的分布式一致性问题，目前基本分为两个思路：1 补偿型事务 2 确保通知型事务

1. 作为比较，先说下一般的acid二阶段递交的流程，dml阶段先锁行，一阶段写redo，二阶段再做真递交+解锁行，这种模型非常慢，latency很大
2. 补偿性事务就是一阶段的时候都直接递交，这样一阶段都成功的话，二阶段可以快速返回，如一阶段有参与者失败，则再使用一条补偿操作，把一阶段的递交给补偿回来。
3. 确保通知型事务则是，发起者先落库，然后再发出一条事务型消息，确保消息发出后，直接返回（此时参与者很多都未落库）

这两个方式的出发点是一致的，普通两阶段，锁行的时间长，返回等待的时间长，无法很好的利用好机器性能，所以牺牲一点一致性，让业务等待时间缩短。

然后再回来说余额类业务，余额型业务的业务特点决定了他非常适合补偿型的事务，由于余额是一个数字的加减，这意味着，加减的顺序不影响最终值，所以它天然适合这类牺牲一定一致性的事务。

余额类的使用方式：

1. 每个余额类应用都可以在一阶段更新余额 update amount = amount - 消费金额 ，而补偿操作很简单，就是 update amount = amount + 消费金额 
2. 如果不想让使用者看到这个不一致的过程，可以再设置一个冻结金额表，第一阶段操作冻结金额的表，二阶段把冻结金额更新到余额上
3. 如果不想让余额被两个同时来的消费请求扣减为负数（也就是第二个请求来时，第一个请求扣减未生效），可以让一阶段检查 余额 - sum(冻结金额），然后再产生一笔新的冻结金额。
具体的参考资料，可以看看支付宝08年公开的一篇ppt，绝对微言大义，内容非常精彩：[Blog of Kami Wan](http:A//kaimingwan.com/post/fen-bu-shi/fen-bu-shi-shi-wu-de-dian-xing-chu-li-fang-shi-2pc-tcc-yi-bu-que-bao-he-zui-da-nu-li-xing) 
目前我在github上也看到有一些开源工程可以参考比如：[changmingxie/tcc-transaction](https://github.com/changmingxie/tcc-transaction)

当然，大并发下的分布式一致性服务，目前的大难点已经是异地跨机房的分钟级别容灾了，这也是目前蚂蚁的分布式一致性服务笑傲群雄的本钱，关于这个的细节和处理，这里就不做赘述（解决方案非常精彩实用，可谓工程典范）
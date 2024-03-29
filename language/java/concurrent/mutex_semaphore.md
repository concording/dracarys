作者：二律背反
链接：https://www.zhihu.com/question/47704079/answer/135859188
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

mutex，一句话：保护共享资源。
典型的例子就是买票：
票是共享资源，现在有两个线程同时过来买票。如果你不用mutex在线程里把票锁住，那么就可能出现“把同一张票卖给两个不同的人（线程）”的情况。
我想这个不需要多解释了。

一般人不明白semaphore和mutex的区别，**根本原因是不知道semaphore的用途。**
semaphore的用途，一句话：**调度线程**。
有的人用semaphore也可以把上面例子中的票“保护"起来以防止共享资源冲突，必须承认这是可行的，但是semaphore不是让你用来做这个的；**如果你要做这件事，请用mutex。**

在网上、包括stackoverflow等著名论坛上，有一个流传很广的厕所例子：
mutex是一个厕所一把钥匙，谁抢上钥匙谁用厕所，谁没抢上谁就等着；semaphore是多个同样厕所多把同样的钥匙 ---- 只要你能拿到一把钥匙，你就可以随便找一个空着的厕所进去。
事实上，这个例子对初学者、特别是刚刚学过mutex的初学者来说非常糟糕 ----- 我第一次读到这个例子的第一反应是：semaphore是线程池？？？所以，**请务必忘记这个例子**。
另外，有人也会说：mutex就是semaphore的value等于1的情况。
这句话不能说不对，但是对于初学者来说，请先把这句话视为错误；等你将来彻底融会贯通这部分知识了，你才能真正理解上面这句话到底是什么意思。总之请务必记住：**mutex干的活儿和semaphore干的活儿不要混起来。**

在这里，我模拟一个最典型的使用semaphore的场景：
a源自一个线程，b源自另一个线程，计算c = a + b也是一个线程。（即一共三个线程）

显然，第三个线程必须等第一、二个线程执行完毕它才能执行。
在这个时候，我们就需要**调度线程**了：让第一、二个线程执行完毕后，再执行第三个线程。
此时，就需要用semaphore了。

```text
int a, b, c;
void geta()
{
    a = calculatea();
    semaphore_increase();
}

void getb()
{
    b = calculateb();
    semaphore_increase();
}

void getc()
{
    semaphore_decrease();
    semaphore_decrease();
    c = a + b;
}

t1 = thread_create(geta);
t2 = thread_create(getb);
t3 = thread_create(getc);
thread_join(t3);

// semaphore的机制我在这里就不讲了，百度一下你就知道。
// semaphore_increase对应sem_post
// semaphore_decrease对应sem_wait

```

这就是semaphore最典型的用法。
说白了，**调度线程，就是：一些线程生产（increase）同时另一些线程消费（decrease），semaphore可以让生产和消费保持合乎逻辑的执行顺序。**
而线程池是程序员根据具体的硬件水平和不同的设计需求、为了达到最佳的运行效果而避免反复新建和释放线程同时对同一时刻启动的线程数量的限制，这完全是两码事。
比如如果你要计算z = a + b +...+ x + y ...的结果，同时每个加数都是一个线程，那么计算z的线程和每个加数的线程之间的逻辑顺序是通过semaphore来调度的；而至于你运行该程序的时候到底要允许最多同时启动几个线程，则是用线程池来实现的。

semaphore和条件锁的区别：
条件锁，本质上还是锁，它的用途，还是围绕“共享资源”的。条件锁最典型的用途就是：防止不停地循环去判断一个共享资源是否满足某个条件。
比如还是买票的例子：
我们除了买票的线程外，现在再加一个线程：如果票数等于零，那么就要挂出“票已售完”的牌子。这种情况下如果没有条件锁，我们就不得不在“挂牌子”这个线程里不断地lock和unlock而在大多数情况下票数总是不等于零，这样的结果就是：占用了很多CPU资源但是大多数时候什么都没做。
另外，假如我们还有一个线程，是在票数等于零时向上级部门申请新的票。同理，问题和上面的一样。而如果有了条件锁，我们就可以避免这种问题，而且还可以一次性地通知所有被条件锁锁住的线程。
这里有个问题，是关于条件锁的：[pthread_cond_wait 为什么需要传递 mutex 参数？](https://www.zhihu.com/question/24116967)
不清楚条件锁的朋友可以看一下。
总之请记住：**条件锁，是为了避免绝大多数情况下都是lock ---> 判断条件 ----> unlock的这种很占资源但又不干什么事情的线程。**它和semaphore的用途是不同的。

简而言之，**锁是服务于共享资源的；而semaphore是服务于多个线程间的执行的逻辑顺序的。** ================ 确定上面的内容都彻底理解后 ===================
感谢

[@灵剑](//www.zhihu.com/people/60cd9664ef2f13d8d5ddba060ef35a8a)

的提醒，在此补充后续内容。

请回头看那个让大家忘记的厕所例子。我之所以让大家忘记这个例子，是因为如果你从这个角度去学习semaphore的话，一定会和mutex混为一谈。semaphore的本质就是**调度线程** ---- 在充分理解了这个概念后，我们再看这个例子。

semaphore是通过一个值来实现线程的调度的，因此借助这种机制，我们**也可以实现对线程数量的限制。**例子我就不写了，如果你看懂了上面的c = a + b的例子，相信你可以轻松写出来用semaphore限制线程数量的例子。而当我们把线程数量限制为1时，你会发现：共享资源受到了保护 ------ 任意时刻只有一个线程在运行，因此共享资源当然等效于受到了保护。但是我要再提醒一下，**如果你要对共享资源进行保护，请用mutex；到底应该用条件锁还是用semaphore，请务必想清楚。**通过semaphore来实现对共享资源的保护的确可行但是是对semaphore的一种**错用**。

**只要你能搞清楚锁、条件锁和semaphore为什么而生、或者说它们是面对什么样的设计需求、为了解决什么样类型的问题才出现的，你自然就不会把他们混淆起来。**

===========================================================
厕所例子：

```text
semaphore sem(2);  // 同时执行的线程数量上限为2
void toiletA()
{
    semaphore_decrease(sem);
    // do something
    semaphore_increase(sem);
}

void toiletB()
{
    semaphore_decrease(sem);
    // do something
    semaphore_increase(sem);
}

void toiletC()
{
    semaphore_decrease(sem);
    // do something
    semaphore_increase(sem);
}

t1 = thread_create(toiletA);
t2 = thread_create(toiletB);
t3 = thread_create(toiletC);
thread_join(toiletA);
thread_join(toiletB);
thread_join(toiletC);

```

应该不需要多解释了。不过要强调一下，虽然semaphore的这种用法和线程池看上去很类似：都是在限制同时执行的线程数量，但是两者是有本质区别的。
**线程池是通过用固定数量的线程去执行任务队列里的任务来达到避免反复创建和销毁线程而造成的资源浪费；而semaphore并没有直接提供这种机制** ---- 上面的例子中虽然同时最多有两个线程在运行，但是最一开始三个线程就已经都创建好了；而线程池则是：一开始就创建两个线程，然后将这三个任务加入到线程池的任务队列中，让线程池利用这两个线程去完成这三个任务。

======================= BONUS ======================

如果你可以使用面向对象语言（C++1x、Java等）而不是必须使用C语言来实现上述设计需求的话，那么promise & future和线程池是个更好的选择。


The article [Mutexes and Semaphores Demystified](http://www.netrino.com/Embedded-Systems/How-To/RTOS-Mutex-Semaphore) by Michael Barr is a great short introduction into what makes mutexes and semaphores different, and when they should and should not be used. I've excerpted several key paragraphs here.

The key point is that mutexes should be used to protect shared resources, while semaphores should be used for signaling. You should generally not use semaphores to protect shared resources, nor mutexes for signaling. There are issues, for instance, with the bouncer analogy in terms of using semaphores to protect shared resources - you can use them that way, but it may cause hard to diagnose bugs.

> While mutexes and semaphores have some similarities in their implementation, they should always be used differently.
> 
> The most common (but nonetheless incorrect) answer to the question posed at the top is that mutexes and semaphores are very similar, with the only significant difference being that semaphores can count higher than one. Nearly all engineers seem to properly understand that a mutex is a binary flag used to protect a shared resource by ensuring mutual exclusion inside critical sections of code. But when asked to expand on how to use a "counting semaphore," most engineers—varying only in their degree of confidence—express some flavor of the textbook opinion that these are used to protect several equivalent resources.

...

At this point an interesting analogy is made using the idea of bathroom keys as protecting shared resources - the bathroom. If a shop has a single bathroom, then a single key will be sufficient to protect that resource and prevent multiple people from using it simultaneously.

If there are multiple bathrooms, one might be tempted to key them alike and make multiple keys - this is similar to a semaphore being mis-used. Once you have a key you don't actually know which bathroom is available, and if you go down this path you're probably going to end up using mutexes to provide that information and make sure you don't take a bathroom that's already occupied.

A semaphore is the wrong tool to protect several of the essentially same resource, but this is how many people think of it and use it. The bouncer analogy is distinctly different - there aren't several of the same type of resource, instead there is one resource which can accept multiple simultaneous users. I suppose a semaphore can be used in such situations, but rarely are there real-world situations where the analogy actually holds - it's more often that there are several of the same type, but still individual resources, like the bathrooms, which cannot be used this way.

...

> The correct use of a semaphore is for signaling from one task to another. A mutex is meant to be taken and released, always in that order, by each task that uses the shared resource it protects. By contrast, tasks that use semaphores either signal or wait—not both. For example, Task 1 may contain code to post (i.e., signal or increment) a particular semaphore when the "power" button is pressed and Task 2, which wakes the display, pends on that same semaphore. In this scenario, one task is the producer of the event signal; the other the consumer.

...

Here an important point is made that mutexes interfere with real time operating systems in a bad way, causing priority inversion where a less important task may be executed before a more important task because of resource sharing. In short, this happens when a lower priority task uses a mutex to grab a resource, A, then tries to grab B, but is paused because B is unavailable. While it's waiting, a higher priority task comes along and needs A, but it's already tied up, and by a process that isn't even running because it's waiting for B. There are many ways to resolve this, but it most often is fixed by altering the mutex and task manager. The mutex is much more complex in these cases than a binary semaphore, and using a semaphore in such an instance will cause priority inversions because the task manager is unaware of the priority inversion and cannot act to correct it.

...

> The cause of the widespread modern confusion between mutexes and semaphores is historical, as it dates all the way back to the 1974 invention of the Semaphore (capital "S", in this article) by Djikstra. Prior to that date, none of the interrupt-safe task synchronization and signaling mechanisms known to computer scientists was efficiently scalable for use by more than two tasks. Dijkstra's revolutionary, safe-and-scalable Semaphore was applied in both critical section protection and signaling. And thus the confusion began.
> 
> However, it later became obvious to operating system developers, after the appearance of the priority-based preemptive RTOS (e.g., VRTX, ca. 1980), publication of academic papers establishing RMA and the problems caused by priority inversion, and a paper on priority inheritance protocols in 1990, 3 it became apparent that mutexes must be more than just semaphores with a binary counter.

Mutex: resource sharing

Semaphore: signaling

Don't use one for the other without careful consideration of the side effects.

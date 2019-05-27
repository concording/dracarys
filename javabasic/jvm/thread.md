Java的目标是要跨平台，而不同的操作系统(如类Unix和Windows)其任务调度机制有很大的不同，故Java在JVM层面抽象了一套自己的线程机制，用以映射不同的操作系统的任务调度。如你所述的一些缺点，Java在1.1版本之后，这个线程模块就是基于内核级别线程[[1]](http://www.cnblogs.com/cando/archive/2012/08/10/2631780.html)。

首先，如果一个机器上只有一个逻辑CPU，此时系统调度器通过轮换时间片的方式来调度任务，所以不存在真正的并行，只是并发；只有多个逻辑CPU的情形下，才会出现并行；

其次，对于不同的操作系统，其任务调度机制不同：

1.  对于类Unix系统而言，一般都是进程作为任务的调度单位，也即是操作系统调度器，只会针对进程来分配CPU等资源。由于进程彼此独立，相互不可进行直接访问，这增加了应用的通信成本。所以后面有了微进程，微进程与进程不同的是，允许一定程度上，彼此可以直接进行访问，详细可参考[LinuxThreads](http://en.wikipedia.org/wiki/LinuxThreads)。JVM在一些类Unix平台下，就是将线程映射到操作系统的微进程，来实现线程调度。这样多线程能够直接被系统调度器进行调度，与此对应的就是其线程的创建和销毁的成本就比较高，而且JVM的线程优先级很难进行匹配，无法提供确切的保证，仅仅是个hint。需要说明的是在linux2.6之后，[LinuxThreads](http://en.wikipedia.org/wiki/LinuxThreads)被[NPTL](http://en.wikipedia.org/wiki/Native_POSIX_Thread_Library)所取代，NPTL提供了1:1形式的线程模型，即每个线程一一对应一个内核调度实体，这样即是内核级别的线程。同样NPTL额外提供可替换的M:N模型[Thread](http://en.wikipedia.org/wiki/Thread_%28computer_science%29#M:N_.28Hybrid_threading.29)。
    linux下jvm创建Thread的调用（openJDK-6-src-b27中的os_linux.cpp中的bool os::create_thread(Thread* thread, ThreadType thr_type, size_t stack_size)中调用以下API）：

    `int ret = pthread_create(&tid, &attr, (void* (*)(void*)) java_start, thread);`

    具体的后续调用跟踪，请参考pthread的实现；
2.  对于Windows系统而言，从其一开始线程就是系统进行调度的最小单位，即Wiki[[Thread]](http://en.wikipedia.org/wiki/Thread_%28computer_science%29#M:N_.28Hybrid_threading.29)中所述的1:1模型。而对于用户态线程和内核态线程，一般是针对N:1模型和M:N模型，即多个用户态线程映射到一个内核线程，系统调度器的调度单位是内核态线程，对用户态线程一无所知。个人对windows的线程模型不是十分了解，按照[NPTL](http://en.wikipedia.org/wiki/Native_POSIX_Thread_Library)和Wiki[[Thread]](http://en.wikipedia.org/wiki/Thread_%28computer_science%29#M:N_.28Hybrid_threading.29)和中的描述，觉得windows直接将线程作为调度单位，不存在用户态线程和内核态线程的区别。需要说明的是，JVM定义的线程优先级和Windows的线程优先级能够很好地匹配。
    windows下jvm创建Thread的调用-（openJDK-6-src-b27中的os_windows.cpp中的bool os::create_thread(Thread* thread, ThreadType thr_type, size_t stack_size)中调用以下API）：

    `HANDLE thread_handle = (HANDLE)_beginthreadex(NULL, (unsigned)stack_size, (unsigned (__stdcall *)(void*)) java_start, thread, CREATE_SUSPENDED | STACK_SIZE_PARAM_IS_A_RESERVATION, &thread_id);`

    具体后续调用请参考windows的API;

最后，回答你的问题：

1.  Java的多线程不是骗人，针对不同的操作系统进行适配，确实实现了并发和并行；
2.  用户级别线程通常发生在进程级别，一般通过系统调用等不同的实现方式来模拟实现并发，而内核级别线程则一般由系统的调度器来调度，是原生级别的并发；
3.  内核级别线程有系统调度器调度，肯定可以享受多核的好处，而用户级别线程是通过模拟实现调度，由于其映射的调度实体(进程或者映射的内核级线程)才是系统调度器的调度单位，所以与内核级线程相比，其对资源的访问确实存在一定的限制。详细的内容或者想继续深入了解，请再多参考参考wiki等文献。
Thread.interrupt() sets the interrupted status/flag of the target thread. Then code running in that target thread MAY poll the interrupted status and handle it appropriately. Some methods that block such as Object.wait() may consume the interrupted status immediately and throw an appropriate exception (usually InterruptedException)

Interruption in Java is not pre-emptive. Put another way both threads have to cooperate in order to process the interrupt properly. If the target thread does not poll the interrupted status the interrupt is effectively ignored.

Polling occurs via the Thread.interrupted() method which returns the current thread's interrupted status AND clears that interrupt flag. Usually the thread might then do something such as throw InterruptedException.

`EDIT (from Thilo comments): Some API methods have built in interrupt handling. Of the top of my head this includes.`

* Object.wait()/Thread.sleep()
* Most java.util.concurrent structures
* Java NIO (but not java.io) and it does NOT use InterruptedException, instead using ClosedByInterruptException.

`EDIT (from @thomas-pornin's answer to exactly same question for completeness)`
Thread interruption is a gentle way to nudge a thread. It is used to give threads a chance to exitcleanly, as opposed to Thread.stop() that is more like shooting the thread with an assault rifle.

***个人总结***

告诉当前线程你该放手了，该结束了。调用Thread.interrupt()之后会立即设置目标线程的interrupted状态，之后代码进入到这个目标线程就能拿到这个状态并进行相应的处理。调用Object.wait()方法会立即消费掉这个状态，同时抛出一个InterruptedException异常(消费掉这个状态之后，状态会被清除)。

[interrupt到底做了啥](http://stackoverflow.com/questions/3590000/what-does-java-lang-thread-interrupt-do)

[lmu java Thread ](http://cs.lmu.edu/~ray/notes/javathreading/)


**Understanding java.lang.Thread.State: WAITING (parking)**


Permit means a permission to continue execution. Parking means suspending execution until permit is available.
Unlike Semaphore's permits, permits of LockSupport are associated with threads (i.e. permit is given to a particular thread) and doesn't accumulate (i.e. there can be only one permit per thread, when thread consumes the permit, it disappears).
You can give permit to a thread by calling unpark(). A thread can suspend its execution until permit is available (or thread is interrupted, or timeout expired, etc) by calling park(). When permit is available, the parked thread consumes it and exits a park() method.

***个人理解***

parking意味着直到可以获得permit之前线程都将处于挂起的状态。LockSupport关联着线程，一个线程只能有一个permit，并且一旦线程消费了这个permit，permit就会消失。调用unpark()方法将会给线程一个permit。一个线程将会永远挂起直到通过调用park方法获得了permit。一旦当permit可用，parked的线程就会消费permit并且退出park()方法。


**Difference between WAIT and BLOCKED thread states**


A thread goes to wait state once it calls wait() on an Object. This is called Waiting State. Once a thread reaches waiting state, it will need to wait till some other thread notify() or notifyAll() on the object.
Once this thread is notified, it will not be runnable. It might be that other threads are also notified(using notifyAll) or the first thread has not finished his work, so it is still blocked till it gets its chance. This is called Blocked State.
Once other threads have left and its this thread chance, it moves to Runnable state after that it is eligible pick up work based on JVM threading mechanism and moves to run state.

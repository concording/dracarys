# Java SynchronousQueue Example
 [JAVA CONCURRENCY](http://www.javarticles.com/category/java/java-concurrency)

A synchronous queue is a blocking queue where two different operations act on the queue, an insert and a remove. The insert operation must wait for a corresponding remove operation by another thread, and vice versa.

Internally SynchronousQueue relies on a dual stack/queue where each node on it represents an operation. If the insert operation is the first one to act on the queue then it is represented by a data node.
If a remove operation is the first one then it is represented by a request node.
When a remove operation is performed on the queue and there is already a waiting data node then the remove operation will consume the data node and annihilate each other from the queue. In this case the remove operation node is called a ‘fuliller’ node as it fulfills the thread in waiting.

The thread remains blocked till there is another thread that comes in to fulfill the need.

[![SynchronousQueue Concept](http://www.javarticles.com/wp-content/uploads/2016/06/SyncQ_ProducerConsumer.png)](http://www.javarticles.com/wp-content/uploads/2016/06/SyncQ_ProducerConsumer.png)

SynchronousQueue Concept

Let’s now review each case and the data structure involved with some examples.

## Dual Stack/Queue

Synchronous queue has two operations, **insert** and **remove**.
If a thread trying to insert an element to a synchronous queue finds no consumer then it remain blocked till a consumer comes in and dequeues it.
Likewise a thread requesting data from a synchronous queue remains blocked till there is a producer thread trying to insert.

Thus a synchronous queue acts as a dual stack/queue which at any given point of time holds nodes that represents either operations inserting data or operations requesting data. By default, the waiting threads contend in LIFO order, one can create a SynchronousQueue with fairness policy in which case the order would be FIFO.

## SynchronousQueue Mode

There is also a third type of mode that represents a fullfiller. Its job is to fulfill the thread in waiting. Suppose the queue holding the data node is blocked as there has been no taker yet then any future call requesting for an item becomes the ‘fulfiller’. The scenario can be vice-versa too thus a fulfiller node is the one that dequeues the complimentary node (data node or request node).

1.  Insert an item `SynchronousQueue.put(item)`



2.  Remove an item `SynchronousQueue.take(item)`

Below are the three modes:

1.  Request
2.  Data
3.  Fulfiller

[![SynchronousQueue Modes](http://www.javarticles.com/wp-content/uploads/2016/06/SNode-Modes.png)](http://www.javarticles.com/wp-content/uploads/2016/06/SNode-Modes.png)

SynchronousQueue Modes

## SynchronousQueue Data Structure

In the below example, we create a producer thread as well as a consumer thread. The producer threads puts item and the consumer thread consumes the item.

We first create `Runnable` that puts item to `SynchronousQueue`.

```
package com.javarticles.threads;
 
import java.util.concurrent.SynchronousQueue;
import static com.javarticles.threads.PrintUtils.print;
 
public class PutRunnable<T> implements Runnable {
    private T value;
    private SynchronousQueue<T> syncQ;
    PutRunnable(SynchronousQueue<T> syncQ, T value) {
        this.syncQ = syncQ;
        this.value = value;
    }
    public void run() {
        try {
            print("Put " + value);
            syncQ.put(value);
            print("Returned from put");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
}
```
Below is the consumer `Runnable` that consumes item from `SynchronousQueue`.
```
package com.javarticles.threads;
 
import java.util.concurrent.SynchronousQueue;
import static com.javarticles.threads.PrintUtils.print;
 
public class TakerRunnable<T> implements Runnable {
    private T value;
    private SynchronousQueue<T> syncQ;
    TakerRunnable(SynchronousQueue<T> syncQ) {
        this.syncQ = syncQ;
    }
    public void run() {
        try {
            print("Retrieve using take");
            value = syncQ.take();
            print("take() returned " + value);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    public T getValue() {
        return value;
    }
}
```
```
package com.javarticles.threads;
 
public class PrintUtils {
    public static void print(String s) {
        System.out.println(Thread.currentThread().getName() + ":" + s);
    }
}
```

In the below example, we create the producer thread and start it. Since there is no consumer thread in waiting yet, the producer thread ends up waiting for a consumer. Next, we start the consumer thread which ends up unblocking the producer thread and returns the item the producer node is holding.

```
package com.javarticles.threads;
 
import java.util.concurrent.SynchronousQueue;
 
public class SynchronousQueueExample {
    public static void main(String[] args) {
        SynchronousQueue<String> sq = new SynchronousQueue<String>();
        Thread putThread = new Thread(new PutRunnable<String>(sq, "One"), "PutThread");
        putThread.start();
        Thread takerThread = new Thread(new TakerRunnable<String>(sq), "TakerThread");
        takerThread.start();
    }
}
```
```
PutThread:Put One
TakerThread:Retrieve using take
PutThread:Returned from put
TakerThread:take() returned One
```


[![SynchronousQueue Data Structure](http://www.javarticles.com/wp-content/uploads/2016/06/SyncQBasicDataStructure.png)](http://www.javarticles.com/wp-content/uploads/2016/06/SyncQBasicDataStructure.png)

SynchronousQueue Data Structure

## Multiple takers

In this example, we will start couple of consumer threads and then the producer threads.
```
package com.javarticles.threads;
 
import java.util.concurrent.SynchronousQueue;
 
public class SynchronousQueueMultipleTakersExample {
    public static void main(String[] args) throws InterruptedException {
        SynchronousQueue<String> sq = new SynchronousQueue<String>();
        Thread takerThread1 = new Thread(new TakerRunnable<String>(sq), "TakerThread1");
        takerThread1.start();
         
        Thread takerThread2 = new Thread(new TakerRunnable<String>(sq), "TakerThread2");
        takerThread2.start();
         
        Thread putThread1 = new Thread(new PutRunnable<String>(sq, "One"), "PutThread1");
        putThread1.start();
         
        Thread putThread2 = new Thread(new PutRunnable<String>(sq, "Two"), "PutThread2");
        putThread2.start();
    }
}
```
The first consumer thread started gets fulfilled by the producer thread so by the time second consumer thread starts it is no more in the queue. The second consumer thread is next fulfilled by the second producer thread.

```
TakerThread1:Retrieve using take
TakerThread2:Retrieve using take
PutThread1:Put One
PutThread1:Returned from put
TakerThread1:take() returned One
PutThread2:Put Two
TakerThread2:take() returned Two
PutThread2:Returned from put
```
In the next example, after the consumer threads are started we wait for a second so that both the consumers end up in the queue waiting. The second consumer is at the head and it points to the first consumer.
Once we start the producer threads, the first producer fulfills the second consumer and dequeues it. The subsequent producer will dequeue the first consumer.

```
package com.javarticles.threads;
 
import java.util.concurrent.SynchronousQueue;
 
public class SynchronousQueueMultipleTakersExample {
    public static void main(String[] args) throws InterruptedException {
        SynchronousQueue sq = new SynchronousQueue();
        Thread takerThread1 = new Thread(new TakerRunnable(sq), "TakerThread1");
        takerThread1.start();
         
        Thread takerThread2 = new Thread(new TakerRunnable(sq), "TakerThread2");
        takerThread2.start();
         
        Thread.sleep(1000);
         
        Thread putThread1 = new Thread(new PutRunnable(sq, "One"), "PutThread1");
        putThread1.start();
         
        Thread putThread2 = new Thread(new PutRunnable(sq, "Two"), "PutThread2");
        putThread2.start();
    }
}
```
```
TakerThread2:Retrieve using take
TakerThread1:Retrieve using take
PutThread1:Put One
PutThread1:Returned from put
TakerThread1:take() returned One
PutThread2:Put Two
PutThread2:Returned from put
TakerThread2:take() returned Two
```
In the below diagram, the first consumer started becomes the head and the thread is blocked. The second consumer started becomes the new head whereas its ‘next’ points to the first consumer. When the producer thread starts, it dequeues the second consumer and removes it from the SynchronousQueue. The second consumer’s ‘match’ now points to the producer fulfilling.

[![SynchronousQueue Multiple Takers](http://www.javarticles.com/wp-content/uploads/2016/06/SyncQMultipleTakers.png)](http://www.javarticles.com/wp-content/uploads/2016/06/SyncQMultipleTakers.png)

Multiple Takers

## Multiple producers

The below diagram depicts the reverse case where the producer threads are started first and then the consumer threads. As you can see the producer nodes are in stack waiting for consumer. Once the consumer thread comes in, it dequeues the second producer.

What is left in the end is the first producer which is still waiting for a consumer.

[![SynchronousQueue Multiple Producers](http://www.javarticles.com/wp-content/uploads/2016/06/SyncQMultipleProducers.png)](http://www.javarticles.com/wp-content/uploads/2016/06/SyncQMultipleProducers.png)

SynchronousQueue Multiple Producers

## SynchronousQueue Offer Example

code>SynchronousQueue.offer(item), inserts the specified item into the queue, if another thread is waiting to receive it. In the below example we fist start the producer thread offering an item but since there is no consumer thread yet, `offer(item)` returns null. Next we start a consumer thread, make sure it is on the queue and then start the producer thread. The `offer(item)` API returns `true` as there is a consumer thread alreeady in waiting.

```
package com.javarticles.threads;
 
import static com.javarticles.threads.PrintUtils.print;
 
import java.util.concurrent.SynchronousQueue;
 
public class OfferRunnable<T> implements Runnable {
    private T value;
    private SynchronousQueue<T> syncQ;
    OfferRunnable(SynchronousQueue<T> syncQ, T value) {
        this.syncQ = syncQ;
        this.value = value;
    }
    public void run() {
        print("Offer(" + value + ")=" + syncQ.offer(value));
    }
}
```

```
package com.javarticles.threads;
 
import java.util.concurrent.SynchronousQueue;
 
public class SynchronousQueueOfferExample {
    public static void main(String[] args) throws InterruptedException {
        SynchronousQueue<String> sq = new SynchronousQueue<String>();
        Thread offerThread = new Thread(new OfferRunnable<String>(sq, "One"), "OfferThread");
        offerThread.start();
         
        offerThread.join();
         
        Thread takerThread = new Thread(new TakerRunnable<String>(sq), "TakerThread");
        takerThread.start();
         
        Thread.sleep(500);
         
        offerThread = new Thread(new OfferRunnable<String>(sq, "One"), "OfferThread");
        offerThread.start();
    }
}
```
```
OfferThread:Offer(One)=false
TakerThread:Retrieve using take
OfferThread:Offer(One)=true
TakerThread:take() returned One
```

## Canceling Node of a SynchronousQueue

A node gets canceled due to either timeout or thread interrupt. Once canceled the node’s ‘match’ points to itself and is removed from the queue.

[![SynchronousQueue Canceling Node](http://www.javarticles.com/wp-content/uploads/2016/06/SyncQClear.png)](http://www.javarticles.com/wp-content/uploads/2016/06/SyncQClear.png)

SynchronousQueue Canceling Node

## SynchronousQueue Timed Offer Example

In the below example we will see usage of time boxed `Offer(item)`. The insert operation will wait for another thread to receive it only for the specified wait time.

```
package com.javarticles.threads;
 
import static com.javarticles.threads.PrintUtils.print;
 
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;
 
public class OfferTimedRunnable<T> implements Runnable {
    private T value;
    private SynchronousQueue<T> syncQ;
    private long mills;
 
    OfferTimedRunnable(SynchronousQueue<T> syncQ, T value, long mills) {
        this.syncQ = syncQ;
        this.value = value;
        this.mills = mills;
    }
 
    public void run() {
        try {
            print("Offer(" + value + ", " + mills + ")="
                    + syncQ.offer(value, mills, TimeUnit.MILLISECONDS));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

The time boxed `offer(item)` will return `true` if successful in inserting the item. In the below case, it manages to insert before the timeout period.

```
package com.javarticles.threads;
 
import java.util.concurrent.SynchronousQueue;
 
public class SynchronousQueueOfferTimedExample {
    public static void main(String[] args) throws InterruptedException {
        SynchronousQueue<String> sq = new SynchronousQueue<String>();
        Thread putThread = new Thread(new PutRunnable<String>(sq, "One"), "PutThread");
        putThread.start();
         
        Thread offerThread = new Thread(new OfferTimedRunnable<String>(sq, "Two", 500), "OfferThread");
        offerThread.start();
         
        Thread takerThread1 = new Thread(new TakerRunnable<String>(sq), "TakerThread1");
        takerThread1.start();
         
        Thread takerThread2 = new Thread(new TakerRunnable<String>(sq), "TakerThread2");
        takerThread2.start();
    }
}
```
```
PutThread:Put One
TakerThread1:Retrieve using take
TakerThread1:take() returned Two
TakerThread2:Retrieve using take
OfferThread:Offer(Two, 500)=true
PutThread:Returned from put
TakerThread2:take() returned One
```

Now we will now explicitly wait for a second and then start the consumer so that `offer(item)` fails as the specified waiting time elapses.

```
package com.javarticles.threads;
 
import java.util.concurrent.SynchronousQueue;
 
public class SynchronousQueueOfferCancelExample {
    public static void main(String[] args) throws InterruptedException {
        SynchronousQueue<String> sq = new SynchronousQueue<String>();
        Thread putThread = new Thread(new PutRunnable<String>(sq, "One"), "PutThread");
        putThread.start();
         
        Thread offerThread = new Thread(new OfferTimedRunnable<String>(sq, "Two", 500), "OfferThread");
        offerThread.start();
         
        Thread.sleep(1000);
         
        Thread takerThread1 = new Thread(new TakerRunnable<String>(sq), "TakerThread1");
        takerThread1.start();
         
        Thread takerThread2 = new Thread(new TakerRunnable<String>(sq), "TakerThread2");
        takerThread2.start();
 
        if (takerThread1.isAlive()) {
            takerThread1.interrupt();
        }
         
        if (takerThread2.isAlive()) {
            takerThread2.interrupt();
        }
    }
}
```

```
PutThread:Put One
OfferThread:Offer(Two, 500)=false
TakerThread1:Retrieve using take
TakerThread1:take() returned One
TakerThread2:Retrieve using take
PutThread:Returned from put
java.lang.InterruptedException
    at java.util.concurrent.SynchronousQueue.take(SynchronousQueue.java:928)
    at com.javarticles.threads.TakerRunnable.run(TakerRunnable.java:15)
    at java.lang.Thread.run(Thread.java:745)
```

## Download the source code

This was an example about SynchronousQueue.

You can download the source code here: [**javaSynchronizedQueueExample.zip**](http://www.javarticles.com/wp-content/uploads/2016/06/javaSynchronizedQueueExample.zip)

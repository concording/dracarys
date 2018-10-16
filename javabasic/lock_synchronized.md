
# Synchronization and Locks

**关键点**

1. 只有方法(或者blocks)能被synchronized，类和变量都不行。
2. 每一个Object只有一个锁(lock)。 
3. 如果两个线程执行某个类的同步方法，并且这两个线程使用的是这个类的同一个实例调用的这个同步方法时，在某一个时刻只有一个线程能够执行这个方法，另一个线程必须等到第一个线程结束调用。
4. 一个类如果有同步和非同步的方法，非同步的方法仍然能够同时被多个线程访问。
5. 一个线程在sleep的时候仍然持有锁。
6. 如果一个线程已经获得某个对象上的锁，那么这个线程在访问这个对象上其他的同步方法的时候不需要再次获得这个锁，JVM会记录该线程已经获得的锁信息。

1. Threads calling non-static synchronized methods in the same class will only block each other if they're invoked using the same instance. That's because they each lock on this instance, and if they're called using two dif- ferent instances, they get two locks, which do not interfere with each other.
2. Threads calling static synchronized methods in the same class will always block each other—they all lock on the same Class instance
3. A static synchronized method and a non-static synchronized method will not block each other, ever. The static method locks on a Class instance while the non-static method locks on the this instance—these actions do not interfere with each other at all.
4. For synchronized blocks, you have to look at exactly what object has been used for locking. (What's inside the parentheses after the word synchro- nized?) Threads that synchronize on the same object will block each other. Threads that synchronize on different objects will not.

**个人理解**

non-static synchronized方法锁的是实例，只有在同一个实例上调用同步代码才会锁住。static synchronized方法锁的是类，只要调用这个类的同步代码块都会被锁。静态和非静态同步方法不会锁住对方。

如果非静态同步方法访问静态同步方法的字段或者静态同步方法访问非静态同步方法的字段会怎样？

However—what if you have a non-static method that accesses a static field? Or a static method that accesses a non-static field (using an instance)? In these cases things start to get messy quickly, and there's a very good chance that things will not work the way you want. If you've got a static method accessing a non-static field, and you synchronize the method, you acquire a lock on the Class object. But what if there's another method that also accesses the non-static field, this time using a non-static method? It probably synchronizes on the current instance (this) instead. Remember that a static synchronized method and a non-static synchronized method will not block each other—they can run at the same time. Similarly, if you access a static field using a non-static method, two threads might invoke that method using two different this instances. Which means they won't block each other, because they use different locks. Which means two threads are simultaneously accessing the same static field—exactly the sort of thing we're trying to prevent.
It gets very confusing trying to imagine all the weird things that can happen here. To keep things simple: in order to make a class thread-safe, methods that access changeable fields need to be synchronized.
Access to static fields should be done from static synchronized methods. Access to non-static fields should be done from non-static synchronized methods.



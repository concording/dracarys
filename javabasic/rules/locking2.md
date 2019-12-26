## Locking rules2

#### [Do not use an instance lock to protect shared static data](https://wiki.sei.cmu.edu/confluence/display/java/LCK06-J.+Do+not+use+an+instance+lock+to+protect+shared+static+data)

***错误的使用方式***
```
public final class CountBoxes implements Runnable {
  private static volatile int counter;
  // ...
  private final Object lock = new Object();
 
  @Override public void run() {
    synchronized (lock) {
      counter++;
      // ...
    }
  }
 
  public static void main(String[] args) {
    for (int i = 0; i < 2; i++) {
    new Thread(new CountBoxes()).start();
    }
  }
}
```
```
public final class CountBoxes implements Runnable {
  private static volatile int counter;
  // ...
 
  public synchronized void run() {
    counter++;
    // ...
  }
  // ...
}
```
***正确的使用方式***
```
public class CountBoxes implements Runnable {
  private static int counter;
  // ...
  private static final Object lock = new Object();
 
  public void run() {
    synchronized (lock) {
      counter++;
      // ...
    }
  }
  // ...
}
```
> Programs must not use instance locks to protect static shared data because instance locks are ineffective when two or more instances of the class are created.

#### [Avoid deadlock by requesting and releasing locks in the same order](https://wiki.sei.cmu.edu/confluence/display/java/LCK07-J.+Avoid+deadlock+by+requesting+and+releasing+locks+in+the+same+order)

***错误的使用方式***
```

```
***正确的使用方式***
```

```

#### [Ensure actively held locks are released on exceptional conditions](https://wiki.sei.cmu.edu/confluence/display/java/LCK08-J.+Ensure+actively+held+locks+are+released+on+exceptional+conditions)

A `ReentrantLock` is owned by the thread last successfully locking, but not yet unlocking it. A thread invoking `lock` will return, successfully acquiring the lock, when the lock is not owned by another thread.

Consequently, an unreleased lock in any thread will prevent other threads from acquiring the same lock. Programs must release all actively held locks on exceptional conditions. Intrinsic locks of class objects used for method and block [synchronization](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-synchro) are automatically released on exceptional conditions (such as abnormal thread termination).

***错误的使用方式***
```
public final class Client {
  private final Lock lock = new ReentrantLock();
 
  public void doSomething(File file) {
    InputStream in = null;
    try {
      lock.lock();
      in = new FileInputStream(file);
 
      // Perform operations on the open file
 
      lock.unlock();
    } catch (FileNotFoundException x) {
      // Handle exception
    } finally {
      if (in != null) {
        try {
          in.close();
        } catch (IOException x) {
          // Handle exception
        } 
      }
    }
  }
}
```
This noncompliant code example attempts to rectify the problem of the lock not being released by invoking `Lock.unlock()` in the `finally` block. This code ensures that the lock is released regardless of whether or not an exception occurs. However, it does not acquire the lock until after trying to open the file. If the file cannot be opened, the lock may be unlocked without ever being locked in the first place.

```
public final class Client {
  private final Lock lock = new ReentrantLock();
 
  public void doSomething(File file) {
    InputStream in = null;
    try {
      in = new FileInputStream(file);
      lock.lock();
      // Perform operations on the open file
    } catch (FileNotFoundException fnf) {
      // Forward to handler
    } finally {
      lock.unlock();
      if (in != null) {
        try {
          in.close();
        } catch (IOException e) {
          // Forward to handler
        }
      }
    }
  }
}


```
```
final class DateHandler {
 
  private final Date date = new Date();
 
  private final Lock lock = new ReentrantLock();
 
  // str could be null
  public void doSomething(String str) {
    lock.lock();
    String dateString = date.toString();
    if (str.equals(dateString)) {
      // ...
    }
    // ...
 
    lock.unlock();
  }
}
```

***正确的使用方式***
```
public final class Client {
  private final Lock lock = new ReentrantLock();
 
  public void doSomething(File file) {
    InputStream in = null;
    lock.lock();
    try {
      in = new FileInputStream(file);
      // Perform operations on the open file
    } catch (FileNotFoundException fnf) {
      // Forward to handler
    } finally {
      lock.unlock();
 
      if (in != null) {
        try {
          in.close();
        } catch (IOException e) {
          // Forward to handler
        }
      }
    }
  }
}
```

#### [Do not perform operations that can block while holding a lock](https://wiki.sei.cmu.edu/confluence/display/java/LCK09-J.+Do+not+perform+operations+that+can+block+while+holding+a+lock)

Holding locks while performing time-consuming or blocking operations can severely degrade system performance and can result in [starvation](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-starvation). Furthermore, [deadlock](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary)  can result if interdependent threads block indefinitely. Blocking operations include network, file, and console I/O (for example, `Console.readLine()`) and object serialization. Deferring a thread indefinitely also constitutes a blocking operation. Consequently, programs must not perform blocking operations while holding a lock.

When the Java Virtual Machine (JVM) interacts with a file system that operates over an unreliable network, file I/O might incur a large performance penalty. In such cases, avoid file I/O over the network while holding a lock. File operations (such as logging) that could block while waiting for the output stream lock or for I/O to complete could be performed in a dedicated thread to speed up task processing. Logging requests can be added to a queue, assuming that the queue's `put()` operation incurs little overhead as compared to file I/O [[Goetz 2006](https://wiki.sei.cmu.edu/confluence/display/java/Rule+AA.+References#RuleAA.References-Goetz06)].

***错误的使用方式***
```

```
***正确的使用方式***
```

```

#### [Use a correct form of the double-checked locking idiom](https://wiki.sei.cmu.edu/confluence/display/java/LCK10-J.+Use+a+correct+form+of+the+double-checked+locking+idiom)

The _double-checked locking idiom_ is a software design pattern used to reduce the overhead of acquiring a lock by first testing the locking criterion without actually acquiring the lock. Double-checked locking improves performance by limiting [synchronization](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-synchro) to the rare case of computing the field's value or constructing a new instance for the field to reference and by foregoing synchronization during the common case of retrieving an already-created instance or value.

Incorrect forms of the double-checked locking idiom include those that allow publication of an uninitialized or partially initialized object. Consequently, only those forms of the double-checked locking idiom that correctly establish a [happens-before relationship](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-happens-beforeorder) both for the `helper` reference and for the complete construction of the `Helper` instance are permitted.

The double-checked locking idiom is frequently used to implement a [singleton factory pattern ](http://www.wikijava.org/wiki/Singleton_Factory_patterns_example)that performs _lazy initialization_. Lazy initialization defers the construction of a member field or an object referred to by a member field until an instance is actually required rather than computing the field value or constructing the referenced object in the class's constructor. Lazy initialization helps to break harmful circularities in class and instance initialization. It also enables other optimizations [[Bloch 2005](https://wiki.sei.cmu.edu/confluence/display/java/Rule+AA.+References#RuleAA.References-Bloch05)].

Lazy initialization uses either a class or an instance method, depending on whether the member object is static. The method checks whether the instance has already been created and, if not, creates it. When the instance already exists, the method simply returns the instance:


***错误的使用方式***

> Writes that initialize the `Helper` object and the write to the `helper` field can be done or perceived out of order. As a result, a thread which invokes `getHelper()` could see a non-null reference to a `helper` object, but see the default values for fields of the `helper` object, rather than the values set in the constructor.

> Even if the compiler does not reorder those writes, on a multiprocessor, the processor or the memory system may reorder those writes, as perceived by a thread running on another processor.

```
// Double-checked locking idiom
final class Foo {
  private Helper helper = null;
  public Helper getHelper() {
    if (helper == null) {
      synchronized (this) {
        if (helper == null) {
          helper = new Helper();
        }
      }
    }
    return helper;
  }
 
  // Other methods and members...
}
```

In this noncompliant code example, the `Helper` class is made [immutable](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary) by declaring its fields final. The JMM guarantees that immutable objects are fully constructed before they become visible to any other thread. The block [synchronization](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-synchro) in the `getHelper()` method guarantees that all threads that can see a non-null value of the helper field will also see the fully initialized `Helper` object.

However, this code is not guaranteed to succeed on all Java Virtual Machine platforms because there is no [happens-before relationship](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-happens-beforeorder) between the first read and third read of `helper`. Consequently, it is possible for the third read of `helper` to obtain a stale null value (perhaps because its value was cached or reordered by the compiler), causing the `getHelper()` method to return a null pointer.

```
public final class Helper {
  private final int n;
  
  public Helper(int n) {
    this.n = n;
  }
  
  // Other fields and methods, all fields are final
}
  
final class Foo {
  private Helper helper = null;
  
  public Helper getHelper() {
    if (helper == null) {            // First read of helper
      synchronized (this) {
        if (helper == null) {        // Second read of helper
          helper = new Helper(42);
        }
      }
    }
    return helper;                   // Third read of helper
  }
}
```
***正确的使用方式***
```
// Works with acquire/release semantics for volatile
// Broken under JDK 1.4 and earlier
final class Foo {
  private volatile Helper helper = null;
 
  public Helper getHelper() {
    if (helper == null) {
      synchronized (this) {
        if (helper == null) {
          helper = new Helper();
        }
      }
    }
    return helper;
  }
}
```
Initialization of the static `helper` field is deferred until the `getInstance()` method is called. The necessary happens-before relationships are created by the combination of the class loader's actions loading and initializing the `Holder` instance and the guarantees provided by the Java [memory model](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-mem) (JMM). This idiom is a better choice than the double-checked locking idiom for lazily initializing static fields [[Bloch 2008](https://wiki.sei.cmu.edu/confluence/display/java/Rule+AA.+References#RuleAA.References-Bloch08)]. However, this idiom cannot be used to lazily initialize instance fields [[Bloch 2001](https://wiki.sei.cmu.edu/confluence/display/java/Rule+AA.+References#RuleAA.References-Bloch01)].

```
final class Foo {
  private static final Helper helper = new Helper();
 
  public static Helper getHelper() {
    return helper;
  }
}
```

This compliant solution (originally suggested by Alexander Terekhov [[Pugh 2004](https://wiki.sei.cmu.edu/confluence/display/java/Rule+AA.+References#RuleAA.References-Pugh04)]) uses a `ThreadLocal` object to track whether each individual thread has participated in the [synchronization](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-synchro) that creates the needed [happens-before relationships](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-happens-bef). Each thread stores a non-null value into its thread-local `perThreadInstance` only inside the synchronized `createHelper()` method; consequently, any thread that sees a null value must establish the necessary happens-before relationships by invoking `createHelper()`.

```
final class Foo {
  private final ThreadLocal<Foo> perThreadInstance =
      new ThreadLocal<Foo>();
  private Helper helper = null;
 
  public Helper getHelper() {
    if (perThreadInstance.get() == null) {
      createHelper();
    }
    return helper;
  }
 
  private synchronized void createHelper() {
    if (helper == null) {
      helper = new Helper();
    }
    // Any non-null value can be used as an argument to set()
    perThreadInstance.set(this);
  }
}
```
```
final class Foo {
  // Lazy initialization
  private static class Holder {
    static Helper helper = new Helper();
  }
 
  public static Helper getInstance() {
    return Holder.helper;
  }
}
```
This compliant solution uses a local variable to reduce the number of unsynchronized reads of the `helper` field to 1\. As a result, if the read of `helper` yields a non-null value, it is cached in a local variable that is inaccessible to other threads and is safely returned.

```
public final class Helper {
  private final int n;
  
  public Helper(int n) {
    this.n = n;
  }
  
  // Other fields and methods, all fields are final
}
  
final class Foo {
  private Helper helper = null;
  
  public Helper getHelper() {
    Helper h = helper;       // Only unsynchronized read of helper
    if (h == null) {
      synchronized (this) {
        h = helper;          // In synchronized block, so this is safe
        if (h == null) {
          h = new Helper(42);
          helper = h;
        }
      }
    }
    return h;
  }
}
```
##### Exceptions

**LCK10-J-EX0:** Use of the noncompliant form of the double-checked locking idiom is permitted for 32-bit primitive values (for example, `int` or `float`) [[Pugh 2004](https://wiki.sei.cmu.edu/confluence/display/java/Rule+AA.+References#RuleAA.References-Pugh04)], although this usage is discouraged. The noncompliant form establishes the necessary [happens-before relationship](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-happens-beforeorder) between threads that see an initialized version of the primitive value. The second happens-before relationship (for the initialization of the fields of the referent) is of no practical value because unsynchronized reads and writes of primitive values up to 32-bits are guaranteed to be atomic. Consequently, the noncompliant form establishes the only needed happens-before relationship in this case. Note, however, that the noncompliant form fails for `long` and `double`because unsynchronized reads or writes of 64-bit primitives lack a guarantee of atomicity and consequently require a second happens-before relationship to guarantee that all threads see only fully assigned 64-bit values (see  [VNA05-J. Ensure atomicity when reading and writing 64-bit values](https://wiki.sei.cmu.edu/confluence/display/java/VNA05-J.+Ensure+atomicity+when+reading+and+writing+64-bit+values) for more information).

#### [Avoid client-side locking when using classes that do not commit to their locking strategy](https://wiki.sei.cmu.edu/confluence/display/java/LCK11-J.+Avoid+client-side+locking+when+using+classes+that+do+not+commit+to+their+locking+strategy)

***错误的使用方式***
```

```
***正确的使用方式***
```

```

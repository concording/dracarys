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

***错误的使用方式***
```

```
***正确的使用方式***
```

```

#### [Use a correct form of the double-checked locking idiom](https://wiki.sei.cmu.edu/confluence/display/java/LCK10-J.+Use+a+correct+form+of+the+double-checked+locking+idiom)

***错误的使用方式***
```

```
***正确的使用方式***
```

```

#### [Avoid client-side locking when using classes that do not commit to their locking strategy](https://wiki.sei.cmu.edu/confluence/display/java/LCK11-J.+Avoid+client-side+locking+when+using+classes+that+do+not+commit+to+their+locking+strategy)

***错误的使用方式***
```

```
***正确的使用方式***
```

```

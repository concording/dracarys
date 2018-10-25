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

***错误的使用方式***
```

```
***正确的使用方式***
```

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

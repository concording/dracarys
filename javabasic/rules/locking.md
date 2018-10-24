## Locking rules

#### [Use private final lock objects to synchronize classes that may interact with untrusted code](https://wiki.sei.cmu.edu/confluence/display/java/LCK00-J.+Use+private+final+lock+objects+to+synchronize+classes+that+may+interact+with+untrusted+code)

```
public class SomeObject {
  private final Object lock = new Object(); // private final lock object
 
  public void changeValue() {
    synchronized (lock) { // Locks on the private Object
      // ...
    }
  }
}
```

#### [Do not synchronize on objects that may be reused](https://wiki.sei.cmu.edu/confluence/display/java/LCK01-J.+Do+not+synchronize+on+objects+that+may+be+reused)

***错误的使用方式***
`private final Boolean initialized = Boolean.FALSE;`  `Boolean` 类型只有2个值: true and false.如果其他代码不小心锁在的`Boolean`字面量上，有可能会死锁。

`private final Integer Lock = count;`

`private final String lock = new String("LOCK").intern();`

`private final String lock = "LOCK";`

 
***正确的使用方式***
```
private int count = 0;
private final Integer Lock = new Integer(count);
 
public void doSomething() {
  synchronized (Lock) {
    count++;
    // ...
  }
}
```
`private final String lock = new String("LOCK");`

`private final Object lock = new Object();`

#### [ Do not synchronize on the class object returned by getClass()](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88487849)

父类和子类有可能锁的不是一个对象。

***错误的使用方式***
```
class Base {
  static DateFormat format =
      DateFormat.getDateInstance(DateFormat.MEDIUM);
 
  public Date parse(String str) throws ParseException {
    synchronized (getClass()) {
      return format.parse(str);
    }
  }
}
 
class Derived extends Base {
  public Date doSomethingAndParse(String str) throws ParseException {
    synchronized (Base.class) {
      // ...
      return format.parse(str);
    }
  }
}
```
```
class Base {
  static DateFormat format =
      DateFormat.getDateInstance(DateFormat.MEDIUM);
 
  public Date parse(String str) throws ParseException {
    synchronized (getClass()) { // Intend to synchronizes on Base.class
      return format.parse(str);
    }
  }
 
  public Date doSomething(String str) throws ParseException {
    return new Helper().doSomethingAndParse(str);
  }
 
  private class Helper {
    public Date doSomethingAndParse(String str) throws ParseException {
      synchronized (getClass()) { // Synchronizes on Helper.class
        // ...
        return format.parse(str);
      }
    }
  }
}
```

***正确的使用方式***
```
class Base {
  static DateFormat format =
      DateFormat.getDateInstance(DateFormat.MEDIUM);
 
  public Date parse(String str) throws ParseException {
    try {
      synchronized (Class.forName("Base")) {
        return format.parse(str);
      }
    } catch (ClassNotFoundException x) {
      // "Base" not found; handle error
    }
    return null;
  }
}
```
```
class Base {
  // ...
 
  public Date parse(String str) throws ParseException {
    synchronized (Base.class) {
      return format.parse(str);
    }
  }
 
  private class Helper {
    public Date doSomethingAndParse(String str) throws ParseException {
      synchronized (Base.class) { // Synchronizes on Base class literal
        // ...
        return format.parse(str);
      }
    }
  }
}
```

#### [Do not synchronize on the intrinsic locks of high-level concurrency objects](https://wiki.sei.cmu.edu/confluence/display/java/LCK03-J.+Do+not+synchronize+on+the+intrinsic+locks+of+high-level+concurrency+objects)

Code that uses the intrinsic lock of a `Lock` object is likely to interact with code that uses the `Lock` interface.These two components will believe they are protecting data with the same lock, while they are, in fact, using two distinct locks. As such, the `Lock` will fail to protect any data.

***错误使用方式***
```
private final Lock lock = new ReentrantLock();
 
public void doSomething() {
  synchronized(lock) {
    // ...
  }
}
```

***正确使用方式***
```
private final Lock lock = new ReentrantLock();
 
public void doSomething() {
  lock.lock();
  try {
    // ...
  } finally {
    lock.unlock();
  }
}
```

#### [Do not synchronize on a collection view if the backing collection is accessible](https://wiki.sei.cmu.edu/confluence/display/java/LCK04-J.+Do+not+synchronize+on+a+collection+view+if+the+backing+collection+is+accessible)

In this example, `HashMap` provides the backing collection for the synchronized map represented by `mapView`, which provides the backing collection for `setView`, as shown in the following figure.

![](https://wiki.sei.cmu.edu/confluence/download/attachments/88487846/con06-j-backing-collection.JPG?version=1&modificationDate=1270734183000&api=v2)

The `HashMap` object is inaccessible, but `mapView` is accessible via the public `getMap()` method. Because the `synchronized`statement uses the intrinsic lock of `setView` rather than of `mapView`, another thread can modify the synchronized map and invalidate the `k` iterator.

***错误的使用方式***
```
private final Map<Integer, String> mapView =
    Collections.synchronizedMap(new HashMap<Integer, String>());
private final Set<Integer> setView = mapView.keySet();
 
public Map<Integer, String> getMap() {
  return mapView;
}
 
public void doSomething() {
  synchronized (setView) {  // Incorrectly synchronizes on setView
    for (Integer k : setView) {
      // ...
    }
  }
}
```
***正确的使用方式***
```
private final Map<Integer, String> mapView =
    Collections.synchronizedMap(new HashMap<Integer, String>());
private final Set<Integer> setView = mapView.keySet();
 
public Map<Integer, String> getMap() {
  return mapView;
}
 
public void doSomething() {
  synchronized (mapView) {  // Synchronize on map, rather than set
    for (Integer k : setView) {
      // ...
    }
  }
}
```

#### [Synchronize access to static fields that can be modified by untrusted code](https://wiki.sei.cmu.edu/confluence/display/java/LCK05-J.+Synchronize+access+to+static+fields+that+can+be+modified+by+untrusted+code)

Methods that can both modify a static field and be invoked from [untrusted code](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-untruste) must [synchronize](https://wiki.sei.cmu.edu/confluence/display/java/Rule+BB.+Glossary#RuleBB.Glossary-synchroniz) access to the static field. Even when client-side locking is a specified requirement of the method, untrusted clients can fail to synchronize (whether inadvertently or maliciously). Because the static field is shared by all clients, untrusted clients may violate the contract by failing to provide suitable locking.

According to Joshua Bloch [[Bloch 2008](https://wiki.sei.cmu.edu/confluence/display/java/Rule+AA.+References#RuleAA.References-Bloch08)]:

> If a method modifies a static field, you must synchronize access to this field, even if the method is typically used only by a single thread. It is not possible for clients to perform external synchronization on such a method because there can be no guarantee that unrelated clients will do likewise.

Documented design intent is irrelevant when dealing with untrusted code because an attacker can always choose to ignore the documentation.

***错误的代码***
```
/* This class is not thread-safe */
public final class CountHits {
  private static int counter;
 
  public void incrementCounter() {
    counter++;
  }
}
```
***正确的代码***
```
/* This class is thread-safe */
public final class CountHits {
  private static int counter;
  private static final Object lock = new Object();
 
  public void incrementCounter() {
    synchronized (lock) {
      counter++;
    }
  }
}

```

#### [Do not use an instance lock to protect shared static data](https://wiki.sei.cmu.edu/confluence/display/java/LCK06-J.+Do+not+use+an+instance+lock+to+protect+shared+static+data)

#### [Avoid deadlock by requesting and releasing locks in the same order](https://wiki.sei.cmu.edu/confluence/display/java/LCK07-J.+Avoid+deadlock+by+requesting+and+releasing+locks+in+the+same+order)

#### [Ensure actively held locks are released on exceptional conditions](https://wiki.sei.cmu.edu/confluence/display/java/LCK08-J.+Ensure+actively+held+locks+are+released+on+exceptional+conditions)

#### [Do not perform operations that can block while holding a lock](https://wiki.sei.cmu.edu/confluence/display/java/LCK09-J.+Do+not+perform+operations+that+can+block+while+holding+a+lock)

#### [Use a correct form of the double-checked locking idiom](https://wiki.sei.cmu.edu/confluence/display/java/LCK10-J.+Use+a+correct+form+of+the+double-checked+locking+idiom)

#### [Avoid client-side locking when using classes that do not commit to their locking strategy](https://wiki.sei.cmu.edu/confluence/display/java/LCK11-J.+Avoid+client-side+locking+when+using+classes+that+do+not+commit+to+their+locking+strategy)
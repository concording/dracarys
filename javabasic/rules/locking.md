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

#### [Do not synchronize on a collection view if the backing collection is accessible](https://wiki.sei.cmu.edu/confluence/display/java/LCK04-J.+Do+not+synchronize+on+a+collection+view+if+the+backing+collection+is+accessible)

#### [Synchronize access to static fields that can be modified by untrusted code](https://wiki.sei.cmu.edu/confluence/display/java/LCK05-J.+Synchronize+access+to+static+fields+that+can+be+modified+by+untrusted+code)

#### [Do not use an instance lock to protect shared static data](https://wiki.sei.cmu.edu/confluence/display/java/LCK06-J.+Do+not+use+an+instance+lock+to+protect+shared+static+data)

#### [Avoid deadlock by requesting and releasing locks in the same order](https://wiki.sei.cmu.edu/confluence/display/java/LCK07-J.+Avoid+deadlock+by+requesting+and+releasing+locks+in+the+same+order)

#### [Ensure actively held locks are released on exceptional conditions](https://wiki.sei.cmu.edu/confluence/display/java/LCK08-J.+Ensure+actively+held+locks+are+released+on+exceptional+conditions)

#### [Do not perform operations that can block while holding a lock](https://wiki.sei.cmu.edu/confluence/display/java/LCK09-J.+Do+not+perform+operations+that+can+block+while+holding+a+lock)

#### [Use a correct form of the double-checked locking idiom](https://wiki.sei.cmu.edu/confluence/display/java/LCK10-J.+Use+a+correct+form+of+the+double-checked+locking+idiom)

#### [Avoid client-side locking when using classes that do not commit to their locking strategy](https://wiki.sei.cmu.edu/confluence/display/java/LCK11-J.+Avoid+client-side+locking+when+using+classes+that+do+not+commit+to+their+locking+strategy)
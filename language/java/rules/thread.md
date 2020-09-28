# Assignment of lazy-initialized members should be the last step with double-checked locking

Double-checked locking can be used for lazy initialization of `volatile` fields, but only if field assignment is the last step in the `synchronized` block. Otherwise you run the risk of threads accessing a half-initialized object.

## Noncompliant Code Example

```
public class MyClass {

  private volatile List<String> strings;

  public List<String> getStrings() {
    if (strings == null) {  // check#1
      synchronized(MyClass.class) {
        if (strings == null) {
          strings = new ArrayList<>();  // Noncompliant
          strings.add("Hello");  //When threadA gets here, threadB can skip the synchronized block because check#1 is false
          strings.add("World");
        }
      }
    }
    return strings;
  }
}
```
## Compliant Solution

```
public class MyClass {

  private volatile List<String> strings;

  public List<String> getStrings() {
    if (strings == null) {  // check#1
      synchronized(MyClass.class) {
        if (strings == null) {
          List<String> tmpList = new ArrayList<>();
          tmpList.add("Hello");
          tmpList.add("World");
          strings = tmpList;
        }
      }
    }
    return strings;
  }
}
```

## See

*   [CERT, LCK10-J.](https://www.securecoding.cert.org/confluence/x/IgAZAg) - Use a correct form of the double-checked locking idiom


# Blocks should be synchronized on "private final" fields

Synchronizing on a class field synchronizes not on the field itself, but on the object assigned to it. So synchronizing on a non-`final` field makes it possible for the field's value to change while a thread is in a block synchronized on the old value. That would allow a second thread, synchronized on the new value, to enter the block at the same time.

The story is very similar for synchronizing on parameters; two different threads running the method in parallel could pass two different object instances in to the method as parameters, completely undermining the synchronization.

## Noncompliant Code Example

```
private String color = "red";

private void doSomething(){
  synchronized(color) {  // Noncompliant; lock is actually on object instance "red" referred to by the color variable
    //...
    color = "green"; // other threads now allowed into this block
    // ...
  }
  synchronized(new Object()) { // Noncompliant this is a no-op.
     // ...
  }
}
```
## Compliant Solution

```
private String color = "red";
private final Object lockObj = new Object();

private void doSomething(){
  synchronized(lockObj) {
    //...
    color = "green";
    // ...
  }
}
```
## See

*   [MITRE, CWE-412](http://cwe.mitre.org/data/definitions/412.html) - Unrestricted Externally Accessible Lock
*   [MITRE, CWE-413](http://cwe.mitre.org/data/definitions/413) - Improper Resource Locking
*   [CERT, LCK00-J.](https://www.securecoding.cert.org/confluence/x/6IEzAg) - Use private final lock objects to synchronize classes that may interact with untrusted code

# Threads should not be started in constructors

The problem with invoking `Thread.start()` in a constructor is that you'll have a confusing mess on your hands if the class is ever extended because the superclass' constructor will start the thread before the child class has truly been initialized.

This rule raises an issue any time `start` is invoked in the constructor of a non-`final` class.

## Noncompliant Code Example

```
public class MyClass {

  Thread thread = null;

  public MyClass(Runnable runnable) {
    thread = new Thread(runnable);
    thread.start(); // Noncompliant
  }
}
```

Starting a thread in a constructor passing in this escapes this. That means that you are actually giving out a reference to your object before it is fully constructed. The thread will start before your constructor finishes. This can result in all kinds of weird behaviors.

## See

*   [CERT, TSM02-J.](https://www.securecoding.cert.org/confluence/x/ZQIRAg) - Do not use background threads during class initialization

# "InterruptedException" should not be ignored

`InterruptedExceptions` should never be ignored in the code, and simply logging the exception counts in this case as "ignoring". The throwing of the `InterruptedException` clears the interrupted state of the Thread, so if the exception is not handled properly the fact that the thread was interrupted will be lost. Instead, `InterruptedExceptions` should either be rethrown - immediately or after cleaning up the method's state - or the thread should be re-interrupted by calling `Thread.interrupt()` even if this is supposed to be a single-threaded application. Any other course of action risks delaying thread shutdown and loses the information that the thread was interrupted - probably without finishing its task.

Similarly, the `ThreadDeath` exception should also be propagated. According to its JavaDoc:

> If `ThreadDeath` is caught by a method, it is important that it be rethrown so that the thread actually dies.

## Noncompliant Code Example
```
public void run () {
  try {
    while (true) {
      // do stuff
    }
  }catch (InterruptedException e) { // Noncompliant; logging is not enough
    LOGGER.log(Level.WARN, "Interrupted!", e);
  }
}
```

## Compliant Solution

```
public void run () {
  try {
    while (true) {
      // do stuff
    }
  }catch (InterruptedException e) {
    LOGGER.log(Level.WARN, "Interrupted!", e);
    // Restore interrupted state...
    Thread.currentThread().interrupt();
  }
}
```

## See

*   [MITRE, CWE-391](http://cwe.mitre.org/data/definitions/391.html) - Unchecked Error Condition
*   [Dealing with InterruptedException](https://www.ibm.com/developerworks/java/library/j-jtp05236/index.html?ca=drs-#2.1)


# Non-primitive fields should not be "volatile"

Marking an array `volatile` means that the array itself will always be read fresh and never thread cached, but the items _in_ the array will not be. Similarly, marking a mutable object field `volatile` means the object _reference_ is `volatile` but the object itself is not, and other threads may not see updates to the object state.

This can be salvaged with arrays by using the relevant AtomicArray class, such as `AtomicIntegerArray`, instead. For mutable objects, the `volatile` should be removed, and some other method should be used to ensure thread-safety, such as synchronization, or ThreadLocal storage.

## Noncompliant Code Example

```
private volatile int [] vInts;  // Noncompliant
private volatile MyObj myObj;  // Noncompliant
```

## Compliant Solution

```
private AtomicIntegerArray vInts;
private MyObj myObj;
```

## See

*   [CERT, CON50-J.](https://www.securecoding.cert.org/confluence/x/twD1AQ) - Do not assume that declaring a reference volatile guarantees safe publication of the members of the referenced object

# "ThreadLocal" variables should be cleaned up when no longer used

`ThreadLocal` variables are supposed to be garbage collected once the holding thread is no longer alive. Memory leaks can occur when holding threads are re-used which is the case on application servers using pool of threads.

To avoid such problems, it is recommended to always clean up `ThreadLocal` variables using the `remove()` method to remove the current thread’s value for the `ThreadLocal` variable.

In addition, calling `set(null)` to remove the value might keep the reference to `this` pointer in the map, which can cause memory leak in some scenarios. Using `remove` is safer to avoid this issue.

## Noncompliant Code Example

```
public class ThreadLocalUserSession implements UserSession {

  private static final ThreadLocal<UserSession> DELEGATE = new ThreadLocal<>();

  public UserSession get() {
    UserSession session = DELEGATE.get();
    if (session != null) {
      return session;
    }
    throw new UnauthorizedException("User is not authenticated");
  }

  public void set(UserSession session) {
    DELEGATE.set(session);
  }

   public void incorrectCleanup() {
     DELEGATE.set(null); // Noncompliant
   }

  // some other methods without a call to DELEGATE.remove()
}
```
## Compliant Solution

```
public class ThreadLocalUserSession implements UserSession {

  private static final ThreadLocal<UserSession> DELEGATE = new ThreadLocal<>();

  public UserSession get() {
    UserSession session = DELEGATE.get();
    if (session != null) {
      return session;
    }
    throw new UnauthorizedException("User is not authenticated");
  }

  public void set(UserSession session) {
    DELEGATE.set(session);
  }

  public void unload() {
    DELEGATE.remove(); // Compliant
  }

  // ...
}
```

## Exceptions

Rule will not detect non-private `ThreadLocal` variables, because `remove()` can be called from another class.

## See

*   [Understanding Memory Leaks in Java](https://www.baeldung.com/java-memory-leaks)

# Overrides should match their parent class methods in synchronization

When `@Overrides` of `synchronized` methods are not themselves `synchronized`, the result can be improper synchronization as callers rely on the thread-safety promised by the parent class.

## Noncompliant Code Example

```
public class Parent {

  synchronized void foo() {
    //...
  }
}

public class Child extends Parent {

 @Override
  public foo () {  // Noncompliant
    // ...
    super.foo();
  }
}
```
## Compliant Solution
```
public class Parent {

  synchronized void foo() {
    //...
  }
}

public class Child extends Parent {

  @Override
  synchronized foo () {
    // ...
    super.foo();
  }
}
```
## See

*   [CERT, TSM00-J](https://www.securecoding.cert.org/confluence/x/XgAZAg) - Do not override thread-safe methods with methods that are not thread-safe

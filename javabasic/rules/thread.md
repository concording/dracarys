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

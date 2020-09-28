# "getClass" should not be used for synchronization

`getClass` should not be used for synchronization in non-`final` classes because child classes will synchronize on a different object than the parent or each other, allowing multiple threads into the code block at once, despite the `synchronized` keyword.

Instead, hard code the name of the class on which to synchronize or make the class `final`.

## Noncompliant Code Example
```
public class MyClass {
  public void doSomethingSynchronized(){
    synchronized (this.getClass()) {  // Noncompliant
      // ...
    }
  }
```
## Compliant Solution
```
public class MyClass {
  public void doSomethingSynchronized(){
    synchronized (MyClass.class) {
      // ...
    }
  }
```
## See

*   [CERT, LCK02-J.](https://www.securecoding.cert.org/confluence/x/bwCaAg) - Do not synchronize on the class object returned by getClass()

# "runFinalizersOnExit" should not be called

Running finalizers on JVM exit is disabled by default. It can be enabled with `System.runFinalizersOnExit` and `Runtime.runFinalizersOnExit`, but both methods are deprecated because they are are inherently unsafe.

According to the Oracle Javadoc:

> It may result in finalizers being called on live objects while other threads are concurrently manipulating those objects, resulting in erratic behavior or deadlock.

If you really want to be execute something when the virtual machine begins its shutdown sequence, you should attach a shutdown hook.

## Noncompliant Code Example
```
public static void main(String [] args) {
  ...
  System.runFinalizersOnExit(true);  // Noncompliant
  ...
}

protected void finalize(){
  doSomething();
}
```
## Compliant Solution
```
public static void main(String [] args) {
  Runtime.addShutdownHook(new Runnable() {
    public void run(){
      doSomething();
    }
  });
  //...
```
## See

*   [CERT, MET12-J.](https://www.securecoding.cert.org/confluence/x/H4cbAQ) - Do not use finalizers

# Non-thread-safe fields should not be static

Not all classes in the standard Java library were written to be thread-safe. Using them in a multi-threaded manner is highly likely to cause data problems or exceptions at runtime.

This rule raises an issue when an instance of `Calendar`, `DateFormat`, `javax.xml.xpath.XPath`, or `javax.xml.validation.SchemaFactory` is marked `static`.

## Noncompliant Code Example
```
public class MyClass {
  private static SimpleDateFormat format = new SimpleDateFormat("HH-mm-ss");  // Noncompliant
  private static Calendar calendar = Calendar.getInstance();  // Noncompliant
```
## Compliant Solution
```
public class MyClass {
  private SimpleDateFormat format = new SimpleDateFormat("HH-mm-ss");
  private Calendar calendar = Calendar.getInstance();
```

# "readObject" should not be "synchronized"

A `readObject` method is written when a `Serializable` object needs special handling to be rehydrated from file. It should be the case that the object being created by `readObject` is only visible to the thread that invoked the method, and the `synchronized` keyword is not needed, and using `synchronized` anyway is just confusing. If this is not the case, the method should be refactored to make it the case.

## Noncompliant Code Example
```
private synchronized void readObject(java.io.ObjectInputStream in)
     throws IOException, ClassNotFoundException { // Noncompliant
  //...
}
```
## Compliant Solution
```
private void readObject(java.io.ObjectInputStream in)
     throws IOException, ClassNotFoundException { // Compliant
  //...
}
```
# ".equals()" should not be used to test the values of "Atomic" classes

`AtomicInteger`, and `AtomicLong` extend `Number`, but they're distinct from `Integer` and `Long` and should be handled differently. `AtomicInteger` and `AtomicLong` are designed to support lock-free, thread-safe programming on single variables. As such, an `AtomicInteger` will only ever be "equal" to itself. Instead, you should `.get()` the value and make comparisons on it.

This applies to all the atomic, seeming-primitive wrapper classes: `AtomicInteger`, `AtomicLong`, and `AtomicBoolean`.

## Noncompliant Code Example
```
AtomicInteger aInt1 = new AtomicInteger(0);
AtomicInteger aInt2 = new AtomicInteger(0);

if (aInt1.equals(aInt2)) { ... }  // Noncompliant
```
## Compliant Solution
```
AtomicInteger aInt1 = new AtomicInteger(0);
AtomicInteger aInt2 = new AtomicInteger(0);

if (aInt1.get() == aInt2.get()) { ... }
```

# "wait", "notify" and "notifyAll" should only be called when a lock is obviously held on an object

By contract, the method `Object.wait(...)`, `Object.notify()` and `Object.notifyAll()` should be called by a thread that is the owner of the object's monitor. If this is not the case an `IllegalMonitorStateException` exception is thrown. This rule reinforces this constraint by making it mandatory to call one of these methods only inside a `synchronized` method or statement.

## Noncompliant Code Example
```
private void removeElement() {
  while (!suitableCondition()){
    obj.wait();
  }
  ... // Perform removal
}
```
or
```
private void removeElement() {
  while (!suitableCondition()){
    wait();
  }
  ... // Perform removal
}
```
## Compliant Solution
```
private void removeElement() {
  synchronized(obj) {
    while (!suitableCondition()){
      obj.wait();
    }
    ... // Perform removal
  }
}
```
or
```
private synchronized void removeElement() {
  while (!suitableCondition()){
    wait();
  }
  ... // Perform removal
}
```

# Synchronization should not be based on Strings or boxed primitives

Objects which are pooled and potentially reused should not be used for synchronization. If they are, it can cause unrelated threads to deadlock with unhelpful stacktraces. Specifically, `String` literals, and boxed primitives such as Integers should not be used as lock objects because they are pooled and reused. The story is even worse for `Boolean` objects, because there are only two instances of `Boolean`, `Boolean.TRUE` and `Boolean.FALSE` and every class that uses a Boolean will be referring to one of the two.

## Noncompliant Code Example

```
private static final Boolean bLock = Boolean.FALSE;
private static final Integer iLock = Integer.valueOf(0);
private static final String sLock = "LOCK";

public void doSomething() {

  synchronized(bLock) {  // Noncompliant
    // ...
  }
  synchronized(iLock) {  // Noncompliant
    // ...
  }
  synchronized(sLock) {  // Noncompliant
    // ...
  }
```
## Compliant Solution

```
private static final Object lock1 = new Object();
private static final Object lock2 = new Object();
private static final Object lock3 = new Object();

public void doSomething() {

  synchronized(lock1) {
    // ...
  }
  synchronized(lock2) {
    // ...
  }
  synchronized(lock3) {
    // ...
  }
```
## See

*   [CERT, LCK01-J.](https://www.securecoding.cert.org/confluence/x/rQGeAQ) - Do not synchronize on objects that may be reused

# "writeObject" should not be the only "synchronized" code in a class

The purpose of synchronization is to ensure that only one thread executes a given block of code at a time. There's no real problem with marking `writeObject` `synchronized`, but it's highly suspicious if this serialization-related method is the only `synchronized` code in a `class`.

## Noncompliant Code Example
```
public class RubberBall {

  private Color color;
  private int diameter;

  public RubberBall(Color color, int diameter) {
    // ...
  }

  public void bounce(float angle, float velocity) {
    // ...
  }

  private synchronized void writeObject(ObjectOutputStream stream) throws IOException { // Noncompliant
    // ...
  }
}
```
## Compliant Solution
```
public class RubberBall {

  private Color color;
  private int diameter;

   public RubberBall(Color color, int diameter) {
    // ...
  }

  public void bounce(float angle, float velocity) {
    // ...
  }

  private void writeObject(ObjectOutputStream stream) throws IOException {
    // ...
  }
}
```
# Double-checked locking should not be used

Double-checked locking is the practice of checking a lazy-initialized object's state both before and after a `synchronized` block is entered to determine whether or not to initialize the object.

It does not work reliably in a platform-independent manner without additional synchronization for mutable instances of anything other than `float` or `int`. Using double-checked locking for the lazy initialization of any other type of primitive or mutable object risks a second thread using an uninitialized or partially initialized member while the first thread is still creating it, and crashing the program.

There are multiple ways to fix this. The simplest one is to simply not use double checked locking at all, and synchronize the whole method instead. With early versions of the JVM, synchronizing the whole method was generally advised against for performance reasons. But `synchronized` performance has improved a lot in newer JVMs, so this is now a preferred solution. If you prefer to avoid using `synchronized` altogether, you can use an inner `static class` to hold the reference instead. Inner static classes are guaranteed to load lazily.

## Noncompliant Code Example
```
@NotThreadSafe
public class DoubleCheckedLocking {
    private static Resource resource;

    public static Resource getInstance() {
        if (resource == null) {
            synchronized (DoubleCheckedLocking.class) {
                if (resource == null)
                    resource = new Resource();
            }
        }
        return resource;
    }

    static class Resource {

    }
}
```
## Compliant Solution
```
@ThreadSafe
public class SafeLazyInitialization {
    private static Resource resource;

    public synchronized static Resource getInstance() {
        if (resource == null)
            resource = new Resource();
        return resource;
    }

    static class Resource {
    }
}
```
With inner static holder:
```
@ThreadSafe
public class ResourceFactory {
    private static class ResourceHolder {
        public static Resource resource = new Resource(); // This will be lazily initialised
    }

    public static Resource getResource() {
        return ResourceFactory.ResourceHolder.resource;
    }

    static class Resource {
    }
}
```
Using "volatile":
```
class ResourceFactory {
  private volatile Resource resource;

  public Resource getResource() {
    Resource localResource = resource;
    if (localResource == null) {
      synchronized (this) {
        localResource = resource;
        if (localResource == null) {
          resource = localResource = new Resource();
        }
      }
    }
    return localResource;
  }

  static class Resource {
  }
}
```
## See

*   [The "Double-Checked Locking is Broken" Declaration](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html)
*   [CERT, LCK10-J.](https://www.securecoding.cert.org/confluence/x/IgAZAg) - Use a correct form of the double-checked locking idiom
*   [MITRE, CWE-609](http://cwe.mitre.org/data/definitions/609.html) - Double-checked locking
*   [JLS 12.4](https://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.4) - Initialization of Classes and Interfaces
*   Wikipedia: [Double-checked locking](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java)

# "Object.wait(...)" and "Condition.await(...)" should be called inside a "while" loop

According to the documentation of the Java `Condition` interface:

> When waiting upon a `Condition`, a "spurious wakeup" is permitted to occur, in general, as a concession to the underlying platform semantics. This has little practical impact on most application programs as a Condition should always be waited upon in a loop, testing the state predicate that is being waited for. An implementation is free to remove the possibility of spurious wakeups but it is recommended that applications programmers always assume that they can occur and so always wait in a loop.

The same advice is also found for the `Object.wait(...)` method:

> waits should always occur in loops, like this one:
```
synchronized (obj) {
   while (<condition does not hold>){
     obj.wait(timeout);
   }
    ... // Perform action appropriate to condition
 }
```

## Noncompliant Code Example
```
synchronized (obj) {
  if (!suitableCondition()){
    obj.wait(timeout);   //the thread can wake up even if the condition is still false
  }
   ... // Perform action appropriate to condition
}
```
## Compliant Solution
```
synchronized (obj) {
  while (!suitableCondition()){
    obj.wait(timeout);
  }
   ... // Perform action appropriate to condition
}
```
## See

*   [CERT THI03-J.](https://www.securecoding.cert.org/confluence/x/9QIhAQ) - Always invoke wait() and await() methods inside a loop

# "this" should not be exposed from constructors

In single-threaded environments, the use of `this` in constructors is normal, and expected. But in multi-threaded environments, it could expose partially-constructed objects to other threads, and should be used with caution.

The classic example is a class with a `static` list of its instances. If the constructor stores `this` in the list, another thread could access the object before it's fully-formed. Even when the storage of `this` is the last instruction in the constructor, there's still a danger if the class is not `final`. In that case, the initialization of subclasses won't be complete before `this` is exposed.

This rule raises an issue when `this` is assigned to any globally-visible object in a constructor, and when it is passed to the method of another object in a constructor

## Noncompliant Code Example
```
public class Monument {

  public static final List<Monument> ALL_MONUMENTS = new ArrayList()<>;
  // ...

  public Monument(String location, ...) {
    ALL_MONUMENTS.add(this);  // Noncompliant; passed to a method of another object

    this.location = location;
    // ...
  }
}
```
## Exceptions

This rule ignores instances of assigning `this` directly to a `static` field of the same class because that case is covered by S3010.

## See

*   [CERT, TSM01-J.](https://www.securecoding.cert.org/confluence/x/aAD1AQ) - Do not let the this reference escape during object construction
*   [CERT, TSM03-J.](https://www.securecoding.cert.org/confluence/x/7ABQAg) - Do not publish partially initialized objects

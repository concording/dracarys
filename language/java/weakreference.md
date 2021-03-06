# Weak References in Java

## **1\. Overview**[](https://www.baeldung.com/java-weak-reference#overview)

In this article, we'll have a look at the concept of a weak reference – in the Java language.

We're going to explain what these are, what they're used for, and how to work with them properly.

## **2\. Weak References**[](https://www.baeldung.com/java-weak-reference#references-what)

**A weakly referenced object is cleared by the Garbage Collector when it's weakly reachable.**

Weak reachability means that an **object has neither strong nor [soft](https://www.baeldung.com/java-soft-references) references pointing to it**. The object can be reached only by traversing a weak reference.

First off, the Garbage Collector clears a weak reference, so the referent is no longer accessible. Then the reference is placed in a reference queue (if any associated exists) where we can obtain it from.

At the same time, formerly weakly-reachable objects are going to be finalized.

### 2.1\. Weak vs Soft References[](https://www.baeldung.com/java-weak-reference#1-weak-vs-soft-references)

Sometimes the difference between weak and soft references is unclear. Soft references are basically a big LRU cache. That is, **we use soft references when the referent has a good chance of being reused in the near future**.

Since a soft reference acts as a cache, it may continue to be reachable even if the referent itself is not. As a matter of fact, a soft reference is eligible for collection if and only if:

*   The referent is not strongly reachable
*   The soft reference is not being accessed recently

So a soft reference may be available for minutes or even hours after the referent becomes unreachable. On the other hand, a weak reference will be available only for as long as its referent is still around.

## **3. ****Use Cases**

As stated by Java documentation,** weak references are most often used to implement canonicalizing mappings**. A mapping is called canonicalized if it holds only one instance of a particular value. Rather than creating a new object, it looks up the existing one in the mapping and uses it.

Of course, the **most known use of these references is the [_WeakHashMap_](https://docs.oracle.com/javase/9/docs/api/java/util/WeakHashMap.html) class**. It's the implementation of the [_Map_](https://docs.oracle.com/javase/9/docs/api/java/util/Map.html) interface where every key is stored as a weak reference to the given key. When the Garbage Collector removes a key, the entity associated with this key is deleted as well.

For more information, check out [our guide to WeakHashMap](https://www.baeldung.com/java-weakhashmap).

Another area where they can be used is **the Lapsed Listener problem**.

A publisher (or a subject) holds strong references to all subscribers (or listeners) to notify them about events that happened. **The problem arises when a listener can't successfully unsubscribe from a publisher.**

Therefore, a listener can't be garbage collected since a strong reference to it's still available to a publisher. Consequently, memory leaks may happen.

The solution to the problem can be a subject holding a weak reference to an observer allowing the former to be garbage collected without the need to be unsubscribed (note that this isn't a complete solution, and it introduces some other issues which aren't covered here).

## **4\. Working With Weak References**[](https://www.baeldung.com/java-weak-reference#references-using)

Weak references are represented by the [_java.lang.ref.WeakReference_](https://docs.oracle.com/javase/8/docs/api/java/lang/ref/WeakReference.html) class. We can initialize it by passing a referent as a parameter. Optionally, we can provide a [_java.lang.ref.ReferenceQueue_](https://docs.oracle.com/javase/8/docs/api/java/lang/ref/ReferenceQueue.html):

```java
Object referent = new Object();
ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();

WeakReference weakReference1 = new WeakReference<>(referent);
WeakReference weakReference2 = new WeakReference<>(referent, referenceQueue);

```

The referent of a reference can be fetched by the [_get_](https://docs.oracle.com/javase/8/docs/api/java/lang/ref/Reference.html#get--) method, and removed manually using the [_clear_](https://docs.oracle.com/javase/8/docs/api/java/lang/ref/Reference.html#clear--) method:

```java
Object referent2 = weakReference1.get();
weakReference1.clear();
```
The pattern for safe working with this kind of references is the same as with [soft references](https://www.baeldung.com/java-soft-references):

```java
Object referent3 = weakReference2.get();
if (referent3 != null) {
    // GC hasn't removed the instance yet
} else {
    // GC has cleared the instance
}
```
## **5\. Conclusion**[](https://www.baeldung.com/java-weak-reference#conclusion)

In this quick tutorial, we had a look at the low-level concept of a weak reference in Java – and focused on the most common scenarios to use these.





Java provides two different types/classes of _Reference Objects_: **strong** and **weak**. Weak Reference Objects can be further divided into _soft_ and _phantom_.

*   Strong
*   Weak
    *   soft
    *   phantom

Let's go point by point.

**Strong Reference Object**

```
StringBuilder builder = new StringBuilder();

```

This is the default type/class of Reference Object, if not differently specified: `builder` is a strong Reference Object. This kind of reference makes the referenced object not eligible for GC. That is, whenever an object is referenced by a _chain of strong Reference Objects_, it cannot be garbage collected.

**Weak Reference Object**

```
WeakReference<StringBuilder> weakBuilder = new WeakReference<StringBuilder>(builder);

```

Weak Reference Objects are not the default type/class of Reference Object and to be used they should be explicitly specified like in the above example. This kind of reference makes the reference object eligible for GC. That is, in case the only reference reachable for the `StringBuilder` object in memory is, actually, the weak reference, then the GC is allowed to garbage collect the `StringBuilder` object. When an object in memory is reachable only by Weak Reference Objects, it becomes automatically eligible for GC.

**Levels of Weakness**

Two different levels of weakness can be enlisted: _soft_ and _phantom_.

A _soft_ Reference Object is basically a weak Reference Object that remains in memory a bit more: normally, it resists GC cycle until no memory is available and there is risk of `OutOfMemoryError` (in that case, it can be removed).

On the other hand, a _phantom_ Reference Object is useful only to know exactly when an object has been effectively removed from memory: normally they are used to fix _weird finalize() revival/resurrection behavior_, since they actually do not return the object itself but only help [in keeping track of their memory presence](https://community.oracle.com/blogs/enicholas/2006/05/04/understanding-weak-references).

Weak Reference Objects are ideal to implement cache modules. In fact, a sort of automatic eviction can be implemented by allowing the GC to clean up memory areas whenever objects/values are no longer reachable by strong references chain. An example is the [WeakHashMap](http://docs.oracle.com/javase/7/docs/api/java/util/WeakHashMap.html) retaining weak keys.

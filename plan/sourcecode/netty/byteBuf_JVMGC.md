According to the book `Netty in Action v10`, `reference counting` is used to handle the pooling of `ByteBuf`. But JVM is not aware of the netty reference counting, so JVM can still GC the `ByteBuf`. If so, why do we still need to care about the reference counting and manually call `release()` method?

I quoted some from the book < Netty in Action v10 > to add some context.

> One of the tradeoffs of reference-counting is that the user have to be carefully when consume **messages**. While the JVM will still be able to GC such a message (as it is not aware of the reference-counting) this message will not be put back in the pool from which it may be obtained before. Thus chances are good that you will run out of resources at one point if you do not carefully release these messages.

And some related threads: [Buffer ownership in Netty 4: How is buffer life-cycle managed?](https://stackoverflow.com/questions/15781276/buffer-ownership-in-netty-4-how-is-buffer-life-cycle-managed)

[https://blog.twitter.com/2013/netty-4-at-twitter-reduced-gc-overhead](https://blog.twitter.com/2013/netty-4-at-twitter-reduced-gc-overhead)

## ADD 1

(Below is some more of my understanding.)

A `ByteBuf` can be categorized from 2 perspectives:

```
1\. Pooled or Unpooled
2\. Heap-based or Direct

```

So there can be 4 combinations:

```
(a) Pooled Heap-based
(b) Pooled Direct
(c) Unpooled Heap-based
(d) Unpooled Direct

```

Only (a) and (c) are affected by JVM GC mechanism because they are heap-based.

In the above quotation from < Netty in Action v10 >, I think the **message** means a Java object, which is in (a) category.

One ultimate rule is, if a Java object is GCed, it's totally gone. So below is what I think Netty does:

*   For (a), Netty allocator MUST **trick JVM GC into believing the object should never be GCed**. And then use ref counting to move the object out of/back into the pool. _This is another form of life cycle_.

*   For (b), JVM GC is not involved as it is not JVM Heap-based. And Netty allocator need to use ref counting to move the object out of/back into the pool.

*   For (c), JVM GC takes full responsibility to control the life of object. Netty allocator just provide API for allocating object.

*   For (d), JVM GC is not involved. And no pooling is needed. So Netty allocator only needs provide API for allocating/releasing the object.




## Answer
Direct buffers are indirectly freed by the Garbage Collector. I will let you read through the answer to this question to understand how that happens: [Are Java DirectByteBuffer wrappers garbage collected?](https://stackoverflow.com/questions/6697709/are-java-directbytebuffer-wrappers-garbage-collected)

Heap buffers need to be copied to the direct memory before being handled by the kernel, when you're performing I/O operations. When you use direct buffers you save that copy operation and that's the main advantage of using direct buffers. A drawback is that direct memory allocation is reasonably more expensive then allocations from the java heap, so Netty introduced the pooling concept.

Pooling objects in Java is a [polemic topic](http://en.wikipedia.org/wiki/Object_pool_pattern#Criticism), but the Netty choice for doing so seems to have paid off and the [Twitter article](https://blog.twitter.com/2013/netty-4-at-twitter-reduced-gc-overhead) you cited shows some evidence of that. For the particular case of allocating buffers, when the size of the buffer is large, you can see that it really brings a benefit both in the direct and heap buffer cases.

Now for pooling, the GC doesn't reclaim the buffer when they are pooled, because either your application has one or several references to it, while you're using the buffer; or Netty's pool has a reference to it, when it has just been allocated and has not yet been given to your application or after your application used it and gave it back to the pool.

Leaks will happen when your application, after having used a buffer and not keeping any further reference to it, doesn't call `release()`, what actually means _put it back into the pool_, if you don't have any further reference to it. In such case, the buffer will eventually be garbage collected, but Netty's pool won't know about it. The pool will then grow believing that you're using more and more buffers that are never returned to the pool. That will probably generate a memory leak because, even if the buffers themselves are garbage collected, the internal data structures used to store the pool will not.


## DirectByteBuffer
In the Sun JDK, a `java.nio.DirectByteBuffer`—created by [`ByteBuffer#allocateDirect(int)`](http://download.oracle.com/javase/6/docs/api/java/nio/ByteBuffer.html#allocateDirect%28int%29)—has a field of type `sun.misc.Cleaner`, which extends [`java.lang.ref.PhantomReference`](http://download.oracle.com/javase/6/docs/api/java/lang/ref/PhantomReference.html).

When this `Cleaner` (remember, a subtype of `PhantomReference`) gets collected and is about to move into the associated `ReferenceQueue`, the collection-related thread running through the nested type `ReferenceHandler` has a special case treatment of `Cleaner` instances: it downcasts and calls on `Cleaner#clean()`, which eventually makes its way back to calling on `DirectByteBuffer$Deallocator#run()`, which in turn calls on `Unsafe#freeMemory(long)`. Wow.

It's rather circuitous, and I was surprised to not see any use of `Object#finalize()` in play. The Sun developers must have had their reasons for tying this in even closer to the collection and reference management subsystem.

In short, you won't run out of memory by virtue of abandoning references to `DirectByteBuffer` instances, so long as the garbage collector has a chance to notice the abandonment and its reference handling thread makes progress through the calls described above.

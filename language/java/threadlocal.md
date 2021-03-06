# [ThreadLocal & Memory Leak](https://stackoverflow.com/questions/17968803/threadlocal-memory-leak)

It is mentioned at multiple posts: improper use of `ThreadLocal` causes Memory Leak. I am struggling to understand how Memory Leak would happen using `ThreadLocal`.

The only scenario I have figured out it as below:

> A web-server maintains a pool of Threads (e.g. for servlets). Those threads can create memory leak if the variables in `ThreadLocal` are not removed because Threads do not die.

This scenario does not mention "Perm Space" memory leak. Is that the only (major) use case of memory leak?



## answers


**PermGen exhaustions** in combination with `ThreadLocal` are often caused by **classloader leaks**.

_An example:_
Imagine an application server which has a pool of _worker threads_.
They will be kept alive until application server termination.
A deployed web application uses a **static** `ThreadLocal` in one of its classes in order to store some thread-local data, an instance of another class (lets call it `SomeClass`) of the web application. This is done within the worker thread (e.g. this action originates from a _HTTP request_).

**Important:**
[By definition](http://docs.oracle.com/javase/6/docs/api/java/lang/ThreadLocal.html), a reference to a `ThreadLocal` _value_ is kept until the "owning" thread dies or if the ThreadLocal itself is no longer reachable.

If the web application **fails to clear the reference** to the `ThreadLocal` **on shutdown**, bad things will happen:
Because the worker thread will usually never die and the reference to the `ThreadLocal` is static, the `ThreadLocal` value **still references** the instance of `SomeClass`, a web application's class - **even if the web application has been stopped!**

As a consequence, the web application's **classloader cannot be garbage collected**, which means that **all classes** (and all static data) of the web application **remain loaded** (this affects the PermGen memory pool as well as the heap).
Every redeployment iteration of the web application will increase permgen (and heap) usage.

**=> This is the permgen leak**

One popular example of this kind of leak is [this bug](https://issues.apache.org/bugzilla/show_bug.cgi?id=50486) in log4j (fixed in the meanwhile).

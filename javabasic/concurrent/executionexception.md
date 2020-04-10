 # [What is the best way to handle an ExecutionException?](https://stackoverflow.com/questions/10437890/what-is-the-best-way-to-handle-an-executionexception)

```
public static byte[] doSomethingWithTimeout( int timeout ) throws ProcessExecutionException, InterruptedException, IOException, TimeoutException {

    Callable<byte[]> callable = new Callable<byte[]>() {
        public byte[] call() throws IOException, InterruptedException, ProcessExecutionException {
            //Do some work that could throw one of these exceptions
            return null;
        }
    };

    try {
        ExecutorService service = Executors.newSingleThreadExecutor();
        try {
            Future<byte[]> future = service.submit( callable );
            return future.get( timeout, TimeUnit.MILLISECONDS );
        } finally {
            service.shutdown();
        }
    } catch( Throwable t ) { //Exception handling of nested exceptions is painfully clumsy in Java
        if( t instanceof ExecutionException ) {
            t = t.getCause();
        }
        if( t instanceof ProcessExecutionException ) {
            throw (ProcessExecutionException)t;
        } else if( t instanceof InterruptedException ) {
            throw (InterruptedException)t;
        } else if( t instanceof IOException ) {
            throw (IOException)t;
        } else if( t instanceof TimeoutException ) {
            throw (TimeoutException)t;
        } else if( t instanceof Error ) {
            throw (Error)t;
        } else if( t instanceof RuntimeException) {
            throw (RuntimeException)t;
        } else {
            throw new RuntimeException( t );
        }
    }
}
```


## Answer

I've looked at this problem in depth, and it's a mess. There is no easy answer in Java 5, nor in 6 or 7\. In addition to the clumsiness, verbosity and fragility that you point out, your solution actually has the problem that the `ExecutionException` that you are stripping off when you call `getCause()` actually contains most of the important stack trace information!

That is, all the stack information of the thread executing the method in the code you presented is only in the ExcecutionException, and not in the nested causes, which only cover frames starting at `call()` in the Callable. That is, your `doSomethingWithTimeout` method won't even appear in the stack traces of the exceptions you are throwing here! You'll only get the disembodied stack from the executor. This is because the `ExecutionException` is the only one that was created on the calling thread (see `FutureTask.get()`).

The only solution I know is complicated. A lot of the problem originates with the liberal exception specification of `Callable` - `throws Exception`. You can define new variants of `Callable` which specify exactly which exceptions they throw, such as:

```
public interface Callable1<T,X extends Exception> extends Callable<T> {

    @Override
    T call() throws X; 
}
```

This allows methods which executes callables to have a more precise `throws` clause. If you want to support signatures with up to N exceptions, you'll need N variants of this interface, unfortunately.

Now you can write a wrapper around the JDK `Executor` which takes the enhanced Callable, and returns an enhanced `Future`, something like guava's [CheckedFuture](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/util/concurrent/CheckedFuture.html). The checked exception type(s) are propagated at compile time from the creation and type of the `ExecutorService`, to the returned `Future`s, and end up on the `getChecked` method on the future.

That's how you thread the compile-time type safety through. This means that rather than calling:

```
Future.get() throws InterruptedException, ExecutionException;
```

You can call:

```
CheckedFuture.getChecked() throws InterruptedException, ProcessExecutionException, IOException
```

So the unwrapping problem is avoided - your method immediately throws the exceptions of the required type and they are available and checked at compile time.

Inside `getChecked`, however you _still_ need to solve the "missing cause" unwrapping problem described above. You can do this by stitching the current stack (of the calling thread) onto the stack of the thrown exception. This a stretching the usual use of a stack trace in Java, since a single stack stretches across threads, but it works and is easy to understand once you know what is going on.

Another option is to create _another_ exception of the same thing as the one being thrown, and set the original as the cause of the new one. You'll get the full stack trace, and the cause relationship will be the same as how it works with `ExecutionException` - but you'll have the right type of exception. You'll need to use reflection, however, and is not guaranteed to work, e.g., for objects with no constructor having the usual parameters.


# note
`FutureTask.get()`只会抛出`ExecutionException`


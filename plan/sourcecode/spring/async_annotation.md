## **1\. Overview**

In this article, we’ll explore the **asynchronous execution support in Spring** – and the _@Async_ annotation.

Simply put – annotating a method of a bean with _@Async_ will make it **execute in a separate thread** i.e. the caller will not wait for the completion of the called method.

One interesting aspect in Spring is that the event support in the framework [**also has support for async processing**](https://www.baeldung.com/spring-events) if you want to go that route.

## **2\. Enable Async Support**

Let’s start by **enabling asynchronous processing** with **Java configuration** – by simply adding the _@EnableAsync_ to a configuration class:

```
@Configuration
@EnableAsync
public class SpringAsyncConfig { ... }
```

The enable annotation is enough, but as you’d expect, there are also a few simple options for configuration as well:

*   _**annotation** – b_y default, _@EnableAsync_ detects Spring’s _@Async_ annotation and the EJB 3.1 _javax.ejb.Asynchronous_; this option can be used to detect other, user-defined annotation types as well
*   _**mode** _– indicates the type of _advice_ that should be used – JDK proxy-based or AspectJ weaving
*   _**proxyTargetClass **_– indicates the type of _proxy _that should be used – CGLIB or JDK; this attribute has effect only if the **_mode_** is set to _AdviceMode.PROXY_
*   _**order**_ – sets the order in which _AsyncAnnotationBeanPostProcessor_ should be applied; by default, it runs last, just so that it can take into account all existing proxies

Asynchronous processing can also be enabled using **XML configuration** – by using the _task_ namespace:

```
<task:executor id="myexecutor" pool-size="5"  />
<task:annotation-driven executor="myexecutor"/>
```
## **3\. The _@Async_ Annotation**

First – let’s go over the rules – _@Async_ has two limitations:

*   it must be applied to _public_ methods only
*   self-invocation – calling the async method from within the same class – won’t work

The reasons are simple – **the method needs to be _public_ **so that it can be proxied. And **self-invocation doesn’t work** because it bypasses the proxy and calls the underlying method directly.

### **3.1\. Methods with void Return Type**

Following is the simple way to configure a method with void return type to run asynchronously:

```
@Async
public void asyncMethodWithVoidReturnType() {
    System.out.println("Execute method asynchronously. "
      + Thread.currentThread().getName());
}
```

### **3.2\. Methods With Return Type** 

_@Async_ can also be applied to a method with return type – by wrapping the actual return in the Future:

```
@Async
public Future<String> asyncMethodWithReturnType() {
    System.out.println("Execute method asynchronously - "
      + Thread.currentThread().getName());
    try {
        Thread.sleep(5000);
        return new AsyncResult<String>("hello world !!!!");
    } catch (InterruptedException e) {
        //
    }
 
    return null;
}
```
Spring also provides an _AsyncResult_ class which implements _Future_. This can be used to track the result of asynchronous method execution.

Now, let’s invoke the above method and retrieve the result of the asynchronous process using the _Future_object.

```
public void testAsyncAnnotationForMethodsWithReturnType()
  throws InterruptedException, ExecutionException {
    System.out.println("Invoking an asynchronous method. "
      + Thread.currentThread().getName());
    Future<String> future = asyncAnnotationExample.asyncMethodWithReturnType();
 
    while (true) {
        if (future.isDone()) {
            System.out.println("Result from asynchronous process - " + future.get());
            break;
        }
        System.out.println("Continue doing something else. ");
        Thread.sleep(1000);
    }
}
```

## **4\. The Executor**

By default, Spring uses a _SimpleAsyncTaskExecutor_ to actually run these methods asynchronously. The defaults can be overridden at two levels – at the application level or at the individual method level.

### **4.1\. Override the Executor at the Method Level**

The required executor needs to be declared in a configuration class:
```
@Configuration
@EnableAsync
public class SpringAsyncConfig {
     
    @Bean(name = "threadPoolTaskExecutor")
    public Executor threadPoolTaskExecutor() {
        return new ThreadPoolTaskExecutor();
    }
}
```

Then the executor name should be provided as an attribute in _@Async_:

```
@Async("threadPoolTaskExecutor")
public void asyncMethodWithConfiguredExecutor() {
    System.out.println("Execute method with configured executor - "
      + Thread.currentThread().getName());
}

```

### **4.2\. Override the Executor at the Application Level**

The configuration class should implement the _AsyncConfigurer_ interface – which will mean that it has the implement the _getAsyncExecutor()_ method. It’s here that we will return the executor for the entire application – this now becomes the default executor to run methods annotated with _@Async_:

```
@Configuration
@EnableAsync
public class SpringAsyncConfig implements AsyncConfigurer {
     
    @Override
    public Executor getAsyncExecutor() {
        return new ThreadPoolTaskExecutor();
    }
     
}
```


## **5\. Exception Handling**

When a method return type is a _Future_, exception handling is easy – _Future.get()_ method will throw the exception.

But, if the return type is _void_, **exceptions will not be propagated to the calling thread**. Hence we need to add extra configurations to handle exceptions.

We’ll create a custom async exception handler by implementing _AsyncUncaughtExceptionHandler_interface. The _handleUncaughtException() _method is invoked when there are any uncaught asynchronous exceptions:

```
public class CustomAsyncExceptionHandler
  implements AsyncUncaughtExceptionHandler {
 
    @Override
    public void handleUncaughtException(
      Throwable throwable, Method method, Object... obj) {
  
        System.out.println("Exception message - " + throwable.getMessage());
        System.out.println("Method name - " + method.getName());
        for (Object param : obj) {
            System.out.println("Parameter value - " + param);
        }
    }
     
}
```

In the previous section, we looked at the _AsyncConfigurer_ interface implemented by the configuration class. As part of that, we also need to override the _getAsyncUncaughtExceptionHandler()_ method to return our custom asynchronous exception handler:
```
@Override
public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
    return new CustomAsyncExceptionHandler();
}
```

## **6\. Conclusion**

In this tutorial, we looked at **running asynchronous code with Spring**. We started with the very basic configuration and annotation to make it work but also looked at more advanced configs such as providing our own executor, or exception handling strategies.

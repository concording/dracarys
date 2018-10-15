
**Understanding Thread Interruption in Java**
     
You may come across use cases where you would need to perform some long running tasks on separate threads. You may have to request these tasks to finish doing their work, may be even before the tasks are completely done, so that the threads they would be running on can be stopped.
Few examples where you may have to finish tasks early and stop threads are:
  
* While servicing a web request you may distribute the processing to multiple threads and some or all of the tasks fail to finish the processing within specified request processing time, or
  
* While shutting down an application that may use more than one thread to do some work, which may not have completed.
In this article I will try to answer the below questions:
  
* How to request a task, running on a separate thread, to finish early?
  
* How to make a task responsive to such a finish request?
Let’s try to answer the two questions by implementing an use case.

**Example Use Case**

The following are the requirements of the use case:
  
1. Make a task that prints 0 through 9 on console.
  
2. After printing a number the task should wait 1 sec before printing the next number.
  
3. The task runs on a separate thread, other than main application thread.
  
4. After starting the task the main application should wait for 3 sec and then shutdown.
  
5. On shutdown the application should request the running task to finish.
  
6. Before shutting down completely the application should, at the max, wait for 1 sec for the task to finish.
  
7. The task should respond to the finish request by stopping immediately.
As per the requirements mentioned in the use case the task will take minimum 9 seconds to complete. Therefore after 3 seconds the main application will have to request the task to finish and if implemented correctly the task will not be able to print all the ten numbers from 0 through 9.
  
In this article, I will be focusing on the implementations of requirement 5 and requirement 7.

**An implementation of the use case using Thread**

The inline comment against a line of code mentions the use case requirement that has been met by the line.
```
public static void main(String[] args) throws InterruptedException {
    Thread taskThread = new Thread(taskThatFinishesEarlyOnInterruption());
    taskThread.start();      // requirement 3
    Thread.sleep(3_000);     // requirement 4
    taskThread.interrupt();  // requirement 5
    taskThread.join(1_000);  // requirement 6
}

private static Runnable taskThatFinishesEarlyOnInterruption() {
    return () -> {
        for (int i = 0; i < 10; i++) {
            System.out.print(i);      // requirement 1
            try {
                Thread.sleep(1_000);  // requirement 2
            } catch (InterruptedException e) {
                break;                // requirement 7
            }
        }
    };
}
```
Understanding the implementation of Requirement 5

In the implementation of requirement 5, the main thread calls the taskThread’s interrupt() method. In Java, one thread cannot stop the other thread. A thread can only request the other thread to stop. The request is made in the form of an interruption. Calling the interrupt() method on an instance of a Thread sets the interrupt status state as true on the instance.
  
Use interruption to request a task, running on a separate thread, to finish.
Understanding the implementation of Requirement 7

In the implementation of requirement 7, InterruptedException is being handled by breaking out of the loop and thus finishing the task early.
Question arises that why did Thread.sleep() throw an InterruptedException? As soon as the taskThread was interrupted by the main thread, the Thread.sleep(1_000) responded to the interruption by throwing the exception. In fact almost all blocking methods respond to interruption by throwing InterruptedException. The decision of what to do in the case of interruption is left to the implementing code, which in this example is breaking out of the for loop as per the requirement in the use case.
Note: Calls to sleep() and join() methods in main() method are blocking and may also throw InterruptedException upon interruption. Handling of the exception here has been omitted for brevity.
  
Handle interruption request, which in most cases is done by handling InterruptedException, in the task to make it responsive to a finish request.

**An implementation of the use case using the Executor**

The same use case can be implemented using Executor framework provided by Java and can be found under the java.util.concurrent package. Usage of the Executor framework is preferred over Threads as it provides separation of task execution from the thread management. In the implementation below the task is submitted to ExecutorService, a sub-interface of Executor, using the submit() method. The service runs the task on the thread it holds. The service’s shutdownNow() method interrupts the currently running task and awaitTermination() method waits for the service to shutdown.

```
public static void main(String[] args) throws InterruptedException {
    ExecutorService executor = Executors.newSingleThreadExecutor();
    executor.submit(taskThatFinishesEarlyOnInterruption());  // requirement 3
    Thread.sleep(3_000);                                     // requirement 4
    executor.shutdownNow();                                  // requirement 5
    executor.awaitTermination(1, TimeUnit.SECONDS);          // requirement 6
}

// implementation of taskThatFinishesEarlyOnInterruption() remains the same
```
When using the Executor framework, you can interrupt a specific task without shutting down the ExecutorService. On submitting a task to the service an instance of Future<?> is returned by the service. You may call the cancel() method on that instance to interrupt the task. In situations when you service a web request by running parallel tasks, this method of cancelling tasks and not shutting down the service helps in re-using the service across multiple requests. In such situations you may want to shutdown the service only on shutdown of your web application. Calling the cancel() with true causes the task to be interrupted.

```
Future<?> submittedTask = executor.submit(someTask());
...
submittedTask.cancel(true) // if conditions to cancel the task have been met
```

The Executor framework is a complete asynchronous task execution framework. If you have not explored it yet, I request to you to read about it. It will be a great addition to your development toolbox.

**InterruptedException and interruption status**

Before I finish, I wanted to emphasize on an important detail about what happens to a thread’s interruption status when a blocking code responds to interruption by throwing InterruptedException. I had left out the detail till now to avoid confusion.
  
Before a blocking code throws an InterruptedException, it marks the interruption status as false. Thus, when handling of the InterruptedException is done, you should also preserve the interruption status by calling Thread.currentThread().interrupt().
Let’s see how this information applies to the example below. In the task that is submitted to the ExecutorService, the printNumbers() method is called twice. When the task is interrupted by a call to shutdownNow(), the first call to the method finishes early and then the execution reaches the second call. The interruption is called by the main thread only once. The interruption is communicated to the second execution of the printNumber() method by the call to Thread.currentThread().interrupt() during the first execution. Hence the second execution also finishes early just after printing the first number. Not preserving the interruption status would have caused the second execution of the method to run fully for 9 seconds.

```
public static void main(String[] args) throws InterruptedException {
    ExecutorService executor = Executors.newSingleThreadExecutor();
    Future<?> future = executor.submit(() -> {
        printNumbers(); // first call
        printNumbers(); // second call
    });
    Thread.sleep(3_000);                                     
    executor.shutdownNow();  // will interrupt the task
    executor.awaitTermination(3, TimeUnit.SECONDS);
}

private static void printNumbers() {
    for (int i = 0; i < 10; i++) {
        System.out.print(i);
        try {
            Thread.sleep(1_000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt(); // preserve interruption status
            break;
        }
    }
}
```

Summary

The answers to the two questions that I had set out to answer are:
  
* How to request a task, running on a separate thread, to finish early? - Use interruption.    
      
    * If using Thread directly in your code, you may call interrupt() on the instance of thread.
      
    * If using Executor framework, you may cancel each task by calling cancel() on Future
      
    * If using Executor framework, you may shutdown the ExecutorService by calling the shutdownNow() method.
    
    
* How to make a task responsive to such a finish request? - Handle interruption request, which in most of the cases is done by handling InterruptedException. Also preserve the interruption status by calling Thread.currentThread().interrupt().

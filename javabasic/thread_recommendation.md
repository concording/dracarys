
The Object.wait() method temporarily cedes possession of a lock so that other threads that may be requesting the lock can proceed. Object.wait() must always be called from a synchronized block or method. The waiting thread resumes execution only after it has been notified, generally as the result of the invocation of the notify() or notifyAll() method by some other thread. The wait() method must be invoked from a loop that checks whether a condition predicate holds. Note that a condition predicate is the negation of the condition expression in the loop. For example, the condition predicate for removing an element from a vector is !isEmpty(), whereas the condition expression for the while loop condition is isEmpty(). Following is the correct way to invoke the wait() method when the vector is empty.

```
private Vector vector;
// ...
 
public void consumeElement() throws InterruptedException {
  synchronized (vector) {
    while (vector.isEmpty()) {
      vector.wait();
    }
 
    // Resume when condition holds
  }
}
```
The notification mechanism notifies the waiting thread and allows it to check its condition predicate. The invocation of notify() or notifyAll() in another thread cannot precisely determine which waiting thread will be resumed. Condition predicate statements allow notified threads to determine whether they should resume upon receiving the notification. Condition predicates are also useful when a thread is required to block until a condition becomes true, for example, when waiting for data to arrive on an input stream before reading the data.

Both safety and liveness  are concerns when using the wait/notify mechanism. The safety property requires that all objects maintain consistent states in a multithreaded environment [Lea 2000]. The liveness property requires that every operation or method invocation execute to completion without interruption.

To guarantee liveness, programs must test the while loop condition before invoking the wait() method. This early test checks whether another thread has already satisfied the condition predicate and sent a notification. Invoking the wait() method after the notification has been sent results in indefinite blocking.

To guarantee safety, programs must test the while loop condition after returning from the wait() method. Although wait() is intended to block indefinitely until a notification is received, it still must be encased within a loop to prevent the following vulnerabilities [Bloch 2001]:

1. Thread in the middle: A third thread can acquire the lock on the shared object during the interval between a notification being sent and the receiving thread resuming execution. This third thread can change the state of the object, leaving it inconsistent. This is a time-of-check, time-of-use (TOCTOU) race condition.
2. Malicious notification: A random or malicious notification can be received when the condition predicate is false. Such a notification would cancel the wait() method.
3. Misdelivered notification: The order in which threads execute after receipt of a notifyAll() signal is unspecified. Consequently, an unrelated thread could start executing and discover that its condition predicate is satisfied. Consequently, it could resume execution despite being required to remain dormant.
4. Spurious wakeups: Certain Java Virtual Machine (JVM) implementations are vulnerable to spurious wakeups that result in waiting threads waking up even without a notification [API 2014].


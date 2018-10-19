## Abstract Queued Synchronizer Class

It provides a framework for implementing blocking locks and related synchronizers like semaphores,  [CountDownLatch](http://www.javarticles.com/2012/10/countdownlatch.html) etc. The basic algorithm for acquire is try acquire, if successful return else enqueue thread if it is not already queued and block the current thread. Similarly basic algorithm for release is try release, if successful, unblock the first thread in the queue else simply return. The threads will wait in a first-in-first-out (FIFO) wait queues.  The abstract methods tryAcquire() and tryRelease() will be implemented by subclasses based on their need.

## Acquire in exclusive mode

Generic algorithm to acquire in exclusive mode

[![AbstractQueuedSynchronizer (AQS)](http://4.bp.blogspot.com/-rJ4nDdj6v0o/UIIw86KQTBI/AAAAAAAAB6Y/mIwqh_XQX1c/s1600/AST_acquire.png "AbstractQueuedSynchronizer (AQS)")](http://4.bp.blogspot.com/-rJ4nDdj6v0o/UIIw86KQTBI/AAAAAAAAB6Y/mIwqh_XQX1c/s1600/AST_acquire.png)

In the above diagram ‘shouldParkAfterFailedAcquire?’ verifies whether the predecessor’s wait status is SIGNAL. If yes, it knows for sure that the predecessor thread will SIGNAL on its release so it will immediately block else it may retry to acquire the lock in case it is the first node in queue.

[![AbstractQueuedSynchronizer (AQS)](http://2.bp.blogspot.com/-BI-n1dZFPJQ/UIEyFpWlkhI/AAAAAAAAB5k/ovzOHn07q2U/s1600/AST_acquired_queued.png "AbstractQueuedSynchronizer (AQS)")](http://2.bp.blogspot.com/-BI-n1dZFPJQ/UIEyFpWlkhI/AAAAAAAAB5k/ovzOHn07q2U/s1600/AST_acquired_queued.png)

## Queue

If a thread fails to acquire the lock, it will be put in a queue. If the queue doesn’t exist yet, it will initialize it using a dummy header and then link itself to it. The ‘next’ of the head and ‘prev’ of the node will be linked. The new node also becomes the tail. The wait status of the header node will be set to SIGNAL so that as the owner thread releases the lock, it can signal the head node’s successor to acquire the lock. One more time the thread will try to acquire the lock to make sure it cannot acquire before it actually parks.

[![AbstractQueuedSynchronizer (AQS)](http://4.bp.blogspot.com/-0U1cmw24RPU/UIEwdjKmwxI/AAAAAAAAB5c/YCkSGRMaAfw/s1600/FirstNodeWaitSignal.png "AbstractQueuedSynchronizer (AQS)")](http://4.bp.blogspot.com/-0U1cmw24RPU/UIEwdjKmwxI/AAAAAAAAB5c/YCkSGRMaAfw/s1600/FirstNodeWaitSignal.png)

Thus the thread which failed to acquire the lock can be safely parked as long as its predecessor node’s waiting status is set to SIGNAL so it can retry to acquire the lock once the predecessor is released.
If the predecessor was cancelled, it will skip over all the canceled predecessors to reset its next and prev pointers to a waiting thread.

## ![AbstractQueuedSynchronizer (AQS)](http://4.bp.blogspot.com/-WY8tMAYuHao/UH1HrFXprHI/AAAAAAAABxk/ZiYZqaH_f0U/s400/NodeWithSomeWaitStatus.png "AbstractQueuedSynchronizer (AQS)")

## Release

[![AbstractQueuedSynchronizer (AQS)](http://1.bp.blogspot.com/-FbEv8bDUrvg/UIIz5A2eV0I/AAAAAAAAB7U/Tl_BJDTloMM/s1600/AST_release.png "AbstractQueuedSynchronizer (AQS)")](http://1.bp.blogspot.com/-FbEv8bDUrvg/UIIz5A2eV0I/AAAAAAAAB7U/Tl_BJDTloMM/s1600/AST_release.png)

The sub classes will implement ‘try Release’ based on their requirement. Once released, the header node’s successor node needs to be signaled so that it can re-try to acquire. If there is no head, it means there are are no threads in queue so there is no one to be signaled. If the head exists, it makes sure wait status is not zero. If it is zero, it means the successor node need not be signaled.

## Unpark successor node’s thread

Thread to unpark is held in successor, which is normally just the next node.

Case 1: If head’s waiting status < 0, clear the waiting status. If the successor node (P1) is not in canceled state, unpark the successor node’s thread so that it can retry to acquire.

[![AbstractQueuedSynchronizer (AQS)](http://2.bp.blogspot.com/-DpNONQNkK6M/UIKZ-pq0-6I/AAAAAAAAB9E/qhxTrd7n2Kw/s1600/unparkSuccessor.png "AbstractQueuedSynchronizer (AQS)")](http://2.bp.blogspot.com/-DpNONQNkK6M/UIKZ-pq0-6I/AAAAAAAAB9E/qhxTrd7n2Kw/s1600/unparkSuccessor.png)

Case 2: If successor node either cancelled or null, traverse backwards from tail to find the actual non-cancelled successor.

[![AbstractQueuedSynchronizer (AQS)](http://1.bp.blogspot.com/-GAw7_OHHJZU/UIKagR9H8_I/AAAAAAAAB9U/_JY_3mVqY1k/s1600/unparkSuccessor1.png "AbstractQueuedSynchronizer (AQS)")](http://1.bp.blogspot.com/-GAw7_OHHJZU/UIKagR9H8_I/AAAAAAAAB9U/_JY_3mVqY1k/s1600/unparkSuccessor1.png)

Once the unparked thread to acquire, its node becomes the new head. The old head will be delinked. If it fails to acquire, it will be re-parked. The head node’s wait status which was set to 0 will be reset to SIGNAL.

## Release shared

This is similar to exclusive release. It also ensures that a release propagates.

## Acquire in shared mode

This is similar to exclusive acquire. It also propagates the release to other waiting threads in queue which are waiting to acquire the shared lock. Once a lock is released, it unparks its successor node which in turn propagates release to its next node.

## Cancel

There may be a runtime exception while trying to acquire in which case the node in context will be canceled. If a node is canceled we must make sure that its successor node is properly linked to a valid predecessor node so the links may have to be adjusted. If its predecessor nodes are already in a canceled state, those nodes will be skipped to arrive to a proper predecessor node which has wait status <= 0.
If the node to be canceled is tail node itself, it will simply be removed. Its predecessor node will become the new tail. The ‘next’ link of the new tail will point to null.
If the wait status is < 0, it means the successor needs signal, try to set predecessor’s next-link so it will get one. If the predecessor is the head node itself then it will wake up its successor node.

Case 1: Node to be canceled is the tail node

[![AbstractQueuedSynchronizer (AQS)](http://1.bp.blogspot.com/-O9q2yuq8kzU/UIT345IoheI/AAAAAAAAB_0/v5A2a5goc_U/s1600/AST_cancel.png "AbstractQueuedSynchronizer (AQS)")](http://1.bp.blogspot.com/-O9q2yuq8kzU/UIT345IoheI/AAAAAAAAB_0/v5A2a5goc_U/s1600/AST_cancel.png)

Case 2: The canceled node’s predecessor is head which now will signal the next node of the canceled node to wake up.

[![AbstractQueuedSynchronizer (AQS)](http://1.bp.blogspot.com/-rie5gDcjQOw/UIT4DnPBTPI/AAAAAAAAB_8/N14b-MNIUJY/s400/AST_cancel_unpark.png "AbstractQueuedSynchronizer (AQS)")](http://1.bp.blogspot.com/-rie5gDcjQOw/UIT4DnPBTPI/AAAAAAAAB_8/N14b-MNIUJY/s1600/AST_cancel_unpark.png)


### Task Execution

***. Finding Exploitable Parallelism***

If you have a batch of computations to submit to an Executor and you want to retrieve their results as they become
available, you could retain the Future associated with each task and repeatedly poll for completion by calling get with a
timeout of zero. This is possible, but tedious. Fortunately there is a better way: a completion service.
CompletionService combines the functionality of an Executor and a BlockingQueue. You can submit Callable tasks
to it for execution and use the queueͲlike methods take and poll to retrieve completed results, packaged as Futures,
as they become available. ExecutorCompletionService implements CompletionService, delegating the computation
to an Executor.

```
public class Renderer {
        private final ExecutorService executor;
        Renderer(ExecutorService executor) { this.executor = executor; }
        void renderPage(CharSequence source) {
            final List<ImageInfo> info = scanForImageInfo(source);
            CompletionService<ImageData> completionService =
                    new ExecutorCompletionService<ImageData>(executor);
            for (final ImageInfo imageInfo : info)
                completionService.submit(new Callable<ImageData>() {
                    public ImageData call() {
                        return imageInfo.downloadImage();
                    }
                });
            renderText(source);
            try {
                for (int t = 0, n = info.size(); t < n; t++) {
                    Future<ImageData> f = completionService.take();
                    ImageData imageData = f.get();
                    renderImage(imageData);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } catch (ExecutionException e) {
                throw launderThrowable(e.getCause());
            }
        }
    }
```

***Placing Time Limits on Tasks***

Sometimes, if an activity does not complete within a certain amount of time, the result is no longer needed and the
activity can be abandoned. For example, a web application may fetch its advertisements from an external ad server, but
if the ad is not available within two seconds, it instead displays a default advertisement so that ad unavailability does
not undermine the site's responsiveness requirements. Similarly, a portal site may fetch data in parallel from multiple
data sources, but may be willing to wait only a certain amount of time for data to be available before rendering the page
without it.
The primary challenge in executing tasks within a time budget is making sure that you don't wait longer than the time
budget to get an answer or find out that one is not forthcoming. The timed version of Future.get supports this
requirement: it returns as soon as the result is ready, but throws TimeoutException if the result is not ready within the
timeout period.
A secondary problem when using timed tasks is to stop them when they run out of time, so they do not waste
computing resources by continuing to compute a result that will not be used. This can be accomplished by having the
task strictly manage its own time budget and abort if it runs out of time, or by cancelling the task if the timeout expires.
Again, Future can help; if a timed get completes with a TimeoutException, you can cancel the task through the
Future. 

```
Page renderPageWithAd() throws InterruptedException {
        long endNanos = System.nanoTime() + TIME_BUDGET;
        Future<Ad> f = exec.submit(new FetchAdTask());
        // Render the page while waiting for the ad
        Page page = renderPageBody();
        Ad ad;
        try {
            // Only wait for the remaining time budget
            long timeLeft = endNanos - System.nanoTime();
            ad = f.get(timeLeft, NANOSECONDS);
        } catch (ExecutionException e) {
            ad = DEFAULT_AD;
        } catch (TimeoutException e) {
            ad = DEFAULT_AD;
            f.cancel(true);
        }
        page.setAd(ad);
        return page;
    }
```

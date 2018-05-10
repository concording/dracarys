
**Task Execution**
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

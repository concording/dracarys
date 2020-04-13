
[相关源码解析blog](https://segmentfault.com/a/1190000007308934)

[eventloopgroup analysis](https://programmer.help/blogs/eventloopgroup-and-eventloop-source-analysis.html)


1.1, boss NioEventLoop loop task
Polling for Accept events.
Handles Accept IO events, establishes a connection with Client, generates NioSocketChannel, and registers NioSocketChannel with a Selector of a work NioEventLoop.
Processing tasks in the task queue.

1.2, work NioEventLoop Loop Loop Loop Task
Polling Read, Write events.
Handles IO events when NioSocketChannel readable and writable events occur.
Processing tasks in the task queue.

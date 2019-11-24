#### 摘要

在NodeJS里，JS的执行时单线程的，但是这并不是意味着Node只能支持单线程。libUV为Node提供了异步非阻塞I/O（Async non-blocking I/O）能力，这肯定要归功于Event loop机制。

下面我们将区分Blocking与Non-Blocking的区别

#### Blocking
If a thread is taking a long time to execute a callback (Event Loop) or a task (Worker), we call it "blocked"。当处于被阻塞状态，那么该进程将无法处理来自其他client的请求。多以通常将避免阻塞event loop和worker pool，基于以下两点考虑：

1. Performance: If you regularly perform heavyweight activity on either type of thread, the throughput (requests/second) of your server will suffer.
2. Security: If it is possible that for certain input one of your threads might block, a malicious client could submit this "evil input", make your threads block, and keep them from working on other clients. This would be a Denial of Service attack.

#### Node JS 提供的处理机制

##### What code runs on the Event Loop?

When they begin, Node applications first complete an initialization phase, require'ing modules and registering callbacks for events. Node applications then enter the Event Loop, responding to incoming client requests by executing the appropriate callback. This callback executes synchronously, and may register asynchronous requests to continue processing after it completes. The callbacks for these asynchronous requests will also be executed on the Event Loop.

##### What code runs on the Worker Pool?

Node's Worker Pool is implemented in libuv, which expose a general task submission API.

Node uses the Worker Pool to handle "expensive" tasks. This includes I/O for which an operating system does not provide a non-blocking version, as well as particularly CPU-intensive tasks.

##### How does Node decide what code to run next?

In truth, the Event loop doesn't actually maintain a queue. Instead, it has a collection of file descriptors that it asks the operating system to monitor, using a mechanism like epoll(Linux), kqueue(OSX), event ports(Solaris), or IOCP(Windows). These file descriptors correspond to network sockets, any files it is watching, and so on. When the operating system says that one of these file descriptors is ready, the Event loop translates it to the appropriate event and invokes the callback(s) associated with the event. 

In contrast, the Worker Pool uses a real queue whose entries are tasks to processed. A Worker pops a task from this queue and works on it, and when finished the Worker raises an "At least one task is finished" event for the Event Loop.


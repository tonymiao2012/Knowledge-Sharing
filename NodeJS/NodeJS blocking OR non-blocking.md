#### 摘要

在NodeJS里，JS的执行时单线程的，但是这并不是意味着Node只能支持单线程。libUV为Node提供了异步非阻塞I/O（Async non-blocking I/O）能力，这肯定要归功于Event loop机制。

下面我们将区分Blocking与Non-Blocking的区别

#### Blocking


## 介绍
v8 基本由负责内存管理的堆空间和执行栈构成（它精简但能解决问题）。而回调队列，事件循环以及其他一些 api （DOM，ajax，setTimeout 等等）则可以在 Chrome（WebAPI） 或者 Node.js（NodeAPI） 里找到：
```
    +------------------------------------------------------------------------------------------+
    | Google Chrome                                                                            |
    |                                                                                          |
    | +----------------------------------------+          +------------------------------+     |
    | | Google V8                              |          |            WebAPIs           |     |
    | | +-------------+ +---------------+      |          |                              |     |
    | | |    Heap     | |     Stack     |      |          |                              |     |
    | | |             | |               |      |          |                              |     |
    | | |             | |               |      |          |                              |     |
    | | |             | |               |      |          |                              |     |
    | | |             | |               |      |          |                              |     |
    | | |             | |               |      |          |                              |     |
    | | +-------------+ +---------------+      |          |                              |     |
    | |                                        |          |                              |     |
    | +----------------------------------------+          +------------------------------+     |
    |                                                                                          |
    |                                                                                          |
    | +---------------------+     +---------------------------------------+                    |
    | |     Event loop      |     |          Callback queue               |                    |
    | |                     |     |                                       |                    |
    | +---------------------+     +---------------------------------------+                    |
    |                                                                                          |
    |                                                                                          |
    +------------------------------------------------------------------------------------------+
```
执行栈是一个由指针组成的栈结构。当函数被调用时，这个函数将会被 push 进栈。直到它 return 时才会出栈。如果这个函数里调用了其他函数，这些函数也会被 push 进栈里。当这些子函数都 return 了，调用它们的函数才能 return 出栈。如果其中一个函数需要更多时间来执行，进程会被阻塞，进程只能等到这个函数 return 并弹出栈，才能继续运行下去。这就是当你使用一个单线程编程语言时，会发生什么。

以上描述了同步函数， 那么异步函数呢？我们以 setTimeout 为例聊聊，setTimeout 函数将会被 push 进调用栈并执行。这时候就轮到回调队列和事件循环发挥它们的作用了。setTimeout 函数能往回调队列中添加函数。这个队列将会在调用栈空闲的时候在事件循环的帮助下运行。

### 任务
一个任务是一个被安排到回调队列里的函数，例如 `setTimeout` 或者 `setInterval` 这种 WebAPIs 。当事件循环开始执行任务的时候，它会准时地在任务队列里运行所有任务。

When the execution stack is empty all the tasks in the microtask queue will be
run, and if any of these tasks add tasks to the microtask queue that will also
be run which is different compared with how the task queue handles this situation.

在 Node.js 里 `setTimeout` 和 `setInterval`...（待续）

### 微任务
Is a function that is executed after current function has run after all the
other functions that are currently on the call stack.

Microtasks internals info can be found in [microtasks](./microtasks.md).


### 微任务队列
当一个 promise 被创造出来，它将会马上被执行，当它被 resovled 的时候你可以通过 `then` 调用它。

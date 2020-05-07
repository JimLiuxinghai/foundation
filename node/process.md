## 进程

关于 Process, 我们需要讨论的是两个概念, ①操作系统的进程, ② Node.js 中的 Process 对象. 操作进程对于服务端而言, 好比 html 之于前端一样基础. 想做服务端编程是不可能绕过 Unix/Linux 的. 在 Linux/Unix/Mac 系统中运行 `ps -ef` 命令可以看到当前系统中运行的进程. 各个参数如下:



| 列名称 | 意义                  |
| ------ | --------------------- |
| UID    | 执行该进程的用户ID    |
| PID    | 进程编号              |
| PPID   | 该进程的父进程编号    |
| C      | 该进程所在的CPU利用率 |
| STIME  | 进程执行时间          |
| TTY    | 进程相关的终端类型    |
| TIME   | 进程所占用的CPU时间   |
| CMD    | 创建该进程的指令      |



### Process

这里来讨论 Node.js 中的 `process` 对象. 直接在代码中通过 `console.log(process)` 即可打印出来. 可以看到 process 对象暴露了非常多有用的属性以及方法, 具体的细节见[官方文档](https://nodejs.org/dist/latest-v6.x/docs/api/process.html), 已经说的挺详细了. 其中包括但不限于:

- 进程基础信息
- 进程 Usage
- 进程级事件
- 依赖模块/版本信息
- OS 基础信息
- 账户信息
- 信号收发
- 三个标准流



`process.nextTick` 并不属于 Event loop 中的某一个阶段, 而是在 Event loop 的每一个阶段结束后, 直接执行 `nextTickQueue` 中插入的 "Tick", 并且直到整个 Queue 处理完. 所以面试时又有可以问的问题了, `递归调用 process.nextTick 会怎么样?`（[答案](https://juejin.im/post/5dd24ecce51d453fb903ff37#heading-6)）



[关于console.log](https://github.com/Q-Angelo/Nodejs-Roadmap/blob/master/docs/nodejs/console.md)



### Child Process

子进程 (Child Process) 是进程中一个重要的概念. 你可以通过 Node.js 的 `child_process` 模块来执行可执行文件, 调用命令行命令, 比如其他语言的程序等. 也可以通过该模块来将 .js 代码以子进程的方式启动. 比较有名的网易的分布式架构 [pomelo](https://github.com/NetEase/pomelo) 就是基于该模块 (而不是 `cluster`) 来实现多进程分布式架构的.

​		child_process.fork 与 POSIX 的 fork 有什么区别?

Node.js 的 `child_process.fork()` 在 Unix 上的实现最终调用了 POSIX [fork(2)](http://man7.org/linux/man-pages/man2/fork.2.html), 而 POSIX 的 fork 需要手动管理子进程的资源释放 (waitpid), child_process.fork 则不用关心这个问题, Node.js 会自动释放, 并且可以在 option 中选择父进程死后是否允许子进程存活.



- spawn() 启动一个子进程来执行命令
  - options.detached 父进程死后是否允许子进程存活
  - options.stdio 指定子进程的三个标准流
- spawnSync() 同步版的 spawn, 可指定超时, 返回的对象可获得子进程的情况
- exec() 启动一个子进程来执行命令, 带回调参数获知子进程的情况, 可指定进程运行的超时时间
- execSync() 同步版的 exec(), 可指定超时, 返回子进程的输出 (stdout)
- execFile() 启动一个子进程来执行一个可执行文件, 可指定进程运行的超时时间
- execFileSync() 同步版的 execFile(), 返回子进程的输出, 如何超时或者 exit code 不为 0, 会直接 throw Error
- fork() 加强版的 spawn(), 返回值是 ChildProcess 对象可以与子进程交互



相关题目：

- 什么是进程和线程？之间的区别？参考：[Interview1](https://www.nodejs.red/#/nodejs/process-threads?id=interview1)
- 什么是孤儿进程？参考：[Interview2](https://www.nodejs.red/#/nodejs/process-threads?id=interview2)
- 创建多进程时，代码里有 `app.listen(port)` 在进行 fork 时，为什么没有报端口被占用？参考：[Interview3](https://www.nodejs.red/#/nodejs/process-threads?id=interview3)
- 什么是 IPC 通信，如何建立 IPC 通信？什么场景下需要用到 IPC 通信？参考：[Interview4](https://www.nodejs.red/#/nodejs/process-threads?id=interview4)
- Node.js 是单线程还是多线程？进一步会提问为什么是单线程？参考：[Interview5](https://www.nodejs.red/#/nodejs/process-threads?id=interview5)
- 关于守护进程，是什么、为什么、怎么编写？参考：[Interview6](https://www.nodejs.red/#/nodejs/process-threads?id=interview6)
- 实现一个简单的命令行交互程序？参考：[Interview7](https://www.nodejs.red/#/nodejs/process-threads?id=interview7)
- 如何让一个 js 文件在 Linux 下成为一个可执行命令程序？参考：[Interview8](https://www.nodejs.red/#/nodejs/process-threads?id=interview8)
- 进程的当前工作目录是什么? 有什么作用？参考：[Interview9](https://www.nodejs.red/#/nodejs/process-threads?id=interview9)
- 多进程或多个 Web 服务之间的状态共享问题？参考：[Interview10](https://www.nodejs.red/#/nodejs/process-threads?id=interview10)
- 什么是僵尸进程？参考：[Interview11](https://www.nodejs.red/#/nodejs/process-threads?id=interview11)


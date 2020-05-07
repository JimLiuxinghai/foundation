## Node Cluster

之前使用 PM2 管理进程还是挺多的，它是怎么实现的呢？在 PM2 的配置文件中可以设置 exec_model:'cluster' 和 instances 两个属性来设置开启多个进程，PM2 其实主要也是利用 Nodejs Cluster 这个模块来实现了，还有 Egg.js 中的 egg-cluster 模块在启动 Worker 进程时也是使用的 Nodejs Cluster 模块。下面来了解下 Nodejs Cluster 这个模块做了什么？

下面参考了 Nodejs 官网提供的一个 Cluster 代码示例，命名为 app.js 下文会用到这个 Demo。

```javascript
// app.js
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    console.log(`Master 进程 ${process.pid} 正在运行`);

  for (let i = 0; i < numCPUs; i++) { // 衍生工作进程。
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => { console.log(`Worker ${worker.process.pid} 已退出`) });
} else {
  http.createServer((req, res) => res.end(`你好世界 ${process.pid}`)).listen(8000);
  console.log(`Worker 进程 ${process.pid} 已启动`);
}

```

### 集群方案

- 方案一：1 个 Node 实例开启多个端口，通过反向代理服务器向各端口服务进行转发
- 方案二：1 个 Node 实例开启多个进程监听同一个端口，通过负载均衡技术分配请求（Master->Worker）

首先第一种方案存在的一个问题是占用多个端口，造成资源浪费，由于多个实例是独立运行的，进程间通信不太好做，好处是稳定性高，各实例之间无影响。

第二个方案多个 Node 进程去监听同一个端口，好处是进程间通信相对简单、减少了端口的资源浪费，但是这个时候就要保证服务进程的稳定性了，特别是对 Master 进程稳定性要求会更高，编码也会复杂。

在 Nodejs 中自带的 Cluster 模块正是采用的第二种方案。

上面的例子：

```javascript
node app.js
Master 进程 45016 正在运行
Worker 进程 45018 已启动
Worker 进程 45019 已启动
Worker 进程 45017 已启动
Worker 进程 45020 已启动
```

因为端口只有一个 8000，所以我们要来看下它是由哪些进程所监听的。

```bash
lsof -i -P -n | grep 8000
node      45016 qufei   40u  IPv6 0x7a7b5a0f7e0fbb4f      0t0  TCP *:8000 (LISTEN)
```



理论上说如果多个进程监听同一个端口是会报端口冲突的，现在我们知道了，8000 端口它并**不是被所有的进程全部的监听，仅受到 Master 进程的监听**，下面让我们在看一个信息。

```bash
ps -ef | grep 45016

502 45016 44110   0  9:49上午 ttys001    0:00.10 node test
502 45017 45016   0  9:49上午 ttys001    0:00.11 /Users/.nvm/versions/node/v12.13.0/bin/node /Users/study/test/test
502 45018 45016   0  9:49上午 ttys001    0:00.11 /Users/.nvm/versions/node/v12.13.0/bin/node /Users/study/test/test
502 45019 45016   0  9:49上午 ttys001    0:00.11 /Users/.nvm/versions/node/v12.13.0/bin/node /Users/study/test/test
502 45020 45016   0  9:49上午 ttys001    0:00.11 /Users/.nvm/versions/node/v12.13.0/bin/node /Users/study/test/test
```

这个清楚展示了 Worker 与 Master 的关系，Master 通过 cluster.fork() 这个方法创建的，本质上还是使用的 child_process.fork() 这个方法，怎么实现进程间端口共享呢？

总结起来一句话：“**Master 进程创建一个 Socket 并绑定监听到该目标端口，通过与子进程之间建立 IPC 通道之后，通过调用子进程的 send 方法，将 Socket（链接句柄）传递过去**”。

下面展示一个使用 child_process.fork() 创建的子进程，进行 Socket 传递的示例：

```javascript
// master.js
const fork = require('child_process').fork;
const cpus = require('os').cpus();
const server = require('net').createServer().listen(3000);

for (let i=0; i<cpus.length; i++) {
    const worker = fork('worker.js');
      // 将 Master 的 server 传递给子进程
    worker.send('server', server);
    console.log('worker process created, pid: %s ppid: %s', worker.pid, process.pid);
}

// worker.js 
const http = require('http');
const server = http.createServer((req, res) => {
    res.end('I am worker, pid: ' + process.pid + ', ppid: ' + process.ppid);
});

let worker;
// 第二个参数 sendHandle 就是句柄，可以是 TCP套接字、TCP服务器、UDP套接字等
process.on('message', function (message, sendHandle) {
    if (message === 'server') {
        worker = sendHandle;
        worker.on('connection', function(socket) {
            server.emit('connection', socket);
        });
    }
});
```

**总结下：** 端口只会被主进程绑定监听一次，但是主进程和子进程在建立 IPC 通信之后，发送 Socket 到子进程实现端口共享，在之后 Master 接收到新的客户端链接之后，通过负载均衡技术再转发到各 Worker 进程，这个下文会讲。



### 多进程通信

还是上面提到的，cluster.fork() 本质上还是使用的 child_process.fork() 这个方法来创建的子进程，进程间通信无非几种：pipe（管道）、消息队列、信号量、Domain Socket。
在 Nodejs 中是通过 pipe（管道）实现的，pipe 作用于之间有血缘关系的进程，通过 fork 传递，其本身也是一个进程，将一个进程的输出做为另外一个进程的输入，常见的 Linux 所提供的管道符 “|” 就是将两个命令隔开,管道符左边命令的输出就会作为管道符右边命令的输入。

推荐看下 [10 个 Nodejs 进程相关的问题解答](https://mp.weixin.qq.com/s/dKN95zcRI7qkwGYKhPXrcg) 也是笔者之前写的文章，有提到进程的通信、还有上面讲解的多个进程监听同一个端口问题，还提供了些例子，可以看看。
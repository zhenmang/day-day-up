## ## child_process实现多进程

#### 示例一： 发送 server 对象

**单一master和worker**

`sendHandle` 参数将TCP server 对象的句柄传给子进程

主进程

```
const subprocess = require('child_process').fork('subprocess.js');

// 打开 server 对象，并发送该句柄。
const server = require('net').createServer();
server.on('connection', (socket) => {
  socket.end('由父进程处理');
});
server.listen(1337, () => {
  subprocess.send('server', server);
});
```

子进程接收 server 对象如下：

```
process.on('message', (m, server) => {
  if (m === 'server') {
    server.on('connection', (socket) => {
      socket.end('由子进程处理');
    });
  }
});
```

**多进程**

master.js 代码如下：

```
const childProcess = require('child_process');
const net = require('net');

// 获取cpu的数量
const cpuNum = require('os').cpus().length;

let workers = [];
let cur = 0;

for (let i = 0; i < cpuNum; ++i) {
  workers.push(childProcess.fork('./worker.js'));
  console.log('worker process-' + workers[i].pid);
}

// 创建TCP服务器
const tcpServer = net.createServer();

/*
 服务器收到请求后分发给工作进程去处理
*/
tcpServer.on('connection', (socket) => {
  workers[cur].send('socket', socket);
  cur = Number.parseInt((cur + 1) % cpuNum);
});

tcpServer.listen(8989, () => {
  console.log('Tcp Server: 127.0.0.8989');
});
```



worker.js 代码如下：

```
// 接收主进程发来的消息
process.on('message', (msg, socket) => {
  if (msg === 'socket' && socket) {
    // 利用setTimeout 模拟异步请求
    setTimeout(() => {
      socket.end('Request handled by worker-' + process.pid);
    },100);
  }
});
```

#### 示例二：发送 socket 对象

类似地， `sendHandle` 参数可用于将 socket 的句柄传给子进程。 以下示例衍生了两个子进程，分别用于处理 "normal" 连接或优先处理 "special" 连接：

```
const { fork } = require('child_process');
const normal = fork('subprocess.js', ['normal']);
const special = fork('subprocess.js', ['special']);

// 开启 server，并发送 socket 给子进程。
// 使用 `pauseOnConnect` 防止 socket 在被发送到子进程之前被读取。
const server = require('net').createServer({ pauseOnConnect: true });
server.on('connection', (socket) => {

  // 特殊优先级。
  if (socket.remoteAddress === '74.125.127.100') {
    special.send('socket', socket);
    return;
  }
  // 普通优先级。
  normal.send('socket', socket);
});
server.listen(1337);
```

`subprocess.js` 会接收该 socket 句柄作为传给事件回调函数的第二个参数：

```
process.on('message', (m, socket) => {
  if (m === 'socket') {
    if (socket) {
      // 检查客户端 socket 是否存在。
      // socket 在被发送与被子进程接收这段时间内可被关闭。
      socket.end(`请求使用 ${process.argv[2]} 优先级处理`);
    }
  }
});
```

不要使用已被传给子进程的 socket 上的 `.maxConnections`。 父进程无法追踪 socket 何时被销毁。

子进程中的任何 `'message'` 句柄都应该验证 `socket` 是否存在，因为连接可能会在它发送给子进程的这段时间内被关闭。

##  cluster实现多进程

```js
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`主进程 ${process.pid} 正在运行`);

  // 衍生工作进程。
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`工作进程 ${worker.process.pid} 已退出`);
  });
} else {
  // 工作进程可以共享任何 TCP 连接。
  // 在本例子中，共享的是 HTTP 服务器。
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('你好世界\n');
  }).listen(8000);

  console.log(`工作进程 ${process.pid} 已启动`);
}
```

运行代码，则工作进程会共享 8000 端口：

```console
$ node server.js
主进程 3596 正在运行
工作进程 4324 已启动
工作进程 4520 已启动
工作进程 6056 已启动
工作进程 5644 已启动
```

 cluster工作原理：工作进程由 [`child_process.fork()`](http://nodejs.cn/s/VDCJMa) 方法创建，因此它们可以使用 IPC 和父进程通信，从而使各进程交替处理连接服务。  



[child_process](https://www.cnblogs.com/tugenhua0707/p/11141076.html#_labe3)  

[cluster](https://wiki.jikexueyuan.com/project/nodejs/cluster.html)
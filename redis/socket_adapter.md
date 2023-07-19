##一、适配器如何运作##  

每个发送给多个客户的数据包(例如： io.to("room1").emit() 或 socket.broadcast.emit())是：  

发送到连接到当前服务器的所有匹配客户端  

在 Redis 通道中发布，并由集群的其他 Socket.IO 服务器接收  

![img](https://socket.io/zh-CN/images/broadcasting-redis.png)  
[适配器源码](https://github.com/socketio/socket.io-redis-adapter)  

安装
```js
npm install @socket.io/redis-adapter redis
```
用法
```js
import { Server } from "socket.io";
import { createAdapter } from "@socket.io/redis-adapter";
import { createClient } from "redis";

const io = new Server();

const pubClient = createClient({ url: "redis://localhost:6379" });
const subClient = pubClient.duplicate();

Promise.all([pubClient.connect(), subClient.connect()]).then(() => {
  io.adapter(createAdapter(pubClient, subClient));
  io.listen(3000);
});
```

##二、Emitter##  

Redis 发射器允许从另一个 Node.js 进程向连接的客户端发送数据包：  
![img](https://socket.io/zh-CN/images/redis-emitter.png)  
安装
```js
npm install @socket.io/redis-emitter redis
```
用法
```js
import { Emitter } from "@socket.io/redis-emitter";
import { createClient } from "redis";

const redisClient = createClient({ url: "redis://localhost:6379" });

redisClient.connect().then(() => {
  const emitter = new Emitter(redisClient);

  setInterval(() => {
    emitter.emit("time", new Date);
  }, 5000);
});
```

## nodejs的docker搭建部署

#### docker简介

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 Linux或Windows机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

一个完整的Docker有以下几个部分组成：

1. DockerClient客户端
2. Docker Daemon守护进程
3. Docker Image镜像
4. DockerContainer容器  

### 部署

### docker-compose.yml

在docker-compose.yml中配置相关服务节点，同时在每个服务节点中配置相关的镜像、网络、环境、磁盘映射等元信息，也可指定具体Dockerfile文件构建镜像使用。

```
version: '3'
services:
  nginx:
    image: nginx:latest
    ports:
      - 80:80
    restart: always  
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - /tmp/logs:/var/log/nginx

  redis-server:
    image: redis:latest
    ports:
      - 6479:6379
    restart: always

  app:
    build: ./
    volumes:
      - ./:/usr/local/app
    restart: always  
    working_dir: /usr/local/app
    ports:
      - 8090:8090
    command: node server/server.js
    depends_on:
      - redis-server
    links:
      - redis-server:rd
```

### redis服务器

首先搭建一个单节点缓存服务，采用官方提供的redis最新版镜像，无需构建。

```
version: '3'
services:
  redis-server:
    image: redis:latest
    ports:
      - 6479:6379
    restart: always
```

关于version具体信息，可参考[Compose and Docker compatibility matrix](https://docs.docker.com/compose/compose-file/)找到对应docker引擎匹配的版本格式。
在services下，创建了一个名为 **redis-server** 的服务，它采用最新的redis官方镜像，并通过宿主机的6479端口向外提供服务。并设置自动重启功能。

此时，在宿主机上可以通过6479端口使用该缓存服务。

### web应用

使用node.js的koa、koa-router可快速搭建web服务器。在本节中，创建一个8090端口的服务器，同时提供两个功能：1. 简单查询单个key的缓存 2. 流水线查询多个key的缓存

docker-compose.yml

```
services:
  app:
    build: ./
    volumes:
      - ./:/usr/local/app
    restart: always  
    working_dir: /usr/local/app
    ports:
      - 8090:8090
    command: node server/server.js
    depends_on:
      - redis-server
    links:
      - redis-server:rd
```

此处创建一个app服务，它使用当前目录下的Dockerfile构建后的镜像，同时通过 volumes 配置磁盘映射，将当前目录下所有文件映射至容器的/usr/local/app，并制定为运行时目录；同时映射宿主机的8090端口，最后执行`node server/server.js`命令运行服务器。

通过**depends_on**设置app服务的依赖，等待 redis-server 服务启动后再启动app服务；通过**links**设置容器间网络连接，在app服务中，可通过别名 **rd** 访问redis-server。

Dockerfile

```
FROM node:8-slim
COPY ./ /usr/local/app
WORKDIR /usr/local/app
RUN npm i --registry=https://registry.npm.taobao.org
ENV NODE_ENV dev
EXPOSE 8090  
```

指定的Dockerfile则做了初始化npm的操作。

web-server sourcecode

```
const Koa = require('koa');
const Router = require('koa-router');
const redis = require('redis');
const { promisify } = require('util');


let app = new Koa();
let router = new Router();
let redisClient = createRedisClient({
    // ip为docker-compose.yml配置的redis-server别名 rd，可在应用所在容器查看dns配置
    ip: 'rd',
    port: 6379,
    prefix: '',
    db: 1,
    password: null
});

function createRedisClient({port, ip, prefix, db}) {
    let client = redis.createClient(port, ip, {
        prefix,
        db,
        no_ready_check: true
    });
    
    client.on('reconnecting', (err)=>{
        console.warn(`redis client reconnecting, delay ${err.delay}ms and attempt ${err.attempt}`);
    });
    
    client.on('error', function (err) {
        console.error('Redis error!',err);
    });
    
    client.on('ready', function() {
        console.info(`redis初始化完成,就绪: ${ip}:${port}/${db}`);
    });
    return client;
}

function execReturnPromise(cmd, args) {
    return new Promise((res,rej)=>{
        redisClient.send_command(cmd, args, (e,reply)=>{
            if(e){
                rej(e);
            }else{
                res(reply);
            }
        });
    });
}

function batchReturnPromise() {
    return new Promise((res,rej)=>{
        let b = redisClient.batch();
        b.exec = promisify(b.exec);
        res(b);
    });
}


router.get('/', async (ctx, next) => {
    await execReturnPromise('set',['testkey','helloworld']);
    let ret = await execReturnPromise('get',['testkey']);
    ctx.body = {
        status: 'ok',
        result: ret,
    };
});

router.get('/batch', async (ctx, next) => {
    await execReturnPromise('set',['testkey','helloworld, batch!']);
    let batch = await batchReturnPromise();
    for(let i=0;i < 10;i++){
        batch.get('testkey');
    }
    let ret = await batch.exec();
    ctx.body = {
        status: 'ok',
        result: ret,
    };
});

app
  .use(router.routes())
  .use(router.allowedMethods())
  .listen(8090);
```

需要注意的是，在web服务所在的容器中，通过别名 **rd** 访问缓存服务。

此时，运行命令 `docker-compose up`后，即可通过 [http://127.0.0.1:8090/](http://127.0.0.1:8090/) [http://127.0.0.1:8090/batch](http://127.0.0.1:8090/batch) 访问这两个缓存服务。

### 转发

目前可以通过宿主机的8090端口访问服务，为了此后web服务的可扩展性，需要在前端加入转发层。实例中使用nginx进行转发：

```
services:
  nginx:
    image: nginx:latest
    ports:
      - 80:80
    restart: always  
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - /tmp/logs:/var/log/nginx
```

采用最新版的nginx官方镜像，向宿主机暴露80端口，通过在本地配置nginx的抓发规则文件，映射至容器的nginx配置目录下实现快速高效的测试。



## 运行与扩展

默认单节点下，直接运行

```
docker-compose up -d
```

即可运行服务。

如果服务节点需要扩展，可通过

```
docker-compose up -d --scale app=3
```

扩展为3个web服务器，同时nginx转发规则需要修改：

```
upstream app_server { # 设置server集群,负载均衡关键指令
    server docker-web-examples_app_1:8090; # 设置具体server,
    server docker-web-examples_app_2:8090;
    server docker-web-examples_app_3:8090;
}

server {
    listen 80;
    charset utf-8;

    location / {
        proxy_pass http://app_server;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

app_server内部的各个服务器名称为**docker-web-examples_app_1**，format为“{path}_{path}_{service}_${number}”,

即第一部分为 docker-compose.yml所在目录名称，如果在根目录则为应用名称；
第二部分为扩展的服务名；
第三部分为扩展序号

通过设置nginx的配置的log_format中upstream_addr变量，可观察到负载均衡已生效。

```
http{
    log_format  main  '$remote_addr:$upstream_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
}
```

<br>

<br>

<br>

[docker官方](https://docs.docker.com/compose/compose-file/)

[docker-nodejs](https://www.cnblogs.com/accordion/p/10450952.html)
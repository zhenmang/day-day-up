[Nginx中文文档](https://www.nginx.cn/doc/index.html)

## 查找目录、文件、进程

查看nginx命令安装在哪

```
如果linux系统
whereis nginx
如果是苹果系统
where nginx
```

查看nginx.conf文件在哪

[find用法](https://www.cnblogs.com/JcHome/p/10852949.html)

```
全局查找文件
find / -name nginx.conf
当前目录下查找文件
find . -name nginx.conf
全局查找nginx目录
find / –type d –name nginx
```

查看nginx进程号

```
ps -ef|grep nginx
```

## 启动、关闭、重启

启动nginx

```
cd usr/local/nginx/sbin
./nginx
或者 cd /bin
systemctl start nginx
```

杀死nginx进程

```
查询nginx主进程号

　　ps -ef | grep nginx

　　从容停止   kill -QUIT 主进程号

　　快速停止   kill -TERM 主进程号

　　强制停止   kill -9 nginx

　　若nginx.conf配置了pid文件路径，如果没有，则在logs目录下

　　kill -信号类型 '/usr/local/nginx/logs/nginx.pid'
　　
　　或者 cd /bin
　　    systemctl stop nginx
```

重启nginx进程（一般因更改配置重启nginx）

```
kill -HUP 主进程号或进程号文件路径
或者使用
cd /usr/local/nginx/sbin
./nginx -s reload

或者 cd /bin
　　 systemctl start nginx
```

 判断配置文件是否正确　

```
nginx -t -c /usr/local/nginx/conf/nginx.conf
或者
cd  /usr/local/nginx/sbin
./nginx -t
```

查询nginx运行情况

```
ps aux | grep nginx
```



## 配置文件

**主配置文件nginx.conf**

//  文件路径
/etc/nginx/nginx.conf

//  文件内容说明

```php
#运行用户，默认即是nginx，可以不进行设置
user  nginx;
#Nginx进程，一般设置为和CPU核数一样
worker_processes  1;   
#错误日志存放目录
error_log  /var/log/nginx/error.log warn;
#进程pid存放位置
pid        /var/run/nginx.pid;

events {
    worker_connections  1024; # 单个后台进程的最大并发数
}

http {
    include       /etc/nginx/mime.types;   #文件扩展名与类型映射表
    default_type  application/octet-stream;  #默认文件类型
    #设置日志模式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;   #nginx访问日志存放位置

    sendfile        on;   #开启高效传输模式
    #tcp_nopush     on;    #减少网络报文段的数量

    keepalive_timeout  65;  #保持连接的时间，也叫超时时间

    #gzip  on;  #开启gzip压缩

    include /etc/nginx/conf.d/*.conf; #包含的子配置项位置和文件
```

**子配置文件**

// 文件路径

/etc/nginx/conf.d/default.conf

//  文件内容说明

```php
server {
    listen       80;   #配置监听端口
    server_name  localhost;  //配置域名

    #charset koi8-r;     
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;     #服务默认启动目录
        index  index.html index.htm;    #默认访问文件
    }

    #error_page  404              /404.html;   # 配置404页面

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;   #错误状态码的显示页面，配置后需要重启
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```
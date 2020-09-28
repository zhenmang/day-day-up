## 反向代理

```
server {
	listen 80;
	location / {
		proxy_pass http://192.168.0.112:8080; # 应用服务器HTTP地址
	}
}
```

## 自动索引

- 处理静态文件，索引文件以及自动索引；

## 负载均衡

将相同的应用部署在多台服务器上，将大量用户的请求分配给多台机器处理。同时带来的好处是，其中一台服务器万一挂了，只要还有其他服务器正常运行，就不会影响用户使用。

```
upstream myweb {
	server 192.168.0.111:8080; # 应用服务器1
	server 192.168.0.112:8080; # 应用服务器2
}
server {
	listen 80;
	location / {
		proxy_pass http://myweb;
	}
}
```

## 虚拟主机

例如将www.aaa.com和www.bbb.com两个网站部署在同一台服务器上，两个域名解析到同一个IP地址，但是用户通过两个域名却可以打开两个完全不同的网站，互相不影响，就像访问两个服务器一样，所以叫两个虚拟主机。

```
server {
	listen 80 default_server;
	server_name _;
	return 444; # 过滤其他域名的请求，返回444状态码
}
server {
	listen 80;
	server_name www.aaa.com; # www.aaa.com域名
	location / {
		proxy_pass http://localhost:8080; # 对应端口号8080
	}
}
server {
	listen 80;
	server_name www.bbb.com; # www.bbb.com域名
	location / {
		proxy_pass http://localhost:8081; # 对应端口号8081
	}
}
```

## 防盗链

```
location ~* \.(gif|jpg|png|swf|flv)$ {
    root html
    valid_referers none blocked *.nginxcn.com;
    if ($invalid_referer) {
        rewrite ^/ [www.nginx.cn](http://www.nginx.cn/)
        return 404;
    }
}
```


前面的root可以不要如果你在server{}中有设置可以不需要设定



## FastCGI

CGI协议虽然解决了语言解析器和web server之间通讯的问题，但是它的效率很低，因为web server每收到一个请求都会创建一个CGI进程，PHP解析器都会解析php.ini文件，初始化环境，请求结束的时候再关闭进程，对于每一个创建的CGI进程都会执行这些操作，所以效率很低，而FastCGI是用来提高CGI性能的，FastCGI每次处理完请求之后不会关闭掉进程，而是保留这个进程，使这个进程可以处理多个请求。这样的话每个请求都不用再重新创建一个进程了，大大提升了处理效率。

```
    location ~ \.php$ {
        root html;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name; #默认脚本路径
        include fastcgi_params;
    }
```


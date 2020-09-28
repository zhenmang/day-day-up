## 一.发现问题

配置nginx代理的时候，发现location配置的路径和代理的上下文路径的组合不同，服务端接收到的uri的路径不同，导致了controller的RequestMapping匹配出现问题，所以就仔细研究了一下nginx路径配置的细节问题；

## 二.实验过程

关于nginx的location路径和proxy_pass代理的上下文路径细节问题，以下分为四种情况来说明：

所有请求nginx服务的url都为：`http://192.168.0.105:8087/api/system/common/doLogin`，

下面通过nginx配置的路径不同，来观察服务端接收到的uri是什么样的；

 

1.情况一：

location后的路径为“/api”，代理服务的上下文为空（`http://192.168.0.105:8089`），看下图：

![img](/imgs/nginx_nopath.png)

服务端接收到的uri为：

```
/api/system/common/doLogin
```

这个结果说明代理的时候，按照请求路径中的/api匹配到了对应的代理，代理到服务端的时候，把请求的ip和port替换为代理服务的ip和port，而对应的uri则完全没有变化，是整体的复制过去。

请求nginx：`http://192.168.0.105:8087/api/system/common/doLogin`

Nginx代理替换之后：`http://192.168.0.105:8089/api/system/common/doLogin`



代理过程：将图中红色部分替换为绿色部分，其他都不变，然后发送到服务端:

![img](/imgs/nopathtip.png)

 

2.情况二：

location后的路径为“/api”，代理服务的上下文为“/”（`http://192.168.0.105:8089/`），看下图：

![img](/imgs/qingkuang2.png)

服务端接收的uri为：

```
//system/common/doLogin
```

这个结果说明按照请求路径中的/api匹配到了对应的代理之后，把请求中的“/api”替换为代理服务上下文路径“/”，“/api”之后的路径不变，替换如下：

请求nginx：`http://192.168.0.105:8087/api/system/common/doLogin`

Nginx代理替换之后：`http://192.168.0.105:8089/ /system/common/doLogin`

代理过程：将红色的部分替换为绿色的部分，其他都不变，然后发送服务端:

![img](/imgs/qingkuang2tip.png)

3.情况三：

location后的路径为“/api/”，代理服务的上下文为空（`http://192.168.0.105:8089`），看下图：

![img](/imgs/qingkuang3.png)

服务端接收的uri为：

```
/api/system/common/doLogin
```



这个结果和情况一是一样的，按照请求路径中的/api/匹配到了对应的代理，代理到服务端的时候，把请求的ip和port替换为代理服务的ip和port，而对应的uri则完全没有变化，是整体的复制过去。

请求nginx `http://192.168.0.105:8087/api/system/common/doLogin`

Nginx代理替换之后：`http://192.168.0.105:8089/api/system/common/doLogin`

 

代理过程：将红色部分替换为绿色部分，其他部分不变:

![img](/imgs/qingkuang3tip.png)

 

4.情况四：

location后的路径为“/api/”，代理服务的上下文为“/”（`http://192.168.0.105:8089/`），看下图：

![img](/imgs/qingkuang4.png)

服务端接收的uri为：

```
/system/common/doLogin
```

这个结果说明和情况二是一样的，按照请求路径中的/api/匹配到了对应的代理之后，把请求中的“/api/”替换为代理服务上下文路径“/”，“/api/”之后的路径不变，替换如下：

请求nginx：`http://192.168.0.105:8087/api/system/common/doLogin`

Nginx代理替换之后：`http://192.168.0.105:8089/system/common/doLogin`

 

代理过程：将红色部分替换为绿色部分，其他部分不变:

![img](/imgs/qingkuang4tip.png)

 

## 三.解决问题：

根据上述四种情况总结，归结为两种代理过程：

 

1.如果代理proxy_pass只有ip和port，没有上下文，则代理过程只替换掉nginx请求的ip和port，其他部分都不变，然后发送。

 

2.如果代理proxy_pass除了ip和port外，还有上下文，则代理过程中，除了替换ip和port之外,还会将location的路径替换为proxy_pass中的上下文， 然后将剩余的路径拼接，发送。
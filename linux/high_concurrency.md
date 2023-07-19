##一、丢包##   

####握手阶段####  

多路径信号衰竭、通道阻塞、数据包损坏都会导致丢包  

netstat -s|grep -iE LISTEN 查看丢包情况，如下  

    35206 times the listen queue of a socket overflowed（队列溢出） 

    39719 SYNs to LISTEN sockets dropped（丢包，没有SYN+ACK响应） 


netstat -n -p TCP | grep SYN_RECV | grep :22 | wc -l  

324  

SYN攻击有很多现成软件可以做到，属DDoS攻击，伪造不存在的请求，发送请求，产生半链接，导致服务器不断重发SYN+ACk，大量占据CPU和内存资源不释放，导致正常请求被丢弃，造成系统瘫痪。

####挥手阶段####  

短链接TIME_WAIT 状态的socket  

    netstat -ant|grep -i time_wait |wc -l  

非CLOSED状态的连接都会占用端口，TIME_WAIT状态时间较长，高并发时会大量存在，导致address already in use : connect 异常，无法建立连接。  

Cannot assign requested address  

端口上限为 65535个(16 bit，2 Byte)=Math.pow(2, 16) - 1  

其中1024以下的为内核调用的端口，非root用户不能调用1024以下的端口；1024～65535之间的为程序或网络可用端口。  

65535是计算机16位二进制最大数，即单核的运算极限 端口号，颜色种类，采样频率，excel表的最大行数  


一句话总结：半链接太多或正常连接太多，都会导致没有端口地址或没有内存去响应SYN-ACK，导致连接无法建立，导致丢包。  

##二、系统限制##

用户进程限制  

主配置文件：/etc/security/limits.conf  

分配置文件：/etc/security/limits.d/*.conf  

ulimit限制的是当前shell进程以及其派生的子进程  

 举例来说，如果用户同时运行了两个shell终端进程，只在其中一个环境中执行了ulimit -s 100,则该shell进程里创建文件的大小收到相应的限制，而同时另一个shell终端包括其上运行的子程序都不会受到其影响  

ulimt -a 显示所有资源限制  

ulimit -n  
 https://baike.baidu.com/item/%E5%86%85%E6%A0%B8?fromModule=lemma_inlink可以同时打开的https://baike.baidu.com/item/%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6?fromModule=lemma_inlink的最大值，默认值是1024时  
  

系统网络控制  

/etc/sysctl.conf  

sysctl -a # 查看所有内核参数及值  

此文件包含了对于SYN队列、TIME-WAIT状态、keepalive连接、收发缓存区的时参数设置，高并发调优也从这几个角度出发调整  


##三、数据库限制##  

慢查（缺少索引）  

最大连接数  

写日志  

缓存操作区小  

##四、高并发调优##  

####1.调整用户进程限制####  

临时配置  

ulimit -SHn  655350 将最大文件打开数调整到65535个  

验证是否设置成功  

ulimit -n  

重启应用生效  

永久配置（忽略）   

* soft nofile 65535
* hard nofile 65535
* soft nproc  65535 
* hard nproc  65535  

配置到配置文件/etc/security/limits.conf，然后退出当前会话，重新登录。 即可生效，重启配置也会保留。  

####2.调整系统控制参数####  

vi /etc/sysctl.conf  

加入下面的几行内容  
```code
#当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击
net.ipv4.tcp_syncookies= 1 
#允许将TIME-WAITsockets重新用于新的TCP连接
net.ipv4.tcp_tw_reuse= 1
#表示开启TCP连接中TIME-WAITsockets的快速回收（NAT内网环境下不能开启此参数，公网环境可以）
net.ipv4.tcp_tw_recycle= 0
#修改系統默认的TIMEOUT 时间
net.ipv4.tcp_fin_timeout= 30
#表示用于向外连接的端口范围。缺省情况下很小
net.ipv4.ip_local_port_range= 9000 65000
#表示SYN队列的长度，默认为1024，加大队列长度为65535，可以容纳更多等待连接的网络连接数。
net.ipv4.tcp_max_syn_backlog= 131070(而socket server可以一次性处理backlog中的所有请求)
#表示系统同时保持TIME_WAIT的最大数量，如果超过这个数字，TIME_WAIT将立刻被清除并打印警告信息。默认为180000，改为5000。
net.ipv4.tcp_max_tw_buckets= 1000
#web应用中listen函数的backlog默认会给我们内核参数的net.core.somaxconn限制到128，而nginx定义的NGX_LISTEN_BACKLOG默认为511，所以有必要调整这个值。
net.core.somaxconn= 131070
net.core.netdev_max_backlog = 131070 # 默认为1000
#设置套接字接受缓存区大小和发送缓存区大小
net.core.rmem_max=26214400（套接字接收缓冲区大小25M,系统默认0.2M）
net.core.wmem_max=26214400（套接字发送缓冲区大小）
#设置链路追踪库
32G内存推荐：
#内存使用最大值：1048576* 376 + 262144 * 16 = 398458880(byte)=380MiB
#设置跟踪链接表的大小32*1024*32
net.netfilter.nf_conntrack_max = 262144（30万长链接得用8388608）
#设置桶数量262144/4
net.netfilter.nf_conntrack_buckets = 65536
#设置TCP挥手相关
net.netfilter.nf_conntrack_tcp_timeout_established = 84600 （1天，默认5天）
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 60（默认120）
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 30（默认60）
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 60（默认120）
#修改长连接的配置将长链接改成20分钟触发探活，默认2小时
net.ipv4.tcp_keepalive_time= 1200
net.ipv4.tcp_keepalive_intvl=10（到7200s后，每隔10s探测一次）
net.ipv4.tcp_keepalive_probes=4（重连9次）
```
执行立即生效  
```code
/sbin/sysctl -p 
/sbin/sysctl -w net.ipv4.route.flush=1
```
验证设置是否生效，就是找个参数看一下，是否与设置的相等  

sysctl -a | grep tcp_keepalive_time  

设置完，要重启应用。  

执行立即生效  
```code
/sbin/sysctl -p 
/sbin/sysctl -w net.ipv4.route.flush=1
```
验证设置是否生效，就是找个参数看一下，是否与设置的相等
```code
echo 65535 > /sys/module/nf_conntrack/parameters/hashsize(等价于/sys/module/nf_conntrack/parameters/hashsize)
```
####3.调整nginx####  

(1).更改系统中关于nginx的配置文件  

/usr/lib/systemd/system/nginx.service  

在[Service]下面，添加  

LimitNOFILE=655350  

执行命令生效： sudo systemctl daemon-reload  

(2).调整nginx配置文件/etc/nginx/nginx.conf  
```code
worker_processes auto;
worker_rlimit_nofile 65535;
events {                                                                                                                          
    use epoll;                                                                                                                    
    worker_connections 65535;                                                                                     
    multi_accept on;                                                                                                             
}
http {
	log_format  main  '$remote_addr - $remote_user [$time_local] "$host" "$request" '    '$status $body_bytes_sent "$http_referer" '                                        
 '"$http_user_agent" "$http_x_forwarded_for"      	"$proxy_add_x_forwarded_for"';                                                                                                                                                                                                                                                                                                                   
    
access_log  /var/log/nginx/access.log  main;
upstream webservers{                                                                                                                                                                                                                                                                                                                                                             
       server  127.0.0.1:19019 weight=8;                                                                                                                                                                     
       server  52.221.16.111:19019 weight=2;                                                                                                                                                                 
}

server {
        listen 80 backlog=32768;
               location ^~ /api/ad {                                                                                                                                                                                
            add_header Access-Control-Allow-Origin *;                                                                                                                                                    
            proxy_set_header        HOST $host;                                                                                                                                                          
            proxy_set_header        X-real-ip $remote_addr;                                                                                                                                              
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;                                                                                                                          
            proxy_pass      http://webservers;                                                                                                                                                        
        }
 }

}
```
执行nginx -s reload使配置生效  

ss -lnt可用于查看backlog的等待队列值  

如果web服务器面对的是一个持续的请求流，那么启用multi_accept可能会造成worker进程一次接受的请求大于worker_connections指定可以接受的请求数。这就是overflow，overflow这部分的请求不会受到处理。  

####4.调整数据库####  
```code
1.每一次的事务提交是否需要把日志都写入磁盘  
show GLOBAL VARIABLES like '%innodb_flush_log_at_trx_commit%';
set GLOBAL innodb_flush_log_at_trx_commit=0;
2.sync_binlog=0，表示MySQL不控制binlog的刷新，一旦系统Crash，在binlog_cache中的所有binlog信息都会被丢失
show GLOBAL VARIABLES like '%sync_binlog%';
set GLOBAL sync_binlog=0;
3.数据和索引的缓存区域
show GLOBAL VARIABLES like '%innodb_buffer_pool_size%';
set GLOBAL innodb_buffer_pool_size=536870912;
关于innodb_buffer_pool_size的设置原则
'Innodb_buffer_pool_pages_data' X 100 / 'Innodb_buffer_pool_pages_total'
当结果 > 95% 则增加 innodb_buffer_pool_size， 建议使用 ram total 75%
当结果 < 95% 则减少 innodb_buffer_pool_size， 
建议 'Innodb_buffer_pool_pages_data' X 'Innodb_page_size' X 1.05 / (1024*1024*1024)
4.慢查询相关
show global variables like 'slow_query_log';查看慢查询是否开启
show global variables like 'long_query_time';查看慢查询阈值
set global slow_query_log=on;开启慢查询
set global long_query_time=0.1;设置慢查询阈值为3s
查询慢查的语句数量
SHOW GLOBAL STATUS LIKE '%Slow_queries%';
慢查日志所在的文件
show variables like '%slow_query_log_file%';
5.最大连接数 
查看最大连接数
SHOW VARIABLES LIKE "max_connections"; 
增加当前会话的mysql最大连接数
SET GLOBAL max_connections = 1000;
永久增加mysql最大连接数
sudo vim /etc/my.cnf
修改 max_connections = 1000 保存重启mysql
部分参考自https://segmentfault.com/a/1190000038391098
```
####5.调整后端代码####  

高并发操作：  

1.insert支持批量插入(合并集中式插入)  

2.并发连接数（tomcat面向nginx）  
```code
server:
  compression:
    enabled: true
  tomcat:
    connection-timeout: 30000
    keep-alive-timeout: 30000
    max-keep-alive-requests: 1000
    accept-count: 65535
    threads:
      max: 1000
      min-spare: 1000
3.调整数据库连接池（java面向数据库）
spring:                                                                       
  datasource:
    hikari:
      pool-name: Retail_HikariCP
      minimum-idle: 20 #最小空闲连接数量
      idle-timeout: 180000 #空闲连接存活最大时间，默认600000（10分钟）
      maximum-pool-size: 42 #连接池最大连接数，默认是10
      auto-commit: true  #此属性控制从池返回的连接的默认自动提交行为,默认值：true
      max-lifetime: 1800000 #此属性控制池中连接的最长生命周期，值0表示无限生命周期，默认1800000即30分钟
      connection-timeout: 30000 #数据库连接超时时间,默认30秒，即30000
      connection-test-query: SELECT 1
      validation-timeout: 5000
4.精简代码中正常日志的写入
```
  
##五、 常用监控指令##
当前TCP连接的状态  
```code
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
LISTEN 12
SYN_RECV 1
ESTABLISHED 454
FIN_WAIT1 1
TIME_WAIT 5000
TIME_WAIT 状态的socket
netstat -ant|grep -i time_wait |wc -l
查看丢包情况
netstat -s|grep -iE LISTEN
    35206 times the listen queue of a socket overflowed                                                                                                                                                      
    39719 SYNs to LISTEN sockets dropped 

查看丢包日志
cat /var/log/messages (messages是最新的日志)
# 查看当前80端口的连接数
netstat -nat|grep -i "80"|wc -l
```

##六、压测##  

调整Jmeter的内存，调大压测能力，1024的整数倍  

bin目录下面，打开Jmeter.bat文件，设置内存  
```code
set HEAP=-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m
```
新建线程组=>新建http请求=>添加监听器(查看结果树、聚合报告)  

https://blog.51cto.com/u_13558591/4977210  
  
网络测试的专业工具（以下忽略）  

测试网络接口层和网络层，每秒可处理的网络包数PPS，内核自带的发包工具pktgen  

测试传输层的 TCP 和 UDP 吞吐量（BPS）、连接数以及延迟，可以用 iperf 或 netperf  

测试应用层吞吐量（BPS）、每秒请求数以及延迟等指标。可以用 wrk、ab 等工具  

扫描漏洞工具 awvs或者nessus都是很爽  

扫描服务器开放了哪些端口Zenmphttps://nmap.org/download#macosx
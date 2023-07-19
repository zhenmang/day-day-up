**1.安装php**  

yum install php php-devel  
  
  
**2.安装php常用拓展**  

yum install php-fpm php-mysql php-gd php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc
yum install php-mysql php-gd php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc  
  
  
**3、重启apache使php生效**  

apachectl restart  
  
  
**4.service httpd status确认服务是否已启动**

Active: running状态
  
    

**5.测试php环境**  

在/var/www/html目录(Apache默认根目录)中新建info.php
vi /var/www/html/info.php
文件内容
<?php phpinfo(); ?>
访问刚才的文件 curl http://localhost/info.php
返回html信息，证明服务可用  
  
  
**6.分站内容部署**  

将分站的文件拖到此目录下 /var/www/html/
浏览器访问http://13.229.243.187/，出现页面，说明环境搭建成功
  
  
  
附：
如下操作可更改目录配置
apache的配置文件/etc/httpd/conf/httpd.conf
检查配置文件是否有书写格式错误apachectl configtest
To prevent this page from ever being used, follow the instructions in the file /etc/httpd/conf.d/welcome.conf.
日志位于
/var/log/httpd/access_log
/var/log/httpd/error_log
查看进程 ps aux | grep httpd  

传输文件
sudo scp -r -i /Users/815355586qq.com/Downloads/站3/zxscapp/fen3.pem /Users/815355586qq.com/Downloads/zxscapp/ centos@13.229.243.187:/var/www/html  

传输文件被拒绝
如果报scp: /var/www/html/hxscapp: Permission denied
则在远端服务器上的目标目录下执行chmod -R 777 /var/www/html
chmod -R 777 /etc/httpd/conf  

访问文件目录被禁止
Forbidden
You don't have permission to access /hyycapp-andriod/index.php on this server.  
找到：apache文件，进入conf文件，打开httpd.conf 文件，做如下修改：
```code
<Directory />
    Options FollowSymLinks
    AllowOverride None
    Order deny,allow
    Deny from all
    Satisfy all
</Directory>

修改成

<Directory />
    Options FollowSymLinks
    AllowOverride None
    Order deny,allow
#    Deny from all
    Allow from all

#允许所有访问
    Satisfy all
</Directory>

第二处：找到：

#   onlineoffline tag - don't remove
    Order Deny,Allow
    Deny from all
    Allow from 127.0.0.1

</Directory>

修改成：

#   onlineoffline tag - don't remove
    Order Deny,Allow
#    Deny from all

#  Allow from 127.0.0.1
    Allow from all

</Directory>
```


配置资料（不用关注）
https://blog.csdn.net/weixin_30706691/article/details/98053873

php的证书配置
https://blog.csdn.net/janet1100/article/details/121855710

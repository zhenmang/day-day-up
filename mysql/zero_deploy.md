**1.查看当前linux版本**  

cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
是7.9版本  
  

**2.查看7版本的yum源**  

https://dev.mysql.com/downloads/repo/yum/
发现只有一个可以对应7版本 mysql80-community-release-el7-5.noarch.rpm
  

**3.依此执行以下脚本**  

wget http://repo.mysql.com/mysql80-community-release-el7-5.noarch.rpm
rpm -ivh mysql80-community-release-el7-5.noarch.rpm
（如果这里出现冲突的情况，可以rpm -qa |grep mysql 查看内容
进行对冲突的包卸载rpm -e --nodeps mysql-community-release-el7-5.noarch）
yum update
yum install mysql-server  
  

**4.权限设置：**  

chmod -R 777 /var/lib/mysql
chown -R mysql:mysql /var/lib/mysql/  
  

**5.初始化 MySQL：**  

mysqld --initialize  
  

**6.启动 MySQL：**  

systemctl start mysqld  
如果出现错误Job for mysqld.service failed because the control process exited with error code. See "systemctl status mysqld.service" and "journalctl -xe" for details
chmod -R 777 /var/lib/mysql  
  
  
**7.查看 MySQL 运行状态：**  

systemctl status mysqld
若显示激活，则为成功启动 Active: active (running)
如果报Error: 13 (Permission Denied)
就再执行一遍chown -R mysql:mysql /var/lib/mysql/
再重新启动systemctl start mysqld
再次查看状态systemctl status mysqld
若显示激活，则为成功启动 Active: active (running)  
  
  
**8.使用客户端执行简单命令，验证服务**  

输入mysql
（如果报ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)，则在文件/etc/my.cnf里，[mysqld]下添加skip-grant-tables）
再执行重启systemctl restart mysqlId即可
再次输入mysql
执行SHOW DATABASES;有返回值，则证明服务可用了。
执行exit退出  
  

**9.设置访问密码**  

执行mysqladmin -u root password "MySql@1237";（一定注意这行代码）  

**10.登录数据库**  

执行mysql -u root -p
如果忘记密码：
首先，vi my.conf，设置skip-grant-tables
在编辑完配置文件需要重启服务
sudo systemctl restart mysqld
  

使用 mysql -uroot -p 不输入密码登录
使用mysql库：use mysql;
修改root的host：update user set host = '%' where user = 'root'  and host='localhost';
修改密码为空：update user set authentication_string='' where user='root';
退出：exit
再次编辑：vi my.conf，注释调skip-grant-tables
使用mysql -uroot -p 不输入密码登录
会提示修改root密码
输入一下修改密码：alter user 'root'@'%' identified by 'MySql@123';  


**11.当对外安全组放开3306端口后**  

执行
mysql;
查看端口号
show global variables like 'port';
查看网络是否对外
show global variables like 'skip_networking';
如果发现端口变成0了，参考解决文章https://www.jianshu.com/p/a2b5a1d4a36a  
  

**12.同步库表数据**  

用 navicat进行库表结构同步【顶部工具】=>【结构同步】

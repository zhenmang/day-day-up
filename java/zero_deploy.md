**1.查看系统版本命令**  

cat /etc/issue

**2.查看yum包含的jdk版本**  

yum search java 或者 yum list java*

**3.全局下载安装java**  

查看系统信息，主要看架构，决定下载哪个架构的java
 uname -a
yum install -y java-11-openjdk.x86_64
卸载(存在这条指令，但是卸载不成功，不能用)
yum erase java 

**4.下载完后查看版本号**  

 java --version
如果版本不是目标版本，说明机器上还有其他java版本，可以通过以下指令查看并选择版本
alternatives --config java


**5.下载JDK**  

全局下载安装JDK，完了之后，可以用javac将.java文件编译成.class
sudo yum install java-deve


**6.拷贝前后端代码**  

将站1的生产代码包拷到站7
1.拷贝 admin.jar
scp -r /opt/app/api/admin/admin.jar root@52.76.18.25:/opt/app/api/admin/
2.拷贝mobile-api.jar
scp -r /opt/app/api/mobile-api/mobile-api.jar root@52.76.18.25:/opt/app/api/mobile-api/
3.拷贝管理后台vue
scp -r /opt/app/ui/dist/ root@52.76.18.25:/opt/app/ui/dist/
4.拷贝uniapp的H5
scp -r /opt/app/ui/h5/ root@52.76.18.25:/opt/app/ui/h5/
5.拷贝upload里面的那些图片资源
scp -r /opt/upload root@52.76.18.25:/opt/

**7.启动服务**  

启动mobile-app
nohup java -jar -Xms4g -Duser.timezone=GMT+8 ./mobile-api.jar --spring.profiles.active=prod --sys.swagger-domain=zyipa.com --sys.app-res-access-prefix=http://zyipa.com --server.port=19019 > mobile-api.log &

访问验证curl http://localhost:19019/api/user/sys/var

启动admin
nohup java -jar -Xms4g -Duser.timezone=GMT+8 ./admin.jar --spring.profiles.active=prod --sys.swagger-domain=hezaiapp.com --sys.app-res-access-prefix=http://hezaiapp.com --server.port=19020 > admin.log &

访问验证curl http://localhost:19020/admin/global/new/msg?passedSeconds=10

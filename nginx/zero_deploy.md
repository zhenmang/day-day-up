**1.配置nginx的yum源**  

vi /etc/yum.repos.d/nginx.repo
把以下内容复制进去，保存退出
[nginx-stable]

name=nginx stable repo

baseurl=http://nginx.org/packages/centos/$releasever/$basearch/

gpgcheck=1

enabled=1

gpgkey=https://nginx.org/keys/nginx_signing.key

[nginx-mainline]

name=nginx mainline repo

baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/

gpgcheck=1

enabled=0

gpgkey=https://nginx.org/keys/nginx_signing.key

**2.然后查看一下是否成功加载了这个安装源**  

yum repolist

**3.通过yum安装nginx**  

yum install nginx 

**4.将站1的nginx配置拷过来**  

scp -r /etc/nginx/nginx.conf root@52.76.18.23:/etc/nginx/


**5.其他nginx操作**  


A.操作nginx
nginx -t                    # 检查nginx.conf的内容书写是否合规
nginx -c nginx.conf  # 首次启动nginx
nginx -s reload         # 热更新

B.【不建议用】以下指令可操作nginx
systemctl enable nginx　　#设置nginx为开机启动  
systemctl start nginx　　#启动nginx服务

C.【不建议用】记得把防火墙关了 systemctl stop firewalld，输入服务器ip，发现ok了

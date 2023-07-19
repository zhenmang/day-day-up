1、查看redis版本列表 https://download.redis.io/releases/

2、下载需要的版本，传输到服务器

3、在Linux中, 将redis压缩包解压到指定的目录，这里是将redis解压到/opt文件夹下，可以使用-C指定到解压的文件夹
tar -zvxf /opt/package/redis-5.0.2.tar.gz -C /opt/package

4、解压后当前的目录出现一个redis-5.0.2的目录，就是我们刚刚解压的目录

5、由于redis是c语言编写的，所以我们需要先安装gcc，安装的命令如下：yum -y install gcc
安装成功后输入 :  gcc -v 查看版本

6、然后进入到redis目录，进入redis-5.0.2，然后输入：make，控制台会输出一编译的信息
cd /opt/package/redis-5.0.2

7、编译成功后，输入：make install,自此redis就安装成功了。

8、输入：redis-server 启动redis,启动成功。

9、关闭redis服务：
    通过redis-cli命令关闭：（正常用这个方式关闭）
    redis-cli shutdown

    通过kill命令：（暴力关闭，容易丢失数据）
    ps -ef|grep redis查看pid
    kill -9 pid

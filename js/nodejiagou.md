## 架构图

​	![](/imgs/nodejiagou.webp)

v8 引擎的作用就是将 js 转成 C++。

libuv 用于在C++中处理并发和进程构建，具有跨平台和异步能力。

c-ares：提供了异步处理 DNS 相关的能力。

http_parser、OpenSSL、zlib 等：提供包括 http 解析、SSL、数据压缩等其他的能力。

其中比较重要的两个库 V8和libuv

#### V8

谷歌开源的 JavaScript 引擎，目的是使 JavaScript 能够在浏览器之外的地方运行。前面说过，Javascript引擎是一个能够将 Javascript 语言转换成浏览器能够识别的低级语言或机器码的程序。

#### Libuv

C++的开源项目，使 Node 能够访问操作系统的底层文件系统（file system），访问网络（networking）并且处理一些高并发相关的问题。

## 代码语言构成

![](/imgs/goucheng.webp)

V8 大约70%由 C++ 实现，30%由 JavaScript 实现。
Libuv 100% 由 C++ 实现。

## 函数调用过程

![](/imgs/jieshiqi.webp)

![](/imgs/hanshudiaoyong.png)


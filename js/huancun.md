

数据库缓存：memcache和redis，避免频繁的数据查询。

服务器缓存：又称为*缓存代理服务器*或者*代理缓存*，是存在于客户端和源服务器之间的一种中间服务器，这种代理服务器接收多个客户端的数据请求，属于**公有缓存**，有效降低源服务器的负载压力。

客户端缓存：浏览器缓存（Browser Caching）是为了节约网络的资源加速浏览，浏览器在用户磁盘上对最近请求过的文档进行存储，当访问者再次请求这个页面时，浏览器就可以从本地磁盘显示文档，这样就可以加速页面的阅览。

### 浏览器缓存

- HTTP缓存（http1.0  http1.1）
- Web Storage （local storage sessionStorage）
- App Cache (manifest相关资源缓存，离线访问)
- IndexedDB (浏览器端的数据库)
- File System Api (h5新特性，读写本地文件)

其中，常用的是http缓存和web storage，重点是http缓存，如下:

### http1.0

1. `Pragma`: 是否使用缓存。可选值为`no-cache`，告诉浏览器在使用缓存前要发请求到服务器进行验证，不可直接使用缓存
2. `Expires`: 缓存的过期日期，单位为*秒*。值为一个具体的日期，例如`Tue, 27 Oct 2020 05:56:47 GMT`，告诉浏览器在这个过期日期之前都可以直接使用缓存

> Pragma为http1.0的产物，已逐步被http1.1的`cache-control:no-cache`替代，功能一致。http1.1响应首部中出现的`Pragma:no-cache`只是为了兼容http1.0，实际Cache-Control的优先级更高。
> Pragma和Expires同时存在时，Expires不会生效，即Pragma的优先级要高于Expires
> Expires的值是一个具体的时间点，这个时间点是相对于服务器的时间，如果客户端的时间和服务器时间不一致，比如手动修改客户机系统时间，那么这个“过期时间”的作用可能会出现偏差，或者说失效

### http1.1

`Cache-Control`用于控制缓存和缓存时间，`Last-Modified`和`Etag`用于缓存过期时，与服务器交互验证资源是否需要更新。

#### Cache-Control

**HTTP可缓存性包括:**

- public：HTTP请求返回时，经过的代理服务器以及客户端都可以对内容进行缓存。
- private：只有发起请求的浏览器可以进行缓存
- **no-cache**：本地和代理服务器可以缓存，**但是每次使用缓存时都要到服务器验证一下**，服务器返回可以使用缓存才能生效。

------

**缓存有效性**

max-age ：可以设置缓存的有效期

s-maxage：代理服务器的缓存有效期。同时设置max-age和s-maxage，客户端会使用max-age，代理服务器会使用s-maxage

max-stale：发起端设置，指明请求可以使用过期的缓存。（浏览器用不到）

**no-store**：本地和代理服务器都不能存储缓存，每次都到原服务器拿数据。

no-transform：不允许改动返回的内容（比如压缩、格式转换）。

------

**验证方面：**

must-revalidate：如果缓存过期，必须到服务器发送请求重新获取数据

proxy-revalidate：缓存服务器的must-revalidate

------

#### Last-Modified

**服务器返回Response**的上次修改时间，配合**客户端发送Request**的If-Modified-Since使用

------

#### Etag

通过数据签名标记这个资源，下次请求时配合Requesst的If-Match/If-Non-Match的对比缓存中的Etag判断是否使用缓存（数据签名：资源对它的内容会产生唯一的签名，如果内容修改，签名会进行修改，例如对内容hash计算）

------

**缓存头信息并不具有强约束性，只起声明作用。**



<br>

<br>

<br>

<br>

[web缓存](https://zhuanlan.zhihu.com/p/90507417)

[安全问题](https://zhuanlan.zhihu.com/p/30649102)
使用 Content-Length 字段的前提条件是，服务器发送回应之前，必须知道回应的数据长度。

对于一些很耗时的动态操作来说，这意味着，服务器要等到所有操作完成，才能发送数据，显然这样的效率不高。更好的处理方法是，产生一块数据，就发送一块，采用"流模式"（stream）取代"缓存模式"（buffer）。

因此，1.1 版规定可以不使用 Content-Length 字段，而使用"分块传输编码"（chunked transfer encoding）。只要请求或回应的头信息有 Transfer-Encoding 字段，就表明回应将由数量未定的数据块组成。

```
Transfer-Encoding: chunked
```

每个非空的数据块之前，会有一个 16 进制的数值，表示这个块的长度。最后是一个大小为 0 的块，就表示本次回应的数据发送完了。下面是一个例子。

```
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

25
This is the data in the first chunk

1C
and this is the second one

3
con

8
sequence

0
```

举个例子，有个用户需要在线看视频的场景，假定我们通过 HTTP 请求返回给用户电影内容，那么代码可能写成这样

```
const http = require('http');
const fs = require('fs');

http.createServer((req, res) => {
   fs.readFile(moviePath, (err, data) => {
      res.end(data);
   });
}).listen(8080);
```

这样的代码又两个明显的问题，

1. 电影文件需要读完之后才能返回给客户，等待时间超长
2. 电影文件需要一次放入内存中，相似动作多了，内存吃不消

用流可以讲电影文件一点点的放入内存中，然后一点点的返回给客户，用户体验得到优化，同时对内存的开销明显下降。

## 参考

- <https://www.ruanyifeng.com/blog/2016/08/http.html>
- <https://www.cloudflare.com/zh-cn/learning/video/what-is-http-live-streaming/>
- <https://zhuanlan.zhihu.com/p/24432941>

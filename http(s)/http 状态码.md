- 1xx：指示信息--表示请求已接收，继续处理
- 2xx：成功--表示请求已被成功接收、理解、接受
- 3xx：重定向--要完成请求必须进行更进一步的操作
- 4xx：客户端错误--请求有语法错误或请求无法实现
- 5xx：服务器端错误--服务器未能实现合法的请求

**301**，Moved Permanently 客户请求的文档在其他地方，新的 URL 在 Location 头中给出，浏览器应该自动地访问新的URL。

```
server
    {
        listen 80 default_server;
	    listen [::]:80 default_server;
        server_name ethsonliu.com www.ethsonliu.com;
        return 301 https://$server_name$request_uri;
    }
```

**404**

not found

**502** Bad Gateway

比如，nginx 访问下游服务端程序，但该服务程序没启动，就会返回 502

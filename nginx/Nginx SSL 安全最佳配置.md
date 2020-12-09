模板如下，参考自：https://gist.github.com/ethsonliu/ccb4fee261073be05b9d363da49cf64d

```conf
http {
	server_tokens off;
	add_header X-Frame-Options SAMEORIGIN;
	add_header X-Content-Type-Options nosniff;
	add_header X-XSS-Protection "1; mode=block";
	add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://ssl.google-analytics.com https://assets.zendesk.com https://connect.facebook.net; img-src 'self' https://ssl.google-analytics.com https://s-static.ak.facebook.com https://assets.zendesk.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://assets.zendesk.com; font-src 'self' https://themes.googleusercontent.com; frame-src https://assets.zendesk.com https://www.facebook.com https://s-static.ak.facebook.com https://tautt.zendesk.com; object-src 'none'";
}
```

```conf
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	server_name example.com;
	return 301 https://$host$request_uri;
}

server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name example.com;

	ssl_certificate /etc/nginx/ssl/example.com.crt;
	ssl_certificate_key /etc/nginx/ssl/example.com.key;

	ssl_session_cache shared:SSL:50m;
	ssl_session_timeout 1d;
	ssl_session_tickets off;
	
	ssl_dhparam /etc/nginx/ssl/dhparam.pem;
	
	ssl_prefer_server_ciphers on;
	
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	
	ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
  
	resolver 8.8.8.8 8.8.4.4;
	ssl_stapling on;
	ssl_stapling_verify on;
	ssl_trusted_certificate /root/.acme.sh/example.com/fullchain.cer;
	
	add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";

  	# ... 其他配置
}
```

以下是针对上述内容的解析，

### server_tokens

http 请求的 response 里面的 header 中，我们会发现有 server 这个参数，它表示服务端使用的是什么 web 服务器。

```
Server:nginx
Server:Tengine
```

很多网站不止返回了 nginx 而且还带了版本号，而像版本号这种东西完全没必要暴露给用户，我们可以通过设置 `server_tokens off` 隐藏掉版本号。

### X-Frame-Options

X-Frame-Options 是一个 HTTP header，用来告诉浏览器这个网页是否可以放在 iFrame/frame/object 内。它有三个可选值：

```
X-Frame-Options: DENY
X-Frame-Options: SAMEORIGIN
X-Frame-Options: ALLOW-FROM http://example.com
```

- DENY 告诉浏览器不要把这个网页放在 iFrame 内，通常的目的就是要帮助用户对抗点击劫持。
- SAMEORIGIN 告诉浏览器只有当架设 iFrame 的网站与发出 X-Frame-Options 的网站相同，才能显示发出 X-Frame-Options 网页的内容。
- ALLOW-FROM uri 告诉浏览器这个网页只能放在 http://example.com 网页架设的 iFrame 内。

不指定 X-Frame-Options 的网页等同表示它可以放在任何 iFrame 内。X-Frame-Options 可以保障你的网页不会被放在恶意网站设定的 iFrame 内，令用户成为点击劫持的受害人。

### X-Content-Type-Options

互联网上的资源有各种类型，通常浏览器会根据响应头的 Content-Type 字段来分辨它们的类型。例如："text/html" 代表 html 文档，"image/png" 是 PNG 图片，"text/css" 是 CSS 样式文档。然而，有些资源的 Content-Type 是错的或者未定义。这时，某些浏览器会启用 MIME-sniffing 来猜测该资源的类型，解析内容并执行。

例如，我们即使给一个 html 文档指定 Content-Type 为 "text/plain"，在 IE8 中这个文档依然会被当做 html 来解析。利用浏览器的这个特性，攻击者甚至可以让原本应该解析为图片的请求被解析为 JavaScript。通过下面这个响应头可以禁用浏览器的类型猜测行为：

```
X-Content-Type-Options: nosniff
```

### X-XSS-Protection

顾名思义，这个响应头是用来防范 XSS 的。现在主流浏览器都支持，并且默认都开启了 XSS 保护，用这个 header 可以关闭它。它有几种配置：

- 0：禁用 XSS 保护
- 1：启用 XSS 保护
- 1; mode=block：启用 XSS 保护，并在检查到 XSS 攻击时，停止渲染页面（例如 IE8 中，检查到攻击时，整个页面会被一个 # 替换）

浏览器提供的 XSS 保护机制并不完美，但是开启后仍然可以提升攻击难度，总之没有特别的理由，不要关闭它。

### Content-Security-Policy

Content Security Policy，简称 CSP。顾名思义，这个规范与内容安全有关，主要是用来定义页面可以加载哪些资源，减少 XSS 的发生。

### Strict-Transport-Security

HTTP Strict Transport Security，简称为 HSTS，它要求浏览器总是通过 HTTPS 来访问一个网站。

我们知道 HTTPS 相对于 HTTP 有更好的安全性，而很多 HTTPS 网站，也可以通过 HTTP 来访问。开发人员的失误或者用户主动输入地址，都有可能导致用户以 HTTP 访问网站，降低了安全性。一般，我们会通过 Web Server 发送 301/302 重定向来解决这个问题。现在有了 HSTS，可以让浏览器帮你做这个跳转，省一次 HTTP 请求。另外，浏览器本地替换可以保证只会发送 HTTPS 请求，避免被劫持。

要使用 HSTS，只需要在你的 HTTPS 网站响应头中，加入下面这行：

```
add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
```

includeSubDomains 是可选的，用来指定是否作用于子域名。

支持 HSTS 的浏览器遇到这个响应头，会把当前网站加入 HSTS 列表，然后在 max-age 指定的秒数内，当前网站所有请求都会被重定向为 https。

### ssl_session_cache

网站启用 https 后，会加剧服务器的负担。每次新的 TLS 连接都需要握手。不过，通过重用 Session 可以提高 https 的性能。

nginx 官方说只使用 shared，性能会更高，配置方法为：`ssl_session_cache shared:SSL:50m;`。

### ssl_dhparam 

openssl dhparam 用于生成和管理 dh 文件。dh（Diffie-Hellman）是著名的密钥交换协议，或称为密钥协商协议，它可以保证通信双方安全地交换密钥。注意，它不是加密算法，仅仅只是保护密钥交换的过程。在 openvpn 中就使用了该交换协议。

dh 文件生成方法，执行前确保目录存在，

```
openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

### ssl_prefer_server_ciphers

服务端加密算法优先。

### resolver

国外服务器建议 `resolver 8.8.8.8 8.8.4.4`。

国内服务器建议 `resolver 223.5.5.5 223.6.6.6`。

### ssl_stapling

上述例子中 ssl_certificate 使用的是 fullchain.pem，带了中间证书，所以只需要指定 resolver 即可，ssl_trusted_certificate 和 ssl_stapling_file 不用也可。

参考：

- <https://imququ.com/post/why-can-not-turn-on-ocsp-stapling.html>
- <https://imququ.com/post/optimize-tls-handshake.html>
- <https://blog.fundebug.com/2018/03/07/nginx_ocsp_stapling/>
- <https://c4ys.com/archives/2148>

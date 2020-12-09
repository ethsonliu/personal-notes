模板如下，参考自：https://gist.github.com/ethsonliu/ccb4fee261073be05b9d363da49cf64d

```conf
http {
	server_tokens off;
	add_header X-Frame-Options SAMEORIGIN;
	add_header X-Content-Type-Options nosniff;
	add_header X-XSS-Protection "1; mode=block";
	add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://ssl.google-analytics.com https://assets.zendesk.com https://connect.facebook.net; img-src 'self' https://ssl.google-analytics.com https://s-static.ak.facebook.com https://assets.zendesk.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://assets.zendesk.com; font-src 'self' https://themes.googleusercontent.com; frame-src https://assets.zendesk.com https://www.facebook.com https://s-static.ak.facebook.com https://tautt.zendesk.com; object-src 'none'";
	add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
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
	ssl_trusted_certificate /etc/nginx/ssl/star_forgott_com.crt;

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



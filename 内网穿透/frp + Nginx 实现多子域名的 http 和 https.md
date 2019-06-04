下面配置所使用的服务器是 CentOS 6，服务器 IP 对应的域名为 example.com。

## 一：实现 http

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/001.png)

在局域网内有大量设备，里边各运行一个 web 服务，现在希望如果用户在外地，可以用手机访问这些设备的 web 服务，自然用到内网穿透，此处选择 frp。

配置文件如下：

```ini
# frps.ini
[common]
bind_port = 7000
vhost_http_port = 8080
subdomain_host = example.com

dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
```

```ini
# frpc.ini
[common]
server_addr = example.com
server_port = 7000

[web]
type = http
local_ip = 127.0.0.1
local_port = 80

# 利用设备唯一标识 ID 来作为子域名的
subdomain = $ID
```

frp的启动命令分别为`./frps -c ./frps.ini`和`./frpc -c ./frpc.ini`，后台启动命令为`nohup ./frps -c frps.ini >/dev/null 2>&1 &`。

然后网页端访问`http://$ID.example.com:8080`即可访问内网的 web 服务。

## 二：实现 https

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/002.png)

现在又要求，启用访问加密，自然想到 https。

但是手机直接访问 frps 是没法使用 https，所以加了一层代理，让手机去连 Nginx，然后 Nginx 反向代理 frps，实现 https。

frps 和 frpc 的配置文件如下：

```ini
# frps.ini
[common]
bind_port = 7000
vhost_http_port = 8080
subdomain_host = example.com

dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
```

```ini
# frpc.ini
[common]
server_addr = example.com
server_port = 7000

# 加密
tls_enable = true

[web]
type = http
local_ip = 127.0.0.1
local_port = 80
subdomain = $ID
```

下面主要是域名证书的申请和 Nginx 的配置，

安装`Nginx`，

```shell
yum install nginx
```

启动它`service nginx start`。

**接着进行下面操作之前请先关闭防火墙**，命令是`service iptables stop`，

然后`cd`到`home`目录下，安装`certbot`，

```shell
wget https://dl.eff.org/certbot-auto

chmod a+x certbot-auto

# 注意这句的时候有几个注意点
# 1. 把下面的域名换成你自己的，对于我们而言，一定要是泛域名！
# 2. 下面的命令执行过程中，会要求你输入邮箱，是否同意它们的啥啥的，你按它的提示填，一直到有一步要你域名
#    DNS 解析那边，加入一个 TXT，详见下面。
./certbot-auto --server https://acme-v02.api.letsencrypt.org/directory -d "*.example.com" --manual --preferred-challenges dns-01 certonly
```

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/003.png)

要域名解析那边添加一条 TXT 记录，确认生效的命令是，`dig _acme-challenge.example.com txt`，如果报错没有`dig`命令，运行以下命令进行安装`yum install bind-utils`。如果成功再继续上面的步骤，不成功就再等等，等解析生效。

![](https://raw.githubusercontent.com/Hapoa/personal-notes/master/_image/004.png)

到了这一步就成功了，你的证书文件都放在一个目录下，它上面已经说了。

然后最后一步，修改文件`/etc/nginx/conf.d/default.conf`，可以直接复制进去（这个文件只要被更新，都需要重新加载配置文件，命令是`nginx -s reload`），

```
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    
    # 泛域名 http
    server_name  *.example.com;
	
    # http 强转 https
    rewrite ^(.*)$  https://$host$1 permanent;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}

server {

    # 多子域名，需要是泛域名证书
    server_name *.example.com;
    
    listen 443 http2 ssl;
    ssl on;
    
    # 存放域名证书位置
    ssl_certificate          /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key      /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate  /etc/letsencrypt/live/example.com/chain.pem;

    location / {
    
      # DNS解析地址，用的谷歌的，你也可以换成别家的
      resolver 8.8.8.8;
	  
      # 相当于向 frps 请求 http://$ID.example.com:8080
      proxy_pass http://$server_name:8080;
      
      # 剩下的不要动，直接复制
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header REMOTE-HOST $remote_addr;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      add_header X-Cache $upstream_cache_status;expires 12h;
    }
}
```

Nginx 重新加载配置文件后，先启动公网上的 frps，再启动位于内网的机器上的 frpc，然后浏览器输入`$ID.example.com`，会自动跳到`https://$ID.example.com`。

## 三：参考链接

- [申请Let's Encrypt通配符HTTPS证书](https://my.oschina.net/kimver/blog/1634575)
- [Nginx + Frp + Let'sEncrypt 泛域名证书](http://morecoder.com/article/1173275.html)
- [原文：记录一次迁移wss WebSocket的事故](https://blog.csdn.net/qq_28804275/article/details/80891921)
- [转载：记录一次迁移wss WebSocket的事故](https://blog.csdn.net/hfismyangel/article/details/82758629)
- [certbot-letencrypt-wildcardcertificates-alydns-au](https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au)

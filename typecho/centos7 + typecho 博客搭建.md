## 一：安装 nginx

```bash
yum install epel-release
yum install nginx
systemctl enable nginx.service
systemctl is-enabled nginx.service
systemctl start nginx
```

## 二：安装 mysql

参考 <https://github.com/ethsonliu/personal-notes/blob/master/%E6%95%B0%E6%8D%AE%E5%BA%93/mysql/CentOS%207%20%E5%AE%89%E8%A3%85%20MySQL%208.md>

不要安装版本 8。如果要安装，要注意在 `/etc/my.cnf` 中加入，

```
[mysqld]
default_authentication_plugin=mysql_native_password
```

然后重启。

## 三：安装 php

```bash
yum install epel-release
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum install php70w-common php70w-fpm php70w-opcache php70w-gd php70w-mysqlnd php70w-mbstring php70w-pecl-redis php70w-pecl-memcached php70w-devel

systemctl enable php-fpm
systemctl start php-fpm
```

## 四：安装 typecho

下载地址 <https://github.com/typecho/typecho>，安装见 <http://docs.typecho.org/install>，使用 typecho 的 master 分支即可。

注意，要现在 mysql 里创建数据库。

## 五：typecho 安全加固

### 修改 admin 目录

打开 config.inc.php，建议用随机密码生成器生成，注意不要有特殊字符，大小写字母和数字即可。

```
/** 后台路径(相对路径) */
define('__TYPECHO_ADMIN_DIR__', '/xxxxxxx---admin/');
```

### 删除安装页面

把 install.php 删掉

### 屏蔽 usr、var 目录下 php 文件的访问

屏蔽 usr、var 目录下 php 文件的访问可以阻止黑客访问到他上传的 php 木马，我们同时屏蔽 config.inc.php 的访问。

屏蔽原理就是把要屏蔽的请求重定向到首页文件，首页文件会当成文章名来解析，没有同名文章就会返回 404 未找到，所以就算黑客上传了木马也只会得到 404 未找到的响应。

在 nginx 的 server 中加入

```
    if (!-e $request_filename) {
        rewrite ^(.*)$ /index.php$1 last;
    }

    rewrite /(var|usr)(.+ph*)$ /index.php;
    rewrite /(config.inc.php|.htaccess)$ /index.php last;
```

## 五：服务器永久禁止 ICMP

打开文件 /etc/sysctl.conf，加入

```
net.ipv4.icmp_echo_ignore_all=1
```

然后执行命令 `sysctl -p` 生效。

## 六：防火墙

参考 <https://github.com/ethsonliu/personal-notes/blob/master/linux/Linux%20常用命令.md#防火墙>

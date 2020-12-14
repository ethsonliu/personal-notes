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

## 三：安装 php

```bash
yum -y install epel-release
yum -y install php php-fpm
php -v
yum install php-mysql
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

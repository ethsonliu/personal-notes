## 一：安装 nginx

```bash
sudo yum install epel-release
sudo yum install nginx
sudo systemctl start nginx
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

下载地址 <https://github.com/typecho/typecho>，安装见 <http://docs.typecho.org/install>

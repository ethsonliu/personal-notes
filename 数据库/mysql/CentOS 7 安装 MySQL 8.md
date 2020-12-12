进入 <http://repo.mysql.com/>，里面包含了所有可用的 MySQL 源。选择一个合适的版本，进行下载：

```shell
wget http://repo.mysql.com/mysql80-community-release-el7.rpm

## 5.7 版本
## wget http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

完成之后，进行安装：

```shell
# 版本 8.0
rpm -ivh mysql80-community-release-el7.rpm

# 版本 5.7
rpm -ivh mysql57-community-release-el7-11.noarch.rpm
```

开始安装 MySQL：

```shell
yum install mysql
yum install mysql-server
yum install mysql-devel
```

mysql 是 MySQL 客户端，mysql-server 是数据库服务器，mysql-devel 则包含了开发用到的库以及头文件。

这步可能会花些时间，安装完成后就会覆盖掉之前的 mariadb。

然后，mysql 的启动、停止、查看状态，

```shell
service mysqld start/stop/status
```

接着想要登录数据库，需要查看初始密码，

```shell
grep "password" /var/log/mysqld.log
```

接着登录，

```shell
mysql -u root -p
```

输入密码，即可进去，最好修改下密码，初始密码太繁琐，因为 mysql 8 启用了更高级的密码等级，所以我们可以把它设置的简单一点，先输入命令，

```shell
mysql> set global validate_password.policy=LOW;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password.length=6;
Query OK, 0 rows affected (0.00 sec)
```

接着可以查看是否生效，

```shell
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 5      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.00 sec)
```

然后就可以设置一个简单点的密码了，

```shell
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '你的密码';
```

默认只能本地登录，可以修改为本地+外部都可登录，

```
use mysql;
select host, user from user;
update user set host = '%' where user = 'root';
select host, user from user;
```

## 参考：

https://blog.csdn.net/liang19890820/article/details/81672538

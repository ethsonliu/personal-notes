#### 1 客户端登录 mysql 8 server 时因为'caching_sha2_password'问题连接被拒绝

 登录 mysql，

```
mysql -u root -p
```

输入密码进入。（请确保 mysql 已经运行，`service mysql start/status/restart`）

修改加密规则，注意，如果你有将localhost改为%，也就是有执行下面第2个问题的命令，则下面的命令里的localhost也要改为%）

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
```

更新密码（mysql_native_password模式），

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';
```

#### 2 远程连接 MYSQL 提示Host is not allowed to connect to this MySQL server

可能是你的帐号不允许从远程登陆，只能在 localhost。这个时候只要在 localhost 的那台电脑，登入mysql后，更改 "mysql" 数据库里的 "user" 表里的 "host" 项，从"localhost"改称"%"，

```
use mysql;
update user set host = '%' where user = 'root';
select host, user from user;
```
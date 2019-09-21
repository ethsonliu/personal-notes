本地装好 mysql，设置好密码，参照 <https://docs.emqx.io/broker/v3/cn/plugins.html#mysql>，创建好数据库和表，

打开`/etc/emqx/emqx.conf`，

```ini
## Allow anonymous authentication by default if no auth plugins loaded.
## Notice: Disable the option in production deployment!
##
## Value: true | false
allow_anonymous = false

## Allow or deny if no ACL rules matched.
##
## Value: allow | deny
acl_nomatch = deny
```

打开`/etc/emqx/plugins/emqx_auth_mysql.conf`，

```ini
##--------------------------------------------------------------------
## MySQL Auth/ACL Plugin
##--------------------------------------------------------------------

## MySQL server address.
##
## Value: Port | IP:Port
##
## Examples: 3306, 127.0.0.1:3306, localhost:3306
auth.mysql.server = 127.0.0.1:3306

## MySQL pool size.
##
## Value: Number
auth.mysql.pool = 8

## MySQL username.
##
## Value: String
auth.mysql.username = root

## MySQL password.
##
## Value: String
auth.mysql.password = 你的密码

## MySQL database.
##
## Value: String
auth.mysql.database = mqtt

## Variables: %u = username, %c = clientid

## Authentication query.
##
## Note that column names should be 'password' and 'salt' (if used).
## In case column names differ in your DB - please use aliases,
## e.g. "my_column_name as password".
##
## Value: SQL
##
## Variables:
##  - %u: username
##  - %c: clientid
##  - %cn: common name of client TLS cert
##  - %dn: subject of client TLS cert
##
auth.mysql.auth_query = select password from mqtt_user where username = '%u' limit 1
## auth.mysql.auth_query = select password_hash as password from mqtt_user where username = '%u' limit 1

## Password hash.
##
## Value: plain | md5 | sha | sha256 | bcrypt
auth.mysql.password_hash = sha256

## sha256 with salt prefix
## auth.mysql.password_hash = salt,sha256

## bcrypt with salt only prefix
## auth.mysql.password_hash = salt,bcrypt

## sha256 with salt suffix
## auth.mysql.password_hash = sha256,salt

## pbkdf2 with macfun iterations dklen
## macfun: md4, md5, ripemd160, sha, sha224, sha256, sha384, sha512
## auth.mysql.password_hash = pbkdf2,sha256,1000,20

## Superuser query.
##
## Value: SQL
##
## Variables:
##  - %u: username
##  - %c: clientid
##  - %cn: common name of client TLS cert
##  - %dn: subject of client TLS cert
##
auth.mysql.super_query = select is_superuser from mqtt_user where username = '%u' limit 1

## ACL query.
##
## Value: SQL
##
## Variables:
##  - %a: ipaddr
##  - %u: username
##  - %c: clientid
##
## Note: You can add the 'ORDER BY' statement to control the rules match order
auth.mysql.acl_query = select allow, ipaddr, username, clientid, access, topic from mqtt_acl where ipaddr = '%a' or username = '%u' or username = '$all' or clientid = '%c'
```

然后，进入终端执行，

```shell
## 当时一直用的 load，但是总是失败，后来改用 reload 之后就一切正常了，很奇怪。
$ emqx_ctl plugins reload emqx_auth_mysql
```

接下来可以测试一下这个插件，数据表`mqtt_user`插入一条，

|  id  |   username    |   password    | salt | is_superuser | created |
| :--: | :-----------: | :-----------: | :--: | :----------: | :-----: |
|  1   | your_username | your_password |      |      0       |         |

数据表`mqtt_acl`插入一条规则，

|  id  | allow | ipaddr | username | clientid | access | topic |
| :--: | :---: | :----: | :------: | :------: | :----: | :---: |
|  1   |   0   |        |   $all   |          |   2    | acl/# |

上面规则的含义是：禁止所有用户发布主题符合`acl/#`的内容。

有几点注意：

- 主题支持通配符匹配，这点很棒。
- 可动态生效，修改数据库后，不需要重新加载插件或重新开启 emqx。
- 数据库内容被修改后的生效需要秒量级的时间。

然后用 mqtt.fx 模拟，User Credentials 加入你的用户名和密码（如果不加是登录不上的）。
- [HTTPS 篇之 SSL 握手过程详解](https://razeencheng.com/post/ssl-handshake-detail)
- [安全背后: 浏览器是如何校验证书的](https://cjting.me/2021/03/02/how-to-validate-tls-certificate/)
- [SSL 双向认证和 SSL 单向认证的区别](https://www.jianshu.com/p/fb5fe0165ef2)

证书包含：签发者、证书用途、公钥、加密算法、hash 算法、签发和到期时间...私钥在 CA 手中。

如果证书就这样分发，容易被别人篡改。把以上内容做一次 hash，得到一个固定长度（比如 128 位），然后用 CA 的私钥加密，得到的就是**数字签名**，附在证书上。


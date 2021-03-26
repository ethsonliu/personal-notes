- [HTTPS 篇之 SSL 握手过程详解](https://razeencheng.com/post/ssl-handshake-detail)

证书包含：签发者、证书用途、公钥、加密算法、hash 算法、签发和到期时间...私钥在 CA 手中。

如果证书就这样分发，容易被别人篡改。把以上内容做一次 hash，得到一个固定长度（比如 128 位），然后用 CA 的私钥加密，得到的就是**数字签名**，附在证书上。


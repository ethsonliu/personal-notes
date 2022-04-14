双向认证的关键点在以下几个函数（服务端和客户端都一样），其他就不细说参看代码注释：

SSL_CTX_set_verify----配置启用双向认证

SSL_CTX_load_verify_locations----加载信任的根证书

SSL_CTX_use_certificate_file----加载自己的证书

SSL_CTX_use_PrivateKey_file----加载自己的私钥

SSL_get_peer_certificate 和 SSL_get_verify_result----真正进行验证，一定要调用这个函数不然前面四个光配置而已并不会进行双向验证（**这地方说的并不对，详见 https://www.openssl.org/docs/man3.0/man3/SSL_connect.html 的描述**）

## 服务端

```c++
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <openssl/ssl.h>
#include <openssl/err.h>

#define MAXBUF 1024

void ShowCerts(SSL * ssl)
{
    X509 *cert;
    char *line;

    cert = SSL_get_peer_certificate(ssl);
    // SSL_get_verify_result()是重点，SSL_CTX_set_verify()只是配置启不启用并没有执行认证，调用该函数才会真证进行证书认证
    // 如果验证不通过，那么程序抛出异常中止连接
    if(SSL_get_verify_result(ssl) == X509_V_OK){
        printf("证书验证通过\n");
    }
    if (cert != NULL) {
        printf("数字证书信息:\n");
        line = X509_NAME_oneline(X509_get_subject_name(cert), 0, 0);
        printf("证书: %s\n", line);
        free(line);
        line = X509_NAME_oneline(X509_get_issuer_name(cert), 0, 0);
        printf("颁发者: %s\n", line);
        free(line);
        X509_free(cert);
    } else
        printf("无证书信息！\n");
}

int main(int argc, char **argv) {
    int sockfd, new_fd;
    socklen_t len;
    struct sockaddr_in my_addr, their_addr;
    unsigned int myport, lisnum;
    char buf[MAXBUF + 1];
    SSL_CTX *ctx;

    if (argv[1])
        myport = atoi(argv[1]);
    else
        myport = 7838;

    if (argv[2])
        lisnum = atoi(argv[2]);
    else
        lisnum = 2;

    /* SSL 库初始化 */
    SSL_library_init();
    /* 载入所有 SSL 算法 */
    OpenSSL_add_all_algorithms();
    /* 载入所有 SSL 错误消息 */
    SSL_load_error_strings();
    /* 以 SSL V2 和 V3 标准兼容方式产生一个 SSL_CTX ，即 SSL Content Text */
    ctx = SSL_CTX_new(SSLv23_server_method());
    /* 也可以用 SSLv2_server_method() 或 SSLv3_server_method() 单独表示 V2 或 V3标准 */
    if (ctx == NULL) {
        ERR_print_errors_fp(stdout);
        exit(1);
    }

    // 双向验证
    // SSL_VERIFY_PEER---要求对证书进行认证，没有证书也会放行
    // SSL_VERIFY_FAIL_IF_NO_PEER_CERT---要求客户端需要提供证书，但验证发现单独使用没有证书也会放行
    SSL_CTX_set_verify(ctx, SSL_VERIFY_PEER|SSL_VERIFY_FAIL_IF_NO_PEER_CERT, NULL);
    // 设置信任根证书
    if (SSL_CTX_load_verify_locations(ctx, "ca.crt",NULL)<=0){
        ERR_print_errors_fp(stdout);
        exit(1);
    }

    /* 载入用户的数字证书， 此证书用来发送给客户端。 证书里包含有公钥 */
    if (SSL_CTX_use_certificate_file(ctx, argv[3], SSL_FILETYPE_PEM) <= 0) {
        ERR_print_errors_fp(stdout);
        exit(1);
    }
    /* 载入用户私钥 */
    if (SSL_CTX_use_PrivateKey_file(ctx, argv[4], SSL_FILETYPE_PEM) <= 0) {
        ERR_print_errors_fp(stdout);
        exit(1);
    }
    /* 检查用户私钥是否正确 */
    if (!SSL_CTX_check_private_key(ctx)) {
        ERR_print_errors_fp(stdout);
        exit(1);
    }

    /* 开启一个 socket 监听 */
    if ((sockfd = socket(PF_INET, SOCK_STREAM, 0)) == -1) {
        perror("socket");
        exit(1);
    } else
        printf("socket created\n");

    bzero(&my_addr, sizeof(my_addr));
    my_addr.sin_family = PF_INET;
    my_addr.sin_port = htons(myport);
    my_addr.sin_addr.s_addr = INADDR_ANY;

    if (bind(sockfd, (struct sockaddr *) &my_addr, sizeof(struct sockaddr))
            == -1) {
        perror("bind");
        exit(1);
    } else
        printf("binded\n");

    if (listen(sockfd, lisnum) == -1) {
        perror("listen");
        exit(1);
    } else
        printf("begin listen\n");

    while (1) {
        SSL *ssl;
        len = sizeof(struct sockaddr);
        /* 等待客户端连上来 */
        if ((new_fd = accept(sockfd, (struct sockaddr *) &their_addr, &len))
                == -1) {
            perror("accept");
            exit(errno);
        } else
            printf("server: got connection from %s, port %d, socket %d\n",
                    inet_ntoa(their_addr.sin_addr), ntohs(their_addr.sin_port),
                    new_fd);

        /* 基于 ctx 产生一个新的 SSL */
        ssl = SSL_new(ctx);
        /* 将连接用户的 socket 加入到 SSL */
        SSL_set_fd(ssl, new_fd);
        /* 建立 SSL 连接 */
        if (SSL_accept(ssl) == -1) {
            perror("accept");
            close(new_fd);
            break;
        }
        ShowCerts(ssl);

        /* 开始处理每个新连接上的数据收发 */
        bzero(buf, MAXBUF + 1);
        strcpy(buf, "server->client");
        /* 发消息给客户端 */
        len = SSL_write(ssl, buf, strlen(buf));

        if (len <= 0) {
            printf("消息'%s'发送失败！错误代码是%d，错误信息是'%s'\n", buf, errno,
                    strerror(errno));
            goto finish;
        } else
            printf("消息'%s'发送成功，共发送了%d个字节！\n", buf, len);

        bzero(buf, MAXBUF + 1);
        /* 接收客户端的消息 */
        len = SSL_read(ssl, buf, MAXBUF);
        if (len > 0)
            printf("接收消息成功:'%s'，共%d个字节的数据\n", buf, len);
        else
            printf("消息接收失败！错误代码是%d，错误信息是'%s'\n",
            errno, strerror(errno));
        /* 处理每个新连接上的数据收发结束 */
        finish:
        /* 关闭 SSL 连接 */
        SSL_shutdown(ssl);
        /* 释放 SSL */
        SSL_free(ssl);
        /* 关闭 socket */
        close(new_fd);
    }
    /* 关闭监听的 socket */
    close(sockfd);
    /* 释放 CTX */
    SSL_CTX_free(ctx);
    return 0;
}
```

## 客户端

```c++
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <sys/socket.h>
#include <resolv.h>
#include <stdlib.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <openssl/ssl.h>
#include <openssl/err.h>

#define MAXBUF 1024

void ShowCerts(SSL * ssl)
{
    X509 *cert;
    char *line;

    cert = SSL_get_peer_certificate(ssl);
    // SSL_get_verify_result()是重点，SSL_CTX_set_verify()只是配置启不启用并没有执行认证，调用该函数才会真证进行证书认证
    // 如果验证不通过，那么程序抛出异常中止连接
    if(SSL_get_verify_result(ssl) == X509_V_OK){
        printf("证书验证通过\n");
    }
    if (cert != NULL) {
        printf("数字证书信息:\n");
        line = X509_NAME_oneline(X509_get_subject_name(cert), 0, 0);
        printf("证书: %s\n", line);
        free(line);
        line = X509_NAME_oneline(X509_get_issuer_name(cert), 0, 0);
        printf("颁发者: %s\n", line);
        free(line);
        X509_free(cert);
    } else
        printf("无证书信息！\n");
}

int main(int argc, char **argv)
{
    int sockfd, len;
    struct sockaddr_in dest;
    char buffer[MAXBUF + 1];
    SSL_CTX *ctx;
    SSL *ssl;

    if (argc != 5) {
        printf("参数格式错误！正确用法如下：\n\t\t%s IP地址 端口\n\t比如:\t%s 127.0.0.1 80\n此程序用来从某个"
             "IP 地址的服务器某个端口接收最多 MAXBUF 个字节的消息",
             argv[0], argv[0]);
        exit(0);
    }

    /* SSL 库初始化，参看 ssl-server.c 代码 */
    SSL_library_init();
    OpenSSL_add_all_algorithms();
    SSL_load_error_strings();
    ctx = SSL_CTX_new(SSLv23_client_method());
    if (ctx == NULL) {
        ERR_print_errors_fp(stdout);
        exit(1);
    }

    // 双向验证
    // SSL_VERIFY_PEER---要求对证书进行认证，没有证书也会放行
    // SSL_VERIFY_FAIL_IF_NO_PEER_CERT---要求客户端需要提供证书，但验证发现单独使用没有证书也会放行
    SSL_CTX_set_verify(ctx, SSL_VERIFY_PEER|SSL_VERIFY_FAIL_IF_NO_PEER_CERT, NULL);
    // 设置信任根证书
    if (SSL_CTX_load_verify_locations(ctx, "ca.crt",NULL)<=0){
        ERR_print_errors_fp(stdout);
        exit(1);
    }

    /* 载入用户的数字证书， 此证书用来发送给客户端。 证书里包含有公钥 */
    if (SSL_CTX_use_certificate_file(ctx, argv[3], SSL_FILETYPE_PEM) <= 0) {
        ERR_print_errors_fp(stdout);
        exit(1);
    }
    /* 载入用户私钥 */
    if (SSL_CTX_use_PrivateKey_file(ctx, argv[4], SSL_FILETYPE_PEM) <= 0) {
        ERR_print_errors_fp(stdout);
        exit(1);
    }
    /* 检查用户私钥是否正确 */
    if (!SSL_CTX_check_private_key(ctx)) {
        ERR_print_errors_fp(stdout);
        exit(1);
    }

    /* 创建一个 socket 用于 tcp 通信 */
    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("Socket");
        exit(errno);
    }
    printf("socket created\n");

    /* 初始化服务器端（对方）的地址和端口信息 */
    bzero(&dest, sizeof(dest));
    dest.sin_family = AF_INET;
    dest.sin_port = htons(atoi(argv[2]));
    if (inet_aton(argv[1], (struct in_addr *) &dest.sin_addr.s_addr) == 0) {
        perror(argv[1]);
        exit(errno);
    }
    printf("address created\n");

    /* 连接服务器 */
    if (connect(sockfd, (struct sockaddr *) &dest, sizeof(dest)) != 0) {
        perror("Connect ");
        exit(errno);
    }
    printf("server connected\n");

    /* 基于 ctx 产生一个新的 SSL */
    ssl = SSL_new(ctx);
    SSL_set_fd(ssl, sockfd);
    /* 建立 SSL 连接 */
    if (SSL_connect(ssl) == -1)
        ERR_print_errors_fp(stderr);
    else {
        printf("Connected with %s encryption\n", SSL_get_cipher(ssl));
        ShowCerts(ssl);
    }

    /* 接收对方发过来的消息，最多接收 MAXBUF 个字节 */
    bzero(buffer, MAXBUF + 1);
    /* 接收服务器来的消息 */
    len = SSL_read(ssl, buffer, MAXBUF);
    if (len > 0)
        printf("接收消息成功:'%s'，共%d个字节的数据\n",
               buffer, len);
    else {
        printf
            ("消息接收失败！错误代码是%d，错误信息是'%s'\n",
             errno, strerror(errno));
        goto finish;
    }
    bzero(buffer, MAXBUF + 1);
    strcpy(buffer, "from client->server");
    /* 发消息给服务器 */
    len = SSL_write(ssl, buffer, strlen(buffer));
    if (len < 0)
        printf
            ("消息'%s'发送失败！错误代码是%d，错误信息是'%s'\n",
             buffer, errno, strerror(errno));
    else
        printf("消息'%s'发送成功，共发送了%d个字节！\n",
               buffer, len);

  finish:
    /* 关闭连接 */
    SSL_shutdown(ssl);
    SSL_free(ssl);
    close(sockfd);
    SSL_CTX_free(ctx);
    return 0;
}
```

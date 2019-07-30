```c
void* thread_http(void*)
{
	// 1. 获取该连接的服务器域名
	struct sockaddr_in address;
	char hostname[] = "example.com";
	struct hostent* host = gethostbyname(hostname);
	int port = 80;
	char* ip;

	if (host == NULL)
	{
		printf("----->%s can't get it's ip\n", hostname);
		return 0;
	}

	else
	{
		ip = inet_ntoa(*(struct in_addr*)host->h_addr_list[0]);
		printf("------>get ip=%s by domain=%s\n", ip, hostname);
	}

	int sockfd = socket(AF_INET, SOCK_STREAM, 0);
	address.sin_family = AF_INET;
	address.sin_addr.s_addr = inet_addr(ip);
	address.sin_port = htons(port);

	int result = connect(sockfd, (struct sockaddr *)&address, sizeof(address));

	if (result == -1)
	{
		printf("---->connect %s:%d failed\n", ip, port);
		perror("------->");
		close(sockfd);
		return 0;
	}
	else
	{
		printf("connect server %s:%d success\n", ip, port);
	}

	char send_buf[256];
	memset(send_buf, 0, sizeof(send_buf));

	strcat(send_buf, "GET /qrlogin HTTP/1.1\n");
	strcat(send_buf, "Host: ");
	strcat(send_buf, hostname);
	strcat(send_buf, "\n\r\n");
    
	int n = write(sockfd, send_buf, strlen(send_buf));
	printf("------>write byts len=%d, send_buf=\n%s\n", n, send_buf);

	char recv_buf[1024];
	memset(recv_buf, 0, sizeof(recv_buf));
	n = read(sockfd, recv_buf, sizeof(recv_buf));
	printf("------>recv len =%d, recv=%s\n", n, recv_buf);

	string domain;

	close(sockfd);
    ...
}
```

```c++
static void SendFrpcState(int sockfd, int isSuccess, const string& domain, const string& hostname, int port, const string& pn)
{
    static int failedInfoSent = 0;
    static string prevDomain = "";

    string headers = "state=";
    headers += uToString(isSuccess);
    headers += "&machineCode=";
    headers += pn;
    headers += "&domain=";
    headers += domain;

    string send_buf;
    send_buf.reserve(1024);
    send_buf = "POST /api/machine/linkState HTTP/1.1\n";
    send_buf += "Host: ";
    send_buf += hostname;
    send_buf += ":";
    send_buf += uToString(port);
    send_buf += "\nContent-Type: application/x-www-form-urlencoded\n";
    send_buf += "Content-Length: ";
    send_buf += uToString(headers.size());
    send_buf += "\n\n";
    send_buf += headers;
    send_buf += "\r\n\r\n";

    write(sockfd, send_buf.c_str(), send_buf.size());
    printf("[frpc] send the result<%d> of exec-frpc to remote server, remote server will write it into databse\n", isSuccess);

    char recv_buf[2048];
    memset(recv_buf, 0, sizeof(recv_buf));
    read(sockfd, recv_buf, sizeof(recv_buf)); // 服务器数据库是否成功存储

    char data[4];
    memset(data, 0, sizeof(data));
    char* error = strstr(recv_buf, "error");
    if (error == NULL) // no error
    {
        data[0] = '1';
        printf("[frpc] the remote server success when writing result<%d> into databse\n", isSuccess);
    }
    else
    {
        data[0] = '0';
        printf("[frpc] the remote server failed when writing result<%d> into databse\n", isSuccess);
    }

    // 防止重复发送信息
    if (isSuccess == 0 && failedInfoSent == 1 && prevDomain == domain)
    {
        printf("[frpc] skip 0x11 send\n");
        return;
    }
    else
        printf("[frpc] 0x11 send\n");

    UiCommunicationSend(0x11, data, strlen(data));

    if (isSuccess == 0)
    {
        failedInfoSent = 1;
        prevDomain = domain;
    }
}
```

## 参考链接

- CSDN . [基于openssl的https client例子](https://blog.csdn.net/jun2016425/article/details/78827670)
- CSDN . [用C语言实现一个简单的HTTP客户端（HTTP Client）](https://blog.csdn.net/gobitan/article/details/1551186)
- 博客园 . [C++：C语言实现HTTP的GET和POST请求](https://www.cnblogs.com/diligenceday/p/6255788.html)

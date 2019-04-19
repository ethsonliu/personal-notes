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

## 参考链接

- CSDN . [基于openssl的https client例子](https://blog.csdn.net/jun2016425/article/details/78827670)
- CSDN . [用C语言实现一个简单的HTTP客户端（HTTP Client）](https://blog.csdn.net/gobitan/article/details/1551186)
- 博客园 . [[C++：C语言实现HTTP的GET和POST请求](https://www.cnblogs.com/diligenceday/p/6255788.html)](https://www.cnblogs.com/diligenceday/p/6255788.html)
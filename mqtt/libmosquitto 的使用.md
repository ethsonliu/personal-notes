开源地址：https://github.com/eclipse/mosquitto

Ubuntu 运行`sudo apt-get install mosquitto`就可以安装可执行程序，有四个 exe 会被安装好，

```
mosquitto : mqtt broker
mosquitto_passwd : 管理 mosquitto 密码文件的命令行工具
mosquitto_sub : mqtt 订阅者程序
mosquitto_pub ： mqtt 发布者程序
```

相关的配置文件安装在 /etc/mosquitto/ 目录下。

现在测试一下客户端和服务端程序。为了测试方便，将客户端和服务端程序都在本机，使用 localhost 连接。执行 `mosquitto -v` 启动 broker ，-v 参数表示打印出运行信息，可以看到默认使用的端口是1883 ，

```
1558493506: mosquitto version 1.6.0 starting
1558493506: Using default config.
1558493506: Opening ipv4 listen socket on port 1883.
1558493506: Opening ipv6 listen socket on port 1883.
```

如果你的系统出现如下问题，就需要添加一个 mosquitto 用户：

```
Error: Invalid user mosquitto 
```

就执行`useradd mosquitto`

然后在第二个终端启动订阅者程序: `mosquitto_sub -h localhost -t test -v`，用 -h 参数指定服务器 IP ，用 -t 参数指定订阅的话题。

在第三个终端启动发布者程序: `mosquitto_pub -h localhost -t test -m "Hello world"`，用 -m 参数指定要发布的信息内容，然后在订阅者的终端就可以看到由 broker 推送的信息。

进行开发，

```c++
#include <stdio.h>
#include <stdlib.h>
#include <mosquitto.h>
#include <string.h>

#define HOST "localhost"
#define PORT  1883
#define KEEP_ALIVE 60

bool session = true;

void my_message_callback(struct mosquitto *mosq, void *userdata, const struct mosquitto_message *message)
{
    if(message->payloadlen){
        printf("%s %s", message->topic, message->payload);
    }else{
        printf("%s (null)\n", message->topic);
    }
    fflush(stdout);
}

void my_connect_callback(struct mosquitto *mosq, void *userdata, int result)
{
    int i;
    if(!result){
        /* Subscribe to broker information topics on successful connect. */
        mosquitto_subscribe(mosq, NULL, "Gai爷:", 2);
    }else{
        fprintf(stderr, "Connect failed\n");
    }
}

void my_subscribe_callback(struct mosquitto *mosq, void *userdata, int mid, int qos_count, const int *granted_qos)
{
    int i;
    printf("Subscribed (mid: %d): %d", mid, granted_qos[0]);
    for(i=1; i<qos_count; i++){
        printf(", %d", granted_qos[i]);
    }
    printf("\n");
}

void my_log_callback(struct mosquitto *mosq, void *userdata, int level, const char *str)
{
    /* Pring all log messages regardless of level. */
    printf("%s\n", str);
}

int main()
{
    struct mosquitto *mosq = NULL;
    //libmosquitto 库初始化
    mosquitto_lib_init();
    //创建mosquitto客户端
    mosq = mosquitto_new(NULL,session,NULL); // 第一个参数指定client id，为空则为随机，注意如果两个mosq实例以相同的client id同时连接相同的mqtt broker，两个实例都会不停的断线重连
    if(!mosq){
        printf("create client failed..\n");
        mosquitto_lib_cleanup();
        return 1;
    }
    //设置回调函数，需要时可使用
    //mosquitto_log_callback_set(mosq, my_log_callback);
    mosquitto_connect_callback_set(mosq, my_connect_callback);
    mosquitto_message_callback_set(mosq, my_message_callback);
    //mosquitto_subscribe_callback_set(mosq, my_subscribe_callback);
    //客户端连接服务器
    if(mosquitto_connect(mosq, HOST, PORT, KEEP_ALIVE)){
        fprintf(stderr, "Unable to connect.\n");
        return 1;
    }
    //循环处理网络消息
    mosquitto_loop_forever(mosq, -1, 1);

    mosquitto_destroy(mosq);
    mosquitto_lib_cleanup();

    return 0;
}
```

```c++
#include <stdio.h>
#include <stdlib.h>
#include <mosquitto.h>
#include <string.h>

#define HOST "localhost"
#define PORT  1883
#define KEEP_ALIVE 60
#define MSG_MAX_SIZE  512

bool session = true;

int main()
{
    char buff[MSG_MAX_SIZE];
    struct mosquitto *mosq = NULL;
    //libmosquitto 库初始化
    mosquitto_lib_init();
    //创建mosquitto客户端
    mosq = mosquitto_new(NULL,session,NULL);
    if(!mosq){
        printf("create client failed..\n");
        mosquitto_lib_cleanup();
        return 1;
    }
    //连接服务器
    if(mosquitto_connect(mosq, HOST, PORT, KEEP_ALIVE)){
        fprintf(stderr, "Unable to connect.\n");
        return 1;
    }
    //开启一个线程，在线程里不停的调用 mosquitto_loop() 来处理网络信息
    int loop = mosquitto_loop_start(mosq);
    if(loop != MOSQ_ERR_SUCCESS)
    {
        printf("mosquitto loop error\n");
        return 1;
    }
    while(fgets(buff, MSG_MAX_SIZE, stdin) != NULL)
    {
                /*发布消息*/
                mosquitto_publish(mosq,NULL,"Gai爷:",strlen(buff)+1,buff,0,0);
                memset(buff,0,sizeof(buff));
    }
    mosquitto_destroy(mosq);
    mosquitto_lib_cleanup();
    return 0;
}
```

在编译程序时需加上-lmosquitto链接。如：

```
$ gcc -o mosquitto_client_sub mosquitto_client_sub.c -lmosquitto
```

其它开发需要的注意点：

1. client id，这在上面代码注释里已经讲过了。
2. mosquitto有两种编程模式，详见 https://github.com/eclipse/mosquitto/issues/1282，多线程场景下需要注意。

参考：

1. [http://shaocheng.li/post/blog/2015-08-11](http://shaocheng.li/post/blog/2015-08-11)
2. [https://blog.csdn.net/dancer__sky/article/details/77855249](https://blog.csdn.net/dancer__sky/article/details/77855249)
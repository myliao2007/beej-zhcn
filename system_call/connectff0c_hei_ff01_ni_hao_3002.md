# 5.4. connect()，嘿!你好。

咱们用几分钟的时间假装你是个 telnet 应用程序，你的用户命令你［就像 TRON 电影里那样］取得一个 socket file descriptor。你运行并调用 socket()。接着用户告诉你连接到 ＂10.12.110.57＂ 的 port 23［标准 telnet port］。哟！你现在该做什麽呢？\
\
你是很幸运的程序，你现在可以细读 connect() 的章节，如何连线到远端主机。所以努力往前读吧！刻不容缓！\
\
connect() call 如下：\


```c
#include <sys/types.h>
#include <sys/socket.h>

int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);
```

sockfd 是我们的好邻居 socket file descriptor，如同 socket() 调用所返回的，serv\_addr 是一个 struct sockaddr，包含了目的 port 及 IP 地址，而 addrlen 是以 byte 为单位的 server 地址结构之长度。\
\
全部的资料都可以从 getaddrinfo() 调用中取得，它很好用。\
\
这样有开始比较有谱了吗？我在这里没办法知道，所以我只能希望是这样没错。\
\
我们有个示例，这边我们用 socket 连接到 ＂www.example.com＂ 的 port 3490：\


```c
struct addrinfo hints, *res;
int sockfd;

// 首先，用 getaddrinfo() 载入 address structs：

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC;
hints.ai_socktype = SOCK_STREAM;

getaddrinfo("www.example.com", "3490", &hints, &res);

// 建立一个 socket：

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

// connect!
connect(sockfd, res->ai_addr, res->ai_addrlen);
```

老式的程序再次填满了它们自己的 struct sockaddr\_ins 并传给 connect()。如果你愿意的话，你可以这样做。请见上面 bind() 章节中类似的提点。\
\
要确定有检查 connect() 返回的值，它在错误时会返回 -1，并设定 errno 变量。\
\
还要注意的是，我们不会调用 bind()。基本上，我们不用管我们的 local port number；我们只在意我们的目地［远端 port］。Kernel 会帮我们选择一个 local port，而我们要连接的站台会自动从我们这里取得资料，不用担心。

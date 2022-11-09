# 9.2. bind()

将一个 socket 关联到一个 IP address 及 port number\
\
**函数原型**

```c
#include <sys/types.h>

#include <sys/socket.h>

int bind(int sockfd, struct sockaddr *my_addr, socklen_t addrlen);
```

**说明**

当远程机器想要连接到你的服务器程序时，他需要两项信息：IP address 及 port number。

而 bind() call 可以让你做这件事。

首先你要先调用 getaddrinfo() 载入 struct sockaddr，以取得 destination address（目地地址）与 port 的信息。然后你调用 socket() 以取得一个 socket descriptor，接着将这个 socket 与 address 传递给 bind()，然后 IP address 跟 port 就会很神奇的跟 socket 绑在一起了（使用真的法术）！

如果你不知道你计算机的 IP address、或你知道你的计算机只有一个 IP address、或你不在意要用计算机上的哪个 IP address 时，你可以单纯地将 getaddrinfo() 的 hints.ai\_flags 参数设置为 AI\_PASSIVE，这里所做的事情就是将特别的值填入 struct sockaddr 的 IP address 字段，以告诉 bind() 说它应该要自动填入这个主机的 IP address。

什么？将什么特别的值填入 struct sockaddr 的 IP address 字段就可以让它自动填上目前主机的 address 呢？

我会告诉你的，但是请记住这只适用于你手动填入 struct sockaddr 时，如果不是，请依据上述方式，使用 getaddrinfo() 返回的结果。在 IPv4 中， struct sockaddr\_in 结构的  sin\_addr.s\_addr 会设置为 INADDR\_ANY；而在 IPv6 中， struct sockaddr\_in6 结构的 sin6\_addr 字段会从全局变量 in6addr\_any 载入。或者，若你正宣告一个新的 struct in6\_addr，你可以将它初始为 IN6ADDR\_ANY\_INIT。

最后，addrlen 参数应该设置为 my\_addr 的大小。

**返回值**

成功时返回零，或者错误时返回 -1（并依据错误设置 errno）\
\
**范例**

```c
// 用现代化的 getaddrinfo() 方式：

struct addrinfo hints, *res;

int sockfd;

// 首先，用 getaddrinfo() 加载地址结构数据：

memset(&hints, 0, sizeof hints);

hints.ai_family = AF_UNSPEC; // 使用 IPv4 或 IPv6，两者皆可

hints.ai_socktype = SOCK_STREAM;

hints.ai_flags = AI_PASSIVE; // 帮我填上我的 IP

getaddrinfo(NULL, "3490", &hints, &res);

// 建立 socket：

//（这边是简化版，照理你应该要查看 "res" linked list 的每个成员，并进行错误检查！）

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

// 将 sockfd 绑定（bind）到我们传递给 getaddrinfo() 的那个 port：

bind(sockfd, res->ai_addr, res->ai_addrlen);

// 手动封装 struct 的范例，IPv4

struct sockaddr_in myaddr;

int s;

myaddr.sin_family = AF_INET;

myaddr.sin_port = htons(3490);

// 你可以指定一个 IP address:

inet_pton(AF_INET, "63.161.169.137", &(myaddr.sin_addr));

// 或者你可以让系统自动选一个 IP address：

myaddr.sin_addr.s_addr = INADDR_ANY;

s = socket(PF_INET, SOCK_STREAM, 0);

bind(s, (struct sockaddr*)&myaddr, sizeof myaddr);
```



**参照**\
**getaddrinfo()**, **socket()**, struct sockaddr\_in, struct in\_addr

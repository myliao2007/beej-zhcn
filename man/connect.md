# 9.3. connect()

使用 socket 连接到服务器\
\
**函数原型**\


```c
#include <sys/types.h>

#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *serv_addr,

            socklen_t addrlen);
```

**说明**\
一旦你用 **socket()** call 建立了一个 socket descriptor 之后，你可以使用已知的 **connect()** system call 将这个 socket 联机到远程服务器。你该需做的是将 socket descriptor 与要连接的服务器 address 传递给 connect()（喔，还有 address 的长度，这个通常会传给这类的函数）。

\
通常这项信息会透过调用 **getaddrinfo()** 取得，但是如果你愿意，你也可以自行填写 struct sockaddr。\
\
若你尚未对这个 socket descriptor 呼叫 **bind()**，则会自动帮你绑定到你的 IP address 与一个随机的 local port（本地端口）。如果你不是服务器，这样还蛮不错的，因为你真的可以不用管理的 local port 是多少了；你只要注意远程的 port，这个可以在 serv\_addr 参数中设置。如果你有需要将 client socket 指定 IP address 与 port，那么你可以选择调用 **bind()** 来处理，不过这种情况是很少见的。

\
一旦 socket 已经用 **connect()** 完成连接，你就可以随意使用这个 socket 来 **send()** 与 **recv()** 数据到你心里想的地方。\
\
特别注意：如果你用 **connect()** 与 SOCK\_DGRAM UDP socket 联机到远程主机，若你愿意，你也可以像使用 sendto() 与 recvfrom() 那样来使用 **send()** 与 **recv()**。

\
**返回值**\
成功时返回零，或者发生错误时返回 -1（并设定相对应的 errno）。\
\
**范例**

```c
// 连接到 www.example.com 的 port 80（http）

struct addrinfo hints, *res;

int sockfd;

// 首先，使用 getaddrinfo() 取得地址数据：

memset(&hints, 0, sizeof hints);

hints.ai_family = AF_UNSPEC; // use IPv4 or IPv6, whichever

hints.ai_socktype = SOCK_STREAM;

// 我们可以在下一行用 "http" 取代 "80"：

getaddrinfo("www.example.com", "http", &hints, &res);

// 建立一个 socket:

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

// 将 socket 连接到我们在 getaddrinfo() 里指定的 address 与 port：

connect(sockfd, res->ai_addr, res->ai_addrlen);
```



**参考**\
**socket()**, **bind()**

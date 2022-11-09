# 9.1. accept()

接受从 listening socket 进来的连接\
\
**函数原型**\


```c
#include <sys/types.h>l;
#include <sys/socket.h>

int accept(int s, struct sockaddr *addr, socklen_t *addrlen);
```

\
**说明**\
一旦你完成取得 SOCK\_STREAM socket 并将 socket 设置好可以用来 **listen()** 进来的连接，接着你就能调用 **accept()** 让自己能取得一个新的 socket descriptor，做为後续与新连接 client 的通讯。\
\
原本用来 listen 的 socket 仍然还是会留着，当有新连接进来时，一样是用 **accept()** call 来接受新的连接。\
\


| s       | **listen()** 中的 socket descriptor。                                                           |
| ------- | -------------------------------------------------------------------------------------------- |
| addr    | 这里会填入连线到你这里的 client 地址。                                                                      |
| addrlen | 这里会填入 addr 参数中传回的数据结构大小。如果你确定你知道一定会收到的是 struct sockaddr\_in，你可以放心忽略这个参数，因为这就是你原本传递的 addr 型别。 |

\
**accept()** 通常会 block（闭锁），而你可以使用 **select()** 事先取得 listen 中的 socket descriptor 状态，检查 socket 是否就绪可读（ready to read）。若为就绪可读，则表示有新的连接正在等待被 **accept()**！另一个方式是将 listen 中的 socket 使用 **fcntl()** 设定 O\_NONBLOCK flag，然後 listen 中的 socket descriptor 就不会造成 block，而是传回 -1，并将 errno 设置为 EWOULDBLOCK。\
\
由 **accept()** 传回的 socket descriptor 是如假包换的 socket descriptor，开启并与远端主机连接，如果你要结束与 client 的连接，必须用**close()** 关闭。\
\
**返回值**\
**accept()** 返回新连接的 socket descriptor，错误时返回 -1，并将 errno 设置适当的值。\
\
**示例**\


```c
struct sockaddr_storage their_addr;
socklen_t addr_size;
struct addrinfo hints, *res;
int sockfd, new_fd;

// 首先：使用 getaddrinfo() 填好地址结构：

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC; // 使用 IPv4 或 IPv6，都可以
hints.ai_socktype = SOCK_STREAM;
hints.ai_flags = AI_PASSIVE; // 帮我填好我的 IP

getaddrinfo(NULL, MYPORT, &hints, &res);

// 建立一个 socket丶bind 它，并对它进行 listen：

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
bind(sockfd, res->ai_addr, res->ai_addrlen);
listen(sockfd, BACKLOG);

// 现在接受进入的连接：

addr_size = sizeof their_addr;
new_fd = accept(sockfd, (struct sockaddr *)&their_addr, &addr_size);

// 准备与 socket descriptor new_fd 沟通！
```

**参照**\
**socket()**, **getaddrinfo()**, **listen()**, struct sockaddr\_in

# 5.6. accept()－"谢谢你调用 port 3490"

准备好，accept() 调用是很奇妙的！会发生的事情就是：很远的人会试着 connect() 到你的电脑正在 listen() 的 port。他们的连接会排队等待被 accept()。你调用 accept()，并告诉它要取得搁置的（pending）连接。它会返回专属这个连接的一个新 socket file descriptor 给你！那是对的，你突然有了两个 _socket file descriptor_！原本的 socket file descriptor 仍然正在 listen 之後的连线，而新建立的 socket file descriptor 则是在最後要准备给 send() 与 recv() 用的。\
\
调用如下：\


```c
#include <sys/types.h>
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

_sockfd_ 是正在进行 listen() 的 socket descriptor。很简单，_addr_ 通常是一个指向 local struct sockaddr\_storage 的指针，关於进来的连接将往哪里去的资料［而你可以用它来得知是哪一台主机从哪一个 port 调用你的］。_addrlen_ 是一个 local 的整数变量，应该在将它的地址传递给 accept() 以前，将它设置为 sizeof(struct sockaddr\_storage)。accept() 不会存放更多的 bytes（字节）到 _addr_。若它存放了较少的 bytes 进去，它会改变 _addrlen_ 的值来表示。\
\
有想到吗？accept() 在错误发生时返回 -1 并设置 errno。不过 BetCha 不这麽认为。\
\
跟以前一样，用一段代码示例会比较好吸收，所以这里有一段示例程供你细读：\


```c
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define MYPORT "3490" // 使用者将连接的 port
#define BACKLOG 10 // 在队列中可以有多少个连接在等待

int main(void)
{
　　struct sockaddr_storage their_addr;
　　socklen_t addr_size;
　　struct addrinfo hints, *res;
　　int sockfd, new_fd;

　　// !! 不要忘了帮这些调用做错误检查 !!

　　// 首先，使用 getaddrinfo() 载入 address struct：

　　memset(&hints, 0, sizeof hints);
　　hints.ai_family = AF_UNSPEC; // 使用 IPv4 或 IPv6，都可以
　　hints.ai_socktype = SOCK_STREAM;
　　hints.ai_flags = AI_PASSIVE; // 帮我填上我的 IP 

　　getaddrinfo(NULL, MYPORT, &hints, &res);

　　// 产生一个 socket，bind socket，并 listen socket：

　　sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
　　bind(sockfd, res->ai_addr, res->ai_addrlen);
　　listen(sockfd, BACKLOG);

　　// 现在接受一个进入的连接：

　　addr_size = sizeof their_addr;
　　new_fd = accept(sockfd, (struct sockaddr *)&their_addr, &addr_size);

　　// 准备好与 new_fd 这个 socket descriptor 进行沟通！
　　.
　　.
　　.
```

\
一样，我们会将 _new\_fd_ socket descriptor 用於 send() 与 recv() 调用。若你只是要取得一个连接，你可以用 close() 关闭正在 listen 的_sockfd_，以避免有更多的连接进入同一个 port，若你有这个需要的话。

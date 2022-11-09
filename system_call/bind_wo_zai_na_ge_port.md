# 5.3. bind()－ 我在哪個 port？

一旦你有了一个 socket，你会想要将这个 socket 与你本地端的 port 进行关联［如果你正想要 listen() 特定 port 进入的连接，通常都会这样做，比如：多人网路连线游戏在它们告诉你＂连接到 192.168.5.10 port 3490＂时这麽做］。port number 是用来让 kernel 可以比对出进入的数据包是属於哪个 process 的 socket descriptor。如果你只是正在进行 connect()［因为你是 client，而不是 server］，这可能就不用。不过还是可以读读，有趣嘛。\


```
#include <sys/types.h>
#include <sys/socket.h>

int bind(int sockfd, struct sockaddr *my_addr, int addrlen);
```

_sockfd_ 是 socket() 传回的 socket file descriptor。_my\_addr_ 是指向包含你的地址资料丶名称及 IP address 的 struct sockaddr 之指针。_addrlen_ 是以 byte 为单位的地址长度。\
\
呼！有点比较好玩了。我们来看一个示例，它将 socket bind（绑定）到运行程序的主机上，port 是 3490：\


```c
struct addrinfo hints, *res;
int sockfd;

// 首先，用 getaddrinfo() 载入地址结构：

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC; // use IPv4 or IPv6, whichever
hints.ai_socktype = SOCK_STREAM;
hints.ai_flags = AI_PASSIVE; // fill in my IP for me

getaddrinfo(NULL, "3490", &hints, &res);

// 建立一个 socket：

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

// 将 socket bind 到我们传递给 getaddrinfo() 的 port：

bind(sockfd, res->ai_addr, res->ai_addrlen);
```

使用 AI\_PASSIVE flag，我可以跟程序说要 bind 它所在主机的 IP。如果你想要 bind 到指定的本地端 IP address，舍弃 AI\_PASSIVE，并改放一个地址到 getaddrinfo() 的第一个参数。\
\
bind() 在错误时也会返回 -1，并将 errno 设置为该错误的值。\
\
许多旧程序都在调用 bind() 以前手动封装 struct sockaddr\_in。很显然地，这是 IPv4 才有的，可是真的没有办法阻止你在 IPv6 做一样的事情，一般来说，使用 getaddrinfo() 会比较简单。总之，旧版的程式看起来会像这样：\


```c
// !!! 这 是 老 方 法 !!!

int sockfd;
struct sockaddr_in my_addr;

sockfd = socket(PF_INET, SOCK_STREAM, 0);

my_addr.sin_family = AF_INET;
my_addr.sin_port = htons(MYPORT); // short, network byte order
my_addr.sin_addr.s_addr = inet_addr("10.12.110.57");
memset(my_addr.sin_zero, '\0', sizeof my_addr.sin_zero);

bind(sockfd, (struct sockaddr *)&my_addr, sizeof my_addr);
```

在上列的代码中，如果你想要 bind 到你本地端的 IP address［就像上面的 AI\_PASSIVE flag］，你也可以将 INADDR\_ANY 指定给 s\_addr 栏位。INADDR\_ANY 的 IPv6 版本是一个 in6addr\_any 全局变量，它会被指定给你的 struct sockaddr\_in6 的 sin6\_addr 栏位。\
\
［也有一个你能用於 variable initializer（变量初始器）的 IN6ADDR\_ANY\_INIT macro（宏）］\
\
另一件调用 bind() 时要小心的事情是：不要用太小的 port number。全部 1024 以下的 ports 都是保留的［除非你是系统管理员］！你可以使用任何 1024 以上的 port number，最高到 65535［提供尚未被其它程序使用的］。\
\
你可能有注意到，有时候你试着重新运行 server，而 bind() 却失败了，它声称＂Address already in use.＂（地址正在使用）。这是什麽意思呢？很好，有些连接到 socket 的连接还悬在 kernel 里面，而它占据了这个 port。你可以等待它自行清除［一分钟之类］，或者在你的程序中新增代码，让它重新使用这个 port，类似这样：\


```
int yes=1;
//char yes='1'; // Solaris 的用户使用这个

// 可以跳过 "Address already in use" 错误信息

if (setsockopt(listener,SOL_SOCKET,SO_REUSEADDR,&yes,sizeof(int)) == -1) {
perror("setsockopt");
exit(1);
}
```

最後一个对 bind() 的额外小提醒：在你不愿意调用 bind() 时。若你正使用 connect() 连接到远端的机器，你可以不用管 local port 是多少（以 telnet 为例，你只管远端的 port 就好），你可以单纯地调用 connect()，它会检查 socket 是否尚未绑定（unbound），并在有需要的时候自动将 socket bind() 到一个尚未使用的 local port。

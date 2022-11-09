# 5.2. socket()－取得 File Descriptor！

我想可以不用再将 socket() 摆放在旁边凉快了，我一定要讲一下 socket() system call，这边是代码片段：\


```c
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```

可是这些参数是什麽？\
\
它们可以让你设定想要的 socket 类型［IPv4 或 IPv6，stream 或 datagram 以及 TCP 或 UDP］。\
\
以前的人得将这些写入固定的值，而你也可以这样做。［_domain_ 是 PF\_INET 或 PF\_INET6，_type_ 是 SOCK\_STREAM 或 SOCK\_DGRAM，而 _protocol_ 可以设置为 0，用来帮给予的 type 选择适当的协议。或者你可以调用 getprotobyname() 来查询你想要的协议，＂tcp＂或＂udp＂］。\
\
［PF\_INET 就是你在初始化 struct sockaddr\_in 的 _sin\_family_ 栏位会用到的，它是 AF\_INET 的亲戚。实际上，它们的关系很接近，所以其实它们的值也都一样，而许多程序设计师会调用 socket()，并以 AF\_INET 替换 PF\_INET 来做为第一个参数传递。\
\
现在，你可以去拿点牛奶跟饼乾，因为又是说故事时间了。\
\
在很久很久以前，人们认为它应该是地址家族（address family），就是 ＂AF\_INET＂中的＂AF＂所代表的意思；而地址家族也要支援协议家族（protocol family）的几个协议，这件事并没有发生，而之後它们都过着幸福快乐的日子，结束。\
\
所以最该做的事情就是在你的 struct sockaddr\_in 中使用 AF\_INET，而在调用 socket() 时使用 PF\_INET。］\
\
总之，这样就够了。你真的该做的只是使用调用 **getaddrinfo()** 得到的值，并将这个值直接提供给 socket()，像这样：\


```c
int s;
struct addrinfo hints, *res;

// 运行查询
// [假装我们已经填好 "hints" struct]
getaddrinfo("www.example.com", "http", &hints, &res);

// [再来，你应该要对 getaddrinfo() 进行错误检查, 并走到 "res" 链表查询能用的资料，
// 而不是假设第一笔资料就是好的［像这些示例一样］
// 实际的示例请参考 client/server 章节。
s = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
```

socket() 单纯返回一个之後 system call 要用的 _socket descriptor_ 给你，错误时会返回 -1。errno 全局变量会设置为该错误的值［细节请见 errno 的 man 手册，而且你需要继续阅读并执行更多与它相关的 system call，这样心里会比较有谱。］

# 5.10. getpeername()－你是谁？

这个函数很简单。\
\
它太简单了，我几乎不想给它一个自己的章节，虽然还是给了。\
\
getpeername() 函数会告诉你另一端连接的 stream socket 是谁，函数原型如下：\


```c
#include <sys/socket.h>
int getpeername(int sockfd, struct sockaddr *addr, int *addrlen);
```

_sockfd_ 是连接的 stream socket 之 descriptor，_addr_ 是指向 struct sockaddr［或 struct sockaddr\_in］的指针，这个数据结构储存了连线另一端的资料，而 _addrlen_ 则是指向 int 的指针，应该将它初始化为 sizeof \*addr 或 sizeof(struct sockaddr)。\
\
函数在错误时返回 -1，并设置相对的 errno。\
\
一旦你取得了它们的地址，你就可以用 inet\_ntop()丶getnameinfo() 或 gethostbyaddr() 印出或取得更多的资料。不过你无法取得它们的登录帐号。\
\
［好好好，如果另一台电脑运行的是 ident daemon 就可以。］然而，这个已经超出本教程的范围，更多资料请参考 RFC 1413 \[19]。\
\[19] http://tools.ietf.org/html/rfc1413

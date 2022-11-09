# 7.2. select()－同步 I/O 多工

这个函数有点奇怪，不过它很好用。看看下面这个情况：如果你是一个 server，而你想要 listen 正在进来的连接，如同不断读取已建立的连接 socket 一样。\
\
你说：没问题，只要用 **accept()** 及一对 **recv()** 就好了。\
\
慢点，老兄！如果你在 **accept()** call 时发生了 blocking 该怎麽办呢？你要如何同时进行 **recv()** 呢？\
\
＂那就使用 non-blocking socket！＂\
\
不行！你不会想成为浪费 CPU 资源的罪人吧。\
\
嗯，那有什麽好方法吗？\
\
**select()** 授予你同时监视多个 sockets 的权力，它会告诉你哪些 sockets 已经有数据可以读取丶哪些 sockets 已经可以写入，如果你真的想知道，还会告诉你哪些 sockets 触发了例外。\
\
即使 **select()** 相当有可移植性，不过却是监视 sockets 最慢的方法。一个比较可行的替代方案是 libevent \[24] 或者其它类似的方法，封装全部的系统相依要素，用以取得 socket 的通知。\
\
好了，不罗唆，下面我提供了 **select()** 的原型：\


```c
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

int select(int numfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
```

这个函数以 _readfds_丶_writefds_ 及 _exceptfds_ 监视 file descriptor（文件描述符）的 ＂sets（组）＂。如果你想要知道你是否能读取 standard input（标准输入）及某个 _sockfd_ socket descriptor，只要将 file descriptor 0 与 _sockfd_ 新增到 _readfds_ set 中。_numfds_ 参数应该要设置为 file descriptor 的最高值加 1。在这个例子中，应该要将 _numfds_ 设置为 _sockfd+1_，因为它必定大於 standard input（0）。\
\
当 **select()** 回传时，_readfds_ 会被修改，用来反映你所设置的 file descriptors 中，哪些已经有数据可以读取，你可以用下列的**FD\_ISSET()** macro（宏）来取得这些可读的 file descriptors。\
\
在继续谈下去以前，我想要说说该如何控制这些 sets。\
\
每个 sets 的型别都是 fd\_set，下列是用来控制这个型别的 macro：\


```c
FD_SET(int fd, fd_set *set);     将 fd 新增到 set。
FD_CLR(int fd, fd_set *set);     从 set 移除 fd。
FD_ISSET(int fd, fd_set *set);   若 fd 在 set 中，返回 true。
FD_ZERO(fd_set *set);            将 set 整个清为零。
```

\
最後，这个令人困惑的 struct timeval 是什麽东西呢？\
\
好，有时你不想要一直花时间在等人家送数据给你，或者明明没什麽事，却每 96 秒就要印出 ＂运行中 ...＂ 到终端（terminal），而这个 time structure 让你可以设置 timeout 的周期。\
\
如果时间超过了，而 **select()** 还没有找到任何就绪的 file descriptor 时，它会回传，让你可以继续做其它事情。\
\
struct timeval 的栏位如下：\


```c
struct timeval {
  int tv_sec; // 秒（second）
  int tv_usec; // 微秒（microseconds）
};
```

\
只要将 _tv\_sec_ 设置为要等待的秒数，并将 _tv\_usec_ 设置为要等待的微秒数。是的，就是微秒，不是毫秒。一毫秒有 1,000 微秒，而一秒有 1,000 毫秒。所以，一秒就有 1,000,000 微秒。\
\
为什麽要用 ＂usec（微秒）＂ 呢？\
\
＂u＂看起来很像我们用来表示 ＂micro（微）＂的希腊字母 μ（Mu）。还有，当函数回传时，会更新 _timeout_，用以表示还剩下多少时间。这个行为取决於你所使用的 Unix 而定。\
\


| <p>译注：<br>因为有些系统平台的 select() 会修改 timeout 的值，而有些系统不会，所以如果要重复调用 select() 的话，每次都应该要重新设置 timeout 的值，以确保程序的行为可以符合预期。</p> |
| ------------------------------------------------------------------------------------------------------------------- |

\
哇！我们有微秒精度的计时器了！\
\
是的，不过别依赖它。无论你将 struct timeval 设置的多小，你可能还要等待一小段 standard Unix timeslice（标准 Unix 时间片段）。\
\
另一件有趣的事：如果你将 struct timeval 的栏位设置为 0，select() 会在轮询过 sets 中的每个 file descriptors 之後，就马上 timeout。如果你将 timeout 参数设置为 NULL，它就永远不会 timeout，并且陷入等待，直到至少一个 file descriptor 已经就绪（ready）。如果你不在乎等待时间，就在调用 **select()** 时将 timeout 参数设置为 NULL。\
\
下列的代码片段 \[25] 等待 2.5 秒後，就会出现 standard input（标准输入）所输入的东西：\


```c
/*
** select.c -- a select() demo
*/
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
#define STDIN 0 // standard input 的 file descriptor
int main(void)
{
  struct timeval tv;
  fd_set readfds;

  tv.tv_sec = 2;
  tv.tv_usec = 500000;

  FD_ZERO(&readfds);
  FD_SET(STDIN, &readfds);

  // 不用管 writefds 与 exceptfds：
  select(STDIN+1, &readfds, NULL, NULL, &tv);

  if (FD_ISSET(STDIN, &readfds))
    printf("A key was pressed!\n");
  else
    printf("Timed out.\n");
  return 0;
}
```

如果你用一行缓冲区（buffer）的终端，那麽你从键盘输入数据後应该要尽快按下 Enter，否则程序就会发生 timeout。\
\
你现在可能在想，这个方法用在需要等待数据的 datagram socket 上很好，而且你是对的：应该是不错的方法。\
\
有些系统会用这个方式来使用 select()，而有些不行，如果你想要用它，你应该要参考你系统上的 man 使用手册说明看是否会有问题。\
\
有些系统会更新 struct timeval 的时间，用来反映 select() 原本还剩下多少时间 timeout；不过有些却不会。如果你想要程序是可移植的，那就不要倚赖这个特性。［如果你需要追踪剩下的时间，可以使用 **gettimeofday()**，我知道这很令人失望，不过事实就是这样。］\
\
如果在 read set 中的 socket 关闭连接，会怎样吗？\
\
好的，这个例子的 **select()** 回传时，会在 socket descriptor set 中说明这个 socket 是 ＂ready to read（就绪可读）＂的。而当你真的用**recv()** 去读取这个 socket 时，**recv()** 则会回传 0 给你。这样你就能知道是 client 关闭连接了。\
\
再次强调 **select()** 有趣的地方：如果你正在 listen() 一个 socket，你可以将这个 socket 的 file descriptor 放在 _readfds_ set 中，用来检查是不是有新的连接。\
\
我的朋友阿，这就是万能 **select()** 函数的速成说明。\
\
不过，应观众要求，这里提供个有深度的范例，毫无疑问地，以前的简单范例和这个范例的难易度会有显着差距。不过你可以先看看，然後读後面的解释。\
\
程序 \[26] 的行为是简单的多用户聊天室 server，在一个窗口中运行 server，然後在其它多个窗口使用 **telnet** 连接到 server［＂**telnet hostname 9034**＂］。当你在其中一个 **telnet** session 中输入某些文字时，这些文字应该会在其它每个窗口上出现。\


```c
/*
** selectserver.c -- 一个 cheezy 的多人聊天室 server
*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>

#define PORT "9034" // 我们正在 listen 的 port

// 取得 sockaddr，IPv4 或 IPv6：
void *get_in_addr(struct sockaddr *sa)
{
  if (sa->sa_family == AF_INET) {
    return &(((struct sockaddr_in*)sa)->sin_addr);
  }

  return &(((struct sockaddr_in6*)sa)->sin6_addr);
}

int main(void)
{
  fd_set master; // master file descriptor 表
  fd_set read_fds; // 给 select() 用的暂时 file descriptor 表
  int fdmax; // 最大的 file descriptor 数目

  int listener; // listening socket descriptor
  int newfd; // 新接受的 accept() socket descriptor
  struct sockaddr_storage remoteaddr; // client address
  socklen_t addrlen;

  char buf[256]; // 储存 client 数据的缓冲区
  int nbytes;

  char remoteIP[INET6_ADDRSTRLEN];

  int yes=1; // 供底下的 setsockopt() 设置 SO_REUSEADDR
  int i, j, rv;

  struct addrinfo hints, *ai, *p;

  FD_ZERO(&master); // 清除 master 与 temp sets
  FD_ZERO(&read_fds);

  // 给我们一个 socket，并且 bind 它
  memset(&hints, 0, sizeof hints);
  hints.ai_family = AF_UNSPEC;
  hints.ai_socktype = SOCK_STREAM;
  hints.ai_flags = AI_PASSIVE;

  if ((rv = getaddrinfo(NULL, PORT, &hints, &ai)) != 0) {
    fprintf(stderr, "selectserver: %s\n", gai_strerror(rv));
    exit(1);
  }

  for(p = ai; p != NULL; p = p->ai_next) {
    listener = socket(p->ai_family, p->ai_socktype, p->ai_protocol);
    if (listener < 0) {
      continue;
    }

    // 避开这个错误信息："address already in use"
    setsockopt(listener, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(int));

    if (bind(listener, p->ai_addr, p->ai_addrlen) < 0) {
      close(listener);
      continue;
    }

    break;
  }

  // 若我们进入这个判断式，则表示我们 bind() 失败
  if (p == NULL) {
    fprintf(stderr, "selectserver: failed to bind\n");
    exit(2);
  }
  freeaddrinfo(ai); // all done with this

  // listen
  if (listen(listener, 10) == -1) {
    perror("listen");
    exit(3);
  }

  // 将 listener 新增到 master set
  FD_SET(listener, &master);

  // 持续追踪最大的 file descriptor
  fdmax = listener; // 到此为止，就是它了

  // 主要循环
  for( ; ; ) {
    read_fds = master; // 复制 master

    if (select(fdmax+1, &read_fds, NULL, NULL, NULL) == -1) {
      perror("select");
      exit(4);
    }

    // 在现存的连接中寻找需要读取的数据
    for(i = 0; i <= fdmax; i++) {
      if (FD_ISSET(i, &read_fds)) { // 我们找到一个！！
        if (i == listener) {
          // handle new connections
          addrlen = sizeof remoteaddr;
          newfd = accept(listener,
            (struct sockaddr *)&remoteaddr,
            &addrlen);

          if (newfd == -1) {
            perror("accept");
          } else {
            FD_SET(newfd, &master); // 新增到 master set
            if (newfd > fdmax) { // 持续追踪最大的 fd
              fdmax = newfd;
            }
            printf("selectserver: new connection from %s on "
              "socket %d\n",
              inet_ntop(remoteaddr.ss_family,
                get_in_addr((struct sockaddr*)&remoteaddr),
                remoteIP, INET6_ADDRSTRLEN),
              newfd);
          }

        } else {
          // 处理来自 client 的数据
          if ((nbytes = recv(i, buf, sizeof buf, 0)) <= 0) {
            // got error or connection closed by client
            if (nbytes == 0) {
              // 关闭连接
              printf("selectserver: socket %d hung up\n", i);
            } else {
              perror("recv");
            }
            close(i); // bye!
            FD_CLR(i, &master); // 从 master set 中移除

          } else {
            // 我们从 client 收到一些数据
            for(j = 0; j <= fdmax; j++) {
              // 送给大家！
              if (FD_ISSET(j, &master)) {
                // 不用送给 listener 跟我们自己
                if (j != listener && j != i) {
                  if (send(j, buf, nbytes, 0) == -1) {
                    perror("send");
                  }
                }
              }
            }
          }
        } // END handle data from client
      } // END got new incoming connection
    } // END looping through file descriptors
  } // END for( ; ; )--and you thought it would never end!

  return 0;
}
```

我说过在代码中有两个 file descriptor sets：_master_ 与 _read\_fds_。前面的 _master_ 记录全部现有连接的 socket descriptors，与正在 listen 新连接的 socket descriptor 一样。\
\
我用 _master_ 的理由是因为 **select()** 实际上会改变你传送过去的 set，用来反映目前就绪可读（ready for read）的 sockets。因为我必须在在两次的 **select()** calls 期间也能够持续追踪连接，所以我必须将这些数据安全地储存在某个地方。最後，我再将 _master_ 复制到_read\_fds_，并接着调用 **select()**。\
\
可是这不就代表每当有新连接时，我就要将它新增到 _master set_ 吗？是的！\
\
而每次连接结束时，我们也要将它从 master set 中移除吗？是的，没有错。\
\
我说过，我们要检查 listen 的 socket 是否就绪可读，如果可读，这代表我有一个待处理的连接，而且我要 **accept()** 这个连接，并将它新增到 _master_ set。同样地，当 client 连接就绪可读且 **recv()** 返回 0 时，我们就能知道 client 关闭了连接，而我必须将这个 socket descriptor 从 master set 中移除。\
\
若 client 的 **recv()** 返回非零的值，因而，我能知道 client 已经收到了一些数据，所以我收下这些数据，并接着到 master 清单，并将数据送给其它已连接的每个 clients。\
\
我的朋友们，以上对万能 **select()** 函数的概述，这真是不简单的事情。\
\
另外，这里有个福利：一个名为 **poll** 的函数，它的行为与 **select()** 很像，但是在管理 file descriptor sets 时用不一样的系统，你可以看看 [poll](https://web.archive.org/web/20180808070543/http://beej-zhcn.netdpi.net/09-man-manual/9-17-poll)。\
\
参考资料\
\[24] http://www.monkey.org/\~provos/libevent/\
\[25] http://beej.us/guide/bgnet/examples/select.c\
\[26] http://beej.us/guide/bgnet/examples/selectserver.c

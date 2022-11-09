# 7.1. Blocking（阻塞）

你听过 blocking，只是它在这里代表什麽鬼东西呢？简而言之，＂block＂ 就是 ＂sleep（休眠）＂的技术术语。你在以前运行 **listener**时你可能有注意到，它只是在那边等，直到有数据包抵达。\
\
很多函数都会 block，**accept()** 会 block，全部的 recv() 函数都会 block。原因是它们有权这麽做。当你先用 **socket()** 建立 socket descriptor 时，kernel（内核）会将它设置为 blocking。若你不想要 blocking socket，你必须调用 **fcntl()**：\


```c
#include <unistd.h>
#include <fcntl.h>
.
.
.
sockfd = socket(PF_INET, SOCK_STREAM, 0);
fcntl(sockfd, F_SETFL, O_NONBLOCK);
.
.
.
```

将 socket 设置为 non-blocking（非阻塞），你就能 ＂poll（轮询）＂socket 以取得数据。如果你试着读取 non-blocking socket，而 socket 没有数据时，函数就不会发生 block，而是返回 -1，并将 errno 设置为 EWOULDBLOCK。\
\
然而，一般来说，这样 polling 是不好的想法。如果你让程序一直忙着查 socket 上是否有数据，则会浪费 CPU 的时间，这样是不合适的。比较漂亮的解法是利用下一节的 **select()** 来检查 socket 是否有数据需要读取。

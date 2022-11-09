# 5.11. gethostname()－我是誰？

比 getpeername() 更简单的函数是 gethostname()，它会返回你运行程序的电脑名，这个名称之後可以用在 gethostbyname()，用来定义你本地端电脑的 IP address。\
\
有什麽更有趣的吗？\
\
我可以想到一些事情，不过这不适合 socket 编程，总之，下面是一段示例：\


```c
#include <unistd.h>
int gethostname(char *hostname, size_t size);
```

参数很简单：_hostname_ 是指向字符数组（array of chars）的指针，它会储存函数返回的主机名（hostname），而 _size_ 是以 byte 为单位的主机名长度。\
\
函数在运行成功时返回 0，在错误时返回 -1，并一样设置 errno。

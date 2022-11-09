# 7.3. 不完整传送的後续处理

还记得前面的 **send()** 章节吗？当时我不是提过 **send()** 可能不会将你所要求的数据全部送出吗？也就是说，虽然你想要送出 512 bytes，但是 send() 只送出 412 bytes。那剩下的 100 个 bytes 到哪去了呢？\
\
好的，其实它们还在你的缓冲区里。因为环境不是你能控制的，kernel 会决定要不要用一个 chunk 将全部的数据送出，而现在，我的朋友，你可以决定要如何处理缓冲区中剩下的数据。\
\
你可以写一个像这样的函数：\


```c
#include <sys/types.h>
#include <sys/socket.h>

int sendall(int s, char *buf, int *len)
{
  int total = 0; // 我们已经送出多少 bytes 的数据
  int bytesleft = *len; // 我们还有多少数据要送
  int n;

  while(total < *len) {
    n = send(s, buf+total, bytesleft, 0);
    if (n == -1) { break; }
    total += n;
    bytesleft -= n;
  }

  *len = total; // 返回实际上送出的数据量

  return n==-1?-1:0; // 失败时返回 -1丶成功时返回 0
}
```

在这个例子里，_s_ 是你想要传送数据的 socket，_buf_ 是储存数据的缓冲区，而 _len_ 是一个指针，指向一个 int 型别的变数，记录了缓冲区中的数据数量。\
\
函数在错误时返回 -1［而 _errno_ 仍然从调用 **send()** 设置］。还有，实际送出的数据数量会在 _len_ 中回传，除非有错误发生，不然这会跟你所要求要传送的数据量相同。**sendall()** 会尽力将数据送出，不过如果有错误发生时，它就会立刻回传给你。\
\
为了完整性，这边有一个调用函数的示例：\


```c
char buf[10] = "Beej!";
int len;

len = strlen(buf);
if (sendall(s, buf, &len) == -1) {
  perror("sendall");
  printf("We only sent %d bytes because of the error!\n", len);
}
```

当数据包的一部分抵达接收端（receiver end）时会发生什麽事情呢？如果数据包的长度是会变动的（variable），接收端要如何知道另一端的数据包何时开始与结束呢？\
\
是的，你或许必须封装（encapsulate）［还记得数据封装（data encapsulation）章节的开头那边吗？那边有详细说明］。

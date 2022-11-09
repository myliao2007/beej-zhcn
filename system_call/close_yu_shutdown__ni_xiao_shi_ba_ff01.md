# 5.9. close() 与 shutdown()－从我面前消失吧！

呼！你已经成天都在 send() 与 recv()了。你正准备要关闭你 socket descriptor 的连接，这很简单，你只要使用常规的 UNIX file descriptor close() 函数：\


```c
close(sockfd);
```

这会避免对 socket 做更多的读写。任何想要对这个远端的 socket 进行读写的人都会收到错误。\
\
如果你想要能多点控制 socket 如何关闭，可以使用 shutdown() 函数。它让你可以切断单向的通信，或者双向［就像是 close() 所做的］，这是函数原型：\


```c
int shutdown(int sockfd, int how);
```

_sockfd_ 是你想要 shutdown 的 socket file descriptor，而 _how_ 是下列其中一个值：\


```c
0 不允许再接收数据
1 不允许再传送数据
2 不允许再传送与接收数据［就像 close()］
```

shutdown() 成功时返回 0，而错误时返回 -1（设置相对的 errno）。\
\
若你在 unconnected datagram socket 上使用 shutdown()，它只会单纯的让 socket 无法再进行 send() 与 recv() 调用［要记住你只能在有 connect() 到 datagram socket 的时候使用］。\
\
重要的是 shutdown() 实际上没有关闭 file descriptor，它只是改变了它的可用性。如果要释放 socket descriptor，你还是需要使用 close()。\
\
没了。\
\
［除了要记得的是，如果你用 Windows 与 Winsock，你应该要调用 closesocket() 而不是 close()。］

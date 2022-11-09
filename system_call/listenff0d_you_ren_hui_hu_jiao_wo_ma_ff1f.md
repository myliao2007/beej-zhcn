# 5.5. listen()－有人会调用我吗？

OK，是该改变步调的时候了。如果你不想要连接到一个远端主机要怎麽做。\
\
我说过，好玩就好，你想要等待进入的连接，并以某种方式处理它们。\
\
这个过程有两个步骤：你要先调用 listen()，接着调用 accept()［参考下一节］。\
\
listen 调用相当简单，不过需要一点说明：\


```c
int listen(int sockfd, int backlog);
```

\
_sockfd_ 是来自 socket() system call 的一般 socket file descriptor。_backlog_ 是进入的队列（incoming queue）中所允许的连接数目。这代表什麽意思呢？好的，进入的连接将会在这个队列中排队等待，直到你 accept() 它们（请见下节），而这限制了排队的数量。多数的系统默认将这个数值限制为 20；你或许可以一开始就将它设置为 5 或 10。\
\
再来，如同往常，listen() 会返回 -1 并在错误时设置 errno。\
\
好的，你可能会想像，我们需要在调用 listen() 以前调用 bind()，让 server 可以在指定的 port 上运行。［你必须能告诉你的好朋友要连接到哪一个 port！］所以如果你正在 listen 进入的连接，你会运行的 system call 顺序是：\


```c
getaddrinfo();
socket();
bind();
listen();
/* accept() 从这里开始 */
```

我只是留下示例程序的位置，因为它相当显而易见。［在下面 accept() 章节中的代码会比较完整］。这整件事情真正需要技巧的部分是调用 accept()。

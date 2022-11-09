# 5.7. send() 与 recv()－ 宝贝，跟我说说话！

这两个用来通讯的函数是透过 stream socket 或 connected datagram ssocket。若你想要使用常规的 unconnected datagram socket，你会需要参考底下的 sendto() 及 recvfrom() 的章节。\
\
send() 调用：\


```c
int send(int sockfd, const void *msg, int len, int flags);
```

_sockfd_ 是你想要送资料过去的 socket descriptor［不论它是不是 socket() 返回的，或是你用 accept() 取得的］。_msg_ 是一个指向你想要传送资料之指标，而 _len_ 是以 byte 为单位的资料长度。而 _flags_ 设置为 0 就好。［更多相关的 flag 资料请见 send() man 手册］。\
\
一些示例代码如下：\


```c
char *msg = "Beej was here!";
int len, bytes_sent;
.
.
.
len = strlen(msg);
bytes_sent = send(sockfd, msg, len, 0);
.
.
.
```

send() 会返回实际有送出的 byte 数目，这可能会少於你所要传送的数目！有时候你告诉 send() 要送整笔的资料，而它就是无法处理这麽多资料。它只会尽量将资料送出，并认为你之後会再次送出剩下没送出的部分。\
\
要记住，如果 send() 返回的值与 _len_ 的值不符合的话，你就需要再送出字串剩下的部分。好消息是：如果数据包很小［比 1K 还要小这类的］，或许有机会一次就送出全部的东西。\
\
一样，错误时会返回 -1，并将 errno 设置为错误码（error number）。\
\
recv() 调用在许多地方都是类似的：\


```c
int recv(int sockfd, void *buf, int len, int flags);
```

_sockfd_ 是要读取的 socket descriptor，_buf_ 是要记录读到资料的缓冲区（buffer），_len_ 是缓冲区的最大长度，而 _flags_ 可以再设置为 0。［关於 flag 资料的细节请参考 recv() 的 man 手册］。\
\
recv() 返回实际读到并写入到缓冲区的 byte 数目，而错误时返回 -1［并设置相对的 errno］。\
\
等等！ recv() 会返回 0，这只能表示一件事情：远端那边已经关闭了你的连接！recv() 返回 0 的值是让你知道这件事情。\
\
这样很简单，不是吗？你现在可以送回数据，并往 stream sockets 迈进！嘻嘻！你是 UNIX 网路程序员了。

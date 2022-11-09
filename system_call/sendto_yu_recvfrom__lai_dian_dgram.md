# 5.8. sendto() 与 recvfrom()－ 用 DGRAM 风格跟我说说话

我听到你说，＂这全部都是上等的好货＂，＂可是我该如何使用 unconnected datagram socket 呢？＂\
\
没问题，朋友。我们正要讲这件事。\
\
因为 datagram socket 没有连线到到远端主机，猜猜看，我们在送出数据包以前会需要哪些资料呢？\
\
对！目的地址！在这里抢先看：\


```c
sendto(int sockfd, const void *msg, int len, unsigned int flags,
const struct sockaddr *to, socklen_t tolen);
```

如你所见，这个调用基本上与调用 send() 一样，只是多了两个额外的资料。_to_ 是一个指向 struct sockaddr［这或许是另一个你可以在最後转型的 struct sockaddr\_in 或 struct sockaddr\_in6 或 struct sockaddr\_storage］的指针，它包含了目的 IP address 与 port。_tolen_ 是一个 int，可以单纯地将它设置为 sizeof \*to 或 sizeof(struct sockaddr\_storage)。\
\
为了能自动处理目的地址结构（destination address structure），你或许可以用底下的 getaddrinfo() 或 recvfrom()，或者你也可以手动填上。\
\
如同 send()，sendto() 会返回实际已传送的资料数量（一样，可能会少於你要传送的资料量！）而错误时返回 -1。\
\
recv() 与 recvfrom() 也是差不多的。recvfrom() 的对照如下：\


```c
int recvfrom(int sockfd, void *buf, int len, unsigned int flags,
struct sockaddr *from, int *fromlen);
```

一样，它跟 recv() 很像，只是多了两个栏位。_from_ 是指向 local struct sockaddr\_storage 的指针，这个数据结构包含了数据包来源的 IP address 与 port。_fromlen_ 是指向 local int 的指针，应该要初始化为 sizeof \*from 或是 sizeof(struct sockaddr\_storage)。当函数返回时，_fromlen_ 会包含实际上储存於 _from_ 中的地址长度。\
\
recvfrom() 返回接收的数据数目，或在发生错误时返回 -1［并设置相对的 errno］。\
\
所以这里有个问题：为什麽我们要用 struct sockaddr\_storage 做为 socket 的型别呢？为什麽不用 struct sockaddr\_in 呢？\
\
因为你知道的，我们不想要让自己绑在 IPv4 或 IPv6，所以我们使用通用的泛型 struct sockaddr\_storage，我们知道这样有足够的空间可以用在 IPv4 与 IPv6。\
\
［所以 ... 这里有另一个问题：为什麽不是 struct sockaddr 本身就可以容纳任何地址呢？我们甚至可以将通用的 struct sockaddr\_storage 转型为通用的 struct sockaddr！似乎没什麽关系又很累赘啊。答案是，它就是不够大，我猜在这个时候更动它会有问题，所以他们就弄了一个新的。］\
\
记住，如果你 connect() 到一个 datagram socket，你可以在你全部的交易中只使用 send() 与 recv()。socket 本身仍然是 datagram socket，而数据包仍然使用 UDP，但是 socket interface 会自动帮你增加目的与来源资料。

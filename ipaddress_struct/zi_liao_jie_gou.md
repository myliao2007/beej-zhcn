# 3.3. 数据结构

很好，终於讲到这里了，该是谈谈编程的时间了。在本节，我会介绍 socket 接口的各种数据型别，因为它们有些会不太好理解。

首先是最简单的：socket descriptor，型别如下：

```
int
```

就是一般的 int。

从这里开始会有点不好理解，所以不用问太多，直接读过就好。

我的第一个 StructTM －struct addrinfo，这个数据结构是最近的发明，用来准备之後要用的 socket 地址数据结构，也用在主机名（host name）及服务名（service name）的查询。当我们之後开始实际应用时，才会开始觉得比较靠谱，现在只需要知道你在建立连接调用时会用到这个数据结构。

```c
struct addrinfo {
    int ai_flags; // AI_PASSIVE, AI_CANONNAME 等。
    int ai_family; // AF_INET, AF_INET6, AF_UNSPEC
    int ai_socktype; // SOCK_STREAM, SOCK_DGRAM
    int ai_protocol; // 用 0 當作 "any"
    size_t ai_addrlen; // ai_addr 的大小，單位是 byte
    struct sockaddr *ai_addr; // struct sockaddr_in 或 _in6
    char *ai_canonname; // 典型的 hostname
    struct addrinfo *ai_next; // 鏈結串列、下個節點
};
```

你可以载入这个数据结构，然後调用 getaddrinfo()。它会返回一个指针，这个指针指向一个新的链表，这个链表有一些数据结构，而数据结构的内容记载了你所需的东西。

你可以在 ai\_family 栏位中设定强制使用 IPv4 或 IPv6，或者将它设定为 AF\_UNSPEC，AF\_UNSPEC 很酷，因为这样你的程序就可以不用管 IP 的版本。

要注意的是，这是个链表：ai\_next 是指向下一个成员（element），可能会有多个结果让你选择。我会直接用它提供的第一个结果，不过你可能会有不同的个人考量；先生！我不是万事通。

你会在 struct addrinfo 中看到 ai\_addr 栏位是一个指向 struct sockaddr 的指针。这是我们开始要了解 IP 地址结构中有哪些细节的地方。有时候，你需要的是调用 getaddrinfo() 帮你填好 struct addrinfo。然而，你必须查看这些数据结构，并将值取出，所以我在这边会进行说明。

［还有，在发明 struct addrinfo 以前的代码都要手动填写这些数据的每个栏位，所以你会看到很多 IPv4 的代码真的用很原始的方式去做这件事。你知道的，本教程在旧版也是这样做］。

有些 structs 是 IPv4，而有些是 IPv6，有些两者都是。我会特别注明清楚它们属於哪一种。

总之，struct sockaddr 记录了很多 sockets 类型的 socket 的地址资料。

```c
struct sockaddr {
    unsigned short sa_family; // address family, AF_xxx
    char sa_data[14]; // 14 bytes of protocol address
};
```

sa\_family 可以是任何东西，不过在这份教程中我们会用到的是 AF\_INET［IPv4］或 AF\_INET6［IPv6］。sa\_data 包含一个 socket 的目地地址与 port number。这样很不方便，因为你不会想要手动的将地址封装到 sa\_data 里。

为了处理 struct sockaddr，程序设计师建立了对等平行的数据结构：struct sockaddr\_in［＂in＂是代表＂internet＂］用在 IPv4。

而这有个重点：指向 struct sockaddr\_in 的指针可以转型（cast）为指向 struct sockaddr 的指针，反之亦然。所以即使 connect() 需要一个 struct sockaddr \*，你也可以用 struct sockaddr\_in，并在最後的时候对它做型别转换！

```c
// （IPv4 專用-- IPv6 請見 struct sockaddr_in6）
struct sockaddr_in {
    short int sin_family; // Address family, AF_INET
    unsigned short int sin_port; // Port number
    struct in_addr sin_addr; // Internet address
    unsigned char sin_zero[8]; // 與 struct sockaddr 相同的大小
};
```

这个数据结构让它很容易可以参考（reference）socket 地址的成员。要注意的是 sin\_zero［这是用来将数据结构补足符合 struct sockaddr 的长度］，应该要使用 memset()函数将 sin\_zero 整个清为零。还有，sin\_family 是对应到 struct sockaddr 中的 sa\_family，并应该设定为＂AF\_INET＂。最後，sin\_port 必须是 Network Byte Order［利用 htons()］。

让我们再更深入点！你可以在 sruct in\_addr 里看到 sin\_addr 栏位。

那是什麽？

好，别太激动，不过它是其中一个最恐怖的 union：

```c
// (仅限 IPv4 — IPv6 请参考 struct in6_addr)
// Internet address (a structure for historical reasons)
struct in_addr {
    uint32_t s_addr; // that's a 32-bit int (4 bytes)
};
```

哇！好耶，它以前是 union，不过这个包袱现在似乎已经不见了。因此，若你已将 ina 宣告为 struct sockaddr\_in 的型别时，那麽 ina.sin\_addr.s\_addr 会参考到 4-byte 的 IP address（以 Network Byte Order）。要注意的是，如果你的系统仍然在 struct in\_addr 使用超恐怖的 union，你依然可以像我上面说的，精确地参考到 4-byte 的 IP address［这是因为 #define］。

那麽 IPv6 会怎样呢？

IPv6 也有提供类似的 struct，比如：

```c
// (IPv6 專用-- IPv4 請見 struct sockaddr_in 與 struct in_addr)
struct sockaddr_in6 {
    u_int16_t sin6_family; // address family, AF_INET6
    u_int16_t sin6_port; // port number, Network Byte Order
    u_int32_t sin6_flowinfo; // IPv6 flow 資訊
    struct in6_addr sin6_addr; // IPv6 address
    u_int32_t sin6_scope_id; // Scope ID
};

struct in6_addr {
    unsigned char s6_addr[16]; // IPv6 address
};
```

要注意到 IPv6 协议有一个 IPv6 address 与一个 port number，就像 IPv4 协议有一个 IPv4 address 与 port number 一样。

我现在还不会介绍 IPv6 的流量资料，或是 Scope ID 栏位 … 这只是一份入门教程嘛 :-)

最後要强调的一点，这个简单的 struct sockaddr\_storage 是设计用来足够储存 IPv4 与 IPv6 structures 的 structure。［你看看，对於某些 calls，你有时无法事先知道它是否会使用 IPv4 或 IPv6 address 来填好你的 struct sockaddr。所以你用这个平行的 structure 来传递，它除了比较大以外，也很类似 struct sockaddr ，因而可以将它转型为你所需的型别］。

```c
struct sockaddr_storage {
    sa_family_t ss_family; // address family
    // 這裡都是填充物（padding），依實作而定，請忽略它：
    char __ss_pad1[_SS_PAD1SIZE];
    int64_t __ss_align;
    char __ss_pad2[_SS_PAD2SIZE];
};
```

重点是你可以在 ss\_family 栏位看到地址家族（address family），检查它是 AF\_INET 或 AF\_INET6（是 IPv4 或 IPv6）。之後如果你愿意的话，你就可以将它转型为 sockaddr\_in 或 struct sockaddr\_in6。

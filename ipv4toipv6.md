# 4. 从 IPv4 移植为 IPv6

＂可是我只想知道要改代码的哪些地方就可以支援 IPv6 了！快点告诉我！＂

OK！OK！

我几乎都是以过来人的身分来讲这边的每件事，这次不考验各位的耐心，来个简洁的短文。［当然有更多比这篇短的文章，不过我是跟自己的这份教程来比较］。

1\. 首先，请试着用 getaddrinfo() 来取得 struct sockaddr 的资料，取代手动填写这个数据结构。这样你就可以不用管 IP 的版本，而且能省略之後很多步骤。

2\. 找出全部与 IP 版本相关的任何代码，试着用一个有用的函数将它们包起来（wrap up）。

3\. 将 AF\_INET 更改为 AF\_INET6。

4\. 将 PF\_INET 更改为 PF\_INET6。

5\. 将 INADDR\_ANY 更改为 in6addr\_any，这里有点不太一样：

```c
struct sockaddr_in sa;
struct sockaddr_in6 sa6;

sa.sin_addr.s_addr = INADDR_ANY;   // 使用我的 IPv4 位址
sa6.sin6_addr = in6addr_any;  // 使用我的 IPv6 位址
```

还有，在宣告 struct in6\_addr 时，IN6ADDR\_ANY\_INIT 的值可以做为初始值，像这样：

```c
struct in6_addr ia6 = IN6ADDR_ANY_INIT;
```

6\. 使用 struct sockaddr\_in6 取代 struct sockaddr\_in，确定要将＂6＂新增到适当的栏位［参考上面的 structs］，但没有 sin6\_zero 栏位。

7\. 使用 struct in6\_addr 取代 struct in\_addr，要确定有将＂6＂新增到适当的栏位［参考上面的structs］。

8\. 使用 inet\_pton() 替换 inet\_aton() 或 inet\_addr()。

9\. 使用 inet\_ntop() 替换 inet\_ntoa()。

10\. 使用很牛的 getaddrinfo() 取代 gethostbyname()。

11\. 使用很牛的 getnameinfo() 取代 gethostbyaddr()［虽然 gethostbyaddr()在 IPv6 中也能正常运作］。

12\. 不要用 INADDR\_BROADCAST 了，请多使用 IPv6 multicast 来替换。

就是这样。

译注：

IPv6 可参考萩野纯一郎, IPv6 网路编程, 博硕, 2004.

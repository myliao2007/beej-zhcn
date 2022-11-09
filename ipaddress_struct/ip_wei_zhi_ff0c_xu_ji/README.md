# 3.4. IP address，Part II

还好你运气不错，有一堆函数让你能够控制 IP address，而不需要亲自用 long 与 << 运算符来处理它们。

咱们说，你有一个 struct sockaddr\_in ina，而且你有一个 ＂10.12.110.57＂ 或 ＂2001:db8:63b3:1::3490＂ 这样的一个 IP address 要储存。你想要使用 inet\_pton() 函数将 IP address 转换为数值与句号的符号，并依照你指定的 AF\_INET 或 AF\_INET6 来决定要储存在 struct in\_addr 或 struct in6\_addr。［＂pton＂的意思是＂presentation to network＂，你可以称之为＂printable to network＂，如果这样会比较好记的话］。

这样的转换可以用如下的方式：

```c
struct sockaddr_in sa; // IPv4
struct sockaddr_in6 sa6; // IPv6
inet_pton(AF_INET, "192.0.2.1", &(sa.sin_addr)); // IPv4
inet_pton(AF_INET6, "2001:db8:63b3:1::3490", &(sa6.sin6_addr)); // IPv6
```

［小记：原本的老方法是使用名为 inet\_addr() 的函数或另一个名为 inet\_aton() 的函数；这些都过时了，而且不适合在 IPv6 中使用］。

目前上述的代码片段还不是很可靠，因为没有错误检查。inet\_pton() 在错误时会返回 -1，而若地址被搞砸了，则会返回 0。所以在使用之前要检查，并确认结果是大於 0 的。

好了，现在你可以将 IP address 字符串转换为它们的二进位表示。

还有其它方法吗？

如果你有一个 struct in\_addr 且你想要以数字与句号印出来的话呢？

［呵呵，或者如果你想要以＂十六进制与冒号＂打印出 struct in6\_addr］。在这个例子中，你会想要使用 inet\_ntop()函数［＂ntop＂意谓＂network to presentation＂－如果有比较好记的话，你可以称它为＂network to prinable＂］，像是这样：

```c
// IPv4:
char ip4[INET_ADDRSTRLEN]; // 储存 IPv4 字符串的空间
struct sockaddr_in sa; // 假這會由某個東西載入
inet_ntop(AF_INET, &(sa.sin_addr), ip4, INET_ADDRSTRLEN);
printf("The IPv4 address is: %s\n", ip4);
// IPv6:
char ip6[INET6_ADDRSTRLEN]; // 储存 IPv6 字符串的空间
struct sockaddr_in6 sa6; // 假這會由某個東西載入
inet_ntop(AF_INET6, &(sa6.sin6_addr), ip6, INET6_ADDRSTRLEN);
printf("The address is: %s\n", ip6);
```

当你调用它时，你会传递地址的型别［IPv4 或 IPv6］，该地址是一个指向储存结果的字符串，与该字符串的最大长度。［有两个 macro（宏）可以很方便地储存你想储存的最大 IPv4 或 IPv6 地址字符串大小：INET\_ADDRSTRLEN 与 INET6\_ADDRSTRLEN］。

［另一个要再次注意的是以前的方法：以前做这类转换的函数名为 inet\_ntoa()，它已经过期了，而也在 IPv6 中也不适用］。

最後，这些函数只能用在数值的 IP address 上，它们不需要 DNS nameserver 来查询主机名，如＂www.example.com＂。你可以使用 getaddrinfo() 来做这件事情，如同你稍後会看到的。

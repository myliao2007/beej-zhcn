# 31.1. Sub network (子网)

为了结构化的理由，有时我们这样宣告是很方便的：＂IP address 的前段是 IP address 的 network（网路），而後面的部分是 host（主机）。＂

例如：在 IPv4，你可能有 192.0.2.12，而我们可以说前面三个 bytes 是 network，而最後一个 byte 是 host。或者换个方式，我们能说 host 12 位在 network 192.0.2.0。［请参考我们如何将 host byte 清为零］。

接下来要讲的是过时的资料了！

真的吗？

很久很久以前，有 subnets（子网）的＂class＂（分类），在这里，地址的第一个丶前二个或前三个 bytes 都是属於 network 的一部分。如果你很幸运可以拥有一个 byte 的 network，而另外三个 bytes 是 host 地址，那在你的网路上，你有价值 24 bits 的 host number［大约两千四百万个地址左右］。这是一个＂Class A＂（A 类）网路；相对则是一个＂Class C＂（C 类）的网路，network 有三个 bytes丶而 host 只有一个 byte［256 个 hosts，而且还要再扣掉两个保留的地址］。

所以，如同你所看到的，只有一些 Class A 网路，一大堆的 Class C 网路，以及一些中等的 Class B 网路。

IP address 的网络地址位数由 netmask（网路掩码）决定，你可以将 IP address 与 netmask 进行 AND bitwise，就能得到 network 的值。Netmask 一般看起来像是 255.255.255.0［如：若你的 IP 是 192.0.2.12，那麽使用这个 netmask 时，你的 network 就会是 192.0.2.12 AND 255.255.255.0 所得到的值：192.0.2.0］。

无庸置疑的，这样的分类对於 Internet 的最终需求而言并不够细腻；我们已经以相当快的速度在消耗 Class C 网路，这是我们都知道一定会耗尽的 Class，所以不用费心去想了。补救的方式是，要能接受任意个 bits 的 netmask，而不单纯是 8丶16 或 24 个而已。所以你可以有个 255.255.255.252 的 netmask，这个 netmask 能切出一个 30 个 bits 的 network 及 2 个 bits 的 host，这个 network 最多有四台 hosts［注意，netmask 的格式永远都是：前面是一连串的 1，然後，後面是一连串的 0］。

不过一大串的数字会有点不好用，比如像 255.192.0.0 这样的 netmask。首要是人们无法直觉地知道有多少个 bits 的 1；其次是这样真的很不严谨。因此，後来的新方法就好多了。你只需要将一个斜线放在 IP address 後面，接着後面跟着一个十进制的数字用以表示 network bits 的数目，类似这样：192.0.2.12/30。

或者在 IPv6 中，类似这样：2001:db8::/32 或 2001:db8:5413:4028::9db9/64。

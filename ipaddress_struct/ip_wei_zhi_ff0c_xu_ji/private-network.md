# 3.4.1. Private Network

很多地方都有防火墙（firewall），由它们的保护将网路隐藏於世界的其它地方。而有时，防火墙会用所谓的网路地址转换（NAT，Network Address Translation）的方法，将＂internal＂（内部的）IP 地址转换为＂external＂（外部的）［世界上的每个人都知道的］IP address。

你又开始紧张了吗？＂他又要扯到哪里去了？＂

好啦，放轻松，去买瓶汽水［或酒精］饮料，因为身为一个初学者，你还可以不要理会 NAT，因为它所做的事情对你而言是透明的。不过我想在你开始对所见的网路数量开始感到困惑以前，谈谈防火墙身後的网路。

比如，我家有一个防火墙，我有两个 DSL 电信公司分配给我的 static IPv4 地址，而我家的网路有七部电脑要用。这有可能吗？两台电脑不能共用同一个 IP address 阿，不然数据就不知道该送去哪一台电脑了！

答案揭晓：它们不会共用同一个 IP address，它们是在一个配有两千四百万个 IP address 的 private network 上，全部都是我的。

好，都是我的，有这麽多地址可以让大家用来上网，而这里要讲的就是为什麽：

如果我登入到一台远端的电脑，它会说我从 192.0.2.33 登入，这是我的 ISP 提供给我的 public IP。不过若是我问我自己本地端节点的电脑，它的 IP address 是什麽时，他会说是 10.0.0.5。是谁转换 IP 的呢？答对了，就是防火墙！它做了 NAT！

10.x.x.x 是其中一个少数保留的网路，只能用在完全无法连上 Internet 的网路［disconnected network］，或是在防火墙後的网路。你可以使用哪个 private network 编号的细节是记在 RFC 1918 \[15]中，不过一般而言，你较常见的是 10.x.x.x 及 192.168.x.x，这里的 x 是指 0-255。较少见的是 172.y.x.x，这里的 y 范围在 16 与 31 之间。

在 NAT 防火墙後的网路可以不必用这些保留的网路，不过它们通常会用。

［真好玩！我的外部 IP 真的不是 192.0.2.33，192.0.2.x 网路保留用来虚构本教程要用的 ＂真实＂IP address，就像本教程也是虚构的一样 Wowzers！］

IPv6 也很合理的会有 private network。它们最早的前缀是 fdxx:［或者未来可能是 fcXX：］，如同 RFC 4193 \[16]。NAT 与 IPv6 通常不会混用，然而［除非你在做 IPv6 到 IPv4 的 gateway，这就不在本教程的讨论范围内了］，理论上，你会有很多地址可以使用，所以根本不再需要使用 NAT。不过，如果你想要在不会路由到外面的网路［封闭网路］上分配地址给你自己，就用 NAT 吧。

\[15] [http://tools.ietf.org/html/rfc1918](http://www.google.com/url?q=http%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc1918\&sa=D\&sntz=1\&usg=AOvVaw3dW23WM-mk9E4MlvDTougj)

\[16] [http://tools.ietf.org/html/rfc4193](http://www.google.com/url?q=http%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc4193\&sa=D\&sntz=1\&usg=AOvVaw09oi8eFPkBTmmcSAuFoOO-)

# 3.1.2. Port Number（连接埠号码）

如果你还记得我之前跟你说过的分层网路模型（Layered Network Model），它将网路层（IP）与主机到主机间的传输层［TCP 与 UDP］分开。

我们要加快脚步了。

除了 IP address 之外［IP 层］，有另一个 TCP［stream socket］使用的地址，刚好 UDP［datagram socket］也是。它就是 port number，这是一个 16-bit 的数字，就像是连线的本地端地址一样。

将 IP address 想成饭店的地址，而 port number 就是饭店的房间号码。这是贴切的比喻；或许以後我会用汽车工业来比喻。

你说想要有一台电脑能处理收到的电子邮件与网页服务－你要如何在一台只有一个 IP 地址的电脑上分辨呢？

好，Internet 上不同的服务都有已知的（well-known）port numbers。你可以在 Big IANA Port 表 \[12] 中找到，或者若你在 Unix 系统上，你可以参考文件 /etc/services。HTTP（网站）是 port 80丶telnet 是 port 23丶SMTP 是 port 25，而 DOOM 游戏 \[13] 使用 port 666 等，诸如此类。Port 1024 以下通常是有特地用途的，而且要有操作系统管理员权限才能使用。

摁，这就是 port number 的介绍。

\[12] http://www.iana.org/assignments/port-numbers

\[13] http://en.wikipedia.org/wiki/Doom\_(video\_game)

# 2.2. 底层漫谈与网路理论

嘿！孩子们，该是学习数据封装（Data Encapsulation）的时候了。这很重要。它就是非常的重要，所以如果你是在加州这里上的网路课程，也只能学到皮毛。

基本上我们会讲到这些：数据包的诞生丶将数据包打包［＂封装＂］到第一个协议［所谓的 TFTP 协议］的 header 中［几乎是最底层了］，接着将全部的东西［包含 TFTP header］封装到下一个协议中［所谓的 UDP］，接着下一个协议［IP］，最後衔接到硬件［实体］层上面的协议［所谓的 Ethernet，乙太网路］。

当另一台电脑收到数据包时，硬件会解开 Ethernet header，而 kernel 会解开 IP 与 UDP header，再来由 TFTP 程序解开 TFTP header，最後程序可以取得数据。

现在我最後要谈个声名狼藉的分层网路模型（Layered Network Model），亦称＂ISO/OSI＂。这个网路模型介绍了一个网路功能系统，有许多其它模型的优点。例如，你可以写刚好一样的 socket 程序，而不用管数据在实体上是怎麽传送的［Serial丶thin Ethernet丶AUI 之类］。因为在底层的程序会帮你处理这件事。真正的网路硬件与拓朴对 socket 程序设计师而言是透明的。

不罗嗦，我将介绍这个成熟模型的分层。为了网路课程的测验，要记住这些。

* Application（应用层）
* Presentation（表现层）
* Session（会谈层）
* Transport（传输层）
* Network（网路层）
* Data Link（数据链结层）
* Physical（实体层）

实体层就是硬件（serial丶Ethernet 等）。而应用层你可以尽可能的想像，这是个用户与网路互动的地方。

现在这个模型已经很普及，所以你如果愿意的话，或许可以将它当作是一本汽车修理指南来使用。与 Unix 比较相容的分层模型有：

* 应用层（Application layer：telnet丶ftp 等）
* 主机到主机的传输层（Transport layer：TCP丶UDP）
* 互联网层（Internet layer：IP 与路由遶送）
* 网路存取层（Network Access Layer：Ethernet丶wi-fi丶诸如此类）

此时，你或许能知道这几层是如何对应到原始数据的封装。

看看在打造一个简单的数据包需要多少工作呢？

天阿！你得自己用＂cat＂将数据填入数据包的 header 里！

开玩笑的啦。

你对 stream socket 需要做的只有用 send() 将数据送出。而在 datagram socket 需要你做的是，用你所选择的方式封装该数据包，并且用 sendto() 送出。Kernel 会自动帮你建立传输层与网路层，而硬件处理网路存取层。啊！真现代化的技术。

所以该结束我们短暂的网路理论之旅了。

喔！对了，我忘记告诉你我想要谈谈 routing（路由）了。恩，没事！没关系，我不打算全部讲完。

Router（路由器）会解开数据包的 IP header，参考自己的 routing table（路由表）…。如果你真的很想知道，你可以读 IP RFC \[9]。如果你永远都不想碰它，其实你也可以过得很好。

\[9] [http://tools.ietf.org/html/rfc791](http://www.google.com/url?q=http%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc791\&sa=D\&sntz=1\&usg=AOvVaw3DMRoLjyzQzt-AeCtgAZ43)

译注：

若读者想要深入了解互联网或 TCP/IP 观念，以下是译者推荐的参考文献，这些都是网路 TCP/IP 观念的经典着作。

\[C1] Behrouz Forouzan, [TCP/IP Protocol Suite](http://www.google.com/url?q=http%3A%2F%2Fwww.amazon.com%2Fgp%2Fproduct%2F0073376043%2Fref%3Das\_li\_tf\_tl%3Fie%3DUTF8%26camp%3D1789%26creative%3D9325%26creativeASIN%3D0073376043%26linkCode%3Das2%26tag%3Dapla0fb9-20\&sa=D\&sntz=1\&usg=AOvVaw0aBv4fWUC2rKA373nsWVhh), 4 edition, Mcgraw-Hill Inc., 2009.

\[C2] Kevin R. Fall and W. Richard Stevens, [TCP/IP Illustrated, The Protocols](http://www.google.com/url?q=http%3A%2F%2Fwww.amazon.com%2Fgp%2Fproduct%2F0321336313%2Fref%3Das\_li\_tf\_tl%3Fie%3DUTF8%26camp%3D1789%26creative%3D9325%26creativeASIN%3D0321336313%26linkCode%3Das2%26tag%3Dapla0fb9-20\&sa=D\&sntz=1\&usg=AOvVaw0a8x8MjgHUvYM-WuYeuOX0), Vol. 1, 2 edition, Addison-Wesley Inc., 2011.

\[C3] Douglas E. Comer, [Internetworking with TCP/IP principles, protocols, and architecture](http://www.google.com/url?q=http%3A%2F%2Fwww.amazon.com%2Fgp%2Fproduct%2F013608530X%2Fref%3Das\_li\_tf\_tl%3Fie%3DUTF8%26camp%3D1789%26creative%3D9325%26creativeASIN%3D013608530X%26linkCode%3Das2%26tag%3Dapla0fb9-20\&sa=D\&sntz=1\&usg=AOvVaw2DXvVtfo6LzZM1bNzZWRd8), Vol. 1, 6 edition, Pearson education Inc., 2013.

\[C4] Pat Eyler, Networking Linux: A Practical Guide to TCP/IP, New Riders Inc., 2001.

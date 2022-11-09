# 2.1. 两种 Internet Sockets

这是什麽？有两种 Internet sockets 吗？

是的，喔不，我骗你的啦。其实有更多种 Internet sockets，只是我不想吓唬你。所以这里我只打算讨论两种，不过我还会告诉你＂Raw Sockets＂，这是很强大的东西，所以你应该要好好研究一下它们。

译注：

一般的 socket 只能读取 transport layer 传输层以上［不含］的数据，raw socket 一般用在设计 network sniffer，可以让应用程序取得网路数据包底层的数据［如 TCP 层丶IP 层，甚至 link layer socket 可以读取到 link layer 层］，并用以分析数据包。这份教程不会谈到这类的编程，有兴趣的读者可自行参考：[Unix Network Programming Vol. 1](http://www.google.com/url?q=http%3A%2F%2Fwww.amazon.com%2Fgp%2Fproduct%2F0131411551%2Fref%3Das\_li\_tf\_tl%3Fie%3DUTF8%26camp%3D1789%26creative%3D9325%26creativeASIN%3D0131411551%26linkCode%3Das2%26tag%3Dapla0fb9-20\&sa=D\&sntz=1\&usg=AOvVaw3WmiGfMTYE25kH5xUOzW7j)丶[TCP/IP 网路程式实验与设计](http://www.google.com/url?q=http%3A%2F%2Fwww.books.com.tw%2Fexep%2Fassp.php%2Fmyliao2007%2Fproducts%2F0010384126%3Futm\_source%3Dmyliao2007%26utm\_medium%3Dap-books%26utm\_content%3Drecommend%26utm\_campaign%3Dap-201404\&sa=D\&sntz=1\&usg=AOvVaw1GRHMXp5KPC2PD7QXpxurF)或[ libpcap](http://www.google.com/url?q=http%3A%2F%2Fsourceforge.net%2Fprojects%2Flibpcap%2F\&sa=D\&sntz=1\&usg=AOvVaw0n\_SibM2GOKK\_PdgOj1kI3)。

好吧，不聊了。那到底是有哪两种 Internet sockets 呢？

其中一个是＂Stream Sockets＂（串流式 Sockets）；而另一个是＂Datagram Sockets＂（讯息式 Sockets），之後我们分别以＂SOCK\_STREAM＂与＂SOCK\_DGRAM＂来表示。Datagram sockets 有时称为＂无连接的 sockets＂（connectionless sockets）（虽然它们也可以用 connect()，如果你想这麽做的话，请见後面章节的 connect()）。

Stream sockets 是可靠的丶双向连接的通讯串流。若你以＂1丶2＂的顺序将两个项目输出到 socket，它们在另一端则会以＂1丶2＂的顺序抵达。而且不会出错。

哪里会用到 stream sockets 呢？

好的，你应该听过 telnet 软件吧，不是吗？它就是用 stream sockets。你所输入的每个字都需要按照你所输入的顺序抵达，有吗？网页浏览器所使用的 HTTP 协议也是用 stream sockets 取得网页。的确，若你以 port 80 telnet 到一个网站，并输入＂GET / HTTP/1.0＂，然後按两下 Enter，它就会输出 HTML 给你！

Stream sockets 是如何达成如此高品质的数据传送呢？

它们用所谓的＂The Transmission Control Protocol＂（传输控制协议），就是常见的＂TCP＂（TCP 的全部细节请参考 RFC 793\[6]）。TCP 保证你的数据可以依序抵达而且不会出错。你以前可能听过＂TCP＂是＂TCP/IP＂比较优的部分，这边的＂IP＂是指＂Internet Protocol＂（互联网协议，请见 RFC 791\[7]）。IP 主要处理 Internet routing（互联网路由），通常不保障数据的完整性。

酷喔。那 Datagram socket 呢？为什麽它们号称无连接呢？这边有什麽好主意？为什麽它们是不可靠的？

好，这里说明一下现况：如果你送出一个 datagram（信息数据包），它可能会顺利到达丶可能不会按照顺序到达，而如果它到达了，数据包的数据就是正确的。

译注：

TCP 会在传输层对将上层送来的过大数据分割成多个 TCP 段（TCP segments），而 UDP 本身不会，UDP 是信息导向的（message oriented），若 UDP 信息过大时（整体数据包长度超过 MTU），则会由 host 或 router 在 IP 层对数据包进行分割，将一个 IP packet 分割成多个 IP fragments。IP fragmention 的缺点是，到达端的系统需要做 IP 数据包的重组，将多个 fragments 重组合并为原本的 IP 数据包，同时也会增加数据包遗失的可能性。如将一个 IP packet 分割成多个 IP fragments，只要其中一个 IP fragment 遗失了，到达端就会无法顺利重组 IP 数据包，因而造成数据包的遗失，若是高可靠度的应用，则上层协议需重送整个 packet 的数据。

\[6] [http://tools.ietf.org/html/rfc793](http://www.google.com/url?q=http%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc793\&sa=D\&sntz=1\&usg=AOvVaw0OHAByfOI7DYJA1-tiRA5z)

\[7] [http://tools.ietf.org/html/rfc791](http://www.google.com/url?q=http%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc791\&sa=D\&sntz=1\&usg=AOvVaw3DMRoLjyzQzt-AeCtgAZ43)

Datagram sockets 也使用 IP 进行 routing（路由），不过它们不用 TCP；而是用＂UDP，User Datagram Protocol＂（用户数据包协议，请见 RFC 768 \[8]）。

为什麽它们是无连接的？

好，基本上，这跟你在使用 stream socket 时不同，你不用维护一个开启的连接，你只需打造数据包丶给它一个 IP header 与目的资料丶送出，不需要连接。通常用 datagram socket 的时机是在没有可用的 TCP stack 时；或者当一些数据包遗失不会造成什麽重大事故时。这类应用程序的例子有：tftp（trivial file transfer protocol，简易文件传输协议，是 FTP 的小兄弟），多人游戏丶串流音乐丶影像会议等。

＂等一下！tftp 和 dhcpd 是用来在一台主机与另一台之间传输二进制的应用数据！你如果想要应用程序能在数据抵达时正常运作，那数据就不能遗失阿！这是什麽黑魔法？＂

好，我的人类朋友，tftp 与类似的程序会在 UDP 的上层使用它们自己的协议。比如：tftp 协议会报告每个收送的数据包，到达端必须送回一个数据包表示：＂我收到了！＂［一个 ＂ACK＂数据包］。若原本数据包的传送端在五秒内没有收到回应，这表示它该重送这个数据包，直到收到 ACK 为止。在实作可靠的 SOCK\_DGRAM 应用程序时，这个回报的过程很重要。

对於无需可靠度的（unreliable）应用程序，如游戏丶音效丶或影像，你只需忽略遗失的数据包，或也许能试着用技巧弥补回来。（雷神之锤的玩家都知道的一个技术名词的影响：accursed lag。在这个例子中，＂accursed＂（受到诅咒）这个字代表各种低级的意思）。

为什麽你要用一个不可靠的底层协议？

有两个理由：第一个理由是速度，第二个理由还是速度。忘了这个数据包是比较快的方式，相较之下，持续追踪全部的数据包否安全抵达，并确保依序抵达是比较慢的。如果你想要传送聊天讯息，TCP 很好；不过如果你想要替全世界的玩家，每秒送出 40 个位置更新的数据，且若遗失一到两个数据包并不会有太大的影响时，此时 UDP 是一个好的选择。

\[8] [http://tools.ietf.org/html/rfc768](http://www.google.com/url?q=http%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc768\&sa=D\&sntz=1\&usg=AOvVaw0bXb6OOeSwzIy3yEAxSPcl)

译注：

stream（串流式）socket 是指应用程序要传输的数据就如水流（串流）在水管中传输一般，经由这个 stream socket 流向目的，串流式 socket 是数据会由传输层负责处理遗失丶依序送达等工作，以在传输层确保应用程序所送出的数据能够可靠且依序抵达，而应用程序若对数据有可靠与依序的需求时，使用 stream socket 就不用自行处理这类的工作。

datagram（信息式）socket 是基於讯息导向的方式传送数据，应用程序送出的每笔数据会如平信的概念送出，由於遶送数据包的路径可能会随着网路条件而改变，每笔数据抵达的顺序不一定会按照送出的顺序抵达，并且如平信般，信件可能在递送过程遗失，而寄件人并无法知道是否递送成功。

初步简单知道应用这两种 sockets 的时机：当需要数据能完整送达目地时，就使用 stream socket，若是部分数据遗失也无妨时，就可以使用 datagram socket。

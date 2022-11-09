# 8. 常见的问题

**我可以从哪边取得那些 header 文件呢?**\
\
如果你的系统还没有这些文件，你可能就不需要它们。检查你平台的手册。若你在 Windows 上开发，那麽你只需要 #include \<winsock.h>。\
\
**在 bind() 回报＂Address already in use＂（地址已经在使用中）时，我该怎麽办呢？**\
\
你必须使用 setsockopt() 对 listen 的 socket 设置 SO\_REUSEADDR 选项。请参考 bind() 及 select() 章节的示例。\
\
**我该如何取得系统上已经开启的 sockets 表呢？**\
\
使用 **netstat**。细节请参考 man 手册，不过你应该只要输入下列的命令就能取得一些不错的资料：\


```
$ netstat
```

\
**我该如何检视 routing table（路由表）呢？**\
\
使用 **route** 命令（多数的 Linux 系统是在 /sbin 底下），或者 **netstat -r** 命令。\
\
**如果我只有一台电脑，我该如何运行 client（客户）与 server（服务器）程序呢？我需要网路来做网路编程吗？**\
\
你很幸运，全部的系统都有实现一个 loopback 虚拟网路 ＂设备＂（device），这个设备位於 kernel（内核），并假装是个网卡［这个接口就是 routing table 中所列出的 ＂lo＂］。\
\
假装你已经登录一个名为＂goat＂的系统，在一个窗口运行 client，并在另一个窗口运行 server。\
\
或者可以在背景运行 server［＂server &＂］，并在同样的窗口运行 client。\
\
loopback 设备的功能是你可以运行 **client goat** 或 **client localhost**［因为＂localhost＂应该已经定义在你的 /etc/hosts 文件］，而你可以让 client 与 server 通讯而不需要网路。\
\
简而言之，不需要改变任何代码，就可以让程序在无网路的本地端系统上运行！好耶！\
\
**我该怎麽识别对方已经关闭连接呢？**\
\
你可以辨别出来，因为 recv() 会返回 0。\
\
**我该如何实现一个＂ping＂工具呢？什麽是 ICMP 呢？我可以在哪里找到更多关於 raw socket 与 SOCK\_RAW 的资料呢？**\
\
你对 raw socket 的全部疑问都可以在 W. Richard Stevens 的 UNIX Network Programming 书本上找到答案。还有，研究 Stevens 的 UNIX Network Programming 代码的 ping 子目录，可以在线下载 \[37]。\
\
**我该如何改变或减少调用 connect() 的 timeout（超时）时间呢？**\
\
我不想跟你说一样的答案：＂W. Richard Stevens 会告诉你＂，我只能建议你阅读 UNIX Network Programming 代码 \[38] 中的 /lib/connect\_nonb.c。\
\
主要是你要用 socket() 建立一个 socket descriptor，将它设置为 non-blocking（非阻塞），调用 **connect()**，而如果一切顺利，**connect()**会立馬返回 -1，并将 errno 设置为 EINPROGRESS。接着你要调用 **select()** 并设置你想要的 timeout 时间，传递读写组（read and write sets）的 socket descriptor。如果 select() 没有发生 timeout，这表示 connect() call 已经完成。此时，你必须使用 **getsockopt()** 设置 SO\_ERROR 选择项以取得 connect() call 的返回值，在没有错误时，这个值应该是零。\
\
最後，在你开始透过 socket 传输数据以前，你可能想要再将它设置回 blocking（阻塞）。\
\
要注意的是，这麽做的好处是让你的程序在连接（connecting）期间也可以另外做点事情。比如：你可以将 timeout 时间设定为类似 500 毫秒，并在每次 timeout 发生时更新显示器画面，然後再次调用 select()。当你已经调用了 select() 时，并且 timeout 了，像这样重复了 20 次，你就会知道应该放弃这个连接了。\
\
如我所述的，请参考 Stevens 的既完美又优秀的代码示例。\
\
**我该如何写 Windows 的网路程序呢？**\
\
首先，请删除 Windows，并安装 Linux 或 BSD。;-)。不是的，实际上，只要参考前言中的［Windows 程序设计师要注意的事情］就可以了。\
\
**我该如何在 Solaris/SunOS 上编译程序呢？在我尝试编译时，一直遇到错误！**\
\
发生 Linker（链接器）错误是因为 Sun 系统在不会自动编入 socket 程序库。请参考前言中的［Solaris/SunOS 程序员要注意的事情］，有如何处理这个问题的示例。\
\
**为什麽 select() 一直跟 signal 吵架呢？**\
\
Signal 试图要让 blocked system call 返回 -1，并将 errno 设置为 EINTR。当你用 **sigaction()** 设置了一个 signal handler（信号处理例程）时，你可以设置 SA\_RESTART flag，这可以在 system call 被中断之後重新打开它。\
\
很自然的是这不会每次都管用。\
\
我最爱的解法是使用一个 goto，你明白这会让你的教授很愤怒，所以放手去做吧！\


<pre class="language-c"><code class="lang-c">select_restart:
if ((err = select(fdmax+1, &#x26;readfds, NULL, NULL, NULL)) == -1) {
    if (errno == EINTR) {
<strong>        // 某个 signal 中断了我们，所以重新启动
</strong>        goto select_restart;
    }
    // 这里处理真正的错误：
    perror("select");
}</code></pre>

当然，在这个例子里，你不需使用 goto；你可以用其它的 structures 来控制，但是我认为用 goto 比较乾净。\
\
**要怎麽样我才能实作调用 recv() 的 timeout 呢？**\
\
使用 **select()**！它可以让你对正在读取的 socket descriptors 指定 timeout 的参数。或者你可以将整个功能包在一个独立的函数中，类似这样：\


```c
#include <unistd.h>
#include <sys/time.h>
#include <sys/types.h>
#include <sys/socket.h>

int recvtimeout(int s, char *buf, int len, int timeout)
{
  fd_set fds;
  int n;
  struct timeval tv;

  // 设置 file descriptor set
  FD_ZERO(&fds);
  FD_SET(s, &fds);

  // 设置 timeout 的数据结构 struct timeval
  tv.tv_sec = timeout;
  tv.tv_usec = 0;

  // 一直等到 timeout 或收到数据
  n = select(s+1, &fds, NULL, NULL, &tv);
  if (n == 0) return -2; // timeout!
  if (n == -1) return -1; // error

  // 数据一定有在这里，所以调用一般的 recv()
  return recv(s, buf, len, 0);
}
.
.
.
// 调用 recvtimeout() 的示例：
n = recvtimeout(s, buf, sizeof buf, 10); // 10 second timeout

if (n == -1) {
  // 发生错误
  perror("recvtimeout");
}
else if (n == -2) {
  // 发生 timeout
} else {
  // 从 buf 收到一些数据
}
.
.
.
```

请注意，**recvtimeout()** 在 timeout 的例子中会返回 -2，那为什麽不是返回 0 呢？好的，如果你还记得，在调用 recv() 返回 0 值时所代表的意思是对方已经关闭了连接。所以该返回值已经用过了，而 -1 表示＂错误＂，所以我选择 -2 做为我的 timeout 表示。\
\
**我该如何在将数据送给 socket 以前将数据加密或压缩呢？**\
\
一个简单的加密方法是使用 SSL（secure sockets layer），只是这超过本教程的范畴了［细节请参考 OpenSSL project \[39]］。\
\
不过假设你想要安插或实现你自己的压缩器（compressor）或加密系统（encryption system），这只不过是将你的数据想成在两个节点间运行连续的步骤，每个步骤以同样的方式改变数据：\
\
1\. server 从文件读取数据［或是什麽地方］\
\
2\. server 加密/压缩数据［你新增这个部分］\
\
3\. server 用 send() 送出加密数据\
\
而另一边则是：\
\
1\. client 用 recv() 接收加密数据\
\
2\. client 译码/解压数据［你新增这个部分］\
\
3\. client 写数据到文件［或是什麽地方］\
\
如果你正要压缩与加密，只要记得先压缩。:-)\
\
只要 client 适当地还原 server 所做的事情，数据在另一端就会完好如初，不论你在中间增加了多少步骤。\
\
所以你用我的代码所需要做的只有：找出读数据与透过网路传送［使用 **send()**］这中间的段落，并在那里加上编码的代码。\
\
**我一直看到的 ＂PF\_INET＂是什麽呢？他跟 AF\_INET 有关系吗？**\
\
是的，有关系，细节请参考 socket() 章节。\
\
**我该怎麽写一个 server，可以接受来自 client 的 shell 命令并运行命令呢？**\
\
为了简化，我们说 client 的连接用 connect()丶send() 以及 close()［即为，没有後续的 system calls，client 没有再次连接。］\
\
client 的处理过程是：\
\
1\. 用 connect() 连接到 server\
\
2\. send("/sbin/ls > /tmp/client.out")\
\
3\. 用 close() 关闭连接\
\
此时，server 正在处理数据并运行命令：\
\
1\. accept() client 的连接\
\
2\. 使用 recv(str) 接收命令字符串\
\
3\. 用 close() 关闭连接\
\
4\. 用 system(str) 运行命令\
\
注意！server 会运行全部 client 所送的命令，就像是提供了远端的 shell 权限，人们可以连接到你的 server 并用你的帐户做点事情。例如：若 client 送出 ＂rm -rf \~＂会怎麽样呢？这会删掉你帐户里全部的数据，就是这样！\
\
所以你学聪明了，你会避免 client 使用任何危险的工具，比如 **foobar** 工具：\


```c
if (!strncmp(str, "foobar", 6)) {
    sprintf(sysstr, "%s > /tmp/server.out", str);
    system(sysstr);
}
```

可是你还是不安全，没错：如果 client 输入 ＂**foobar; rm -rf \~**＂呢？\
\
最安全的方式是写一个小机制，将命令参数中的非字母数字字符前面放个［＂\＂］字符［如果适合的话，要包括空白］。\
\
如你所见，当 server 开始运行 client 送来的东西时，安全（security）是个问题。\
\
**我正在传送大量数据，可是当我 recv() 时，它一次只收到 536 bytes 或 1460 bytes。可是如果我在我本地端运行，它就会一次就收到全部的数据，这是怎麽回事呢？**\
\
你碰到的是 MTU，即 physical medium（物理媒体）能处理的最大尺寸。在本地端上，你用的是 loopback 设备，它可以处理 8K 或更多数据也没有问题。但是在 Ethernet（以太网），它只能处理 1500 bytes［有 header］，你碰到这个限制。透过 modem 的话，MTU 是 576 bytes［一样，有 header］，你遇到比较低的限制。\
\
你必须确认有送出全部的数据。［细节请参考 **sendall()** 函数的实务］。一旦你有确认，那麽你就需要在循环中调用 recv()，直到收到全部的数据。\
\
对於使用多重调用 **recv()** 来接收完整数据包的细节，请参考数据封装之子（Son of Data Encapsulation）一节。\
\
**我用的是 Windows 系统，而且我没有 fork() system call 或任何的 struct sigaction 可以用，该怎麽办呢？**\
\
如果你问的是它们在哪里，它们会在 POSIX 程序库里，这个会包装在你的编译器中。因为我没有 Windows 系统，所以我真的无法回答你，不过我似乎记得 Microsoft 有一个 POSIX 兼容层，那里会有 **fork()**。［而且甚至会有 sigaction。］\
\
在 VC++ 的手册搜寻 ＂fork＂ 或 ＂POSIX＂，看它是否能给你什麽线索。\
\
如果这样一点都没有用，拿掉 fork()/sigaction 这些东西，用 Win32 中等价的函数来替换：**CreateProcess()**。我不知道怎麽用**CreateProcess()**，它有多的数不清的参数，不过在 VC++ 的资料中应该可以找到怎麽使用它。\
\
**我在防火墙（firewall）後面，我该如何让防火墙外面的人知道我的 IP 地址，让他们可以连接到我的电脑呢？**\
\
毫无疑问地，防火墙的目的就是要防止防火墙外面的人连到防火墙里面的电脑，所以你让他们进来基本上会被认为是安全漏洞。\
\
但也不是说完全不行，有一个方法，你仍然可以透过防火墙频繁的进行 **connect()**，如果防火墙是使用某种伪装（masquerading）或 NAT 或类似的方式。你只要让程序一直在做初始化连接，那麽你有机会成功的。\
\
如果这样还不是很满意，你可以要求系统管理员在防火墙开一个小洞（hole），让人们可以连进你的电脑。防火墙可以透过 NAT 软件或 proxy（代理）或类似的方法将数据包转送给你。\
\
要留意，不要对防火墙中的一个小洞掉以轻心。你必须确保你不会放坏人进来存取内网；如果你是新手，做软件安全的难度是远远超过你的想像。\
\
不要让你的系统管理员对我发脾气 ;-)\
\
**我该怎麽写 packet sniffer 呢？我要怎麽将我的 Ethernet interface 设置为 promiscuous mode（混杂模式）呢？**\
\
这些事情是在底层运作的，当网卡设置为＂promiscuous mode＂时，它会转送全部的数据给操作系统，而不只是地址属於这台电脑的数据包而已。［我们这里谈的是 Ethernet 层的地址，而不是 IP 地址，可是因为 ethernet 是在 IP 底层，所以全部的 IP 地址实际上都会转送。细节请参考＂底层漫谈与网路理论＂一节］。\
\
这是 packet sniffer 如何运作的基础，它将网卡设置为 promiscuous mode，接着 OS 会收到经过网线的每个数据，你会有一个可以用来读取数据的某种型别 socket。\
\
毫无疑问地，这个问题的答案依平台而异，不过如果你用百度或 Google 搜寻，例如：＂windows promiscuous ioctl＂，你或许会在某个地方找到，看起来跟 Linux Journal \[40] 中写的一样好的。\
\
**我该如何为 TCP 或 UDP socket 设定一个自订的 timeout 值呢？**\
\
这个按照你的系统而定，你可以在网上搜寻 SO\_RCVTIMEO 与 SO\_SNDTIMEO［用在 **setsockopt()**］，看看是否你的系统有支持这样的功能。\
\
Linux man 手册建议使用 **alarm()** 或 **setitimer()** 作为替代品。\
\
**我要如何辨别哪些 ports 可以使用呢？有没有＂官方＂的 port numbers 呢？**\
\
通常这不会有问题，如果你正在写像 web server 这样的程序，那麽在你的程序使用 port 80 是个好主意。如果你只是想要写自己的 server，那麽随机选择一个 port［不过要大於 1023］，然後试试看。\
\
如果 port 已经在使用中，你将会在尝试 **bind()** 时遇到＂Address already in use＂错误。选择另一个 port。［利用 config 配置文件或命令行参数，让你的软件用户能指定 port 也是个不错的想法］。\
\
有一个官方的 port nubmer \[41] 表，由 Internet Assigned Numbers Authority（IANA）所维护的。在表中的 ports［超过 1023］并不代表你不能使用，比如，Id 软件的 DOOM 跟 ＂mdqs＂ 用一样的 port，不管那是什麽，最重要的是_在同一台机器上_没有人用掉你要用的 port。\
\
\[37] http://www.unpbook.com/src.html\
\[38] http://www.unpbook.com/src.html\
\[39] http://www.openssl.org/\
\[40] http://interactive.linuxjournal.com/article/4659\
\[41] http://www.iana.org/assignments/port-numbers

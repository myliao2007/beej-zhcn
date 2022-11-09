# 7.5. 数据封装

不管怎样，意思真的是封装数据吗？以最简单的例子而言，这表示你会需要增加一个 header（标头），用来代表识别的资料或数据长度，或者都有。\
\
你的 header 看起来像什麽呢？\
\
好的，它就只是某个用来表示你觉得完成专案会需要的二进制数据。\
\
哇，好抽象。\
\
Okay，举例来说，咱们说你有一个使用 SOCK\_STREAM 的多重用户聊天程序。当某个用户输入［＂says＂］某些字，会有两笔资料要传送给 server：\
\
＂谁＂以及＂说了什麽＂。\
\
到目前为止都还可以吗？\
\
你问：＂会有什麽问题吗？＂\
\
问题是讯息的长度是会变动的。一个叫做 ＂tom＂的人可能会说 ＂Hi（嗨）＂，而另一个叫做＂Benjamin（班杰明）＂的人可能说：＂Hey guys what is up？（嘿！兄弟最近你好吗？）＂\
\
所以你在收到全部的数据之後，将它全部 **send()** 给 clients。你输出的 data stream（数据串流）类似这样：\


```
t o m H i B e n j a m i n H e y g u y s w h a t i s u p ?
```

类似这样。那 client 要如何知道讯息何时开始与结束呢？\
\
如果你愿意，是可以的，只要让全部的讯息都一样长，并只要调用我们之前实作的 **sendall()** 就行了。但是这样会浪费带宽（bandwidth）！我们并不想用 **send()** 送出了 1024 个 bytes 的数据，却只有携带了 ＂tome＂说了 ＂Hi＂这样的有效资料。\
\
所以我们以小巧的 header 与数据包结构封装（encapsulate）数据。Client 与 server 都知道如何封装（pack）与解封装（unpack）这笔数据［有时候称为 ＂marshal＂与 ＂unmarshal＂］。现在先不要想太多，我们会开始定义一个协议（protocol），用来描述 client 与 server 是如何沟通的！\
\
在这个例子中，咱们假设用户的名称是固定 8 个字符，并用 '\0' 结尾。然後接着让我们假设数据的长度是变动的，最多高达 128 个字符。我们看个可能在这个情况会用到的数据包结构范例。\
\
1\. len［1 个 byte，unsigned（无号）］：数据包的总长度，计算 8 个 bytes 的用户名称，以及聊天数据。\
\
2\. name［8 个 bytes］：用户名称，如果有需要，结尾补上 NUL。\
\
3\. chatdata［n 个 bytes］：数据本身，最多 128 bytes。数据包的长度应该要以这个数据长度加 8 ［上面的 name 栏位长度］来计算。\
\
为什麽我选择 8 个 bytes 与 128 个 bytes 长度的栏位呢？我假设这样就已经够用了，或许，8 个 bytes 对你的需求而言太少了，你也可以有 30 个 bytes 的 name 栏位，总之，你可以自己决定！\
\
使用上列的数据包定义，第一个数据包由下列的资料组成［以 hex 与 ASCII］：\


```
   0A    74 6F 6D 00 00 00 00 00     48 69
(length) T  o  m    (padding)        H  i
```

而第二个也是差不多：\


```
   18    42 65 6E 6A 61 6D 69 6E       48 65 79 20 67 75 79 73 20 77 ...
(length) B  e  n  j  a  m  i  n        H  e  y     g  u  y  s     w  ...
```

［长度（length）是以 Network Byte Order 储存，当然，在这个例子只有一个 byte，所以没差，但是一般而言，你会想要让你全部的二进制整数能以 Network Byte Order 储存在你的数据包中。］\
\
当你传送数据时，你应该要谨慎点，使用类似前面的 **sendall()** 指令，因而你可以知道全部的数据都有送出，即便要将数据全部送出会多花几次的 **send()**。\
\
同样地，当你接收这笔数据时，你需要额外做些处理。如果要保险一点，你应该假设你可能只会收到部分的数据包内容［如我们可能会从上面的班杰明那里收到 ＂18 42 65 6E 6A＂］，但是我们这次调用 **recv()** 全部就只收到这些数据。我们需要一次又一次的调用**recv()**，直到完整地收到数据包内容。\
\
可是要怎麽做呢？\
\
好的，我们可以知道所要接收的数据包它全部的 byte 数量，因为这个数量会记载在数据包前面。我们也知道最大的数据包大小是 1 + 8 + 128，或者 137 bytes［因为这是我们自己定义的］。\
\
实际上你在这边可以做两件事情，因为你知道每个数据包是以长度（length）做开头，所以你可以调用 **recv()** 只取得数据包长度。接着，你知道长度以後，你就可以再次调用 **recv()**，这时候你就可以正确地指定剩下的数据包长度［或者重复取得全部的数据］，直到你收到完整的数据包内容为止。这个方法的优点是你只需有一个足以存放一个数据包的缓冲区，而缺点是你为了要接收全部的数据，至少调用两次的 **recv()**。\
\
另一个方法是直接调用 **recv()**，并且指定你所要接收的数据包之最大数据量。这样的话，无论你收到多少，都将它写入缓冲区，并最後检查数据包是否完整。当然，你可能会收到下一个数据包的内容，所以你需要有足够的空间。\
\
你所能做的是宣告（declare）一个足以容纳两个数据包的阵列，这是你在数据包到达时，你可以重新建构（reconstruct）数据包的地方。\
\
每次你用 **recv()** 接收数据时，你会将数据接在工作缓冲区（work buffer）的後端，并检查数据包是否完整。在缓冲区中的数据数量大於或等於 数据包 header 中所指定的长度时［+1，因为 header 中的长度没有包含 length 本身的长度］。若缓冲区中的数据长度小於 1，那麽很明显地，数据包是不完整的。你必须针对这种情况做个特别处理，因为第一个 byte 是垃圾，而你不能用它来取得正确的数据包长度。\
\
一旦数据包已经完整接收了，你就可以做你该做的处理，将数据拿来使用，并在用完之後将它从工作缓冲区中移除。\
\
呼呼！Are you juggling that in your head yet？\
\
好的，这里是第二次的冲击：你可能在一次的 **recv()** call 就已经读到了一个数据包的结尾，还读到下一个数据包的内容，即是你的工作缓冲区有一个完整的数据包，以及下一个数据包的一部分！该死的家伙。［但是这就是为什麽你需要让你的工作缓冲区可以容纳两个数据包的原因，就是会发生这种情况！］\
\
因为你从 header 得知第一个数据包的长度，而你也有持续追踪工作缓冲区的数据量，所以你可以相减，并且计算出工作缓冲区中有多少数据是属於第二个［不完整的］数据包的。当你处理完第一个数据包後，你可以将第一个数据包的数据从工作缓冲区中清掉，并将第二个数据包的部分内容移到缓冲区的前面，准备进行下一次的 **recv()**。\
\
［部分读者会注意到，实际地将第二个数据包的部份数据移动到缓冲区的开头需要花费时间，而程序可以写成利用环状缓冲区（circular buffer），就不需要这样做。如果你还是很好奇，可以找一本数据结构的书来读。］\
\
我从未说过这很简单，好吧，我有说过这很简单。而你所需要的只是多练习，然後很快的你就会习惯了。我发誓！\
\
**7.6. 广播数据包（Broadcast Packet）：Hello World！**\
\
到了这里，本文已经谈了如何将数据从一台主机传送到另一台主机。但是，我坚持你可能会需要究极的权力，同时将数据送给多个主机！\
\
用 UDP［只能是 UDP，TCP 不行］与标准的 IPv4，可以透过一种叫作广播（broadcasting）的机制达成。IPv6 不支援广播，所以你必须要采用比较高级的技术－群播（multicasting），很遗憾地，我现在不会讨论这个，我受够了异想天开的未来，我们现在还停留在 32-bit 的 IPv4 世界呢！\
\
可是，请等一下！不管你愿不愿意，你不能走呀，开始说说广播吧。\
\
你必须在将广播数据包送到网路之前，先设置 SO\_BROADCAST socket 选项。这类似一个推送导弹开关的小塑胶盖！就只是你的手上掌握了多少的权力。\
\
不过认真说来，使用广播数据包是很危险的，因为每个收到广播数据包的系统都要拨开一层层的数据封装，直到系统知道这笔数据是要送给哪个 port 为止。然後系统会开始处理这笔数据或者丢掉它。在另一种情况，对每部收到广播数据包的机器而言这很费工，因为他们都在同一个区域网路（local network），这样会让很多电脑做不少多馀的工作。当 Doom 游戏出现时，就有人在说它的网路程序写的不好。\
\
现在，有很多方法可以解决这个问题 ... \
\
等一下，真的有很多方法吗？\
\
那是什麽表情阿？哎呀，一样阿，送广播数据包的方法很多。所以重点就是：你该如何指定广播讯息的目地地址呢？\
\
有两种常见的方法：\
\
1\. 将数据送给子网路（subnet）的广播地址，就是将 subnet's network（子网路网段）的 host（主机）那部分全部填 1，举例来说，我家里的网路是 192.168.1.0，而我的 netmask（网路遮罩）是 255.255.255.0，所以地址的最後一个 byte 就是我的 host number［因为依据 netmask，前三个 bytes 是 network number］。所以我的广播地址就是 192.168.1.255。在 Unix 底下，**ifconfig** 指令实际上都会给你这些数据。［如果你有兴趣，取得你广播地址的逻辑运算方式是 network\_number OR (Not netmask) ］。你可以用跟区域网路一样的方式，将这类型的广播数据包送到远端网路（remote network），不过风险是数据包可能会被目地端的　router（路由器）丢弃。［如果 router 没有将数据包丢弃，那麽有个随机的蓝色小精灵会开始用广播流量对它们的区域网路造成水灾。］\
\
2\. 将数据送给 ＂global（全局的）＂广播地址，255.255.255.255，又称为 INADDR\_BROADCAST，很多机器会自动将它与你的 network number 进行 AND bitwise，以转换为网路广播地址，但是有些机器不会这样做。Routers 不会将这类的广播数据包转送（forward）出你的区域网路，够讽刺的。\
\
所以如果你想要将数据送到广播地址，但是没有设置 SO\_BROADCAST socket 选项时会怎样呢？好，我们用之前的 **talker** 与 **listener**来炒冷饭，然後看看会发生什麽事情。\


```shell
$ talker 192.168.1.2 foo
sent 3 bytes to 192.168.1.2
$ talker 192.168.1.255 foo
sendto: Permission denied
$ talker 255.255.255.255 foo
sendto: Permission denied
```

是的，没有很顺利 ... 因为我们没有设置 SO\_BROADCAST socket 选项，设置它，然後现在你就可以用 **sendto()** 将数据送到你想送的地方了！\
\
事实上，这就是 UDP 应用程序能不能广播的差异点。所以我们改一下旧的 **talker** 应用程序，设置 SO\_BROADCAST socket 选项。这样我们就能调用 broadcaster.c 程序了 \[36]：\


```c
/*
** broadcaster.c -- 一个类似 talker.c 的 datagram "client"，
** 差异在於这个可以广播
*/
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>

#define SERVERPORT 4950 // 所要连接的 port

int main(int argc, char *argv[])
{
  int sockfd;
  struct sockaddr_in their_addr; // 连接者的地址资料
  struct hostent *he;
  int numbytes;
  int broadcast = 1;
  //char broadcast = '1'; // 如果上面这行不能用的话，改用这行

  if (argc != 3) {
    fprintf(stderr,"usage: broadcaster hostname message\n");
    exit(1);
  }

  if ((he=gethostbyname(argv[1])) == NULL) { // 取得 host 资料
    perror("gethostbyname");
    exit(1);
  }

  if ((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) == -1) {
    perror("socket");
    exit(1);
  }

  // 这个 call 就是要让 sockfd 可以送广播数据包
  if (setsockopt(sockfd, SOL_SOCKET, SO_BROADCAST, &broadcast,
    sizeof broadcast) == -1) {
    perror("setsockopt (SO_BROADCAST)");
    exit(1);
  }

  their_addr.sin_family = AF_INET; // host byte order
  their_addr.sin_port = htons(SERVERPORT); // short, network byte order
  their_addr.sin_addr = *((struct in_addr *)he->h_addr);
  memset(their_addr.sin_zero, '\0', sizeof their_addr.sin_zero);

  if ((numbytes=sendto(sockfd, argv[2], strlen(argv[2]), 0,
          (struct sockaddr *)&their_addr, sizeof their_addr)) == -1) {
    perror("sendto");
    exit(1);
    }

  printf("sent %d bytes to %s\n", numbytes,
      inet_ntoa(their_addr.sin_addr));

  close(sockfd);

  return 0;
}
```

这个跟 ＂一般的＂ UDP client/server 有什麽不同呢？\
\
没有！［除了 client 可以送出广播数据包］\
\
同样地，我们继续，并在其中一个窗口运行旧版的 UDP **listener** 程序，然後在另一个窗口运行 **broadcaster**，你应该可以顺利运行了。\


```shell
$ broadcaster 192.168.1.2 foo
sent 3 bytes to 192.168.1.2
$ broadcaster 192.168.1.255 foo
sent 3 bytes to 192.168.1.255
$ broadcaster 255.255.255.255 foo
sent 3 bytes to 255.255.255.255
```

而你应该会看到 **listener** 回应说它已经收到数据包。［如果 **listener** 没有回应，可能是因为它绑到 IPv6 地址了，试着将 listener.c 中的 AF\_UNSPEC 改成 AF\_INET，强制使用 IPv4］。\
\
好，真令人兴奋，可是现在要在同一个网路上的另一台电脑运行 **listener**，所以你会有两个复本正在运行，每个机器上各有一个，然後再次用你的广播地址来运行 **broadcaster** ... 嘿！你只有调用一次 **sendto()**，但是两个 **listeners** 都收到了你的数据包。酷喔！\
\
如果 **listener** 收到你直接送给它的数据，但不是在广播地址没有数据，可能是因为你本地端（local machine）上有防火墙（firewall）封锁了这些数据包。［是的，谢谢 Pat 与 Bapper 的说明，让我知道为什麽我的范例程序无法运作。我跟你们说过我会在文件中提到你们，就是这里了，感恩。］\
\
再次提醒，使用广播数据包一定要小心，因为 LAN 上面的每台电脑都会被迫处理这类数据包，无论它们有没有用 **recvfrom()** 接收，这类数据包会造成整个电脑网路相当大的负担，所以一定要谨慎丶适当地使用广播。\
\
\[34] http://beej.us/guide/bgnet/examples/pack2.c\
\[35] http://tools.ietf.org/html/rfc4506\
\[36] http://beej.us/guide/bgnet/examples/broadcaster.c

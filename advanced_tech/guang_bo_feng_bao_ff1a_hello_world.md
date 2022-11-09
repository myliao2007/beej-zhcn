# 7.6. 广播数据包：Hello World！

到了这里，本文已经谈了如何将数据从一台主机发送到另一台主机。但是，我坚持你可能会需要究极的权力，同时将数据送给多个主机！\
\
用 UDP［只能是 UDP，TCP 不行］与标准的 IPv4，可以透过一种叫作广播（broadcasting）的机制达成。IPv6 不支持广播，所以你必须要采用比较高级的技术－群播（multicasting），很遗憾地，我现在不会讨论这个，我受够了异想天开的未来，我们现在还停留在 32-bit 的 IPv4 世界呢！\
\
可是，请等一下！不管你愿不愿意，你不能走呀，开始说说广播吧。\
\
你必须在将广播数据包送到网络之前，先设置 SO\_BROADCAST socket 选项。这类似一个推送导弹开关的小塑胶盖！就只是你的手上掌握了多少的权力。\
\
不过认真说来，使用广播数据包是很危险的，因为每个收到广播数据包的系统都要拨开一层层的数据封装，直到系统知道这笔数据是要送给哪个 port 为止。然后系统会开始处理这笔数据或者丢掉它。在另一种情况，对每部收到广播数据包的机器而言这很费工，因为他们都在同一个局域网（local network），这样会让很多计算机做不少多余的工作。当 Doom 游戏出现时，就有人在说它的网络程式写的不好。\
\
现在，有很多方法可以解决这个问题 ... \
\
等一下，真的有很多方法吗？\
\
那是什么表情阿？哎呀，一样阿，送广播数据包的方法很多。所以重点就是：你该如何指定广播讯息的目地位址呢？\
\
有两种常见的方法：\
\
1\. 将数据送给子网络（subnet）的广播位址，就是将 subnet's network（子网络网段）的 host（主机）那部分全部填 1，举例来说，我家里的网络是 192.168.1.0，而我的 netmask（网络遮罩）是 255.255.255.0，所以位址的最后一个 byte 就是我的 host number［因为依据 netmask，前三个 bytes 是 network number］。所以我的广播位址就是 192.168.1.255。在 Unix 底下，**ifconfig**指令实际上都会给你这些数据。［如果你有兴趣，取得你广播位址的逻辑运算方式是 network\_number OR (Not netmask) ］。你可以用跟局域网一样的方式，将这类型的广播数据包送到远程网络（remote network），不过风险是数据包可能会被目地端的　router（路由器）丢弃。［如果 router 没有将数据包丢弃，那么有个随机的蓝色小精灵会开始用广播流量对它们的局域网造成水灾。］\
\
2\. 将数据送给 ＂global（全局的）＂广播位址，255.255.255.255，又称为 INADDR\_BROADCAST，很多机器会自动将它与你的 network number 进行 AND 比特运算，以转换为网络广播位址，但是有些机器不会这样做。Routers 不会将这类的广播数据包转送（forward）出你的局域网，够讽刺的。\
\
所以如果你想要将数据送到广播位址，但是没有设置 SO\_BROADCAST socket 选项时会怎样呢？好，我们用之前的 **talker** 与**listener** 来炒冷饭，然后看看会发生什么事情。\


```c
$ talker 192.168.1.2 foo
sent 3 bytes to 192.168.1.2
$ talker 192.168.1.255 foo
sendto: Permission denied
$ talker 255.255.255.255 foo
sendto: Permission denied
```

是的，没有很顺利 ... 因为我们没有设置 SO\_BROADCAST socket 选项，设置它，然后现在你就可以用 **sendto()** 将数据送到你想送的地方了！\
\
事实上，这就是 UDP 应用程序能不能广播的差异点。所以我们改一下旧的 **talker** 应用程序，设置 SO\_BROADCAST socket 选项。这样我们就能调用 broadcaster.c 程式了 \[36]：\
\[36] [http://beej.us/guide/bgnet/examples/broadcaster.c](https://web.archive.org/web/20191106174820/http://beej.us/guide/bgnet/examples/broadcaster.c)\


```c
/*
** broadcaster.c -- 一个类似 talker.c 的 datagram "client"，
** 差异在于这个可以广播
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

#define SERVERPORT 4950 // 所要连接数的 port

int main(int argc, char *argv[])
{
  int sockfd;
  struct sockaddr_in their_addr; // 连接数者的位址信息
  struct hostent *he;
  int numbytes;
  int broadcast = 1;
  //char broadcast = '1'; // 如果上面这行不能用的话，改用这行

  if (argc != 3) {
    fprintf(stderr,"usage: broadcaster hostname message\n");
    exit(1);
  }

  if ((he=gethostbyname(argv[1])) == NULL) { // 取得 host 信息
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

这个跟 ＂一般的＂ UDP client/server 有什么不同呢？\
\
没有！［除了 client 可以送出广播数据包］\
\
同样地，我们继续，并在其中一个视窗执行旧版的 UDP **listener** 程式，然后在另一个视窗执行 **broadcaster**，你应该可以顺利执行了。\


```shell
$ broadcaster 192.168.1.2 foo
sent 3 bytes to 192.168.1.2
$ broadcaster 192.168.1.255 foo
sent 3 bytes to 192.168.1.255
$ broadcaster 255.255.255.255 foo
sent 3 bytes to 255.255.255.255
```

而你应该会看到 **listener** 回应说它已经收到数据包。［如果 **listener** 没有回应，可能是因为它绑到 IPv6 位址了，试着将 listener.c 中的 AF\_UNSPEC 改成 AF\_INET，强制使用 IPv4］。\
\
好，真令人兴奋，可是现在要在同一个网络上的另一台计算机执行 **listener**，所以你会有两个复本正在执行，每个机器上各有一个，然后再次用你的广播位址来执行 **broadcaster** ... 嘿！你只有调用一次 **sendto()**，但是两个 **listeners** 都收到了你的数据包。酷喔！\
\
如果 **listener** 收到你直接送给它的数据，但不是在广播位址没有数据，可能是因为你本机（local machine）上有防火墙（firewall）封锁了这些数据包。［是的，谢谢 Pat 与 Bapper 的说明，让我知道为什么我的范例程式无法运作。我跟你们说过我会在文档中提到你们，就是这里了，感恩。］\
\
再次提醒，使用广播数据包一定要小心，因为 LAN 上面的每台计算机都会被迫处理这类数据包，无论它们有没有用 **recvfrom()** 接收，这类数据包会造成整个计算机网络相当大的负担，所以一定要谨慎、适当地使用广播。

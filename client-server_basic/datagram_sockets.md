# 6.3. Datagram Sockets

我们已经在讨论 sendto()与 recvfrom()时涵盖了 UDP datagram socket 的基础，所以我会展示一对示范程序：talker.c 与 listener.c。\
\
Listener 位於一台设备中，等待进入 port 4950 的数据包。Talker 则从指定的机器传送数据包给这个 port，数据包的内容包含用户从命令行所输入的资料。\
\
这里就是 listener.c 的代码 \[22]：\
\


```c
/*
** listener.c -- 一个 datagram sockets "server" 的 demo
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

#define MYPORT "4950" // 用戶所要连线的 port
#define MAXBUFLEN 100

// get sockaddr, IPv4 or IPv6:
void *get_in_addr(struct sockaddr *sa)
{
  if (sa->sa_family == AF_INET) {
  return &(((struct sockaddr_in*)sa)->sin_addr);
}

  return &(((struct sockaddr_in6*)sa)->sin6_addr);
}

int main(void)
{
  int sockfd;
  struct addrinfo hints, *servinfo, *p;
  int rv;
  int numbytes;
  struct sockaddr_storage their_addr;
  char buf[MAXBUFLEN];
  socklen_t addr_len;
  char s[INET6_ADDRSTRLEN];

  memset(&hints, 0, sizeof hints);

  hints.ai_family = AF_UNSPEC; // 设定 AF_INET 以强制使用 IPv4
  hints.ai_socktype = SOCK_DGRAM;
  hints.ai_flags = AI_PASSIVE; // 使用我的 IP

  if ((rv = getaddrinfo(NULL, MYPORT, &hints, &servinfo)) != 0) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rv));
    return 1;
  }

  // 用循环找出全部的结果，并 bind 到首先找到能 bind 的
  for(p = servinfo; p != NULL; p = p->ai_next) {

    if ((sockfd = socket(p->ai_family, p->ai_socktype,
         p->ai_protocol)) == -1) {
      perror("listener: socket");
      continue;
    }

    if (bind(sockfd, p->ai_addr, p->ai_addrlen) == -1) {
      close(sockfd);
      perror("listener: bind");
      continue;
    }

    break;
  }

  if (p == NULL) {
    fprintf(stderr, "listener: failed to bind socket\n");
    return 2;
  }

  freeaddrinfo(servinfo);
  printf("listener: waiting to recvfrom...\n");
  addr_len = sizeof their_addr;

  if ((numbytes = recvfrom(sockfd, buf, MAXBUFLEN-1 , 0, 
    (struct sockaddr *)&their_addr, &addr_len)) == -1) {

    perror("recvfrom");
    exit(1);
  }

  printf("listener: got packet from %s\n",

  inet_ntop(their_addr.ss_family,

  get_in_addr((struct sockaddr *)&their_addr), s, sizeof s));

  printf("listener: packet is %d bytes long\n", numbytes);

  buf[numbytes] = '\0';

  printf("listener: packet contains \"%s\"\n", buf);

  close(sockfd);

  return 0;
}
```

\
要注意的是，在我们调用 getaddrinfo()时，我们是使用 SOCK\_DGRAM。还要注意到，不需要 listen()或是 accept()，这是使用（无连接）datagram sockets 的一个好处！\
\
接着是 talker.c 的代码 \[23]：\
\


```c
/*
** talker.c -- 一个 datagram "client" 的 demo
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

#define SERVERPORT "4950" // 用户所要连接的 port

int main(int argc, char *argv[])
{
  int sockfd;
  struct addrinfo hints, *servinfo, *p;
  int rv;
  int numbytes;

  if (argc != 3) {
    fprintf(stderr,"usage: talker hostname message\n");
    exit(1);
  }

  memset(&hints, 0, sizeof hints);
  hints.ai_family = AF_UNSPEC;
  hints.ai_socktype = SOCK_DGRAM;

  if ((rv = getaddrinfo(argv[1], SERVERPORT, &hints, &servinfo)) != 0) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rv));
    return 1;
  }

  // 用循环找出全部的结果，并产生一个 socket
  for(p = servinfo; p != NULL; p = p->ai_next) {
    if ((sockfd = socket(p->ai_family, p->ai_socktype,
        p->ai_protocol)) == -1) {
      perror("talker: socket");
      continue;
    }

    break;
  }

  if (p == NULL) {
    fprintf(stderr, "talker: failed to bind socket\n");
    return 2;
  }

  if ((numbytes = sendto(sockfd, argv[2], strlen(argv[2]), 0,
     p->ai_addr, p->ai_addrlen)) == -1) {

    perror("talker: sendto");
    exit(1);
  }

  freeaddrinfo(servinfo);
  printf("talker: sent %d bytes to %s\n", numbytes, argv[1]);
  close(sockfd);
  return 0;
}
```

\
全部就这些了！在某个机器上运行 listener，接着在另一台机器运行 talker。观察它们的沟通！这真的超有趣。\
\
这次你甚至不用运行 server！可以只运行 talker，而它只会很开心的将数据包丢到网路上，如果另一端没有人用 recvfrom()来接收的话，这些数据包就只是消失而已。\
\
要记得：使用 UDP datagram socket 传送的数据是不会使命必达的！\
\
我要再提之前提过无数次的小细节：connected datagram socket。我在这里要再讲一下，因为我们正在 datagram 这个章节。\
\
我们说 talker 调用 connect()并指定 listener 的地址。从这开始，talker 就只能从 connect()所指定的地址进行传送与接收。因此，你不用使用 sendto()与 recvfrom()，可以单纯使用 send()与 recv() 就好。\
\
\[22] http://beej.us/guide/bgnet/examples/listener.c\
\[23] http://beej.us/guide/bgnet/examples/talker.c

# 6.2. 简易的 Stream Client

Client 这家伙比 server 简单多了，client 所需要做的就是：连线到你在命令行所指定的主机 3490 port，接着，client 会收到 server 送回的字符串。\
\
Client 的代码 \[21]：\
\[21] http://beej.us/guide/bgnet/examples/client.c\


```c
/*
/*
** client.c -- 一个 stream socket client 的 demo
*/
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>

#define PORT "3490" // Client 所要连接的 port
#define MAXDATASIZE 100 // 我们一次可以收到的最大字节数量（number of bytes）

// 取得 IPv4 或 IPv6 的 sockaddr：
void *get_in_addr(struct sockaddr *sa)
{
　　if (sa->sa_family == AF_INET) {
　　　　return &(((struct sockaddr_in*)sa)->sin_addr);
　　}

　　return &(((struct sockaddr_in6*)sa)->sin6_addr);
}

int main(int argc, char *argv[])
{
　　int sockfd, numbytes;
　　char buf[MAXDATASIZE];
　　struct addrinfo hints, *servinfo, *p;
　　int rv;
　　char s[INET6_ADDRSTRLEN];

　　if (argc != 2) {
　　　　fprintf(stderr,"usage: client hostname\n");
　　　　exit(1);
　　}

　　memset(&hints, 0, sizeof hints);
　　hints.ai_family = AF_UNSPEC;
　　hints.ai_socktype = SOCK_STREAM;

　　if ((rv = getaddrinfo(argv[1], PORT, &hints, &servinfo)) != 0) {
　　　　fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rv));
　　　　return 1;
　　}

　　// 用循环取得全部的结果，并先连接到能成功连接的
　　for(p = servinfo; p != NULL; p = p->ai_next) {
　　　　if ((sockfd = socket(p->ai_family, p->ai_socktype,
　　　　　　p->ai_protocol)) == -1) {
　　　　　　perror("client: socket");
　　　　　　continue;
　　　　}

　　　　if (connect(sockfd, p->ai_addr, p->ai_addrlen) == -1) {
　　　　　　close(sockfd);
　　　　　　perror("client: connect");
　　　　　　continue;
　　　　}

　　　　break;
　　}

　　if (p == NULL) {
　　　　fprintf(stderr, "client: failed to connect\n");
　　　　return 2;
　　}

　　inet_ntop(p->ai_family, get_in_addr((struct sockaddr *)p->ai_addr), s, sizeof s);

　　printf("client: connecting to %s\n", s);

　　freeaddrinfo(servinfo); // 全部皆以这个 structure 完成

　　if ((numbytes = recv(sockfd, buf, MAXDATASIZE-1, 0)) == -1) {
　　　　perror("recv");
　　　　exit(1);
　　}

　　buf[numbytes] = '\0';
　　printf("client: received '%s'\n",buf);

　　close(sockfd);
　　return 0;
}
```

\
要注意的是，你如果没有在运行 client 以前先启动 server 的话，connect()会返回 ＂Connection refused＂，这很有帮助。

# 5.1. getaddrinfo()－准备开始！

这是个有很多选项（options）的工作马（workhorse）函数，但是却相当容易上手。它帮你设定之後需要的 struct。

谈点历史：它前身是你用来做 DNS 查询的 gethostbyname()。而当时你需要手动将资料写入 struct sockaddr\_in，并在你的调用中使用。

感谢老天，现在已经不用了。［如果你想要设计能通用於 IPv4 与 IPv6 的程序也不用！］在现代，你有 getaddrinfo() 函数，可以帮你做许多事情，包含 DNS 与 service name 查询，并填好你所需的 structs。

让我们来看看！

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo(const char *node, // 例如： "www.example.com" 或 IP
                const char *service, // 例如： "http" 或 port number
                const struct addrinfo *hints,
                struct addrinfo **res);
```

你给这个函数三个输入参数，结果它会返回一个指向链表的指针给你 － res。

_node_ 参数是要连接的主机名，或者一个 IP address（地址）。

下一个参数是 _service_，这可以是 port number，像是 ＂80＂，或者特定的服务名［可以在你 UNIX 系统上的 IANA Port List \[17] 或 /etc/services 文件中找到］，像是 ＂http＂ 或 ＂ftp＂ 或 ＂telnet＂ 或 ＂smtp＂ 诸如此类的。

\[17] [http://www.iana.org/assignments/port-numbers](http://www.google.com/url?q=http%3A%2F%2Fwww.iana.org%2Fassignments%2Fport-numbers\&sa=D\&sntz=1\&usg=AOvVaw0idcm6YHeyV091ES9nJYYS)

最後，_hints_ 参数指向一个你已经填好相关资料的 struct addrinfo。

这里是一个调用示例，如果你是一部 server（服务器），想要在你主机上的 IP address 及 port 3490 运行 listen。

要注意的是，这边实际上没有做任何的 listening 或网路设置；它只有设置我们之後要用的 structures 而已。

```c
int status;
struct addrinfo hints;c
struct addrinfo *servinfo; // 将指向结果

memset(&hints, 0, sizeof hints); // 确保 struct 为空
hints.ai_family = AF_UNSPEC;  // 不用管是 IPv4 或 IPv6
hints.ai_socktype = SOCK_STREAM; // TCP stream sockets
hints.ai_flags = AI_PASSIVE; // 幫我填好我的 IP 

if ((status = getaddrinfo(NULL, "3490", &hints, &servinfo)) != 0) {
  fprintf(stderr, "getaddrinfo error: %s\n", gai_strerror(status));
  exit(1);
}

// servinfo 目前指向一个或多个 struct addrinfos 的链表

// ... 做每件事情，一直到你不再需要 servinfo  ....

freeaddrinfo(servinfo); // 释放这个链表
```

注意一下，我将 _ai\_family_ 设置为 AF\_UNSPEC，这样代表我不用管我们用的是 IPv4 或 IPv6 address。如果你想要指定的话，你可以将它设置为 AF\_INET 或 AF\_INET6。

还有，你会在这里看到 AI\_PASSIVE flag；这个会告诉 getaddrinfo() 要将我本地端的地址（address of local host）指定给 socket structure。这样很好，因为你就不用写固定的地址了［或者你可以将特定的地址放在 getaddrinfo() 的第一个参数中，我现在写 NULL 的那个参数］。

然後我们进行调用，若有错误发生时［getaddrinfo 会返回非零的值］，如你所见，我们可以使用 gai\_strerror() 函数将错误打印出来。若每件事情都正常运作，那麽 _serinfo_ 就会指向一个 struct addrinfos 的链表，表中的每个成员都会包含一个我们之後会用到的某种 struct sockaddr。

最後，当我们终於使用 getaddrinfo() 配置的链表完成工作後，我们可以［也应该］要调用 freeaddrinfo() 将链表全部释放。

这边有一个调用示例，如果你是一个想要连接到特定 server 的 client，比如是：＂www.example.net＂的 port 3490。再次强调，这里并没有真的进行连接，它只是设置我们之後要用的 structure。

```c
int status;
struct addrinfo hints;
struct addrinfo *servinfo; // 将指向结果

memset(&hints, 0, sizeof hints); // 确保 struct 为空
hints.ai_family = AF_UNSPEC; // 不用管是 IPv4 或 IPv6
hints.ai_socktype = SOCK_STREAM; // TCP stream sockets

// 准备好连接
status = getaddrinfo("www.example.net", "3490", &hints, &servinfo);

// servinfo 现在指向有一个或多个 struct addrinfos 的链表

我一直说 serinfo 是一个链表，它有各种的地址资料。让我们写一个能快速 demo 的程序，来呈现这个资料。这个小程序 [18] 会打印出你在命令行中所指定的主机 IP address：
** showip.c -- 顯示命令列中所給的主機 IP address
*/
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
#include <arpa/inet.h>
#include <netinet/in.h>

int main(int argc, char *argv[])
{
  struct addrinfo hints, *res, *p;
  int status;
  char ipstr[INET6_ADDRSTRLEN];

  if (argc != 2) {
    fprintf(stderr,"usage: showip hostname\n");
    return 1;
  }

  memset(&hints, 0, sizeof hints);
  hints.ai_family = AF_UNSPEC; // AF_INET 或 AF_INET6 可以指定版本
  hints.ai_socktype = SOCK_STREAM;

  if ((status = getaddrinfo(argv[1], NULL, &hints, &res)) != 0) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(status));
    return 2;
  }

  printf("IP addresses for %s:\n\n", argv[1]);

  for(p = res;p != NULL; p = p->ai_next) {
    void *addr;
    char *ipver;

    // 取得本身地址的指针
     // 在 IPv4 与 IPv6 中的栏位不同：
    if (p->ai_family == AF_INET) { // IPv4
      struct sockaddr_in *ipv4 = (struct sockaddr_in *)p->ai_addr;
      addr = &(ipv4->sin_addr);
      ipver = "IPv4";
    } else { // IPv6
      struct sockaddr_in6 *ipv6 = (struct sockaddr_in6 *)p->ai_addr;
      addr = &(ipv6->sin6_addr);
      ipver = "IPv6";
    }

    // convert the IP to a string and print it:
    inet_ntop(p->ai_family, addr, ipstr, sizeof ipstr);
    printf(" %s: %s\n", ipver, ipstr);
  }

  freeaddrinfo(res); // 释放链表

  return 0;
}
```

如你所见，代码使用你在命令行输入的参数调用 getaddrinfo()，它填好 _res_ 所指的链表，并接着我们就能重复那行并打印出东西或做点类似的事。

［有点不好意思！我们在讨论 struct sockaddrs 它的型别差异是因 IP 版本而异之处有点鄙俗。我不确定是否有较优雅的方法。］

在下面运行示例！来看看大家喜欢看的运行画面：

```c
$ showip www.example.net
IP addresses for www.example.net:

  IPv4: 192.0.2.88

$ showip ipv6.example.com
IP addresses for ipv6.example.com:

  IPv4: 192.0.2.101
  IPv6: 2001:db8:8c00:22::171
```

现在已经在我们的掌控之下，我们会将 getaddrinfo() 返回的结果送给其它的 socket 函数，而且终於可以建立我们的网路连接了！

让我们继续看下去！

\[17] [http://www.iana.org/assignments/port-numbers](http://www.google.com/url?q=http%3A%2F%2Fwww.iana.org%2Fassignments%2Fport-numbers\&sa=D\&sntz=1\&usg=AOvVaw0idcm6YHeyV091ES9nJYYS)

\[18] [http://beej.us/guide/bgnet/examples/showip.c](http://www.google.com/url?q=http%3A%2F%2Fbeej.us%2Fguide%2Fbgnet%2Fexamples%2Fshowip.c\&sa=D\&sntz=1\&usg=AOvVaw00wgrc\_N35Sc3G1LKfRmrn)

\[19] [http://tools.ietf.org/html/rfc1413](http://www.google.com/url?q=http%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc1413\&sa=D\&sntz=1\&usg=AOvVaw1d4lF2B4J\_oat23DMIH7eD)

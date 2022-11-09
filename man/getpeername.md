# getpeername()

傳回遠端連線的位址資訊

## 函式原型

```c
#include <sys/socket.h>

int getpeername(int s, struct sockaddr *addr, socklen_t *len);
```

## 說明

一旦你接受了［accept()］遠端的連線，或者連線到［connect()］一個 server，你現在已經知道對方端點（peer）的資訊了，這個 peer 就是你所連線的電腦，透過一個 IP address 與一個 port 來識別，所以 ...

getpeername() 單純傳回一個 struct sockaddr\_in，裡面包含了與你連線的主機資訊。

為什麼它稱為 "name" 呢？

好，有許多不同的 socket 類型，不僅是我們這本書所使用的 Internet Sockets，所以 "name" 是一個好的通用術語，可以涵蓋全部的例子。以我們這個例子而言，peer 的 "name" 就是它的 IP address 與 port。

因為這個函式傳回 len 所指定的 address 大小，所以你必須預先將 addr 的大小存放在 len 裡面。

## 傳回值

成功時傳回零，錯誤時傳回 -1（並設定相對應的 errno）。

## 範例

```c
// 假設 s 是已連線的 socket

socklen_t len;
struct sockaddr_storage addr;
char ipstr[INET6_ADDRSTRLEN];
int port;

len = sizeof addr;
getpeername(s, (struct sockaddr*)&addr, &len);

// 處理 IPv4 與 IPv6：
if (addr.ss_family == AF_INET) {
    struct sockaddr_in *s = (struct sockaddr_in *)&addr;
    port = ntohs(s->sin_port);
    inet_ntop(AF_INET, &s->sin_addr, ipstr, sizeof ipstr);
} else { // AF_INET6
    struct sockaddr_in6 *s = (struct sockaddr_in6 *)&addr;
    port = ntohs(s->sin6_port);
    inet_ntop(AF_INET6, &s->sin6_addr, ipstr, sizeof ipstr);
}

printf("Peer IP address: %s\n", ipstr);
printf("Peer port      : %d\n", port);
```

## 參考

gethostname(), gethostbyname(), gethostbyaddr()

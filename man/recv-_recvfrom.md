# recv(), recvfrom()

接收 socket 的資料

## 函式原型

```c
#include <sys/types.h>
#include <sys/socket.h>

ssize_t recv(int s, void *buf, size_t len, int flags);
ssize_t recvfrom(int s, void *buf, size_t len, int flags,
                 struct sockaddr *from, socklen_t *fromlen);
```

## 說明

一旦你設定好 socket 以及建立了連線，你就可以開始讀取遠端送來的資料（TCP SOCK\_STREAM socket 使用 recv() 、UDP SOCK\_DGRAM socket 使用 recvfrom()）。

這兩個函式都會用到 socket descriptor、指向 buf 緩衝區的指標、緩衝區的大小（單位是 byte）、以及要控制函式運作方式的 flags set。

此外，recvfrom() 會需要一個 struct sockaddr\*，這可以讓你知道資料來自何處，並將 struct sockaddr 的長度填入 fromlen（你必須將 fromlen 初始化為 from 或 struct sockaddr 的大小）。

所以你可以傳遞那些奇妙的 flags 呢？這裡列出一部分，但是你系統所支援的功能你應該要自行查看你的 man 使用手冊。你可以用 OR 位元運算來同時使用這些 flags，如果你只是想要使用一般香草口味的 recv()，那只要將 flags 設定為 0 就可以了。

* MSG\_OOB    接收 Out of Band（頻外）資料，這個要接收的資料是有設定 MSG\_OOB flag 的 send() 所送來的。身為接收端，會有 SIGURG 訊號來告訴你有緊急資料（urgent data）需要處理。所以你可以在這個訊號的處理常式（handler）中搭配 MSG\_OOB flag 來呼叫 recv()。
* MSG\_PEEK    如果你想要假裝呼叫 recv()（裝個樣子），你可以搭配這個 flag 來呼叫 flag，這個可以讓你知道緩衝區中有哪些資料存在，這個 flag 可以讓 recv() 先預覽緩衝區中的資料而沒有真正的接收進來，下次沒有用 MSG\_PEEK 的 recv() 才會真的將這些資料收進來。
* MSG\_WAITALL    你可以用這個 flag 告訴 recv()，資料長度沒有達到你在 len 參數指定的長度以前別想返回，雖然在特殊的情況下還是事與願違，比如遇到訊號中斷了 recv() call、發生了一些錯誤、或者遠端關閉連線之類的情況。這就別跟它計較了。

當你呼叫 recv() 時，它會 block，直到讀到了一些資料，如果你希望它不會 block，那麼可以將 socket 設定為 non-blocking、或者是在呼叫 recv() 或 recvfrom() 以前，使用 select() 或 poll() 檢查 socket 上是否有資料。

## 傳回值

傳回實際接收的資料數（可能會比你在 len 參數中指定的還要少）、錯誤時傳回 -1（並設定相對應的 errno）。

若遠端關閉了連線，則 recv() 會傳回 0，這是一般用來判斷遠端是否關閉連線的方式。

## 範例

```c
// stream sockets 與 recv()

struct addrinfo hints, *res;
int sockfd;
char buf[512];
int byte_count;

// 取得 host 資訊，建立 socket，並建立連線
memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC;  // 使用 IPv4 或 IPv6，兩者皆可
hints.ai_socktype = SOCK_STREAM;
getaddrinfo("www.example.com", "3490", &hints, &res);
sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
connect(sockfd, res->ai_addr, res->ai_addrlen);

// 好的！現在我們已經連線，我們可以開始接收資料！
byte_count = recv(sockfd, buf, sizeof buf, 0);
printf("recv()'d %d bytes of data in buf\n", byte_count);
// datagram sockets 與 recvfrom()

struct addrinfo hints, *res;
int sockfd;
int byte_count;
socklen_t fromlen;
struct sockaddr_storage addr;
char buf[512];
char ipstr[INET6_ADDRSTRLEN];

// 取得 host 資訊、建立 socket、將 socket bind 到 port 4950
memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC;  // 使用 IPv4 或 IPv6，兩者皆可
hints.ai_socktype = SOCK_DGRAM;
hints.ai_flags = AI_PASSIVE;
getaddrinfo(NULL, "4950", &hints, &res);
sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
bind(sockfd, res->ai_addr, res->ai_addrlen);

// 不需要使用 accept()，直接使用 recvfrom()：

fromlen = sizeof addr;
byte_count = recvfrom(sockfd, buf, sizeof buf, 0, &addr, &fromlen);

printf("recv()'d %d bytes of data in buf\n", byte_count);
printf("from IP address %s\n",
    inet_ntop(addr.ss_family,
        addr.ss_family == AF_INET?
            ((struct sockadd_in *)&addr)->sin_addr:
            ((struct sockadd_in6 *)&addr)->sin6_addr,
        ipstr, sizeof ipstr);
```

## 參考

send(), sendto(), select(), poll(), Blocking

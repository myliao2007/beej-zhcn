# socket()

配置一個 socket descriptor

## 函式原型

```c
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```

## 說明

傳回一個新的 socket descriptor，這通常是寫 socket 程式用到的第一個呼叫，而你可以將這個 socket descriptor 用在後續的 listen()、bind()、accept() 或各種其它的函式。

一般的用法是從呼叫 getaddrinfo() 取得的參數值，如同下列範例所示，但是若你有意願，你也可以手動填寫。

* domain    domain 說明你有興趣的 socket 類型。相信我，這可以是很廣泛的東西，只是因為我們這邊談的是 socket，所以這裡只會用到 IPv4 的 PF\_INET 及 IPv6 的 PF\_INET6。
* type    還有，type 參數可以是各種東西，不過你大概只會用到可靠 TCP socket 的 SOCK\_STREAM（send(), recv()）或者不可靠快速 UDP socket 之 SOCK\_DGRAM（sendto(), recvfrom()）［另一個有趣的 socket type 是 SOCK\_RAW，這個可以用來手動建構封包表頭，它超酷的！］
* protocol    最後，protocol 參數指定在特定的 socket type 要用的 protocol（通訊協定），正如我所說的，比如：SOCK\_STREAM 使用 TCP。你很好命，在使用 SOCK\_STREAM 或 SOCK\_DGRAM 時，你可以直接將 protocol 設定為 0，這樣它會自動使用適合的 protocol，要不然你可以用 getprotobyname() 查詢適合的協定編號（protocol number）。

## 傳回值

傳回之後呼叫要用的新 socket descriptor，錯誤時傳回 -1（並設定相對應的 errno）

## 範例

```c
struct addrinfo hints, *res;
int sockfd;

// 首先，使用 getaddrinfo() 載入位址結構：

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC;     // AF_INET、AF_INET6 或 AF_UNSPEC
hints.ai_socktype = SOCK_STREAM; // SOCK_STREAM 或 SOCK_DGRAM

getaddrinfo("www.example.com", "3490", &hints, &res);

// 使用 getaddrinfo() 取得的資訊來建立 socket
sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
```

## 參考

accept(), bind(), getaddrinfo(), listen()

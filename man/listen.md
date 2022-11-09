# listen()

告訴 socket 要監聽（listen）請求的連線

## 函式原型

```c
#include <sys/socket.h>

int listen(int s, int backlog);
```

## 說明

你可以用你的 socket descriptor（利用 socket() system call 建立的），並將它提供給 listen() 用來聽取請求的連線。各位，這就是 server 與 clients 的差異。

參數 backlog 代表在 kernel 開始拒絕新連線以前，你可以有多少的連線在等待。所以當新的連線進入時，你應該盡快的用 accept() 來處理，讓 backlog 不會滿出來。試著將它設定成 10 之類，然後在高負載時，若你的 clients 開始出現＂拒絕連線＂時要試著將值提高。

在呼叫 listen() 以前，你的 server 應該要先呼叫 bind()，將 socket 綁定到一個 port number，這個 port number 跟 server 的 IP address 是用來讓 clients 連線的。

## 傳回值

成功時傳回零，或者錯誤時傳回 -1（並設定對應的 errno）。

## 範例

```c
struct addrinfo hints, *res;
int sockfd;

// 首先，使用 getaddrinfo() 載入位址結構：

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC;  // 使用 IPv4 或 IPv6，兩者皆可
hints.ai_socktype = SOCK_STREAM;
hints.ai_flags = AI_PASSIVE;     // 幫我填上我的 IP

getaddrinfo(NULL, "3490", &hints, &res);

// 建立 socket：

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

// 將它綁定到我們於 getaddrinfo() 中傳遞的 port：

bind(sockfd, res->ai_addr, res->ai_addrlen);

listen(sockfd, 10); // 將 s 設定為 server（監聽的）的 socket

// 接著這裡會有個迴圈用來呼叫 accept() 接受連線
```

## 參考

accept(), bind(), socket()

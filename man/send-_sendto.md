# send(), sendto()

透過 socket 送出資料

## 函式原型

```c
#include <sys/types.h>
#include <sys/socket.h>

ssize_t send(int s, const void *buf, size_t len, int flags);
ssize_t sendto(int s, const void *buf, size_t len,
               int flags, const struct sockaddr *to,
               socklen_t tolen);
```

## 說明

這些函式透過 socket 傳送資料，一般而言，send() 用在需連線的 TCP SOCK\_STREAM socket，而 sendto() 用在免連線的 UDP SOCK\_DGRAM socket。對於免連線 sockets，你每次都必須指定封包的目地，而這就是為什麼 sendto() 的最後一個參數需要定義封包的目地。

在 send() 與 sendto() 中，s 參數是 socket、buf 是指向要傳送資料的指標、len 是你想要傳送的資料量、而 flags 讓你可以指定更多資訊，決定要如何傳送資料。如果你只是想要送一般的資料，可以將 flags 設定為零。這裡有些常用的 flags，細節可以參考你電腦上的 send() man 使用手冊。

* MSG\_OOB    以 "out of band" 送出，TCP 支援這項功能，而這個方法可以用來告訴接收端，有高優先權的資料需要接收。接收端會收到 SIGURG 訊號，並且可以先從佇列中接收這筆資料。
* MSG\_DONTROUTE    不要透過 router 送出這筆資料，只將它留在區域網路內部。
* MSG\_DONTWAIT    若 send() 會因為外部流量堵塞而導致 block，讓它直接傳回 EAGAIN。這類似＂只針對這次的 send() 啟用 non-blocking＂，細節請參考 7.1 blocking 章節。
* MSG\_NOSIGNAL    若你 send() 的遠端主機不再接收資料了，一般你會收到 SIGPIPE 訊號，新增這個 flag 可以避免觸發這個訊號。

## 傳回值

傳回實際送出的資料數量，錯誤時傳回 -1（並設定相對應的 errno）。注意，實際上送出的資料量有可能會少於你要求傳送的數量！請參考 7.3 不完整傳送的後續處理 章節。

還有，若對方的 socket 已經關閉，則 process 呼叫 send() 會產生 SIGPIPE 訊號（除非在呼叫 send() 時有搭配 MSG\_NOSIGNAL flag，才不會收到）。

## 範例

```c
int spatula_count = 3490;
char *secret_message = "The Cheese is in The Toaster";

int stream_socket, dgram_socket;
struct sockaddr_in dest;
int temp;

// 先以 TCP stream sockets：

// 假設已建立 sockets 並連線
// stream_socket = socket(...
// connect(stream_socket, ...

// 轉換為 network byte order
temp = htonl(spatula_count);
// 一般方式傳送資料：
send(stream_socket, &temp, sizeof temp, 0);

// 頻外方式傳送秘密訊息
send(stream_socket, secret_message, strlen(secret_message)+1, MSG_OOB);

// 現在用 UDP datagram sockets：
//getaddrinfo(...
//dest = ...  // 假設 "dest" 承載目的端的位址
//dgram_socket = socket(...

// 以一般方式傳送秘密訊息：
sendto(dgram_socket, secret_message, strlen(secret_message)+1, 0, 
       (struct sockaddr*)&dest, sizeof dest);
```

## 參考

recv(), recvfrom()

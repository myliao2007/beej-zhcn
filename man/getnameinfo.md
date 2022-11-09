# getnameinfo()

由 struct sockaddr 提供的資訊查詢主機名稱（host name）及服務名稱（service name）資訊。

## 函式原型

```c
#include <sys/socket.h>
#include <netdb.h>

int getnameinfo(const struct sockaddr *sa, socklen_t salen,
                char *host, size_t hostlen,
                char *serv, size_t servlen, int flags);
```

## 說明

這個函式是 getaddrinfo() 的對比，將已填好的 struct sockaddr 代入這個函式，就可以查詢 hostname 與 service。它取代了舊有的 gethostbyaddr() 與 getservbyport() 函式。

你必須將一個指向 struct sockaddr 的指標（實際上這個可能是你轉型過的 struct sockaddr\_in 或 struct sockaddr\_in6 ）傳遞給 sa 參數，而這個 struct 的長度是 salen。

結果會將查到的 host name 與 service name 寫入 host 與 serv 參數所指的區域空間裡，當然，你必須用 hostlen 與 servlen 來指定這些緩衝區的最大長度。

最後，有幾個你能傳遞的 flags（旗標），但是這裡有兩個好物。NI\_NOFQDN 會讓 host 只包含 host name，而不是全部的 domain name（網域名稱）；如果在 DNS 查詢時無法找到 name 的時候，NI\_NAMEREQD 會讓函式發生失敗（如果你沒有指定這個 flag，而又無法找到 name 時，那 getnameinfo() 就會改為將一個字串版本的 IP address 放在 host 裡面）。

同樣地，完整的內容請參考你電腦上的 man 使用手冊。

## 傳回值

成功時傳回零，或錯誤時傳回非零。若傳回值是非零，則可以將傳回值傳遞給 gai\_strerror()，以取得我們可讀的字串，細節請參考 getaddrinfo 。

## 範例

```c
struct sockaddr_in6 sa; // 如果你想，也可以是 IPv4 
char host[1024];
char service[20];

// 假設 sa 充滿了主機與 port 相關的資訊 ...

getnameinfo(&sa, sizeof sa, host, sizeof host, service, sizeof service, 0);

printf(" host: %s\n", host); // 例如: "www.example.com"
printf("service: %s\n", service); // 例如: "http"
```

## 參考

getaddrinfo(), gethostbyaddr()

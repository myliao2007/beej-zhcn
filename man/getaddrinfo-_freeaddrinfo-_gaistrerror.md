# getaddrinfo(), freeaddrinfo(), gai\_strerror()

取得主機名稱或服務的資訊，並將結果載入 struct sockaddr。

## 函式原型

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo(const char *nodename, const char *servname,
                const struct addrinfo *hints, struct addrinfo **res);

void freeaddrinfo(struct addrinfo *ai);

const char *gai_strerror(int ecode);

struct addrinfo {
  int ai_flags; // AI_PASSIVE, AI_CANONNAME, ...
  int ai_family; // AF_xxx
  int ai_socktype; // SOCK_xxx
  int ai_protocol; // 0 (auto) or IPPROTO_TCP, IPPROTO_UDP

  socklen_t ai_addrlen; // ai_addr 的長度
  char *ai_canonname; // canonical name for nodename
  struct sockaddr *ai_addr; // 二進制格式位址
  struct addrinfo *ai_next; // linked list 中的下個資料結構
};
```

## 說明

getaddrinfo() 是很優秀的函式，可以傳回特別的主機名稱資訊（比如它的 IP address）以及為你載入 struct sockaddr，要注意一些細節（像是 IPv4 或 IPv6）。它會取代舊有的 gethostbyname() 及 getservbyname() 函式。下列的說明包含許多資訊，可能看起來有點難，但實際上卻很簡單。先看看範例是值得的。

你感興趣的 host name 在 node name 參數中，address 可以是個 host name，像 "www.example.com" 或者是 IPv4 或 IPv6 位址（以字串傳遞）。如果你使用 AI\_PASSIVE flag，那這個參數也可以是 NULL（參考下面）。

servname 參數基本上就是 port number，它可以是個 port number（以字串傳遞，如 "80"），或者是個 service name，像是 "http"、"tftp"、"smtp"、或 "pop" 等。常見的 service name 可以在 IANA Port List \[42] 或你的 /etc/services 中找到。

最後的 input 參數是 hints，這就是你要定義 getaddinfo() 函式要做什麼事的地方，使用以前先用 memset() 將整個資料結構清為零，在用它之前，我們先看一下需要設定的欄位。

ai\_flags 可以設定成各種東西，但是這邊有件重要的事情。（可以用 OR 位元運算［｜］指定多個 flags）完整的 flags 清單請參考你的 man 使用手冊。

AI\_CANONNAME 會讓 ai\_canonname 填上主機的 canonical (real) name，AI\_PASSIVE 讓 IP address 填上 INADDR\_ANY （IPv4）或 in6addr\_any（IPv6）；這讓之後在呼叫 bind() 時，可以自動用目前 host 的 address 來填上 struct sockaddr 的 IP address。這在設定 server 且你不想要寫死 address 時非常好用。

如果你使用 AI\_PASSIVE flag，那麼你可以在 nodename 中傳遞 NULL（因為 bind() 在之後會幫你填上）

\[42] [http://www.iana.org/assignments/port-numbers](http://www.iana.org/assignments/port-numbers)

繼續談輸入的參數，你應該會想要將 ai\_family 設定為 AF\_UNSPEC，這樣可以讓 getaddrinfo() 知道 IPv4 與 IPv6 addresses 都需要查詢。你也能自己以 AF\_INET 或 AF\_INET6 自訂要使用 IPv4 或 IPv6。

再來，socktype 欄位應該要設定為 SOCK\_STREAM 或 SOCK\_DGRAM，取決與你需要哪種類型的 socket。

最後，你可以將 ai\_protocol 保留為 0，可以自動選擇你的 protocol type。

在你取得全部的東西之後，你終於可以呼叫 getaddrinfo() 了！

當然，這個地方開始有趣了，res 會指向一個 struct addrinfo 的鏈結串列，而你可以透過這個串列取得全部的 addresses（符合你在 hints 中指定的 address 類型）。

現在可能會取得一些因為某些理由無法正常運作的 addresses，所以 Linux man 使用手冊提供的方法是不斷用迴圈讀取串列，然後試著呼叫 socket()、connect()（如果以 AI\_PASSIVE flag 設定 server，則是 bind()），直到成功為止。

最後，當你處理完鏈結串列之後，你需要呼叫 freeaddrinfo() 來釋放記憶體（否則會發生 memory leak，這會讓有些人不安。）

## 傳回值

成功時傳回零，或錯誤時傳回非零值。若傳回非零值，你可以代入 gai\_strerror() 函式取得一個文字版的錯誤訊息。

## 範例

```c
// client 連線到 server 的程式碼
// 透過 stream socket 連線到 www.example.com 的 port 80 (http)
// 不是 IPv4 就是 IPv6

int sockfd;
struct addrinfo hints, *servinfo, *p;
int rv;

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC; // 設定 AF_INET6 表示強迫使用 IPv6
hints.ai_socktype = SOCK_STREAM;

if ((rv = getaddrinfo("www.example.com", "http", &hints, &servinfo)) != 0) {
  fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rv));
  exit(1);
}

// 不斷執行迴圈，直到我們可以連線成功
for(p = servinfo; p != NULL; p = p->ai_next) {
  if ((sockfd = socket(p->ai_family, p->ai_socktype,　p->ai_protocol)) == -1) {
      perror("socket");
      continue;
  }

  if (connect(sockfd, p->ai_addr, p->ai_addrlen) == -1) {
    close(sockfd);
    perror("connect");
    continue; 
  }

  break; // if we get here, we must have connected successfully
}

if (p == NULL) {
  // 迴圈已經執行到 list 的最後，都無法連線
  fprintf(stderr, "failed to connect\n");
  exit(2);
}

freeaddrinfo(servinfo); // 釋放 servinfo 記憶體空間

// code for a server waiting for connections
// namely a stream socket on port 3490, on this host's IP
// either IPv4 or IPv6.

int sockfd;
struct addrinfo hints, *servinfo, *p;
int rv;

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC; // 使用 AF_INET6 表示一定要用 IPv6
hints.ai_socktype = SOCK_STREAM;
hints.ai_flags = AI_PASSIVE; // 使用我的 IP address

if ((rv = getaddrinfo(NULL, "3490", &hints, &servinfo)) != 0) {
  fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rv));
  exit(1);
}

// 不斷執行迴圈，直到我們可以成功綁定
for(p = servinfo; p != NULL; p = p->ai_next) {
  if ((sockfd = socket(p->ai_family, p->ai_socktype,
      p->ai_protocol)) == -1) {
    perror("socket");
    continue;
  }

  if (bind(sockfd, p->ai_addr, p->ai_addrlen) == -1) {
    close(sockfd);
    perror("bind");
    continue;
  }

  break; // 若執行到這行，表示我們一定已經成功連線
}

if (p == NULL) {
  // 整個迴圈執行完畢，到了 linked-list 結尾都無法成功綁定（bind）
  fprintf(stderr, "failed to bind socket\n");
  exit(2);
}

freeaddrinfo(servinfo); // 使用完畢時釋放這個資料結構的空間
```

## 參考

gethostbyname(), getnameinfo()

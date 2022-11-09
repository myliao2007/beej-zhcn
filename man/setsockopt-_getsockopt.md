# setsockopt(), getsockopt()

為 socket 設定多個選項

## 函式原型

```c
#include <sys/types.h>
#include <sys/socket.h>

int getsockopt(int s, int level, int optname, void *optval,
               socklen_t *optlen);
int setsockopt(int s, int level, int optname, const void *optval,
               socklen_t optlen);
```

## 說明

這裡會講些基礎的 Sockets 設定。

很明顯的，這些函式是用來取得 socket 的資訊或設定 socket。在 Linux 系統上，全部的 socket 資訊都在 man 使用手冊第 7 節（輸入 "man 7 socket" 就可以查看這些資訊）。

至於參數，s 是想要設定的 socket， level 應該要設定為 SOL\_SOCKET，然後你可以將 optname 設定為你有興趣的名稱，再來，完整的參數請參考你的 man 使用手冊，這邊只會介紹比較好玩的那一部分：

* SO\_BINDTODEVICE    將這個 socket bind 到類似 eth0 名稱的 symbolic device，而不是用 bind() 將 socket bind  到一個 IP address，在 Unix 底下輸入 ifconfig 可以查看網路裝置名稱。
* SO\_REUSEADDR    除非已經有 listening socket 綁定到這個 port，不然可以允許其它的 sockets bind() 這個 port。這樣可以讓你在 server 當機之後，重新啟動 server 時不會遇到 "Address already in use（位址已經在使用中）" 這個錯誤訊息。
*   SO\_BROADCAST    讓 UDP datagram (SOCK\_DGRAM) sockets 可以傳送或接收封包到廣播位址（broadcast address）！

    至於 optval 參數，通常是個指向一個 int 型別變數的指標，若是 booleans，零為 false（假），而非零為 true（真）。除非在你的系統不一樣，不然這是不爭的事實。如果沒有參數要傳遞，可以將 optval 設定為 NULL。

最後一個 optlen 參數是你用 getsockopt() 取得的，而你必須在 setsockopt() 時指定它，大小可能是 sizeof(int)。

警告：在一些系統上（特別是 Sun 與 Windows），所要設定的選項會是個字元，而不是 int，比如：是一個 '1' 的字元值，而不是 1 的整數值。還有，細節請參考你 man 使用手冊，用 "man setsockopt" 以及 "man 7 socket"！

## 傳回值

成功時傳回零，或者錯誤時傳回 -1（並設定相對應的 errno）。

## 範例

```c
int optval;
int optlen;
char *optval2;

// 表示對 socket 設定 SO_REUSEADDR(1) 是 true：
optval = 1;
setsockopt(s1, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof optval);

// 將 socket 綁定（bind）到一個裝置名稱（可能不是每個系統都有效果）：
optval2 = "eth1"; // 4 bytes long, so 4, below:
setsockopt(s2, SOL_SOCKET, SO_BINDTODEVICE, optval2, 4);

// 檢測是否有設定 SO_BROADCAST flag：
getsockopt(s3, SOL_SOCKET, SO_BROADCAST, &optval, &optlen);
if (optval != 0) {
    print("SO_BROADCAST enabled on s3!\n");
}
```

## 參考

fcntl()

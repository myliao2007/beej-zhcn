# close()

關閉 socket descriptor

## 函式原型

```c
#include <unistd.h>

int close(int s);
```

## 說明

無論你設計了多棒的機制，在你用完 socket 且不想要 send() 或 recv() 時，也就是不會再對這個 socket 做任何事情時，你可以將它 close()，而它就會被釋放，而且永遠不會再用到。

遠端可以用兩種方法判斷對方是否已經關閉 socket，一種是：如遠端呼叫的 recv() 傳回 0；另一種是：若遠端呼叫 send() 時收到一個 SIGPIPE 的訊號（signal），且 send() 傳回 -1 並將 errno 設定為 EPIPE。

Windows 系統的使用者：你要用的函式是 closesocket() 而不是 close()，如果你試著使用 close() 關閉 socket descriptor，那麼 Windows 系統有可能會震怒 ... 它如果敢生氣你就別再愛它了。

## 傳回值

成功時傳回零，或者錯誤時傳回 -1（並設定相對應的 errno）。

## 範例

```c
s = socket(PF_INET, SOCK_DGRAM, 0);
.
.
.
// a whole lotta stuff...*BRRRONNNN!*
.
.
.
close(s); // 沒有多少，真的。
```

## 參考

socket(), shutdown()

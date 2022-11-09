# fcntl()

控制 socket descriptors

## 函式原型

```c
#include <sys/unistd.h>
#include <sys/fcntl.h>

int fcntl(int s, int cmd, long arg);
```

## 說明

這個函式通常用來進行檔案鎖定與其它檔案導向的用途，但是它也有幾個與 socket 相關的功能，你之後可能會用的到。

參數 s 是 你想要控制的 socket descriptor，而 cmd 應該要設定為 F\_SETFL，至於 arg 可以是下列任一個指令（如我所述，參數比我這邊介紹的還要多，但是我只會探討與 socket 相關的部分）。

* O\_NONBLOCK    將 socket 設定為 non-blocking，細節請參考 blocking 章節。
* O\_ASYNC     將 socket 設定為非同步 I/O（asynchronous I/O），當 socket 上的資料就緒可收時，會觸發 SIGIO 訊號。這幾乎不會遇到，並且在本書的討論範圍之外，而我認為這只會在某些系統上出現。

## 傳回值

成功時傳回零，錯誤時傳回 -1（並設定相對應的 errno）

fcntl() system call 依據不同的用法會有不同的傳回值，但是我在這裡不會全部涵蓋，因為有些與 socket 無關，細節請參考 fcntl() 的 man 手冊。

## 範例

```c
int s = socket(PF_INET, SOCK_STREAM, 0);

fcntl(s, F_SETFL, O_NONBLOCK);  // 設定為非阻塞（non-blocking）
fcntl(s, F_SETFL, O_ASYNC);     // 設定為非同步 I/O
```

## 參考

Blocking, send()

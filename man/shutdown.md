# shutdown()

停止對 socket 繼續傳送與接收

## 函式原型

```c
#include <sys/socket.h>

int shutdown(int s, int how);
```

## 說明

如果我不需要再對 socket 進行傳送［send()］，但是我仍然想要接收［recv()］socket 的資料，反之亦然，那我該怎麼做呢？

當你使用 close() 關閉 socket descriptor 時，它會將 socket 的傳送與接收兩端都關閉，並且釋放 socket descriptor。若你只想要關閉其中一端，你就可以使用 shutdown() call。

在這些參數中，s 顯然是你想要進行動作的 socket，而要進行什麼樣的動作，則要由 how 參數指定。要如何使用 SHUT\_RD 來關閉接收，SHUT\_WR 以關閉傳送，或者 SHUT\_RDWR 將收送功能都關閉。

要注意 shutdown() 並沒有釋放 socket descriptor，所以即使 socket 已經整個 shutdown 了，最終仍然得透過 close() 關閉 socket。

這是蠻少用的 system call。

## 傳回值

成功時傳回零，或者錯誤時傳回 -1（並設定相對應的 errno）。

## 範例

```c
int s = socket(PF_INET, SOCK_STREAM, 0);

// ...在這裡進行一些 send() 與工作...

// 而現在我們已經完成該做的事情，不再需要傳送了：
shutdown(s, SHUT_WR);
```

## 參考

close()

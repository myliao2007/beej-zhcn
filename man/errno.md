# errno

儲存上次 system call 的錯誤碼

## 函式原型

```c
#include <errno.h>

int errno;
```

## 說明

這個變數用來儲存 system call 的錯誤資訊，比如像：socket() 與 listen() 錯誤時會傳回 -1，並設定 errno 的值，讓你可以知道發生了什麼樣的錯誤。

標頭檔 errno.h 列出了許多錯誤的常數符號，比如：EADDRINUSE、EPIPE、ECONNREFUSED 等。你本機上的 man 手冊會告訴你錯誤時所傳回的錯誤碼有哪些，而你可以在執行期時利用這些錯誤碼來分別處理不同的錯誤。

或者，更普遍的是，你可以呼叫 perror() 或 strerror() 取得人們方便閱讀的錯誤訊息。

對於喜歡用多執行緒（multithreading）的人，有件事需要注意：就是多數系統上的 errno 會定義為 threadsafe 的方式。（也就是說，它並不是一個全域變數，但是在單執行緒的環境中，它的行為跟全域變數一樣）。

## 傳回值

變數值記錄最新的錯誤，如果上次的動作是成功的，這個值就會是 "success"。

## 範例

```c
s = socket(PF_INET, SOCK_STREAM, 0);
if (s == -1) {
    perror("socket"); // 或者使用 strerror()
}

tryagain:
if (select(n, &readfds, NULL, NULL) == -1) {
    // 發生錯誤！！

    // 如果我們只有被中斷，則只需重新啟動 select() call:
    if (errno == EINTR) goto tryagain;  // AAAA!  goto!!!

    // 否則，就是個嚴重的錯誤：
    perror("select");
    exit(1);
}
```

## 參考

perror(), strerror()

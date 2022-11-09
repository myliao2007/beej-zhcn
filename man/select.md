# select()

檢查 sockets descriptors 是否就緒可讀或可寫

## 函式原型

```c
#include <sys/select.h>

int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds,
           struct timeval *timeout);

FD_SET(int fd, fd_set *set);
FD_CLR(int fd, fd_set *set);
FD_ISSET(int fd, fd_set *set);
FD_ZERO(fd_set *set);
```

## 說明

select() 函式讓你可以同步檢查多個 sockets，檢查它們是否有資料需要接收，或者是否你可以送出資料而不會發生 blocking，或者是否有例外發生。

你可以使用如上述的 FD\_SET() macros（巨集）來調整 socket descriptors set。

如果你想要知道 set 上的哪些 sockets 有資料可以接收（就緒可讀），則將這個 set 放在 readfds 參數；

如果你想要知道 set 上的哪些 sockets 有資料可以傳送（就緒可寫），則將這個 set 放在 writefds 參數；

若你想知道哪些 sockets 有例外（錯誤）發生，則可以將 set 擺在 exceptfds 參數。

沒興趣知道的事件可以將 select() 相對應的參數設定為 NULL。

在 select() 返回之後，這些 sets 的值會改變，用以標示哪些 sockets 是就緒可讀、可寫，及有例外發生等事件。

第一個參數 n 的值是最大的那個 socket descriptor 數值在加上 1。

最後，struct timeval 可以設定 timeout 的時間，用來告訴 select() 要等待多久的時間。 select() 會在有任何事件發生或 timeout 之後返回。struct timeval 有兩個欄位：tv\_sec 設定秒（second）、tv\_usec 設定微秒（microsecond）［一秒鐘有 1,000,000 微秒］。

用到的 macros 功能如下：

* FD\_SET(int fd, fd\_set \*set); 將 fd 新增至 set。
* FD\_CLR(int fd, fd\_set \*set); 從 set 中移除 fd。
* FD\_ISSET(int fd, fd\_set \*set); 若 fd 在 set 中則傳回 true。
* FD\_ZERO(fd\_set \*set); 將 set 清為零。

## 傳回值

成功時傳回 set 中的 descriptors 數量，若發生 timeout 時傳回 0，錯誤時傳回 -1（並設定相對應的 errno），還有，sets 會被改過，用以表示哪幾個 sockets 是已經就緒的。

## 範例

```c
int s1, s2, n;
fd_set readfds;
struct timeval tv;
char buf1[256], buf2[256];

// 假裝我們此時已經都連線到 server 了
//s1 = socket(...);
//s2 = socket(...);
//connect(s1, ...)...
//connect(s2, ...)...

// 事先清除 set
FD_ZERO(&readfds);

// 將我們的 descriptors 新增到 set
FD_SET(s1, &readfds);
FD_SET(s2, &readfds);

// 因為 s2 是後來才取得的，所以它的數值會＂比較大＂，所以我們用它作為 select() 中的 n 參數

n = s2 + 1;

// 在 timeout 以前會一直等待，看是否已經有資料可以接收的 socket（timeout 時間是 10.5 秒）
tv.tv_sec = 10;
tv.tv_usec = 500000;
rv = select(n, &readfds, NULL, NULL, &tv);

if (rv == -1) {
    perror("select"); // select() 發生錯誤
} else if (rv == 0) {
    printf("Timeout occurred!  No data after 10.5 seconds.\n");
} else {
    // 至少一個 descriptor(s) 有資料
    if (FD_ISSET(s1, &readfds)) {
        recv(s1, buf1, sizeof buf1, 0);
    }
    if (FD_ISSET(s2, &readfds)) {
        recv(s2, buf2, sizeof buf2, 0);
    }
}
```

## 參考

poll()

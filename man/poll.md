# poll()

同時對多個 socket 進行事件測試

## 函式原型

```c
#include <sys/poll.h>

int poll(struct pollfd *ufds, unsigned int nfds, int timeout);
```

## 說明

這個函式非常類似 select()，它們兩者都監看整組 file descriptor 的事件，比如進入的資料是就緒可收［ready to recv()］、socket 是就緒可送［ready to send()］ 資料、out-of-band 資料是就緒可收［ready to recv()］、錯誤等。

基本的想法是，你透過 ufds 傳遞一個有 nfds 個 struct pollfd 的陣列，並以毫秒（millisecond）為單位設定 timeout 時間（一秒有 1,000 毫秒），若陣列中的每個 socket descriptor 都沒有事件發生時，則隨著 timeout 時間耗盡，poll() 就會返回。如果你不需要 timeout，要讓 poll() 一直等待，那麼可以將 timeout 設定為負值。

陣列中的每個 struct pollfd 元素（element）表示一個 socket descriptor，且包含了下列的欄位：

```c
struct pollfd {
    int fd;         // the socket descriptor
    short events;   // bitmap of events we're interested in
    short revents;  // when poll() returns, bitmap of events that occurred
};
```

在呼叫 poll() 以前，將 fd 帶入 socket descriptor 的值（若你將 fd 設定為負值，這個 struct pollfd 就會被忽略掉，並且將它的 revents 欄位設定為零），接著用 OR 位元運算下列的 macros（巨集）以建構 events 欄位：

* POLLIN          當這個 socket 上的資料已經就緒可以收［recv()］時，通知（alert）我。
* POLLOUT         當我可以送［send()］資料到這個 socket 而不會 blocking 時，通知我。
*   POLLPRI         當 out-of-band data 就緒可收時，通知我。

    一旦 poll() call 返回時，會透過對上述欄位進行 OR 位元運算來建構 revents 欄位，以告訴你哪些 descriptor 目前已經有事件發生，此外，其它的欄位可能會保持原值：
* POLLERR         在這個 socket 上已經發生錯誤
* POLLHUP         遠端連線已經斷線。
* POLLNVAL        fd socket descriptor 有點問題，或許是因為沒有初始化？

## 傳回值

傳回 ufds 陣列中，有事件發生的元素（element）數量；如果發生 timeout 時，這個值會是零。錯誤時會傳回 -1（並設定相對應的 errno）。

## 範例

```c
int s1, s2;
int rv;
char buf1[256], buf2[256];
struct pollfd ufds[2];

s1 = socket(PF_INET, SOCK_STREAM, 0);
s2 = socket(PF_INET, SOCK_STREAM, 0);

// 假設此時我們已經都連線到 server 了
//connect(s1, ...)...
//connect(s2, ...)...

// 設定 file descriptors 陣列
//
// 在此例中，我們想要知道何時有就緒的一般資料或 out-of-band（頻外）資料要收
// 

ufds[0].fd = s1;
ufds[0].events = POLLIN | POLLPRI; // 要檢查是一般資料或 out-of-band 資料

ufds[1] = s2;
ufds[1].events = POLLIN; // 只檢查一般的資料

// 等待 sockets 上的事件，timeout 時間是 3.5 秒
rv = poll(ufds, 2, 3500);

if (rv == -1) {
    perror("poll"); // poll() 時發生錯誤
} else if (rv == 0) {
    printf("Timeout occurred!  No data after 3.5 seconds.\n");
} else {
    // 檢查 s1 上的事件：
    if (ufds[0].revents & POLLIN) {
        recv(s1, buf1, sizeof buf1, 0); // 接收一般的資料
    }
    if (ufds[0].revents & POLLPRI) {
        recv(s1, buf1, sizeof buf1, MSG_OOB); // out-of-band 資料
    }

    // 檢查 s2 上的事件：
    if (ufds[1].revents & POLLIN) {
        recv(s2, buf2, sizeof buf2, 0);
    }
}
```

## 參考

select()

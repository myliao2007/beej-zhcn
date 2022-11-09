# gethostname()

傳回系統的主機名稱（host name）

## 函式原型

```c
#include <sys/unistd.h>
int gethostname(char *name, size_t len);
```

## 說明

你的系統有一個名字，大家都有的，這比較偏向系統層面，而不是我們正在談論的網路層面，只是它仍有其用途。

例如：你可以取得你的主機名稱，接著呼叫 gethostbyname() 找出你電腦的 IP address。

name 參數應該指向一個存有主機名稱的緩衝區，而 len 是該緩衝區的大小，以 byte 為單位。gethostname() 不會覆寫緩衝區的結尾（可能會傳回錯誤，或者只是單純停止寫入），而且如果緩衝區有足夠的空間，它還會保留字串的 NUL-結尾。

## 傳回值

成功時傳回零，或者錯誤時傳回 -1（並設定相對應的 errno）。

## 範例

```c
char hostname[128];

gethostname(hostname, sizeof hostname);
printf("My hostname: %s\n", hostname);
```

## 參考

gethostbyname()

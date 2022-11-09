# perror(), strerror()

輸出易讀的錯誤訊息字串

## 函式原型

```c
#include <stdio.h>
#include <string.h>   // strerror() 需要

void perror(const char *s);
char *strerror(int errnum);
```

## 說明

因為大多的函式錯誤都是傳回 -1，並設定 errno 變數值為某個數，如果你還可以很簡單地印出適合你的格式，這樣就更完美了。

感謝上天，perror() 可以。如果你想要在錯誤訊息前加上一些敘述，你可以將訊息透過 s 參數傳遞（或你可以將 s 參數設定為 NULL，這樣就不會印出額外的訊息）。

簡單說，這個函式透過 errno 值，比如：ECONNRESET，然後印出比較有意義的訊息，像是 "Connection reset by peer"（對方重置連線）。

函式 strerror() 類似 perror()，不同之處是 strerror() 會依據所給的值，傳回一個指向錯誤訊息的指標（通常你會傳 errno 變數）。

## 傳回值

strerror() 傳回指向錯誤訊息字串的指標。

## 範例

```c
int s;

s = socket(PF_INET, SOCK_STREAM, 0);

if (s == -1) { // some error has occurred
    // 印出 "socket error： " + 錯誤訊息：
    perror("socket error");
}

// 類似的
if (listen(s, 10) == -1) {
    // 這會印出 "一個錯誤： " + errno 的錯誤訊息：
    printf("an error: %s\n", strerror(errno));
}
```

## 參考

errno

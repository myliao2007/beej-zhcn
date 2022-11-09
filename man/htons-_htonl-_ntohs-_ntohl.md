# htons(), htonl(), ntohs(), ntohl()

將多位元組整數型別（multi-byte integer types）由 host byte order 轉換為 network byte order

## 函式原型

```c
#include <netinet/in.h>

uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

## 說明

這真的只是來亂的，不同的電腦在多位元組整數［比如：任何比字元長的整數］都用不同的 byte orderings（位元組順序）。結果變成，如果你從 Intel 主機用 send() 送出一個兩 bytes 的 short int 給 Mac 主機［我是指在它們都成為 Intel 架構以前的事了］，在一台電腦上認為是 1 的數值，到了另一台電腦上卻可能是 256，反之亦然。

避免這個問題的方法是請大家拋開這些差異，並同意 Motorola 與 IBM 才是正道，而 Intel 的方法是旁門左道。所以我們應該在送出資料以前，先將我們的 byte ordering 轉換為 "big-endian"，因為 Intel 是 "little-endian" 的電腦。我們將偏好的 byte order 稱為 "Network Byte Order" 是很正確的，所以這些函式會將你電腦的 byte order 轉換為 network byte order，然後對方再轉回來。

（這表示在 Intel 電腦上，這些函式旋轉了全部的 bytes 順序，而在 PowerPC 上，它們什麼都不用做，因為 PowerPC 就是以 Network Byte Order 的方式儲存資料。但無論如何你的程式碼應該每次都要用這些函式，因為有人可能會想在 Intel 電腦上執行你的程式，而你的程式還是得正常運作。）

注意，相關的型別是 32-bit（4 byte，可能是 int）與 16-bit（2 byte，很有可能是 short）的數值。64-bit 的電腦可能有一個 htonll() 來轉換 64-bit 的整數，但是我還沒看過，你可以自己寫一個。

不管怎樣，這些函式運作的方式就是，你要先決定是從（你電腦的）host byte order 轉換，還是從 network byte order 轉換。如果是從 "host"，那你要呼叫的函式開頭就是 "h"，否則就是 network 的 "n"。中間的函式名稱永遠都是 "to"，因為你是從某一種順序轉換到（"to"）另一種，而倒數第二個字母表示你要轉換的目的格式。最後的字母就是資料的大小，"s" 表示 short、"l" 表示 long，如下所示：

```c
htons()   host to network short
htonl()   host to network long
ntohs()   network to host short
ntohl()   network to host long
```

## 傳回值

每個函式傳回轉換過的值

## 範例

```
uint32_t some_long = 10;
uint16_t some_short = 20;

uint32_t network_byte_order;

// 轉換與傳送
network_byte_order = htonl(some_long);
send(s, &network_byte_order, sizeof(uint32_t), 0);

some_short == ntohs(htons(some_short)); // 這行判斷式為真
```

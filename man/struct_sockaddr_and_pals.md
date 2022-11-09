# struct sockaddr and pals

處理 internet addresses 的資料結構

## 函式原型

```c
#include <netinet/in.h>

// 全部指向 socket address structures 的指標，通常會在各種函式或 system calls 使用以前，
// 轉型為對相對應型別的指標：

struct sockaddr {
    unsigned short    sa_family;    // address family, AF_xxx
    char              sa_data[14];  // 14 bytes 的 protocol address
};

// IPv4 AF_INET sockets:

struct sockaddr_in {
    short            sin_family;   // 例如：AF_INET, AF_INET6
    unsigned short   sin_port;     // 例如：htons(3490)
    struct in_addr   sin_addr;     // 參考下列的 struct in_addr
    char             sin_zero[8];  // 若你想要的話，將這個設定為零
};

struct in_addr {
    unsigned long s_addr;          // 用 inet_pton() 載入
};


// IPv6 AF_INET6 sockets:

struct sockaddr_in6 {
    u_int16_t       sin6_family;   // address family, AF_INET6
    u_int16_t       sin6_port;     // port number, Network Byte Order
    u_int32_t       sin6_flowinfo; // IPv6 flow 資訊
    struct in6_addr sin6_addr;     // IPv6 address
    u_int32_t       sin6_scope_id; // Scope ID
};

struct in6_addr {
    unsigned char   s6_addr[16];   // 用 inet_pton() 載入
};


// 承載 socket address 的 structure，要足以承載
// struct sockaddr_in 或 struct sockaddr_in6 data：

struct sockaddr_storage {
    sa_family_t  ss_family;     // address family

    // 這個全部都是填充的內容，依據實作而定，請忽略它：
    char      __ss_pad1[_SS_PAD1SIZE];
    int64_t   __ss_align;
    char      __ss_pad2[_SS_PAD2SIZE];
};
```

## 說明

對於 internet address 全部的 syscalls 及函式都有基本的資料結構，通常你會用 getaddinfo() 填好這些資料結構，然後當你需要時，就可以讀取它們。

印象中，struct sockaddr\_in 與 struct sockaddr\_in6 會共用相同的 struct sockaddr 起始資料結構，而你能自由地將一種型別轉型為另一種型別，除非你到了宇宙的盡頭，不然是不會有任何問題的。

宇宙的盡頭這件事只是開個玩笑 ... 若當你將 struct sockaddr\_in _轉型為 struct sockaddr_ 時已經到了宇宙的盡頭，那我保證這必定純屬巧合，所以你根本不用擔心這件事啦。

恩，還要記得一點，如果一個函式允許你代入一個 struct sockaddr _參數時，你就可以放心地將你的 struct sockaddr\_in_、struct sockaddr\_in6 _或 struct sockadd\_storage_ 轉型成這個型別。

struct sockaddr\_in 是 IPv4 addresses（例如："192.0.2.10"）在用的資料結構，它承載了一個 address family (AF\_INET)：在 sin\_port 有一個 port，而在 sin\_addr 有一個 IPv4 address in sin\_addr。

還有 struct sockadd\_in 中的 sin\_zero 欄位，有些人認為這個一定要設定成零，但有些人認為不用設定任何值（Linux 文件一點也沒有提過這件事），而且設定成零似乎沒有實際的用途。不過如果你喜歡，還是可以用 memset() 將它設定為零。

目前 struct in\_addr 在不同的系統上是個怪東西，有時候它是瘋狂的大總匯（crazy union），有各種 #define 與一堆鬼東西。不過你該做的只是用這個 structure 的 s\_addr 欄位，因為多數的系統只會實作這個欄位。

struct sockadd\_in6 與 struct in6\_addr 不僅都適用於 IPv6，而且都非常相似。

當你試著寫與 IP 版本無關的程式時，struct sockaddr\_storage 是你可以傳遞給 accept() 或 recvfrom() 的資料結構，而且你不需知道新的 address 是走 IPv4 或 IPv6 協定。資料結構 struct sockaddr\_storage 不像原本小小的 struct sockaddr，而是大到足以承載兩種型別。

## 範例

```c
// IPv4:

struct sockaddr_in ip4addr;
int s;

ip4addr.sin_family = AF_INET;
ip4addr.sin_port = htons(3490);
inet_pton(AF_INET, "10.0.0.1", &ip4addr.sin_addr);

s = socket(PF_INET, SOCK_STREAM, 0);
bind(s, (struct sockaddr*)&ip4addr, sizeof ip4addr);
// IPv6:

struct sockaddr_in6 ip6addr;
int s;

ip6addr.sin6_family = AF_INET6;
ip6addr.sin6_port = htons(4950);
inet_pton(AF_INET6, "2001:db8:8714:3a90::12", &ip6addr.sin6_addr);

s = socket(PF_INET6, SOCK_STREAM, 0);
bind(s, (struct sockaddr*)&ip6addr, sizeof ip6addr);
```

## 參考

accept(), bind(), connect(), inet\_aton(), inet\_ntoa()

# inet\_ntop(), inet\_pton()

將 IP addresses 轉換為可讀的格式，及轉換回來。

## 函式原型

```c
#include <arpa/inet.h>

const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);

int inet_pton(int af, const char *src, void *dst);
```

## 說明

這些函式用於處理將可讀的 IP addresses 轉換為許多函式與 system calls 使用的二進位格式，"n" 表示 "network"，而 "p" 表示 "presentation" 或 "text presentation"。可是你可以想成它是＂可印的＂，而 "ntop" 表示 "network to printable"，這樣知道了嗎？

有時你不想要看一堆二進制數字格式的 IP address，你想要比較好懂的格式，類似 192.0.2.180 或 2001:db8:8714:3a90::12 這樣的格式。在這種情況下，你可以使用 inet\_ntop()。

inet\_ntop() 在 af 參數中代入 address family（不是 AF\_INET 就是 AF\_INET6），src 參數應該是個指向 struct in\_addr 或 struct in6\_addr 的指標，會包含你想要轉換為字串的 address。最後，dst 與 size 是指向目地字串與該字串最大長度的指標。

dst 字串的最大長度是多少呢？ IPv4 與 IPv6 address 的最大長度是多少呢？很幸運地，有兩個 macros（巨集）可以幫你做這件事，最大長度是：INET\_ADDRSTRLEN 與 INET6\_ADDRSTRLEN。

有時你可能會需要將字串格式的 IP address 封裝到 struct sockaddr\_in 或 struct sockaddr\_in6，此時，相對的 inet\_pton() 就是你需要的函式。

inet\_pton() 也會在 af 參數代入一個 address family（不是 AF\_INET 就是 AF\_INET6）、src 參數是指向可列印格式的 IP address 字串、最後的 dst 參數指向要儲存結果的地方，這可能是 struct in\_addr 或 struct in6\_addr。

上述的這些函式不能做 DNS 查詢，你需要使用 getaddinfo()。

## 傳回值

inet\_ntop() 成功時傳回 dst 參數，失敗時傳回 NULL（並設定 errno）。

inet\_pton() 成功時傳回 1，有錯誤時傳回 -1（設定 errno）；若輸入的 IP address 不正確，則傳回 0。

## 範例

```c
// inet_ntop() 與 inet_pton() 的 IPv4 demo

struct sockaddr_in sa;
char str[INET_ADDRSTRLEN];

// 將這個 IP address 儲存在 sa：
inet_pton(AF_INET, "192.0.2.33", &(sa.sin_addr));

// 現在取回並印出來
inet_ntop(AF_INET, &(sa.sin_addr), str, INET_ADDRSTRLEN);

printf("%s\n", str); // 印出 "192.0.2.33"
// inet_ntop() 與 inet_pton() 的 IPv6 Demo
// （基本上一樣，除了多個 6）

struct sockaddr_in6 sa;
char str[INET6_ADDRSTRLEN];

// 將 IP address 儲存在 sa：
inet_pton(AF_INET6, "2001:db8:8714:3a90::12", &(sa.sin6_addr));

// 現在取回並印出來
inet_ntop(AF_INET6, &(sa.sin6_addr), str, INET6_ADDRSTRLEN);

printf("%s\n", str); // 印出 "2001:db8:8714:3a90::12"
// 你會用到的實用函式：

//將 sockaddr address 轉換為字串，包含 IPv4 與 IPv6：

char *get_ip_str(const struct sockaddr *sa, char *s, size_t maxlen)
{
    switch(sa->sa_family) {
        case AF_INET:
            inet_ntop(AF_INET, &(((struct sockaddr_in *)sa)->sin_addr),
                    s, maxlen);
            break;

        case AF_INET6:
            inet_ntop(AF_INET6, &(((struct sockaddr_in6 *)sa)->sin6_addr),
                    s, maxlen);
            break;

        default:
            strncpy(s, "Unknown AF", maxlen);
            return NULL;
    }

    return s;
}
```

## 參考

getaddrinfo()

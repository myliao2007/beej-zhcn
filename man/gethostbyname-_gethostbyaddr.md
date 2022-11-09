# gethostbyname(), gethostbyaddr()

取得 hostname 的 IP address for a hostname，反之亦然

## 函式原型

```c
#include <sys/socket.h>
#include <netdb.h>

struct hostent *gethostbyname(const char *name); // 不建議使用！
struct hostent *gethostbyaddr(const char *addr, int len, int type);
```

## 說明

請注意：這兩個函式已經由 getaddrinfo() 與 getnameinfo() 取而代之！實際上，gethostbyname() 無法在 IPv6 中正常運作。

這些函式可以轉換 host names 與 IP addresses。例如：你可以用 gethostbyname() 取得 " 其 IP addresses，並儲存在 struct in\_addr。

反之，如果你有一個 struct in\_addr 或 struct in6\_addr，你可以用 gethostbyaddr() 取回 hostname。gethostbyaddr() 與 IPv6 相容，但是你應該使用新的 getnameinfo() 取代之。

（如果你有一個字串是句點與數字組成的格式，你想要查詢它的 hostname，你在使用 getaddrinfo() 時最好要搭配 AI\_CANONNAME flag）。

gethostbyname() 接收一個類似 "www.yahoo.com" 的字串，然後傳回一個 struct hostent，裡面包含幾萬噸的資料，包括了 IP address（其它的資訊是官方的 host name、一連串的別名、位址型別、位址長度、以及位址清單。這是個通用的資料結構，在特定的用途上也很易於使用）。

在 gethostbyaddr() 代入一個 struct in\_addr 或 struct in6\_addr，然後就會提供你一個相對應的 host name（如果有），因此，它是 gethostbyname() 的相反式。至於參數，addr 是一個 char\*，你實際上想要用一個指向 struct in\_addr 的指標傳遞；len 應是 sizeof(struct in\_addr)，而 type 應為 AF\_INET。所以這個 struct hostent 會帶回什麼呢？它有許多欄位，包含 host 的相關資訊。

char _h\_name 真正的 real canonical host name。 char **h\_aliases 一連串的別名，可以用陣列存取—最後一個元素（element）是 NULL。 int h\_addrtype address type 的答案，這個在我們的用途應該是 AF\_INET。 int length address 的長度（以 byte 為單位），這個在 IP (version 4) address 是 4。 char** h\_addr\_list 這個主機的 IP addresses 清單。雖然這是個 char\*\*，不過實際上是 struct in\_addr_s 所偽裝的陣列，最後一個元素是 NULL。 h\_addr 為 h\_addr\_list\[0] 所定義的通用別名，如果你只是想要任意一個舊有的 IP addres，就用這個欄位吧。（耶，它們可以大於一個）。

## 傳回值

成功時傳回指向 struct hostent 結果的指標，錯誤時傳回 NULL。

跟你平常使用的錯誤報告工具不同，也不是一般的 perror()，這些函式在 h\_errno 變數中有同樣的結果，可以使用 herror() 或 hstrerror() 函式印出來，這些函式運作的方式類似你常用的典型 errnor、perror() 及 strerror() 函式。

## 範例

```c
// 不建議使用這個方式取得 host name
// 建議使用 getaddrinfo()！

#include <stdio.h>
#include <errno.h>
#include <netdb.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main(int argc, char *argv[])
{
  int i;
  struct hostent *he;
  struct in_addr **addr_list;

  if (argc != 2) {
    fprintf(stderr,"usage: ghbn hostname\n");
    return 1;
  }

  if ((he = gethostbyname(argv[1])) == NULL) { // 取得 host 資訊
    herror("gethostbyname");
    return 2;
  }

  // 印出關於這個 host 的資訊：
  printf("Official name is: %s\n", he->h_name);
  printf(" IP addresses: ");
  addr_list = (struct in_addr **)he->h_addr_list;
  for(i = 0; addr_list[i] != NULL; i++) {
    printf("%s ", inet_ntoa(*addr_list[i]));
  }
  printf("\n");

  return 0;
}
// 這個方法已經被 getnameinfo() 取代了

struct hostent *he;
struct in_addr ipv4addr;
struct in6_addr ipv6addr;

inet_pton(AF_INET, "192.0.2.34", &ipv4addr);
he = gethostbyaddr(&ipv4addr, sizeof ipv4addr, AF_INET);
printf("Host name: %s\n", he->h_name);

inet_pton(AF_INET6, "2001:db8:63b3:1::beef", &ipv6addr);
he = gethostbyaddr(&ipv6addr, sizeof ipv6addr, AF_INET6);
printf("Host name: %s\n", he->h_name);
```

## 參考

getaddrinfo(), getnameinfo(), gethostname(), errno, perror(), strerror(), struct in\_addr

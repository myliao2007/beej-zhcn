# inet\_ntoa(), inet\_aton(), inet\_addr

將 IP addresses 從句號與數字格式的字串轉換為 struct in\_addr，以及轉換回來

## 函式原型

```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

// 這些都不建議使用！請使用 inet_pton() 或 inet_ntop() 取代！

char *inet_ntoa(struct in_addr in);
int inet_aton(const char *cp, struct in_addr *inp);
in_addr_t inet_addr(const char *cp);
```

## 說明

因為這些函式無法處理 IPv6，所以建議不要使用！請使用 inet\_ntop() 或 inet\_pton() 來代替！將它們放上來的理由是因為還是可以在某些程式碼看到它們的蹤跡。

這裡的函式都是將 struct in\_addr（struct sockaddr\_in 的一部分）轉換為句號與數字組成的字串格式（例如："192.168.5.10"），反之亦然。如果你將一個 IP address 透過命令列或某種方式傳遞，這是將 struct in\_addr 提供給 connect() 最簡單的方式，諸如此類。如果你需要更多權力，可以試試一些 DNS 用途的函式，比如： gethostbyname()，或者試著在你的國家發動政變。

inet\_ntoa() 函式將 network address 由 struct in\_addr 轉換為句號與數字組成的字串格式，依照過去的習慣，"ntoa" 裡的 "n" 表示 network，而 "a" 表示 ASCII（所以這是 "network To ASCII" - "toa" 後綴類似它的好朋友，C 函式庫的 atoi()，這是用來將 ASCII 字串轉換為整數 ）。

inet\_aton() 函式則是相反的功能，將句號與數字組成的字串格式轉換到 in\_addr\_t（這你 struct in\_addr 中 s\_addr 欄位的型別）。

最後，inet\_addr() 是個舊函式，基本上與 inet\_aton() 是一樣的東西，理論上不宜使用，但是你還是會很常遇到它，而且如果你用了，警察也不會來找你。

## 傳回值

inet\_aton() 若 address 是合法的，則傳回非零的值，而若位址是非法的，則傳回零。

inet\_ntoa() 在 static 緩衝區中傳回句點與數字格式的字串，每次呼叫這個函式時都會覆蓋緩衝區。

inet\_addr() 以 in\_addr\_t 傳回 address，若發生錯誤時傳回 -1（如果你試著轉換一個合法的 IP address 字串 "255.255.255.255"，也是會有同樣的結果，這就是為什麼 inet\_aton() 比較好了）。

## 範例

```c
struct sockaddr_in antelope;
char *some_addr;

inet_aton("10.0.0.1", &antelope.sin_addr); // 將 IP 存在 antelope

some_addr = inet_ntoa(antelope.sin_addr); // 傳回 IP
printf("%s\n", some_addr); // 輸出 "10.0.0.1"

// 而這個呼叫與上面的 inet_aton() call 相同:
antelope.sin_addr.s_addr = inet_addr("10.0.0.1");
```

## 參見

inet\_ntop(), inet\_pton(), gethostbyname(), gethostbyaddr()

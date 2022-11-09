# 7.4. Serialization：如何封装数据

要将文字数据透过网路传送很简单，你已经知道了，不过如果你想要送一些 ＂二进制＂ 的数据，如 int 或 float，会发生什麽事情呢？这里有一些选择。\
\
1\. 将数字转换为文字，使用如 **sprintf()** 的函数，接着传送文字。接收者会使用如 **strtol()** 函数解析文字，并转换为数字。\
\
2\. 直接以原始数据传送，将指向数据的指针传递给 **send()**。\
\
3\. 将数字编码（encode）为可移植的二进制格式，接收者会将它译码（decode）。\
\
先睹为快！只在今晚！\
\
［序幕］\
Beej 说：＂我偏好上面的第三个方法！＂\
［结束］\
\
（在我热切开始本章节之前，我应该要跟你说有现成的程序库可以做这件事情，而要自制个可移植及无错误的作品会是相当大的挑战。所以在决定要自己实作这部分时，可以先四处看看，并做完你的家庭作业。我在这里引用些类似这个作品的有趣的资料。）\
\
实际上，上面全部的方法都有它们的缺点与优点，但是如我所述，通常我偏好第三个方法。首先，咱们先谈谈另外两个的优缺点。\
\
第一个方法，在传送以前先将数字编码为文字，优点是你可以很容易打印出及读取来自网路的数据。有时，人类易读的协定比较适用於频带不敏感（non-bandwidth-intensive）的情况，例如：Internet Relay Chat（IRC）\[27]。然而，缺点是转换耗时，且总是需要比原本的数字使用更多的空间。\
\
第二个方法：传送原始数据（raw data），这个方法相当简单［但是危险！］：只要将数据指针提供给 send()。\


```c
double d = 3490.15926535;

send(s, &d, sizeof d, 0); /* 危险，不具可移植性！ */
```

接收者类似这样接收：\


```c
double d;

recv(s, &d, sizeof d, 0); /* 危险，不具可移植性！ */
```

\
快速又简单，那有什麽不好的呢？\
\
好的，事实证明不是全部的架构都能表示 double［或 int］。［嘿！或许你不需要可移植性，在这样的情况下这个方法很好，而且快速。］\
\
当封装整数型别时，我们已经知道 **htons()** 这类的函数如何透过将数字转换为 Network Byte Order（网路字节顺序），来让东西可以移植。毫无疑问地，没有类似的函数可以供 float 型别使用，全部的希望都落空了吗？\
\
别怕！［你有担心了一会儿吗？没有吗？一点都没有吗？］\
\
我们可以做件事情：我们可以将数据封装为接收者已知的二进制格式，让接收着可以在远端解压。\
\
我所谓的 ＂已知的二进制格式＂是什麽意思呢？\
\
好的，我们已经看过了 **htons()** 范例了，不是吗？它将数字从 host 格式改变［或是 ＂编码＂］为 Network Byte Order 格式；如果要反转［译码］这个数字，接收端会调用 **ntohs()**。\
\
可是我不是才刚说过，没有这样的函数可供非整数型别使用吗？\
\
是的，我说过。而且因为 C 语言并没有规范标准的方式来做，所以这有点麻烦［that a gratuitous pun there for you Python fans］。\
\
要做的事情是将数据封装到已知的格式，并透过网路送出。例如：封装 float，这里的东西有很大的改善空间：\[28]\


```c
#include <stdint.h>

uint32_t htonf(float f)
{
  uint32_t p;
  uint32_t sign;

  if (f < 0) { sign = 1; f = -f; }
  else { sign = 0; }

  p = ((((uint32_t)f)&0x7fff)<<16) | (sign<<31); // whole part and sign
  p |= (uint32_t)(((f - (int)f) * 65536.0f))&0xffff; // fraction

  return p;
}

float ntohf(uint32_t p)
{
  float f = ((p>>16)&0x7fff); // whole part
  f += (p&0xffff) / 65536.0f; // fraction

  if (((p>>31)&0x1) == 0x1) { f = -f; } // sign bit set

  return f;
}
```

上列的代码是一个 native（原生的）实作，将 float 储存为 32-bit 的数字。High bit（高比特）［31］用来储存数字的正负号［＂1＂表示负数］，而接下来的七个比特［30-16］是用来储存 float 整个数字的部分。最後，剩下的比特［15-0］用来储存数字的小数（fractional portion）部分。\
\
使用方式相当直觉：\


```c
#include <stdio.h>

int main(void)
{
  float f = 3.1415926, f2;
  uint32_t netf;

  netf = htonf(f); // 转换为 "network" 形式
  f2 = ntohf(netf); // 转回测试

  printf("Original: %f\n", f); // 3.141593
  printf(" Network: 0x%08X\n", netf); // 0x0003243F
  printf("Unpacked: %f\n", f2); // 3.141586

  return 0;
}
```

好处是：它很小丶很简单且快速，缺点是：它在空间的使用没有效率，而且对范围有严格的限制－试着在那边储存一个大於 32767 的数，它就会不高兴！\
\
你也可以在上面的例子看到，最後一对的十进位空间并没有正确保存。\
\
我们该怎麽改呢？\
\
好的，用来储存浮点数（float point number）的标准方式是已知的 IEEE-754 \[29]。多数的电脑会在内部使用这个格式做浮点运算，所以在这些例子里，严格说来，不需要做转换。但是如果你想要你的代码具可移植性，就要假设你不需要转换。［换句话说，如果你想要让程序很快，你应该要在不需要做转换的平台上进行最佳化！这就是 **htons()** 与它的家族使用的方法。］\
\
这边有段代码可以将 float 与 double 编码为 IEEE-754 格式 \[30]。［主要的功能，它不会编码 NaN 或 Infinity，不过可以将它改成可以。］\


```c
#define pack754_32(f) (pack754((f), 32, 8))
#define pack754_64(f) (pack754((f), 64, 11))
#define unpack754_32(i) (unpack754((i), 32, 8))
#define unpack754_64(i) (unpack754((i), 64, 11))

uint64_t pack754(long double f, unsigned bits, unsigned expbits)
{
  long double fnorm;
  int shift;
  long long sign, exp, significand;
  unsigned significandbits = bits - expbits - 1; // -1 for sign bit

  if (f == 0.0) return 0; // get this special case out of the way

  // 检查正负号并开始正规化
  if (f < 0) { sign = 1; fnorm = -f; }
  else { sign = 0; fnorm = f; }

  // 取得 f 的正规化型式并追踪指数
  shift = 0;
  while(fnorm >= 2.0) { fnorm /= 2.0; shift++; }
  while(fnorm < 1.0) { fnorm *= 2.0; shift--; }
  fnorm = fnorm - 1.0;

  // 计算有效位数数据的二进制格式（非浮点数）
  significand = fnorm * ((1LL<<significandbits) + 0.5f);

  // get the biased exponent
  exp = shift + ((1<<(expbits-1)) - 1); // shift + bias

  // 返回最後的解答
  return (sign<<(bits-1)) | (exp<<(bits-expbits-1)) | significand;
}

long double unpack754(uint64_t i, unsigned bits, unsigned expbits)
{
  long double result;
  long long shift;
  unsigned bias;
  unsigned significandbits = bits - expbits - 1; // -1 for sign bit

  if (i == 0) return 0.0;

  // pull the significand

  result = (i&((1LL<<significandbits)-1)); // mask
  result /= (1LL<<significandbits); // convert back to float
  result += 1.0f; // add the one back on

  // deal with the exponent
  bias = (1<<(expbits-1)) - 1;
  shift = ((i>>significandbits)&((1LL<<expbits)-1)) - bias;
  while(shift > 0) { result *= 2.0; shift--; }
  while(shift < 0) { result /= 2.0; shift++; }

  // sign it
  result *= (i>>(bits-1))&1? -1.0: 1.0;

  return result;
}
```

我在那里的顶端放一些方便的 macro 用来封装与解除封装 32-bit［或许是 float］与 64-bit［或许是 double］的数字，但是 **pack754()** 函数可以直接调用，并告知编码几个比特的数据［expbits 的哪几个比特要保留给正规化数值的指数。］\
\
这里是使用范例：\


```c
#include <stdio.h>
#include <stdint.h> // 定义 uintN_t 型别
#include <inttypes.h> // 定义 PRIx macros

int main(void)
{
  float f = 3.1415926, f2;
  double d = 3.14159265358979323, d2;
  uint32_t fi;
  uint64_t di;

  fi = pack754_32(f);
  f2 = unpack754_32(fi);

  di = pack754_64(d);
  d2 = unpack754_64(di);

  printf("float before : %.7f\n", f);
  printf("float encoded: 0x%08" PRIx32 "\n", fi);
  printf("float after : %.7f\n\n", f2);

  printf("double before : %.20lf\n", d);
  printf("double encoded: 0x%016" PRIx64 "\n", di);
  printf("double after : %.20lf\n", d2);

  return 0;
}
```

上面的代码会产生下列的输出：\


```c
float before : 3.1415925
float encoded: 0x40490FDA
float after  : 3.1415925

double before : 3.14159265358979311600
double encoded: 0x400921FB54442D18
double after  : 3.14159265358979311600
```

你可能遭遇的另一个问题是你该如何封装 struct 呢？\
\
对你来说没有问题的，编译器会自动将一个 struct 中的全部空间填入。［你不会病到听成 ＂不能这样做＂丶＂不能那样做＂？抱歉！引述一个朋友的话：＂当事情出错了，我都会责怪 Microsoft。＂这次固然可能不是 Microsoft 的错，不过我朋友的陈述完全符合事实。］\
\
回到这边，透过网路送出 struct 的最好方式是将每个栏位独立封装，并接着在它们抵达另一端时，将它们解封装到 struct。\
\
你正在想，这样要做很多事情。是的，的确是。一件你能做的事情是写一个有用的函数来帮你封装数据，这很好玩！真的！\
\
在 Kernighan 与 Pike 着作的 ＂The Practice of Programming＂\[31] 这本书，他们实作类似 **printf()** 的函数，名为 **pack()** 与 **unpack()**，可以完全做到这件事。我想要连结到这些函数，但是这些函数显然地无法从网路上取得。\
\
［The Practice of Programming 是值得阅读的好书，Zeus saves a kitten every time I recommend it。］\
\
此时，我正要舍弃指向我从未用过的 BSD 授权类型的参数语言 C API（BSD-licensed Typed Parameter Language C API）\[32] 的指针，可是看起来整个很可敬。Python 与 Perl 程序设计师想要找出他们语言的 **pack()** 与 **unpack()** 函数，用来完成同样的事情。而 Java 有一个能用於同样用途的 big-ol' Serializable interface。\
\
不过如果你想要用 C 写你自己的封装工具，K\&P 的技巧是使用变动参数列（variable argument list），来让类似 **printf()** 的函数建立数据包。我自己所编造的版本 \[33] 希望足以供你了解这样的东西是如何运作的。\
\
［这段代码参考到上面的 **pack754()** 函数，**packi\*()** 函数的运作方式类似 **htons()** 家族，除非它们是封装到一个 char 数组（array）而不是另一个整数。］\


```c
#include <ctype.h>
#include <stdarg.h>
#include <string.h>
#include <stdint.h>
#include <inttypes.h>

// 供浮点数型别的变动比特
// 随着架构而变动

typedef float float32_t;
typedef double float64_t;

/*
** packi16() -- store a 16-bit int into a char buffer (like htons())
*/
void packi16(unsigned char *buf, unsigned int i)
{
  *buf++ = i>>8; *buf++ = i;
}

/*
** packi32() -- store a 32-bit int into a char buffer (like htonl())
*/
void packi32(unsigned char *buf, unsigned long i)
{
  *buf++ = i>>24; *buf++ = i>>16;
  *buf++ = i>>8; *buf++ = i;
}

/*
** unpacki16() -- unpack a 16-bit int from a char buffer (like ntohs())
*/
unsigned int unpacki16(unsigned char *buf)
{
  return (buf[0]<<8) | buf[1];
}

/*
** unpacki32() -- unpack a 32-bit int from a char buffer (like ntohl())
*/
unsigned long unpacki32(unsigned char *buf)
{
  return (buf[0]<<24) | (buf[1]<<16) | (buf[2]<<8) | buf[3];
}

/*
** pack() -- store data dictated by the format string in the buffer
**
** h - 16-bit l - 32-bit
** c - 8-bit char f - float, 32-bit
** s - string (16-bit length is automatically prepended)
*/
int32_t pack(unsigned char *buf, char *format, ...)
{
  va_list ap;
  int16_t h;
  int32_t l;
  int8_t c;
  float32_t f;
  char *s;
  int32_t size = 0, len;

  va_start(ap, format);
  
  for(; *format != '\0'; format++) {
    switch(*format) {
    case 'h': // 16-bit
      size += 2;
      h = (int16_t)va_arg(ap, int); // promoted
      packi16(buf, h);
      buf += 2;
      break;

    case 'l': // 32-bit
      size += 4;
      l = va_arg(ap, int32_t);
      packi32(buf, l);
      buf += 4;
      break;

    case 'c': // 8-bit
      size += 1;
      c = (int8_t)va_arg(ap, int); // promoted
      *buf++ = (c>>0)&0xff;
      break;

    case 'f': // float
      size += 4;
      f = (float32_t)va_arg(ap, double); // promoted
      l = pack754_32(f); // convert to IEEE 754
      packi32(buf, l);
      buf += 4;
      break;

    case 's': // string
      s = va_arg(ap, char*);
      len = strlen(s);
      size += len + 2;
      packi16(buf, len);
      buf += 2;
      memcpy(buf, s, len);
      buf += len;
      break;
    }
  }

  va_end(ap);

  return size;
}
/*
** unpack() -- unpack data dictated by the format string into the buffer
*/
void unpack(unsigned char *buf, char *format, ...)
{
  va_list ap;
  int16_t *h;
  int32_t *l;
  int32_t pf;
  int8_t *c;
  float32_t *f;
  char *s;
  int32_t len, count, maxstrlen=0;

  va_start(ap, format);

  for(; *format != '\0'; format++) {
    switch(*format) {
    case 'h': // 16-bit
      h = va_arg(ap, int16_t*);
      *h = unpacki16(buf);
      buf += 2;
      break;

    case 'l': // 32-bit
      l = va_arg(ap, int32_t*);
      *l = unpacki32(buf);
      buf += 4;
      break;

    case 'c': // 8-bit
      c = va_arg(ap, int8_t*);
      *c = *buf++;
      break;

    case 'f': // float
      f = va_arg(ap, float32_t*);
      pf = unpacki32(buf);
      buf += 4;
      *f = unpack754_32(pf);
      break;

    case 's': // string
      s = va_arg(ap, char*);
      len = unpacki16(buf);
      buf += 2;
      if (maxstrlen > 0 && len > maxstrlen) count = maxstrlen - 1;
      else count = len;
      memcpy(s, buf, count);
      s[count] = '\0';
      buf += len;
      break;

    default:
      if (isdigit(*format)) { // track max str len
        maxstrlen = maxstrlen * 10 + (*format-'0');
      }
    }

    if (!isdigit(*format)) maxstrlen = 0;
  }

  va_end(ap);
}
```

不管你是自己写的程序，或者用别人的代码，基於持续检查 bugs 的理由，能有通用的数据封装机制集是个好主意，而且不用每次都手动封装每个 bit（比特）。\
\
在封装数据时，使用哪种格式会比较好呢？\
\
好问题，很幸运地，RFC 4506 \[35]，the External Data Representation Standard 已经定义了一堆各类型的二进位格式，如：浮点数型别丶整数型别丶数组丶原始数据等。如果你打算自己写程序来封装数据，我建议要与标准符合。只是不会强制你一定要这样做。数据包的政策不会刚好就在你门口。至少，我不认为它们会在。\
\
不管怎样，在你送出数据以前，以某种或其它方法将数据编码是对的做事方法。\
\
\[27] http://en.wikipedia.org/wiki/Internet\_Relay\_Chat\
\[28] http://beej.us/guide/bgnet/examples/pack.c\
\[29] http://en.wikipedia.org/wiki/IEEE\_754\
\[30] http://beej.us/guide/bgnet/examples/ieee754.c\
\[31] http://cm.bell-labs.com/cm/cs/tpop/\
\[32] http://tpl.sourceforge.net/\
\[33] http://beej.us/guide/bgnet/examples/pack2.c

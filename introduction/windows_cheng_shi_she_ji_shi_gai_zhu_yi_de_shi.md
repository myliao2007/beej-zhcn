# 1.5. Windows 程序员要注意的事情

本教程以前只讨论一点 Windows，纯粹是我很不喜欢。不过我应该要客观的说 Windows 其实提供很多基本安装，所以显然是个完备的操作系统。

人家说：小别胜新婚，这里我相信这句话是对的［或许是年纪的关系］。不过我只能说，我已经十几年没有用 Microsoft 的操作系统来做自己的工作了，这样我很开心！

其实我可以安然地打安全牌告诉你：＂没问题阿，你尽量去用 Windows 吧！＂… 没错，其实我是咬着牙根说这些话的。

所以我还是在拉拢你来试试 Linux \[[1](http://www.google.com/url?q=http%3A%2F%2Fwww.linux.com%2F\&sa=D\&sntz=1\&usg=AOvVaw0SOaUEXpRgbhWDQVZcHPlz)]丶BSD \[[2](http://www.google.com/url?q=http%3A%2F%2Fwww.bsd.org%2F\&sa=D\&sntz=1\&usg=AOvVaw3DnteIy74oHj15pKbMpUfn)]，或一些 Unix 风格的系统。

不过人们各有所好，而 Windows 的用戶也乐於知道这份教程的内容能用在 Windows，只是需要改变一点代码而已。

你可以安装一个酷玩意儿－ Cygwin \[[3](http://www.google.com/url?q=http%3A%2F%2Fwww.cygwin.com%2F\&sa=D\&sntz=1\&usg=AOvVaw2wZqp8x7z5IrgPf\_iTT8YS)]，这是让 Windows 平台使用的 Unix 工具集。我曾在秘密情报网听过，这个能让全部的代码不经过修改就能编译。

不过有些人可能想要用纯 Windows 的方法来做。只能说你很有勇气，而你所要做的事就是：立刻去弄个 Unix！喔，不是，我开玩笑的。这些日子以来，大家一直认为我对 Windows 是很友善的。

你所要做的事情就是［除非你安装了 Cygwin！］：首先要忽略我这边提过的很多系统 header（标头档），而你唯一需要 include （引用）的是：

```
#include <winsock.h>
```

等等，在你用 socket 库做任何事情之前，必须要先调用 WSAStartup()。代码看起来像这样：

```c
#include <winsock.h>
{
  WSADATA wsaData; // if this doesn't work
  //WSAData wsaData; // then try this instead

  // MAKEWORD(1,1) for Winsock 1.1, MAKEWORD(2,0) for Winsock 2.0:

  if (WSAStartup(MAKEWORD(1,1), &wsaData) != 0) {
    fprintf(stderr, "WSAStartup failed.\n");
    exit(1);
  }
```

你也必须告诉编译器要链接（link） Winsock 程序库，在 Winsock 2.0 通常称为 wsock32.lib 或 winsock32.lib 或 ws2\_32.lib。在 VC++ 底下，可以透过项目（Project）菜单，在设置（Settings）底下 …。按下 Link tag，并找到＂Object/library modules＂的标题。新增＂wsock32.lib＂（或者你想要用的程序库）到表中。

最後，当你用好 socket 程序库时，你需要调用 WSACleanup()，细节请参考在线帮助手册。

只要你做好这些工作，本教程後面的示例应该都能顺利编译，只有少部分例外。

还有一件事情，你不能用 close() 关闭 socket，你要用 closesocket() 来取代。而且 select() 只能用在 socket descriptors 上，不能用在 file descriptors（像 stdin 就是 0）。

还有一种你能用的 socket 类型，CSocket，细节请查询你的编译器手册。

要取得更多关於 Winsock 的信息可以先阅读 Winsock FAQ \[[4](http://www.google.com/url?q=http%3A%2F%2Ftangentsoft.net%2Fwskfaq%2F\&sa=D\&sntz=1\&usg=AOvVaw3dqfoR-017v7jpewYXqrYn)]。

最後，我听说 Windows 没有 fork() 系统调用，我在一些示例中会用到。你可能需要连结到 POSIX 程序库或要让程序能动的一些程序库，或许你也可以用 CreateProcess() 来取代。fork() 不需要参数，但是 CreateProcess() 却需要大约 480 亿个参数。如果你不想用，CreateThread() 会稍微比较容易理解 … 不过多线程（multithreading）的讨论则不在本教程的范畴中。我只能尽量提到而已，你明白的！

译注：作者说 480 亿个参数只是想表达 CreateProcess() 需要的参数很多。

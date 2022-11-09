# 3.2. Byte Order（字节的顺序）

长久以来都有两种 byte orderings，不过後来才知道，根本差多了。

我开玩笑的，不过其中一个真的比另一个好 :-)

这真的不太好解释，所以我只会扯蛋：你的电脑可能背着你用相反的顺序来储存 bytes。

我知道！没有人跟你说。

Byte Order 其实就是，在 Internet 世界中的每个人一般都已经同意的，如果你想要用两个 bytes 的十六进制数字来表示，比如说 b34f，你可以将它以 b34f 的顺序储存。很合理，而 Wilford Brimley \[14] 会跟你说，这麽做是对的。这个数字是先储存比较大的那一边（big end），所以称为 Big-Endian。

毫无疑问地，世界上的电脑那麽多，像 Intel 或 Intel 兼容的中央处理器就是将 bytes 反过来储存，所以 b34f 存在内存中的顺序就是 4fb3，这样的储存方式称为 Little-Endian。

不过，等等。我还没解释名词！照理说，Big-Endian 又称为 Network Byte Order，因为这个顺序与我们网路型别的顺序一样。

你的电脑会以 Host Byte Order 储存数字，如果是 Intel 80x86，Host Byte Order 是 Little-Endian；若是 Motorola 68k，则 Host Byte Order 是 Big-Endian；若是 PowerPC，Host Byte Order 就是 … 恩，这要看你的 PowerPC 而定。

大多数当你在打造数据包或填写数据结构时，你需要确认你的两个数字跟四个数字都是 Network Byte Order。只是如果你不知道本地端的 Host Byte Order，那该怎麽做呢？

好消息是你只需要假设 Host Byte Order 不正确，然後每次都透过一个函数将值设定为 Network Byte Order。如果有必要，该函数会进行魔法的转换，而这个方式可以让你的代码能方便的移植到不同 endian 的机器上。

你可以转换两种型别的数值：short［两个 bytes］与 long［四个 bytes］。这些函数也可以用在 unsigned 变量。比如说，你想要将 short 从 Host Byte Order 转换为 Network Byte Order，用＂h＂代表＂host＂，用＂n＂代表＂network＂，而＂s＂代表＂short＂，所以是：h-to-n-s，或者htons()［读做：＂Host to Network Short＂］。

这真是太简单了…

你可以用任何你想要的方式来组合＂n＂丶＂h＂丶＂s＂与＂l＂，不过别用太蠢的组合，比如：没有这样的函数 stolh()［＂Short to Long Host＂］，没有这种东西，不过有：

```c
htons() host to network short
htonl() host to network long
ntohs() network to host short
ntohl() network to host long
```

基本上，你需要在送出以前将数值转换为 Network Byte Order，并在收到之後将数值转回 Host Byte Order。

抱歉，我不知道 64-bit 的改变，如果你想要做浮点数的话，可以参考第 7-4 节。

\[14] http://en.wikipedia.org/wiki/Wilford\_Brimley

如果我没特别强调的话，本文中的数值默认值是 Host Byte Order。

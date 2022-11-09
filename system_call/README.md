# 5. System call 或 Bust

我们在本章开始讨论如何让你存取 UNIX 系统或 BSD丶Windows丶Linux丶Mac 等系统的 system call（系统调用）丶socket API 及其它 function calls（函数调用）等网路功能。当你调用其中一个函数时，kernel 会接管，并且自动帮你处理全部的工作。

多数人会停滞在这里是因为不知道要用什麽样的顺序来调用这些函数，而你在找 man 手册时会觉得很难用。好，为了要帮忙解决这可怕的困境，我已经试着在下列的章节精确地布局（layout） system call，你在编程时只要照着一样的顺序调用就可以了。

为了要链接一些代码，需要一些牛奶跟饼乾［这恐怕你要自行准备］，以及一些决心与勇气，而你就能将数据发送到互联网上，彷佛是 Jon Postel 之子般。

［请注意，为了简洁，下列许多代码片段并没有包含错误检查的代码。而且它们很喜欢假设调用 **getaddrinfo()** 的结果都会成功，并会返回链表（link-list）中的一个有效资料。这两种情况在单独运行的程序都有严谨的定位，所以，我们还是将它们当作模型来使用吧。］
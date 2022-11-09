# 2. 何谓 Socket

你一直听到人家在讲＂sockets＂（套接字），你可能也想知道这些是什麽东西。

好的，其实它们是：利用标准 UNIX file descriptors（文件描述符）与其它程序沟通的一种方式。

什麽？

OK，你可能有听过有些黑客（hacker）说过：＂我的天呀！在 UNIX 系统中的任何东西都可以视为文件！＂

他说的是事实。当 UNIX 程序要做任何类型的 I/O 时，它们会读写 file descriptor。File descriptor 单纯只是跟已开启文件有关的整数。只是［关键在於］，该文件可以是一个网路连接丶FIFO丶pipe（管道）丶terminal（终端）丶真实的磁盘文件丶或只是相关的东西。在 UNIX 所见都是文件！所以当你想要透过 Internet（互联网）跟其它的程序沟通时，你需要透过一个 file descriptor 来达成，这点你一定要相信。

＂那麽，Smarty-Pants 先生，我在哪里可以取得这个用在网路通訊的 file descriptor 呢？＂

这可能是你现在心里的问题，我会跟你说的：你能调用 socket() system routine（系统例程）。它会传回 socket descriptor，你可以用精心设计的 send() 与 recv() socket calls［man send丶man recv］来透过 socket descriptor 进行通讯。

＂不过，嘿嘿！＂

现在你可能在想：＂既然只是个 file descriptor，为什麽我不能用一般的 read() 与 write() call 透过 socket 进行通讯，而要用这什麽鬼东西？＂

简单说：＂可以！＂

详细点的说法是：＂可以，不过 send() 与 recv() 让你能对数据传输有更多的控制权＂。

接下来呢？

这麽说吧：有很多种 sockets，如 DARPA Internet Sockets（互联网地址）丶本地节点的路径名（path names on a local node，UNIX Sockets）丶CCITT X.25 地址（你可以放心忽略 X.25 Sockets），可能还有其它的，要看你用的是哪种 UNIX 系统。在这里我们只讨论第一种：Internet Sockets。

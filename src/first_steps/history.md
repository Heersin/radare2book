## 历史

2006年， Sergi Àlvarez (aka pancake) 在当时从事取证分析的工作。由于不允许使用私有软件满足个人所需，他决定编写一个具有小型16进制编辑器，使其具有如下特点：

* 高便携性 (unix 友好, 命令行界面, C语言, 小巧)
* 能以64位的偏移量打开磁盘设备
* 可以搜索字符串和十六进制字节串
* 查看并将结果转存到磁盘上

这个编辑器原本是为了恢复HFS+上一个被删除的文件。

之后，pancake决定扩展该工具，使其有一个可插拔的io接口，能够附着到进程上，实现类似debugger的功能，并且能支持多种架构，以及能够进行代码分析。

从那时起，这个项目发展为提供一个用于分析二进制文件的完整框架，项目中借用了UNIX的一些基本概念，包括著名的“everything is a file”，“small programs that interact using stdin/stdout”和“keep it simple”范式。

脚本化的需求暴露了初代设计上的不足：工具单一而庞大，造成其API很难使用，因此需要对项目进行深度重构。2009年，radare2（r2）作为radare1的一个分支诞生了。重构加入了灵活性以及动态特性，使得工具能够更好地进行集成，从而为r2在[不同编程语言](https://github.com/radareorg/radare2-bindings)中的使用铺平了道路。不久后，[r2pipe API](https://github.com/radareorg/radare2-r2pipe) 使我们能通过pipe在不同语言中使用radare2

从最初的个人项目开始，渐渐在2014年发展为基于社区的项目，用户在飞速地增加，原作者与主开发人员不得不将角色从程序员切换到项目经理， 以整合项目中的来自不同开发者的工作。

xx用户反馈的问题使得本项目能够确定新的发展方向，项目所有内容都放在[radare2's GitHub](https://github.com/radareorg/radare2)进行管理， 并在Telegram频道中进行讨论。

在编写本书时，该项目依旧活跃，并且还有许多衍生项目，比如带图形化界面的radare2 ([Cutter](https://github.com/radareorg/cutter)), 反编译器 ([r2dec](https://github.com/wargio/r2dec-js), [radeco](https://github.com/radareorg/radeco)), frida整合工具 ([r2frida](https://github.com/nowsecure/r2frida)), Yara, Unicorn, Keystone, 以及许多在[r2pm](https://github.com/radareorg/radare2-pm)(radare2 的包管理器) 仓库中索引的项目。

从2016年起，社区每年都会在巴塞罗那的 [r2con](https://www.radare.org/con/) 上进行聚会， 这是一个与radare2相关的会议。
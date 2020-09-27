# 文件

Radar2调试器允许用户列出和处理目标进程中的文件描述符。这是一个未在其他debugger中找到过的有用功能。

该功能与lsof相似，但是其具有额外的子命令用于seek，close以及duplicate文件描述符。

因此，debug session中的任何时刻都可以用r2创建的socket替换stdio这一描述符，或是通过替换socket劫持网络连接。

该功能在r2frida中同样可用，命令是`\dd`。在r2中可以用`dd?`获取更多的信息。

# Basic Debugger Session

指定`-d`选项可启动radare2对程序进行调试。 记住，你可以attach到一个运行中的进程，只需要指定其PID即可。或者通过指定名字和参数启动一个程序。

```
$ pidof mc
32220
$ r2 -d 32220
$ r2 -d /bin/ls
$ r2 -a arm -b 16 -d gdb://192.168.1.43:9090
...
```

在上面例子中的第二种情况下，debugger会fork一个用于调试的ls进程并加载到内存中。它将停在`ld.so`动态链接库前，因此在这一刻，您无法看到程序入口点以及任何的共享库。

可以通过如下方法，用其他名字替换默认的entry breakpoint， 改变这个默认行为：在radare2的启动脚本里加上`e dbg.bep=entry` 或 `e dbg.bep=main` 命令。radare2的启动脚本通常是`~/.config/radare2/radare2rc`。

另一种运行到特定地址的方式是使用`dcu`命令， 这个缩写的意思是 “debug continue until”， 后接停止运行的地址。比如：

```
dcu main
```

请注意，实际上某些恶意软件或其他棘手的程序可以在main（）之前执行代码（例如程序构造函数或tls初始化程序），因此无法通过main处的断点控制它们。

下面列出的是一些debugger常用的命令：
```
> d?            ; get help on debugger commands
> ds 3          ; step 3 times
> db 0x8048920  ; setup a breakpoint
> db -0x8048920 ; remove a breakpoint
> dc            ; continue process execution
> dcs           ; continue until syscall
> dd            ; manipulate file descriptors
> dm            ; show process maps
> dmp A S rwx   ; change permissions of page at A and size S
> dr eax=33     ; set register value. eax = 33
```

Radare内的debugger还提供可视化模式，可能更好上手。该模式下不需要去记忆那么多命令，也不需要时时刻刻在心里记住程序的状态信息。

使用`Vpp`命令进入可视化模式:

```
[0xb7f0c8c0]> Vpp
```

进入可视化模式后的初始视图为当前地址（PC，例如x86上的EIP）下的十六进制转储。
按下`p`可以在多种可视化模式中循环切换，通过`P`和`p`选择不同的可视化视图。使用F7或`s`单步步入当前指令，使用F8单步执行当前指令。可以用`c`键切换到光标模式， 标选字节范围（比如，在之后的工作要用nop填充这一范围内的字节）。 `F2`则是用于在当前位置设置断点。

在可视化模式下， 可以以`:`开头调用radare内的命令。比如，转储一块从ESI开始的内存：
```
<Press ':'>
x @ esi
```
在可视化模式中按下`?`即可获取帮助， 使用方向键可以翻阅帮助手册， 按下`q`可以退出帮助手册。

`dr`也是一个常用的命令， 可以用它对目标的通用寄存器进行读写， 也可以操作硬件和扩展/浮点寄存器。使用`dr=`命令可以以更紧凑的格式显示寄存器内容。


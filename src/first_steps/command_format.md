## Command Format

radare2命令的格式如下:
```
[.][times][cmd][~grep][@[@iter]addr!size][|>pipe] ;
```
使用Vim进行日常工作且熟悉其命令的用户对此格式不会感到陌生，而会觉得几乎没有学习成本。遵循这个格式的命令将会贯穿本书，这些命令通常由一个区分大小写的字符（[a-zA-Z]）作为标识。

在命令前加上数字前缀代表重复执行该命令相应次数：
```
px    # run px
3px   # run px 3 times
```

`!`前缀用于执行系统shell环境中的命令，如果您想使用I/O插件中的cmd回调，需要在前面加上`=!`前缀。

请注意，通过单感叹号调用并运行的命令将通过RConsAPI打印输出信息，执行过程会是阻塞式的， 而不会是可交互的。用双感叹号--`!!`可以调用system call。

可以用`cfg.sandbox`内的配置变量对socket，文件系统和执行API进行限制。

下面是一些radare2命令的例子:
```
ds                    ; call the debugger's 'step' command
px 200 @ esp          ; show 200 hex bytes at esp
pc > file.c           ; dump buffer as a C byte array to file.c
wx 90 @@ sym.*        ; write a nop on every symbol
pd 2000 | grep eax    ; grep opcodes that use the 'eax' register
px 20 ; pd 3 ; px 40  ; multiple commands in a single line
```

在radare2 shell内同样能使用标准的UNIX管道 `|`。可以用管道结合一些程序对输出信息进行过滤，比如`grep`，`less`，`wc`。如果您不想通过管道派生进程之类的，或者说无法在目标系统上派生任何东西，比如说没有相应的UNIX工具组（Windows或者嵌入式系统），在这些情况下你也可以使用内置的grep(`~`)。

在radare2 shell内输入 `~?` 可以获得更多帮助。

`~` 字符起到类似grep的作用，可以过滤任何命令的输出信息:
```
pd 20~call            ; disassemble 20 instructions and grep output for 'call'
```
此外，还可以进行列或行的过滤
```
pd 20~call:0          ; get first row
pd 20~call:1          ; get second row
pd 20~call[0]         ; get first column
pd 20~call[1]         ; get second column
```
或者将二者结合:
```
pd 20~call:0[0]       ; grep the first column of the first row matching 'call'
```
内置grep的功能是支持radare2工作脚本化的关键特性，因为它可以用于遍历一系列偏移地址，或者由反编译器生成的各种数据，遍历一个范围或与其他命令结合。参阅[loops](../scripting/loops.md)章节（迭代器）了解更多信息。

`@`字符用于标识一个临时的偏移量，`@`左边的命令从该地址开始执行，执行结束后seek的位置恢复到命令执行之前。

例如, `pd 5 @ 0x100000fce` 代表反编译0x100000fce处开始的5条指令。

大部分的命令都提供了`TAB`自动补全， 例如`s`eek或`f`lags命令。自动补全将会显示所有可能的结果，以及补全flag的名称。
顺带一提， radare2支持命令历史， `!~...` 提供了一个可视化界面浏览radare2的命令历史。

若要扩展自动补全的功能，使其支持更多命令，您自定义的命令或者I/O插件的命令， 需要用`!!!`命令来完成这些工作。
# 使用gdbserver远程调试

Radee2允许通过gdb远程协议进行远程调试。这样就可以在远程运行gdbserver并使用radare2连接到它以进行远程调试。连接的语法是：
```
$ r2 -d gdb://<host>:<port>
```

注意下面这条命令完成的是同样的操作，若r2能找到uri中指定的debug插件，则r2会使用它。

```
$ r2 -D gdb gdb://<host>:<port>
```

可以在运行时使用`dL`或`Ld`改变debug插件。

如果gdbserver以扩展模式运行，则可以attach到主机上的进程：
```
$ r2 -d gdb://<host>:<port>/<pid>
```

同样你可以在使用`doof`命令对文件分析后启动调试，该命令在打开gdb后会对当前session中的数据进行重建。
```
[0x00404870]> doof gdb://<host>:<port>/<pid>
```

连接后，可以像往常那样使用r2 debug命令。

radare2不从gdbserver那加载符号信息，因此需要本地中存在该文件才能进行符号加载。有些情况下即使本地存在该文件，radare2也没有加载符号信息，需要手动去指定文件路径`e dbg.exe.path`:
```
$ r2 -e dbg.exe.path=<path> -d gdb://<host>:<port>
```

如果符号被加载到错误的基址上，可以试着用`e bin.baddr`重新设定基址。
```
$ r2 -e bin.baddr=<baddr> -e dbg.exe.path=<path> -d gdb://<host>:<port>
```

通常gdbserver会报告其支持的最大数据包大小，否则，radare2会采用可行的默认设置。但是你可以使用环境变量`R2_GDB_PKTSZ`指定最大数据包大小，还可以在与IO系统进行会话的期间检查和设置最大数据包大小，即使用`=!`：
```
$ export R2_GDB_PKTSZ=512
$ r2 -d gdb://<host>:<port>
= attach <pid> <tid>
Assuming filepath <path/to/exe>
[0x7ff659d9fcc0]> =!pktsz
packet size: 512 bytes
[0x7ff659d9fcc0]> =!pktsz 64
[0x7ff659d9fcc0]> =!pktsz
packet size: 64 bytes
```

gdb IO系统提供了一些有用的命令，这些命令可能与任何标准radare2命令都不同。可以使用`=!?`列出这些命令。（请记住，`=！`用于访问底层的IO插件的`system（）`）。
```
[0x7ff659d9fcc0]> =!?
Usage: =!cmd args
 =!pid             - show targeted pid
 =!pkt s           - send packet 's'
 =!monitor cmd     - hex-encode monitor command and pass to target interpreter
 =!rd              - show reverse debugging availability
 =!dsb             - step backwards
 =!dcb             - continue backwards
 =!detach [pid]    - detach from remote/detach specific pid
 =!inv.reg         - invalidate reg cache
 =!pktsz           - get max packet size used
 =!pktsz bytes     - set max. packet size as 'bytes' bytes
 =!exec_file [pid] - get file which was executed for current/specified pid
```

注意`=！dsb`和`=！dcb`仅在特殊的gdbserver实现中可用，例如[Mozilla的rr]（https://github.com/mozilla/rr），默认的gdbserver不包括远程反向调试的功能。使用`=！rd`显示当前可用的反向调试功能。

如果对radare2与gdbserver之间的交互感兴趣，则可以使用`=!monitor set remote-debug 1`在gdbserver控制台中打开gdb远程协议数据包的日志记录功能，以及使用`=!monitor set debug 1`以在控制台显示常规的gdb调试信息。

radare2同样提供了一个自有的gdbserver实现：

```
$ r2 -
[0x00000000]> =g?
|Usage:  =[g] [...] # gdb server
| gdbserver:
| =g port file [args]   listen on 'port' debugging 'file' using gdbserver
| =g! port file [args]  same as above, but debug protocol messages (like gdbserver --remote-debug)
```

可以这样启动它:

```
$ r2 -
[0x00000000]> =g 8000 /bin/radare2 -
```

然后像对其他gdbserver那样连接即可，例如，使用radare2：

```
$ r2 -d gdb://localhost:8000
```


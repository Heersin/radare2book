# Debugger

Debugger是作为IO插件实现的，因此radare可根据不同的URI类型派生程序或attach到进程上。完整的IO插件列表可以通过`r2 -L`命令显示出来。插件列表的第一列若含有"d"代表该插件支持debugging，例如：

```
r_d  debug       Debug a program or pid. dbg:///bin/ls, dbg://1388 (LGPL3)
rwd  gdb         Attach to gdbserver, 'qemu -s', gdb://localhost:1234 (LGPL3)
```

针对不同的架构和操作系统如GNU/Linux， Windows， MacOS X， （Net，Free，Open）BSD和Solaris，都提供了不同的后端调试程序

进程的内存空间被视为一个平坦式文件，被调试程序中所有已映射到内存页数据以及所链接的库，既可以视为代码，也可以视为数据。

radare与debugger IO层之间的通信被打包在`system()`调用中，`system()`接受一个字符串作为参数，会将该字符串作为命令进行执行。执行的结果将会在output console中进行缓存，可以用脚本进一步处理其内容。对IO系统的访问是通过`=!`命令完成的，大多数的IO插件都提供了`=!?`或`=!help`用于获取帮助信息，例如：

```
$ r2 -d /bin/ls
...
[0x7fc15afa3cc0]> =!help
Usage: =!cmd args
 =!ptrace   - use ptrace io
 =!mem      - use /proc/pid/mem io if possible
 =!pid      - show targeted pid
 =!pid <#>  - select new pid
```

总之，debugger的命令是适用于多种操作系统和架构的。尽管radare想在不同架构和不同操作系统上构筑相同的功能，有些事情还是需要单独进行处理。这些事情包括shellcode注入和异常处理，例如在MIPS架构上，其硬件层面并不支持单步调试这一功能，因此在这种情况下，radare2通过使用综合的代码分析以及软件断点，提供了一个MIPS上单步调试的实现。

使用`d?`获取debugger的基本帮助：

```
Usage: d   # Debug commands
| db[?]                    Breakpoints commands
| dbt[?]                   Display backtrace based on dbg.btdepth and dbg.btalgo
| dc[?]                    Continue execution
| dd[?]                    File descriptors (!fd in r1)
| de[-sc] [perm] [rm] [e]  Debug with ESIL (see de?)
| dg <file>                Generate a core-file (WIP)
| dH [handler]             Transplant process to a new handler
| di[?]                    Show debugger backend information (See dh)
| dk[?]                    List, send, get, set, signal handlers of child
| dL[?]                    List or set debugger handler
| dm[?]                    Show memory maps
| do[?]                    Open process (reload, alias for 'oo')
| doo[args]                Reopen in debug mode with args (alias for 'ood')
| doof[file]               Reopen in debug mode from file (alias for 'oodf')
| doc                      Close debug session
| dp[?]                    List, attach to process or thread id
| dr[?]                    Cpu registers
| ds[?]                    Step, over, source line
| dt[?]                    Display instruction traces
| dw <pid>                 Block prompt until pid dies
| dx[?]                    Inject and run code on target process (See gs)
```

输入`oo`或`oo+`重启debugging session，至于使用哪个则取决于你想要如何打开该文件：

```
oo                 reopen current file (kill+fork in debugger)
oo+                reopen current file in read-write
```

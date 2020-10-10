# 逆向调试

Radare2拥有逆向调试器，用于将PC寄存器拨回之前的状态。（就像GDB里的reverse-next，reverse-continue）
首先需要选定为逆向调试起始记录点，代表从该时刻其保存程序状态，语法为：

```
[0x004028a0]> dts+
```

`dts`命令可以记录程序状态并进行相应管理，状态被记录之后就可以任意切换PC寄存器的值以恢复状态。
比如说现在完成了状态记录，要逆向调试一步：

```
[0x004028a0]> 2dso
[0x004028a0]> dr rip
0x004028ae
[0x004028a0]> dsb
continue until 0x004028a2
hit breakpoint at: 4028a2
[0x004028a0]> dr rip
0x004028a2
```

`dsb`命令会让debugger回溯到前一个记录的状态，然后再执行至指定的断点处。

也可以试试reverse continue：

```
[0x004028a0]> db 0x004028a2
[0x004028a0]> 10dso
[0x004028a0]> dr rip
0x004028b9
[0x004028a0]> dcb
[0x004028a0]> dr rip
0x004028a2
```

`dcb`命令回拨PC直至最新的一个断点，因此当你设置好一个新的断点时，可以用该命令将程序状态调整至该处。

可以通过`dts`查看当前已记录的所有程序状态：

```
[0x004028a0]> dts
session: 0   at:0x004028a0   ""
session: 1   at:0x004028c2   ""
```

注意：程序记录是可以随时保存的。这些记录采用差异化方式备份，仅保存与之前记录不同的内存区域，相比于完整转储节省空间。

还可以为记录增加注释：

```
[0x004028c2]> dtsC 0 program start
[0x004028c2]> dtsC 1 decryption start
[0x004028c2]> dts
session: 0   at:0x004028a0   "program start"
session: 1   at:0x004028c2   "decryption start"
```

可以为每条记录都留下一些注释，以备不时之需。
如果存在多条记录的话，`dsb`和`dcb`命令只将程序状态恢复至最新的一条记录。

这些记录都可以被导入和导出，操作方法如下：

```
[0x004028c2]> dtst records_for_test
Session saved in records_for_test.session and dump in records_for_test.dump
[0x004028c2]> dtsf records_for_test
session: 0, 0x4028a0 diffs: 0
session: 1, 0x4028c2 diffs: 0
```

在ESIL模式下同样能用逆向调试，只是程序状态的管理是使用`aets`命令完成的。

```
[0x00404870]> aets+
```

单步回溯使用 `aesb`:

```
[0x00404870]> aer rip
0x00404870
[0x00404870]> 5aeso
[0x00404870]> aer rip
0x0040487d
[0x00404870]> aesb
[0x00404870]> aer rip
0x00404879
```

除了使用radare2原生逆向调试功能外，同样支持通过gdb远程调试对具有逆向调试功能的gdbserver进行操作。
`=!dsb`和`=!dcb`即为此场景下`dsb`和`dcb`的替代命令，可以参考[GDB远程调试文档](remote_gdb.md)获取更多信息。

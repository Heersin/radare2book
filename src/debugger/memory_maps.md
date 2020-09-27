# 内存映射

对于诸多不同的逆向工程任务而言，理解和操纵已调试程序的内存映射的能力很重要。Radare2提供了一组丰富的命令来处理二进制文件中的内存映射。包括列出当前调试的二进制文件的内存映射，删除内存映射，处理已加载的库等。

首先让我们看看`dm`的帮助信息，该命令用于处理内存映射：

```
[0x55f2104cf620]> dm?
Usage: dm   # Memory maps commands
| dm                               List memory maps of target process
| dm address size                  Allocate <size> bytes at <address> (anywhere if address is -1) in child process
| dm=                              List memory maps of target process (ascii-art bars)
| dm.                              Show map name of current address
| dm*                              List memmaps in radare commands
| dm- address                      Deallocate memory map of <address>
| dmd[a] [file]                    Dump current (all) debug map region to a file (from-to.dmp) (see Sd)
| dmh[?]                           Show map of heap
| dmi [addr|libname] [symname]     List symbols of target lib
| dmi* [addr|libname] [symname]    List symbols of target lib in radare commands
| dmi.                             List closest symbol to the current address
| dmiv                             Show address of given symbol for given lib
| dmj                              List memmaps in JSON format
| dml <file>                       Load contents of file into the current map region
| dmm[?][j*]                       List modules (libraries, binaries loaded in memory)
| dmp[?] <address> <size> <perms>  Change page at <address> with <size>, protection <perms> (perm)
| dms[?] <id> <mapaddr>            Take memory snapshot
| dms- <id> <mapaddr>              Restore memory snapshot
| dmS [addr|libname] [sectname]    List sections of target lib
| dmS* [addr|libname] [sectname]   List sections of target lib in radare commands
| dmL address size                 Allocate <size> bytes at <address> and promote to huge page
```

在本章节中，我们会通过一些简单的例子介绍`dm`中最有用的一些子命令。下面我们将用一个简单`helloworld`程序进行演示，不过其中的操作都可以推广到其他二进制文件上。

首先，以debug模式打开程序：

```
$ r2 -d helloworld
Process with PID 20304 started...
= attach 20304 20304
bin.baddr 0x56136b475000
Using 0x56136b475000
asm.bits 64
[0x7f133f022fb0]>
```

> 注意我们使用了一个不带"./"的"helloworld"作为radare2的参数，radare2在未指定"./"的情况下会首先在当前目录中搜索该文件，然后才搜索$PATH中的路径。这个行为与UNIX系统的默认行为相冲突，但是对于windows用户来说该行为与系统是一致的。

让我们用`dm`输出刚打开的程序的内存映射：

```
[0x7f133f022fb0]> dm
0x0000563a0113a000 - usr   4K s r-x /tmp/helloworld /tmp/helloworld ; map.tmp_helloworld.r_x
0x0000563a0133a000 - usr   8K s rw- /tmp/helloworld /tmp/helloworld ; map.tmp_helloworld.rw
0x00007f133f022000 * usr 148K s r-x /usr/lib/ld-2.27.so /usr/lib/ld-2.27.so ; map.usr_lib_ld_2.27.so.r_x
0x00007f133f246000 - usr   8K s rw- /usr/lib/ld-2.27.so /usr/lib/ld-2.27.so ; map.usr_lib_ld_2.27.so.rw
0x00007f133f248000 - usr   4K s rw- unk0 unk0 ; map.unk0.rw
0x00007fffd25ce000 - usr 132K s rw- [stack] [stack] ; map.stack_.rw
0x00007fffd25f6000 - usr  12K s r-- [vvar] [vvar] ; map.vvar_.r
0x00007fffd25f9000 - usr   8K s r-x [vdso] [vdso] ; map.vdso_.r_x
0xffffffffff600000 - usr   4K s r-x [vsyscall] [vsyscall] ; map.vsyscall_.r_x
```

对于更喜欢以图形展现结果的用户，可以用`dm=`以ASCII风格的条柱图展示内存映射。这个特性在想查看内存映射的分布时也很方便。

如果想查看当前在内存映射中的何处可以用`dm.`:

```
[0x7f133f022fb0]> dm.
0x00007f947eed9000 # 0x00007f947eefe000 * usr   148K s r-x /usr/lib/ld-2.27.so /usr/lib/ld-2.27.so ; map.usr_lib_ld_2.27.so.r_x
```

使用`dmm`可以“列出modules（已加载到内存中的库，二进制文件）”，这也是一个很方便的命令，可以查看哪些模块被加载了。

```
[0x7fa80a19dfb0]> dmm
0x55ca23a4a000 /tmp/helloworld
0x7fa80a19d000 /usr/lib/ld-2.27.so
```
> 要说明的是`dm`子命令的输出以及`dmm`命令在不同系统上以及不同二进制文件中输出结果可能不同。

可以看到，同`helloworld`程序一起被加载的还有`ld-2.27.so`。我们在里面没有找到`libc`，这是因为radare2在`libc`加载前即中断了运行过程。使用`dcu`命令（**d**ebug **c**ontinue **u**ntil）让程序执行到入口点（radare将入口点标记为`entry0`）。

```
[0x7fa80a19dfb0]> dcu entry0
Continue until 0x55ca23a4a520 using 1 bpsize
hit breakpoint at: 55ca23a4a518
[0x55ca23a4a520]> dmm
0x55ca23a4a000 /tmp/helloworld
0x7fa809de1000 /usr/lib/libc-2.27.so
0x7fa80a19d000 /usr/lib/ld-2.27.so
```

现在可以看到`libc-2.27.so`被一起加载了，好耶！

再谈谈`libc`，二进制exploit中的一个常见工作就是找到库中一个特定符号的地址。只要有了这一信息，可以构筑比如说一个基于ROP的攻击。可以用`dmi`命令完成这一任务，比如说我们想获得加载到内存中`libc`的[`system()`](http://man7.org/linux/man-pages/man3/system.3.html)地址，就执行如下命令：

```
[0x55ca23a4a520]> dmi libc system
514 0x00000000 0x7fa809de1000  LOCAL  FILE    0 system.c
515 0x00043750 0x7fa809e24750  LOCAL  FUNC 1221 do_system
4468 0x001285a0 0x7fa809f095a0 LOCAL  FUNC  100 svcerr_systemerr
5841 0x001285a0 0x7fa809f095a0 LOCAL  FUNC  100 svcerr_systemerr
6427 0x00043d10 0x7fa809e24d10  WEAK  FUNC   45 system
7094 0x00043d10 0x7fa809e24d10 GLBAL  FUNC   45 system
7480 0x001285a0 0x7fa809f095a0 GLBAL  FUNC  100 svcerr_systemerr
```

类似`dm.`命令，使用`dmi.`可以找到距离当前地址最近的一个符号。

另一个有用的命令用于列出库文件内的节区信息，在下面的例子中我们列出了`ld-2.27.so`的节区：

```
[0x55a7ebf09520]> dmS ld-2.27
[Sections]
00 0x00000000     0 0x00000000     0 ---- ld-2.27.so.
01 0x000001c8    36 0x4652d1c8    36 -r-- ld-2.27.so..note.gnu.build_id
02 0x000001f0   352 0x4652d1f0   352 -r-- ld-2.27.so..hash
03 0x00000350   412 0x4652d350   412 -r-- ld-2.27.so..gnu.hash
04 0x000004f0   816 0x4652d4f0   816 -r-- ld-2.27.so..dynsym
05 0x00000820   548 0x4652d820   548 -r-- ld-2.27.so..dynstr
06 0x00000a44    68 0x4652da44    68 -r-- ld-2.27.so..gnu.version
07 0x00000a88   164 0x4652da88   164 -r-- ld-2.27.so..gnu.version_d
08 0x00000b30  1152 0x4652db30  1152 -r-- ld-2.27.so..rela.dyn
09 0x00000fb0 11497 0x4652dfb0 11497 -r-x ld-2.27.so..text
10 0x0001d0e0 17760 0x4654a0e0 17760 -r-- ld-2.27.so..rodata
11 0x00021640  1716 0x4654e640  1716 -r-- ld-2.27.so..eh_frame_hdr
12 0x00021cf8  9876 0x4654ecf8  9876 -r-- ld-2.27.so..eh_frame
13 0x00024660  2020 0x46751660  2020 -rw- ld-2.27.so..data.rel.ro
14 0x00024e48   336 0x46751e48   336 -rw- ld-2.27.so..dynamic
15 0x00024f98    96 0x46751f98    96 -rw- ld-2.27.so..got
16 0x00025000  3960 0x46752000  3960 -rw- ld-2.27.so..data
17 0x00025f78     0 0x46752f80   376 -rw- ld-2.27.so..bss
18 0x00025f78    17 0x00000000    17 ---- ld-2.27.so..comment
19 0x00025fa0    63 0x00000000    63 ---- ld-2.27.so..gnu.warning.llseek
20 0x00025fe0 13272 0x00000000 13272 ---- ld-2.27.so..symtab
21 0x000293b8  7101 0x00000000  7101 ---- ld-2.27.so..strtab
22 0x0002af75   215 0x00000000   215 ---- ld-2.27.so..shstrtab
```

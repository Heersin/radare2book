## The Framework

Radare2项目是一组小型命令行工具，这些工具可以被独立使用，也可以结合着使用。本章节会让您快速了解它们，您可以本书末尾查看每种工具的具体用法。

### radare2

这是整个框架的核心工具，它具有debugger和Hexeditor的核心功能，使您能够像打开普通的文件一样，打开许多输入/输出源，包括磁盘、网络连接、内核驱动和处于调试中的进程等。

它实现了一个高级的命令行界面，可用于在文件内部活动和浏览，分析数据，反编译，打补丁，比较数据，搜索，替换和可视化。您可以用多种编程语言编写radare2的脚本，包括Python, Ruby, JavaScript, Lua, 和 Perl。

### rabin2

该程序用于从可执行文件中提取信息，例如ELF, PE, Java CLASS, Mach-O, 以及各种r2引擎所支持的二进制文件格式。r2核心使用rabin2获取数据，例如导出的符号，导入的函数和DLL等，文件的元信息，交叉引用（xrefs），依赖库以及文件区段信息。

### rasm2

针对多种架构的命令行编译器和反编译器（支持Intel x86和x86-64, MIPS, ARM, PowerPC, Java等等）。

#### 示例
```
$ rasm2 -a java 'nop'
00
```
```
$ rasm2 -a x86 -d '90'
nop
```
```
$ rasm2 -a x86 -b 32 'mov eax, 33'
b821000000
```
```
$ echo 'push eax;nop;nop' | rasm2 -f -
509090
```

### rahash2

基于块的哈希工具，无论是一段短小的字符串，还是巨大的磁盘文件，rahash2都支持多种算法进行hash，包括MD4， MD5, CRC16, CRC32, SHA1， SHA256以及其他种种。
rahash2可用于完整性检查，或是追踪大文件、内存转储或磁盘上的变化。

### 示例
```
$ rahash2 file
file: 0x00000000-0x00000007 sha256: 887cfbd0d44aaff69f7bdbedebd282ec96191cce9d7fa7336298a18efc3c7a5a
```
```
$ rahash2 -a md5 file
file: 0x00000000-0x00000007 md5: d1833805515fc34b46c2b9de553f599d
```
### radiff2

一个二进制差异比较工具， 实现了多种算法。它支持二进制文件的字节级比较以及差分比较， 以及支持代码差异比较，用于发现在Radare2分析工作中代码块发生的更改。

### rafind2

用于在文件中根据pattern找到对应的字节串。

### ragg2

r_egg的前端程序，ragg2可以将高级语言编写的程序编译为微型二进制程序。支持x86,x86-64和ARM架构。

#### 示例

```
$ cat hi.r
/* hello world in r_egg */
write@syscall(4); //x64 write@syscall(1);
exit@syscall(1); //x64 exit@syscall(60);

main@global(128) {
 .var0 = "hi!\n";
 write(1,.var0, 4);
 exit(0);
}
$ ragg2 -O -F hi.r
$ ./hi
hi!

$ cat hi.c
main@global(0,6) {
 write(1, "Hello0", 6);
 exit(0);
}
$ ragg2 hi.c
$ ./hi.c.bin
Hello
```

### rarun2

一个程序启动器，可使得程序以不同的环境变量和不同的参数启动，还支持以不同权限，在不同目录下启动，并可覆写程序默认的文件描述符。这些特性在以下场景中很有用：

* Solving crackmes
* Fuzzing
* Test suites

#### rarun2 script的一个范例
```
$ cat foo.rr2
#!/usr/bin/rarun2
program=./pp400
arg0=10
stdin=foo.txt
chdir=/tmp
#chroot=.
./foo.rr2
```

#### 将程序与socket绑定
```
$ nc -l 9999
$ rarun2 program=/bin/ls connect=localhost:9999
```

#### 启动调试程序， 并将stdio重定向至另一个终端

1 - open a new terminal and type 'tty' to get a terminal name:

```
$ tty ; clear ; sleep 999999
/dev/ttyS010
```

2 - Create a new file containing the following rarun2 profile named foo.rr2:
```
#!/usr/bin/rarun2
program=/bin/ls
stdio=/dev/ttys010
```

3 - Launch the following radare2 command:
```
r2 -r foo.rr2 -d /bin/ls
```

### rax2

一个小巧简约的数学表达式求解器，主要用于浮点数、十六进制的转换，十六进制串与ASCII的转换，八进制与整数之间的转换等，且支持设定字节序。在没有提供任何参数时可以用作一个交互式shell。

#### 示例

```
$ rax2 1337
0x539

$ rax2 0x400000
4194304

$ rax2 -b 01111001
y

$ rax2 -S radare2
72616461726532

$ rax2 -s 617765736f6d65
awesome
```

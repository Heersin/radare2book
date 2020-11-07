## Comparing Bytes

对于大多数逆向工作而言，若要找到两个二进制文件的不同之处（比如哪些字节有变化），或是图形化地显示二者之间的差别，可以使用radiff2完成这些工作:

```
$ radiff2 -h
```

而在r2中，radiff2所提供的这些功能放在`c`命令下。

`c` ("compare"的缩写)类命令可以对不同源文件中的字节进行比较。该命令可接受多种格式的输入，然后与当前位置上的数据进行比较。

```
[0x00404888]> c?
Usage: c[?dfx] [argument]   # Compare
| c [string]               Compare a plain with escaped chars string
| c* [string]              Same as above, but printing r2 commands instead
| c1 [addr]                Compare 8 bits from current offset
| c2 [value]               Compare a word from a math expression
| c4 [value]               Compare a doubleword from a math expression
| c8 [value]               Compare a quadword from a math expression
| cat [file]               Show contents of file (see pwd, ls)
| cc [at]                  Compares in two hexdump columns of block size
| ccc [at]                 Same as above, but only showing different lines
| ccd [at]                 Compares in two disasm columns of block size
| ccdd [at]                Compares decompiler output (e cmd.pdc=pdg|pdd)
| cf [file]                Compare contents of file at current seek
| cg[?] [o] [file]         Graphdiff current file and [file]
| cu[?] [addr] @at         Compare memory hexdumps of $$ and dst in unified diff
| cud [addr] @at           Unified diff disasm from $$ and given address
| cv[1248] [hexpairs] @at  Compare 1,2,4,8-byte (silent return in $?)
| cV[1248] [addr] @at      Compare 1,2,4,8-byte address contents (silent, return in $?)
| cw[?] [us?] [...]        Compare memory watchers
| cx [hexpair]             Compare hexpair string (use '.' as nibble wildcard)
| cx* [hexpair]            Compare hexpair string (output r2 commands)
| cX [addr]                Like 'cc' but using hexdiff output
| cd [dir]                 chdir
| cl|cls|clear             Clear screen, (clear0 to goto 0, 0 only)
```

使用`cx`命令可以将当前位置上的内存数据与给定一串数据进行比较：
```
[0x08048000]> p8 4
7f 45 4c 46

[0x08048000]> cx 7f 45 90 46
Compare 3/4 equal bytes
0x00000002 (byte=03)   90 ' '  ->  4c 'L'
[0x08048000]>
```

`c`中的一个子命令`cc`（代表"compare code"）可将字节序列与内存中的序列进行比较：

```
[0x4A13B8C0]> cc 0x39e8e089 @ 0x4A13B8C0
```

若要比较两个函数则指定他们的名字:

```
[0x08049A80]> cc sym.main2 @ sym.main
```

`c8` 会将当前地址（底下的例子中是0x0）上的四个字与给定的表达式的计算结果相比较:
```
[0x00000000]> c8 4

Compare 1/8 equal bytes (0%)
0x00000000 (byte=01)   7f ' '  ->  04 ' '
0x00000001 (byte=02)   45 'E'  ->  00 ' '
0x00000002 (byte=03)   4c 'L'  ->  00 ' '
```

数字参数可以是合法的表达式（允许在其中使用flag名字等）

```
[0x00000000]> cx 7f469046

Compare 2/4 equal bytes
0x00000001 (byte=02)   45 'E'  ->  46 'F'
0x00000002 (byte=03)   4c 'L'  ->  90 ' '
```

可以用下面的命令将当前块与之前dump下来的文件相比较:

```
r2 /bin/true
[0x08049A80]> s 0
[0x08048000]> cf /bin/true
Compare 512/512 equal bytes
```

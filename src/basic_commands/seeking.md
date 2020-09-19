## 定位（跳转）

为了移动我们正在检查的文件，我们将需要使用`s`命令更改偏移量。该参数是一个数学表达式，可以包含标志名称，括号，加法，减法，乘积，以及使用方括号获取到的对应内存地址上的数据。

下面是一些示例命令，注意观察偏移量如何变化。：
```
[0x00000000]> s 0x10
[0x00000010]> s+4
[0x00000014]> s-
[0x00000010]> s+
[0x00000014]>
```

第一行将当前偏移量移至地址0x10。

第二个是相对当前位置，向后偏移4个字节。

最后的2个命令分别是撤消前面的操作和重做最后一个查找操作。

我们不仅可以使用数字，还可以使用复杂的表达式或基本的算术运算来表示要查找的地址。请查看？$？帮助消息，里面描述了可以在表达式中使用的内部变量， 例如下面的表达式与s + 4是相同的：

```
[0x00000000]> s $$+4
```

在调试器中（或在仿真时），我们还可以将寄存器名称用作引用。通过`.dr*`命令将寄存器名称作为标志加载，该命令会在后台运行。

```
[0x00000000]> s rsp+0x40
```

下面是关于`s`命令的帮助信息，之后将详细介绍它。

```
[0x00000000]> s?
Usage: s    # Help for the seek commands. See ?$? to see all variables
| s                 Print current address
| s.hexoff          Seek honoring a base from core->offset
| s:pad             Print current address with N padded zeros (defaults to 8)
| s addr            Seek to address
| s-                Undo seek
| s-*               Reset undo seek history
| s- n              Seek n bytes backward
| s--[n]            Seek blocksize bytes backward (/=n)
| s+                Redo seek
| s+ n              Seek n bytes forward
| s++[n]            Seek blocksize bytes forward (/=n)
| s[j*=!]           List undo seek history (JSON, =list, *r2, !=names, s==)
| s/ DATA           Search for next occurrence of 'DATA'
| s/x 9091          Search for next occurrence of \x90\x91
| sa [[+-]a] [asz]  Seek asz (or bsize) aligned to addr
| sb                Seek aligned to bb start
| sC[?] string      Seek to comment matching given string
| sf                Seek to next function (f->addr+f->size)
| sf function       Seek to address of specified function
| sf.               Seek to the beginning of current function
| sg/sG             Seek begin (sg) or end (sG) of section or file
| sl[?] [+-]line    Seek to line
| sn/sp ([nkey])    Seek to next/prev location, as specified by scr.nkey
| so [N]            Seek to N next opcode(s)
| sr pc             Seek to register
| ss                Seek silently (without adding an entry to the seek history)

> 3s++        ; 3 times block-seeking
> s 10+0x80   ; seek at 0x80+10
```

如果要检查数学表达式的结果，可以使用`？`命令对其求值。只需将表达式作为参数传递。结果可以以十六进制，十进制，八进制或二进制格式显示。

```
> ? 0x100+200
0x1C8 ; 456d ; 710o ; 1100 1000
```

“？”子命令能以一种特定格式（以10为基数，以16为底数...）显示输出。参见`？v`和`？vi`。

在可视化模式下，您可以在搜索历史记录中按`u`（撤消）或`U`（重做）以返回到上一个位置或前进到下一个位置。

## 打开文件

我们使用以Linux ELF格式编译的简单`hello_world.c`作为测试文件，编译之后，让我们用radare2打开它：
```
$ r2 hello_world
```

现在命令行提示符已经成功显示:

```
[0x00400410]>
```

是时候向更深处进击了。

## 定位到任意的地址

所有接受地址作为命令参数的查找命令都可以使用任意进制，例如十六进制，八进制，二进制或十进制。定位到地址0x0的一个简便命令就是`0x0`

```
[0x00400410]> s 0x0
[0x00000000]>
```

显示当前地址:
```
[0x00000000]> s
0x0
[0x00000000]>
```

显示当前地址的另一个方法是使用：`？v $$`。

可以像下面这样向地址高位跳转N个位置， 有没有空格都无所谓:

```
[0x00000000]> s+ 128
[0x00000080]>
```

撤销前两个seek操作，回到最初地址：

```
[0x00000080]> s-
[0x00000000]> s-
[0x00400410]>
```

现在，我们回到了_0x00400410_， 下面的命令可以展示我们的seek历史。

```
[0x00400410]> s*
f undo_3 @ 0x400410
f undo_2 @ 0x40041a
f undo_1 @ 0x400410
f undo_0 @ 0x400411
# Current undo/redo position.
f redo_0 @ 0x4005b4
```


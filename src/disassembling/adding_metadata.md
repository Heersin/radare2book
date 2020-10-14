## 为反汇编添加元数据
二进制逆向工作中一个经典的任务就是为分析结果提供有用而关键的注解。
Radare提供了多种方式存取该元信息。

遵循UNIX原则，我们采用脚本语言使用`odjdump`，`otool`以及其他小工具结合起来提取二进制文件内的信息，并将这些信息导入radare2中。例如[radare2ida](https://github.com/radareorg/radare2ida)中的`idc2r.py`，可以用`idc2r.py file.idc > file.r2`调用之，其读取IDA pro导出的IDC文件，然后创建一个包含相同注释、函数名和其它数据的r2脚本把这些数据引入r2中。可以在radare2内用`.`命令执行该r2脚本:
```
[0x00000000]> . file.r2
```

`.`命令用于调用外部的radare命令输入源, 包括文件和程序的输出内容。例如想省略中间文件的生成并直接导入脚本，可以使用如下的命令组合：
```
[0x00000000]> .!idc2r.py < file.idc
```

请注意，导入IDA Pro中的IDC dump的元数据是一个已被废弃的机制，在未来的版本中很可能不再生效。更推荐的做法是使用基于`ida2r2.py`的[python-idb](https://github.com/williballenthin/python-idb)直接打开IDB文件，该做法不需要系统上安装IDA pro。

`C`命令用于管理注释和数据转换的部分，你可以将一个范围内的字节作为代码解释，作为二进制数据解释或者作为字符串进行解释。同样也支持指定的标识位置外执行外部代码，以从外部文件或数据库中获取一些元信息，比如注释之类的。

radare包含大量不同的元数据操作命令，下面是他们的简要介绍：

```
[0x00404cc0]> C?
| Usage: C[-LCvsdfm*?][*?] [...]   # Metadata management
| C                                              list meta info in human friendly form
| C*                                             list meta info in r2 commands
| C*.                                            list meta info of current offset in r2 commands
| C- [len] [[@]addr]                             delete metadata at given address range
| C.                                             list meta info of current offset in human friendly form
| CC! [@addr]                                    edit comment with $EDITOR
| CC[?] [-] [comment-text] [@addr]               add/remove comment
| CC.[addr]                                      show comment in current address
| CCa[-at]|[at] [text] [@addr]                   add/remove comment at given address
| CCu [comment-text] [@addr]                     add unique comment
| CF[sz] [fcn-sign..] [@addr]                    function signature
| CL[-][*] [file:line] [addr]                    show or add 'code line' information (bininfo)
| CS[-][space]                                   manage meta-spaces to filter comments, etc..
| C[Cthsdmf]                                     list comments/types/hidden/strings/data/magic/formatted in human friendly form
| C[Cthsdmf]*                                    list comments/types/hidden/strings/data/magic/formatted in r2 commands
| Cd[-] [size] [repeat] [@addr]                  hexdump data array (Cd 4 10 == dword [10])
| Cd. [@addr]                                    show size of data at current address
| Cf[?][-] [sz] [0|cnt][fmt] [a0 a1...] [@addr]  format memory (see pf?)
| Ch[-] [size] [@addr]                           hide data
| Cm[-] [sz] [fmt..] [@addr]                     magic parse (see pm?)
| Cs[?] [-] [size] [@addr]                       add string
| Ct[?] [-] [comment-text] [@addr]               add/remove type analysis comment
| Ct.[@addr]                                     show comment at current or specified address
| Cv[bsr][?]                                     add comments to args
| Cz[@addr]                                      add string (see Cs?)
```

在指定的行/地址加入一个注释，只需使用`Ca`命令：

```
[0x00000000]> CCa 0x0000002 this guy seems legit
[0x00000000]> pd 2
0x00000000    0000         add [rax], al
;      this guy seems legit
0x00000002    0000         add [rax], al
```

`C?`命令族可以用于将范围内的数据标记为几种类型之一，基本的三种类型为：code(通过asm.arch完成反汇编)，data（一个包含数据元素的数组）或string。使用`Cs`命令可以定义一个字符串，使用`Cd`命令可以定义一个数组，使用`Cf`命令则可以定义一些更复杂的数据结构，比如struct。

对数据类型的注解通常在可视化模式下更容易完成，使用"d"键即可进行注解，其是"data type change"的缩写。首先用光标选择一个字节范围（按下`c`键进入光标模式，在里面通过HJKL键进行选择），之后按下'd'键可以获取菜单，菜单中包含允许进行的操作或可能的类型。例如想要将一个范围内的字节标记为字符串时，选择该菜单中的`s`选项。当然也可以在r2 shell下使用`Cs`命令达到同样的效果。

```
[0x00000000]> f string_foo @ 0x800
[0x00000000]> Cs 10 @ string_foo
```

`Cf`命令用于定义一个内存格式化字符串（与`pf`命令使用的格式相同）。这里有一个例子：

```
[0x7fd9f13ae630]> Cf 16 2xi foo bar
[0x7fd9f13ae630]> pd
;-- rip:
0x7fd9f13ae630 format 2xi foo bar {
0x7fd9f13ae630 [0] {
 foo : 0x7fd9f13ae630 = 0xe8e78948
 bar : 0x7fd9f13ae634 = 14696
}
0x7fd9f13ae638 [1] {
 foo : 0x7fd9f13ae638 = 0x8bc48949
 bar : 0x7fd9f13ae63c = 571928325
}
} 16
0x7fd9f13ae633    e868390000   call 0x7fd9f13b1fa0
0x7fd9f13ae638    4989c4       mov r12, rax
```

传递给`Cf`的`[sz]`参数用于定义反汇编中该结构体占据的字节数，该参数与格式化字符串中所定义的字节数是完全独立的。听起来可能有一点迷惑，不过这个设定有几个用途，例如，你可能想在反汇编中看到经过格式化规整后的结构体，同时仍然将结构体中对应位置显示为偏移量和原始的字节。有时，你可能只识别出一个大结构体中的几个成员，又或者只对其中的某个成员感兴趣，那么你可以使用格式化字符串和'skip'跳过，让r2仅仅显示这些成员区域，同时反汇编依旧可以不受阻碍地继续下去，因为其使用`sz`命令所指定的数值作为结构体的大小。

通过使用`Cf`，我们能够使用简单的命令定义复杂的结构体。查看`pf?`的输出信息获取更多帮助。
记住，所有这些`C`命令都可以在可视化模式下通过`d`（data conversion）键使用。还要注意的是，和[`t`](../analysis/types.md)命令不同，`Cf`并不改变分析结果，它仅仅用于优化展示效果。

有时候仅仅添加一行文字注释可能无法满足需要，radare2允许创建一个指向文本文件的链接。使用`CC,`命令或者在可视化模式按下`,`键，之后会打开一个`$EDITOR`创建新文件，如果文件名已经存在了，那么将会创建一个指向文件的链接。之后，它在反汇编窗口中会显示这样的注释：

```
[0x00003af7 11% 290 /bin/ls]> pd $r @ main+55 # 0x3af7
│0x00003af7  call sym.imp.setlocale        ;[1] ; ,(locale-help.txt) ; char *setlocale(int category, const char *locale)
│0x00003afc  lea rsi, str.usr_share_locale ; 0x179cc ; "/usr/share/locale"
│0x00003b03  lea rdi, [0x000179b2]         ; "coreutils"
│0x00003b0a  call sym.imp.bindtextdomain   ;[2] ; char *bindtextdomain(char *domainname, char *dirname)
```

注意，`,(locale-help.txt)`出现在了注释中，如果在可视化模式下再次按下`,`就会打开该文件。通过该机制就可以在反汇编窗口中的注释位置上创建一个长描述，外链到数据表或者相关的文章。

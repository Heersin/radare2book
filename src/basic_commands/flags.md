## Flags

Radare中的flag类似书签， 将一个名称与文件中的偏移量进行绑定。Flags可以划分为多个标志空间（flags space）， 每个标志空间都是一组flags的命名空间， 将具有相似特征或相似类型的flags集中于组内。常见的标志空间有：sections， registers， symbols。

创建一个flag:

```
[0x4A13B8C0]> f flag_name @ offset
```

可以通过后接`-`移除一个flag，大多数命令中的`-`前缀都起到删除的作用。 

```
[0x4A13B8C0]> f-flag_name
```

在标志空间直接切换或创建标志空间使用`fs`命令：

```
[0x00005310]> fs?
|Usage: fs [*] [+-][flagspace|addr] # Manage flagspaces
| fs            display flagspaces
| fs*           display flagspaces as r2 commands
| fsj           display flagspaces in JSON
| fs *          select all flagspaces
| fs flagspace  select flagspace or create if it doesn't exist
| fs-flagspace  remove flagspace
| fs-*          remove all flagspaces
| fs+foo        push previous flagspace and set
| fs-           pop to the previous flagspace
| fs-.          remove the current flagspace
| fsq           list flagspaces in quiet mode
| fsm [addr]    move flags at given address to the current flagspace
| fss           display flagspaces stack
| fss*          display flagspaces stack in r2 commands
| fssj          display flagspaces stack in JSON
| fsr newname   rename selected flagspace
[0x00005310]> fs
0  439 * strings
1   17 * symbols
2   54 * sections
3   20 * segments
4  115 * relocs
5  109 * imports
[0x00005310]>
```
下面是一些例子：

```
[0x4A13B8C0]> fs symbols ; select only flags in symbols flagspace
[0x4A13B8C0]> f          ; list only flags in symbols flagspace
[0x4A13B8C0]> fs *       ; select all flagspaces
[0x4A13B8C0]> f myflag   ; create a new flag called 'myflag'
[0x4A13B8C0]> f-myflag  ; delete the flag called 'myflag'
```

通过`fr`可以重命名flag。

### 局部flag

出于寻址的考虑，每个flag都应该是独一无二的。但是我们也常常需要一些简单而普遍的flag名字，比如在函数内部，类似`loop`或`return`这样的标记。为此我们可以使用局部flag，它们仅与它们所在处的函数相关联，可以用`f.`添加局部flag：

```
[0x00003a04]> pd 10
│      0x00003a04      48c705c9cc21.  mov qword [0x002206d8], 0xffffffffffffffff ;
[0x2206d8:8]=0
│      0x00003a0f      c60522cc2100.  mov byte [0x00220638], 0     ; [0x220638:1]=0
│      0x00003a16      83f802         cmp eax, 2
│  .─< 0x00003a19      0f84880d0000   je 0x47a7
│  │   0x00003a1f      83f803         cmp eax, 3
│ .──< 0x00003a22      740e           je 0x3a32
│ ││   0x00003a24      83e801         sub eax, 1
│.───< 0x00003a27      0f84ed080000   je 0x431a
││││   0x00003a2d      e8fef8ffff     call sym.imp.abort           ; void abort(void)
││││   ; CODE XREF from main (0x3a22)
││╰──> 0x00003a32      be07000000     mov esi, 7
[0x00003a04]> f. localflag @ 0x3a32
[0x00003a04]> f.
0x00003a32 localflag   [main + 210]
[0x00003a04]> pd 10
│      0x00003a04      48c705c9cc21.  mov qword [0x002206d8], 0xffffffffffffffff ;
[0x2206d8:8]=0
│      0x00003a0f      c60522cc2100.  mov byte [0x00220638], 0     ; [0x220638:1]=0
│      0x00003a16      83f802         cmp eax, 2
│  .─< 0x00003a19      0f84880d0000   je 0x47a7
│  │   0x00003a1f      83f803         cmp eax, 3
│ .──< 0x00003a22      740e           je 0x3a32                    ; main.localflag
│ ││   0x00003a24      83e801         sub eax, 1
│.───< 0x00003a27      0f84ed080000   je 0x431a
││││   0x00003a2d      e8fef8ffff     call sym.imp.abort           ; void abort(void)
││││   ; CODE XREF from main (0x3a22)
││`──>  .localflag:
││││   ; CODE XREF from main (0x3a22)
││`──> 0x00003a32      be07000000     mov esi, 7
[0x00003a04]>
```

### Flag Zones

radare2提供了flag zone的功能，使您可以在滚动条上标记不同的偏移量，从而更方便地浏览大型二进制文件。可以用如下命令在当前偏移量下设置flag zone。

```
[0x00003a04]> fz flag-zone-name
```

设置`scr.scrollbar=1`然后进入可视化模式， 可以在窗口右端的滚动条上看到flag zone。参阅 `fz?` 获取更多相关信息。
# 反汇编界面

## 导航

在反汇编界面中可以用方向键或`hjkl`进行移动，`g`可以用于跳转到flag所在位置或者某个偏移位置上，当出现`[offset]>`提示符时就可进行输入了。
在本例中按下`1`将会跟随`sym.imp.__libc_start_main`，也就是跳转到该符号所在的偏移量上。

```
0x00404894      e857dcffff     call sym.imp.__libc_start_main ;[1]
```

可以用`u`返回到上一个位置，`U`则可以用于重现刚刚的跳转操作。

## `d` as define

`d`可以用于改变当前块的数据类型，支持多种基本的类型和结构，若需要更复杂的数据结构可以使用`pf`模板。

```
d → ...
0x004048f7      48c1e83f       shr rax, 0x3f
d → b
0x004048f7 .byte 0x48
d → B
0x004048f7 .word 0xc148
d → d
0x004048f7 hex length=165 delta=0
0x004048f7  48c1 e83f 4801 c648 d1fe 7415 b800 0000
...
```

可以改变radare2在反汇编中显示的数据类型以获得更佳的可读性，默认情况下大部分的数值都是以16进制表示的。有时你可能需要以十进制、二进制显示，或者将其作为一个自定义的常量显示。可以使用`d`键，然后按下`i`键选择进制，这个操作与`ahi`是等效的:

```
d → i → ...
0x004048f7      48c1e83f       shr rax, 0x3f
d → i →  10
0x004048f7      48c1e83f       shr rax, 63
d → i →  2
0x004048f7      48c1e83f       shr rax, '?'
```

### 使用光标进行插入/修补...

要记住，若想编辑加载的文件需要以`-w`选项启动radare2，否则该文件处于只读模式下。

按下小写的`c`键可以切换到光标模式，在该模式下选中的字节(或者说字节范围)会以高亮显示。

![Cursor at 0x00404896](cursor.png)

光标通常用于选择一个范围内的字节，或者仅仅是标记某个字节。可以跳转到目标地址上，按下`f`键然后输入flag名字在该位置上创建一个flag。
如果以写模式(`-w`选项或者`o+`命令)打开文件，还可以覆写光标选中的字节。首先选中一个范围内的字节（按住SHIFT然后使用HJKL移动），然后按下`i`键，输入十六进制值，则会用该值覆盖选中的范围，举个例子：
```
<select 10 bytes in visual mode using SHIFT+HJKL>
<press 'i' and then enter '12 34'>
```
则选中的十个字节会变为"12 34 12 34 12 ...".


可视化汇编器能在输入新指令进行patch时可以提供一个实时的预览。若要开启该功能则跳转到想要patch的地方，或者将光标移到该处，然后按下`A`，多条指令需要用`;`将各条指令分割开。

## 交叉引用（XREF）

radare2在分析时发现的XREF都会显示在反汇编界面里，并有一个`XREF`标签：

```
; DATA XREF from 0x00402e0e (unk)
str.David_MacKenzie:
```

按下`x`可以找到字符串的引用位置。若想跳转到数据的引用处则在键盘上按下数据对应的数字[0-9]。(该功能类似`axt`)

`X`代表逆操作，即`axf`。

## 显示参数

可以将配置变量设置为true启用该功能 `e dbg.funcarg = true`

![funcarg](funcarg.png)

## 添加注释

按下`;`可以添加注释。

## 调用其它命令

想快速调用其它命令可以按`:`.

## 搜索

`/`: 在当前输出中高亮搜索的字符串
`:cmd` 通过这个可以使用"/?"下的命令进行更精确的搜索。

## 控制界面

### "用户友好型"界面

可以通过`??`快捷键进入用户友好型界面中（特别是对于新手来说），该面板相当于一个备忘录，方便你查找命令并运行。对于老手来说，其它专用的界面可能更好用一些。

### "flag/注释/函数/.. 界面"

可以用`_`键显示该面板，该界面中会列出定义的所有flag，并允许你直接跳转到对应位置。可以通过键盘输入快速过滤出匹配的flag。

使用`^C`可以关闭输入模式，不输入任何字符串就按下`backspace`时将会退出面板。

## 配置反汇编视图的样式

反汇编的显示风格和样式取决于"asm.*"变量，可以用`e`命令对这些变量进行修改，也可以通过可视化模式中的变量编辑器对这些变量进行修改。

## 可视化变量编辑器

该面板可以通过在可视化模式下按`e`键进入，在编辑器里可以很容易地查看并修改radare2变量。例如我们想修改反汇编的输出结果，首先在列表里选择`asm`，然后浏览并选择想修改的变量，按下`Enter`键对该变量进行修改。如果该变量是个bool值，那么其将会显示一个下拉条，其他的则会提示输入一个新值。

![First Select asm](select_asm.png)

切换到伪汇编输出：

![Pseudo disassembly disabled](pseudo_disable.png)


![Pseudo disassembly enabled](pseudo_enable.png)

底下是一些与反汇编相关的变量。

## 例子

#### asm.arch: 设置架构 && asm.bits: 设置汇编器中字的大小

可以用`e asm.arch=?`列出所有架构。

```
e asm.arch = dalvik
0x00404870      31ed4989       cmp-long v237, v73, v137
0x00404874      d15e4889       rsub-int v14, v5, 0x8948
0x00404878      e24883e4       ushr-int/lit8 v72, v131, 0xe4
0x0040487c      f0505449c7c0   +invoke-object-init-range {}, method+18772 ;[0]
0x00404882      90244100       add-int v36, v65, v0
```

```
e asm.bits = 16
0000:4870      31ed           xor bp, bp
0000:4872      49             dec cx
0000:4873      89d1           mov cx, dx
0000:4875      5e             pop si
0000:4876      48             dec ax
0000:4877      89e2           mov dx, sp
```
这个设置位数的命令还可以通过可视化模式中的`&`完成。


#### asm.pseudo: 启用伪代码格式

```
e asm.pseudo = true
0x00404870      31ed           ebp = 0
0x00404872      4989d1         r9 = rdx
0x00404875      5e             pop rsi
0x00404876      4889e2         rdx = rsp
0x00404879      4883e4f0       rsp &= 0xfffffffffffffff0
```

#### asm.syntax: 选择汇编语言的语法 (intel, att, masm...)

```
e asm.syntax = att
0x00404870      31ed           xor %ebp, %ebp
0x00404872      4989d1         mov %rdx, %r9
0x00404875      5e             pop %rsi
0x00404876      4889e2         mov %rsp, %rdx
0x00404879      4883e4f0       and $0xfffffffffffffff0, %rsp
```

#### asm.describe: S显示操作码的描述

```
e asm.describe = true
0x00404870  xor ebp, ebp   ; logical exclusive or
0x00404872  mov r9, rdx    ; moves data from src to dst
0x00404875  pop rsi        ; pops last element of stack and stores the result in argument
0x00404876  mov rdx, rsp   ; moves data from src to dst
0x00404879  and rsp, -0xf  ; binary and operation between src and dst, stores result on dst
```



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

## The HUDS

### The "UserFriendly HUD"

The "UserFriendly HUD" can be accessed using the `??` key-combination. This HUD acts as an interactive Cheat Sheet that one can use to more easily find and execute commands. This HUD is particularly useful for new-comers. For experienced users, the other HUDS which are more activity-specific may be more useful.

### The "flag/comment/functions/.. HUD"

This HUD can be displayed using the `_` key, it shows a list of all the flags defined and lets you jump to them. Using the keyboard you can quickly filter the list down to a flag that contains a specific pattern.

Hud input mode can be closed using ^C. It will also exit when backspace is pressed when the user input string is empty.

## Tweaking the Disassembly

The disassembly's look-and-feel is controlled using the "asm.* configuration keys, which can be
changed using the `e` command. All configuration keys can also be edited through the Visual Configuration Editor.

## Visual Configuration Editor

This HUD can be accessed using the `e` key in visual mode. The editor allows you to easily examine and change radare2's configuration. For example, if you want to change something about the disassembly display, select `asm` from the list, navigate to the item you wish to modify it, then select it by hitting `Enter`.
If the item is a boolean variable, it will toggle, otherwise you will be prompted to provide a new value.


![First Select asm](select_asm.png)


Example switch to pseudo disassembly:

![Pseudo disassembly disabled](pseudo_disable.png)


![Pseudo disassembly enabled](pseudo_enable.png)

Following are some example of eval variable related to disassembly.

## Examples

#### asm.arch: Change Architecture && asm.bits: Word size in bits at assembler

You can view the list of all arch using `e asm.arch=?`

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
This latest operation can also be done using `&` in Visual mode.


#### asm.pseudo: Enable pseudo syntax

```
e asm.pseudo = true
0x00404870      31ed           ebp = 0
0x00404872      4989d1         r9 = rdx
0x00404875      5e             pop rsi
0x00404876      4889e2         rdx = rsp
0x00404879      4883e4f0       rsp &= 0xfffffffffffffff0
```

#### asm.syntax: Select assembly syntax (intel, att, masm...)

```
e asm.syntax = att
0x00404870      31ed           xor %ebp, %ebp
0x00404872      4989d1         mov %rdx, %r9
0x00404875      5e             pop %rsi
0x00404876      4889e2         mov %rsp, %rdx
0x00404879      4883e4f0       and $0xfffffffffffffff0, %rsp
```

#### asm.describe: Show opcode description

```
e asm.describe = true
0x00404870  xor ebp, ebp   ; logical exclusive or
0x00404872  mov r9, rdx    ; moves data from src to dst
0x00404875  pop rsi        ; pops last element of stack and stores the result in argument
0x00404876  mov rdx, rsp   ; moves data from src to dst
0x00404879  and rsp, -0xf  ; binary and operation between src and dst, stores result on dst
```



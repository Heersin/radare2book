## 配置变量

下面列出了最常用的一些配置变量，可以用`e`命令（不加参数）列出所有变量。例如，若要查看定义于"cfg"命名空间下的所有变量，就使用`e cfg.`命令（注意结尾有个点），还可以使用`e? cfg.`列出这些命令的帮助信息。

`e??`命令则能获取radare2中所有配置项的帮助信息，由于输出信息非常之多，最好使用`～`命令过滤出你所需要的部分:

![e??~color](../img/configuration/e--color.png)

可视化模式下可以使用`Vbe`命令进入变量菜单进行浏览。

### asm.arch

定义了目标的CPU架构，该信息会使用在反汇编(`pd`, `pD`命令)及代码分析(`a`命令)阶段。`e asm.arch=?`或`rasm2 -L`命令会显示该配置项可选的架构。
添加新架构支持非常容易，r2提供了相应的接口。以x86为例，其利用了多种第三方反汇编引擎，包括GNU binutils， Udis86以及自己编写的一些工具。

### asm.bits

该配置项决定了当前架构寄存器的位宽，可选的值包含：8, 16, 32, 64。不过要注意的是，不是所有的架构都支持这些asm.bits位数。

### asm.syntax

该配置项用于选择反汇编语句的语法风格（Intel或AT&T）。目前该配置项仅能影响Udis86反汇编引擎对Intel 32/Intel 64目标的分析，可选的值包含`intel`和`att`。

### asm.pseudo

该配置项为一个bool变量，用于设置汇编伪代码的语法风格，"False"表示使用原生风格，由当前架构定义，"true"则启用伪代码风格。例如，其会将：

```
│           0x080483ff      e832000000     call 0x8048436
│           0x08048404      31c0           xor eax, eax
│           0x08048406      0205849a0408   add al, byte [0x8049a84]
│           0x0804840c      83f800         cmp eax, 0
│           0x0804840f      7405           je 0x8048416
```
转化为

```
│           0x080483ff      e832000000     0x8048436 ()
│           0x08048404      31c0           eax = 0
│           0x08048406      0205849a0408   al += byte [0x8049a84]
│           0x0804840c      83f800         var = eax - 0
│           0x0804840f      7405           if (!var) goto 0x8048416
```
在对一些不清楚不熟悉的架构进行反汇编时，这个配置很有用。

### asm.os

设置当前加载的二进制文件的目标系统。通常情况下OS会通过`rabin -rI`自动检测，但使用`asm.OS`可用于切换到另一个OS所用的syscall表。

### asm.flags

如果设置为"true"，反汇编视图中会显示flags列。

### asm.lines.call

如果设置为"true"，在反汇编(`pd`, `pD`命令)输出结果的左侧会有跳线，可以直观地显示当前代码块内控制流的变化（jump和call)。同样的还可以看看`asm.lines.out`配置项。

### asm.lines.out

若设置为"true"，还会绘制那些超出当前代码块的控制流线。

### asm.linestyle(存疑)

该配置项为一个bool变量，控制控制流的分析方向。若设置为"false"，其会自顶向下地分析代码块，否则自底向上进行分析。默认设置为"false"，对于改善代码可读性来说这是一个更好的选择。

### asm.offset

Bool变量，控制是否显示汇编语句的offset。

### asm.trace

该配置项为Bool变量，控制是否在opcode的左边显示追踪信息（Sequence和counter信息），该配置项的设计目的在于辅助程序的追踪分析。

### asm.bytes

用于控制是否显示指令的原始字节，为一个bool变量。

### asm.sub.reg

该配置项为一个bool变量，用于控制是否以将寄存器的名字替换为参数名或其在指令中扮演的角色。

例如现在有这么一段代码:
```
│           0x080483ea      83c404         add esp, 4
│           0x080483ed      68989a0408     push 0x8049a98
│           0x080483f7      e870060000     call sym.imp.scanf
│           0x080483fc      83c408         add esp, 8
│           0x08048404      31c0           xor eax, eax
```
设置该变量后将变成:
```
│           0x080483ea      83c404         add SP, 4
│           0x080483ed      68989a0408     push 0x8049a98
│           0x080483f7      e870060000     call sym.imp.scanf
│           0x080483fc      83c408         add SP, 8
│           0x08048404      31c0           xor A0, A0
```

### asm.sub.jmp

控制是否在反汇编输出中替换jump、call分支的对象。例如，启用该选项时，`jal 0x80001a40`将显示为`jal fcn.80001a40`。

### asm.sub.rel

该配置项为一个bool变量，控制是否在反汇编输出中替换相对于pc的地址。若启用此选项，引用的地址将显示为引用字符串。

例如：

```
0x5563844a0181      488d3d7c0e00.  lea rdi, [rip + 0xe7c]    ; str.argv__2d_:__s
```
启用该变量时，上面的语句将会被显示为如下样式：

```
0x5563844a0181      488d3d7c0e00.  lea rdi, str.argv__2d_:__s    ; 0x5563844a1004 ; "argv[%2d]: %s\n"
```

### asm.sub.section

该配置决定是否在地址前加上该地址所属的节区。

也就是说，对于下面这个语句：

```
0x000067ea      488d0def0c01.  lea rcx, [0x000174e0]
```

启用该配置后会被转换为：
```
0x000067ea      488d0def0c01.  lea rcx, [fmap.LOAD1.0x000174e0]
```

### asm.sub.varonly

该配置项是一个bool变量，控制是否将本地变量的表达式替换为一个本地变量名。
例如：`rbp - var_14h`会被替换为`var_14h`

### cfg.bigendian

改变大小端，"true"代表大端，"false"代表小端。

### cfg.newtab

若启用该配置，自动补全时将会显示完整命令的名字及其帮助信息。

### scr.color

该变量可以指定屏幕输出的色彩模式："false"(或0)代表无色，"true"(或1)代表16色模式，2代表256色模式，3代表真彩色模式。如果发现你钟爱的色彩主题看起来怪怪的，就试试提高该变量的值吧。

### scr.seek

该变量接受一个表达式或一个指针/flag（比如 eip)。设置该变量后radare2在启动时会跳转到该地址。 

### scr.scrollbar

如果你曾设置过[标志空间(flagzones)](http://book.rada.re/basic_commands/flags.html#flag-zones) (`fz?`)，则设置该变量可以在可视化模式下显示带flagzone的滚动条。
设置为`1`时滚动条放在右端，为`2`时滚动条在顶部，而`3`时滚动条在底部。

### scr.utf8

该配置项为一个bool值，控制在输出中是否使用UTF-8替换原本的ANSI字符。

### cfg.fortunes

启用或禁用每次radare2启动时的"签语(fortune message)"。

### cfg.fortunes.type

这些签语分为不同类型，当`cfg.fortune`为`true`时该配置项将决定哪些类型的签语可以在radare2启动时显示，因此可以通过配置项针对不同的受众对签语进行微调。目前的签语类型包括`tips`，`fun`，`nsfw`，`creepy`。

### stack.size

该变量控制debug时栈的大小，以字节为单位。
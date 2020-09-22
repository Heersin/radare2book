# 调用约定

Radare2使用调用约定的信息帮助对函数的形参和返回值进行识别，调用约定的信息同样用于指导函数原型和类型传递的推断。

```
[0x00000000]> afc?
|Usage: afc[agl?]
| afc convention  Manually set calling convention for current function
| afc             Show Calling convention for the Current function
| afc=([cctype])  Select or show default calling convention
| afcr[j]         Show register usage for the current function
| afca            Analyse function for finding the current calling convention
| afcf[j] [name]  Prints return type function(arg1, arg2...), see afij
| afck            List SDB details of call loaded calling conventions
| afcl            List all available calling conventions
| afco path       Open Calling Convention sdb profile from given path
| afcR            Register telescoping using the calling conventions order
[0x00000000]>
```

* 使用`afcl`命令可以列出当前架构可用的调用约定。

```
[0x00000000]> afcl
amd64
ms
```
* 使用`afcf`命令可以显示标准库函数的函数原型。

```
[0x00000000]> afcf printf
int printf(const char *format)
[0x00000000]> afcf fgets
char *fgets(char *s, int size, FILE *stream)
```

这些信息是从名为`/libr/anal/d/cc-[arch]-[bits].sdb`的sdb中加载的。

```
default.cc=amd64

ms=cc
cc.ms.name=ms
cc.ms.arg1=rcx
cc.ms.arg2=rdx
cc.ms.arg3=r8
cc.ms.arg3=r9
cc.ms.argn=stack
cc.ms.ret=rax
```

`cc.x.argi=rax` 用于将该调用约定的第i个参数设置为`rax`寄存器。

`cc.x.argn=stack` 代表所有参数（如果是在指定了argi调用约定的情况下，则代表除argi之外的所有参数）从左到右存在栈上。

`cc.x.argn=stack_rev` 与 `cc.x.argn=stack` 相同，除了前者代表参数是从右到左储存之外。
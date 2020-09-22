# 变量管理

Radare2允许对本地变量进行管理，无论他们位于何处，是在栈上或寄存器中。对于变量的自动分析是默认开启的，不过你也可以通过`anal.vars`配置选项禁用自动分析。

与变量相关的命令主要在`afv`命名空间下：

```
Usage: afv  [rbs]
| afv*                          output r2 command to add args/locals to flagspace
| afv-([name])                  remove all or given var
| afv=                          list function variables and arguments with disasm refs
| afva                          analyze function arguments/locals
| afvb[?]                       manipulate bp based arguments/locals
| afvd name                     output r2 command for displaying the value of args/locals in the debugger
| afvf                          show BP relative stackframe variables
| afvn [new_name] ([old_name])  rename argument/local
| afvr[?]                       manipulate register based arguments
| afvR [varname]                list addresses where vars are accessed (READ)
| afvs[?]                       manipulate sp based arguments/locals
| afvt [name] [new_type]        change type for given argument/local
| afvW [varname]                list addresses where vars are accessed (WRITE)
| afvx                          show function variable xrefs (same as afvR+afvW)
```

`afvr`，`afvb`和`afvs`命令是相同的命令，他们分别允许对基于寄存器的参数和变量，基于BP/FP的参数和变量，以及基于SP的参数和变量进行修改。`afvr`的帮助信息中显示的用法同样适用于另外两个命令:

```
|Usage: afvr [reg] [type] [name]
| afvr                        list register based arguments
| afvr*                       same as afvr but in r2 commands
| afvr [reg] [name] ([type])  define register arguments
| afvrj                       return list of register arguments in JSON format
| afvr- [name]                delete register arguments at the given index
| afvrg [reg] [addr]          define argument get reference
| afvrs [reg] [addr]          define argument set reference
```

与其他许多功能一样，radare2会自动完成变量分析，但可以用这些控制参数/变量的命令改变这些结果。变量的分析严重依赖于加载的函数原型以及其调用约定，因此加载程序中的符号可以改善分析结果。此外，在对一些东西进行修改后，我们可以用`afva`重新执行变量分析，变量分析常借助[类型分析](types.md)的信息进行，详情参见`afta`命令。

逆向工程中最终要的莫过于赋名，你同样可以对这些变量进行重命名，变量在所有引用处都会被重命名。可以使用`afvn`对 _任何_ 类型的参数和变量完成这个操作。你也可以用`afv-`移除指定的变量和参数。

如前面所提到的那样，变量分析阶段高度依赖于类型信息，因此下一个介绍的重要命令是-`afvt`，使用它可以改变变量的类型。

```
[0x00003b92]> afvs
var int local_8h @ rsp+0x8
var int local_10h @ rsp+0x10
var int local_28h @ rsp+0x28
var int local_30h @ rsp+0x30
var int local_32h @ rsp+0x32
var int local_38h @ rsp+0x38
var int local_45h @ rsp+0x45
var int local_46h @ rsp+0x46
var int local_47h @ rsp+0x47
var int local_48h @ rsp+0x48
[0x00003b92]> afvt local_10h char*
[0x00003b92]> afvs
var int local_8h @ rsp+0x8
var char* local_10h @ rsp+0x10
var int local_28h @ rsp+0x28
var int local_30h @ rsp+0x30
var int local_32h @ rsp+0x32
var int local_38h @ rsp+0x38
var int local_45h @ rsp+0x45
var int local_46h @ rsp+0x46
var int local_47h @ rsp+0x47
var int local_48h @ rsp+0x48
```

接下来是不太常用的功能，其也仍处于开发阶段 - 区分读取与写入的变量。可以通过`afvR`列出变量在哪些位置被读取，`afvR`在哪些位置被写入。这两个命令都列出了读写操作发生的地址。

```
[0x00003b92]> afvR
local_48h  0x48ee
local_30h  0x3c93,0x520b,0x52ea,0x532c,0x5400,0x3cfb
local_10h  0x4b53,0x5225,0x53bd,0x50cc
local_8h  0x4d40,0x4d99,0x5221,0x53b9,0x50c8,0x4620
local_28h  0x503a,0x51d8,0x51fa,0x52d3,0x531b
local_38h
local_45h  0x50a1
local_47h
local_46h
local_32h  0x3cb1
[0x00003b92]> afvW
local_48h  0x3adf
local_30h  0x3d3e,0x4868,0x5030
local_10h  0x3d0e,0x5035
local_8h  0x3d13,0x4d39,0x5025
local_28h  0x4d00,0x52dc,0x53af,0x5060,0x507a,0x508b
local_38h  0x486d
local_45h  0x5014,0x5068
local_47h  0x501b
local_46h  0x5083
local_32h
[0x00003b92]>
```

## 类型推断

局部变量和参数的类型推断很好地整合在`afta`命令中。让我们以[hello_world](https://github.com/radareorg/radare2book/tree/master/examples/hello_world) 这个二进制文件作为例子进行讲解。

```
[0x000007aa]> pdf
|           ;-- main:
/ (fcn) sym.main 157
| sym.main ();
| ; var int local_20h @ rbp-0x20
| ; var int local_1ch @ rbp-0x1c
| ; var int local_18h @ rbp-0x18
| ; var int local_10h @ rbp-0x10
| ; var int local_8h @ rbp-0x8
| ; DATA XREF from entry0 (0x6bd)
| 0x000007aa  push rbp
| 0x000007ab  mov rbp, rsp
| 0x000007ae  sub rsp, 0x20
| 0x000007b2  lea rax, str.Hello          ; 0x8d4 ; "Hello"
| 0x000007b9  mov qword [local_18h], rax
| 0x000007bd  lea rax, str.r2_folks       ; 0x8da ; " r2-folks"
| 0x000007c4  mov qword [local_10h], rax
| 0x000007c8  mov rax, qword [local_18h]
| 0x000007cc  mov rdi, rax
| 0x000007cf  call sym.imp.strlen         ; size_t strlen(const char *s)
```

* 使用 `afta` 后

```
[0x000007aa]> afta
[0x000007aa]> pdf
| ;-- main:
| ;-- rip:
/ (fcn) sym.main 157
| sym.main ();
| ; var size_t local_20h @ rbp-0x20
| ; var size_t size @ rbp-0x1c
| ; var char *src @ rbp-0x18
| ; var char *s2 @ rbp-0x10
| ; var char *dest @ rbp-0x8
| ; DATA XREF from entry0 (0x6bd)
| 0x000007aa  push rbp
| 0x000007ab  mov rbp, rsp
| 0x000007ae  sub rsp, 0x20
| 0x000007b2  lea rax, str.Hello          ; 0x8d4 ; "Hello"
| 0x000007b9  mov qword [src], rax
| 0x000007bd  lea rax, str.r2_folks       ; 0x8da ; " r2-folks"
| 0x000007c4  mov qword [s2], rax
| 0x000007c8  mov rax, qword [src]
| 0x000007cc  mov rdi, rax                ; const char *s
| 0x000007cf  call sym.imp.strlen         ; size_t strlen(const char *s)
```

它同时也从类似`printf ("fmt : %s , %u , %d", ...)`的格式化字符串中提取类型信息，格式的规范是从`anal/d/spec.sdb`中获得的。

您也可以根据不同的库/操作系统/编程语言，创建一个用于指定一组格式字符的新配置文件，如下所示：

```
win=spec
spec.win.u32=unsigned int
```
Then change your default specification to newly created one using this config variable `e anal.spec = win`

For more information about primitive and user-defined types support in radare2 refer to [types](types.md) chapter.

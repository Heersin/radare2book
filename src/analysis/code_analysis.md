# 代码分析

代码分析是在提取汇编码内信息的一种普遍技术，radare2核心实现多种代码分析技术，可以通过不同的命令调用。

由于r2的全部功能都可以通过API或命令来实现，因此你可以使用任何编程语言来实现自己的代码分析方法， 甚至可以用r2 oneliners/shellscripts/原生插件等进行实现。

r2的分析将显示内部数据结构，以识别基本块和函数树，并提取操作码级别的信息。

radare2最常用的分析命令就是`aa`，代表“analyze all”，其中的“all”代指所有的symbols和EP（入口点）。如果是一个stripped的二进制文件，那可能还需要使用其他命令，比如`aaa`，`aab`，`aar`，`aac`等。

花些时间了解各个命令都完成了哪些工作，以及他们会返回哪些信息，再从中选择最适合你的。

```
[0x08048440]> aa
[0x08048440]> pdf @ main
		   ; DATA XREF from 0x08048457 (entry0)
/ (fcn) fcn.08048648 141
|     ;-- main:
|     0x08048648    8d4c2404     lea ecx, [esp+0x4]
|     0x0804864c    83e4f0       and esp, 0xfffffff0
|     0x0804864f    ff71fc       push dword [ecx-0x4]
|     0x08048652    55           push ebp
|     ; CODE (CALL) XREF from 0x08048734 (fcn.080486e5)
|     0x08048653    89e5         mov ebp, esp
|     0x08048655    83ec28       sub esp, 0x28
|     0x08048658    894df4       mov [ebp-0xc], ecx
|     0x0804865b    895df8       mov [ebp-0x8], ebx
|     0x0804865e    8975fc       mov [ebp-0x4], esi
|     0x08048661    8b19         mov ebx, [ecx]
|     0x08048663    8b7104       mov esi, [ecx+0x4]
|     0x08048666    c744240c000. mov dword [esp+0xc], 0x0
|     0x0804866e    c7442408010. mov dword [esp+0x8], 0x1 ;  0x00000001
|     0x08048676    c7442404000. mov dword [esp+0x4], 0x0
|     0x0804867e    c7042400000. mov dword [esp], 0x0
|     0x08048685    e852fdffff   call sym..imp.ptrace
|        sym..imp.ptrace(unk, unk)
|     0x0804868a    85c0         test eax, eax
| ,=< 0x0804868c    7911         jns 0x804869f
| |   0x0804868e    c70424cf870. mov dword [esp], str.Don_tuseadebuguer_ ;  0x080487cf
| |   0x08048695    e882fdffff   call sym..imp.puts
| |      sym..imp.puts()
| |   0x0804869a    e80dfdffff   call sym..imp.abort
| |      sym..imp.abort()
| `-> 0x0804869f    83fb02       cmp ebx, 0x2
|,==< 0x080486a2    7411         je 0x80486b5
||    0x080486a4    c704240c880. mov dword [esp], str.Youmustgiveapasswordforusethisprogram_ ;  0x0804880c
||    0x080486ab    e86cfdffff   call sym..imp.puts
||       sym..imp.puts()
||    0x080486b0    e8f7fcffff   call sym..imp.abort
||       sym..imp.abort()
|`--> 0x080486b5    8b4604       mov eax, [esi+0x4]
|     0x080486b8    890424       mov [esp], eax
|     0x080486bb    e8e5feffff   call fcn.080485a5
|        fcn.080485a5() ; fcn.080484c6+223
|     0x080486c0    b800000000   mov eax, 0x0
|     0x080486c5    8b4df4       mov ecx, [ebp-0xc]
|     0x080486c8    8b5df8       mov ebx, [ebp-0x8]
|     0x080486cb    8b75fc       mov esi, [ebp-0x4]
|     0x080486ce    89ec         mov esp, ebp
|     0x080486d0    5d           pop ebp
|     0x080486d1    8d61fc       lea esp, [ecx-0x4]
\     0x080486d4    c3           ret
```

在这个例子中，我们分析了整个文件（aa），然后打印了main（）函数的反汇编（pdf）。`aa`命令属于自动分析命令族，仅执行最基本的命令自动分析步骤。在radare2中，有许多不同类型的自动分析命令，其中包含不同程度的分析，包括进行部分仿真：`aa`，`aaa`，`aab`，`aaaa`，...这些命令也映射到r2的命令行选项：`r2 -A`，`r2 -AA`等。

我们都知道，完全自动化的分析产生的结果不一定正确，因此Radar2为分析的特定阶段提供了单独的命令，可在细粒度上控制分析过程。此外，radare2还提供了一些配置变量来控制分析结果。可以在配置变量中的`anal.*`和`emu.*`命名空间中找到它们。

## 分析函数

最终要的分析命令之一就是`af`命令族，`af`的意思是”analyze function“。使用该命令可以进行全自动分析，或手动对其分析。

```
[0x00000000]> af?
|Usage: af
| af ([name]) ([addr])     analyze functions (start at addr or $$)
| afr ([name]) ([addr])    analyze functions recursively
| af+ addr name [type] [diff]  hand craft a function (requires afb+)
| af- [addr]               clean all function analysis data (or function at addr)
| afb+ fcnA bbA sz [j] [f] ([t]( [d]))  add bb to function @ fcnaddr
| afb[?] [addr]            List basic blocks of given function
| afbF([0|1])              Toggle the basic-block 'folded' attribute
| afB 16                   set current function as thumb (change asm.bits)
| afC[lc] ([addr])@[addr]  calculate the Cycles (afC) or Cyclomatic Complexity (afCc)
| afc[?] type @[addr]      set calling convention for function
| afd[addr]                show function + delta for given offset
| aff                      re-adjust function boundaries to fit
| afF[1|0|]                fold/unfold/toggle
| afi [addr|fcn.name]      show function(s) information (verbose afl)
| afj [tableaddr] [count]  analyze function jumptable
| afl[?] [ls*] [fcn name]  list functions (addr, size, bbs, name) (see afll)
| afm name                 merge two functions
| afM name                 print functions map
| afn[?] name [addr]       rename name for function at address (change flag too)
| afna                     suggest automatic name for current offset
| afo[?j] [fcn.name]       show address for the function name or current offset
| afs[!] ([fcnsign])       get/set function signature at current address (afs! uses cfg.editor)
| afS[stack_size]          set stack frame size for function at current address
| afsr [function_name] [new_type]  change type for given function
| aft[?]                   type matching, type propagation
| afu [addr]               resize and analyze function from current address until addr
| afv[absrx]?               manipulate args, registers and variables in function
| afx                      list function references
```
使用`afl`可以列出分析中发现的函数。

`afl`有许多有用的下属子命令，比如`aflj`可以以JSON格式将发现的函数列出来，`aflm`以makefile的格式列出这些函数。

这里还有一个`afl=`命令，可以以ASCII字符画的风格用条柱显示函数的范围。

其余的命令可以通过`afl?`查看。

执行函数分析时，最难的一些任务是函数名绑定，微调和调整大小。与其他分析命令一样，有两种模式可以选择：半自动和手动。
半自动模式下，可以使用`afm <function name>`将当前的函数和参数中给出的名字进行绑定。`aff`用于在函数修改后对函数进行调整，`afu <address>`用于将当前函数的范围调整到指定地址，并重新分析。
与半自动模式不同，可以通过`af+`进入手动模式，然后通过`afb`命令对基础块进行编辑。
在修改函数的基本块之前，建议先看看现有的基本块：

```
[0x00003ac0]> afb
0x00003ac0 0x00003b7f 01:001A 191 f 0x00003b7f
0x00003b7f 0x00003b84 00:0000 5 j 0x00003b92 f 0x00003b84
0x00003b84 0x00003b8d 00:0000 9 f 0x00003b8d
0x00003b8d 0x00003b92 00:0000 5
0x00003b92 0x00003ba8 01:0030 22 j 0x00003ba8
0x00003ba8 0x00003bf9 00:0000 81
```

有两个非常重要的命令：“ afc”和“ afB”，对于某些平台（例如ARM），后者是必不可少的命令。因为它提供了一种修改特定函数的“bitness”的方法，主要来说就是其允许选择在ARM和Thumb模式之间进行。`afc` 则是允许手动指定调用约定，可以在[calling_conventions](calling_conventions.md)这一节了解更多信息。

## 递归分析

有5个重要的半自动分析命令用于程序内分析:

 - `aab` - perform basic-block analysis ("Nucleus" algorithm)
 - `aac` - analyze function calls from one (selected or current function)
 - `aaf` - analyze all function calls
 - `aar` - analyze data references
 - `aad` - analyze pointers to pointers references

这些只是通用的半自动化引用搜索方法，Radare2提供了各式各样的引用创建方法，如要进行细粒度的控制，可以使用`ax`命令。

```
Usage: ax[?d-l*]   # see also 'afx?'
| ax              list refs
| ax*             output radare commands
| ax addr [at]    add code ref pointing to addr (from curseek)
| ax- [at]        clean all refs/refs from addr
| ax-*            clean all refs/refs
| axc addr [at]   add generic code ref
| axC addr [at]   add code call ref
| axg [addr]      show xrefs graph to reach current function
| axg* [addr]     show xrefs graph to given address, use .axg*;aggv
| axgj [addr]     show xrefs graph to reach current function in json format
| axd addr [at]   add data ref
| axq             list refs in quiet/human-readable format
| axj             list refs in json format
| axF [flg-glob]  find data/code references of flags
| axm addr [at]   copy data/code references pointing to addr to also point to curseek (or at)
| axt [addr]      find data/code references to this address
| axf [addr]      find data/code references from this address
| axv [addr]      list local variables read-write-exec references
| ax. [addr]      find data/code references from and to this address
| axff[j] [addr]  find data/code references from this function
| axs addr [at]   add string ref
```

最常用的`ax`族命令是`axt`和`axf`，尤其是作为r2pipe脚本的一部分时。假设在数据中发现了字符串，我们想找到所有引用它的位置，需要使用`axt`：
```
[0x0001783a]> pd 2
;-- str.02x:
; STRING XREF from 0x00005de0 (sub.strlen_d50)
; CODE XREF from 0x00017838 (str.._s_s_s + 7)
0x0001783a     .string "%%%02x" ; len=7
;-- str.src_ls.c:
; STRING XREF from 0x0000541b (sub.free_b04)
; STRING XREF from 0x0000543a (sub.__assert_fail_41f + 27)
; STRING XREF from 0x00005459 (sub.__assert_fail_41f + 58)
; STRING XREF from 0x00005f9e (sub._setjmp_e30)
; CODE XREF from 0x0001783f (str.02x + 5)
0x00017841 .string "src/ls.c" ; len=9
[0x0001783a]> axt
sub.strlen_d50 0x5de0 [STRING] lea rcx, str.02x
(nofunc) 0x17838 [CODE] jae str.02x
```

在`axt`下还有些其他有用的命令，`axtg`可以输出radare2的命令，这些命令可以可以帮助你根据XREFs生成交叉引用图。

```
[0x08048320]> s main
[0x080483e0]> axtg
agn 0x8048337 "entry0 + 23"
agn 0x80483e0 "main"
age 0x8048337 0x80483e0
```

使用`axt*`可以将这些命令分割开，然后在特定的XREFs位置上设置标记。

`axg`也在`ax`命令下，可以以XREFs图的形式显示该地址到当前函数/位置之间的路径，比如：

```
:> axg sym.imp.printf
- 0x08048a5c fcn 0x08048a5c sym.imp.printf
  - 0x080483e5 fcn 0x080483e0 main
  - 0x080483e0 fcn 0x080483e0 main
    - 0x08048337 fcn 0x08048320 entry0
  - 0x08048425 fcn 0x080483e0 main
```
使用`axg*`生成的radare2命令可以帮助你根据XREFs，借助`agn`和`age`命令绘制XREF图。

除了用预定义算法识别函数之外，还可以指定一个带有配置选项“anal.prelude”的函数前导码。例如，像
`e anal.prelude = 0x554889e5`，其在x86\_64平台代表如下含义：

```
push rbp
mov rbp, rsp
```

在x86\_64平台上，任何分析命令 _之前_ 都应有该前导码存在。 

## 配置

Radare2几乎允许对任何分析阶段或命令的行为进行修改。
底下是不同种类的配置选项：

 - Flow control
 - Basic blocks control
 - References control
 - IO/Ranges
 - Jump tables analysis control
 - Platform/target specific options

### 控制流配置

改变radare2中控制流分析行为的两个最常用的选项是`anal.hasnext`和`anal.afterjump`。第一个允许radare2在函数结束后强制进行分析，即使接下来的代码块不曾被调用，因此能分析所有潜在的函数。后一个选项允许radare2强制对无条件跳转之后的代码部分继续分析。

除了这些，我们还可以设置`anal.ijmp`来跟随间接跳转，继续进行分析；设置`anal.pushret`将`push ...;ret；`序列也作为跳转进行分析;设置`anal.nopskip`跳过函数开头的NOP序列。

现在，radare2还允许您使用`anal.bb.maxsize`选项更改最大基本块大小。默认块值在大多数情况下都适用，但是在分析混淆代码时增加该值会比较有用。要注意的是，块控制相关的选项在未来可能会取消，我们
将来更倾向于采用自动化的方式对该值进行设置。

对于一些不寻常的二进制文件或目标，有一个选项`anal.noncode`可用。Radare2默认情况下不会将数据段作为代码进行分析，但是在某些情况下比如恶意软件，加壳的二进制文件，或者是嵌入式系统上的文件通常需要这么分析，因此我们便提供了这个选项。

### 引用控制

这个关键选项可以彻底改变分析结果。有时候可以禁用此选项以节省分析大文件的时间，以及节省内存。

- `anal.jmpref` - to allow references creation for unconditional jumps
- `anal.cjmpref` - same, but for conditional jumps
- `anal.datarefs` - to follow the data references in code
- `anal.refstr` - search for strings in data references
- `anal.strings` - search for strings and creating references

注意默认情况下string reference是禁用的，因为它会增加分析消耗的时间。

### 分析范围

这里是一些用于控制分析范围的选项:

- `anal.limits` - enables the range limits for analysis operations
- `anal.from` - starting address of the limit range
- `anal.to` - the corresponding end of the limit range
- `anal.in` - specify search boundaries for analysis. You can set it to `io.maps`, `io.sections.exec`, `dbg.maps` and many more. For example:
  - To analyze a specific memory map with `anal.from` and `anal.to`, set `anal.in = dbg.maps`.
  - To analyze in the boundaries set by `anal.from` and `anal.to`, set `anal.in=range`.
  - To analyze in the current mapped segment or section, you can put `anal.in=bin.segment` or `anal.in=bin.section`, respectively.
  - To analyze in the current memory map, specify `anal.in=dbg.map`.
  - To analyze in the stack or heap, you can set `anal.in=dbg.stack` or `anal.in=dbg.heap`.
  - To analyze in the current function or basic block, you can specify `anal.in=anal.fcn` or `anal.in=anal.bb`.

请参阅 `e anal.in=??` 获取完整的选项列表。

### 跳转表

跳转表是二进制逆向工程中最棘手的目标之一，有几百个不同的类型，其最终的结果取决于编译器/链接器和LTO优化阶段。因此，radare2允许使用`anal.jmptbl`选项启用一些实验性质的跳转表检测算法。最终，算法一旦开始在每个受支持的平台/目标/测试用例上工作，便进入默认的分析循环。
下面的两个选项也能影响跳转表分析的工作:

- `anal.ijmp` - follow the indirect jumps, some jump tables rely on them
- `anal.datarefs` - follow the data references, some jump tables use those

### 特定于平台的一些控制

分析嵌入式目标时，有两个常见问题：ARM/Thumb检测和MIPS GP值。如果是ARM二进制文件，radare2支持对ARM/Thumb模式切换的某些自动检测，但是要注意它使用部分ESIL仿真技术，从而减慢了分析过程。如果你不愿意接受，可以使用`afB`命令覆盖特定函数的ARM/Thumb模式。

MIPS GP问题更加棘手。基本上来说，GP值不仅在整个程序可以是不同的，而且在函数内也可以不同。`anal.gp`和`anal.gp2`可以部分解决这些问题。第一个选项设置整个程序或特定函数的GP值，后一个选项则是在某些函数想要修改GP值时，去”固定“GP值，即在GP值被修改之后恢复到原来状态。这些都是实验性质的选项，在未来倾向于以更自动化的方式代替。

## 可视化

查看分析命令与变量更改的最简单方法之一是在“ Vv”视觉模式下滚动屏幕，并且可以预览函数：

![vv](code_analysis_vv.png)

当我们在面对大函数时， 若想看看分析对结果有何影响，可以使用minimap，以便在相同的屏幕尺寸下查看更大的流程图。通过`VV`然后按下2次`p`可以进入minimap模式。

![vv2](code_analysis_vv2.png)

这种模式可以分别查看每个节点的反汇编，只需使用“ Tab”键在它们之间切换即可。

## 分析推断（analysis hints）

即使您尝试了每一个配置选项，分析结果也仍不完美的情况并不少见。这时候Radare2的“analysis hints”技术就该登场啦。它允许重写一些基本的操作码或元信息属性，甚至重写整个操作码串。这些命令位于“ah”命名空间下：

```
Usage: ah[lba-]  Analysis Hints
| ah?                show this help
| ah? offset         show hint of given offset
| ah                 list hints in human-readable format
| ah.                list hints in human-readable format from current offset
| ah-                remove all hints
| ah- offset [size]  remove hints at given offset
| ah* offset         list hints in radare commands format
| aha ppc @ 0x42     force arch ppc for all addrs >= 0x42 or until the next hint
| aha 0 @ 0x84       disable the effect of arch hints for all addrs >= 0x84 or until the next hint
| ahb 16 @ 0x42      force 16bit for all addrs >= 0x42 or until the next hint
| ahb 0 @ 0x84       disable the effect of bits hints for all addrs >= 0x84 or until the next hint
| ahc 0x804804       override call/jump address
| ahd foo a0,33      replace opcode string
| ahe 3,eax,+=       set vm analysis string
| ahf 0x804840       override fallback address for call
| ahF 0x10           set stackframe size at current offset
| ahh 0x804840       highlight this address offset in disasm
| ahi[?] 10          define numeric base for immediates (2, 8, 10, 10u, 16, i, p, S, s)
| ahj                list hints in JSON
| aho call           change opcode type (see aho?) (deprecated, moved to "ahd")
| ahp addr           set pointer hint
| ahr val            set hint for return value of a function
| ahs 4              set opcode size=4
| ahS jz             set asm.syntax=jz for this opcode
| aht [?] <type>     Mark immediate as a type offset (deprecated, moved to "aho")
| ahv val            change opcode's val field (useful to set jmptbl sizes in jmp rax)
```

最常见的情况之一是为立即数设置特定的数字基数：

```
[0x00003d54]> ahi?
Usage: ahi [2|8|10|10u|16|bodhipSs] [@ offset]   Define numeric base
| ahi <base>  set numeric base (2, 8, 10, 16)
| ahi 10|d    set base to signed decimal (10), sign bit should depend on receiver size
| ahi 10u|du  set base to unsigned decimal (11)
| ahi b       set base to binary (2)
| ahi o       set base to octal (8)
| ahi h       set base to hexadecimal (16)
| ahi i       set base to IP address (32)
| ahi p       set base to htons(port) (3)
| ahi S       set base to syscall (80)
| ahi s       set base to string (1)

[0x00003d54]> pd 2
0x00003d54      0583000000     add eax, 0x83
0x00003d59      3d13010000     cmp eax, 0x113
[0x00003d54]> ahi d
[0x00003d54]> pd 2
0x00003d54      0583000000     add eax, 131
0x00003d59      3d13010000     cmp eax, 0x113
[0x00003d54]> ahi b
[0x00003d54]> pd 2
0x00003d54      0583000000     add eax, 10000011b
0x00003d59      3d13010000     cmp eax, 0x113
```

值得注意的是，某些分析阶段中或某些命令会添加radare2内部的分析推断，可以用ah命令检查：

```
[0x00003d54]> ah
 0x00003d54 - 0x00003d54 => immbase=2
[0x00003d54]> ah*
 ahi 2 @ 0x3d54
```

有时我们需要覆盖跳转或调用地址，例如，在棘手的情况下进行重定位，这对于radare2是未知的，因此我们得手动更改该值。
可以使用ao命令检查有关特定操作码的当前分析信息，我们可以使用`ahc`命令执行这样的更改：

```
[0x00003cee]> pd 2
0x00003cee      e83d080100     call sub.__errno_location_530
0x00003cf3      85c0           test eax, eax
[0x00003cee]> ao
address: 0x3cee
opcode: call 0x14530
mnemonic: call
prefix: 0
id: 56
bytes: e83d080100
refptr: 0
size: 5
sign: false
type: call
cycles: 3
esil: 83248,rip,8,rsp,-=,rsp,=[],rip,=
jump: 0x00014530
direction: exec
fail: 0x00003cf3
stack: null
family: cpu
stackop: null
[0x00003cee]> ahc 0x5382
[0x00003cee]> pd 2
0x00003cee      e83d080100     call sub.__errno_location_530
0x00003cf3      85c0           test eax, eax
[0x00003cee]> ao
address: 0x3cee
opcode: call 0x14530
mnemonic: call
prefix: 0
id: 56
bytes: e83d080100
refptr: 0
size: 5
sign: false
type: call
cycles: 3
esil: 83248,rip,8,rsp,-=,rsp,=[],rip,=
jump: 0x00005382
direction: exec
fail: 0x00003cf3
stack: null
family: cpu
stackop: null
[0x00003cee]> ah
 0x00003cee - 0x00003cee => jump: 0x5382
```

正如你所见到的那样，尽管反汇编视图不变，但操作码中的跳转地址已更改（`jump`选项）。

如果先前描述的任何方法都不起效，则可以用任何你喜欢的东西简单地覆盖显示的反汇编：
```
[0x00003d54]> pd 2
0x00003d54      0583000000     add eax, 10000011b
0x00003d59      3d13010000     cmp eax, 0x113
[0x00003d54]> "ahd myopcode bla, foo"
[0x00003d54]> pd 2
0x00003d54                     myopcode bla, foo
0x00003d55      830000         add dword [rax], 0
```

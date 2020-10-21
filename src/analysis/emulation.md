# 模拟执行

逆向工程中最重要的事情之一就是记住静态分析和动态分析之间的核心不同点。正如大多数人所知的那样，静态分析受困于路径爆炸的问题，在不采用部分模拟的情况下即使连最基本的分析也达不到。

因此许多专业的逆向工具在对二进制代码进行分析时采用模拟执行的方式，radare2在这点上与其他工具无二。

radare2采用自有的[ESIL](../disassembling/esil.md)中间语言和虚拟机进行部分模拟（或者不精确的完全模拟方式）。

Radare2为所有实现ESIL提炼的架构平台提供了部分模拟的功能。(x86/x86_64, ARM, arm64, MIPS, powerpc, sparc, AVR, 8051, Gameboy, ...)。

这类模拟最常见的用途就是用于间接跳转和条件跳转的计算。

若要查看程序的ESIL表示，可以使用`ao`命令或者将`asm.esil`配置变量设置为`true`。可以看看下面的程序中ESIL提炼的结果是否正确，并把握到ESIL的效果：

```
[0x00001660]> pdf
. (fcn) fcn.00001660 40
│   fcn.00001660 ();
│     ; CALL XREF from 0x00001713 (entry2.fini)
│     0x00001660  lea rdi, obj.__progname      ; 0x207220
│     0x00001667  push rbp
│     0x00001668  lea rax, obj.__progname      ; 0x207220
│     0x0000166f  cmp rax, rdi
│     0x00001672  mov rbp, rsp
│ .─< 0x00001675  je 0x1690
│ │   0x00001677  mov rax, qword [reloc._ITM_deregisterTMCloneTable] ; [0x206fd8:8]=0
│ │   0x0000167e  test rax, rax
│.──< 0x00001681  je 0x1690
│││   0x00001683  pop rbp
│││   0x00001684  jmp rax
│``─> 0x00001690  pop rbp
`     0x00001691  ret
[0x00001660]> e asm.esil=true
[0x00001660]> pdf
. (fcn) fcn.00001660 40
│   fcn.00001660 ();
│     ; CALL XREF from 0x00001713 (entry2.fini)
│     0x00001660  0x205bb9,rip,+,rdi,=
│     0x00001667  rbp,8,rsp,-=,rsp,=[8]
│     0x00001668  0x205bb1,rip,+,rax,=
│     0x0000166f  rdi,rax,==,$z,zf,=,$b64,cf,=,$p,pf,=,$s,sf,=,$o,of,=
│     0x00001672  rsp,rbp,=
│ .─< 0x00001675  zf,?{,5776,rip,=,}
│ │   0x00001677  0x20595a,rip,+,[8],rax,=
│ │   0x0000167e  0,rax,rax,&,==,$z,zf,=,$p,pf,=,$s,sf,=,$0,cf,=,$0,of,=
│.──< 0x00001681  zf,?{,5776,rip,=,}
│││   0x00001683  rsp,[8],rbp,=,8,rsp,+=
│││   0x00001684  rax,rip,=
│``─> 0x00001690  rsp,[8],rbp,=,8,rsp,+=
`     0x00001691  rsp,[8],rip,=,8,rsp,+=
```

若要进行ESIL非精确模拟，需要使用如下的命令序列：

- `aei` 用于初始化ESIL VM
- `aeim` 初始化 ESIL VM memory (stack)
- `aeip` 初始化 ESIL VM IP (instruction pointer)
- 一系列的 `aer` 设定寄存器的初值。

在进行模拟时需要记住的是，ESIL VM不支持模拟外部调用或系统调用，此外也不支持SIMD指令。因此最常见的情况就是使用其对一小部分的代码进行模拟，比如加密/解密，脱壳或计算什么东西之类的。

在成功设置ESIL VM之后，可在类似平常的debug mode下与之进行交互。
ESIL VM的命令行接口与debug mode中几乎是相同的：

- `aes` 步入 (或者visual mode里的`s`)
- `aesi` 步过函数
- `aesu <address>` 运行至某地址
- `aesue <ESIL expression>` 运行至遇见某个汇编指令
- `aec` 运行至命令行打断 (Ctrl-C), 不过由于系统调用几乎无处不在，这个命令很少被用到

在visual mode模式下所有的debug快捷键也都能在ESIL模拟模式下使用。

如通常的模拟一样，这里也提供了记录和重播的功能：

- `aets` 列出现存的 ESIL R&R sessions
- `aets+` 创建一个新的session
- `aesb` 在现在的ESIL R&R session中进行回溯

可以阅读[Reverse Debugging](../debugger/revdebug.md)这一章节了解该操作。

## 在analysis loop阶段进行模拟

除了手动进行模拟的模式外，其还支持在循环分析阶段继续自动模拟。
例如`aaaa`命令在ESIL模拟阶段及其它阶段进行ESIL模拟。可以配置`anal.esil`变量禁用或启用该特性。这里还有一个更重要的选项，尽管设置该选项有一定危险性，特别是在进行病毒分析的时候 - `emu.write`使得ESIL VM能够对内存进行修改。有时候可能会需要这个功能，尤其是在去混淆或脱壳的时候。

可以设置`asm.emu`变量显示模拟过程，其将会在反汇编的注释区显示计算出的寄存器值和内存值。

```
[0x00001660]> e asm.emu=true
[0x00001660]> pdf
. (fcn) fcn.00001660 40
│   fcn.00001660 ();
│     ; CALL XREF from 0x00001713 (entry2.fini)
│     0x00001660  lea rdi, obj.__progname ; 0x207220 ; rdi=0x207220 -> 0x464c457f
│     0x00001667  push rbp                ; rsp=0xfffffffffffffff8
│     0x00001668  lea rax, obj.__progname ; 0x207220 ; rax=0x207220 -> 0x464c457f
│     0x0000166f  cmp rax, rdi            ; zf=0x1 -> 0x2464c45 ; cf=0x0 ; pf=0x1 -> 0x2464c45 ; sf=0x0 ; of=0x0
│     0x00001672  mov rbp, rsp            ; rbp=0xfffffffffffffff8
│ .─< 0x00001675  je 0x1690               ; rip=0x1690 -> 0x1f0fc35d ; likely
│ │   0x00001677  mov rax, qword [reloc._ITM_deregisterTMCloneTable] ; [0x206fd8:8]=0 ; rax=0x0
│ │   0x0000167e  test rax, rax           ; zf=0x1 -> 0x2464c45 ; pf=0x1 -> 0x2464c45 ; sf=0x0 ; cf=0x0 ; of=0x0
│.──< 0x00001681  je 0x1690               ; rip=0x1690 -> 0x1f0fc35d ; likely
│││   0x00001683  pop rbp                 ; rbp=0xffffffffffffffff -> 0x4c457fff ; rsp=0x0
│││   0x00001684  jmp rax                 ; rip=0x0 ..
│``─> 0x00001690  pop rbp                 ; rbp=0x10102464c457f ; rsp=0x8 -> 0x464c457f
`     0x00001691  ret                     ; rip=0x0 ; rsp=0x10 -> 0x3e0003
```

注意一下这里的`likely`这个注释，这表示ESIL对该条件跳转进行分支预测后的结果。

除了基本的ESIL VM设置外，还可以使用位于`emu.`和`seil.`配置空间下的其它选项改变模拟行为。

可以使用如下的选项修改ESIL内存和栈的行为：

- `esil.stack` 在`asm.emu`mode下启用或禁用临时栈
- `esil.stack.addr` 在ESIL VM设置栈位置（类似`aeim`命令）
- `esil.stack.size` 设置ESIL VM中的栈大小（类似`aeim`命令）
- `esil.stack.depth` 限制对栈进行PUSH操作的次数（栈的深度）
- `esil.romem` 指定ESIL内存为只读
- `esil.fillstack` 和 `esil.stack.pattern` 允许使用一个pattern，用于ESIL VM栈初始化时进行数据填充
- `esil.nonull` 如果设置此变量，则当读取/写入NULL指针时停止ESIL的执行。
# 反汇编

反汇编在radare2中只是一组字节的不同表示形式，其作为`p`命令下一种特殊的输出格式存在。

在过去radare的核心程序还比较小巧的时候，反汇编器是由外部rsc文件进行管理的，即radare首先将当前的代码块转储到一个文件中，然后简单地调用配置好的`objdump`对Intel，ARM或其他支持的架构进行反汇编。

该方案是可行的，且对Unix很友好，但是它的效率低下，因为在该方案中没有cache缓存，只会一边又一边地执行这些高代价的操作，导致滚动浏览汇编代码时非常缓慢。

因而我们需要创造一个通用的反汇编库，以支持针对不同架构的反汇编插件，可以使用下面的命令列出当前加载的插件：

```
$ rasm2 -L
```

或者在radare2里使用：

```
> e asm.arch=??
```

在突破出现之前的很多年里，r2使用的都是udis86和olly disassemblers，以及GNU binutils中的工具。

现在，反汇编器的支持已经变成了radare2的一个基本功能，现在的它具有许多选项，支持许多字节序，以及不同的目标架构风格，还拥有多种反汇编器。

使用`pd`可以看到对应的汇编代码，`pd`接受一个数字型参数，用于指定显示当前代码块中多少条操作码。Radare2中的多数命令对于代码块的大小有一个默认的限制，如果你想对更多的字节进行反汇编，需使用`b`重设代码块的大小。

```
[0x00000000]> b 100    ; set block size to 100
[0x00000000]> pd       ; disassemble 100 bytes
[0x00000000]> pd 3     ; disassemble 3 opcodes
[0x00000000]> pD 30    ; disassemble 30 bytes
```

`pD`命令与`pd`命令是类似的，差别在于其接受的参数代表字节数而非`pd`中的操作码数。

对于人类来说，“pesudo”伪语法可能比默认的汇编代码更容易理解。不过在面对大量代码时可能会对此感到厌烦。若要尝试体验这个功能，就像下面这样进行设置：

```
[0x00405e1c]> e asm.pseudo = true
[0x00405e1c]> pd 3
		  ; JMP XREF from 0x00405dfa (fcn.00404531)
		  0x00405e1c    488b9424a80. rdx = [rsp+0x2a8]
		  0x00405e24    64483314252. rdx ^= [fs:0x28]
		  0x00405e2d    4889d8       rax = rbx

[0x00405e1c]> e asm.syntax = intel
[0x00405e1c]> pd 3
		  ; JMP XREF from 0x00405dfa (fcn.00404531)
		  0x00405e1c    488b9424a80. mov rdx, [rsp+0x2a8]
		  0x00405e24    64483314252. xor rdx, [fs:0x28]
		  0x00405e2d    4889d8       mov rax, rbx

[0x00405e1c]> e asm.syntax=att
[0x00405e1c]> pd 3
		  ; JMP XREF from 0x00405dfa (fcn.00404531)
		  0x00405e1c    488b9424a80. mov 0x2a8(%rsp), %rdx
		  0x00405e24    64483314252. xor %fs:0x28, %rdx
		  0x00405e2d    4889d8       mov %rbx, %rax
```


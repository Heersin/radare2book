## 汇编

汇编可以将人类可读的计算机指令（助记符）转化为一串可在机器上直接执行的字节。

Radare2中汇编器和反汇编器的逻辑实现于r_asm_* API，可以在命令行里用`pa`和`pad`命令使用它们，也可以通过`rasm2`调用这些功能。

Rasm2可以快速地获取给定机器指令对应的十六进制字节串，便于复制粘贴。下面的例子展示了如何将x86/32上的mov指令进行汇编为机器码。

```
$ rasm2 -a x86 -b 32 'mov eax, 33'
b821000000
```

除了将指令作为`rasm`命令的参数输入之外，还可以通过pipe管道传入：

```
$ echo 'push eax;nop;nop' | rasm2 -f -
5090
```

正如所见那样，rasm2可以汇编一条或多条指令。若要在一行命令中汇编多条指令，需要用分号`;`分隔指令，不过你也可以选择直接从文件中读取汇编指令，文件中只需遵循通常的nasm/gas/..语法。若想了解更多信息，可以查看rasm2的man手册页面。

`pa`和`pad`是print的子命令，代表仅打印汇编/反汇编。若想要将指令写入则需要使用`wa`或`wx`命令，后接对应的汇编字符串或字节。

汇编器能理解如下的语言及语法风格：
`x86` (Intel and AT&T variants), `olly` (OllyDBG syntax), `powerpc` (PowerPC), `arm` and `java`. 
对于Intel语法，rasm2模仿了NASM和GAS的做法。

在rasm2的源码目录下有多个例子，通过它们可以了解如何根据rasm2描述法，汇编出原始二进制文件。

首先我们创建一个名为`selfstop.rasm`的汇编文件:

```asm
;
; Self-Stop shellcode written in rasm for x86
;
; --pancake
;

.arch x86
.equ base 0x8048000
.org 0x8048000  ; the offset where we inject the 5 byte jmp

selfstop:
  push 0x8048000
  pusha
  mov eax, 20
  int 0x80

  mov ebx, eax
  mov ecx, 19
  mov eax, 37
  int 0x80
  popa
  ret
;
; The call injection
;

  ret
```

将其汇编并在r2中分析：

```
[0x00000000]> e asm.bits = 32
[0x00000000]> wx `!rasm2 -f a.rasm`
[0x00000000]> pd 20
	   0x00000000    6800800408   push 0x8048000 ;  0x08048000
	   0x00000005    60           pushad
	   0x00000006    b814000000   mov eax, 0x14 ;  0x00000014
	   0x0000000b    cd80         int 0x80
		  syscall[0x80][0]=?
	   0x0000000d    89c3         mov ebx, eax
	   0x0000000f    b913000000   mov ecx, 0x13 ;  0x00000013
	   0x00000014    b825000000   mov eax, 0x25 ;  0x00000025
	   0x00000019    cd80         int 0x80
		  syscall[0x80][0]=?
	   0x0000001b    61           popad
	   0x0000001c    c3           ret
	   0x0000001d    c3           ret
```

### 可视化模式

在radare2可视化模式下同样可以进行汇编，按下`A`键可以在当前偏移量处插入汇编语句。
最酷的是在可视化模式下，按下回车之前进行的汇编写入操作都是在内存中完成的。因此在保存修改之前我们可以检查代码块的大小以及是否有哪个指令会产生代码覆盖。

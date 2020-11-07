# ESIL

ESIL代表'可求值字符串形中间语言'，旨在为各种目标CPU opcode提供一个[Forth](https://en.wikipedia.org/wiki/Forth_%28programming_language%29)-like的语义描述。可以对ESIL表示进行求值（运行），用于模拟一些独立的指令。每个ESIL表示都通过逗号进行分隔，大致可用如下的形式描述ESIL虚拟机：
```
   while ((word=haveCommand())) {
     if (word.isOperator()) {
       esilOperators[word](esil);
     } else {
       esil.push (word);
     }
     nextCommand();
   }
```

同计算器中类似，ESIL使用一个基于栈的解释器。其接受两类参数：数值和操作符，数值被压栈，操作符则将栈顶值弹出（或者说该操作符的参数），操作完成后将结果（如果有的话）压栈。可以将ESIL当作指令的一种逆波兰表示（post-fix表示）。

让我们看看一个例子：
```
4,esp,-=,ebp,esp,=[4]
```
可以猜猜这条语句在做什么，如果将这个逆波兰表示的语句转换为正常的指令，可以得到：
```
esp -= 4
4bytes(dword) [esp] = ebp
```
我们可以看到这正是x86指令`push ebp`的另一种形式！是不是很cool？ESIL的目标是能够对CPU的常见操作进行解释，比如二进制算数操作、内存加载和存储、系统调用等。通过这种方法，如果能够将指令转化为ESIL，即使面对一个加密架构且我们没有该架构对应的debug，我们也能在观察到该程序在运行时都做了什么。

## 使用ESIL

r2的可视化模式对于观察ESIL执行非常有帮助。

底下是些很重要的环境变量，可用于观察程序行为：
```
[0x00000000]> e emu.str = true
```

`asm.emu` 告诉r2是否将ESIL的信息展示出来。如果设置为`true`，反汇编的注释区域中将显示经过该指令后寄存器和内存地址会如何变化。例如，假设这里有一条指令减去了寄存器中的值，注释中会告诉你原先的值是多少，最终结果又是多少。这真的特别有用，使得你无需坐在位置上手动追踪数值的变化。

但是它存在的一个问题是一次显示的信息太多了，而我们有时候不需要这么多信息。针对这个问题，r2有一个很好的折中方案，就是使用`emu.str`变量（在<=2.2版本是`asm.emustr`）。不输出每个寄存器的值，取而代之的是一些真正有用的信息。例如，显示程序中引用地址上的字符串，或者判断哪个跳转分支比较可能发生。

第三个重要的变量就是`asm.esil`，这个开关控制反汇编不再显示真实的反汇编指令，而是显示它们的ESIL表示，描述该指令正进行什么操作。
因此如果你想要看看指令是如何以ESIL表示的，只需将`asm.esil` 设置为true即可。

```
[0x00000000]> e asm.esil = true
```

在可视化模式下可以用`0`键切换至ESIL表示。

## ESIL 命令

* "ae" : 对ESIL进行求值。

```
[0x00000000]> "ae 1,1,+"
0x2
[0x00000000]>
```

* "aes" : ESIL单步.

```
[0x00000000]> aes
[0x00000000]>10aes
```
* "aeso" : ESIL步过.

```
[0x00000000]> aeso
[0x00000000]>10aeso
```

* "aesu" : ESIL单步直到某个地址.

```
[0x00001000]> aesu 0x1035
ADDR BREAK
[0x00001019]>
```

* "ar" : 显示/修改ESIL寄存器.

```
[0x00001ec7]> ar r_00 = 0x1035
[0x00001ec7]> ar r_00
0x00001035
[0x00001019]>
```

### ESIL指令集

这里是ESIL VM使用的完整指令集：

ESIL Opcode | Operands | Name | Operation| example
--- | --- | --- | --- | ----------------------------------------------
TRAP  | src | Trap | Trap signal |
**$** | src | Syscall | syscall  |
**$$** | src | Instruction address | Get address of current instruction<br>stack=instruction address |
**==** | src,dst | Compare | stack = (dst == src) ; <br> update_eflags(dst - src) |
**<** | src,dst | Smaller (signed comparison) | stack = (dst < src) ; <br> update_eflags(dst - src) | [0x0000000]> "ae 1,5,<" <br>0x0<br>&gt; "ae 5,5"<br>0x0"
**<=** | src,dst | Smaller or Equal (signed comparison) | stack = (dst <= src) ; <br> update_eflags(dst - src) | [0x0000000]> "ae 1,5,<" <br>0x0<br>&gt; "ae 5,5"<br>0x1"
**>** | src,dst | Bigger (signed comparison) | stack = (dst > src) ; <br> update_eflags(dst - src) | &gt; "ae 1,5,>"<br>0x1<br>&gt; "ae 5,5,>"<br>0x0
 **>=** | src,dst | Bigger or Equal (signed comparison) | stack = (dst >= src) ; <br> update_eflags(dst - src) | &gt; "ae 1,5,>="<br>0x1<br>&gt; "ae 5,5,>="<br>0x1
 **<<** | src,dst | Shift Left | stack = dst << src | &gt; "ae 1,1,<<"<br>0x2<br>&gt; "ae 2,1,<<"<br>0x4
 **>>** | src,dst | Shift Right | stack = dst >> src | &gt; "ae 1,4,>>"<br>0x2<br>&gt; "ae 2,4,>>"<br>0x1
 **<<<** | src,dst | Rotate Left | stack=dst ROL src | &gt; "ae 31,1,<<<"<br>0x80000000<br>&gt; "ae 32,1,<<<"<br>0x1
**>>>** | src,dst | Rotate Right | stack=dst ROR src | &gt; "ae 1,1,>>>"<br>0x80000000<br>&gt; "ae 32,1,>>>"<br>0x1
**&** | src,dst | AND | stack = dst & src | &gt; "ae 1,1,&"<br>0x1<br>&gt; "ae 1,0,&"<br>0x0<br>&gt;  "ae 0,1,&"<br>0x0<br>&gt; "ae 0,0,&"<br>0x0
**&#x7c;** | src,dst | OR | stack = dst &#x7c; src | &gt; "ae 1,1,&#x7c;"<br>0x1<br>&gt; "ae 1,0,&#x7c;"<br>0x1<br>&gt; "ae 0,1,&#x7c;"<br>0x1<br>&gt; "ae 0,0,&#x7c;"<br>0x0
**^** | src,dst | XOR | stack = dst ^src  | &gt; "ae 1,1,^"<br>0x0<br>&gt; "ae 1,0,^"<br>0x1<br>&gt; "ae 0,1,^"<br>0x1<br>&gt; "ae 0,0,^"<br>0x0
**+** | src,dst | ADD | stack = dst + src | &gt; "ae 3,4,+"<br>0x7<br>&gt; "ae 5,5,+"<br>0xa
**-** | src,dst | SUB | stack = dst - src | &gt; "ae 3,4,-"<br>0x1<br>&gt; "ae 5,5,-"<br>0x0<br>&gt; "ae 4,3,-"<br>0xffffffffffffffff
**\*** | src,dst | MUL | stack = dst * src | &gt; "ae 3,4,\*"<br>0xc<br>&gt; "ae 5,5,\*"<br>0x19
**/** | src,dst | DIV | stack = dst / src  | &gt; "ae 2,4,/"<br>0x2<br>&gt; "ae 5,5,/"<br>0x1<br>&gt; "ae 5,9,/"<br>0x1
**%** | src,dst | MOD | stack = dst % src | &gt; "ae 2,4,%"<br>0x0<br>&gt; "ae 5,5,%"<br>0x0<br>&gt; "ae 5,9,%"<br>0x4
**!** | src | NEG | stack = !!!src | &gt; "ae 1,!"<br>0x0<br>&gt; "ae 4,!"<br>0x0<br>&gt; "ae 0,!"<br>0x1<br>
**++** | src | INC | stack = src++ | &gt; ar r_00=0;ar r_00<br>0x00000000<br>&gt; "ae r_00,++"<br>0x1<br>&gt; ar r_00<br>0x00000000<br>&gt; "ae 1,++"<br>0x2
**--** | src | DEC | stack = src-- | &gt; ar r_00=5;ar r_00<br>0x00000005<br>&gt; "ae r_00,--"<br>0x4<br>&gt; ar r_00<br>0x00000005<br>&gt; "ae 5,--"<br>0x4
**=** | src,reg | EQU | reg = src | &gt; "ae 3,r_00,="<br>&gt; aer r_00<br>0x00000003<br>&gt; "ae r_00,r_01,="<br>&gt; aer r_01<br>0x00000003
**+=** | src,reg | ADD eq | reg = reg + src | &gt; ar r_01=5;ar r_00=0;ar r_00<br>0x00000000<br>&gt; "ae r_01,r_00,+="<br>&gt; ar r_00<br>0x00000005<br>&gt; "ae 5,r_00,+="<br>&gt; ar r_00<br>0x0000000a
**-=** | src,reg | SUB eq | reg = reg - src | &gt; "ae r_01,r_00,-="<br>&gt; ar r_00<br>0x00000004<br>&gt; "ae 3,r_00,-="<br>&gt; ar r_00<br>0x00000001
**\*=** | src,reg | MUL eq | reg = reg * src | &gt; ar r_01=3;ar r_00=5;ar r_00<br>0x00000005<br>&gt; "ae r_01,r_00,\*="<br>&gt; ar r_00<br>0x0000000f<br>&gt; "ae 2,r_00,\*="<br>&gt; ar r_00<br>0x0000001e
 **/=** | src,reg | DIV eq | reg = reg / src | &gt; ar r_01=3;ar r_00=6;ar r_00<br>0x00000006<br>&gt; "ae r_01,r_00,/="<br>&gt; ar r_00<br>0x00000002<br>&gt; "ae 1,r_00,/="<br>&gt; ar r_00<br>0x00000002
 **%=** | src,reg | MOD eq | reg = reg % src | &gt;  ar r_01=3;ar r_00=7;ar r_00<br> 0x00000007<br> &gt; "ae r_01,r_00,%="<br> &gt; ar r_00<br> 0x00000001<br> &gt;  ar r_00=9;ar r_00<br> 0x00000009<br> &gt; "ae 5,r_00,%="<br> &gt; ar r_00<br> 0x00000004
**<<=** | src,reg | Shift Left eq | reg = reg << src | &gt; ar r_00=1;ar r_01=1;ar r_01<br>0x00000001<br>&gt; "ae r_00,r_01,<<="<br>&gt; ar r_01<br>0x00000002<br>&gt; "ae 2,r_01,<<="<br>&gt; ar r_01<br>0x00000008
**>>=** | src,reg | Shift Right eq | reg = reg << src | &gt; ar r_00=1;ar r_01=8;ar r_01<br>0x00000008<br>&gt; "ae r_00,r_01,>>="<br>&gt; ar r_01<br>0x00000004<br>&gt; "ae 2,r_01,>>="<br>&gt; ar r_01<br>0x00000001
**&=** | src,reg |  AND eq | reg = reg & src | &gt; ar r_00=2;ar r_01=6;ar r_01<br>0x00000006<br>&gt; "ae r_00,r_01,&="<br>&gt; ar r_01<br>0x00000002<br>&gt; "ae 2,r_01,&="<br>&gt; ar r_01<br>0x00000002<br>&gt; "ae 1,r_01,&="<br>&gt; ar r_01<br>0x00000000
**&#x7c;=** | src,reg | OR eq| reg = reg &#x7c; src | &gt; ar r_00=2;ar r_01=1;ar r_01<br>0x00000001<br>&gt; "ae r_00,r_01,&#x7c;="<br>&gt; ar r_01<br>0x00000003<br>&gt; "ae 4,r_01,&#x7c;="<br>&gt; ar r_01<br>0x00000007
 **^=** | src,reg | XOR eq | reg = reg ^ src | &gt; ar r_00=2;ar r_01=0xab;ar r_01<br>0x000000ab<br>&gt; "ae r_00,r_01,^="<br>&gt; ar r_01<br>0x000000a9<br>&gt; "ae 2,r_01,^="<br>&gt; ar r_01<br>0x000000ab
**++=** | reg | INC eq | reg = reg + 1 | &gt; ar r_00=4;ar r_00<br>0x00000004<br>&gt; "ae r_00,++="<br>&gt; ar r_00<br>0x00000005
**--=** | reg | DEC eq | reg = reg - 1 | &gt; ar r_00=4;ar r_00<br>0x00000004<br>&gt; "ae r_00,--="<br>&gt; ar r_00<br>0x00000003
**!=** | reg | NOT eq | reg = !reg | &gt; ar r_00=4;ar r_00<br>0x00000004<br>&gt; "ae r_00,!="<br>&gt; ar r_00<br>0x00000000<br>&gt; "ae r_00,!="<br>&gt; ar r_00<br>0x00000001
--- | --- | --- | --- | ----------------------------------------------
=[]<br>=[\*]<br>=[1]<br>=[2]<br>=[4]<br>=[8] | src,dst | poke |\*dst=src | <br>&gt; "ae 0xdeadbeef,0x10000,=[4],"<br><br>&gt; pxw 4@0x10000<br>0x00010000  0xdeadbeef                                ....<br><br>&gt; "ae 0x0,0x10000,=[4],"<br><br>&gt; pxw 4@0x10000<br>0x00010000  0x00000000
[]<br>[\*]<br>[1]<br>[2]<br>[4]<br>[8] | src | peek | stack=\*src | <br>&gt; w test@0x10000<br><br>&gt; "ae 0x10000,[4],"<br>0x74736574<br><br>&gt; ar r_00=0x10000<br><br>&gt; "ae r_00,[4],"<br>0x74736574
&#x7c;=[]<br>&#x7c;=[1]<br>&#x7c;=[2]<br>&#x7c;=[4]<br>&#x7c;=[8] | reg | nombre | code | &gt; <br>&gt;
SWAP |  | Swap | Swap two top elements | SWAP
PICK | n | Pick | Pick nth element<br> from the top of the stack | 2,PICK
RPICK | m | Reverse Pick | Pick nth element<br> from the base of the stack | 0,RPICK
DUP | | Duplicate | Duplicate top element in stack | DUP
NUM | | Numeric | If top element is a reference <br> (register name, label, etc),<br> dereference it and push its real value | NUM
CLEAR | | Clear | Clear stack | CLEAR
BREAK | | Break | Stops ESIL emulation | BREAK
GOTO | n | Goto | Jumps to Nth ESIL word | GOTO 5
TODO | | To Do | Stops execution<br> (reason: ESIL expression not completed) | TODO

### ESIL标志

ESIL VM有一个内置的状态标志，其是只读的，可用于将这些标志导入到基础的目标CPU标志位上。这是由于ESIL VM在每次操作后都计算所有标志位的改变，而目标CPU只在特定情况或特定语句下更新标志位。

内置标志位以`$`字符为前缀。

```
z      - zero flag，仅在操作结果为0是置位
b      - borrow, 需要指定具体的bit位（例子：$b4 - 检查是否向bit 4进行借位）
c      - carry, 类似上面（例子：$c7 - bit 7是否进位）
o      - overflow
p      - parity
r      - regsize ( asm.bits/8 )
s      - sign
ds     - delay slot 的状态
jt     - jump target
js     - jump target set
[0-9]* - 用于无副作用地设置标志位和寄存器，例如设置esil_cur, esil_old和esil_lastsz
         (例子： "$0,of,="用于重置overflow标志位)
```

## 语法和命令
目标opcode转化为以逗号分隔的ESIL表示：
```
xor eax, eax    ->    0,eax,=,1,zf,=
```
内存访问是通过`[]`操作符定义的：
```
mov eax, [0x80480]   ->   0x80480,[],eax,=
```
默认的操作数大小是根据目的操作数大小决定的：
```
movb $0, 0x80480     ->   0,0x80480,=[1]
```

`?`操作符将根据其参数的值决定是否执行`{}`中的表达式

1. 如果为0      -> 跳过
2. 若不为0      -> 执行

```
cmp eax, 123  ->   123,eax,==,$z,zf,=
jz eax        ->   zf,?{,eax,eip,=,}
```


如果想在某个条件下执行多个表达式，需将表达式放入`{}`中：
```
zf,?{,eip,esp,=[],eax,eip,=,$r,esp,-=,}
```

空格，换行以及其他字符将被忽略，因为在执行一个ESIL的第一个步骤就是移除空字符。
```
esil = r_str_replace (esil, " ", "", R_TRUE);
```

系统调用需要特殊处理，其通过表达式开头的`$`进行标识，可以选择传给它一个数值，用于指定系统调用。一个ESIL模拟器必须能够处理系统调用，参阅（r_esil_syscall)

## 非关联运算的参数顺序

如IRC上所讨论，目前的ESIL实现是类似下面这样的：

```
a,b,-      b - a
a,b,/=     b /= a
```
这种实现方式可读性更好，但对于栈不够友好。

### 特殊指令

NOP以空字符串进行表示，而如前文所述，系统调用则通过`$`命令标记。例如，`0x80, $`。其将ESIL机器中的模拟请求转发给一个回调函数，该回调函数为特定的OS/kernel实现了系统调用。

`TRAP`命令实现了陷阱指令，用于为非法指令、除0、内存读取错误或特定架构中所规定的情形抛出异常。

### 快速分析

以下是一个快查表，用于从ESIL字符串中检索信息。相关信息可能会在列表的第一个表达式中找到。
```
indexOf('[')    -> have memory references
indexOf("=[")   -> write in memory
indexOf("pc,=") -> modifies program counter (branch, jump, call)
indexOf("sp,=") -> modifies the stack (what if we found sp+= or sp-=?)
indexOf("=")    -> retrieve src and dst
indexOf(":")    -> unknown esil, raw opcode ahead
indexOf("$")    -> accesses internal esil vm flags ex: $z
indexOf("$")    -> syscall ex: 1,$
indexOf("TRAP") -> can trap
indexOf('++')   -> has iterator
indexOf('--')   -> count to zero
indexOf("?{")   -> conditional
equalsTo("")    -> empty string, aka nop (wrong, if we append pc+=x)
```

Common operations:
 * Check dstreg
 * Check srcreg
 * Get destinaion
 * Is jump
 * Is conditional
 * Evaluate
 * Is syscall

### CPU Flags

CPU标志通常在RReg配置文件中定义为单个的位寄存器，它们有时能在'flg'寄存器类型下找到

### 变量

VM变量相关内容：

1. 他们没有预定义的宽度，因此在之后可以很容易地扩展至128,256和512位，例如在MMX，SSE，AVX，Neon SIMD上。

2. 变量数目无限制，这是为了SSA-格式的兼容性。

3. 寄存器名字没有特殊语法，只是字符串而已。

4. 数值可以以RNum所支持的任意基数进行表示（dec，hex，oct，binary ...）。

5. 每个ESIL的后端都需要一个相关联的RReg配置，描述ESIL上寄存器应遵循的规范

### Bit组

What to do with them? What about bit arithmetics if use variables instead of registers?

### 算术操作

1. ADD ("+")
2. MUL ("\*")
3. SUB ("-")
4. DIV ("/")
5. MOD ("%")


### 位运算操作

1. AND  "&"
2. OR   "|"
3. XOR  "^"
4. SHL  "<<"
5. SHR  ">>"
6. ROL  "<<<"
7. ROR  ">>>"
8. NEG  "!"

### 浮点单元的支持

在撰写本文时，ESIL尚不支持FPU。但是您可以使用r2pipe实现对不受支持的指令的支持。
Eventually we will get proper support for multimedia and floating point.

### 在ESIL中处理x86的REP前缀

ESIL指定解析控制流的命令必须为大写。请记住，某些体系结构具有大写的寄存器名称。相应的寄存器配置文件中应注意不要重复使用以下任何内容：
```
3,SKIP   - skip N instructions. used to make relative forward GOTOs
3,GOTO   - goto instruction 3
LOOP     - alias for 0,GOTO
BREAK    - stop evaluating the expression
STACK    - dump stack contents to screen
CLEAR    - clear stack
```

#### 使用例子:

rep cmpsb
```
cx,!,?{,BREAK,},esi,[1],edi,[1],==,?{,BREAK,},esi,++,edi,++,cx,--,0,GOTO
```

### 未实现/未能处理的指令

这些用“ TODO”命令表示。充当"BREAK"的作用，但会显示警告消息，说明一条指令未实现且将不会被模拟。例如：

```
fmulp ST(1), ST(0)      =>      TODO,fmulp ST(1),ST(0)
```

### ESIL 反汇编示例:

```
[0x1000010f8]> e asm.esil=true
[0x1000010f8]> pd $r @ entry0
0x1000010f8    55           8,rsp,-=,rbp,rsp,=[8]
0x1000010f9    4889e5       rsp,rbp,=
0x1000010fc    4883c768     104,rdi,+=
0x100001100    4883c668     104,rsi,+=
0x100001104    5d           rsp,[8],rbp,=,8,rsp,+=
0x100001105    e950350000   0x465a,rip,= ;[1]
0x10000110a    55           8,rsp,-=,rbp,rsp,=[8]
0x10000110b    4889e5       rsp,rbp,=
0x10000110e    488d4668     rsi,104,+,rax,=
0x100001112    488d7768     rdi,104,+,rsi,=
0x100001116    4889c7       rax,rdi,=
0x100001119    5d           rsp,[8],rbp,=,8,rsp,+=
0x10000111a    e93b350000   0x465a,rip,= ;[1]
0x10000111f    55           8,rsp,-=,rbp,rsp,=[8]
0x100001120    4889e5       rsp,rbp,=
0x100001123    488b4f60     rdi,96,+,[8],rcx,=
0x100001127    4c8b4130     rcx,48,+,[8],r8,=
0x10000112b    488b5660     rsi,96,+,[8],rdx,=
0x10000112f    b801000000   1,eax,=
0x100001134    4c394230     rdx,48,+,[8],r8,==,cz,?=
0x100001138    7f1a         sf,of,!,^,zf,!,&,?{,0x1154,rip,=,} ;[2]
0x10000113a    7d07         of,!,sf,^,?{,0x1143,rip,} ;[3]
0x10000113c    b8ffffffff   0xffffffff,eax,= ;  0xffffffff
0x100001141    eb11         0x1154,rip,= ;[2]
0x100001143    488b4938     rcx,56,+,[8],rcx,=
0x100001147    48394a38     rdx,56,+,[8],rcx,==,cz,?=
```

### 自省(introspection)

为减轻ESIL解析的压力，我们需要有一个方法，能执行自省表达式以提取我们所需的数据。例如我们想获取jump的目的地址，ESIL表达式的解析器应提供一个API以完成这个工作，使得分析表达式提取信息变得更容易。

```
>  ao~esil,opcode
opcode: jmp 0x10000465a
esil: 0x10000465a,rip,=
```
我们需要有一个方式能获取'rip'的数值，这个例子很简单。但这里还有一个更复杂的例子，类似条件分支。我们需要能从表达式中获取：

- opcode type
- destination of a jump
- condition depends on
- all regs modified (write)
- all regs accessed (read)

### API 钩子(API hooker)

对于模拟执行来说能在解析器中设置hook是很重要的，如此一来我们可通过扩展该程序实现分析，而不必一次又一次地修改它。也就是说，每次要执行指令操作时，都会调用一个用户挂钩。例如，它可以用于确定'RIP'是否要更改，或者指令是否会更新堆栈。
之后，我们可以将该回调函数一分为多，以拥有event-based的分析API，比如在javascript中：
```
esil.on('regset', function(){..
esil.on('syscall', function(){esil.regset('rip'
```

若要了解API，参见`hook_flag_read()`, `hook_execute()`和`hook_mem_read()`。
For the API, see the functions `hook_flag_read()`, `hook_execute()` and `hook_mem_read()`. 若想覆盖操作，则相应的回调函数应该返回true或1。例如，拒绝对某块内存区域的读取，或者避免内存写入，从而使其变为只读。
若想跟踪ESIL表示的解析过程，则回调函数返回false或0即可。

其他需要绑定到外部功能才能起作用的操作，在本例中就是`r_ref`和`r_io`，必须在ESIL VM初始化时定义它们。

* Io Get/Set
  ```
  Out ax, 44
  44,ax,:ou
  ```
* Selectors (cs,ds,gs...)
  ```
  Mov eax, ds:[ebp+8]
  Ebp,8,+,:ds,eax,=
  ```

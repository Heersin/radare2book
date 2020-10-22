# 循环

自动化最常见的一个工作就是循环地处理某些东西，在radare2中有多种途径实现该任务。

可以在flags上进行loop
```
@@ flagname-regex
```

比如，我们想要通过`afi`命令获得函数信息
```
[0x004047d6]> afi
#
offset: 0x004047d0
name: entry0
size: 42
realsz: 42
stackframe: 0
call-convention: amd64
cyclomatic-complexity: 1
bits: 64
type: fcn [NEW]
num-bbs: 1
edges: 0
end-bbs: 1
call-refs: 0x00402450 C
data-refs: 0x004136c0 0x00413660 0x004027e0
code-xrefs:
data-xrefs:
locals:0
args: 0
diff: type: new
[0x004047d6]>
```
现在，比如说我们想要查看目前发现的所有函数中某个特定字段（如上面输出里的字段），可以通过在函数flag空间（带`fcn.`前缀的那些函数）上进行loop达到该目的。
```
[0x004047d6]> afi @@ fcn.* ~name
```
该命令将会从`afi`命令的输出中提取`name`字段信息，对每个匹配了正则`fcn.*`的函数flag都会如此。

同样还可以在包含一系列偏移的列表上进行loop，使用如下形式：
```
@@=1 2 3 ... N
```
例如，我们想查看2个偏移地址上的opcode信息：当前的，以及当前地址+2这两个位置：

```
[0x004047d6]> ao @@=$$ $$+2
address: 0x4047d6
opcode: mov rdx, rsp
prefix: 0
bytes: 4889e2
refptr: 0
size: 3
type: mov
esil: rsp,rdx,=
stack: null
family: cpu
address: 0x4047d8
opcode: loop 0x404822
prefix: 0
bytes: e248
refptr: 0
size: 2
type: cjmp
esil: 1,rcx,-=,rcx,?{,4212770,rip,=,}
jump: 0x00404822
fail: 0x004047da
stack: null
cond: al
family: cpu
[0x004047d6]>
```
注意，我们正使用`$$`变量，其求值的结果正是当前偏移量。还要注意的是`$$+2`是在loop过程之前求值的，因此我们才可以使用这一简单的算术表达式。


进行loop的第三种方法是从文件加载偏移量，该文件每一行都包含一个偏移量。
```
[0x004047d0]> ?v $$ > offsets.txt
[0x004047d0]> ?v $$+2 >> offsets.txt
[0x004047d0]> !cat offsets.txt
4047d0
4047d2
[0x004047d0]> pi 1 @@.offsets.txt
xor ebp, ebp
mov r9, rdx
```

radare2同样提供了多种`foreach`构造器进行loop，最有用的一个就是用于遍历函数中的所有指令。
```
[0x004047d0]> pdf
╒ (fcn) entry0 42
│; UNKNOWN XREF from 0x00400018 (unk)
│; DATA XREF from 0x004064bf (sub.strlen_460)
│; DATA XREF from 0x00406511 (sub.strlen_460)
│; DATA XREF from 0x0040b080 (unk)
│; DATA XREF from 0x0040b0ef (unk)
│0x004047d0  xor ebp, ebp
│0x004047d2  mov r9, rdx
│0x004047d5  pop rsi
│0x004047d6  mov rdx, rsp
│0x004047d9  and rsp, 0xfffffffffffffff0
│0x004047dd  push rax
│0x004047de  push rsp
│0x004047df  mov r8, 0x4136c0
│0x004047e6  mov rcx, 0x413660      ; "AWA..AVI..AUI..ATL.%.. "
0A..AVI..AUI.
│0x004047ed  mov rdi, main          ; "AWAVAUATUH..S..H...." @
0
│0x004047f4  call sym.imp.__libc_start_main
╘0x004047f9  hlt
[0x004047d0]> pi 1 @@i
mov r9, rdx
pop rsi
mov rdx, rsp
and rsp, 0xfffffffffffffff0
push rax
push rsp
mov r8, 0x4136c0
mov rcx, 0x413660
mov rdi, main
call sym.imp.__libc_start_main
hlt
```
在这个例子中对当前函数（entry0）中的每条指令执行`pi 1`命令。
这里同样有一些其他的选项（并非完整列表，通过`@@?`了解更多信息）
 - `@@k sdbquery` - 遍历sdbquery返回的所有偏移量
 - `@@t`- 在所有线程上进行遍历（参见dp）
 - `@@b` - 在当前函数的所有基本块上进行遍历（参见afb）
 - `@@f` - 在所有指令上进行遍历 (参见aflq)

最后一种类型的遍历允许对任何预定义的类型进行遍历

 - symbols
 - imports
 - registers
 - threads
 - comments
 - functions
 - flags

这是通过@@@命令做到的，前面那个列出函数信息的例子也可以通过`@@@`命令完成：

```
[0x004047d6]> afi @@@ functions ~name
```
这个命令将会从`afi`的输出中提取`name`字段，并会输出一长串函数名，我们可以仅选择第二列。
若要去掉每行中重复的`name`：
```
[0x004047d6]> afi @@@ functions ~name[1]
```

**请当心, @@@ 与 JSON commands 不兼容**
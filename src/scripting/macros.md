# 宏(Macro)

除了序列化命令和循环之外，radare2中允许简单的宏定义，例如：
```
[0x00404800]> (qwe; pd 4; ao)
```

这个指令定义了`qwe`宏，该宏代表先后运行`pd 4`和`ao`两个命令。
调用宏很简单，只需遵照`.(macro)`这样的语法规则。

```
[0x00404800]> (qwe; pd 4; ao)
[0x00404800]> .(qwe)
0x00404800  mov eax, 0x61e627      ; "tab"
0x00404805  push rbp
0x00404806  sub rax, section_end.LOAD1
0x0040480c  mov rbp, rsp

address: 0x404800
opcode: mov eax, 0x61e627
prefix: 0
bytes: b827e66100
ptr: 0x0061e627
refptr: 0
size: 5
type: mov
esil: 6415911,rax,=
stack: null
family: cpu
[0x00404800]>
```

`(*`可以列出所有宏:
```
[0x00404800]> (*
(qwe ; pd 4; ao)
```

如果想要移除某个宏，需在宏名之前添加`-`号：
```
[0x00404800]> (-qwe)
Macro 'qwe' removed.
[0x00404800]>
```

此外也可以定义带参数的宏，这些宏在一些需要脚本化的场景下用起来会很方便。
创建带参数的宏很简单，在定义时将宏参数加在后面即可:
```
[0x00404800]
[0x004047d0]> (foo x y; pd $0; s +$1)
[0x004047d0]> .(foo 5 6)
;-- entry0:
0x004047d0      xor ebp, ebp
0x004047d2      mov r9, rdx
0x004047d5      pop rsi
0x004047d6	mov rdx, rsp
0x004047d9	and rsp, 0xfffffffffffffff0
[0x004047d6]>
```
如例子所示，在宏内是通过index获取参数的，index从0开始计数： $0, $1, ...

# 别名

radare2支持别名(alias)，对于执行频率很高的命令，使用别名可以节省时间。别名相关的用法在`$?`输出的结果中可以看到。

最常见的用法就是: `$alias=cmd`
```
[0x00404800]> $disas=pdf
```

上面的命令为`pdf`创建了一个别名`disas`，下面的例子中将会输出main函数的反汇编结果。
```
[0x00404800]> $disas @ main
```

除了为命令创建别名，还可以为一段文本创建别名，方便输出文本内容。
```
[0x00404800]> $my_alias=$test input
[0x00404800]> $my_alias
test input
```


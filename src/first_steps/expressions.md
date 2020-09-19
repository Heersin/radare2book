## 表达式

radare2中数学表达式的值位长为64位，可以以不同格式进行显示，可以用于所有接受数字参数的命令，或进行比较。表达式中可以使用传统的算术运算，以及二进制数和布尔值。
```
[0xb7f9d810]> ?vi 0x8048000
134512640
[0xv7f9d810]> ?vi 0x8048000+34
134512674
[0xb7f9d810]> ?vi 0x8048000+0x34
134512692
[0xb7f9d810]> ? 1+2+3-4*3
hex     0xfffffffffffffffa
octal   01777777777777777777772
unit    17179869184.0G
segment fffff000:0ffa
int64   -6
string  "\xfa\xff\xff\xff\xff\xff\xff\xff"
binary  0b1111111111111111111111111111111111111111111111111111111111111010
fvalue: -6.0
float:  nanf
double: nan
trits   0t11112220022122120101211020120210210211201
```
支持的算术操作有：

 *  \+ : addition
 *  \- : subtraction
 *  \* : multiplication
 *  / : division
 *  % : modulus
 *  \> : shift right
 *  < : shift left

```
[0x00000000]> ?vi 1+2+3
6
```

使用逻辑OR时需要用用引号进行标记，避免执行`|`管道:
```
[0x00000000]> "? 1 | 2"
hex     0x3
octal   03
unit    3
segment 0000:0003
int32   3
string  "\x03"
binary  0b00000011
fvalue: 2.0
float:  0.000000f
double: 0.000000
trits   0t10
```

可以以不同格式显示结果:
```
0x033   : hexadecimal can be displayed
3334    : decimal
sym.fo  : resolve flag offset
10K     : KBytes  10*1024
10M     : MBytes  10*1024*1024
```

可以结合变量和seek位置创建复杂的表达式。

使用`?$?`命令可以列出所有可用的命令，或者你也可以参考本书的refcard章节

```
$$    here (the current virtual seek)
$l    opcode length
$s    file size
$j    jump address (e.g. jmp 0x10, jz 0x10 => 0x10)
$f    jump fail address (e.g. jz 0x10 => next instruction)
$m    opcode memory reference (e.g. mov eax,[0x10] => 0x10)
$b    block size
```

更多的一些例子:
```
[0x4A13B8C0]> ? $m + $l
140293837812900 0x7f98b45df4a4 03771426427372244 130658.0G 8b45d000:04a4 140293837812900 10100100 140293837812900.0 -0.000000
```
```
[0x4A13B8C0]> pd 1 @ +$l
0x4A13B8C2   call 0x4a13c000
```

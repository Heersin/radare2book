## Rax2

`rax2`是radare框架中的一个小工具，其设计目的是为shell提供一个小型的表达式计算器。其可用于浮点数、十六进制字节的进制转换，十六进制转ascii，八进制转整数等。若未传入任何参数还可作为一个计算器shell使用。

底下是rax2的帮助信息，该工具可以用于命令行中，也可以从标准输入流（stdin）中读取数值交互式地使用，总之可以将其视为一个多进制计算器。

在r2中，rax2的功能归在`?`命令下，例如：

```
[0x00000000]> ? 3+4
```

正如所见，数值表达式可以包含诸如加法、减法这样的数学表达式，此外也支持括号进行分组操作。

不同的表达式语法定义了数字的基数，例如：

* 3 : decimal, base 10
* 0xface : hexadecimal, base 16
* 0472 : octal, base 8
* 2M : units, 2 megabytes
* ...

下面就是`rax2 -h`输出的帮助信息，里面包含了更多的语法规则。

```
$ rax2 -h
Usage: rax2 [options] [expr ...]
  =[base]                      ;  rax2 =10 0x46 -> output in base 10
  int     ->  hex              ;  rax2 10
  hex     ->  int              ;  rax2 0xa
  -int    ->  hex              ;  rax2 -77
  -hex    ->  int              ;  rax2 0xffffffb3
  int     ->  bin              ;  rax2 b30
  int     ->  ternary          ;  rax2 t42
  bin     ->  int              ;  rax2 1010d
  ternary ->  int              ;  rax2 1010dt
  float   ->  hex              ;  rax2 3.33f
  hex     ->  float            ;  rax2 Fx40551ed8
  oct     ->  hex              ;  rax2 35o
  hex     ->  oct              ;  rax2 Ox12 (O is a letter)
  bin     ->  hex              ;  rax2 1100011b
  hex     ->  bin              ;  rax2 Bx63
  ternary ->  hex              ;  rax2 212t
  hex     ->  ternary          ;  rax2 Tx23
  raw     ->  hex              ;  rax2 -S < /binfile
  hex     ->  raw              ;  rax2 -s 414141
  -l                           ;  append newline to output (for -E/-D/-r/..
  -a      show ascii table     ;  rax2 -a
  -b      bin -> str           ;  rax2 -b 01000101 01110110
  -B      str -> bin           ;  rax2 -B hello
  -d      force integer        ;  rax2 -d 3 -> 3 instead of 0x3
  -e      swap endianness      ;  rax2 -e 0x33
  -D      base64 decode        ;
  -E      base64 encode        ;
  -f      floating point       ;  rax2 -f 6.3+2.1
  -F      stdin slurp code hex ;  rax2 -F < shellcode.[c/py/js]
  -h      help                 ;  rax2 -h
  -i      dump as C byte array ;  rax2 -i < bytes
  -k      keep base            ;  rax2 -k 33+3 -> 36
  -K      randomart            ;  rax2 -K 0x34 1020304050
  -L      bin -> hex(bignum)   ;  rax2 -L 111111111 # 0x1ff
  -n      binary number        ;  rax2 -n 0x1234 # 34120000
  -N      binary number        ;  rax2 -N 0x1234 # \x34\x12\x00\x00
  -r      r2 style output      ;  rax2 -r 0x1234
  -s      hexstr -> raw        ;  rax2 -s 43 4a 50
  -S      raw -> hexstr        ;  rax2 -S < /bin/ls > ls.hex
  -t      tstamp -> str        ;  rax2 -t 1234567890
  -x      hash string          ;  rax2 -x linux osx
  -u      units                ;  rax2 -u 389289238 # 317.0M
  -w      signed word          ;  rax2 -w 16 0xffff
  -v      version              ;  rax2 -v
```

一些例子
```
$ rax2 3+0x80
0x83
```
```
$ rax2 0x80+3
131
```
```
$ echo 0x80+3 | rax2
131
```
```
$ rax2 -s 4142
AB
```
```
$ rax2 -S AB
4142
```
```
$ rax2 -S < bin.foo
...
```
```
$ rax2 -e 33
0x21000000
```
```
$ rax2 -e 0x21000000
33
```
```
$ rax2 -K 90203010
+--[0x10302090]---+
|Eo. .            |
| . . . .         |
|      o          |
|       .         |
|        S        |
|                 |
|                 |
|                 |
|                 |
+-----------------+
```
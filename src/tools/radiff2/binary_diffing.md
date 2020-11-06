# 二进制文件比较

本章节基于 http://radare.today 上的文章"[binary diffing](http://radare.today/binary-diffing/)"

若未指定任何参数，`radiff2`默认会显示不同的字节数据，以及这些字节对应的偏移量。
```
$ radiff2 genuine cracked
0x000081e0 85c00f94c0 => 9090909090 0x000081e0
0x0007c805 85c00f84c0 => 9090909090 0x0007c805

$ rasm2 -d 85c00f94c0
test eax, eax
sete al
```
这两个语句都被NOP指令覆盖了。

在需要批量处理的时候，分析者可能希望站在更高层次检查文件之间的差异，因此radare2提供了计算两个文件差异程度的选项`-s`:
```
$ radiff2 -s /bin/true /bin/false
similarity: 0.97
distance: 743
```

若是想要一个更加具体的量化数据，可以用`-c`计算二者之间的差异数量。
```
$ radiff2 -c genuine cracked
2
```

如果你不确定目前分析的文件之间是否相似，可以使用`-C`进行检查，检查它们之间是否存在匹配的函数。在该模式下会输出3列信息:"第一个文件中的偏移量"，"匹配程度","第二个文件的偏移量"。
```
$ radiff2 -C /bin/false /bin/true
  entry0  0x4013e8 |   MATCH  (0.904762) | 0x4013e2  entry0
  sym.imp.__libc_start_main  0x401190 |   MATCH  (1.000000) | 0x401190  sym.imp.__libc_start_main
  fcn.00401196  0x401196 |   MATCH  (1.000000) | 0x401196  fcn.00401196
  fcn.0040103c  0x40103c |   MATCH  (1.000000) | 0x40103c  fcn.0040103c
  fcn.00401046  0x401046 |   MATCH  (1.000000) | 0x401046  fcn.00401046
  fcn.000045e0   24 0x45e0 | UNMATCH  (0.916667) | 0x45f0    24 fcn.000045f0
  ...
```
此外，还可加上`-A`在radiff2比较之前调用r2对二进制文件进行`aaa`分析，且能像下面这样指定架构:
```
$ radiff2 -AC -a x86 /bin/true /bin/false | grep UNMATCH
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze len bytes of instructions for references (aar)
[x] Analyze function calls (aac)
[ ] [*] Use -AA or aaaa to perform additional experimental analysis.
[x] Constructing a function name for fcn.* and sym.func.* functions (aan))
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze len bytes of instructions for references (aar)
[x] Analyze function calls (aac)
[ ] [*] Use -AA or aaaa to perform additional experimental analysis.
[x] Constructing a function name for fcn.* and sym.func.* functions (aan))
                        sub.fileno_500   86 0x4500 | UNMATCH  (0.965116) | 0x4510    86 sub.fileno_510
                    sub.__freading_4c0   59 0x44c0 | UNMATCH  (0.949153) | 0x44d0    59 sub.__freading_4d0
                        sub.fileno_440  120 0x4440 | UNMATCH  (0.200000) | 0x4450   120 sub.fileno_450
                     sub.setlocale_fa0   64 0x3fa0 | UNMATCH  (0.104651) | 0x3fb0    64 sub.setlocale_fb0
                          fcn.00003a50  120 0x3a50 | UNMATCH  (0.125000) | 0x3a60   120 fcn.00003a60
```

现在r2还添加了一个新特性：使用`-g`选项将差异图形化，如[DarunGrim](http://www.darungrim.org/)。可以在此选项后加上一个要分析的symbol name，或者，当同样的函数在两个文件中使用了不同的名字，也可以指定两个偏移量进行比较。
例如，`radiff2 -g main /bin/true /bin/false | xdot -`将会显示Unix中`true`和`false`两个程序中`main()`函数的不同之处。可以试着与`radiff2 -g main /bin/false /bin/true`（注意观察二者的参数顺序）的结果进行比较，会得到两个版本。
下面的图片就是该结果

![/bin/true vs /bin/false](../../pics/true_false.png)

黄色的部分代表有一些偏移量上二者不匹配，灰色部分则代表完美匹配，红色代表二者之间存在较大差异。如果你仔细观察，会发现左边的图片中最后是`mov edi, 0x1; call sym.imp.exit`，而右边则是`xor edi, edi; call sym.imp.exit`。

二进制级别的比较在逆向中是非常重要的一部分，可以用于分析[安全补丁](https://en.wikipedia.org/wiki/Patch_Tuesday)，被感染的二进制文件，固件变更等等...

我们此处仅仅展示了部分的radiff代码差异分析的功能，而radare2还支持更多种类的比较：字节级、差异相似度，以及更多待加入的比较类型。

我们计划在r2里实现更多的二进制比较算法，添加ASCII字符画风格的比较，并与其它工具能更好地整合在一起。


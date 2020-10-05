# 测试插件

该插件被用于rasm2和r2中，可以用如下命令确认该插件被正确加载了：
```
$ rasm2 -L | grep mycpu
_d  mycpu        My CPU disassembler  (LGPL3)
```

打开一个使用'mycpu'架构的新文件，然后往里面随机地写入一些代码：

```
$ r2 -
 -- I endians swap
[0x00000000]> e asm.arch=mycpu
[0x00000000]> woR
[0x00000000]> pd 10
           0x00000000    888e         mov r8, 14
           0x00000002    b2a5         ifnot r10, r5
           0x00000004    3f67         ret
           0x00000006    7ef6         bl r15, r6
           0x00000008    2701         xor r0, 1
           0x0000000a    9826         mov r2, 6
           0x0000000c    478d         xor r8, 13
           0x0000000e    6b6b         store r6, 11
           0x00000010    1382         add r8, r2
           0x00000012    7f15         ret
```
好耶！ 插件工作正常.. 下面这个单行命令也是能正常起作用的！

```
r2 -nqamycpu -cwoR -cpd' 10' -
```


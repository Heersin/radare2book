## 入口点（EP）

使用rabin2的`-e`选项可以输出程序的入口点，底下是两个例子:
```
$ rabin2 -e /bin/ls
[Entrypoints]
vaddr=0x00005310 paddr=0x00005310 baddr=0x00000000 laddr=0x00000000 haddr=0x00000018 type=program

1 entrypoints

$ rabin2 -er /bin/ls
fs symbols
f entry0 1 @ 0x00005310
f entry0_haddr 1 @ 0x00000018
s entry0
```

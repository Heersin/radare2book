IOLI 0x00
=========

第一个IOLI crackme，也是最简单的一个。

```
$ ./crackme0x00
IOLI Crackme Level 0x00
Password: 1234
Invalid Password!
```

首先检查一下口令是否就是存在于文件内的字符串。在这个例子中，无需进行任何反汇编操作，只需要使用rabin2 -z搜索二进制文件内的字符串。

```
$ rabin2 -z ./crackme0x00
[Strings]
nth paddr      vaddr      len size section type  string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00000568 0x08048568 24  25   .rodata ascii IOLI Crackme Level 0x00\n
1   0x00000581 0x08048581 10  11   .rodata ascii Password: 
2   0x0000058f 0x0804858f 6   7    .rodata ascii 250382
3   0x00000596 0x08048596 18  19   .rodata ascii Invalid Password!\n
4   0x000005a9 0x080485a9 15  16   .rodata ascii Password OK :)\n
```

现在我们知道这些都是些什么节区了，这正是程序运行时所显示的标头信息。
```
nth paddr      vaddr      len size section type  string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00000568 0x08048568 24  25   .rodata ascii IOLI Crackme Level 0x00\n
```

从这里面可以获得口令的一些线索和提示：
```
1   0x00000581 0x08048581 10  11   .rodata ascii Password: 
```

当输入错误口令时会产生如下错误信息。
```
3   0x00000596 0x08048596 18  19   .rodata ascii Invalid Password!\n
```

这个则是口令正确时产生的提示信息。
```
4   0x000005a9 0x080485a9 15  16   .rodata ascii Password OK :)\n
```

那么，底下这串数字又是什么，它是一个字符串，但运行程序时还并没有见到它。

```
2   0x0000058f 0x0804858f 6   7    .rodata ascii 250382
```

让我们试一下：
```
$ ./crackme0x00
IOLI Crackme Level 0x00
Password: 250382
Password OK :)
```

至此我们知道250328就是该口令，完成crackme。
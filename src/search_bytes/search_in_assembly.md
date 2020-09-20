## 汇编指令搜索

如果要搜索一个汇编操作码， 可以使用`/a`命令， 下面的`/ad/ jmp [esp]`命令搜索了对应的汇编助记符：

```
[0x00404888]> /ad/ jmp qword [rdx]
f hit_0 @ 0x0040e50d   # 2: jmp qword [rdx]
f hit_1 @ 0x00418dbb   # 2: jmp qword [rdx]
f hit_2 @ 0x00418fcb   # 3: jmp qword [rdx]
f hit_3 @ 0x004196ab   # 6: jmp qword [rdx]
f hit_4 @ 0x00419bf3   # 3: jmp qword [rdx]
f hit_5 @ 0x00419c1b   # 3: jmp qword [rdx]
f hit_6 @ 0x00419c43   # 3: jmp qword [rdx]
```

`/a jmp eax`命令将该字符串对应的汇编代码转为机器码，然后搜索该机器码：
```
[0x00404888]> /a jmp eax
hits: 1
0x004048e7 hit3_0 ffe00f1f8000000000b8
```
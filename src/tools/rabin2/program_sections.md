## 程序段（Section）

使用`-S`选项调用Rabin2可以获得一个可执行文件的所有段信息，包含每个段的Index，offset，size，alignment，type和permission信息，下面这个例子就展示了这个功能：
```
$ rabin2 -S /bin/ls
[Sections]
00 0x00000000     0 0x00000000     0 ----
01 0x00000238    28 0x00000238    28 -r-- .interp
02 0x00000254    32 0x00000254    32 -r-- .note.ABI_tag
03 0x00000278   176 0x00000278   176 -r-- .gnu.hash
04 0x00000328  3000 0x00000328  3000 -r-- .dynsym
05 0x00000ee0  1412 0x00000ee0  1412 -r-- .dynstr
06 0x00001464   250 0x00001464   250 -r-- .gnu.version
07 0x00001560   112 0x00001560   112 -r-- .gnu.version_r
08 0x000015d0  4944 0x000015d0  4944 -r-- .rela.dyn
09 0x00002920  2448 0x00002920  2448 -r-- .rela.plt
10 0x000032b0    23 0x000032b0    23 -r-x .init
11 0x000032d0  1648 0x000032d0  1648 -r-x .plt
12 0x00003940    24 0x00003940    24 -r-x .plt.got
13 0x00003960 73931 0x00003960 73931 -r-x .text
14 0x00015a2c     9 0x00015a2c     9 -r-x .fini
15 0x00015a40 20201 0x00015a40 20201 -r-- .rodata
16 0x0001a92c  2164 0x0001a92c  2164 -r-- .eh_frame_hdr
17 0x0001b1a0 11384 0x0001b1a0 11384 -r-- .eh_frame
18 0x0001e390     8 0x0021e390     8 -rw- .init_array
19 0x0001e398     8 0x0021e398     8 -rw- .fini_array
20 0x0001e3a0  2616 0x0021e3a0  2616 -rw- .data.rel.ro
21 0x0001edd8   480 0x0021edd8   480 -rw- .dynamic
22 0x0001efb8    56 0x0021efb8    56 -rw- .got
23 0x0001f000   840 0x0021f000   840 -rw- .got.plt
24 0x0001f360   616 0x0021f360   616 -rw- .data
25 0x0001f5c8     0 0x0021f5e0  4824 -rw- .bss
26 0x0001f5c8   232 0x00000000   232 ---- .shstrtab
```

通过`-Sr`选项，rabin2会标记每个段的开头和结尾，然后将剩下的信息作为注释输出：
```
$ rabin2 -Sr /bin/ls | head
fs sections
S 0x00000000 0x00000000 0x00000000 0x00000000  0
f section. 0 0x00000000
f section_end. 1 0x00000000
CC section 0 va=0x00000000 pa=0x00000000 sz=0 vsz=0 rwx=----  @ 0x00000000
S 0x00000238 0x00000238 0x0000001c 0x0000001c .interp 4
f section..interp 28 0x00000238
f section_end..interp 1 0x00000254
CC section 1 va=0x00000238 pa=0x00000238 sz=28 vsz=28 rwx=-r-- .interp @ 0x00000238
S 0x00000254 0x00000254 0x00000020 0x00000020 .note.ABI_tag 4
```


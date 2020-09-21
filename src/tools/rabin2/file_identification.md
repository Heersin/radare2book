## 文件属性识别

文件类型信息的识别是通过`-I`参数完成的。使用此选项，rabin2将信息输出该文件的二进制信息，例如其编码，使用的字节序，文件内的类，操作系统类型等：
```
$ rabin2 -I /bin/ls
arch     x86
binsz    128456
bintype  elf
bits     64
canary   true
class    ELF64
crypto   false
endian   little
havecode true
intrp    /lib64/ld-linux-x86-64.so.2
lang     c
linenum  false
lsyms    false
machine  AMD x86-64 architecture
maxopsz  16
minopsz  1
nx       true
os       linux
pcalign  0
pic      true
relocs   false
relro    partial
rpath    NONE
static   false
stripped true
subsys   linux
va       true
```

若要使rabin2输出radare2核心程序可理解的信息，可以使用`-Ir`选项：
```
$ rabin2 -Ir /bin/ls
e cfg.bigendian=false
e asm.bits=64
e asm.dwarf=true
e bin.lang=c
e file.type=elf
e asm.os=linux
e asm.arch=x86
e asm.pcalign=0
```


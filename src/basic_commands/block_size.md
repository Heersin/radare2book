## 块大小

块大小决定了在未显式指定参数时radare2进行分析的字节数，可以通过指定一个数字参数临时改变该数值，例如：`px 20`：

```
[0xB7F9D810]> b?
| Usage: b[f] [arg]
| Get/Set block size
| b 33     set block size to 33
| b eip+4  numeric argument can be an expression
| b        display current block size
| b+3      increase blocksize by 3
| b-16     decrease blocksize by 16
| b*       display current block size in r2 command
| bf foo   set block size to flag size
| bj       display block size information in JSON
| bm 1M    set max block size
```

`b`命令用于修改块大小：

```
[0x00000000]> b 0x100   ; block size = 0x100
[0x00000000]> b+16      ;  ... = 0x110
[0x00000000]> b-32      ;  ... = 0xf0
```

`bf`命令用于修改一个flag所对应的块大小，比如对于符号（symbols）来说，其块大小代表函数的大小。
```
[0x00000000]> bf sym.main    ; block size = sizeof(sym.main)
[0x00000000]> pd @ sym.main  ; disassemble sym.main
...
```

用`pdf`这条命令即可以将两个操作结合：

```
[0x00000000]> pdf @ sym.main
```


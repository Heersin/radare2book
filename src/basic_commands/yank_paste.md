## 复制/粘贴

Radare2有一个内置的剪贴板，用于保存和写入从当前io层加载的部分内存。

可以用`y`命令对该剪贴板进行操作

两个基本的操作为：

* copy (yank)
* paste

复制操作会将N个字节（由参数给出）读取至剪贴板中，可以用`yy`命令将之前读取的内容粘贴至文件中。

可以在光标模式（`Vc`）中用`y`和`Y`进行复制和粘贴，其分别对应命令行界面中的`y`和`yy`。

```
[0x00000000]> y?
Usage: y[ptxy] [len] [[@]addr]   # See wd? for memcpy, same as 'yf'.
| y!              open cfg.editor to edit the clipboard
| y 16 0x200      copy 16 bytes into clipboard from 0x200
| y 16 @ 0x200    copy 16 bytes into clipboard from 0x200
| y 16            copy 16 bytes into clipboard
| y               show yank buffer information (srcoff len bytes)
| y*              print in r2 commands what's been yanked
| yf 64 0x200     copy file 64 bytes from 0x200 from file
| yfa file copy   copy all bytes from file (opens w/ io)
| yfx 10203040    yank from hexpairs (same as ywx)
| yj              print in JSON commands what's been yanked
| yp              print contents of clipboard
| yq              print contents of clipboard in hexpairs
| ys              print contents of clipboard as string
| yt 64 0x200     copy 64 bytes from current seek to 0x200
| ytf file        dump the clipboard to given file
| yw hello world  yank from string
| ywx 10203040    yank from hexpairs (same as yfx)
| yx              print contents of clipboard in hexadecimal
| yy 0x3344       paste clipboard
| yz [len]        copy nul-terminated string (up to blocksize) into clipboard
```

示例:

```
[0x00000000]> s 0x100    ; seek at 0x100
[0x00000100]> y 100      ; yanks 100 bytes from here
[0x00000200]> s 0x200    ; seek 0x200
[0x00000200]> yy         ; pastes 100 bytes
```

可以用`yt`命令在单行内进行复制和粘贴操作。语法如下：

```
[0x4A13B8C0]> x
   offset   0 1  2 3  4 5  6 7  8 9  A B  0123456789AB
0x4A13B8C0, 89e0 e839 0700 0089 c7e8 e2ff ...9........
0x4A13B8CC, ffff 81c3 eea6 0100 8b83 08ff ............
0x4A13B8D8, ffff 5a8d 2484 29c2           ..Z.$.).

[0x4A13B8C0]> yt 8 0x4A13B8CC @ 0x4A13B8C0

[0x4A13B8C0]> x
   offset   0 1  2 3  4 5  6 7  8 9  A B  0123456789AB
0x4A13B8C0, 89e0 e839 0700 0089 c7e8 e2ff ...9........
0x4A13B8CC, 89e0 e839 0700 0089 8b83 08ff ...9........
0x4A13B8D8, ffff 5a8d 2484 29c2           ..Z.$.).
```

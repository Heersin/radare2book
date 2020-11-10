# 可视化模式

可视化模式是radare2中一个对用户比较友好的命令行界面。其具有基本的导航功能，可以用光标选择字节，并提供了一系列的快捷键简化调试时的操作。
使用`V`命令可进入该模式，在该模式中按下`q`可退回到原本的命令行界面中。

## 导航

支持使用HJKL键或方向键与PageUp/PageDown进行导航，也支持通常的Home/End键。如vim一样，可以在前面加上数字将动作重复多次，比如说`5j`将会下移5行，`2l`则会右移2个字符宽度。

![Visual Mode](visualmode.png)

## 切换输出模式

可以在可视化模式中循环地切换"输出模式"，默认的循环顺序为:

↻ **Hexdump panel** -> **Disassembly panel** → **Debugger panel** → **Hexadecimal words dump panel** → **Hex-less hexdump panel** → **Op analysis color map panel** → **Annotated hexdump panel** ↺.

另外值得一提的是，界面顶部包含了当前所使用的输出命令，例如反汇编界面中显示的是：

```
[0x00404890 16% 120 /bin/ls]> pd $r @ entry0
```

## 获取帮助

按下`?`可以查看可视化模式中定义的快捷键的帮助信息:
```
Visual mode help:
 ?        show this help
 ??       show the user-friendly hud
 %        in cursor mode finds matching pair, or toggle autoblocksz
 @        redraw screen every 1s (multi-user view)
 ^        seek to the begining of the function
 !        enter into the visual panels mode
 _        enter the flag/comment/functions/.. hud (same as VF_)
 =        set cmd.vprompt (top row)
 |        set cmd.cprompt (right column)
 .        seek to program counter
 \        toggle visual split mode
 "        toggle the column mode (uses pC..)
 /        in cursor mode search in current block
 :cmd     run radare command
 ;[-]cmt  add/remove comment
 0        seek to beginning of current function
 [1-9]    follow jmp/call identified by shortcut (like ;[1])
 ,file    add a link to the text file
 /*+-[]   change block size, [] = resize hex.cols
 </>      seek aligned to block size (seek cursor in cursor mode)
 a/A      (a)ssemble code, visual (A)ssembler
 b        browse symbols, flags, configurations, classes, ...
 B        toggle breakpoint
 c/C      toggle (c)ursor and (C)olors
 d[f?]    define function, data, code, ..
 D        enter visual diff mode (set diff.from/to
 e        edit eval configuration variables
 f/F      set/unset or browse flags. f- to unset, F to browse, ..
 gG       go seek to begin and end of file (0-$s)
 hjkl     move around (or HJKL) (left-down-up-right)
 i        insert hex or string (in hexdump) use tab to toggle
 mK/'K    mark/go to Key (any key)
 M        walk the mounted filesystems
 n/N      seek next/prev function/flag/hit (scr.nkey)
 g        go/seek to given offset
 O        toggle asm.pseudo and asm.esil
 p/P      rotate print modes (hex, disasm, debug, words, buf)
 q        back to radare shell
 r        refresh screen / in cursor mode browse comments
 R        randomize color palette (ecr)
 sS       step / step over
 t        browse types
 T        enter textlog chat console (TT)
 uU       undo/redo seek
 v        visual function/vars code analysis menu
 V        (V)iew graph using cmd.graph (agv?)
 wW       seek cursor to next/prev word
 xX       show xrefs/refs of current function from/to data/code
 yY       copy and paste selection
 z        fold/unfold comments in disassembly
 Z        toggle zoom mode
 Enter    follow address of jump/call
Function Keys: (See 'e key.'), defaults to:
  F2      toggle breakpoint
  F4      run to cursor
  F7      single step
  F8      step over
  F9      continue
```

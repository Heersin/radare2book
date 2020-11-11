# 可视化面板

## 概念

可视化面板具有如下核心功能：

1. 分屏
2. 多屏显示，诸如符号、寄存器、栈以及自定义的面板
3. 包含常用命令的菜单，无需记忆大量命令

结合了GUI中有用的部分作为菜单，这就是可视化面板。

可以使用`v`命令或者在可视化模式里按`!`进入该面板

## 总览

![Panels Overview](panels_overview.png)

## 命令
```
|Visual Ascii Art Panels:
| |      split the current panel vertically
| -      split the current panel horizontally
| :      run r2 command in prompt
| ;      add/remove comment
| _      start the hud input mode
| \      show the user-friendly hud
| ?      show this help
| !      run r2048 game
| .      seek to PC or entrypoint
| *      show decompiler in the current panel
| "      create a panel from the list and replace the current one
| /      highlight the keyword
| (      toggle snow
| &      toggle cache
| [1-9]  follow jmp/call identified by shortcut (like ;[1])
| ' '    (space) toggle graph / panels
| tab    go to the next panel
| Enter  start Zoom mode
| a      toggle auto update for decompiler
| b      browse symbols, flags, configurations, classes, ...
| c      toggle cursor
| C      toggle color
| d      define in the current address. Same as Vd
| D      show disassembly in the current panel
| e      change title and command of current panel
| f      set/add filter keywords
| F      remove all the filters
| g      go/seek to given offset
| G      go/seek to highlight
| i      insert hex
| hjkl   move around (left-down-up-right)
| HJKL   move around (left-down-up-right) by page
| m      select the menu panel
| M      open new custom frame
| n/N    seek next/prev function/flag/hit (scr.nkey)
| p/P    rotate panel layout
| q      quit, or close a tab
| Q      close all the tabs and quit
| r      toggle callhints/jmphints/leahints 
| R      randomize color palette (ecr)
| s/S    step in / step over
| t/T    tab prompt / close a tab
| u/U    undo / redo seek
| w      start Window mode
| V      go to the graph mode
| xX     show xrefs/refs of current function from/to data/code
| z      swap current panel with the first one
```

## 基本用法

可以用`tab`在各面板之间切换，直到切换到目标面板为止。接着如vim中一样使用`hjkl`可在面板中移动。
`S`和`s`键分别能执行步过和步入，在调试时所有面板（包括寄存器和栈面板）都会动态地更新。之后的部分里还会解释如何插入十六进制字节值修改变量。
尽管键入`tab`可以在面板之间切换，我们还是强烈推荐你使用`m`键打开菜单，里面包含了很多有用的东西，在菜单中像其他地方一样用`hjkl`移动就行了。
还可以用`"`键快速浏览radare2提供的多个视图，以及修改选定面板里的内容。

## 分屏

`|` 用于竖分屏，`-` 用于横分屏. 可以按下`X`键删除面板.

在窗口模式（`w`键进入）下可以调整面板的大小。

## 窗口模式命令
```
|Panels Window mode help:
| ?      show this help
| ??     show the user-friendly hud
| Enter  start Zoom mode
| c      toggle cursor
| hjkl   move around (left-down-up-right)
| JK     resize panels vertically
| HL     resize panels horizontally
| q      quit Window mode
```

## 编辑值

在寄存器面板或栈面板里可以对其中的值进行修改。使用`c`可激活光标，同样使用`hjkl`进行移动。之后，按下`i`插入一个值（类似vim切换至插入模式那样）

## Tab
可视化面板还提供了标签，方便用户快速访问多种类型的信息。按下`t`可以进入标签模式，所有的标签编号都列在右上角。默认情况下仅有一个标签页，可以在标签模式里按下`t`创建新的标签页，或者在该模式下按下`T`创建一个全新的面板。

在标签模式下输入标签编号可以切换到对应标签，而在该模式下输入`-`则会删除当前标签页。
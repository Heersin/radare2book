# Dietline

Radare2具有如[readline](https://en.wikipedia.org/wiki/GNU_Readline)处理输入的能力，通过Lean库处理命令编辑和历史导航的功能，允许用户移动光标、进行历史搜索，并实现了自动补全功能。
归功于radare2的可移植性，在radare2支持的所有平台上Dietline的体验都是统一的，在radare2的所有子shell - 主界面，SDB shell，可视化界面以及偏移量界面都使用了Dietline，其也实现了常见的特性以及按键行为，与GNU Readline相兼容。

Dietline支持两类配置模式： Emacs-mode和Vi-mode

其还支持著名的`Ctrl-R`-历史字符串逆向搜索功能。使用`TAB`键可以显示出自动补全建议。

# 自动补全

radare2里的每个shell都支持自动不全，其支持多种模式 - files，flags，以及SDB keys/namespaces。为了能够方便地选择补全选项，可以将`scr.prompt.popup`设置为`true`，启用下拉式补全窗口。

# Emacs (default) mode

默认的dietline模式与readline的Emacs-like模式兼容，因此对应的行为为：

## 移动
- `Ctrl-a` - 移动至行开头
- `Ctrl-e` - 移动至行结尾 
- `Ctrl-b` - 向左移动一个字符
- `Ctrl-f` - 向右移动一个字符

## 删除
- `Ctrl-w` - 删除前一个词
- `Ctrl-u` - 删除整行
- `Ctrl-h` - 删除左边的字符
- `Ctrl-d` - 删除右边的字符
- `Alt-d` -  删除光标右边的一个词

## Killing and Yanking
- `Ctrl-k` - kill至行尾
- `Ctrl-x` - 反向kill至行的开头
- `Ctrl-t` - kill至词尾, 若在二词之间则kill至下一个词结束。词边界的界定与下面相同
- `Ctrl-w` - 反向kill至词头，词以空格作为界定。kill的内容都存在kill-ring中
- `Ctrl-y` - 将kill ring的顶部（最近的一个内容）粘贴到此处
- `Ctrl-]` - 将kill ring旋转，然后将新的kill ring的顶部内容粘贴于此。仅可在前一条命令为yank或yank-pop才可进行此操作。

## History
- `Ctrl-r` - the reverse search in the command history

# Vi mode

Radare2 also comes with in vi mode that can be enabled by toggling `scr.prompt.vi`. The various keybindings available in this mode are:

## Entering command modes
- `ESC` - enter into the control mode
- `i` - enter into the insert mode

## Moving
- `j` - acts like up arrow key
- `k` - acts like down arrow key
- `a` - move cursor forward and enter into insert mode
- `I` - move to the beginning of the line and enter into insert mode
- `A` - move to the end of the line and enter into insert mode
- `^` - move to the beginning of the line
- `0` - move to the beginning of the line
- `$` - move to the end of the line
- `h` - move one character backward
- `l` - move one character forward

## Deleting and Yanking
- `x` - cuts the character
- `dw` - delete the current word
- `diw` - deletes the current word.
- `db` - delete the previous word
- `D` - delete the whole line
- `dh` - delete a character to the left
- `dl` - delete a character to the right
- `d$` - kill the text from point to the end of the line.
- `d^` - kill backward from the cursor to the beginning of the current line.
- `de` - kill from point to the end of the current word, or if between words, to the end of the next word. Word boundaries are the same as forward-word.
- `p` - yank the top of the kill ring into the buffer at point.
- `c` - acts similar to d based commands, but goes into insert mode in the end by prefixing the commands with numbers, the command is performed multiple times.

If you are finding it hard to keep track of which mode you are in, just set `scr.prompt.mode=true` to update the color of the prompt based on the vi-mode.

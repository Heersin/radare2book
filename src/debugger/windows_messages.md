# Windows Messages

在Windows上，可以在debug时用`dbW`在指定窗口的message句柄上设置断点。

获取当前
用dW获取当前窗口进程的列表：
```
[0x7ffe885c1164]> dW
.----------------------------------------------------.
| Handle      | PID   | TID    | Class Name          |
)----------------------------------------------------(
| 0x0023038e  | 9432  | 22432  | MSCTFIME UI         |
| 0x0029049e  | 9432  | 22432  | IME                 |
| 0x002c048a  | 9432  | 22432  | Edit                |
| 0x000d0474  | 9432  | 22432  | msctls_statusbar32  |
| 0x00070bd6  | 9432  | 22432  | Notepad             |
`----------------------------------------------------'
```

通过message类型加上窗口类型名/句柄设置断点：

```
[0x7ffe885c1164]> dbW WM_KEYDOWN Edit
Breakpoint set.
```

或

``` 
[0x7ffe885c1164]> dbW WM_KEYDOWN 0x002c048a
Breakpoint set.
```

如果不确定该在哪个窗口上下断点，用`dWi`启用使用鼠标进行识别：

```
[0x7ffe885c1164]> dWi
Move cursor to the window to be identified. Ready? y
Try to get the child? y
.--------------------------------------------.
| Handle      | PID   | TID    | Class Name  |
)--------------------------------------------(
| 0x002c048a  | 9432  | 22432  | Edit        |
`--------------------------------------------'
```

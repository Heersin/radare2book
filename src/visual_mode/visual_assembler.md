# 汇编界面

在可视化模式下使用`A`可进行汇编。
举个例子，将`push`替换为`jmp`：

![Before](../pics/before.png)

注意反汇编预览视图和那个箭头：

![After](../pics/after.png)

若真要进行patch需要以写模式打开文件（`r2 -w`或`oo+`），还可以使用cache模式：`e io.cache = true`以及`wc?`。

要记住debug模式里的patch操作只在内存中完成，并未patch到文件中。

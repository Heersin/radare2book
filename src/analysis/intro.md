# 数据与代码分析

Radare2具有一组非常丰富的命令和配置选项，可以用于执行代码和数据的分析，以及从二进制文件中提取有用的信息，例如指针，字符串引用，基本块，操作码数据，跳转目标，交叉引用等等。
这些操作由`a`（analyze）命令族处理：

```
|Usage: a[abdefFghoprxstc] [...]
| aa[?]              analyze all (fcns + bbs) (aa0 to avoid sub renaming)
| a8 [hexpairs]      analyze bytes
| ab[b] [addr]       analyze block at given address
| abb [len]          analyze N basic blocks in [len] (section.size by default)
| abt [addr]         find paths in the bb function graph from current offset to given address
| ac [cycles]        analyze which op could be executed in [cycles]
| ad[?]              analyze data trampoline (wip)
| ad [from] [to]     analyze data pointers to (from-to)
| ae[?] [expr]       analyze opcode eval expression (see ao)
| af[?]              analyze Functions
| aF                 same as above, but using anal.depth=1
| ag[?] [options]    draw graphs in various formats
| ah[?]              analysis hints (force opcode size, ...)
| ai [addr]          address information (show perms, stack, heap, ...)
| an [name] [@addr]  show/rename/create whatever flag/function is used at addr
| ao[?] [len]        analyze Opcodes (or emulate it)
| aO[?] [len]        Analyze N instructions in M bytes
| ap                 find prelude for current offset
| ar[?]              like 'dr' but for the esil vm. (registers)
| as[?] [num]        analyze syscall using dbg.reg
| av[?] [.]          show vtables
| ax[?]              manage refs/xrefs (see also afx?)
```

事实上，`a`命名空间是radare2最大的命名空间之一，可以控制分析过程中不同的部分：

 - 程序流分析
 - 数据引用
 - 加载符号
 - 管理各种视图，例如CFG和call graph
 - 管理变量
 - 管理类型
 - 使用ESIL VM模拟执行
 - Opcode introspection
 - 其他的一些对象信息，例如Virtual Table


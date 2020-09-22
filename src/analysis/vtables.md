# 虚函数表

r2中支持基本的虚函数表解析（RTTI机制及其他）。在执行该分析之前最重要的是检查`anal.cpp.abi`选项是否设置正确，如果没有记得修改它。

所有与虚函数表相关的命令位于`av`命名空间下。目前，对于虚函数表的支持还比较基础，仅允许检视解析的表。

```
|Usage: av[?jr*] C++ vtables and RTTI
| av           search for vtables in data sections and show results
| avj          like av, but as json
| av*          like av, but as r2 commands
| avr[j@addr]  try to parse RTTI at vtable addr (see anal.cpp.abi)
| avra[j]      search for vtables and try to parse RTTI at each of them
```

`av`空间中主要的命令是`avr`和`av`，`av`列出r2在打开该文件时找到的所有虚表，如果对该结果不满意，可以考虑用`avr`命令去对特定地址上的虚表进行解析。`avra`命令用于搜索和解析二进制文件中的所有虚表，就如r2打开文件时做的那样。

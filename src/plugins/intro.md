# Plugins

Radare2是基于大量的函数库实现的，这些库几乎都支持用插件对其功能进行扩展，或是实现对不同目标的支持。

本章节着重于解释什么是插件，如何编写插件以及如何使用插件

## plugins类型
```
$ ls libr/*/p | grep : | awk -F / '{ print $2 }'
anal      # analysis plugins
asm       # assembler/disassembler plugins
bin       # binary format parsing plugins
bp        # breakpoint plugins
core      # core plugins (implement new commands)
crypto    # encrypt/decrypt/hash/...
debug     # debugger backends
egg       # shellcode encoders, etc
fs        # filesystems and partition tables
io        # io plugins
lang      # embedded scripting languages
parse     # disassembler parsing plugins
reg       # arch register logic
```

## 插件列表

r2工具包中的一些工具支持用`-L`列出相关的插件：
```
rasm2 -L    # list asm plugins
r2 -L       # list io plugins
rabin2 -L   # list bin plugins
rahash2 -L  # list hash/crypto/encoding plugins
```

在r2land中还有更多的插件，可以在r2中使用后缀`L`列出它们。

底下是一些命令:
```
L          # list core plugins
iL         # list bin plugins
dL         # list debug plugins
mL         # list fs plugins
ph         # print support hash algoriths
```

可以用`?`作为参数，获取相关变量的所有可能值。

```
e asm.arch=?   # list assembler/disassembler plugins
e anal.arch=?  # list analysis plugins
```
## Notes

未来radare2版本中很可能会解决现有的一些不兼容问题。

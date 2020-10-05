## 实现一个debug插件

* 在shlr/gdb/src/core.c里添加debugger的寄存器配置
* 在 libr/debug/p/debug_native.c和libr/debug/p/debug_gdb.c里加入寄存器配置和架构支持
* 在`r_debug_gdb_attach(RDebug *dbg, int pid)`添加代码启用配置

如果想为gdb添加新的支持，可以在gdb session内用`maint print registers`命令查看寄存器。

## 更多信息..

* 相关文章: http://radare.today/posts/extending-r2-with-new-plugins/

与实现新架构相关的commits:

* Extensa: https://github.com/radareorg/radare2/commit/6f1655c49160fe9a287020537afe0fb8049085d7
* Malbolge: https://github.com/radareorg/radare2/pull/579
* 6502: https://github.com/radareorg/radare2/pull/656
* h8300: https://github.com/radareorg/radare2/pull/664
* GBA: https://github.com/radareorg/radare2/pull/702
* CR16: https://github.com/radareorg/radare2/pull/721/ && 726
* XCore: https://github.com/radareorg/radare2/commit/bb16d1737ca5a471142f16ccfa7d444d2713a54d
* SharpLH5801: https://github.com/neuschaefer/radare2/commit/f4993cca634161ce6f82a64596fce45fe6b818e7
* MSP430: https://github.com/radareorg/radare2/pull/1426
* HP-PA-RISC: https://github.com/radareorg/radare2/commit/f8384feb6ba019b91229adb8fd6e0314b0656f7b
* V810: https://github.com/radareorg/radare2/pull/2899
* TMS320: https://github.com/radareorg/radare2/pull/596

## 实现一个新的伪代码插件

下面是针对z80的一个简单插件，可以将其作为例子进行参考：

https://github.com/radareorg/radare2/commit/8ff6a92f65331cf8ad74cd0f44a60c258b137a06

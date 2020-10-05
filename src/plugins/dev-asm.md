## 实现一个新的ASM插件

Radare2具有模块化架构，如果你能熟练使用C语言编程，那么为r2加入新架构的支持就很容易了。出于各种原因的考虑，我们认为先在源码树外进行插件实现比较容易，我们现在只需要创建一个单独的C文件，命名为`asm_mycpu.c`，以及其对应的Makefile。

RAsm插件中的一个关键点是struct
```c
RAsmPlugin r_asm_plugin_mycpu = {
	.name = "mycpu",
	.license = "LGPL3",
	.desc = "MYCPU disassembly plugin",
	.arch = "mycpu",
	.bits = 32,
	.endian = R_SYS_ENDIAN_LITTLE,
	.disassemble = &disassemble
};
```

其中的`.disassemble` 是一个指向反汇编函数的指针, 该函数接受Byte Buffer和长度作为参数:

```c
static int disassemble(RAsm *a, RAsmOp *op, const ut8 *buf, int len)
```

**Makefile**

```makefile
NAME=asm_snes
R2_PLUGIN_PATH=$(shell r2 -H R2_USER_PLUGINS)
LIBEXT=$(shell r2 -H LIBEXT)
CFLAGS=-g -fPIC $(shell pkg-config --cflags r_anal)
LDFLAGS=-shared $(shell pkg-config --libs r_anal)
OBJS=$(NAME).o
LIB=$(NAME).$(LIBEXT)

all: $(LIB)

clean:
	rm -f $(LIB) $(OBJS)

$(LIB): $(OBJS)
	$(CC) $(CFLAGS) $(LDFLAGS) $(OBJS) -o $(LIB)

install:
	cp -f asm_mycpu.$(SO_EXT) $(R2_PLUGIN_PATH)

uninstall:
	rm -f $(R2_PLUGIN_PATH)/asm_mycpu.$(SO_EXT)
```

**asm_mycpu.c**

```c
/* radare - LGPL - Copyright 2018 - user */

#include <stdio.h>
#include <string.h>
#include <r_types.h>
#include <r_lib.h>
#include <r_asm.h>

static int disassemble(RAsm *a, RAsmOp *op, const ut8 *buf, int len) {
	struct op_cmd cmd = {
		.instr = "",
		.operands = ""
	};
	if (len < 2) return -1;
	int ret = decode_opcode (buf, len, &cmd);
	if (ret > 0) {
		snprintf (op->buf_asm, R_ASM_BUFSIZE, "%s %s",
			  cmd.instr, cmd.operands);
	}
	return op->size = ret;
}

RAsmPlugin r_asm_plugin_mycpu = {
	.name = "mycpu",
	.license = "LGPL3",
	.desc = "MYCPU disassembly plugin",
	.arch = "mycpu",
	.bits = 32,
	.endian = R_SYS_ENDIAN_LITTLE,
	.disassemble = &disassemble
};

#ifndef R2_PLUGIN_INCORE
R_API RLibStruct radare_plugin = {
	.type = R_LIB_TYPE_ASM,
	.data = &r_asm_plugin_mycpu,
	.version = R2_VERSION
};
#endif
```

编译完成后，此插件会在radare2的输出的插件列表中显示：
```
_d__  _8_32      mycpu        LGPL3   MYCPU
```

### 将Plugin加入源码树
在r2的main branch中增加一个新架构需要修改多处文件，使之同其他插件一样被正确构建：

需要修改的文件如下:

* `plugins.def.cfg` : add the `asm.mycpu` plugin name string in there
* `libr/asm/p/mycpu.mk` : build instructions
* `libr/asm/p/asm_mycpu.c` : implementation
* `libr/include/r_asm.h` : add the struct definition in there

可以阅读下面的commits，参考NIOS II CPU反汇编插件是如何进行实现的：

Implement RAsm plugin:
https://github.com/radareorg/radare2/commit/933dc0ef6ddfe44c88bbb261165bf8f8b531476b

Implement RAnal plugin:
https://github.com/radareorg/radare2/commit/ad430f0d52fbe933e0830c49ee607e9b0e4ac8f2

## 实现一个新的分析插件

在实现反汇编插件后，可能会注意到输出的效果并不是很好 - 没有合适的代码高亮，没有引用线等等。这是因为radare2要求每个架构插件提供对每个opcode的分析信息。目前反汇编和opcode分析是被划分为两个模块的 - RAsm和RAnal，因此我们还需要编写一个分析插件。编写分析插件的过程和编写反汇编插件很像 - 只需要创建一个C文件并提供对应的Makefile，

RAnal插件结构体如下：

```c
RAnalPlugin r_anal_plugin_v810 = {
	.name = "mycpu",
	.desc = "MYCPU code analysis plugin",
	.license = "LGPL3",
	.arch = "mycpu",
	.bits = 32,
	.op = mycpu_op,
	.esil = true,
	.set_reg_profile = set_reg_profile,
};
```

同disassembly一样，需要为分析插件编写一个关键函数 - `mycpu_op`，该函数的功能是扫描opcode并构建RAnalOp结构体。
此外，本例中的分析插件还兼有提炼ESIL的功能（可以通过`..esil = true`语句启用之）。因此`mycpu_op`还需要完成opcode对应的RAnalOp中ESIL字段的填充工作。
若要完成ESIL的提炼和模拟，还需要进行寄存器的配置，得在`set_reg_profile`函数内进行设置，就像debugger中做的那样。

**Makefile**

```makefile
NAME=anal_snes
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
	cp -f anal_snes.$(SO_EXT) $(R2_PLUGIN_PATH)

uninstall:
	rm -f $(R2_PLUGIN_PATH)/anal_snes.$(SO_EXT)
```

**anal_snes.c:**

```c
/* radare - LGPL - Copyright 2015 - condret */

#include <string.h>
#include <r_types.h>
#include <r_lib.h>
#include <r_asm.h>
#include <r_anal.h>
#include "snes_op_table.h"

static int snes_anop(RAnal *anal, RAnalOp *op, ut64 addr, const ut8 *data, int len) {
	memset (op, '\0', sizeof (RAnalOp));
	op->size = snes_op[data[0]].len;
	op->addr = addr;
	op->type = R_ANAL_OP_TYPE_UNK;
	switch (data[0]) {
		case 0xea:
			op->type = R_ANAL_OP_TYPE_NOP;
			break;
	}
	return op->size;
}

struct r_anal_plugin_t r_anal_plugin_snes = {
	.name = "snes",
	.desc = "SNES analysis plugin",
	.license = "LGPL3",
	.arch = R_SYS_ARCH_NONE,
	.bits = 16,
	.init = NULL,
	.fini = NULL,
	.op = &snes_anop,
	.set_reg_profile = NULL,
	.fingerprint_bb = NULL,
	.fingerprint_fcn = NULL,
	.diff_bb = NULL,
	.diff_fcn = NULL,
	.diff_eval = NULL
};

#ifndef R2_PLUGIN_INCORE
R_API RLibStruct radare_plugin = {
	.type = R_LIB_TYPE_ANAL,
	.data = &r_anal_plugin_snes,
	.version = R2_VERSION
};
#endif
```
After compiling radare2 will list this plugin in the output:
```
_dA_  _8_16      snes        LGPL3   SuperNES CPU
```

**snes_op_table**.h: https://github.com/radareorg/radare2/blob/master/libr/asm/arch/snes/snes_op_table.h

Example:

* **6502**: https://github.com/radareorg/radare2/commit/64636e9505f9ca8b408958d3c01ac8e3ce254a9b
* **SNES**: https://github.com/radareorg/radare2/commit/60d6e5a1b9d244c7085b22ae8985d00027624b49


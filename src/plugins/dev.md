## 实现新架构

radare2将CPU中的逻辑划分至不同的模块中，因此需要通过编写多个插件实现对一个新架构的完整支持，下面列出了支持新架构所需要编写的插件：

* r_asm : assembler and disassembler
* r_anal : code analysis (opcode,type,esil,..)
* r_reg : registers
* r_syscall : system calls
* r_debug : debugger

大多数人对于新架构支持最基本的需求就是反汇编，毕竟我们首先需要将字节转换为人类可读的形式。

记住，插件可以是静态编译也可以是动态编译，这个选择意味着你是想将其作为r2核心库的一部分，或是将其作为一个独立的共享库实现。`./configure-plugins`脚本接受 --shared和 --static选项，可以通过指定二者之一决定使用何种方式编译。你也可以手动在`plugins.def.cfg`进行配置，然后删除`plugins.cfg`文件，重新运行`./configure-plugins`更新`libr/config.mk`和`libr/config.h`。

在[radare2-extras](https://github.com/radareorg/radare2-extras)仓库里应该能发现更多的外部插件。

## 编写r_asm插件

官方推荐的制作第三方插件的方法是将其发布在一个独立的repo里，下面是一个disasm插件示例：

```Makefile
$ cd my-cpu
$ cat Makefile
NAME=mycpu
R2_PLUGIN_PATH=$(shell r2 -hh|grep R2_LIBR_PLUGINS|awk '{print $$2}')
CFLAGS=-g -fPIC $(shell pkg-config --cflags r_asm)
LDFLAGS=-shared $(shell pkg-config --libs r_asm)
OBJS=$(NAME).o
SO_EXT=$(shell uname|grep -q Darwin && echo dylib || echo so)
LIB=$(NAME).$(SO_EXT)

all: $(LIB)

clean:
	rm -f $(LIB) $(OBJS)

$(LIB): $(OBJS)
	$(CC) $(CFLAGS) $(LDFLAGS) $(OBJS) -o $(LIB)

install:
	cp -f $(NAME).$(SO_EXT) $(R2_PLUGIN_PATH)

uninstall:
	rm -f $(R2_PLUGIN_PATH)/$(NAME).$(SO_EXT)
```

```c
$ cat mycpu.c
/* example r_asm plugin by pancake at 2014 */

#include <r_asm.h>
#include <r_lib.h>

#define OPS 17

static const char *ops[OPS*2] = {
	"nop", NULL,
	"if", "r",
	"ifnot", "r",
	"add", "rr",
	"addi", "ri",
	"sub", "ri",
	"neg", "ri",
	"xor", "ri",
	"mov", "ri",
	"cmp", "rr",
	"load", "ri",
	"store", "ri",
	"shl", "ri",
	"br", "r",
	"bl", "r",
	"ret", NULL,
	"sys", "i"
};

/* Main function for disassembly */
//b for byte, l for length
static int disassemble (RAsm *a, RAsmOp *op, const ut8 *b, int l) {
	char arg[32];
        int idx = (b[0]&0xf)\*2;
	op->size = 2;
	if (idx>=(OPS*2)) {
		strcpy (op->buf_asm, "invalid");
		return -1;
	}
	strcpy (op->buf_asm, ops[idx]);
	if (ops[idx+1]) {
		const char \*p = ops[idx+1];
		arg[0] = 0;
		if (!strcmp (p, "rr")) {
			sprintf (arg, "r%d, r%d", b[1]>>4, b[1]&0xf);
		} else
		if (!strcmp (p, "i")) {
			sprintf (arg, "%d", (char)b[1]);
		} else
		if (!strcmp (p, "r")) {
			sprintf (arg, "r%d, r%d", b[1]>>4, b[1]&0xf);
		} else
		if (!strcmp (p, "ri")) {
			sprintf (arg, "r%d, %d", b[1]>>4, (char)b[1]&0xf);
		}
		if (*arg) {
			strcat (op->buf_asm, " ");
			strcat (op->buf_asm, arg);
		}
	}
	return op->size;
}

/* Structure of exported functions and data */
RAsmPlugin r_asm_plugin_mycpu = {
        .name = "mycpu",
        .arch = "mycpu",
        .license = "LGPL3",
        .bits = 32,
        .desc = "My CPU disassembler",
        .disassemble = &disassemble,
};

#ifndef CORELIB
struct r_lib_struct_t radare_plugin = {
        .type = R_LIB_TYPE_ASM,
        .data = &r_asm_plugin_mycpu
};
#endif
```

构建并安装该插件：

```
$ make
$ sudo make install
```

# Python 插件

在用python写r2插件之前，需要安装一个r2lang的插件 `r2pm -i lang-python` 。
注意 - 为了可读性，下面的例子中省略了实际的解码函数。

编写python插件的流程如下：
1. `import r2lang` and `from r2lang import R` (for constants)
2. 创建一个函数，其包含两个子函数 - `assemble` 和 `disassemble` 并返回一个一个plugin struct - 用于RAsm插件
```python
def mycpu(a):
    def assemble(s):
        return [1, 2, 3, 4]

    def disassemble(memview, addr):
        try:
            opcode = get_opcode(memview) # https://docs.python.org/3/library/stdtypes.html#memoryview
            opstr = optbl[opcode][1]
            return [4, opstr]
        except:
            return [4, "unknown"]
```
3. 该结构体需要包含指向这两个函数的指针 - `assemble`和`disassemble`

```python
    return {
            "name" : "mycpu",
            "arch" : "mycpu",
            "bits" : 32,
            "endian" : R.R_SYS_ENDIAN_LITTLE,
            "license" : "GPL",
            "desc" : "MYCPU disasm",
            "assemble" : assemble,
            "disassemble" : disassemble,
    }
```
4. 创建一个函数，其包含两个子函数 - `set_reg_profile` 和 `op`， 并返回一个plugin struct - 用于RAnal插件
```python
def mycpu_anal(a):
       def set_reg_profile():
        profile = "=PC	pc\n" + \
		"=SP	sp\n" + \
		"gpr	r0	.32	0	0\n" + \
		"gpr	r1	.32	4	0\n" + \
		"gpr	r2	.32	8	0\n" + \
		"gpr	r3	.32	12	0\n" + \
		"gpr	r4	.32	16	0\n" + \
		"gpr	r5	.32	20	0\n" + \
		"gpr	sp	.32	24	0\n" + \
		"gpr	pc	.32	28	0\n"
        return profile

    def op(memview, pc):
		analop = {
            "type" : R.R_ANAL_OP_TYPE_NULL,
            "cycles" : 0,
            "stackop" : 0,
            "stackptr" : 0,
			"ptr" : -1,
            "jump" : -1,
            "addr" : 0,
            "eob" : False,
            "esil" : "",
        }
        try:
            opcode = get_opcode(memview) # https://docs.python.org/3/library/stdtypes.html#memoryview
            esilstr = optbl[opcode][2]
            if optbl[opcode][0] == "J": # it's jump
                analop["type"] = R.R_ANAL_OP_TYPE_JMP
                analop["jump"] = decode_jump(opcode, j_mask)
                esilstr = jump_esil(esilstr, opcode, j_mask)

        except:
            result = analop
		# Don't forget to return proper instruction size!
        return [4, result]

```
5. 该结构体需要包含指向这两个函数的指针 - `set_reg_profile`和`op`

```python
    return {
            "name" : "mycpu",
            "arch" : "mycpu",
            "bits" : 32,
            "license" : "GPL",
            "desc" : "MYCPU anal",
            "esil" : 1,
            "set_reg_profile" : set_reg_profile,
            "op" : op,
    }
```
6. 分别用`r2lang.plugin("asm")` and `r2lang.plugin("anal")`进行注册：

```python
print("Registering MYCPU disasm plugin...")
print(r2lang.plugin("asm", mycpu))
print("Registering MYCPU analysis plugin...")
print(r2lang.plugin("anal", mycpu_anal))
```

可以把上面这些写在一个文件里，然后用`-i`选项加载就行了：
```
r2 -I mycpu.py some_file.bin
```
或者可以在r2 shell里进行加载： `#!python mycpu.py` 

参见:

* [Python](https://github.com/radareorg/radare2-bindings/blob/master/libr/lang/p/test-py-asm.py)
* [Javascript](https://github.com/radareorg/radare2-bindings/blob/master/libr/lang/p/dukasm.js)

### 用python实现一个文件格式插件

注意 - 为了可读性，下面的例子中省略了实际的解码函数。

步骤:
1. `import r2lang`
2. 创建一个函数，其包含下面这些子函数:
   - `load`
   - `load_bytes`
   - `destroy`
   - `check_bytes`
   - `baddr`
   - `entries`
   - `sections`
   - `imports`
   - `relocs`
   - `binsym`
   - `info`

   并返回一个plugin struct - 用于 RAsm plugin：
```python
def le_format(a):
    def load(binf):
        return [0]

    def check_bytes(buf):
        try:
			if buf[0] == 77 and buf[1] == 90:
                lx_off, = struct.unpack("<I", buf[0x3c:0x40])
                if buf[lx_off] == 76 and buf[lx_off+1] == 88:
                    return [1]
            return [0]
        except:
            return [0]
```
诸如此类。 注意检查每个函数的参数和返回值的格式，`entries`，`sections`，`imports`，`relocs`都返回一个包含字典的列表，但不同列表中包含的字典类型是不同的。
其余函数的返回值只是一个包含数值的列表，甚至可能只包含一个元素。
下面是一个特殊的函数，其返回文件信息结构 - `info`：
```python
    def info(binf):
        return [{
                "type" : "le",
                "bclass" : "le",
                "rclass" : "le",
                "os" : "OS/2",
                "subsystem" : "CLI",
                "machine" : "IBM",
                "arch" : "x86",
                "has_va" : 0,
                "bits" : 32,
                "big_endian" : 0,
                "dbg_info" : 0,
                }]
```

3. 返回的plugin struct需要包含大部分的核心函数指针，比如`check_bytes`, `load`和`load-bytes`, `entries`, `relocs`, `imports`。

```python
    return {
            "name" : "le",
            "desc" : "OS/2 LE/LX format",
            "license" : "GPL",
            "load" : load,
            "load_bytes" : load_bytes,
            "destroy" : destroy,
            "check_bytes" : check_bytes,
            "baddr" : baddr,
            "entries" : entries,
            "sections" : sections,
            "imports" : imports,
            "symbols" : symbols,
            "relocs" : relocs,
            "binsym" : binsym,
            "info" : info,
    }
```
4. 作为为文件格式插件进行注册:

```python
print("Registering OS/2 LE/LX plugin...")
print(r2lang.plugin("bin", le_format))
```

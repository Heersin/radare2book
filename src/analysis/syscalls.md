# Syscalls

Radare2允许使用者手动搜索诸如syscall操作这样的汇编码。例如在ARM平台上，`svc`就代表系统调用，而在其他平台上可能是另一条指令，例如X86平台上的`syscall`。
```
[0x0001ece0]> /ad/ svc
...
0x000187c2   # 2: svc 0x76
0x000189ea   # 2: svc 0xa9
0x00018a0e   # 2: svc 0x82
...
```
系统调用检测由`asm.os`, `asm.bits`和`asm.arch`驱动，确保这些配置选项被相应地设置了。可以使用`asl`命令来检查系统调用的支持是否正确设置以及是否符合期望，该命令也列出了平台支持的系统调用。
```
[0x0001ece0]> asl
...
sd_softdevice_enable = 0x80.16
sd_softdevice_disable = 0x80.17
sd_softdevice_is_enabled = 0x80.18
...
```

如果你使用`aei`或`aeim`设置了ESIL栈，则可以使用`/as`命令进行搜索，找到特定系统调用的地址并列出它们。
```
[0x0001ece0]> aei
[0x0001ece0]> /as
0x000187c2 sd_ble_gap_disconnect
0x000189ea sd_ble_gatts_sys_attr_set
0x00018a0e sd_ble_gap_sec_info_reply
...
```
若要减小搜索时间，可以[限制搜索范围](../search_bytes/configurating_the_search.md)，通过`/as @e:search.in=io.maps.x`设置为将其限制在可执行的节区或段上。

使用[ESIL仿真]（emulation.md）radare2可以以反汇编的格式输出syscall的参数，要启用线性（这个功能还非常粗糙）仿真，请配置`asm.emu`变量：
```
[0x0001ece0]> e asm.emu=true
[0x0001ece0]> s 0x000187c2
[0x000187c2]> pdf~svc
   0x000187c2   svc 0x76  ; 118 = sd_ble_gap_disconnect
[0x000187c2]>
```

在执行`aae`（或者会调用`aae`的`aaaa`命令）时，radare2会将找到的所有`syscall`放入`syscall.`标志空间中，在实现流程自动化时这个特性比较有用。
```
[0x000187c2]> fs
0    0 * imports
1    0 * symbols
2 1523 * functions
3  420 * strings
4  183 * syscalls
[0x000187c2]> f~syscall
...
0x000187c2 1 syscall.sd_ble_gap_disconnect.0
0x000189ea 1 syscall.sd_ble_gatts_sys_attr_set
0x00018a0e 1 syscall.sd_ble_gap_sec_info_reply
...
```


也可以在HUD模式（`V_`）中进行交互式导航。
```
0> syscall.sd_ble_gap_disconnect
 - 0x000187b2  syscall.sd_ble_gap_disconnect
   0x000187c2  syscall.sd_ble_gap_disconnect.0
   0x00018a16  syscall.sd_ble_gap_disconnect.1
   0x00018b32  syscall.sd_ble_gap_disconnect.2
   0x0002ac36  syscall.sd_ble_gap_disconnect.3
```

在radare2进行调试时，可以用`dcs`继续执行直至遇到下一个系统调用，可以用`dcs*`追踪所有系统调用。
```
[0xf7fb9120]> dcs*
Running child until syscalls:-1 
child stopped with signal 133
--> SN 0xf7fd3d5b syscall 45 brk (0xffffffda)
child stopped with signal 133
--> SN 0xf7fd28f3 syscall 384 arch_prctl (0xffffffda 0x3001)
child stopped with signal 133
--> SN 0xf7fc81b2 syscall 33 access (0xffffffda 0xf7fd8bf1)
child stopped with signal 133
```

Radare2还有将syscall名字转为syscall代号的小工具，可以通过syscall名字获取对应的syscall代号，反之亦然，这一切都可在r2 shell中完成。

```
[0x08048436]> asl 1
exit
[0x08048436]> asl write
4
[0x08048436]> ask write
0x80,4,3,iZi
```

参阅`as?`获取更多关于该工具的信息。

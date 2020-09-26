# WinDBG Kernel-mode Debugging (KD)

WinDBG KD接口支持r2 attach到运行windows的VM上，通过一个串口/网络对内核进行调试。

同样也可以使用远程GDB调试接口连接到windows上，从而在不依赖于windows功能的情况下对windows进行内核调试。

要注意的是，对WinDBG KD的支持仍在开发中，目前仅仅是一个初期的实现版本，它将随着时间的推移慢慢变好的。

## Windows上配置KD

> 完整步骤请参考 Microsoft的[文档](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/setting-up-kernel-mode-debugging-in-windbg--cdb--or-ntsd).

### 串口
在Windows Vista或更高版本中启动串口KD：

```
bcdedit /debug on
bcdedit /dbgsettings serial debugport:1 baudrate:115200
```

或者在Windows XP中可以这样：
	Open boot.ini and add /debug /debugport=COM1 /baudrate=115200:
```
[boot loader]
timeout=30
default=multi(0)disk(0)rdisk(0)partition(1)\WINDOWS
[operating systems]
multi(0)disk(0)rdisk(0)partition(1)\WINDOWS="Debugging with Cable" /fastdetect /debug /debugport=COM1 /baudrate=57600
```
使用VMware的场景下:
```
	Virtual Machine Settings -> Add -> Serial Port
	Device Status:
	[v] Connect at power on
	Connection:
	[v] Use socket (named pipe)
	[_/tmp/winkd.pipe________]
	From: Server To: Virtual Machine
```
若使用VirtualBox需要进行的配置：
```
    Preferences -> Serial Ports -> Port 1

    [v] Enable Serial Port
    Port Number: [_COM1_______[v]]
    Port Mode:   [_Host_Pipe__[v]]
                 [v] Create Pipe
    Port/File Path: [_/tmp/winkd.pipe____]
```
或者用QEMU启动虚拟机:
```
$ qemu-system-x86_64 -chardev socket,id=serial0,\
     path=/tmp/winkd.pipe,nowait,server \
     -serial chardev:serial0 -hda Windows7-VM.vdi
```

### 网络
在Windows 7或更高版本启动网络KD（KDNet）如下：
```
bcdedit /debug on
bcdedit /dbgsettings net hostip:w.x.y.z port:n
```
从Windows 8开始，无法在每次启动都强制进入debug了，但是可以通过配置boot option中的高级选项，允许使用内核调试：
```
bcedit /set {globalsettings} advancedoptions true
```

## r2连接到KD上

### 串口
Radare2可以用`winkd` IO插件连接到vbox/qemu创建的socket文件上。另外，`winkd`调试器插件也要指定x86-32才行（支持32和64位调试）。
```
$ r2 -a x86 -b 32 -D winkd winkd:///tmp/winkd.pipe
```

在windows上需要运行如下r2命令：
```
$ radare2 -D winkd winkd://\\.\pipe\com_1
```

### 网络
```
$ r2 -a x86 -b 32 -d winkd://<hostip>:<port>:w.x.y.z
```

## Using KD
当连接到KD时，r2会发送一个中断包，用于中断目标的运行，程序将停在如下地方：
```
[0x828997b8]> pd 20
	;-- eip:
	0x828997b8    cc           int3
	0x828997b9    c20400       ret 4
	0x828997bc    cc           int3
	0x828997bd    90           nop
	0x828997be    c3           ret
    0x828997bf    90           nop
```

若要跳过trap，得修改eip并运行两次`dc`命令：
```
dr eip=eip+1
dc
dr eip=eip+1
dc
```

现在可以重新与Windows VM进行交互了。需要杀掉r2进程然后重新attach到Windows上控制内核。

另外，`dp`命令可以列出所有进程，`dpa`或`dp=`可以attach到进程上，并显示进程在物理内存中的基址。

# WinDBG Backend for Windows (DbgEng)

在Windows上，radare2可以用`DbgEng.dll`作为调试后端，使其可以使用WinDBG的功能，比如支持dump文件，本地/远程用户和内核模式调试。

可以用Windows内包含的debugging DLL，也可以从[download page](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools)下载最新版本（推荐）

> 不能用windows商店里的`WinDbg Preview`应用文件夹里的DLL，因为对于普通用户来说，这些DLL未被标记为可执行文件。

> 在搜索windows默认的DLL库路径前， radare2会尝试从`_NT_DEBUGGER_EXTENSION_PATH`环境变量指出的路径加载`dbgeng.dll`。

## 使用插件

若要使用`windbg`插件，只需传递运行`WinDBG`/`kd`时同样的参数（参见Microsoft的[文档](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/windbg-command-line-options)）即可，有时候需要用引号将其括起，或者使用转义符：

```
> r2 -d "windbg://-remote tcp:server=Server,port=Socket"
```
```
> r2 -d "windbg://MyProgram.exe \"my arg\""
```
```
> r2 -d "windbg://-k net:port=<n>,key=<MyKey>"
```
```
> r2 -d "windbg://-z MyDumpFile.dmp"
```

可以像往常一样进行debug（参见`d?`命令）或用`=!`命令在后端shell中直接进行交互。

```
[0x7ffcac9fcea0]> dcu 0x0007ffc98f42190
Continue until 0x7ffc98f42190 using 1 bpsize
ModLoad: 00007ffc`ab6b0000 00007ffc`ab6e0000   C:\WINDOWS\System32\IMM32.DLL
Breakpoint 1 hit
hit breakpoint at: 0x7ffc98f42190

[0x7fffcf232190]> =!k4
Child-SP          RetAddr           Call Site
00000033`73b1f618 00007ff6`c67a861d r_main!r_main_radare2
00000033`73b1f620 00007ff6`c67d0019 radare2!main+0x8d
00000033`73b1f720 00007ff6`c67cfebe radare2!invoke_main+0x39
00000033`73b1f770 00007ff6`c67cfd7e radare2!__scrt_common_main_seh+0x12e
```

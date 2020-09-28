# IO 插件

对文件的访问，对网络的访问，debugger以及所有的输入输出都包纳于IO抽象层中。使得radare2能将这些对象作为文件处理。

IO插件用于在虚拟文件系统封装open，read，write和'system'操作，可以令radare将任何东西视为一个平坦文件对象，例如socket连接，远程radare session，文件，进程，设备，gdb session等。

当radare读入字节块时，IO插件起到的作用是从文件中获取字节并将其放置在内置的Buffer里。Radare会根据文件URI选择不同的IO插件去打开文件，例如：

* Debugging URIs
```
$ r2 dbg:///bin/ls<br />
$ r2 pid://1927
```
* Remote sessions
```
$ r2 rap://:1234<br />
$ r2 rap://<host>:1234//bin/ls
```
* Virtual buffers
```
$ r2 malloc://512<br />
shortcut for
$ r2 -
```
使用`radare2 -L`可以列出radare2中的IO插件：
```
$ r2 -L
rw_  ar       Open ar/lib files [ar|lib]://[file//path] (LGPL3)
rw_  bfdbg    BrainFuck Debugger (bfdbg://path/to/file) (LGPL3)
rwd  bochs    Attach to a BOCHS debugger (LGPL3)
r_d  debug    Native debugger (dbg:///bin/ls dbg://1388 pidof:// waitfor://) (LGPL3) v0.2.0 pancake
rw_  default  open local files using def_mmap:// (LGPL3)
rwd  gdb      Attach to gdbserver, 'qemu -s', gdb://localhost:1234 (LGPL3)
rw_  gprobe   open gprobe connection using gprobe:// (LGPL3)
rw_  gzip     read/write gzipped files (LGPL3)
rw_  http     http get (http://rada.re/) (LGPL3)
rw_  ihex     Intel HEX file (ihex://eeproms.hex) (LGPL)
r__  mach     mach debug io (unsupported in this platform) (LGPL)
rw_  malloc   memory allocation (malloc://1024 hex://cd8090) (LGPL3)
rw_  mmap     open file using mmap:// (LGPL3)
rw_  null     null-plugin (null://23) (LGPL3)
rw_  procpid  /proc/pid/mem io (LGPL3)
rwd  ptrace   ptrace and /proc/pid/mem (if available) io (LGPL3)
rwd  qnx      Attach to QNX pdebug instance, qnx://host:1234 (LGPL3)
rw_  r2k      kernel access API io (r2k://) (LGPL3)
rw_  r2pipe   r2pipe io plugin (MIT)
rw_  r2web    r2web io client (r2web://cloud.rada.re/cmd/) (LGPL3)
rw_  rap      radare network protocol (rap://:port rap://host:port/file) (LGPL3)
rw_  rbuf     RBuffer IO plugin: rbuf:// (LGPL)
rw_  self     read memory from myself using 'self://' (LGPL3)
rw_  shm      shared memory resources (shm://key) (LGPL3)
rw_  sparse   sparse buffer allocation (sparse://1024 sparse://) (LGPL3)
rw_  tcp      load files via TCP (listen or connect) (LGPL3)
rwd  windbg   Attach to a KD debugger (windbg://socket) (LGPL3)
rwd  winedbg  Wine-dbg io and debug.io plugin for r2 (MIT)
rw_  zip      Open zip files [apk|ipa|zip|zipall]://[file//path] (BSD)
```

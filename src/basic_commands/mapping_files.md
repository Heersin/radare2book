## 映射文件

Radare的I/O子系统允许您将文件内容映射到一个已加载二进制文件的I/O空间中。新映射的内容可以被放置在任意地址上。

`o`命令允许用户打开一个文件，该文件映射至偏移量0处。若其具有已知的二进制文件头，则radare2将其映射至虚拟地址。

有时，我们想重设一个二进制文件，或是将其加载/映射到一个不同的地址上，当我们启动r2时就可以用`-B`选项改变基址。但要注意的是，当我们打开一个含有未知文件头的文件时（比如bootloader），需要使用`-m`选项对其进行映射（或是将其作为`o`命令的参数传递）。

radare2可以打开文件，然后在任意位置映射文件的一部分，并设置一些属性，例如权限和名字。其通过加载和映射二进制文件所有的依赖库复现环境（如core文件，debug session），在针对这些场景时其是一个理想的工具。

打开文件（以及映射文件）是通过`o`命令完成的，我们可以看看帮助信息：

```
[0x00000000]> o?
|Usage: o [com- ] [file] ([offset])
| o                         list opened files
| o-1                       close file descriptor 1
| o-!*                      close all opened files
| o--                       close all files, analysis, binfiles, flags, same as !r2 --
| o [file]                  open [file] file in read-only
| o+ [file]                 open file in read-write mode
| o [file] 0x4000 rwx       map file at 0x4000
| oa[-] [A] [B] [filename]  Specify arch and bits for given file
| oq                        list all open files
| o*                        list opened files in r2 commands
| o. [len]                  open a malloc://[len] copying the bytes from current offset
| o=                        list opened files (ascii-art bars)
| ob[?] [lbdos] [...]       list opened binary files backed by fd
| oc [file]                 open core file, like relaunching r2
| of [file]                 open file and map it at addr 0 as read-only
| oi[-|idx]                 alias for o, but using index instead of fd
| oj[?]                     list opened files in JSON format
| oL                        list all IO plugins registered
| om[?]                     create, list, remove IO maps
| on [file] 0x4000          map raw file at 0x4000 (no r_bin involved)
| oo[?]                     reopen current file (kill+fork in debugger)
| oo+                       reopen current file in read-write
| ood[r] [args]             reopen in debugger mode (with args)
| oo[bnm] [...]             see oo? for help
| op [fd]                   prioritize given fd (see also ob)
| ox fd fdx                 exchange the descs of fd and fdx and keep the mapping
```

prepare a simple layout:

```sh
$ rabin2 -l /bin/ls
[Linked libraries]
libselinux.so.1
librt.so.1
libacl.so.1
libc.so.6

4 libraries
```

打开一个文件:

```
[0x00001190]> o /bin/zsh 0x499999
```

列出已映射的文件:

```
[0x00000000]> o
- 6 /bin/ls @ 0x0 ; r
- 10 /lib/ld-linux.so.2 @ 0x100000000 ; r
- 14 /bin/zsh @ 0x499999 ; r
```

打印/bin/zsh中的十六进制字节:

```
[0x00000000]> px @ 0x499999
```

使用`o-`命令可以取消文件映射，将对应的文件描述符作为参数传递即可。

```
[0x00000000]> o-14
```

类似的，可以用如下命令查看以ascii风格的表格查看已打开的文件：

```
[0x00000000]> ob=
```

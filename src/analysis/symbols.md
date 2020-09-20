# Symbols

Radare2会自动解析二进制文件中的导入和导出部分，此外，radare2也支持加载调试信息文件，主要是两种格式：DWARF和PDB（针对windows二进制文件）。与许多工具不同，radare2不依赖于Windows API对PDB文件进行解析，因此在支持radare2的平台上都可以加载PDB文件，例如Linux和OS X。

默认情况下会自动加载DWARF调试信息，因为DWARF调试信息会存储在可执行文件中。只有PDB与众不同，它始终以独立的二进制文件存在，因此需要用不同的程序逻辑去处理它。

常见的一种场景是对来自windows发行版的文件进行分析，在此情况下所有的PDB文件都可以从Microsoft server上获得， 这也是radare2分析pdb文件时默认采用的选项。 下面展示的是radare2 所有pdb配置选项：

```
pdb.autoload = 0
pdb.extract = 1
pdb.server = https://msdl.microsoft.com/download/symbols
pdb.useragent = Microsoft-Symbol-Server/6.11.0001.402
```

使用`pdb.server`变量可以设置radare2获取PDB文件的地址， radare2可以通过存储在可执行文件头部的GUID尝试获取对应的PDB文件。可以通过分号分隔多个服务器的地址:
```
e pdb.server = https://msdl.microsoft.com/download/symbols;https://symbols.mozilla.org/
```
在Windows上， 也可以用本地网络共享的路径（UNC paths）作为symbol服务器。

通常情况下不需要改变默认的`pdb.useragent`变量，不过谁知道会不会有用上它的一天呢。

由于服务器上的PDB文件通常以"cab"格式储存，指定`pdb.extract=1`代表自动解压这些文件。要注意，这些需要系统上存在"cabextract"工具，以及wget或curl。

你也可以不在radare2完成这些工作， rabin2中有两个很方便的选项：

```
 -P              show debug/pdb information
 -PP             download pdb file for binary
```

`-PP`根据上面那些`pdb.*`选项，自动下载二进制文件对应的pdb文件。`-P`将会转储PDB文件内的内容，在快速了解PDB中存储的符号时还是很有用的。

除了基本的文件打开等操作之外， 还可以通过`id`命令对PDB文件进行操作：

```
[0x000051c0]> id?
|Usage: id Debug information
| Output mode:
| '*'              Output in radare commands
| id               Source lines
| idp [file.pdb]   Load pdb file information
| idpi [file.pdb]  Show pdb file information
| idpd             Download pdb file on remote server
```

`idpi`与`rabin2 -P`是等价的。`idp`除了可以在静态分析中使用，也可以在debug模式下使用，即使是通过WinDbg调试进程时也可以。

为了简化加载PDB文件的工作，尤其对于那些具有许多DLL的进程，radare2可以自动加载需要的PDB，可以用`e pdb.autoload=true`选项启用这个特性。接着如果需要在Windows对一些文件进行调试，可以用`r2 -d file.exe`或`r2 -d 2345`（attach到pid 2345）， 所有相关的PDB文件就会被自动加载了。

另一方面， DWARF信息完全采用自动加载的方式，不需要运行任何命令或设置任何选项：

```
r2 `which rabin2`
[0x00002437 8% 300 /usr/local/bin/rabin2]> pd $r
0x00002437  jne 0x2468                  ;[1]
0x00002439  cmp qword reloc.__cxa_finalize_224, 0
0x00002441  push rbp
0x00002442  mov rbp, rsp
0x00002445  je 0x2453                   ;[2]
0x00002447  lea rdi, obj.__dso_handle   ; 0x207c40 ; "@| "
0x0000244e  call 0x2360                 ;[3]
0x00002453  call sym.deregister_tm_clones ;[4]
0x00002458  mov byte [obj.completed.6991], 1 ; obj.__TMC_END__ ; [0x2082f0:1]=0
0x0000245f  pop rbp
0x00002460  ret
0x00002461  nop dword [rax]
0x00002468  ret
0x0000246a  nop word [rax + rax]
;-- entry1.init:
;-- frame_dummy:
0x00002470  push rbp
0x00002471  mov rbp, rsp
0x00002474  pop rbp
0x00002475  jmp sym.register_tm_clones  ;[5]
;-- blob_version:
0x0000247a  push rbp                    ; ../blob/version.c:18
0x0000247b  mov rbp, rsp
0x0000247e  sub rsp, 0x10
0x00002482  mov qword [rbp - 8], rdi
0x00002486  mov eax, 0x32               ; ../blob/version.c:24 ; '2'
0x0000248b  test al, al                 ; ../blob/version.c:19
0x0000248d  je 0x2498                   ;[6]
0x0000248f  lea rax, str.2.0.1_182_gf1aa3aa4d ; 0x60b8 ; "2.0.1-182-gf1aa3aa4d"
0x00002496  jmp 0x249f                  ;[7]
0x00002498  lea rax, 0x000060cd
0x0000249f  mov rsi, qword [rbp - 8]
0x000024a3  mov r8, rax
0x000024a6  mov ecx, 0x40               ; section_end.ehdr
0x000024ab  mov edx, 0x40c0
0x000024b0  lea rdi, str._s_2.1.0_git__d___linux_x86__d_git._s_n ; 0x60d0 ; "%s 2.1.0-git %d @ linux-x86-%d git.%s\n"
0x000024b7  mov eax, 0
0x000024bc  call 0x2350                 ;[8]
0x000024c1  mov eax, 0x66               ; ../blob/version.c:25 ; 'f'
0x000024c6  test al, al
0x000024c8  je 0x24d6                   ;[9]
0x000024ca  lea rdi, str.commit:_f1aa3aa4d2599c1ad60e3ecbe5f4d8261b282385_build:_2017_11_06__12:18:39 ; ../blob/version.c:26 ; 0x60f8 ; "commit: f1aa3aa4d2599c1ad60e3ecbe5f4d8261b282385 build: 2017-11-06__1
0x000024d1  call sym.imp.puts           ;[?]
0x000024d6  mov eax, 0                  ; ../blob/version.c:28
0x000024db  leave                       ; ../blob/version.c:29
0x000024dc  ret
;-- rabin_show_help:
0x000024dd  push rbp                    ; .//rabin2.c:27
```

正如所见的那样， 它将函数名和源码行标信息都加载进来了。


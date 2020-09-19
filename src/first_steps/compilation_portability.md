## 编译和可移植性

目前为止， radare2的核心组件可在多种架构和系统上完成编译，但是大部分的开发工作主要还是在GNU/Linux上通过GCC完成，以及在MacOS X上通过clang完成。Radare则能够在多种系统和架构上完成编译（包括TCC和SunStudio）。

用户常希望能够在逆向工程中将radare作为debugger使用，目前，radare2的debugger层可以在Windows, GNU/Linux (Intel x86 and x86_64, MIPS, and ARM), OS X, FreeBSD, NetBSD, and OpenBSD (Intel x86 and x86_64)正常使用。

与radare2核心相比，调试器的可移植性受到的限制更多，如果调试器暂时没有移植到您喜欢的平台上，那么在编译radare2时可以使用--without-debugger选项禁用调试器层的功能。

要注意的是，radare2中的一些I/O插件会使用GDB，WinDbg或Wine作为后端，因此这些插件需要有这些第三方工具的存在才能正常工作。（在远程调试中，仅需要目标主机上存在这些工具）

若要在系统上使用 `acr` 和 `GNU Make` 进行编译的话（例如在*BSD系统上）：
```
$ ./configure --prefix=/usr
$ gmake
$ sudo gmake install
```
项目中也有一个脚本可以自动完成这些工作:
```
$ sys/install.sh
```
### 静态构建

可以通过如下命令，借助其它的一些工具完成radare2的静态编译。
```
$ sys/static.sh
```
### Docker

Radare2 仓库维护了一个 [Dockerfile](https://github.com/radareorg/radare2/blob/master/Dockerfile)。

这个dockerfile同样被来自SANS的Remnux版本所使用，可以在[registryhub](https://registry.hub.docker.com/u/remnux/radare2/)找到它

## 清理旧版本的Radare2
```
./configure --prefix=/old/r2/prefix/installation
make purge
```


## 下载radare2

从[http://radare.org](http://radare.org)获取Radare2,
或者通过github repo: [https://github.com/radareorg/radare2](https://github.com/radareorg/radare2)

二进制文件包可以适用于多种操作系统(Ubuntu, Maemo, Gentoo, Windows, iPhone, and so on），但是强烈建议您使用源码编译的方式获得它，这样对于radare2的依赖会有一个更好的理解，且能够比较方便地找到示例，当然，好处还在于能够用上最新版的radare2。

通常每个月都会发布一个新的stable版本，有时候在[http://bin.rada.re/](http://bin.rada.re/)也会有Nightly tarballs释出。

radare2的开发仓库版本通常来说比‘stable’版本更加稳定，可以用如下命令获得最新的版本：
```
$ git clone https://github.com/radareorg/radare2.git
```
可能需要一些时间，趁这个时候喝杯咖啡，然后继续阅读本书吧。

可以在本地的radare2仓库内任意位置使用 `git pull` 更新radare2仓库。
```
$ git pull
```
如果你在本地对源码进行了修改，通过如下命令还原（会失去修改）：
```
$ git reset --hard HEAD
```
或者，给我们发一个patch：
```
$ git diff > radare-foo.patch
```
更新radare2并在系统内进行安装最常用的方法还是下面这个：
```
$ sys/install.sh
```
### Building with meson + ninja

There is also a work-in-progress support for Meson.

Using clang and ld.gold makes the build faster:
```bash
CC=clang LDFLAGS=-fuse-ld=gold meson . release --buildtype=release --prefix ~/.local/stow/radare2/release
ninja -C release
# ninja -C release install
```

### 帮助脚本

您可以看看 `sys/`中的脚本，他们用于一些自动化的工作，例如同步、构建以及安装r2，以及一些绑定工作。

最终要的当属 `sys/install.sh`， 这个脚本会完成pull，clean，build以及在系统上进行symstall安装r2.


Symstall 即通过在系统内创建符号链接完成程序、库、文档和数据文件的安装， 而非通过复制项目文件进行安装。

默认情况下它会被安装在 `/usr/local`, 不过你也可以通过指定 `--prefix` 选择其他目录进行安装。

对于开发者来说这种sysmstall安装方式很有用，因为该方式使得开发者只需要运行'make'，然后就可以调试新增加的功能，而不需要运行make install。

### 清理

清理源码目录是很重要的，这有助于避免在某些ABI改变后，链接依旧指向未更新或者旧版的文件。下面的命令能帮助你保持git repo的处于最新版本：
```
$ git clean -xdf
$ git reset --hard @~10
$ git pull
```
可以通过如下命令，卸载系统上旧版本的radare2
```
$ ./configure --prefix=/usr/local
$ make purge
```

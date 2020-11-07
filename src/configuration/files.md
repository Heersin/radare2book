## radare2相关文件

`r2 -H`命令可以列出所有重要的环境变量，并了解r2在哪些路径下搜索需要的文件。这些路径具体是取决于构建r2的操作系统以及构建时采用的方法和配置。
```
R2_PREFIX=/usr
MAGICPATH=/usr/share/radare2/2.8.0-git/magic
PREFIX=/usr
INCDIR=/usr/include/libr
LIBDIR=/usr/lib64
LIBEXT=so
RCONFIGHOME=/home/user/.config/radare2
RDATAHOME=/home/user/.local/share/radare2
RCACHEHOME=/home/user/.cache/radare2
LIBR_PLUGINS=/usr/lib/radare2/2.8.0-git
USER_PLUGINS=/home/user/.local/share/radare2/plugins
USER_ZIGNS=/home/user/.local/share/radare2/zigns
```

## RC配置文件

RC文件是在启动时加载的r2脚本，该文件至少须放在下面三个位置之一:

### System

radare2首先会尝试加载/usr/share/radare2/radare2rc

### Home目录

系统上的每个用户都可以拥有自己的r2启动脚本，用于配置色彩主题及其他自定义的选项

* ~/.radare2rc
* ~/.config/radare2/radare2rc
* ~/.config/radare2/radare2rc.d/

### 目标文件的目录

如果想在每次打开文件时都运行某个脚本，那么就在对应位置创建一个同名文件，并在文件名后添加一个`.r2`后缀。
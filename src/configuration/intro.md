# 配置

Radare2核心在启动时会读取`~/.config/radare2/radare2rc`，可以在该文件中添加`e`类命令，根据自己的喜好调整radare2配置。

如果不想Radare2在启动时解析该配置文件，启动时就要指定`-N`选项。

radare2中一切配置都是通过`eval`命令实现的，典型的启动配置文件长这样:
```sh
$ cat ~/.radare2rc
e scr.color = 1
e dbg.bep   = loader
```
也可以通过在命令行指定`-e` <config=value>选项修改配置，这个方法可以在保持.radare2.rc文件不变的情况下调整配置。例如下面的这个命令将在不读取配置文件的情况下启动radare2，并修改`scr.color`和`asm.syntax`变量。
```sh
$ radare2 -N -e scr.color=1 -e asm.syntax=intel -d /bin/ls
```
在r2内部，配置存在一个哈希表里，分割为不同的命名空间：`cfg.`, `file.`, `dbg.`, `scr.`等等。

获取所有的配置变量只需在命令行中敲入`e`，若仅想显示指定空间中的变量就在`e`后面加上对应的名称，以`.`作为结尾。例如`e file.`会显示"file"命名空间中定义的所有变量。

`e?`可以了解相应的帮助信息
```
Usage: e [var[=value]]  Evaluable vars
| e?asm.bytes     show description
| e??             list config vars with description
| e a             get value of var 'a'
| e a=b           set var 'a' the 'b' value
| e var=?         print all valid values of var
| e var=??        print all valid values of var with description
| e.a=b           same as 'e a=b' but without using a space
| e,k=v,k=v,k=v   comma separated k[=v]
| e-              reset config vars
| e*              dump config vars in r commands
| e!a             invert the boolean value of 'a' var
| ec [k] [color]  set color for given key (prompt, offset, ...)
| eevar           open editor to change the value of var
| ed              open editor to change the ~/.radare2rc
| ej              list config vars in JSON
| env [k[=v]]     get/set environment variable
| er [key]        set config key as readonly. no way back
| es [space]      list all eval spaces [or keys]
| et [key]        show type of given config variable
| ev [key]        list config vars in verbose format
| evj [key]       list config vars in verbose format in JSON
```

`e`命令有一个更易用的图形化替代品，可以通过`Ve`命令进入，使用方向键(上，下，左，右)在配置变量间进行切换，按下`q`可以退出。编辑配置的初始界面看起来是这样的：

```
[EvalSpace]

    >  anal
       asm
       scr
       asm
       bin
       cfg
       diff
       dir
       dbg
       cmd
       fs
       hex
       http
       graph
       hud
       scr
       search
       io
```

对于一些值存在选择范围的配置变量来说，可以用`=?`列出可选的值:

```
[0x00000000]> e scr.nkey = ?
scr.nkey = fun, hit, flag
```

如果不确定配置变量对应的效果，可以用`e?[conf_var]`显示该变量对应的描述:
If you are not sure about the effect of configuration, you can use `e?[conf_var]` to display the corresponding description.
```
[0x00005b20]> e?scr.nkey
            scr.nkey: Select visual seek mode (affects n/N visual commands)
```

显示所有配置变量的描述然后从中抓取所想要的特性/配置也很方便，下面的这个命令获取`scr.`下所有命令的描述，然后仅展示其中包含`utf`的行。
It's also very convenient to list all the configuration variables and grep the features/configuration you want.
The following command fetches the description to variables under `scr.` and display only those lines contain `utf`.
```
[0x00005b20]> e?scr ~utf
            scr.utf8: Show UTF-8 characters instead of ANSI
      scr.utf8.curvy: Show curved UTF-8 corners (requires scr.utf8)
```
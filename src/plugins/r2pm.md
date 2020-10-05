# 为插件创建r2pm包

radare2有自己的[包管理器](../tools/r2pm/intro.md)，可以很容易地向里面添加新写的插件，其他用户可以从该仓库下载。

所有的包都放在[radare2-pm](https://github.com/radareorg/radare2-pm)仓库里，包的文本格式都很简单：

```
R2PM_BEGIN

R2PM_GIT "https://github.com/user/mycpu"
R2PM_DESC "[r2-arch] MYCPU disassembler and analyzer plugins"

R2PM_INSTALL() {
	${MAKE} clean
	${MAKE} all || exit 1
	${MAKE} install R2PM_PLUGDIR="${R2PM_PLUGDIR}"
}

R2PM_UNINSTALL() {
	rm -f "${R2PM_PLUGDIR}/asm_mycpu."*
	rm -f "${R2PM_PLUGDIR}/anal_mycpu."*
}

R2PM_END
```

把它加入radare2-pm repo的`/db`目录下，然后向mainline发一个PR就行。
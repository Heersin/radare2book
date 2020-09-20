## 配置搜索选项

radare2的搜索引擎是通过配置中的一些变量进行设置的，可以用`e`命令修改他们。 
```
e cmd.hit = x         ; radare2 command to execute on every search hit
e search.distance = 0 ; search string distance
e search.in = [foo]   ; pecify search boundarie. Supported values are listed under e search.in=??
e search.align = 4    ; only show search results aligned by specified boundary.
e search.from = 0     ; start address
e search.to = 0       ; end address
e search.asmstr = 0   ; search for string instead of assembly
e search.flags = true ; if enabled, create flags on hits
```
`search.align`变量用于将搜索结果限制在对齐的地址上，例如使用`e search.align=4`会使得命中结果只出现在4字节对齐的地址上。

`search.flags`布尔变量设置搜索引擎是否标记命中结果，以便之后进行引用。如果用户在搜索过程中用`Ctrl-C`中断了搜索， 中断位置将会被标记为`search_stop`。

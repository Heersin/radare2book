## 脚本化

Radare2提供提供丰富的手段用于自动化处理一些无聊工作，从最简单的命令序列到通过IPC调用脚本或其他程序，这些称之为r2pipe。

如前所述，可以通过`;`符号对命令进行序列化
```
[0x00404800]> pd 1 ; ao 1
           0x00404800      b827e66100     mov eax, 0x61e627      ; "tab"
address: 0x404800
opcode: mov eax, 0x61e627
prefix: 0
bytes: b827e66100
ptr: 0x0061e627
refptr: 0
size: 5
type: mov
esil: 6415911,rax,=
stack: null
family: cpu
[0x00404800]>
```
同shell一样，其仅是简单地在执行完第一条命令后执行第二条命令。

第二种序列化命令的方式是使用简单的管道符`|`
```
ao|grep address
```
注意，`|`管道仅能将r2命令的输出导向至外部（shell）命令，比如系统程序或内置的shell命令。
这里还有相似的序列化r2命令的一种方法，就是使用反引号，`` `command` ``，被其包含的部分会进行命令替换，命令输出将被作为一个命令行参数使用。

比如，我们想要查看`mov eax, addr`这条指令里addr上的一些字节，无需跳转到该地址，只需要使用下面的命令序列：
```
[0x00404800]> pd 1
              0x00404800      b827e66100     mov eax, 0x61e627      ; "tab"
[0x00404800]> ao
address: 0x404800
opcode: mov eax, 0x61e627
prefix: 0
bytes: b827e66100
ptr: 0x0061e627
refptr: 0
size: 5
type: mov
esil: 6415911,rax,=
stack: null
family: cpu
[0x00404800]> ao~ptr[1]
0x0061e627
0
[0x00404800]> px 10 @ `ao~ptr[1]`
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x0061e627  7461 6200 2e69 6e74 6572                 tab..inter
[0x00404800]>
```
当然同样可以用`>`和`>>`将r2命令的输出结果重定向至文件里。
命令如下：
```
[0x00404800]> px 10 @ `ao~ptr[1]` > example.txt
[0x00404800]> px 10 @ `ao~ptr[1]` >> example.txt
```

Radare2同样提供了一些Unix类型的文件处理命令，比如head、tail、cat、grep等等。其中的一个命令是[Uniq](https://en.wikipedia.org/wiki/Uniq)，用于对文件进行过滤，并输出其中非重复的部分。若需要创建一个仅包含非重复内容的文件，可以这样：
```
[0x00404800]> uniq file > uniq_file
```

[head](https://en.wikipedia.org/wiki/Head_%28Unix%29)用于查看文件开头N行的内容，相似的[tail](https://en.wikipedia.org/wiki/Tail_(Unix))命令则允许查看最后N行的内容。
```
[0x00404800]> head 3 foodtypes.txt
1 Protein
2 Carbohydrate
3 Fat
[0x00404800]> tail 2 foodtypes.txt
3 Shake
4 Milk
```

[join](https://en.wikipedia.org/wiki/Join_%28Unix%29)可以用于将两个具有相同字段的文件合并。
```
[0x00404800]> cat foodtypes.txt
1 Protein
2 Carbohydrate
3 Fat
[0x00404800]> cat foods.txt
1 Cheese 
2 Potato
3 Butter
[0x00404800]> join foodtypes foods.txt
1 Protein Cheese
2 Carbohydrate Potato
3 Fat Butter
```

类似的，对文件排序可以通过[sort](https://en.wikipedia.org/wiki/Sort_%28Unix%29)命令实现，典型的例子如下：
```
[0x00404800]> sort file
eleven
five
five
great
one
one
radare
```

`?$?`命令描述了几个有用的命令，帮助你更容易地完成类似工作。比如`$v`代表"immediate value"变量，`$m`代表opcode中的内存引用变量。


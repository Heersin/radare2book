## 基本的搜索方法

在文件中搜索一段文本的基本方法如下：

    $ r2 -q -c "/ lib" /bin/ls
    Searching 3 bytes from 0x00400000 to 0x0041ae08: 6c 69 62 
    hits: 9
    0x00400239 hit0_0 "lib64/ld-linux-x86-64.so.2"
    0x00400f19 hit0_1 "libselinux.so.1"
    0x00400fae hit0_2 "librt.so.1"
    0x00400fc7 hit0_3 "libacl.so.1"
    0x00401004 hit0_4 "libc.so.6"
    0x004013ce hit0_5 "libc_start_main"
    0x00416542 hit0_6 "libs/"
    0x00417160 hit0_7 "lib/xstrtol.c"
    0x00417578 hit0_8 "lib"

从上面的输出信息可以看出，radare2会为找到的每个条目生成一个"hit"标志， 可以使用`ps`命令查看这组条目对应地址上的字符串，条目的名称格式为`hit0_ <index>`：

    [0x00404888]> / ls
    ...
    [0x00404888]> ps @ hit0_0
    lseek

可以用`/w`命令搜索宽字符串（例如unicode字符）：

    [0x00000000]> /w Hello
    0 results found.

若在搜索中需要区分大小写， 使用`/i`：

    [0x0040488f]> /i Stallman
    Searching 8 bytes from 0x00400238 to 0x0040488f: 53 74 61 6c 6c 6d 61 6e
    [# ]hits: 004138 < 0x0040488f  hits = 0

在搜索字符串时也可以结合十六进制字节， 以"\x"作为转义符：

    [0x00000000]> / \x7FELF

不过如果您想要搜索十六进制串的话， 更好的做法使用`/x`命令：

    [0x00000000]> /x 7F454C46

搜索完成后，结果将存储在`searches`标志空间内。

    [0x00000000]> fs
    0    0 . strings
    1    0 . symbols
    2    6 . searches

    [0x00000000]> f
    0x00000135 512 hit0_0
    0x00000b71 512 hit0_1
    0x00000bad 512 hit0_2
    0x00000bdd 512 hit0_3
    0x00000bfb 512 hit0_4
    0x00000f2a 512 hit0_5

如果不需要搜索结果， 使用`f-hit*`可以移除所有的"hit"标记。

在漫长的搜索会话中，可能需要频繁执行最新一次的搜索，在radare2中可以用`//`重复上一次的搜索。

    [0x00000f2a]> //     ; repeat last search

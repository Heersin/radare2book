# User Interfaces

Radare2多年来见证了许多不同的用户界面的开发。

维护GUI不属于开发逆向工程工具包的核心功能的范围：最好有一个单独的项目和社区，允许两个项目进行协作并共同改进-而不是强迫cli开发人员考虑gui问题 并让他们在图形和底层逻辑之间来回跳转。

过去，r2至少有5种不同的原生用户界面（ragui，r2gui，gradare，r2net，bokken），但是都没有足够的维护能力来支撑它们发展，因而这些项目全部都消亡了。

此外r2具有嵌入式Web服务器，并附带了一些用html/js编写的基本用户界面。可以像这样启动它们：
```
$ r2 -c=H /bin/ls
```
经过3年的私人开发，Hugo Teso; Bokken（r2的python-gtk gui）的作者向公众发布了r2的另一个前端，这次是用c ++和qt编写的，受到了社区的欢迎。

这个GUI被命名为Iaito，但当他不想在维护该项目时，Xarkes便决定以Cutter（社区投票的名称）的名义fork它，并领导该项目。目前看起来是这样的：

* [https://github.com/radareorg/cutter](https://github.com/radareorg/cutter).

![Cutter](Cutter.png)

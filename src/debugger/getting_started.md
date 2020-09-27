# Getting Started

## Small session in radare2 debugger

* `r2 -d /bin/ls`: 在radare2原生debugger中以debug模式打开 `/bin/ls`,但并不会运行该程序。你将会看到 (radare2) 这样一个提示符 - 所有的示例都将从该提示符开始。

* `db flag`: 在一个flag处打上断点，flag可以是一个地址也可以是函数名。

* `db - flag`: 移除flag位置上的断点，flag可以是一个地址也可以是函数名。

* `db`: 列出现有的断点

* `dc`: 运行程序

* `dr`: 显示寄存器的状态

* `drr`: 显示寄存器的引用 (telescoping) (类似 peda 中那样)

* `ds`: 步入指令

* `dso`: 步过指令

* `dbt`: 显示回溯信息（backtrace）

* `dm`: 显示内存映射

* `dk <signal>`: 向child进程发送KILL信号

* `ood`: 以debug模式重新打开

* `ood arg1 arg2`: 以debug模式重新打开，但带上arg1,arg2参数

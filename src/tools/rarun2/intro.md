# Rarun2

Rarun2是用于设置一个特定执行环境的工具，例如重新定义stdin/stdout，管道，修改环境变量，以及其他有助于为调试创造边界条件的种种设定。

```
$ rarun2 -h
Usage: rarun2 -v|-t|script.rr2 [directive ..]
```

其接受`key=value`这样格式的文本文件以设置执行环境。
Rarun2可以用作一个独立的工具，也可作为radare2的一部分使用。可以用radare2的`-r`选项加载rarun2的配置文件，或者使用`-R`直接依据字符串文本进行设置。

配置文件的格式很简单，记住最重要的两个键 - `program`和`arg*`

最常见的使用场景 - 将radare2中所要调试的程序进行输出重定向。
这需要用到`stdio`，`stdout`，`stdin`，`input`以及其他类似的键。

一个基础的配置文件:

```
program=/bin/ls
arg1=/bin
# arg2=hello
# arg3="hello\nworld"
# arg4=:048490184058104849
# arg5=:!ragg2 -p n50 -d 10:0x8048123
# arg6=@arg.txt
# arg7=@300@ABCD # 300 chars filled with ABCD pattern
# system=r2 -
# aslr=no
setenv=FOO=BAR
# unsetenv=FOO
# clearenv=true
# envfile=environ.txt
timeout=3
# timeoutsig=SIGTERM # or 15
# connect=localhost:8080
# listen=8080
# pty=false
# fork=true
# bits=32
# pid=0
# pidfile=/tmp/foo.pid
# #sleep=0
# #maxfd=0
# #execve=false
# #maxproc=0
# #maxstack=0
# #core=false
# #stdio=blah.txt
# #stderr=foo.txt
# stdout=foo.txt
# stdin=input.txt # or !program to redirect input from another program
# input=input.txt
# chdir=/
# chroot=/mnt/chroot
# libpath=$PWD:/tmp/lib
# r2preload=yes
# preload=/lib/libfoo.so
# setuid=2000
# seteuid=2000
# setgid=2001
# setegid=2001
# nice=5
```

# 远程访问的支持

Radare可以在本地运行，也可以作为服务器进程启动，由本地的radare2控制。这个方案之所以可行，是因为它们都使用了Radar的IO子系统，该子系统抽象了对system（），cmd（）和所有基本IO操作的访问，从而使radare2可以通过网络工作。

一些与radare远程访问相关的帮助信息：

```
[0x00405a04]> =?
Usage:  =[:!+-=ghH] [...]   # connect with other instances of r2

remote commands:
| =                             list all open connections
| =<[fd] cmd                    send output of local command to remote fd
| =[fd] cmd                     exec cmd at remote 'fd' (last open is default one)
| =! cmd                        run command via r_io_system
| =+ [proto://]host:port        connect to remote host:port (*rap://, raps://, tcp://, udp://, http://)
| =-[fd]                        remove all hosts or host 'fd'
| ==[fd]                        open remote session with host 'fd', 'q' to quit
| =!=                           disable remote cmd mode
| !=!                           enable remote cmd mode

servers:
| .:9000                        start the tcp server (echo x|nc ::1 9090 or curl ::1:9090/cmd/x)
| =:port                        start the rap server (o rap://9999)
| =g[?]                         start the gdbserver
| =h[?]                         start the http webserver
| =H[?]                         start the http webserver (and launch the web browser)

other:
| =&:port                       start rap server in background (same as '&_=h')
| =:host:port cmd               run 'cmd' command on remote server

examples:
| =+tcp://localhost:9090/       connect to: r2 -c.:9090 ./bin
| =+rap://localhost:9090/       connect to: r2 rap://:9090
| =+http://localhost:9090/cmd/  connect to: r2 -c'=h 9090' bin
| o rap://:9090/                start the rap server on tcp port 9090
```

可以通过`radare2 -L`列出的IO插件了解radare2的远程访问能力。通过例子来说明可能会更好理解一些，一个典型的remote session看起来就像这样：

远程主机1（host1）:

```
$ radare2 rap://:1234
```

远程主机2（host2）:

```
$ radare2 rap://:1234
```

本地主机:

```
$ radare2 -
```

在本地添加远程主机：

```
[0x004048c5]> =+ rap://<host1>:1234//bin/ls
Connected to: <host1> at port 1234
waiting... ok

[0x004048c5]> =
0 - rap://<host1>:1234//bin/ls
```

可以通过在添加主机时指定URI以debug模式（或者使用其他IO插件）打开远程主机上的文件：

```
[0x004048c5]> =+ =+ rap://<host2>:1234/dbg:///bin/ls
Connected to: <host2> at port 1234
waiting... ok
0 - rap://<host1>:1234//bin/ls
1 - rap://<host2>:1234/dbg:///bin/ls
```

在host1上执行命令:

```
[0x004048c5]> =0 px
[0x004048c5]> = s 0x666
```

打开与host2的session:

```
[0x004048c5]> ==1
fd:6> pi 1
...
fd:6> q
```

移除主机 (关闭连接):

```
[0x004048c5]> =-
```

还可以将radare的输出重定向至TCP/UDP服务器（就如使用`nc -l`）。首先使用`=+ tcp://` 或 `=+ udp://`添加服务器，接着就可以用如下命令将输出重定向至该服务器：

```
[0x004048c5]> =+ tcp://<host>:<port>/
Connected to: <host> at port <port>
5 - tcp://<host>:<port>/
[0x004048c5]> =<5 cmd...
```

`=<`命令会将`cmd`的执行结果输出到远程连接N上（如果没有指定N，就会输出到最后一次使用的连接上）。

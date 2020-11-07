# SDB

SDB的含义是String DataBase，它是一个由pancake开发的仅用于操作字符串的键值对数据库。因其小巧而高效，可作为steroids中的哈系表，r2中的许多地方都用其作为磁盘数据库或内存数据库。

SDB是一个简单的字符串键值数据库，基于djb的cdb磁盘存储开发。其还支持JSON和数组对象的自省。

sdbtypes则是一个vala库，在sdb或memcache实例之上实现了多种数据结构。

SDB 支持：
    
- namespaces (multiple sdb paths)
- atomic database sync (never corrupted)
- bindings for vala, luvit, newlisp and nodejs
- commandline frontend for sdb databases
- memcache client and server with sdb backend
- arrays support (syntax sugar)
- json parser/getter


## 用法示例
我们先创建一个数据库！
```
$ sdb d hello=world
$ sdb d hello
world
```

使用数组
```
$ sdb - '[]list=1,2' '[0]list' '[0]list=foo' '[]list' '[+1]list=bar'
1
foo
2
foo
bar
2
```

试试json
```
$ sdb d g='{"foo":1,"bar":{"cow":3}}'
$ sdb d g?bar.cow
3
$ sdb - user='{"id":123}' user?id=99 user?id
99
```

不使用磁盘数据库的情况下使用命令行:
```
$ sdb - foo=bar foo a=3 +a -a
bar
4
3

$ sdb -
foo=bar
foo
bar
a=3
+a
4
-a
3
```
移除数据库
```
$ rm -f d

```

## 还有呢？
现在可以在radare2 session中完成这些了！

我们以一个小文件作为例子讲解，看看有哪些东西已经被sdb化(_sdbized_)了
```
$ cat test.c
int main(){
	puts("Hello world\n");
}
$ gcc test.c -o test
```

```
$ r2 -A ./test
[0x08048320]> k **
bin
anal
syscall
debug
```

```
[0x08048320]> k bin/**
fd.6
[0x08048320]> k bin/fd.6/*
archs=0:0:x86:32
```

对应于第六个文件描述符的文件是x86_32二进制文件。

```
[0x08048320]> k anal/meta/*
meta.s.0x80484d0=12,SGVsbG8gd29ybGQ=
[...]
[0x08048320]> ?b64- SGVsbG8gd29ybGQ=
Hello world
```
string是经过Base64编码后保存的。

---

## 更多的例子


列出所有命名空间
```
k **
```
列出所有子命名空间
```
k anal/**
```
列出所有key
```
k *
k anal/*
```
设置key
```
k foo=bar
```
获取key对应的值
```
k foo
```
列出所有syscall
```
k syscall/*~^0x
```
列出所有注释
```
k anal/meta/*~.C.
```
显示给定偏移位置上的注释
```
k %anal/meta/[1]meta.C.0x100005000
```

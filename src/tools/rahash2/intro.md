# rahash2

rahash2可以用来计算文件、磁盘设备或字符串的校验和，支持多种不同的哈希算法进行块哈希或全哈希。

这个工具也可用来完成一些诸如base64,异或等编解码的操作。

这里有一个用例：
```
$ rahash2 -a md5 -s "hello world"
```

这里提一下rahash2支持从stdin中流式地读取数据，因此计算一个4GB文件的哈希值并不需要4GB的内存空间。

## 块哈希

进行取证工作时，常需要计算部分校验和，这么做的原因是你可能想将大文件拆分文件多个小文件，以便于通过文件内容或所在磁盘位置对这些文件进行区分。这样，我们便能通过相同哈希值找到包含相同内容的块，比如说用来检查某个块内的数据是否全为0。

这个功能也可用于检查在两次dump之间是否有哪个块被修改了。在分析虚拟机中dump下来的内存时这个功能就很有用，其对应的命令示例如下：

```
$ rahash2 -B 1M -b -a sha256 /bin/ls
```

## 配合rabin2进行哈希

rabin2工具不仅可以解析二进制文件头，还可借助rhash插件计算二进制文件中区段对应的校验和。

```
$ rabin2 -K md5 -S /bin/ls
```

## 在radare2 session中获取哈希值

用`ph`命令可以在radare2中计算当前块的校验和，需将要使用哈希算法作为参数传递给该命令。例子：

```
$ radare2 /bin/ls
[0x08049790]> bf entry0
[0x08049790]> ph md5
d2994c75adaa58392f953a448de5fba7
```

可以使用`rahash2`支持的所有哈希算法：

```
[0x00000000]> ph?
md5
sha1
sha256
sha384
sha512
md4
xor
xorpair
parity
entropy
hamdist
pcprint
mod255
xxhash
adler32
luhn
crc8smbus
crc15can
crc16
crc16hdlc
crc16usb
crc16citt
crc24
crc32
crc32c
crc32ecma267
crc32bzip2
crc32d
crc32mpeg2
crc32posix
crc32q
crc32jamcrc
crc32xfer
crc64
crc64ecma
crc64we
crc64xz
crc64iso
```

`ph`命令还可接受一个可选的数字参数，用于指定要哈希的字节范围长度，代替原本的默认块大小，例如：

```
[0x08049A80]> ph md5 32
9b9012b00ef7a94b5824105b7aaad83b
[0x08049A80]> ph md5 64
a71b087d8166c99869c9781e2edcf183
[0x08049A80]> ph md5 1024
a933cc94cd705f09a41ecc80c0041def
```
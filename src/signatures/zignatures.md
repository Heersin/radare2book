# 签名

radare2有自己的签名格式，可以在创建时即时地进行加载/应用。这些命令存在于`z`命令空间下：

```
[0x00000000]> z?
Usage: z[*j-aof/cs] [args]   # Manage zignatures
| z            show zignatures
| z.           find matching zignatures in current offset
| zb[?][n=5]   search for best match
| z*           show zignatures in radare format
| zq           show zignatures in quiet mode
| zj           show zignatures in json format
| zk           show zignatures in sdb format
| z-zignature  delete zignature
| z-*          delete all zignatures
| za[?]        add zignature
| zg           generate zignatures (alias for zaF)
| zo[?]        manage zignature files
| zf[?]        manage FLIRT signatures
| z/[?]        search zignatures
| zc[?]        compare current zignspace zignatures with another one
| zs[?]        manage zignspaces
| zi           show zignatures matching information
```
需要使用`zo`命令从SDB文件中加载签名然后创建签名，如果SDB文件经过压缩，则需要使用`zoz`命令。

创建签名前首先要创建函数，之后可以通过函数创建它：
```
r2 /bin/ls
[0x000051c0]> aaa # this creates functions, including 'entry0'
[0x000051c0]> zaf entry0 entry
[0x000051c0]> z
entry:
  bytes: 31ed4989d15e4889e24883e4f050544c............48............48............ff..........f4
  graph: cc=1 nbbs=1 edges=0 ebbs=1
  offset: 0x000051c0
[0x000051c0]>
```
可以看到，它从`entry0`函数中创建了一个名为`entry`的签名。此外你可以以JSON格式显示结果，JSON格式的结果在能够用于脚本之中:
```
[0x000051c0]> zj~{}
[
  {
    "name": "entry",
    "bytes": "31ed4989d15e4889e24883e4f050544c............48............48............ff..........f4",
    "graph": {
      "cc": "1",
      "nbbs": "1",
      "edges": "0",
      "ebbs": "1"
    },
    "offset": 20928,
    "refs": [
    ]
  }
]
[0x000051c0]>
```
使用`z-entry`可以移除签名。

如果想保存所有的签名，可以用`zos myentry`将签名存入SDB文件中。

之后可以加载该文件并应用之，首先重新打开一个文件：
```
r2 /bin/ls
 -- Log On. Hack In. Go Anywhere. Get Everything.
[0x000051c0]> zo myentry
[0x000051c0]> z
entry:
  bytes: 31ed4989d15e4889e24883e4f050544c............48............48............ff..........f4
  graph: cc=1 nbbs=1 edges=0 ebbs=1
  offset: 0x000051c0
[0x000051c0]>
```
这些消息代表成功得从`myentry`文件里加载签名，而且可以通过搜索找到对应函数：
```
[0x000051c0]> z.
[+] searching 0x000051c0 - 0x000052c0
[+] searching function metrics
hits: 1
[0x000051c0]>
```
注意`z.`命令仅仅对照当前的地址搜索签名，如果想要在全文件中进行签名搜索则需要一些不同的操作。
底下展示的是一个重要时刻，当我们试图搜索签名时 -- r2却找不到任何东西：
```
[0x000051c0]> z/
[+] searching 0x0021dfd0 - 0x002203e8
[+] searching function metrics
hits: 0
[0x000051c0]>
```
这是由于搜索地址配置的原因，我们需要先[调整搜索](../search_bytes/configurating_the_search.md)的范围：
```
[0x000051c0]> e search.in=io.section
[0x000051c0]> z/
[+] searching 0x000038b0 - 0x00015898
[+] searching function metrics
hits: 1
[0x000051c0]>
```
我们将搜索模式设置为`io.section`（默认的是`file`）用于在当前节区进行搜索（假设我们目前所处的位置是`.text`节区）。
现在来看看radare2为我们找到了什么信息：
```
[0x000051c0]> pd 5
;-- entry0:
;-- sign.bytes.entry_0:
0x000051c0      31ed           xor ebp, ebp
0x000051c2      4989d1         mov r9, rdx
0x000051c5      5e             pop rsi
0x000051c6      4889e2         mov rdx, rsp
0x000051c9      4883e4f0       and rsp, 0xfffffffffffffff0
[0x000051c0]>
```
现在我们看到了`entry0`的注释，其是在ELF解析阶段时添加的，同时还有一个`sign.bytes.entry_0`，它正是签名匹配的结果。

签名配置在`zign.`配置变量空间中：
```
[0x000051c0]> e? zign.
       zign.autoload: Autoload all zignatures located in ~/.local/share/radare2/zigns
          zign.bytes: Use bytes patterns for matching
   zign.diff.bthresh: Threshold for diffing zign bytes [0, 1] (see zc?)
   zign.diff.gthresh: Threshold for diffing zign graphs [0, 1] (see zc?)
          zign.graph: Use graph metrics for matching
           zign.hash: Use Hash for matching
          zign.maxsz: Maximum zignature length
          zign.mincc: Minimum cyclomatic complexity for matching
          zign.minsz: Minimum zignature length for matching
         zign.offset: Use original offset for matching
         zign.prefix: Default prefix for zignatures matches
           zign.refs: Use references for matching
      zign.threshold: Minimum similarity required for inclusion in zb output
          zign.types: Use types for matching
[0x000051c0]>
```

## 寻找最佳匹配 `zb`

常常你知道二进制文件中存在一个签名，但是`z/`和`z.`都无法找到。这通常是因为签名和函数之间的差异非常小导致的，有可能是编译器交换了两条指令，或者你搜索的签名对于该版本的函数并不适用。
在此情景下，`zb`命令能够帮你指向相似的匹配位置。

```
[0x000040a0]> zb?
Usage: zb[r?] [args]  # search for closest matching signatures
| zb [n]           find n closest matching zignatures to function at current offset
| zbr zigname [n]  search for n most similar functions to zigname
```

`zb`（zign best）命令会展示5个与函数最相近的签名，每个签名都会有一个打分，分值从1.0到0.0不等。

```
[0x0041e390]> s sym.fclose
[0x0040fc10]> zb
0.96032  0.92400 B  0.99664 G   sym.fclose
0.65971  0.35600 B  0.96342 G   sym._nl_expand_alias
0.65770  0.37800 B  0.93740 G   sym.fdopen
0.65112  0.35000 B  0.95225 G   sym.__run_exit_handlers
0.62532  0.34800 B  0.90264 G   sym.__cxa_finalize
```

在上面那个例子里，`zb`正确地将`sym.fclose`签名与当前函数相关联，而`z/`和`z.`命令则无法发现，这是因为其`B`yte和`G`raph的分值都小于1.0。
第一相近结果和第二相近结果之间30%的偏差度也意味着第一个结果是个正确的匹配。

`zbr`（zign best reverse）接受一个签名作为参数，然后找到最相近的匹配函数。在此之前记得先用`aa`发现函数。

```
[0x00401b20]> aa
[x] Analyze all flags starting with sym. and entry0 (aa)
[0x00401b20]> zo ./libc.sdb
[0x00401b20]> zbr sym.__libc_malloc 10
0.94873  0.89800 B  0.99946 G   sym.malloc
0.65245  0.40600 B  0.89891 G   sym._mid_memalign
0.59470  0.38600 B  0.80341 G   sym._IO_flush_all_lockp
0.59200  0.28200 B  0.90201 G   sym._IO_file_underflow
0.57802  0.30400 B  0.85204 G   sym.__libc_realloc
0.57094  0.35200 B  0.78988 G   sym.__calloc
0.56785  0.34000 B  0.79570 G   sym._IO_un_link.part.0
0.56358  0.36200 B  0.76516 G   sym._IO_cleanup
0.56064  0.26000 B  0.86127 G   sym.intel_check_word.constprop.0
0.55726  0.28400 B  0.83051 G   sym.linear_search_fdes
```

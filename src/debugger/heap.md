# 堆（Heap）

radare2的`dm`子命令同样可以显示heap分布，对于那些想要查看heap及其内部数据的用户来说很有用。只需执行`dmh`命令即可显示heap的分布：

```
[0x7fae46236ca6]> dmh
  Malloc chunk @ 0x55a7ecbce250 [size: 0x411][allocated]
  Top chunk @ 0x55a7ecbce660 - [brk_start: 0x55a7ecbce000, brk_end: 0x55a7ecbef000]
```

还可以像下面这样以图的形式查看heap的布局：

```
[0x7fae46236ca6]> dmhg
Heap Layout
    .────────────────────────────────────.
    │    Malloc chunk @ 0x55a7ecbce000   │
    │ size: 0x251                        │
    │  fd: 0x0, bk: 0x0                  │
    `────────────────────────────────────'
        │
    .───'
    │
    │
  .─────────────────────────────────────────────.
  │    Malloc chunk @ 0x55a7ecbce250            │
  │ size: 0x411                                 │
  │  fd: 0x57202c6f6c6c6548, bk: 0xa21646c726f  │
  `─────────────────────────────────────────────'
      │
  .───'
  │
  │
.────────────────────────────────────────────────────.
│  Top chunk @ 0x55a7ecbce660                        │
│ [brk_start:0x55a7ecbce000, brk_end:0x55a7ecbef000] │
`────────────────────────────────────────────────────'
```

其它heap命令都在`dmh`命令下，查看`dmh?`的信息获取完整的命令列表：

```
[0x00000000]> dmh?
|Usage:  dmh # Memory map heap
| dmh                 List chunks in heap segment
| dmh [malloc_state]  List heap chunks of a particular arena
| dmha                List all malloc_state instances in application
| dmhb                Display all parsed Double linked list of main_arena's bins instance
| dmhb [bin_num|bin_num:malloc_state]          Display parsed double linked list of bins instance from a particular arena
| dmhbg [bin_num]     Display double linked list graph of main_arena's bin [Under developemnt]
| dmhc @[chunk_addr]  Display malloc_chunk struct for a given malloc chunk
| dmhf                Display all parsed fastbins of main_arena's fastbinY instance
| dmhf [fastbin_num|fastbin_num:malloc_state]  Display parsed single linked list in fastbinY instance from a particular arena
| dmhg                Display heap graph of heap segment
| dmhg [malloc_state] Display heap graph of a particular arena
| dmhi @[malloc_state]Display heap_info structure/structures for a given arena
| dmhm                List all elements of struct malloc_state of main thread (main_arena)
| dmhm [malloc_state] List all malloc_state instance of a particular arena
| dmht                Display all parsed thead cache bins of main_arena's tcache instance
| dmh?                Show map heap help
```

若要打印带有demangled pointers的safe-linked链（glibc >= 2.32)，要将 `dbg.glibc.demangle` 设置为true。

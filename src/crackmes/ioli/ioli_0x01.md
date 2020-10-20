IOLI 0x01
=========

这是第二个IOLI crackme

```
$ ./crackme0x01
IOLI Crackme Level 0x01
Password: test
Invalid Password!
```

用rabin2检查字符串。
```
$ rabin2 -z ./crackme0x01
[Strings]
nth paddr      vaddr      len size section type  string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00000528 0x08048528 24  25   .rodata ascii IOLI Crackme Level 0x01\n
1   0x00000541 0x08048541 10  11   .rodata ascii Password: 
2   0x0000054f 0x0804854f 18  19   .rodata ascii Invalid Password!\n
3   0x00000562 0x08048562 15  16   .rodata ascii Password OK :)\n
```

这次不像0x00那么容易了，试试用r2进行反汇编。

```
$ r2 ./crackme0x01 
-- Use `zoom.byte=printable` in zoom mode ('z' in Visual mode) to find strings
[0x08048330]> aa
[0x08048330]> pdf@main
            ; DATA XREF from entry0 @ 0x8048347
┌ 113: int main (int argc, char **argv, char **envp);
│           ; var int32_t var_4h @ ebp-0x4
│           ; var int32_t var_sp_4h @ esp+0x4
│           0x080483e4      55             push ebp
│           0x080483e5      89e5           mov ebp, esp
│           0x080483e7      83ec18         sub esp, 0x18
│           0x080483ea      83e4f0         and esp, 0xfffffff0
│           0x080483ed      b800000000     mov eax, 0
│           0x080483f2      83c00f         add eax, 0xf                ; 15
│           0x080483f5      83c00f         add eax, 0xf                ; 15
│           0x080483f8      c1e804         shr eax, 4
│           0x080483fb      c1e004         shl eax, 4
│           0x080483fe      29c4           sub esp, eax
│           0x08048400      c70424288504.  mov dword [esp], str.IOLI_Crackme_Level_0x01 ; [0x8048528:4]=0x494c4f49 ; "IOLI Crackme Level 0x01\n"
│           0x08048407      e810ffffff     call sym.imp.printf         ; int printf(const char *format)
│           0x0804840c      c70424418504.  mov dword [esp], str.Password: ; [0x8048541:4]=0x73736150 ; "Password: "
│           0x08048413      e804ffffff     call sym.imp.printf         ; int printf(const char *format)
│           0x08048418      8d45fc         lea eax, [var_4h]
│           0x0804841b      89442404       mov dword [var_sp_4h], eax
│           0x0804841f      c704244c8504.  mov dword [esp], 0x804854c  ; [0x804854c:4]=0x49006425
│           0x08048426      e8e1feffff     call sym.imp.scanf          ; int scanf(const char *format)
│           0x0804842b      817dfc9a1400.  cmp dword [var_4h], 0x149a
│       ┌─< 0x08048432      740e           je 0x8048442
│       │   0x08048434      c704244f8504.  mov dword [esp], str.Invalid_Password ; [0x804854f:4]=0x61766e49 ; "Invalid Password!\n"
│       │   0x0804843b      e8dcfeffff     call sym.imp.printf         ; int printf(const char *format)
│      ┌──< 0x08048440      eb0c           jmp 0x804844e
│      │└─> 0x08048442      c70424628504.  mov dword [esp], str.Password_OK_: ; [0x8048562:4]=0x73736150 ; "Password OK :)\n"
│      │    0x08048449      e8cefeffff     call sym.imp.printf         ; int printf(const char *format)
│      │    ; CODE XREF from main @ 0x8048440
│      └──> 0x0804844e      b800000000     mov eax, 0
│           0x08048453      c9             leave
└           0x08048454      c3             ret
```

"aa"令r2分析整个二进制文件，从而获得符号名称。

"pdf" 代表

*	Print

*	Disassemble

*	Function

其将会打印main函数（或者说每个人都熟悉的`main()`）的反汇编结果，可以从中看到几个东西：奇怪的命名，箭头，等等。

*	"imp." 代表imports，即一些导入的符号，比如printf()

*	"str." 代表字符串， 即字符串（显而易见）

如果仔细地观察，可以看到一个`cmp`指令，有一个跟随着指令的常量0x149a。`cmp`是x86的一个比较指令，0x前缀则意味着数字以16作为基数，或者说是hex（十六进制数）。

```
0x0804842b    817dfc9a140. cmp dword [ebp + 0xfffffffc], 0x149a
```

可以使用radare2中的`？`命令将数字转换为其他基数显示。

```
[0x08048330]> ? 0x149a
int32   5274
uint32  5274
hex     0x149a
octal   012232
unit    5.2K
segment 0000:049a
string  "\x9a\x14"
fvalue: 5274.0
float:  0.000000f
double: 0.000000
binary  0b0001010010011010
trits   0t21020100
```

现在我们知道了0x149a就是十进制的5274，试试用其作为口令。

```
$ ./crackme0x01
IOLI Crackme Level 0x01
Password: 5274
Password OK :)
```

Bingo，口令就是5247！在这个例子里，位于0x804842b的口令函数将输入与十六进制的0x149a进行比较。由于用户输入的通常是十进制数，因此可以肯定地认为输入就是十进制数，即5247。作为黑客，好奇心驱使着我们去看看如果以16进制输入会发生什么。

```
$ ./crackme0x01
IOLI Crackme Level 0x01
Password: 0x149a
Invalid Password!
```

值得一试，不过它最终不起作用，这是因为`scanf()`将会把0x149a前的0作为数字0读入，而不是将其作为16进制数读入。

IOLI 0x01的总结到此结束。
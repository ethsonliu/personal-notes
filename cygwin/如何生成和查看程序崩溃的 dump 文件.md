在 `/etc/profile` 中加入 `export CYGWIN="error_start=E:/cygwin64/bin/dumper.exe"`，然后 `source /etc/profile`。

Makefile 编译的时候注意加入 `-g` 参数，生成的 exe 还要生成反汇编，命令 `objdump -D -S core_dump_demo.exe > core_dump_demo.rasm`，如果程序较大，这个过程就会很漫长，结束之后就可以运行了，
运行如果崩溃就会生成 `*.exe.stackdump`，

```
# 节选的反汇编
int f2() {
   1004010e0:   55                      push   %rbp
   1004010e1:   48 89 e5                mov    %rsp,%rbp
   1004010e4:   48 83 ec 30             sub    $0x30,%rsp
    printf("entering %s...\n", __func__);
   1004010e8:   48 8d 15 6b 1f 00 00    lea    0x1f6b(%rip),%rdx        # 10040305a <__func__.3391>
   1004010ef:   48 8d 0d 3a 1f 00 00    lea    0x1f3a(%rip),%rcx        # 100403030 <.rdata>
   1004010f6:   e8 f5 00 00 00          callq  1004011f0 <printf>
    char *buff = "0123456789";
   1004010fb:   48 8d 05 3e 1f 00 00    lea    0x1f3e(%rip),%rax        # 100403040 <.rdata+0x10>
   100401102:   48 89 45 f8             mov    %rax,-0x8(%rbp)
    free(buff);  // core dump location
   100401106:   48 8b 45 f8             mov    -0x8(%rbp),%rax
   10040110a:   48 89 c1                mov    %rax,%rcx
   10040110d:   e8 ce 00 00 00          callq  1004011e0 <free>
    printf("leaving %s...\n", __func__);
   100401112:   48 8d 15 41 1f 00 00    lea    0x1f41(%rip),%rdx        # 10040305a <__func__.3391>
   100401119:   48 8d 0d 2b 1f 00 00    lea    0x1f2b(%rip),%rcx        # 10040304b <.rdata+0x1b>
   100401120:   e8 cb 00 00 00          callq  1004011f0 <printf>
    return 0;
   100401125:   b8 00 00 00 00          mov    $0x0,%eax
}
   10040112a:   48 83 c4 30             add    $0x30,%rsp
   10040112e:   5d                      pop    %rbp
   10040112f:   c3                      retq   
```

```
# 运行命令
E:\share>core_dump_demo.exe
entering main...
entering f1...
entering f2...
      1 [main] core_dump_demo 5476 cygwin_exception::open_stackdumpfile: Dumping
 stack trace to core_dump_demo.exe.stackdump
```

```
# 生成的 stackdump
Stack trace:
Frame        Function    Args
000FFFFC400  0018005C48C (000FFFFE3F4, 0010000F54C, 0018006D05E, 000FFFFDE50)
000FFFFC4A0  0018005DA6B (00000000000, 00100000000, 0000000014C, 00000000000)
000FFFFC6F0  0018011F0D0 (00000000000, 00000000000, 00000000000, 000FFFFC9A4)
000FFFFC9E0  0018011BDAE (00000000000, 00000000000, 00000000000, 000FFFFC9F0)
000FFFFCB80  0018011C249 (00000000002, 00000000000, 000FFFFCC62, 00000000006)
000FFFFCB80  0018011C41A (000FFFFCA50, 00100403060, 00180143308, 001801430C0)
000FFFFCB80  0018011C6DF (00000000000, 000FFFFCB80, 00100403040, 00100403040)
000FFFFCB80  00180154325 (001801FB870, 00100403030, 000FFFFCB60, 00000000000)
000FFFFCB80  001800B9A63 (0010040305A, 000FFFFC8CC, 0018013D2D0, 00180219E83)
000FFFFCB80  00180117A4B (0010040305A, 000FFFFC8CC, 0018013D2D0, 00180219E83)
000FFFFCB80  00100401112 (0010040305D, 000FFFFC8FC, 0018013D2D0, 000FFFFCBE0)
000FFFFCBB0  00100401150 (00100403060, 00000000000, 00000000000, 000FFFFCCC0)
000FFFFCBE0  00100401193 (00000000020, FF06FF010000FF00, 00180047931, 000FFFFCDF0)
000FFFFCCC0  001800479A2 (00000000000, 00000000000, 00000000000, 00000000000)
00000000000  00180045733 (00000000000, 00000000000, 00000000000, 00000000000)
000FFFFFFF0  001800457E4 (00000000000, 00000000000, 00000000000, 00000000000)
End of stack trace (more stack frames may be present)
```

在 stackdump 中观察 Function 段的地址，然后在反汇编中找到即可。

参考：

- <https://blog.csdn.net/cbbbc/article/details/44593723>
- <https://www.jianshu.com/p/4fbb0b863c6e>

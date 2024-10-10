
```
*** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
Build fingerprint:
'Android/jileniao.net/jileniao:5.0.1/LRX22G/root03231947:userdebug/test-keys'
Revision: '33696'
ABI: 'arm'
pid: 2686, tid: 3065, name: gps_proc  >>> /system/bin/gpsserver <<<
signal 11 (SIGSEGV), code 2 (SEGV_ACCERR), fault addr 0xb170a219
W/NativeCrashListener(856): Couldn't find ProcessRecord for pid 2686
r0 000fde18  r1 ae3f4248  r2 0007ef0c  r3 00008000
AM write failure (32 / Broken pipe) I/DEBUG(349):     r4 001fb799  r5 ffffa7e4  r6 ae4d00c0  r7 0001c5a2
r8 00000081  r9 00000001  sl b160c400  fp 00000001
ip ae4d00c0  sp af0bb8a0  lr b160c401  pc b5d08680  cpsr 280b0010
backtrace:
#00 pc 0011d680  /system/lib/gps_ma87.so (Method0+984)
#01 pc 0007694c  /system/lib/gps_ma87.so (Method1+724)
#02 pc 00069c2c  /system/lib/gps_ma87.so (Method2+1116)
#03 pc 000676f4  /system/lib/gps_ma87.so (Method3+224)
#04 pc 00066bef  /system/lib/gps_ma87.so (GPSCPPClassName::Method4(char*, int)+390)
#05 pc 000438bf  /system/lib/gps_ma87.so (gps::Method5(void)+134)
#06 pc 00042fe1  /system/lib/gps_ma87.so (gps::Method6(void*)+40)
#07 pc 00042429  /system/lib/gps_ma87.so (gps::Method7(void*)+172)
#08 pc 000162e3  /system/lib/libc.so (__pthread_start(void*)+30)
#09 pc 000142d3  /system/lib/libc.so (__start_thread+6)
```

backtrace 中，从 #00 到 #09 共十行信息代表 是crash 时函数调用关系，从下往上倒着看，#09 行的方法调用了 #08 行的方法；#08 行的方法调用了 #07 行的方法；向上类推，最后就是 #01 行的方法调用了 #00 行的方法。而最终出现的 crash 就是在 #00 行中。

举个例子，以下 backtrace中，

```
# ./a.out
=========>>>catch signal 11 <<<=========
Dump stack start...
Obtained 9 stack frames.
./a.out(dump_stack+0x1f) [0x40098c]
./a.out(signal_handler+0x2e) [0x400a36]
/lib64/libc.so.6(+0x36400) [0x7fcc5b063400]
./libfunc.so(func_c+0x10) [0x7fcc5b3fb705]
./libfunc.so(func_b+0x9) [0x7fcc5b3fb716]
./libfunc.so(func_a+0x9) [0x7fcc5b3fb721]
./a.out(main+0x28) [0x400a83]
/lib64/libc.so.6(__libc_start_main+0xf5) [0x7fcc5b04f555]
./a.out() [0x4008a9]
Dump stack end...
段错误(吐核)
```

出错的位置：`./libfunc.so(func_c+0x10) [0x7fcc5b3fb705]`

```
# nm libfunc.so | grep func_c
00000000000006f5 T func_c
```

```
0x6f5 + 0x10 = 0x705
```

```
# addr2line -e libfunc.so 0x705
func.c:6
```

**addr2line 得到的行号确定不可能出错**

如果使用 addr2line 命令得到的行号对应的源代码中就是一句很简单的赋值语句这样的情况，也就是我们非常确定这行代码不可能出什么错误异常的。此时就要看调用该方法的地方是否有错，很有可能是调用该方法的地方就出错了，所以系统报错时候就定位到这一行了。

**addr2line 得到行号为 ??:? 或 ??:0 的原因**

如果遇到 addr2line 得到 ??:? 或 ??:0 的情况，原因就是编译得到的 so 文件没有附加上符号表(symbolic)信息。










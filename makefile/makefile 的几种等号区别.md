## =

make 会将整个 makefile 展开后，再决定变量的值。也就是说，变量的值将会是整个 makefile 中最后被指定的值。看例子：

```makefile
x = foo
y = $(x) bar
x = xyz
```

在上例中，y 的值将会是 xyz bar ，而不是 foo bar。

## :=

变量的值决定于它在 makefile 中的位置，而不是整个 makefile 展开后的最终值。

```makefile
x := foo
y := $(x) bar
x := xyz
```

在上例中，y 的值将会是 foo bar，而不是 xyz bar 了。
      
## ?=

表示如果该变量没有被赋值，则赋予等号后的值。

```makefile
VIR ?= new_value
```

上述 VIR 在之前没有被赋值，所以 VIR 的值就为 new_value。

```makefile
VIR := old_value
VIR ?= new_value
```

这种情况下，VIR 的值就是 old_value。

## +=

这个就和平时写代码的理解是一样的，表示将等号后面的值添加到前面的变量上

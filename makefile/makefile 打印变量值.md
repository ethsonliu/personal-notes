### 使用 info/warning/error 增加调试信息

```
# 但是此不能打印出 .mk 的行号
$(info, "here add the debug info")

$(warning "here add the debug info")

# 这个可以停止当前 makefile 的编译
$(error "error: this will stop the compile")

# 打印变量的值
$(info $(TARGET_DEVICE) )
```

### 使用 echo 增加调试信息（echo 只能在 target: 后面的语句中使用，且前面是个 TAB）

```
test:$(OBJS)
    @echo "start the compilexxxxxxxxxxxxxxxxxxxxxxx"
    @echo $(files)
```

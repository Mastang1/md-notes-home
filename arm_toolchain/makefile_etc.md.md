# STM32 裸机项目最常用 arm-none-eabi-gcc 编译链接选项全解析  
（适用于 STM32F407 / F103 / F4 / H7 等 Cortex-M 系列）

## 一、核心工具链命令（记住这几个就够一辈子）

| 变量名      | 命令                        | 作用                                      |
|-------------|-----------------------------|-------------------------------------------|
| `CC`        | `arm-none-eabi-gcc`         | C 和汇编编译器（.c/.s → .o）               |
| `AS`        | `arm-none-eabi-as`          | 纯汇编（一般不用，gcc 直接搞定）           |
| `LD`        | `arm-none-eabi-ld`          | 直接链接器（一般不用，交给 gcc）           |
| `OBJCOPY`   | `arm-none-eabi-objcopy`     | elf → hex / bin                           |
| `OBJDUMP`   | `arm-none-eabi-objdump`     | 生成反汇编 .lst 文件                      |
| `SIZE`      | `arm-none-eabi-size`        | 查看 flash 和 ram 占用                    |
| `GDB`       | `arm-none-eabi-gdb`         | 调试器                                    |

## 二、CFLAGS（编译选项）—— 这些你一辈子都逃不掉

| 选项                                    | 必须/强烈建议 | 说明（白话版）                                              |
|-----------------------------------------|---------------|------------------------------------------------------------|
| `-mcpu=cortex-m4`                       | 必须          | 指定 CPU 核心，不写直接报错                                 |
| `-mthumb`                               | 必须          | Cortex-M 全用 Thumb-2 指令集，不开体积翻倍                  |
| `-mfpu=fpv4-sp-d16 -mfloat-abi=hard`    | F4/F7/H7 必须 | F407 有硬浮点 FPU，必须加，不然浮点运算慢如狗               |
| `-Wall -Wextra`                         | 强烈建议      | 把傻逼错误全警告出来，不开迟早踩坑                          |
| `-ffunction-sections -fdata-sections`   | 神级必须      | 每个函数/变量单独占一个 section，配合 gc-sections 死未用代码 |
| `-O0`                                   | debug 用      | 不优化，调试信息最全                                        |
| `-Os`                                   | 强烈推荐      | 优化体积，嵌入式首选，比 -O2 更省 flash                     |
| `-O2 / -O3`                             | release 用    | 极致性能，但体积可能更大                                    |
| `-g -g3`                                | 必须          | 生成完整调试信息（-g3 包含宏定义）                          |
| `-MMD -MP`                              | 神级必须      | 自动生成 .d 依赖文件，改头文件自动重编                      |
| `-std=gnu99` 或 `-std=gnu11`            | 推荐          | 支持 C99/C11 + GNU 扩展                                     |
| `-fno-builtin`                          | 常用          | 禁用 gcc 自带的 memcpy/memset，防止和你手写的冲突           |

### STM32F407 最强 debug CFLAGS（直接抄）
```makefile
CFLAGS = -mcpu=cortex-m4 -mthumb -mfpu=fpv4-sp-d16 -mfloat-abi=hard \
         -Wall -Wextra -ffunction-sections -fdata-sections \
         -O0 -g3 -MMD -MP -std=gnu11

```

## makefile 规则笔记

![[Pasted image 20251217000504.png]]
### 伪目标

否则如果目录里恰好有个叫 `clean` 的文件，Make 会以为“已经做完了”，然后拒绝执行。
这也是老工程里最常见的隐性 bug 之一
```makefile
.PHONY: clean

clean:
	rm -f *.o app

```

### variable 变量（Makefile 工程化的起点）


```makefile
CC = gcc
CFLAGS = -Wall -g
OBJS = main.o add.o

app: $(OBJS)
	$(CC) $(OBJS) -o app

%.o: %.c add.h
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f *.o app

```

### makefile 常用函数列举
```makefile
#上手瞧一瞧
#核心1是获取1.o 2.o 3.o ...
#核心2是规则模式.c.o
CC = gcc
CFLAGS = -Wall -g
SRCS = main.c add.c util.c
OBJS = $(SRCS:.c=.o)

app: $(OBJS)
	$(CC) $(OBJS) -o app

%.o: %.c common.h
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) app

```


---
|**函数类型**|**原型**|**示例与说明**|
|--|---|---|
|**文本处理**|`$(subst from,to,text)`|`$(subst .c,.o,$(FILES))`<br><br>  <br><br>将 `text` 中所有 `from` 替换为 `to`。常用于批量修改文件名后缀。|
|**文本处理**|`$(patsubst pattern,replacement,text)`|`$(patsubst %.c,%.o,$(FILES))`<br><br>  <br><br>模式替换。将符合 `pattern` 的字符串（使用 `%` 通配）替换为 `replacement`。|
|**文本处理**|`$(strip string)`|`$(strip " hello world ")`<br><br>  <br><br>去除 `string` 头部和尾部多余的空格和制表符，并将内部连续空格替换为一个空格。|
|**文本处理**|`$(findstring find,text)`|`$(findstring "foo", "bar foo baz")`<br><br>  <br><br>如果 `find` 存在于 `text` 中，则返回 `find`，否则返回空。常用于条件判断。|
|**文本处理**|`$(filter pattern...,text)`|`$(filter %.c,a.c b.h c.c)`<br><br>  <br><br>过滤 `text` 中**符合** `pattern` 的单词列表并返回。|
|**文本处理**|`$(filter-out pattern...,text)`|`$(filter-out %.c,a.c b.h c.c)`<br><br>  <br><br>过滤 `text` 中**不符合** `pattern` 的单词列表并返回。|
|**文件名处理**|`$(dir names...)`|`$(dir src/main.c src/utils.c)`<br><br>  <br><br>提取文件名列表 `names` 中每个文件的**目录**部分（带斜杠 `/`）。|
|**文件名处理**|`$(notdir names...)`|`$(notdir src/main.c lib/data.a)`<br><br>  <br><br>提取文件名列表 `names` 中每个文件的**非目录**部分（文件名本身）。|
|**文件名处理**|`$(suffix names...)`|`$(suffix main.c utils.h data.a)`<br><br>  <br><br>提取文件名列表 `names` 中每个文件的**后缀**。|
|**文件名处理**|`$(basename names...)`|`$(basename main.c utils.h data.a)`<br><br>  <br><br>提取文件名列表 `names` 中每个文件的**主文件名**（去除后缀）。|
|**文件名处理**|`$(addsuffix suffix,names...)`|`$(addsuffix .o,main utils)`<br><br>  <br><br>为列表 `names` 中的每个单词**添加后缀** `suffix`。|
|**文件名处理**|`$(addprefix prefix,names...)`|`$(addprefix obj/,main.o utils.o)`<br><br>  <br><br>为列表 `names` 中的每个单词**添加前缀** `prefix`。|
|**流程控制**|`$(foreach var,list,text)`|`$(foreach f,$(FILES),$(f).o)`<br><br>  <br><br>遍历 `list`，每次将元素赋值给 `var`，然后展开 `text`，并将所有结果连接起来。|
|**流程控制**|`$(if condition,then-part[,else-part])`|`$(if $(VAR),$(VAR),default_value)`<br><br>  <br><br>如果 `condition` 不为空，则展开 `then-part`，否则展开可选的 `else-part`。|
|**流程控制**|`$(or condition1[,condition2...])`|`$(or $(VAR1),$(VAR2),$(VAR3))`<br><br>  <br><br>返回第一个非空参数的值，如果所有参数都为空，则返回空。|
|**流程控制**|`$(and condition1[,condition2...])`|`$(and $(VAR1),$(VAR2))`<br><br>  <br><br>只有当所有参数都非空时，才返回**最后一个**参数的值；否则返回空。|
|**变量引用**|`$(call variable,param,...)`|`$(call MY_FUNC,arg1,arg2)`<br><br>  <br><br>展开 `variable` 作为模板，并用 `param...` 替换模板中的 `$0` (`$1`, `$2`...)。常用于定义带参的宏或函数。|
|**系统调用**|`$(shell command)`|`FILES = $(shell find . -name "*.c")`<br><br>  <br><br>执行操作系统 `command` 并返回其输出结果，用空格分隔。|
|**查找文件**|`$(wildcard pattern)`|`$(wildcard src/*.c lib/*.c)`<br><br>  <br><br>查找符合 `pattern` 的**已存在**的文件名，并返回列表。|
|**调试输出**|`$(warning text)`|`$(warning "Warning: variable X is empty")`<br><br>  <br><br>在 Make 执行过程中打印 `text` 作为警告信息，但**不停止** Make 进程。|
|**调试输出**|`$(error text)`|`$(error "Error: SRC_DIR not set.")`<br><br>  <br><br>在 Make 执行过程中打印 `text` 作为错误信息，并**终止** Make 进程。|
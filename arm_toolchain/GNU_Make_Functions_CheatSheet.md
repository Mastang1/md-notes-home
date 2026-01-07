# GNU Make 常用内置函数速查表

## 一、字符串处理类（最常用）

### `$(subst from,to,text)`
```make
$(subst .c,.o,main.c util.c)   # 字符串替换，把 .c 替换成 .o
```

### `$(patsubst pattern,replacement,text)`
```make
$(patsubst %.c,%.o,main.c util.c)   # 模式替换，比 subst 强，支持 %
```

### `$(strip string)`
```make
$(strip  a  b   c )   # 去掉首尾空格，并压缩中间多余空格
```

### `$(findstring find,in)`
```make
$(findstring foo,foobar)   # 若找到子串则返回 find，否则返回空
```

### `$(filter pattern…,text)`
```make
$(filter %.c,$(SRCS))   # 过滤出匹配模式的单词
```

### `$(filter-out pattern…,text)`
```make
$(filter-out %.h,$(FILES))   # 过滤掉匹配模式的单词
```

### `$(sort list)`
```make
$(sort b a c a)   # 排序并去重：a b c
```

## 二、文件名 / 路径处理

### `$(dir names…)`
```make
$(dir src/main.c include/a.h)   # 提取目录部分
```

### `$(notdir names…)`
```make
$(notdir src/main.c)   # 提取文件名
```

### `$(basename names…)`
```make
$(basename main.c util.o)   # 去后缀
```

### `$(suffix names…)`
```make
$(suffix main.c util.o)   # 提取后缀
```

### `$(addprefix prefix,names…)`
```make
$(addprefix build/,$(OBJS))   # 加前缀
```

### `$(addsuffix suffix,names…)`
```make
$(addsuffix .o,main util)   # 加后缀
```

### `$(abspath names…)`
```make
$(abspath ../src/main.c)   # 绝对路径
```

### `$(realpath names…)`
```make
$(realpath ./link/file)   # 解析软链接
```

## 三、列表 / 单词操作

### `$(words text)`
```make
$(words a b c)   # 单词数量
```

### `$(word n,text)`
```make
$(word 2,a b c)   # 第 n 个单词
```

### `$(wordlist s,e,text)`
```make
$(wordlist 2,3,a b c d)   # 子列表
```

### `$(firstword text)`
```make
$(firstword a b c)   # 第一个单词
```

### `$(lastword text)`
```make
$(lastword a b c)   # 最后一个单词
```

## 四、条件与逻辑

### `$(if condition,then-part[,else-part])`
```make
$(if $(DEBUG),-g,-O2)
```

### `$(or a,b,c)`
```make
$(or $(A),$(B),default)
```

### `$(and a,b,c)`
```make
$(and $(A),$(B))
```

## 五、变量 / 展开控制

### `$(value var)`
```make
$(value CFLAGS)
```

### `$(eval text)`
```make
$(eval CFLAGS += -DDEBUG)
```

### `$(call func,arg1,arg2,…)`
```make
$(call myfunc,a,b)
```

### `$(origin var)`
```make
$(origin CC)
```

### `$(flavor var)`
```make
$(flavor CFLAGS)
```

## 六、Shell 与外部世界

### `$(shell command)`
```make
$(shell uname -r)
```

## 七、错误与调试

### `$(warning text)`
```make
$(warning building without DEBUG)
```

### `$(error text)`
```make
$(error compiler not found)
```

### `$(info text)`
```make
$(info OBJS=$(OBJS))
```

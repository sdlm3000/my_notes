# Makefile

[TOC]

## Makefile简介

###Makefile

​	gcc hello.c -o hello

​	gcc aa.c bb.c cc.c dd.c ...

​	make工具和Makefile

​	一个工程中的源文件不计其数，其按类型、功能、模块分别放在若干个目录中，makefile定义了一系列的规则来指定哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为 makefile就像一个Shell脚本一样，也可以执行操作系统的命令。



###make和Makefile的关系

​	make工具:找出修改过的文件，根据依赖关系，找出受影响的相关文件，最后按照规则单独编译这些文件。

​	Makefile文件: 记录依赖关系和编译规则。

​	Makefile的本质: 无论多么复杂的语法，都是为了更好地解决项目文件之间的依赖关系。



### Makefile三要素

​	目标、依赖、命令。

​	.PHONY: 可以指定伪目标

```makefile
# TODO: 测试一下直接用.c然后不加依赖
mp3:main.o mp3.o
	gcc main.o mp3.o -o mp3
main.o:
	gcc -c main.c -o main.o
mps.o:
	gcc -c mp3.c -o mp3.o

.PHONY:clean

clean:
	rm mp3
```

### Makefile的变量和模式匹配

#### 系统变量

​	$(CC): 指代编译器

​	$(AS): 指代汇编器

​	$(MAKE): 指代make工具

####自定义变量

​	=，延迟赋值，只有变量被引用时才会赋值

​	:=，立即赋值

​	?=，空赋值，只有当变量值为空时才能被赋值

​	+=，追加赋值

#### 自动化变量

​	$<: 第一个依赖文件

​	$^: 全部的依赖文件

​	$@: 目标

通过这些变量设置可以将上面那个Makefile文件修改为如下：

```makefile
CC=gcc
TRAGET=mp3
OBJS=main.o mp3.o

$(TARGET):$(OBJS)
	$(CC) $^ -o $@

main.o:main.c
	$(CC) -c main.c -o main.o

mp3.o:mp3.c
	$(CC) -c mp3.c -o mp3.o
	
.PHONY:clear
	
clean:
	rm mp3
```

#### 模式匹配

​	%: 匹配任意多个非空字符。注意它与*的区别，详见[%和\*的区别](https://blog.csdn.net/qq_37688116/article/details/68948289?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)。

通过模式匹配可以将上述修改为如下：

```makefile
CC=gcc
TRAGET=mp3
OBJS=main.o mp3.o

$(TARGET):$(OBJS)
	$(CC) $^ -o $@

%.o:%.c
	$(CC) -c $< -o $@
	
.PHONY:clear
	
clean:
	rm mp3
```

#### 默认规则

​	Makefile中所有的.o文件都是默认使用对应的.c进行编译的。则上述Makefile中的`$(CC) -c $< -o $@`可以注释掉。



### Makefile的条件分支

```makefile
ifeq (var1,var2)
...
else
...
endif
```

```makefile
ifneq (var1,var2)
...
else
...
endif
```

​	使用条件分支可以在上述代码中添加关于gcc编译器平台的选择相关的代码，如下：

```makefile
ARCH ?= x86

ifeq ($(ARCH),x86)
	CC=gcc
else
	CC=arm-linux-guneabihf-gcc
endif
```

在执行的时候可以指定编译的平台，如make ARCH=arm



### Makefile的常用函数

​	patsubst: 用于将文件模式进行替换，替换通配符。

​	notdir: 用于去掉文件的绝对路径，只保留文件名。

​	wildcard: 列出当前目录下所有符合模式格式的文件名。

​	foreach: 一个循环函数，语法为`$(foreach VAR,LIST,TEXT)`，执行时把“LIST”中使用空格分割的单词依次取出赋值给变量“VAR”，然后执行“TEXT”表达式。

```makefile
echo $(patsubat %.c,%.o,x.c.c bar.c)
# 会输出 x.c.o bar.o

echo $(notdir src/test.c)
# 会输出 test.c

echo $(wildcard *.c)
# 会输出当前目录下所有.c文件

dirs:=a b c d
files:=$(foreach dir,$(dirs),$(wildcard $(dir)/*)
# files等于a、b、c和d目录下的所有文件名
```

使用这些函数，将上述的Makefile文件格式规范化一些，将main相关的源文件放入module1文件夹，mp3相关的源文件放入module2文件夹，然后最终输出的文件放在build文件夹中。

```makefile
ARCH ?= x86

ifeq ($(ARCH),x86)
	CC=gcc
else
	CC=arm-linux-guneabihf-gcc
endif

TRAGET=mp3
BUILD_DIR=build
SRC_DIR=module1 module2

SOURCES=$(foreach dir,$(SRC_DIR),$(wildcard $(dir)/*.c))
OBJS=$(patsubst %.c,$(BUILD_DIR)/%.o,$(notdir $(SOURCES))
VPATH=$(SRC_DIR)

$(BUILD_DIR)/$(TRAGET):$(OBJS)
	$(CC) $^ -o $@

$(BUILD_DIR/%.o):%c | create_build
	$(CC) -c $< -o $@

.PHONY:clear create_build

clean:
	rm -r $(BUILD_DIR)

create_build:
	mkdir -p $(BUILD_DIR)
```



### 头文件依赖

```makefile
ARCH ?= x86

ifeq ($(ARCH),x86)
	CC=gcc
else
	CC=arm-linux-guneabihf-gcc
endif

TRAGET=mp3
BUILD_DIR=build
SRC_DIR=module1 module2
INC_DIR=include
CFLAGS=$(patsubst %,-I%,$(INC_DIR));
INCLUDES=$(foreach dir,$(INC_DIR),$(wildcard $(dir)/*.h))

SOURCES=$(foreach dir,$(SRC_DIR),$(wildcard $(dir)/*.c))
OBJS=$(patsubst %.c,$(BUILD_DIR)/%.o,$(notdir $(SOURCES))
VPATH=$(SRC_DIR)

$(BUILD_DIR)/$(TRAGET):$(OBJS)
	$(CC) $^ -o $@

$(BUILD_DIR/%.o):%.c $(INCLUDES)| create_build
	$(CC) -c $< -o $@ $(CFLAGS)

.PHONY:clear create_build

clean:
	rm -r $(BUILD_DIR)

create_build:
	mkdir -p $(BUILD_DIR)
```







 

​	

​	
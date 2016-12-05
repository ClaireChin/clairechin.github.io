---
layout:     post
title:      "Makefile书写学习part four"
subtitle:   "一起学习makefile"
date:       2016-12-05
author:     "ClaireChin"
header-img: "img/post-bg-yourname.jpg"
catalog: true
tags:
    - linux
    - makefile
    - function
---
>

# How to write a Makefile

------

> * make的内嵌函数（续）

## foreach函数

### 函数语法

     $(foreach VAR,LIST,TEXT)
     
### 函数功能

如果存在变量或者函数的引用，首先展开变量"VAR"和"LIST"的引用，而表达式"TEXT"中的变量引用不展开。
执行时把"LIST"中使用空格分割的单词依次取出赋值给变量"VAR"，然后执行"TEXT"表达式。重复直到"LIST"的最后一个单词（为空时结束）。
"TEXT"中的变量或者函数引用在执行时才被展开，因此如果在"TEXT"中存在对"VAR"的引用，那么"VAR"的值在每一次展开式将会到的不同的值。

### 返回值

空格分割的多次表达式“TEXT”的计算的结果。

### 举例

     dirs := a b c d
     files := $(foreach dir,$(dirs),$(wildcard $(dir)/*))
     
### 函数说明

函数中参数"VAR"是一个局部的临时变量，它只在"foreach"函数的上下文中有效，它的定义不会影响其它部分定义的同名"VAR"变量的值。
在函数的执行过程中它是一个“直接展开”式变量。

## if函数

### 函数语法

     $(if CONDITION,THEN-PART[,ELSE-PART])
     
### 函数功能

第一个参数"CONDITION"，在函数执行时忽略其前导和结尾空字符，如果包含对其他变量或者函数的引用则进行展开。
如果"CONDITION"的展开结果非空，则条件为真，就将第二个参数"THEN_PATR"作为函数的计算表达式；
"CONDITION"的展开结果为空，将第三个参数"ELSE-PART"作为函数的表达式，函数的返回结果为有效表达式的计算结果。

### 返回值

根据条件决定函数的返回值是第一个或者第二个参数表达式的计算结果。当不存在第三个参数"ELSE-PART"，并且"CONDITION"展开为空，函数返回空。

### 函数说明

函数的条件表达式"CONDITION"决定了函数的返回值只能是"THEN-PART"或者"ELSE-PART"两者之一。

### 举例

     SUBDIR += $(if $(SRC_DIR) $(SRC_DIR),/home/src)
     
## call函数

### 函数语法

     $(call VARIABLE,PARAM,PARAM,...)
     
### 函数功能

在执行时，将它的参数"PARAM"依次赋值给临时变量"$(1)"、"$(2)"（这些临时变量定义在"VARIABLE"的值中，参考下边的例子......
call函数对参数的数目没有限制，也可以没有参数值，没有参数值的"call"没有任何实际存在的意义。
执行时变量"VARIABLE"被展开为在函数上下文有效的临时变量，变量定义中的"$(1)"作为第一个参数，并将函数参数值中的第一个参数赋值给它；
变量中的"$(2)"一样被赋值为函数的第二个参数值；依此类推（变量"$(0)"代表变量"VARIABLE"本身）。之后对变量"VARIABLE"表达式的计算值。

### 返回值

参数值"PARAM"依次替换"$(1)"、"$(2)"......之后变量"VARIABLE"定义的表达式的计算值。

### 函数说明

1. 函数中"VARIABLE"是一个变量名，而不是变量引用。因此，通常"call"函数中的"VARIABLE"中不包含"$"（当然，除非此变量名是一个计算的变量名）；
2. 当变量"VARIABLE"是一个make内嵌的函数名时（如"if"、"foreach"、"strip"等），对"PARAM"参数的使用需要注意，因为不合适或者不正确的参数将会导致函数的返回值难以预料。
3. 函数中多个"PARAM"之间使用逗号分割。
4. 变量"VARIABLE"在定义时不能定义为直接展开式！只能定义为递归展开式。

### 举例

     reverse = $(2) $(1)
     foo = $(call reverse,a,b)
     
## value函数

### 函数语法

     $(value VARIABLE)
     
### 函数功能

不对变量"VARIBLE"进行任何展开操作，直接返回变量"VARIBALE"的值。这里"VARIABLE"是一个变量名，一般不包含"$"（除非计算的变量名）。

### 返回值

变量"VARIBALE"所定义文本值（如果变量定义为递归展开式，其中包含对其他变量或者函数的引用，那么函数不对这些引用进行展开。函数的返回值是包含有引用值）。

### 举例

     # sample Makefile
     FOO = $PATH
     all:
      @echo $(FOO)
      @echo $(value FOO)

## eval函数

### 函数功能

函数"eval"对它的参数进行展开，展开的结果作为Makefile的一部分，make可以对展开内容进行语法解析。展开的结果可以包含一个新变量、目标、隐含规则或者是明确规则等。

### 返回值

函数"eval"的返回值时空，也可以说没有返回值。

### 函数说明

"eval"函数执行时会对它的参数进行两次展开。第一次展开过程发是由函数本身完成的，第二次是函数展开后的结果被作为Makefile内容时由make解析时展开的。

### 举例

     # sample Makefile
     PROGRAMS = server client
     server_OBJS = server.o server_priv.o server_access.o
     server_LIBS = priv protocol
     client_OBJS = client.o client_api.o client_mem.o
     client_LIBS = protocol
     # Everything after this is generic
     .PHONY: all
     all: $(PROGRAMS)
     define PROGRAM_template
     $(1): $$($(1)_OBJ) $$($(1)_LIBS:%=-l%)
     ALL_OBJS += $$($(1)_OBJS)
     endef
     $(foreach prog,$(PROGRAMS),$(eval $(call PROGRAM_template,$(prog))))
     $(PROGRAMS):
        $(LINK.o) $^ $(LDLIBS) -o $@
     clean:
        rm -f $(ALL_OBJS) $(PROGRAMS)
        
"$(foreach prog,$(PROGRAM),$(eval $(call PROGRAM_template,$(prog))))"展开为：

     server : $(server_OBJS) –l$(server_LIBS)
     client : $(client_OBJS) –l$(client_LIBS)
     
## origin函数

### 函数语法

     $(origin VARIABLE)
     
### 函数功能

函数"origin"查询参数"VARIABLE"（变量名）的出处。

### 函数说明

"VARIABLE"是一个变量名而不是一个变量的引用。因此通常它不包含"$"（计算的变量名例外）。

### 返回值

返回值有以下几种情况：
（1）undefined<br>
变量"VARIABLE"没有被定义。

（2）default<br>
变量"VARIABLE"默认定义（内嵌变量）。如"CC"、"MAKE"、"RM"等变量。如果在Makefile中重新定义这些变量，函数返回值将相应发生变化。

（3）environment<br>
变量"VARIABLE"是系统环境变量，并且make没有使用命令行选项"-e"。

（4）environment override<br>
变量"VARIABLE"是一个系统环境变量，并且make使用了命令行选项"-e"。Makefile中存在一个同名的变量定义，使用"make -e"时环境变量值替代了文件中的变量定义。

（5）file<br>
变量"VARIABLE"在某一个makefile文件中定义。

（6）command line<br>
变量"VARIABLE"在命令行中定义。

（7）override<br>
变量"VARIABLE"在makefile文件中定义并使用"override"指示符声明。

（8）automatic<br>
变量"VARIABLE"是自动化变量。

## shell函数

### 函数功能

函数"shell"所实现的功能和shell中的引用（''）相同。实现对命令的扩展。这就意味着需要一个shell命令作为此函数的参数。

### 返回值

函数"shell"的参数（一个shell命令）在shell环境中的执行结果。

### 函数说明

函数本身的返回值是其参数的执行结果，没有进行任何处理。对结果的处理是由make进行的。当对函数的引用出现在规则的命令行中，命令行在执行时函数才被展开。
展开时函数参数（shell命令）的执行是在另外一个shell进程中完成的，因此需要对出现在规则命令行的多级“shell”函数引用需要谨慎处理，否则会影响效率。

## 举例

     contents := $(shell cat foo)
     files := $(shell echo *.c)

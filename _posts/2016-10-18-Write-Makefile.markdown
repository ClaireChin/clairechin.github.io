---
layout:     post
title:      "Makefile书写学习part one"
subtitle:   "一起学习makefile"
date:       2016-10-18
author:     "ClaireChin"
header-img: "img/post-bg-makefile1.jpg"
catalog: true
tags:
    - linux
    - makefile
    - build
---
>在linux环境下工作的程序员如果不会通过书写Makefile来创建和构造工程，应该不能算是一位合格的程序员。
<br>linux文件中"#"起头的字符串表示注释。

# How to write a Makefile

------

> * 基本概念（Basic concepts）

> * 定义变量

> * Makefile自动推导

> * 另类风格的Makefile

> * 参考资料


------

## 基本概念

### Makefile的规则
一个最简单的Makefile规则组成如下：

    TARGET... : PREREQUISITES...
        COMMAND
        ...
        ...
      
      
**TARGET**：规则的目标。通常是最后需要生成的文件名或者为了实现这个目的而必需的中间过程文件名。
可以是.o文件（目标文件）、也可以是最后的可执行程序的文件名等。另外，目标也可以是标签或make执行的动作的名称，如目标“clean”。
<br>
<br>**PREREQUISITES**：规则的依赖。生成规则的目标所需要的文件名列表或者目标，通常一个目标依赖于一个或多个文件。
<br>
<br>**COMMAND**：规则的命令行。是规则所要执行的动作（任意的shell命令或者是可在shell下执行的程序）。
一个规则可以有多个命令行，每一条命令占一行。
<br>
**注意：每一个命令行必须以[Tab]字符开始，[Tab]字符告诉make此行是一个命令行。make按照命令完成相应的动作。
这也是书写Makefile中容易产生，而且比较隐蔽的错误。**
<br>举个简单的例子，

    try : foo.o
      cc -o try foo.o

    foo.o : foo.c foo.h
      cc -c foo.c	
    clean :
      rm try foo.o  
      
执行make try即可进行编译。  
其中，foo.h的内容是，

    #define tddMode 1
    
foo.h的内容是，

    #include <stdio.h>
    #include "foo.h"

    int main()
    {
	    if (1 == tddMode)
	    {
		    printf("Duplex mode: TDD!\n");
	    }
	    return 0;
    }

针对较长行，可以使用反斜线（\）来分解为多行，这样可以使Makefile书写清晰、容易阅读理解。**但需要注意：反斜线之后不能有空格！**
<br>《GNU make中文手册》所举的例子如下所示：

    #sample Makefile
    edit : main.o kbd.o command.o display.o \
	 insert.o search.o files.o utils.o
    	cc -o edit main.o kbd.o command.o display.o \
    		insert.o search.o files.o utils.o
    main.o : main.c defs.h
    	cc -c main.c
    kbd.o : kbd.c defs.h command.h
    	cc -c kbd.c
    command.o : command.c defs.h command.h
    	cc -c command.c
    display.o : display.c defs.h buffer.h
    	cc -c display.c
    insert.o : insert.c defs.h buffer.h
    	cc -c insert.c
    search.o : search.c defs.h buffer.h
    	cc -c search.c
    files.o : files.c defs.h buffer.h command.h
    	cc -c files.c
    utils.o : utils.c defs.h
    	cc -c utils.c
    clean :
    	rm edit main.o kbd.o command.o display.o \
    		insert.o search.o files.o utils.o

### Makefile的工作方式

默认的情况下，make执行的是Makefile中的第一个规则，此规则的第一个目标称之为“最终目的”或者“终极目标”。
<br>当在shell提示符下输入“make”命令以后，<br>
1. make读取当前目录下的Makefile文件；<br>
2. 将Makefile中的第一个目标作为终极目标并处理该目标的所有依赖文件；<br>
3. 如果目标文件“try”不存在，则执行规则以创建目标"try"；<br>
4. 目标文件“try”存在，其依赖文件中有一个或者多个文件比它“更新”，则根据规则重新链接生成“try”；<br>
5. 目标文件“try”存在，它比它的任何一个依赖文件都“更新”，那么什么也不做。

目标文件".o"的处理也是遵循3，4，5的处理方式。

## 定义变量

针对《GNU make中文手册》中所举的例子可以发现，在目标edit所在的规则中，.o文件列表出现了两次，在这种情况下可以定义一个代替.o文件列表的变量，例如
“objects”、“OBJECTS”、“objs”、“OBJS”、“obj”或者“OBJ等。因此在上面的Makefile中可以添加这么一项，

    objects = main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o

Makefile中后续用到.o文件列表时可以使用$(objects)来表示。
<br>当需要增减.o文件时直接修改变量objects的定义即可。

## Makefile自动推导

Makefile中给出[.o]文件名时，make会自动为该文件推导合适的依赖文件（对应的.o文件）及依赖关系后面的命令，因此只需要给出那些特定的规则描述（.o目标需要的.h文件）。上述Makefile可以修改成如下所示：

	# sample Makefile
	objects = main.o kbd.o command.o display.o \
	       insert.o search.o files.o utils.o
	edit : $(objects)
		cc -o edit $(objects)
	main.o : defs.h
	kbd.o : defs.h command.h
	command.o : defs.h command.h
	display.o : defs.h buffer.h
	insert.o : defs.h buffer.h
	search.o : defs.h buffer.h
	files.o : defs.h buffer.h command.h
	utils.o : defs.h
	.PHONY : clean
	clean :
		rm edit $(objects)
	
上述即是Makefile的隐晦规则，其中".PHONY"表示"clean"是个伪目标，关于伪目标，后续的博客中再进行介绍。

## 另类风格的Makefile

上述Makefile的写法是根据**依赖**对规则进行分组的，也可以将Makefile根据**目标**对规则进行分组。

	#sample Makefile
	objects = main.o kbd.o command.o display.o \
	       insert.o search.o files.o utils.o
	edit : $(objects)
		cc -o edit $(objects)
	$(objects) : defs.h
	kbd.o command.o files.o : command.h
	display.o insert.o search.o files.o : buffer.h
	.PHONY : clean
	clean :
		rm edit $(objects)
	
事实上，这种风格的Makefile并不被推荐，将不同目标的依赖放在同一个规则中描述会使依赖关系变得混乱，并且可读性较差。
<br>**此处需要强调一下书写规则的建议方式：“单目标，多依赖。就是说尽量要做到一个规则中只存在一个目标文件，可有多个依赖文件。尽量避免使用多目标，单依赖的方式。”**

## 参考资料

* 《GNU make中文手册》
*  <http://wiki.ubuntu.org.cn/%E8%B7%9F%E6%88%91%E4%B8%80%E8%B5%B7%E5%86%99Makefile:MakeFile%E4%BB%8B%E7%BB%8D>

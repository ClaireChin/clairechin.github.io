---
layout:     post
title:      "Makefile书写"
subtitle:   "一起学习makefile"
date:       2016-10-18
author:     "ClaireChin"
header-img: "img/post-bg-gnu-make.jpg"
catalog: true
tags:
    - linux
    - makefile
    - build
---
>在linux环境下工作的程序员如果不会书写Makefile，就不是合格的linux程序员。

# How to write a Makefile

------

文中以"//"开始的内容表示对相应命令的注释。本文主要由基本概念和主要应用场景两部分组成。

> * 基本概念（Basic concepts）

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

### Makefile的工作方式

默认的情况下，make执行的是Makefile中的第一个规则，此规则的第一个目标称之为“最终目的”或者“终极目标”。
<br>当在shell提示符下输入“make”命令以后，<br>
1. make读取当前目录下的Makefile文件；<br>
2. 将Makefile中的第一个目标作为终极目标并处理该目标的所有依赖文件；<br>
3. 如果目标文件“try”不存在，则执行规则以创建目标"try"；<br>
4. 目标文件“try”存在，其依赖文件中有一个或者多个文件比它“更新”，则根据规则重新链接生成“try”；<br>
5. 目标文件“try”存在，它比它的任何一个依赖文件都“更新”，那么什么也不做。

目标文件".o"的处理也是遵循3，4，5的处理方式。

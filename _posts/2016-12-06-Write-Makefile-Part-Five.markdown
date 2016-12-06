---
layout:     post
title:      "Makefile书写学习part five"
subtitle:   "一起学习makefile"
date:       2016-12-06
author:     "ClaireChin"
header-img: "img/post-bg-yourname5.jpg"
catalog: true
tags:
    - linux
    - makefile
    - variable
---
>

# How to write a Makefile

------

> * make的内嵌变量

## MAKELEVEL变量

### 变量含义

在多级递归调用的make执行过程中。变量"MAKELEVEL"代表了调用的深度。在make一级级的执行过程中变量"MAKELEVEL"的值不断的发生变化，
通过它的值我们可以了解当前make递归调用的深度。最上一级时"MAKELEVEL"的值为"0"、下一级时为"1"、再下一级为"2"......

###用途

这个变量主要用在条件测试指令中。例如：可以通过测试此变量的值来决定是否执行递归方式的make调用或者其他方式的make调用。
希望一个子目录必须被上层make调用才可以执行此目录下的Makefile，而不允许直接在此目录下执行make。

### 举例

在Main目录下的Makefile内容如下，

     #maindir Makefile
     ………
     ………
     .PHONY :test
     test:
      @echo “main makelevel = $(MAKELEVEL)”
      @$(MAKE) –C subdir dislevel
     #subdir Makefile
     ………
     ………
     .PHONY : test
     test :
      @echo “subdir makelevel = $(MAKELEVEL)”
      
如果在Main目录下执行"make test"，将得到显示打印的信息：

     main makelevel = 0
     make[1]: Entering directory `/…../ subdir '
     subdir makelevel = 1
     make[1]: Leaving directory `/…../ subdir '
     
## MAKEFLAGS变量

### 变量含义

在make的递归执行过程中。最上层（可以称之为主控）make的命令行选项"-k"、"-s"等会被自动的通过环境变量"MAKEFLAGS"传递给子make进程。
传递过程中变量"MAKEFLAGS"的值会被主控make自动的设置为包含执行make时的命令行选项的字符串。因此可以借助make的环境变量"MAKEFLAGS" 
传递我们在主控make所使用的命令行选项给子make进程。

### 例外

需要注意的是"-C"、"-f"、"-o"和"-W"这几个特殊的命令行选项例外，它们是不会被赋值给变量"MAKEFLAGS"的。

### 变量说明

执行多级的make调用时，当不希望传递"MAKEFLAGS"的给子make时，需要在调用子make是对这个变量进行赋空。

     subsystem:
      cd subdir && $(MAKE) MAKEFLAGS=
      
Make命令行选项中一个比较特殊的是"-j"选项。在支持这个选项的操作系统上，如果给它指定了一个数值"N"（多任务的系统unix、Linux支持，MS-DOS不支持），
那么主控make和子make进程会在执行过程中使用通信机制来限制系统在同一时刻
（包括所有的递归调用的make进程，否则，将会导致make任务的数目无法控制而使别的任务无法到的执行）所执行任务的数目不大于"N"。
另外，当使用的操作系统不能支持make执行过程中的父子间通信，
那么无论在执行主控make时指定的任务数目"N"是多少，变量"MAKEFLAGS"中选项"-j"的数目会都被设置为"1"，这样确保系统的正常运转。

## MAKEFILES环境变量

### 变量说明

如果在当前环境定义了一个"MAKEFILES"环境变量，make执行时*首先*将此变量的值作为需要读入的Makefile文件，多个文件之间使用空格分开。
类似使用指示符"include"包含其它Makefile文件一样，如果文件名非绝对路径而且当前目录也不存在此文件，make会在一些默认的目录去寻找。

### 与"include"的区别

（1）环境变量指定的makefile文件中的“目标”不会被作为make执行的“终极目标”。<br>
（2）环境变量所定义的文件列表，在执行make时，如果不能找到其中某一个文件（不存在或者无法创建），make不会提示错误，也不退出。<br>
（3）make在执行时，首先读取的是环境变量"MAKEFILES"所指定的文件列表，之后才是工作目录下的makefile文件;
"include"所指定的文件是在make发现此关键字的时、暂停正在读取的文件而转去读取"include"所指定的文件。

### 用途

变量"MAKEFILES"主要用在“make”的递归调用过程中的的通信。实际应用中很少设置此变量，
因为一旦设置了此变量，在多级make调用时；由于每一级make都会读取"MAKEFILES"变量所指定的文件，将导致执行出现混乱。<br>
不过可以使用此环境变量来指定一个定义了通用“隐含规则”和变量的文件，比如设置默认搜索路径；
通过这种方式设置的“隐含规则”和定义的变量可以被任何make进程使用（有点像C语言中的全局变量）。

## MAKEFILE_LIST变量

### 变量含义

make程序在读取多个makefile文件时，包括由环境变量"MAKEFILES"指定、命令行指令、当前工作下的默认的以及使用指示符"include"指定包含的，
在对这些文件进行解析执行之前make读取的文件名将会被自动依次追加到变量"MAKEFILE_LIST"的定义域中。

### 用途

可以通过测试此变量的最后一个字来获取当前make程序正在处理的makefile文件名。
具体地说就是在一个makefile文件中如果使用指示符"include"包含另外一个文件之后，
变量"MAKEFILE_LIST"的最后一个字只可能是指示符"include"指定所要包含的那个文件的名字。

### 举例

某个Makefile的内容如下：

     name1 := $(word $(words $(MAKEFILE_LIST)),$(MAKEFILE_LIST))
     include inc.mk
     name2 := $(word $(words $(MAKEFILE_LIST)),$(MAKEFILE_LIST))
     all:
      @echo name1 = $(name1)
      @echo name2 = $(name2)
      
执行make后得到的结果是

     name1 = Makefile
     name2 = inc.mk
     
其中，"word"和"words"这两个内嵌函数的定义可以在《Makefile书写学习part three》中找到。

未完待续。。。。。。

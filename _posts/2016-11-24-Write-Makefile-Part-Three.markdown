---
layout:     post
title:      "Makefile书写学习part three"
subtitle:   "一起学习makefile"
date:       2016-11-24
author:     "ClaireChin"
header-img: "img/post-bg-yourname3.jpg"
catalog: true
tags:
    - linux
    - makefile
    - function
---
>

# How to write a Makefile

------

> * make的内嵌函数

## 函数的调用方式

以"$"开始表示一个引用，

     $(FUNCTION ARGUMENTS)
    
或者

     ${FUNCTION ARGUMENTS}
     
其中，"FUNCTION"是需要调用的函数名，并且它是make内嵌的函数名；"ARGUMENTS"表示函数的参数，推荐函数和参数之间使用一个空格分开，如果使用多个参数，则每个
参数之间使用逗号隔开。

## 文本处理函数

### $(subst FROM,TO,TEXT)

函数名称 | 字符串替换函数——subst
---- | ---
函数功能 | 把"TEXT"中的"FROM"字符替换成"TO"
返回值 |  替换后的新字符串
举例 | $(subst ee,EE,feet on the street)
举例结果 | fEET on the strEEt

### $(patsubst PATTERN,REPLACEMENT,TEXT)

函数名称 | 模式替换函数——patsubst
---- | ---
函数功能 | 搜索“TEXT”中以空格分开的单词，将符合模式“TATTERN”的替换为“REPLACEMENT”
返回值 |  替换后的新字符串
函数说明 | 参数“TEXT”单词之间的多个空格在处理时被合并为一个空格，并忽略前导和结尾空格
举例 | $(patsubst %.c,%.o,x.c.c bar.c)
举例结果 | x.c.o bar.o
有点类似于变量的引用替换：

     $(VAR:PATTERN=REPLACEMENT)
     
### $(strip STRINT)

函数名称 | 去空格函数——strint
---- | ---
函数功能 | 去掉字串（若干单词，使用若干空字符分割）“STRINT”开头和结尾的空字符，并将其中多个连续空字符合并为一个空字符
返回值 |  无前导和结尾空字符、使用单一空格分割的多单词字符串
函数说明 | 空字符包括空格、[Tab]等不可显示字符
举例 | STR = a b c
      | LOSTR = $(strip $(STR))
举例结果 | a b c

### $(findstring FIND,IN)

函数名称 | 查找字符串函数—findstring
---- | ---
函数功能 | 搜索字符串"IN"，查找字符串"FIND"
返回值 |  如果在"IN"之中存在"FIND"，则返回"FIND"，否则返回空
函数说明 | 字串"IN"之中可以包含空格、[Tab]。搜索需要是严格的文本匹配
举例1 | $(findstring a,a b c)
举例结果 | a
举例2 | $(findstring a,b c)
举例结果 | 空字符

### $(filter PATTERN…,TEXT)

函数名称 | 过滤函数—filter
---- | ---
函数功能 | 过滤掉字串"TEXT"中所有不符合模式"PATTERN"的单词，保留所有符合此模式的单词。
        | 可以使用多个模式。模式中一般需要包含模式字符"%"。存在多个模式时，模式表达式之间使用空格分割
返回值 |  空格分割的"TEXT"字串中所有符合模式"PATTERN"的字符串
函数说明 | 可以用来去除一个变量中的某些字符串
举例 | sources := foo.c bar.c baz.s ugh.h
        | foo: $(sources)
        | cc $(filter %.c %.s,$(sources)) -o foo
举例结果 | foo.c bar.c baz.s

### $(filter-out PATTERN...,TEXT)

函数名称 | 反过滤函数—filter-out
---- | ---
函数功能 | 和“filter”函数实现的功能相反。过滤掉字串“TEXT”中所有符合模式“PATTERN”的单词，保留所有不符合此模式的单词。
        |  可以有多个模式。存在多个模式时，模式表达式之间使用空格分割
返回值 |  空格分割的"TEXT"字串中所有不符合模式"PATTERN"的字串
函数说明 | 也可以用来去除一个变量中的某些字符串
举例 | objects=main1.o foo.o main2.o bar.o
     | mains=main1.o main2.o
举例结果 | foo.o bar.o

### $(sort LIST)

函数名称 | 排序函数—sort
---- | ---
函数功能 | 给字串"LIST"中的单词以首字母为准进行排序（升序），并取掉重复的单词
返回值 |  空格分割的没有重复单词的字符串
函数说明 | 两个功能，排序和去字串中的重复单词。可以单独使用其中一个功能
举例 | $(sort foo bar lose foo)
举例结果 | bar foo lose

### $(word N,TEXT)

函数名称 | 取单词函数—word
---- | ---
函数功能 | 取字符串"TEXT"中第"N"个单词（“N”的值从1开始）。
返回值 |  返回字串"TEXT"中第"N"个单词
函数说明 | 如果"N"值大于字串"TEXT"中单词的数目，返回空字符串。如果"N"为0，出错！
举例 | $(word 2, foo bar baz)
举例结果 | bar

### $(wordlist S,E,TEXT)

函数名称 | 取字符串函数—wordlist
---- | ---
函数功能 | 从字符串"TEXT"中取出从"S"开始到"E"的单词串。"S"和"E"表示单词在字串中位置的数字
返回值 |  字串"TEXT"中从第"S"到"E"（包括"E"）的单词字串
函数说明 | "S"和"E"都是从1开始的数字。当"S"比"TEXT"中的字数大时，返回空。
        | 如果"E"大于"TEXT"字数，返回从"S"开始，到"TEXT"结束的单词串。
        | 如果“S”大于"E"，返回空。
举例 | $(wordlist 2, 3, foo bar baz)
举例结果 | bar baz

### $(words TEXT)

函数名称 | 统计单词数目函数—words
---- | ---
函数功能 | 计算字符串"TEXT"中单词的数目
返回值 |  "TEXT"字串中的单词数
举例 | $(words, foo bar)
举例结果 | 2

### $(firstword NAMES…)

函数名称 | 取首单词函数—firstword
---- | ---
函数功能 | 取字串"NAMES…"中的第一个单词
返回值 | 字符串"NAMES…"的第一个单词
函数说明 | "NAMES"被认为是使用空格分割的多个单词（名字）的序列。函数忽略"NAMES…"中除第一个单词以外的所有的单词
举例 | $(firstword foo bar)
举例结果 | foo

## 文件名处理函数

### $(dir NAMES…)

函数名称 | 取目录函数—dir
---- | ---
函数功能 | 从文件名序列"NAMES…"中取出各个文件名的目录部分。
        | 文件名的目录部分就是包含在文件名中的最后一个斜线（"/"）（包括斜线）之前的部分
返回值 | 空格分割的文件名序列"NAMES…"中每一个文件的目录部分
函数说明 | 如果文件名中没有斜线，认为此文件为当前目录（"./"）下的文件
举例 | $(dir src/foo.c hacks)
举例结果 | src/ ./

### $(notdir NAMES…)

函数名称 | 取文件名函数——notdir
---- | ---
函数功能 | 从文件名序列"NAMES…"中取出非目录部分。目录部分是指最后一个斜线（"/"）（包括斜线）之前的部分。
        | 删除所有文件名中的目录部分，只保留非目录部分
返回值 | 文件名序列"NAMES…"中每一个文件的非目录部分
函数说明 | 如果“NAMES…”中存在不包含斜线的文件名，则不改变这个文件名。
        | 以反斜线结尾的文件名，是用空串代替
        | 当"NAMES…"中存在多个这样的文件名时，返回结果中分割各个文件名的空格数目将不确定
举例 | $(notdir src/foo.c hacks)
举例结果 | foo.c hacks

### $(suffix NAMES…)

函数名称 | 取后缀函数—suffix
---- | ---
函数功能 | 从文件名序列"NAMES…"中取出各个文件名的后缀。
        | 后缀是文件名中最后一个以点“.”开始的（包含点号）部分，如果文件名中不包含一个点号，则为空
返回值 | 以空格分割的文件名序列"NAMES…"中每一个文件的后缀序列
函数说明 | "NAMES…"是多个文件名时，返回值是多个以空格分割的单词序列。如果文件名没有后缀部分，则返回空。
举例 | $(suffix src/foo.c src-1.0/bar.c hacks)
举例结果 | .c .c

### $(basename NAMES…)

函数名称 | 取前缀函数—basename
---- | ---
函数功能 | 从文件名序列"NAMES…"中取出各个文件名的前缀部分（点号之后的部分）。
        | 前缀部分指的是文件名中最后一个点号之前的部分。
返回值 | 以空格分割的文件名序列"NAMES…"中每一个文件的后缀序列
函数说明 | 如果"NAMES…"中包含没有后缀的文件名，此文件名不改变。
        | 如果一个文件名中存在多个点号，则返回值为此文件名的最后一个点号之前的文件名部分
举例 | $(basename src/foo.c src-1.0/bar.c /home/jack/.font.cache-1 hacks)
举例结果 | src/foo src-1.0/bar /home/jack/.font hacks

### $(addsuffix SUFFIX,NAMES…)

函数名称 | 加后缀函数—addsuffix
---- | ---
函数功能 | 为"NAMES…"中的每一个文件名添加后缀"SUFFIX"。
        | 参数"NAMES…"为空格分割的文件名序列，将"SUFFIX"追加到此序列的每一个文件名的末尾
返回值 | 以单空格分割的添加了后缀“SUFFIX”的文件名序列
函数说明 | 如果"NAMES…"中包含没有后缀的文件名，此文件名不改变。
        | 如果一个文件名中存在多个点号，则返回值为此文件名的最后一个点号之前的文件名部分
举例 | $(addsuffix .c,foo bar)
举例结果 | foo.c bar.c

### $(addprefix PREFIX,NAMES…)

函数名称 | 加前缀函数—addprefix
---- | ---
函数功能 | 为"NAMES…"中的每一个文件名添加前缀"PREFIX"。
        | 参数"NAMES…"是空格分割的文件名序列，将"PREFIX"添加到此序列的每一个文件名之前。
返回值 | 以单空格分割的添加了前缀"PREFIX"的文件名序列
举例 | $(addprefix src/,foo bar)
举例结果 | src/foo src/bar

### $(join LIST1,LIST2)

函数名称 | 单词连接函数——join
---- | ---
函数功能 | 将字串“LIST1”和字串“LIST2”各单词进行对应连接。就是将“LIST2”中的第一个单词追加“LIST1”第一个单词字后合并为一个单词；
        | 将“LIST2”中的第二个单词追加到“LIST1”的第一个单词之后并合并为一个单词，……依次列推。
返回值 | 单空格分割的合并后的字（文件名）序列
函数说明 | 如果"LIST1"和"LIST2"中的字数目不一致时，两者中多余部分将被作为返回序列的一部分
举例1 | $(join a b , .c .o)
举例结果 | a.c b.o
举例2 | $(join a b c , .c .o)
举例结果 | a.c b.o c

### $(wildcard PATTERN)

函数名称 | 获取匹配模式文件名函数—wildcard
---- | ---
函数功能 | 列出当前目录下所有符合模式"PATTERN"格式的文件名
返回值 | 空格分割的、存在当前目录下的所有符合模式"PATTERN"的文件名
函数说明 | "PATTERN"使用shell可识别的通配符，包括"?"（单字符）、"*"（多字符）
举例 | $(wildcard *.c)
举例结果 | 当前目录下所有.c源文件列表

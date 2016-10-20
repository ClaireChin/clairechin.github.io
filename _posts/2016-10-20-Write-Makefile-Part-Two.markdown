---
layout:     post
title:      "Makefile书写学习part two"
subtitle:   "一起学习makefile"
date:       2016-10-20
author:     "ClaireChin"
header-img: "img/post-bg-gnu-make.jpg"
catalog: true
tags:
    - linux
    - makefile
    - object
---
>

# How to write a Makefile

------

> * About Makefile

## Makefile里有什么

Makefile里包含了这些内容，

* 显示规则
<br>如何更新一个或者多个被称为目标的文件（Makefile的目标文件）。
书写Makefile时需要明确地给出目标文件、目标的依赖文件列表以及更新目标文件所需要的命令。
* 隐式规则
<br>在“Makefile书写学习part one”中已经提到过。此处不再赘述。
* 变量定义
<br>使用一个字符或字符串代表一段文本串，当定义了一个变量以后，Makefile后续在需要使用此文本串的地方，通过引用这个变量来实现对文本串的使用。
* 指示符
<br>指示符指明在make程序读取Makefile文件过程中所要执行的一个动作。这包括：
<br>1. 读取一个文件，读取给定文件名的文件，将其内容作为makefile文件的一部分。
<br>2. 决定（通常是根据一个变量的值）处理或者忽略Makefile中的某一特定部分。
<br>3. 定义一个多行变量。
* 注释
<br>关于注释，在上一篇博客里有提到，Makefile中"#"之后的字符被作为为注释，注释行的结尾如果存在反斜线（\），那么下一行也被当作注释行。
推荐在书写Makefile时将注释作为成独立的行，不要与有效行写在一行之内。
当在Makefile中需要使用字符“#”时，可以使用反斜线加“#”（\#）来实现（对特殊字符“#”的转义）。

## Makefile命名

make会在文件目录中依次寻找以"GNUmakefile"、"Makefile"和"makefile"为文件名的文件，"Makefile"因大写字母比较显著而被推荐使用，"GNUmakefile"因为只有
"GNU make"才能识别而不被推荐。当make在工作目录中无法找到以上三个文件中的任何一个时，它将不读取其他任何文件作为解析对象。

当makefile的命名不是这三种文件中的任意一个时，可以通过"make -f NAME"或"make --file=NAME"命令来指定需要make的文件名"NAME"。当通过这种方式读取
makefile文件时，make就不再自动查找这三个标准命名的makefile文件。

## 包含其他的Makefile

Makefile中使用关键字include可以将其他的makefile包含进来，语法如下：

     include FILENAMES...
    
其中，FILENAME是shell可支持的文件名（包括通配符）。
<br>指示符“include”所在的行可以一个或者多个空格（make程序在处理时将忽略这些空格）开始，
但切忌不能以[Tab]字符开始，因为任意以[Tab]字符开始的行将被make程序作为一个命令行来处理。
指示符“include”和文件名之间、多个文件之间使用空格或者[Tab]键隔开。行尾的空白字符在处理时被忽略。
并且使用指示符包含进来的Makefile中，如果存在变量或者函数的引用。它们将会在包含它们的Makefile中被展开。
举个例子，

     include foo *.mk $(bar)
     
将被展开为

     include foo a.mk b.mk c.mk bish bash

make程序在处理指示符include时，将暂停对当前使用指示符“include”的makefile文件的读取，而转去依此读取由“include”指示符指定的文件列表。
直到完成所有这些文件以后再回过头继续读取指示符“include”所在的makefile文件。

指示符"include"适用的场合：
<br>1. 有多个不同的程序，由不同目录下的几个独立的Makefile来描述其重建规则。它们需要使用一组通用的变量定义或者模式规则。
通用的做法是将这些共同使用的变量或者模式规则定义在一个文件中（没有具体的文件命名限制），在需要使用的Makefile中使用指示符“include”来包含此文件。
<br>2. 当根据源文件自动产生依赖文件时，可将自动产生的依赖关系保存在另外一个文件中，主Makefile使用指示符“include”包含这些文件。
这样的做法比直接在主Makefile中追加依赖文件的方法要明智得多。
以指示符"include"指定的文件的查找方式...
<br>通常在Makefile中可使用“-include”来代替“include”来忽略由于包含文件不存在或者无法创建时的错误提示。
“-”的意思是告诉make，忽略此操作的错误，make继续执行。

     -include FILENAMES...
     
这种方式下当所要包含的文件不存在时不会有错误提示、make也不会退出，除此之外，效果与第一种方式相同。

## Makefiles环境变量

如果为当前环境定义了环境变量"Makefiles"，make在执行时会首先根据该环境变量的值来获取需要读入的Makefile文件，执行一个类似"include"的动作。
但需要注意，它与"include"是有区别的：环境变量指定的makefile文件中的目标将不会作为make的终极目标；在执行make时，如果既不能找到也不能创建
环境变量中指定的make文件时，make不会提示错误；make在执行时，首先读取环境变量"Makefiles"中的文件，然后才是当前工作目录下的make文件，
而"include"则是在make发现此关键字的时、暂停正在读取的文件而转去读取“include”所指定的文件。

非常不推荐设置这个环境变量，可以想象，当进行多级make调用时，每一级make都需要读取该环境变量中的文件，会使调用关系出现混乱。

## 参考资料

* 《GNU make中文手册》
*  <http://wiki.ubuntu.org.cn/%E8%B7%9F%E6%88%91%E4%B8%80%E8%B5%B7%E5%86%99Makefile:MakeFile%E4%BB%8B%E7%BB%8D>

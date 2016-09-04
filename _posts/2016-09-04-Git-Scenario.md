---
layout:     post
title:      "Git基础应用"
subtitle:   "一些简单场景下的git命令使用"
date:       2016-09-04
author:     "ClaireChin"
header-img: "img/post-bg-alitrip.jpg"
catalog: true
tags:
    - git
    - command
    - develop
---
>这是对我在工作中遇到的一些git命令使用场景的归纳和总结，所有的命令都已经经过我的实践，如有不妥之处，敬请指出。

# Git Scenario

------

文中以"//"开始的内容表示对相应命令的注释。本文主要由基本概念和主要应用场景两部分组成。

> * 基本概念（Basic concepts）
> * 主要使用场景

![git-logo](http://www.theziqi.cn/wp-content/uploads/2013/04/gitcafelogo.png)

------

## 基本概念
![git-repository](http://static.oschina.net/uploads/img/201510/20133801_xFlQ.jpg)
<br>
首先需要分清一些基本概念，其中，
**workspace**：或者称为working-tree，是你当前所工作的目录，每当你在代码中进行了修改，working tree的状态就改变了；
<br>
**index**：staging area，它是连接workspace和local repository的桥梁，每当使用git add命令进行登记后，index的内容就改变了，此时index和workspace同步。
<br>
**commit**：是最后的阶段，只有commit了，我们的代码才真正进入了仓库（local repository）。git commit命令就是将index file的内容提交到commit中。
<br>
**master**：Git会使用master作为分支（branch）的默认名字。
<br>
**HEAD**：指向本地当前branch，本地当前的branch可以用"git branch"查看。
<br>

### 1. Git与Svn功能对照
* svn checkout
<br>
git clone https://github.com/ClaireChin/clairechin.github.io.git
<br>
//从该网址克隆一份代码到当前目录
* svn show log
<br>
git log --oneline --decorate --graph --all
* svn check for modificaitons
<br>
git diff
<br>
此命令比较的是workspace和index之间的差异，也就是修改之后还没有暂存起来的变化内容。
<br>
git diff --cached（或者git diff --staged）
<br>
//查看index file与commit的差别，显示结果中的a表示index, b表示commit
<br>
git diff HEAD                           
<br>
//查看working tree和commit的差别
* svn revert
<br>
如果在workspace中进行的修改还没有add到index file中，则可以通过git checkout -- file来撤销对工作区的修改。这个命令是以最新的存储时间节点（add和commit）为参照，覆盖工作区对应文件file，这个命令改变的是工作区。
如果已经将workspace中进行的修改add到index区域，则可以使用git reset HEAD -- file清空add命令向暂存区提交的关于file文件的修改（Unstage）；这个命令仅改变暂存区，并不改变工作区，这意味着在无任何其他操作的情况下，工作区中的实际文件同该命令运行之前无任何变化。
* svn add
<br>
git add filepath
<br>
//add to index only files created or modified and not those deleted
<br>
可以通过git add filepath的形式把filepath添加到索引库中，filepath可以是文件也可以是目录。
git不仅能判断出filepath中，修改（不包括已删除）的文件，还能判断出新添的文件，并把它们的信息添加到index中。
<br>
使用git rm files命令时，删除文件这一变化会被添加到index中，再使用git checkout -- file也无法将被删除的文件撤回。也可以手动直接将某个文件删除，例如xxx.txt，若再使用git checkout--file可以将被删除的文件撤回，但是git add无法将该变化添加到index中。对于这种场景的解决方法是使用git add -A或者 使用 git add --all，这里 -A 是 --all 的一个简写。运行git add -A 后，原来手动删除的文件，在git中也被删除了。
<br>
git add -A: [filepath]表示把filepath中所有tracked文件中被修改过或已删除文件和所有untracted的文件信息添加到索引库。省略filepath表示则当前目录。
<br>
git add -u
<br>
//add to index only files modified or deleted and not those created 
git add -u [filepath]: 把<path>中所有tracked文件中被修改过或已删除文件的信息添加到索引库。它不会处理untracted的文件。省略filepath表示当前目录。
<br>
那么存在一个问题，如何暂存当前正在进行的工作？
<br>
可以使用git stash。git stash可用来暂存当前正在进行的工作， 比如想pull最新代码， 又不想加新commit， 或者另外一种情况，为了fix一个紧急的bug,  先stash, 使返回到自己上一个commit, 改完bug之后再stash pop, 继续原来的工作。
<br>
但多次git stash之后又会出现新的问题，应该使用栈中的哪个修改版本？
<br>
git stash list命令可以将当前的Git栈信息打印出来，你只需要将找到对应的版本号，例如使用git stash apply stash@{1}就可以将你指定版本号为stash@{1}的工作取出来，当你将所有的栈都应用回来的时候，可以使用git stash clear来将栈清空。
* svn merge
<br>
对应的git命令会在第二部分的场景2和场景3中有详细介绍。
* svn commit
<br>在git add, git commit, git rebase进行成功之后可以使用git push将代码push到Gerrit上。
<br>关于push命令，可以理解一下：    
    <br>（1） origin/master：origin远程库（remote repository）的master分支
    <br>（2） HEAD指向当前工作的branch，master不一定指向当前工作的branch
    <br>（3） git push origin 本地分支A : 远程分支B //push 本地分支A到远程库origin的分支B
<br>在clone完成之后，Git会自动为你将此远程仓库命名为origin（origin只相当于一个别名，运行git remote–v或者查看.git/config可以看到origin的含义），并下载其中所有的数据，建立一个指向它的master分支的指针，我们用(远程仓库名)/(分支名) 这样的形式表示远程分支，所以origin/master指向的是一个remote branch（从那个branch我们clone数据到本地），但你无法在本地更改其数据。同时，Git会建立一个属于你自己的本地master 分支，它指向的是你刚刚从remote server传到你本地的副本。随着你不断的改动文件，git add, git commit，master的指向会自动移动，你也可以通过merge（fast forward）来移动master的指向。
<br>一些git push命令的介绍：
<br>git push origin master:master 
<br>在local repository中找到名字为master的branch，使用它去更新remote repository下名字为master的branch，如果remote repository下不存在名字是master的branch，那么新建一个。
<br>git push origin master     
<br>省略<dst>等价于”git push origin master:master”。
<br>git push origin master:refs/for/mybranch 
<br>在local repository中找到名字为master的branch，用它去更新remote repository下面名字为mybranch的branch。
<br>git push origin HEAD:refs/for/mybranch
<br>HEAD指向当前工作的branch，master不一定指向当前工作的branch，推荐使用。
<br>git push origin    :mybranch 
<br>在origin repository里面查找mybranch，删除它。用一个空的去更新它，就相当于删除了。
<br>通常不建议使用git pull，具体原因可以查看下面的网址，
<br>git fetch and merge, don't pull (http://www.oschina.net/translate/git-fetch-and-merge?cmp)。

------

作者：ClaireChin     
2016 年 09月 04日    


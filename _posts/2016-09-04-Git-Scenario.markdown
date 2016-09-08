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
>这是对我在工作中遇到的一些git命令使用场景的归纳和总结，所有的命令都已经过实践，如有不妥之处，敬请指出。

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
任何知识的学习都需要从基本概念开始。
<br>
**workspace**：或者称为working-tree，是你当前所工作的目录，每当你在代码中进行了修改，working tree的状态就改变了；
<br>
**index**：staging area，它是连接workspace和local repository的桥梁，每当使用*git add*命令进行登记后，index的内容就改变了，此时index和workspace同步。
<br>
**commit**：是最后的阶段，只有commit了，我们的代码才真正进入了仓库（local repository）。git commit命令就是将index file的内容提交到commit中。
<br>
**master**：Git会使用master作为分支（branch）的默认名字。
<br>
**HEAD**：指向本地当前branch，本地当前的branch可以用*git branch*查看。前面带“*”的branch表示当前branch。
<br>

### 1. Git与Svn功能对照
* svn checkout
<br>
git clone https://github.com/ClaireChin/clairechin.github.io.git
<br>
//从该网址克隆一份代码到当前目录
* svn show log
<br>
git log - -oneline - -decorate - - graph - -all
* svn check for modificaitons
<br>
git diff
<br>
//此命令比较的是workspace和index之间的差异，也就是修改之后还没有暂存起来的变化内容。
<br>
git diff - -cached（或者git diff - -staged）
<br>
//查看index file与commit的差别，显示结果中的a表示index, b表示commit
<br>
git diff HEAD                           
//查看working tree和commit的差别
* svn revert
<br>
如果在workspace中进行的修改还没有add到index file中，则可以通过*git checkout - - file*来撤销对工作区的修改。这个命令是以最新的存储时间节点（add和commit）为参照，覆盖工作区对应文件file，这个命令改变的是工作区。
如果已经将workspace中进行的修改add到index区域，则可以使用*git reset HEAD  - - file*清空add命令向暂存区提交的关于file文件的修改（Unstage）；这个命令仅改变暂存区，并不改变工作区，这意味着在无任何其他操作的情况下，工作区中的实际文件同该命令运行之前无任何变化。
* svn add
<br>
git add filepath
<br>
//add to index only files created or modified and not those deleted
<br>
可以通过*git add filepath*的形式把filepath添加到索引库中，filepath可以是文件也可以是目录。
git不仅能判断出filepath中，修改（不包括已删除）的文件，还能判断出新添的文件，并把它们的信息添加到index中。
<br>
使用*git rm files*命令时，删除文件这一变化会被添加到index中，再使用*git checkout - - file*也无法将被删除的文件撤回。也可以手动直接将某个文件删除，例如xxx.txt，若再使用*git checkout - -xxx.txt*可以将被删除的文件撤回，但是*git add*无法将该变化添加到index中。对于这种场景的解决方法是使用*git add -A*或者使用*git add - -all*，这里 -A 是- -all 的一个简写。运行*git add -A*后，原来手动删除的文件，在git中也被删除了。
<br>
git add -A: [filepath]表示把filepath中所有tracked文件中被修改过或已删除文件和所有untracted的文件信息添加到索引库。省略filepath表示则当前目录。
<br>
git add -u
<br>
//add to index only files modified or deleted and not those created 
git add -u [filepath]: 把filepath中所有tracked文件中被修改过或已删除文件的信息添加到索引库。它不会处理untracted的文件。省略filepath表示当前目录。
<br>
<br>
那么存在一个问题，如何暂存当前正在进行的工作？
<br>
可以使用git stash。git stash可用来暂存当前正在进行的工作， 比如想pull最新代码， 又不想加新commit， 或者另外一种情况，为了fix一个紧急的bug,  先stash, 使返回到自己上一个commit, 改完bug之后再stash pop, 继续原来的工作。
<br>
<br>
但多次git stash之后又会出现新的问题，应该使用栈中的哪个修改版本？
<br>
git stash list命令可以将当前的Git栈信息打印出来，你只需要将找到对应的版本号，例如使用git stash apply stash@{1}就可以将你指定版本号为stash@{1}的工作取出来，当你将所有的栈都应用回来的时候，可以使用git stash clear来将栈清空。
* svn merge
<br>
对应的git命令会在第二部分的场景2和场景3中有详细介绍。
* svn commit
<br>在git add, git commit, git rebase进行成功之后可以使用git push将代码push到Gerrit上。

### 2. 关于push命令
在clone完成之后，Git会自动为你将此远程仓库命名为origin（origin只相当于一个别名，运行git remote–v或者查看.git/config可以看到origin的含义），并下载其中所有的数据，建立一个指向它的master分支的指针，我们用(远程仓库名)/(分支名) 这样的形式表示远程分支，所以origin/master指向的是一个remote branch（从那个branch我们clone数据到本地），但你无法在本地更改其数据。同时，Git会建立一个属于你自己的本地master 分支，它指向的是你刚刚从remote server传到你本地的副本。随着你不断的改动文件，git add, git commit，master的指向会自动移动，你也可以通过merge（fast forward）来移动master的指向。

    （1） origin/master：origin远程库（remote repository）的master分支;
    （2） HEAD指向当前工作的branch，master不一定指向当前工作的branch；
    （3） git push origin 本地分支A : 远程分支B //push 本地分支A到远程库origin的分支B。
    
一些git push命令的介绍：
* git push origin master:master 
<br>//在local repository中找到名字为master的branch，使用它去更新remote repository下名字为master的branch，如果remote repository下不存在名字是master的branch，那么新建一个。
* git push origin master     
//省略<dst>等价于*git push origin master:master*。
* git push origin master:refs/for/mybranch 
<br>//在local repository中找到名字为master的branch，用它去更新remote repository下面名字为mybranch的branch。
* git push origin HEAD:refs/for/mybranch
<br>//HEAD指向当前工作的branch，master不一定指向当前工作的branch，推荐使用。
* git push origin    :mybranch 
<br>//在origin repository里面查找mybranch，删除它。用一个空的去更新它，就相当于删除了。

通常不建议使用git pull，具体原因可以查看下面的网址，
<br>git fetch and merge, don't pull (http://www.oschina.net/translate/git-fetch-and-merge?cmp)。

## 主要使用场景
### a. Scenario 1

场景描述：Gerrit上没有需要review的代码，当前的工作目录的修改可能是基于最新版本的代码或者与最新版本的代码已经差了好几个他人提交的submit，准备提交工作目录的修改到Gerrit上。
<br>在当前master分支上基于某个版本进行了修改，

    git add .                                //添加所有的修改文件到index中
    git commit                               //将index中的内容提交到local repository中
    git fetch                                //将remote repository的内容获取到local repository
    git rebase origin/master 
    
进行rebase时会有以下两种情况，
* case 1
<br>在输入git rebase origin/master 后提示“Current branch master is up to date.”，这种情况说明没有冲突产生，并且rebase成功完成，直接输入git push origin HEAD:refs/for/master，将代码push到remote repository。
* case 2
<br>提示有冲突产生，并且在括号内的master后面有rebase进度提示。

针对case2，可以输入以下命令，

    git mergetool                            //在当前系统或terminal的默认合并工具下解决冲突并保存修改，关闭合并工具窗口
    git rebase --continue
    git status                               //查看是否因为使用merge tool而产生了一些.orig文件
    git clean -nf                            //查看并确认哪些文件将被删除
    git clean -f                             //删除这些untratced且不需要git add的文件
    git push origin HEAD:refs/for/master      

此处补充mergetool的安装步骤，可使用的合并工具有tortoisemerge，kdiff3，meld等。
以使用kdiff3作为merge工具为例，

    git config --global merge.tool KDiff3    //配置merge工具
    
如果提示"The merge tool KDiff3 is not available as 'KDiff3', 说明此机或当前terminal上并未安装KDiff3，可以将以下内容复制到.git/config文件中，

    setting
    [merge]
    tool = kdiff3

    [mergetool "kdiff3"]
        path = C:/Program Files/KDiff3/kdiff3.exe
        keepBackup = false
    trustExitCode = false
    
同样地，如果想设置git diff的tool，

    git config --global diff.tool KDiff3

然后在.git/config文件中输入，

    [diff]
            tool = kdiff3
            guitool = kdiff3
    [difftool "kdiff3"]
            path = c:/Program Files/KDiff3/kdiff3.exe
            
diff工具也可以配置成KBeyond Compare 4。
<br>如何理解KDiff3中的base，local和remote，以及tortoisemerge中的mine和theirs概念，可以参考(http://stackoverflow.com/questions/3051461/git-rebase-keeping-track-of-local-and-remote)。
<br>另外，使用tortoisemerge进行冲突解决会产生以".orig"为后缀的untracked文件，使用KDiff3进行冲突解决会产生多个备份文件（untracted），此时可以使用git clean来清除untracted的文件，

    git clean -f                        //删除 untracked files
    git clean -fd                       //连untracked的目录也一起删掉
    git clean -xfd                      //连gitignore的untrack文件/目录也一起删掉（慎用，一般这个是用来删掉编译出来的 .o之类的文件用的）
    
在用上述git clean命令前，强烈建议加上-n参数来先看看会删掉哪些文件，防止重要文件被误删。

    git clean -nf 
    git clean -nfd
    git clean -nxfd 
    
更多的merge tool可以参考(http://blog.jobbole.com/97911/)。

### b. Scenario 2

场景描述：代码已经push到Gerrit上，在submit之前的review需要经历一定的时间，而在这个时间段内产生了多个submit，并且可能在这些submit中有一个或多个submit修改了与自己相同的一个或多个文件，因此在rebase的过程中会产生冲突。
<br>针对工作区内已经被修改过的文件,
+ 被修改了但不需要被提交的文件
<br>比如在编译过程中被修改的文件或debug过程中产生的debug文件夹, 将不需要*git add*的untracted文件在工作区中手动删除或使用*git clean -f*删除;将在编译过程中被修改的文件使用*git checkout - -file*撤销对工作区修改。
+ 被修改且需要提交的文件

对于被修改且需要提交的文件来说，

    git add .
    git commit                    //或git commit --amend
    git fetch                     //千万不要直接使用pull！！！
    git rebase origin/master                          
    
如果在rebase的过程中有报冲突，保持rebase的状态不变（即不要使用git rebase --abort），解决冲突，解决冲突的方法如下:

    git mergetool
    
如果没有安装特定的merge tool的话，此时会有提示"Hit return to start merge resolution tool (tortoisemerge):"，按回车键选择默认合并工具，在合并工具中针对产生冲突的地方进行修改，修改完成后保存修改并关闭工具窗口返回git bash的rebase状态。冲突解决也可以直接手动解决，如果手动解决，在rebase之前需要先执行git add。解决完冲突后依次输入如下命令。

    git rebase --continue                        //完成rebase过程
    git status                                   //查看是否因为使用merge tool而产生了一些.orig文件
    git clean -nf                                //查看并确认哪些文件将被删除
    git clean -f                                 //删除这些untratced且不需要git add的文件
    git push origin HEAD:refs/for/master

需要注意的是，在确保工作区中没有unstage的文件后进行rebase。成功submit以后在保证工作目录是干净的情况下可以使用git pull更新本地代码，否则谨慎使用git pull。

### c. Scenario 3

场景描述：代码push到Gerrit上以后，review的过程比较长，在等待review完成的过程中需要继续一些task。另外，在该场景处理的过程中不存在任何冲突。
<br>解决方法：新建branch。
<br>参考博客来源(http://blog.csdn.net/hustpzb/article/details/7287948)。

+ 关于branch
<br>Git中的分支，其实本质上仅仅是个指向commit对象的可变指针。Git会使用master作为分支的默认名字。在若干次提交后，你其实已经有了一个指向最后一次提交对象的master分支，它在每次提交的时候都会自动向前移动。Git 是通过创建一个新的分支指针来创建一个新的分支的。

+ Git是如何知道你当前在哪个分支上工作的呢？
<br>它保存着一个名为HEAD的特别指针。请注意它和你熟知的许多其他版本控制系统（比如 Subversion 或 CVS）里的HEAD概念大不相同。在Git 中，它是一个指向你正在工作中的本地分支的指针（注：将HEAD想象为当前分支的别名）。

+ 值得牢记
<br>Git会把工作目录的内容恢复为检出某分支时它所指向的那个提交对象的快照。它会自动添加、删除和修改文件以确保目录的内容和你当时提交时完全一样。


    git branch branchName                  //新建分支，分支名字是branchName                  
    git branch                             //查看有哪些分支，并且在当前分支的名字前面会有一个"*"
    git checkout xxx                       //切换到xxx分支
    
在新建的branchName分支上继续task，task完成后准备提交代码,

    git add .                              //task完成后将需要提交的文件添加到index中
    git commit                             //将index中的内容提交到local respository中

等待原来的review过程结束并成功submit，在这期间针对review可能包含多次修改和重新push，

    git checkout master
    git status
    git fetch
    git rebase origin/master
    
无冲突提示，

    git merge branchName                   //将branchName分支合并到master分支
    //进行fast-forward合并且无冲突提示
    //git status查看一下状态，应该会提示有一个commit需要push
    git branch --merged                    //查看已merge的分支状态（相应地，如果建了多个branch，还可以使用git branch --no-merged查看未合并的分支状态）
    
确定分支已经合并成功，

    git fetch
    git rebase origin/master
    //无冲突提示
    git push origin HEAD:refs/for/master
    git branch -d branchName               //删除分支

### d. Scenario 4

场景描述：代码push到gerrit上以后，review的过程比较长，在等待review完成的过程中需要继续一些task。该场景与scenario 3的区别在于进行分支合并的过程中会产生一系列的冲突。

可以输入如下命令，

    git branch不带参数：                  //列出本地已经存在的分支，并且在当前分支的前面加“*”号标记
    git branch xxx：                      //创建一个新的本地分支xxx，需要注意，此处只是创建分支，不进行分支切换
    git checkout branchName：             //切换到名字为“branchName”的分支上

在名字为“branchName”的分支上进行一系列修改后，在该分支上,

    git add 
    git commit
    
待master分支代码根据review意见进行修改并成功submit之后，

    git checkout master                  //切换到master分支
    git fetch                            //在master分支下
    git rebase origin/master
    git merge branchName                 //将branchName分支合并到master分支

如果报冲突，手动解决冲突或者git mergetool解决，

    git fetch
    git rebase origin/master
    
报冲突解决冲突，解决方法同上，

    git rebase --continue
    
如果提示如下错误并且rebase阶段显示为"REBASE 1/2",

    Applying：Study the review delay problem with branch
    No changes - did you forget to use 'git add' ?
    If there is nothing left to stage, chances are that something else 
    already introduced the same changes; you might want to skip this patch.
    
输入命令*git rebase --skip*后（http://wholemeal.co.nz/blog/2010/06/11/no-changes-did-you-forget-to-use-git-add/）
rebase阶段会由REBASE 1/2变成REBASE 2/2。

    git rebase --continue
    //报冲突解决冲突
    git rebase --continue
    git push origin HEAD:refs/for/master
    git branch -d branchName
    
注意不能在当前所在的branchName分支上删除branchName分支，必须切换到其他分支才能删除branchName分支，比如切换到master再删除branchName分支。如果要删除的分支已经成功合并到当前分支，删除分支的操作会直接成功；如果要删除的分支没有合并到当前所在分支，则会出现提示，如果确定无须合并而要直接删除，则执行命令：git branch –D branchName进行强删。如何查看分支是否合并，可以参考Scenario 3。

### e. Scenario 5

场景描述：你push到Gerrit上的代码需要review，但是正好之前person A push到Gerrit上的代码也正在review的过程中，并且在你push代码到Gerrit上时，由于代码中修改了一部分与person A相同的代码，person A和你的Gerritreview页面均出现confilct提示。
或者，你push到Gerrit上的代码需要review，但是在你push代码到Gerrit上之后，person A也将修改的代码push到Gerrit上进行review，并且person A修改了一部分与你处于相同位置的代码，person A和你的Gerrit review页面均出现confilct提示。

+ Case 1： person A的review先完成并且submit
<br>自己的Gerrit页面会出现"Cannot Merge"的提示。

此时需要在本地解决冲突并重新push到remote repository，

    git fetch
    git rebase origin/master
    git mergetool
    git rebase --continue

清理不必要的文件，

    git status
    git clean -nf
    git clean -f
    git push origin HEAD:refs/for/master

+ Case 2：你的review先完成
<p>那么直接submit就行了。


------

作者：ClaireChin     
2016 年 09月 04日    


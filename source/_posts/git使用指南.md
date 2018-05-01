title: "git使用指南"
date: 2016-05-07 16:05:16
categories: 技术 #文章文类
tags: [git]  #文章标签，多于一项时用这种格式[]
description: git的常用命令和分支管理策略
keywords: 版本管理工具
---
　　使用git差不多一年多了，越来越喜欢。相比svn等其他的，离线也能工作，并且有强大的分支管理能力。机缘巧合之下（其实是组长让整理的），我整理了一下git的常用命令和我推崇的分支管理策略。

### git工作流
![](/imgs/201605070001.png)
 **工作区**：

 　　就是你在电脑上看到的目录，比如目录下testgit里的文件(.git隐藏目录版本库除外)。或者以后需要再新建的目录文件等等都属于工作区范畴。

 **版本库(Repository)**：

 　　工作区有一个隐藏目录.git，这个不属于工作区，这是版本库。其中版本库里面存了很多东西，其中最重要的就是stage(暂存区)。

 **Git提交文件到版本库有两步：**

 　　第一步：是使用 git add 把文件添加进去，实际上就是把文件添加到暂存区。
 　　第二步：使用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支上。图中的head永远指向当前分支的最新版本。它是一个指针，切换分支它会跟着变。（ps：git内部跟多高效的设计跟这个head有关，我研究的不深，有兴趣自己去探索。。。）


 <!--more-->

### git分支策略
![](/imgs/201605070002.png)
**master**：

　　用来上线的版本。当代码测试通过，把代码合并到master分支，打上tag。上线打包只用master分支拉代码。

**fixbug**：

　　用于线上bug修复。当线上出现比较严重的bug需要紧急修复。从master分支checkout -b 一个fixbug版本。修复测试通过，merge到master。再由master打包上线。

**release**：

　　测试版本。开发自测完毕之后，merge代码到release版本。出现bug，再以release版本为基线，拉出fixbug版本去修复。修复完毕，合并fixbug版本。确定测试ok之后，合并代码到master和develop分支。

**develop**：

　　开发用的版本。

**feature**：

　　候选特效开发，暂时不上线的功能放这里。

　　我们日常开发都在develop分支完成，**建议每开发一个功能点就以develop为基线拉一个新版本，在新版本上开发，开发完毕合并到develop。这种方式你的代码非常安全，不管怎么被别人覆盖，总能找回来。你未完成的中间代码也不会影响到其他童鞋。因为是多人协同开发，不可避免会出现冲突（事先分配好任务，修改公共的代码在群里吼一声），不要暴力解决冲突，找到冲突的代码，找写冲突的代码的人，协商去解决。
push之前一定要先pull！！！**
    <br/>

### git命令
1. **创建分支**

    创建分支很简单：git branch <分支名>

2. **切换分支**

    git checkout <分支名>

    该语句和上一个语句可以和起来用一个语句表示：git checkout -b <分支名>

3. **把远程分支内容拉到本地并且创建对应的本地分支**

    git checkout -b <分支名> origin/<分支名>

    在这样创建的分支执行git push，可以直接push到对应的远程分支，不必要使用git push origin <分支名>指定

4. **分支合并**

    比如，如果要将开发中的分支（develop），合并到稳定分支（master），

    首先切换的master分支：git checkout master。

    然后执行合并操作：git merge develop。

    如果有冲突，会提示你，调用git status查看冲突文件。

    解决冲突，然后调用git add或git rm将解决后的文件暂存。

    所有冲突解决后，git commit 提交更改。

5. **分支衍合**

    分支衍合和分支合并的差别在于，分支衍合不会保留合并的日志，不留痕迹，而 分支合并则会保留合并的日志。

    要将开发中的分支（develop），衍合到稳定分支（master）。

    首先切换的master分支：git checkout master。

    然后执行衍和操作：git rebase develop。

    如果有冲突，会提示你，调用git status查看冲突文件。

    解决冲突，然后调用git add或git rm将解决后的文件暂存。

    所有冲突解决后，git rebase --continue 提交更改。

6. **删除本地分支**

    执行git branch -d <分支名>

    如果该分支没有合并到主分支会报错，可以用以下命令强制删除git branch -D <分支名>

7. **删除远程分支**

    删除远程分支develop：git push origin :develop

8. **同步本地远程分支**

    执行git branch -a 可以查看本地和远程分支，但是其他小伙伴创建的远程分支，你不知道咋办？

    执行git fetch origin，再执行git branch -a就可以看到最新的远程分支了

9. **缓存本地更改**

    你在develop分支写代码写的正开心，但是但是突然有紧急bug需要修复，你需要切换到master分支去修复bug。

    假如你直接git checkout master会把正在开发的代码带到master分支。Checkout之前先提交？但是功能又没写完。

    这时候可以使用 git stash，将可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作。

    修复完成之后回到develop分支，执行git stash pop，可以恢复缓存的更改。

10. **回退**

    git checkout -- <filename> 将未commit的一个或者多个文件回退到HEAD版本。head永远指向当前分支的最新版本。

    git还有更强大的功能，让我们在历史版本之间来回穿梭，使用git reset --hard commit_id

    使用git log可以查看提交的历史，找到你想回退的版本号。

    使用git reflog可以查看命令历史，找到你回到未来的版本。

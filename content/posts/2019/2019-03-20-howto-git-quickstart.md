---
title: "Git快速指南"
slug: git-quickstart
date: 2019-03-20T18:45:00+08:00
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-flower.jpg
thumbnailImagePosition: right
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-flower.jpg
categories:
- dev
tags:
- git
- github
---

很多公司内部使用GitLab来做版本管理，假设你已经按照公司wiki设置好了Git账户。那么如何快速开始使用Git进行代码版本管理，下面是一些常用流程步骤。

<!--more-->

{{< toc >}}

## 1. 快速开发

接到**Boss**命令，XX项目必须明天完成上线，而此时你才配置好git账户。一脸懵逼的你得知小伙伴已经将项目基础代码上传至Git版本库，地址就在`https://xxx.xxx.xxx.xxx/xxxxx/xxxxx.git`中的`project-dev`开发分支下。小激动一下，已经等不急就开始开发，赶紧完成上线，可不能影响周末的休息skrskr。

### 1.1 初始化本地版本库

在`Git Bash`中运行如下命令：

```
cd /project/gitlab
```

`/project/gitlab`你存放项目的本地目录。

```
git init
```

上述命令主要用来初始化一个空的`git`本地仓库。执行完上面的命令，当前目录下会自动生成`.git`隐藏文件夹，该隐藏文件夹就是`git`版本库. 用`ls -ah`命令就可以看见。


### 1.2 建立与远程版本库的连接

```
git remote add origin https://xxx.xxx.xxx.xxx/xxxxx/xxxxx.git
```

### 1.3 拉取远程开发分支到本地

```
git fetch origin project-dev
```

### 1.4 在本地创建分支

```
git checkout -b project-dev-xxx origin/project-dev
```

此时，远程版本库的project-dev分支内容已经被拉取到本地了。OK，还未平复刚才的小激动，而此时已经可以开动键盘和鼠标，代码~~ctrl-c/ctrl-v~~写起来了。

不一会功夫，高效的你已经进行了上百次下面的代码提交操作：

```
git add xxxx(修改的文件或者目录)
git commit -m '提交的简要说明'
```

“可是在Git版本库的web页上没看到提交记录啊”，对项目进度盯地很紧的PM默默在你身后嘀咕了这么一句，一下子打断了你周末去哪high的头脑风暴。

怎么可能，一定是你没刷新，在我电脑上明明…………，哦，不要急哈，我还没push，稍等哈。向来对代码质量要求苛刻的你，肯定会检查仔细才能把完全没有bug的代码上传嘛，明明离上线还有18个小时~~

### 1.5 分支策略

你知道这些修改只是在本地分支`project-dev-xxx`中进行了提交，要推送到远程版本库，有两种方法：

第一种方法：直接推到远程版本库的`project-dev`开发分支中：

```
git push origin HEAD:project-dev
```

这个方法直接，但是多人协作的时候，容易产生冲突，到时解决冲突很花时间。**你决定不这么做。**

第二种方法：通过分支合并的方式。将`project-dev-xxx`合并到`project-dev`

对于`project-dev-xxx`这样一个**临时**开发分支，你决定将它push到远程版本库中（也可以不push到远程，看自己需要），因为有时需要换一台工作站继续站着开发(Just kidding :D)。

```
git push origin project-dev-xxx
```

本地分支`project-dev-xxx`与已上传到远程版本库`project-dev-xxx`建立连接

```
git push --set-upstream origin project-dev-xxx
```

下次只要

```
git push
```

即可直接push到`project-dev-xxx`


## 2. 合并分支

开发完成了，在自己本地也测试OK了，这时你决定立刻将自己的修改合并到`project-dev`主开发分支中去，交差然后早点下班。

你严格按照前面的步骤进行的Git操作，此时输入命令`git branch`时，你只能看到一个分支：

```
* project-dev-xxx
```

`*`号表示你当前所在分支。现在你打算将该分支下的修改合并到远程版本库`origin/project-dev`分支中去。首先将远程版本库的`project-dev`分支拉取下来，并在本地创建分支

```
git checkout -b project-dev origin/project-dev
```

执行完这个命令，会自动切换至该分支，此时输入命令`git branch`，看到当前分支已经切换至`project-dev`，如下：

```
* project-dev
  project-dev-xxx
```

可以执行`merge`分支合并操作了，命令很简单，执行：

```
git merge project-dev-xxx
```

大功告成，你之前分支中`project-dev-xxx`辛苦两小时开发的内容已经合并到`project-dev`，下面只需一步`push`到远程版本库，就完成了代码的提交。

```
git push
```

周末的休息，稳了。


## 3. 更新分支

这次上线工作好在是小菜一碟，一个人就轻松搞定了。后来业务做大了，团队也壮大了，**开发力量翻了整整一倍**。另一个人刚开始用`git`，经过你的悉心指教，已经熟练掌握上面的步骤，很快进入两人协作开发模式，也实现了你梦寐以求的`pair programming`。

俗话说的好：
> 有人的地方就有~~浆糊~~江湖，有江湖的地方就有冲突。

两人开发难免遇到这样的问题，开始用git，不知道如何协作，都在闷头写代码，一个月之后一起提交了，我槽，上万个冲突。对于程序员来说，必须要坚决捍卫自己的代码。有冲突只能对方改，结对的小船说翻就翻。

其实很简单，`git`的本质就是更方便的进行冲突管理，大型的同性交友网站`Github`就是得益于`git`的强大的管理机制。

对于简单的多人协作，掌握一下`rebase`即可。

别人提交了代码，自己本地主分支`project-dev`需要更新一下。按照下面几个步骤，可以保证尽量少冲突，即使有冲突也拿到本地来解决。

1. 先切换到本地主分支

```
git checkout project-dev
```

2. 更新本地主分支

```
git pull
```

3. 然后切换到自己的开发分支

```
git checkout project-dev-xxx
```

4. 将自己的开发分支`rebase`到最新的主分支中

```
git rebase project-dev
```

在`rebase`时，`git`会将`project-dev`分支`merge`到`project-dev-xxx`分支，并且保持`commit`历史记录清爽宜人。完全不用担心代码冲突带来的不快，遇到冲突自己在本地编辑一下，就当啥事没发生。代码千万行，友谊第一行。

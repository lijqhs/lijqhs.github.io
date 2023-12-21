---
title: "Github Pages Hugo博客主题更新踩坑小记"
slug: pitfalls-in-updating-hugo-blog-theme
date: 2022-03-24
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-nature.jpg
thumbnailImagePosition: right
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-nature.jpg
categories:
- blog
tags:
- hugo
- github
- git
---

好久没用Github，这个博客也很长时间没有更新了，几乎已经被遗忘了。之前喜欢在工作之余利用Github来记一些笔记<!--more-->，后来一方面因为工作类型的转换，从做算法研究写代码转到做项目管理，一下子从跟电脑打交道转变为更多地与人打交道，节奏变化了记笔记的方式也因此变化了；另一方面也是因为懒，年龄越大越不喜欢折腾。以前喜欢用Markdown做笔记，甚至用VSCode把Github作为自己的笔记本，后来工作之外的时间越来越少，而且大部分时间花在与人沟通上面，自然就少了很多精力去折腾笔记的形式了。这个时候简单易用最重要，试过很多笔记应用，最喜欢的还是Evernote，非常适合项目管理过程中做项目进度、项目会议、任务计划管理的相关笔记。不过有时间还是想通过markdown用博客的形式来记录一些经验心得，比如踩坑经历。

{{< toc >}}

当我准备在这个博客上重新更新的时候，发现之前很熟练的git命令和markdown tag忘得差不多了，果真是用进废退啊。在折腾更新博客主题时，踩了好些坑，在此记下来避免后续再遇到同类问题。

## 更新Hugo

### 升级CommandLineTools

因为这个博客是用Hugo搭建的，好久不用自然就想着先升级一下Hugo，运行
```bash
brew upgrade hugo
```

弹出如下错误信息：
```bash
Error: Your CLT does not support macOS 12.2.
It is either outdated or was modified.
Please update your CLT or delete it if no updates are available.
```

因为Homebrew是需要CommandLineTools的，这里提示CLT过时了，通过删除和重新安装xcode-select可以解决此问题：
```
sudo rm -rf /Library/Developer/CommandLineTools
sudo xcode-select --install
```

### 更新Homebrew

更新软件包之前，最好还是先升级一下Homebrew，于是运行
```bash
brew update
```

又报错
```bash
cannot open .git/FETCH_HEAD: Permission denied
```

查了一下stackoverflow，这个错误是因为我在Mac上用了跟之前安装Homebrew不一样的用户，而涉及Homebrew的多用户使用需要做一些复杂的[配置][homebrew-multiuser]，而我不想把时间花在这个上面，于是切回到之前的Mac用户来升级Homebrew，顺便升级所有软件包，依次执行：
```bash
brew update
brew upgrade
```

这里需要提醒的是，要根据所在地考虑更换Homebrew镜像源，比如在国内可以更换为科大或者阿里的镜像源，运行上面命令时会快一点。

配置成阿里镜像源，用
- `https://mirrors.aliyun.com/homebrew/brew.git`
- `https://mirrors.aliyun.com/homebrew/homebrew-core.git`

如果需恢复镜像源配置，则将命令中的git URL换成
- `https://github.com/Homebrew/brew.git`
- `https://github.com/Homebrew/homebrew-core.git`

并且删除下面的`HOMEBREW_BOTTLE_DOMAIN`即可。

配置镜像源依次执行以下命令：
```bash
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git

brew update
```

然后在`.zshrc`中增加`HOMEBREW_BOTTLE_DOMAIN`配置：
```bash
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```

## 更新Hugo博客主题

### Github Personal Access Token

在[Hugo主题网站](https://themes.gohugo.io/)上看到我用的主题[hugo-tranquilpeak-theme](https://github.com/kakawait/hugo-tranquilpeak-theme)有更新，于是就决定最好更新一下博客主题，而且最新版可以支持评论模块，比如disqus.

在命令行运行git pull先更新一下本地的博客管理blog repo（这个和博客发布的repo不一样，注意区别），结果发现Github的认证已经过期了（实在是太久没上Github了。。）
```bash
blog on main 
❯ git pull
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.
fatal: Authentication failed for 'https://github.com/lijqhs/blog.git/'
```

看了一下提示信息中的[链接](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/)，从2021年8月13日开始，Github不再支持账号密码方式的身份验证，而要使用基于{{< hl-text green >}}个人访问令牌{{< /hl-text >}}进行身份验证，也就是说在命令行中运行`git clone`输入账号密码时，不能简单输入用户名和密码，而要输入用户名和令牌的那一串码。

在Github账号设置中的Developer settings下的Personal access tokens中[创建新的token][personal-access-token]，可以选择权限和时间长度。

### 更新子模块submodule

由于主题是通过`git submodule`的方式放在博客管理repo下的，所以现在需要用submodule来更新。之前的[文章]({{< ref "/posts/2020/2020-11-15-howto-change-hugo-blog-theme.md" >}})有详细的submodule讲解，幸好之前花了点时间记录下了配置的细节，为现在的troubleshooting省了很多时间。

更新submodule的命令很简单：
```bash
git submodule update --init
```

## 重新部署博客

既然博客主题更新好了，那么采用Hugo博客主题下`exampleSite`的最新版配置文件`config.toml`进行重新配置也很有必要。然后基于新版Hugo和新版主题重新部署博客就是一句命令的事：
```bash
zsh deploy.sh
```

然而在这之前，我做了一件傻事，让我又绕了一个小弯，从而让这篇文章又增加了以下的文字。我想既然主题更新了，重新部署的时候可能会跟之前的`public`(指向博客发布repo:lijqhs.github.io的目录)不一样可能让文件变得混乱，那么我最好先把旧版的lijqhs.github.io的repo先清空，然后再做rebuild，我当时还为自己的谨慎感到佩服。于是先`git clone` lijqhs.github.io.git到本地，delete掉几乎所有文件，只留了readme，`git commit`，`git push`，只用了不到一分钟。

然后运行`zsh deploy.sh`，按理说只需几秒即可完成，毕竟快就是Hugo最大的优势。

> Hugo: The world’s fastest framework for building websites

结果报了错：{{< hl-text danger >}}HEAD detached from 0b73022{{< /hl-text >}}。看到这个就意识到前面单独清空的操作很不合适很不专业，不过不算麻烦，切换到`public`目录下，进行以下操作即可解决：

```bash
git branch temp
git checkout main
git merge temp
```

至此，博客主题更新好了，又可以继续在本地写博客了，要记得常写哦。。



[homebrew-multiuser]: https://stackoverflow.com/questions/41840479/how-to-use-homebrew-on-a-multi-user-macos-sierra-setup
[personal-access-token]: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token
[git-submodule]: https://git-scm.com/docs/git-submodule
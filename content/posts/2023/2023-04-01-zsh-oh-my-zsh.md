---
title: Zsh, Oh My Zsh
slug: zsh-oh-my-zsh
date: 2023-04-01
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-trees.jpg
thumbnailImagePosition: right
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-trees.jpg
categories:
- dev
tags:
- zsh
---

Zsh（Z shell）是一个Unix shell，旨在成为流行的Bash shell的一个交互式和强大的替代品。它是一个开源的shell，可用于大多数基于Unix的操作系统，包括Linux和macOS，已成为macOS默认的shell。

<!--more-->

{{< toc >}}

## 安装Zsh

在Ubuntu上安装，运行命令：

```bash
sudo apt install zsh
```

验证安装

```bash
zsh --version
```

设置为默认shell

```bash
chsh -s $(which zsh)
```

退出并重新登录以使配置生效。运行 `echo $SHELL` 应返回 `/usr/bin/zsh`。

## 安装oh-my-zsh

Oh My Zsh是一个开源的工具，用于管理zsh配置，[https://github.com/ohmyzsh/ohmyzsh](https://github.com/ohmyzsh/ohmyzsh)

安装：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

注意，任何以前的.zshrc将被重命名为 `.zshrc.pre-oh-my-zsh` 。安装后，你可以将你想保留的配置移到新的 `.zshrc`中。

## 选择主题

通过oh-my-zsh可以为终端配置漂亮的theme，编辑`~/.zshrc`文件，找到 `ZSH_THEME`，修改为想要的主题，[预览主题样式](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes)，如选择"`agnoster"`，修改配置并保存：

```bash
ZSH_THEME="agnoster"
```

## 安装字体

许多主题需要安装[Powerline Font](https://github.com/powerline/powerline)**或**[Nerd Font](https://www.nerdfonts.com/)，以便正确呈现。没有它们，这些主题会呈现出奇怪的提示符号。

### 安装Powerline Font

ubuntu环境下：

```bash
sudo apt install fonts-powerline
```

Mac环境下：

```bash
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```

### 安装Nerd Font

在Ubuntu中，以下代码保存为`get_font.sh`，然后在终端运行`bash get_font.sh`，[Nerd Font字体下载网站](https://www.nerdfonts.com/font-downloads)。

```bash
echo "[-] Download fonts [-]"
echo "https://github.com/ryanoasis/nerd-fonts/releases/download/v2.3.3/Hack.zip"
wget https://github.com/ryanoasis/nerd-fonts/releases/download/v2.3.3/Hack.zip
unzip Hack.zip -d ~/.fonts
fc-cache -fv
echo "done!"
```

在macOS上安装Nerd Font：（完整的安装列表参考[这里](https://gist.github.com/davidteren/898f2dcccd42d9f8680ec69a3a5d350e)）

```bash
# Nerd Fonts for your IDE
# https://www.nerdfonts.com/font-downloads

brew tap homebrew/cask-fonts
brew install --cask font-hack-nerd-font
brew install --cask font-noto-nerd-font
brew install --cask font-fira-code-nerd-font
brew install --cask font-fira-mono-nerd-font
brew install --cask font-dejavu-sans-mono-nerd-font
brew install --cask font-droid-sans-mono-nerd-font
```

隐藏命令行的用户名和主机名显示，修改主题的配置文件，

```bash
vi ~/.oh-my-zsh/themes/agnoster.zsh-theme
```

注释掉`prompt_context`这行：

```bash
build_prompt() {
  RETVAL=$?
  prompt_status
  prompt_virtualenv
  prompt_aws
  # prompt_context
  prompt_dir
  prompt_git
  prompt_bzr
  prompt_hg
  prompt_end
}
```

## 在VS Code上配置字体

经过上述步骤，使用VS Code的时候，也可以使用zsh，如果已经按照之前的步骤安装了Hack字体，直接在VS Code中使用，安装完的字体名称可以在Terminal的Preference字体选择里面找到。

比如在Ubuntu中：

```json
  "editor.fontFamily": "'Hack NF', 'Noto Mono', 'Droid Sans Mono', 'monospace', monospace",
  "terminal.integrated.fontFamily": "Hack NFM"
```

在macOS中：

```json
{
    "editor.fontFamily": "'NotoMono Nerd Font Mono', 'Droid Sans Mono', 'monospace', monospace",
    "terminal.integrated.fontFamily": "Hack Nerd Font Mono"
}
```

或

```json
{
    "editor.fontFamily": "'NotoMono Nerd Font Mono', 'Droid Sans Mono', 'monospace', monospace",
    "terminal.integrated.fontFamily": "NotoMono Nerd Font Mono"
}
```

如果安装了Powerline字体，VS Code终端提示显示却有点问题，此时Powerline字体需要一点额外设置（安装Powerline字体的补丁版本），前面的 "快速安装"（`sudo apt-get install fonts-powerline`）在这种情况下不起作用。需要直接从`https://github.com/powerline/fonts`，安装你想要的字体的补丁版本，参考[StackOverflow](https://stackoverflow.com/questions/62710890/font-issues-while-integrating-zsh-on-visual-studio-code)。

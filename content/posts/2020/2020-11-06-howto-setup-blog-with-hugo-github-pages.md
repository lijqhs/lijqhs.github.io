---
title: "How To Setup a Blog with Hugo and Github Pages on Mac"
date: 2020-11-06T23:19:48+08:00
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-away.jpg
thumbnailImagePosition: right
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-away.jpg
categories:
- blog
tags:
- hugo
- github
- git
---

GitHub is not only a cool community for developers who can create teams, share code, and even make friends, but it also provides free tools for people who like to toss around ideas. <!--more-->[GitHub Pages](https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/about-github-pages#about-github-pages) is such a fancy service where you can easily setup your own websites by just a few mouse clicks. Companies and project teams can easily create product homepages on it, and individuals who like to document things and share ideas can also easily build blogs on it.



Table of contents:

- [Prerequisites](#prerequisites)
- [Install Hugo](#install-hugo)
- [Install blog themes](#install-blog-themes)
  - [Install themes with Git](#install-themes-with-git)
  - [Modify config.toml](#modify-configtoml)
- [Write your first blog post](#write-your-first-blog-post)
  - [images for posts](#images-for-posts)
- [Deploy Hugo as a Github Pages](#deploy-hugo-as-a-github-pages)
  - [Prepare two repos](#prepare-two-repos)
  - [Upload repos](#upload-repos)
    - [Blog repo](#blog-repo)
    - [Hugo site repo](#hugo-site-repo)
  - [Deploy script](#deploy-script)
  - [Switch to another computer](#switch-to-another-computer)
- [Conclusion](#conclusion)

There are tons of different ways to setup blogs on GitHub Pages. The most straightforward one is to create by Jekyll which is a static site generator with built-in support for GitHub Pages and a simplified build process. Various [Jekyll](https://github.com/search?q=jekyll) blog frameworks have been provided for you to build a blog in just minutes.

[Hexo](https://hexo.io) is another blog framework based on Node.js, which is much more faster and more powerful with hundreds of blog themes. But it's rather tricky to set up.

I prefer [Hugo](https://gohugo.io), the world's fastest framework for building websites, which is written in Go (aka Golang). If you know how to create a blog repository on GitHub, you can build a Hugo blog in minutes! Here are the steps.

## Prerequisites

Please make sure that you have a [GitHub](https://github.com) account, othervise you should sign up one. And then finish the following tasks:

- [Install Git on Mac](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Connecting Github with SSH](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh)
- [Install Homebrew on Mac](https://brew.sh)

## Install Hugo

Now you have all your need to setup a Hugo blog.

```sh
> brew install hugo
# check if done
> hugo version
Hugo Static Site Generator v0.75.1/extended darwin/amd64 BuildDate: unknown
```

Change directory to the one you intend to keep your site files, then run:

```sh
> hugo new site my-blog
> cd my-blog
> git init

# show tree structure of my-blog, you may need 'brew install tree'
> tree my-blog
my-blog
├── archetypes
│   └── default.md
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes

6 directories, 2 files
```

`themes` is the directory for blog themes. You can browse [Hugo Themes](https://themes.gohugo.io) to choose your favorite one.

## Install blog themes

On [Hugo Themes](https://themes.gohugo.io), you can download the theme you like and then put it in the `themes` folder and edit `config.toml` to add `theme=<theme_name>`. However, there is an easier way to do this.

### Install themes with Git

Find the Github repo page of the theme you choose and just clone the repo into the `themes` folder. For example, I found [hugo-theme-even](https://github.com/olOwOlo/hugo-theme-even) to be my type of theme and easy to use.

So I just clone the repo to `themes` by

```sh
git clone https://github.com/olOwOlo/hugo-theme-even.git themes/even
```

One thing to note here, later when upload the `my-blog` site to Github repo (maybe private repo to backup this site files), you might encounter some warnings related to the theme repo since you clone another repo in an existing git repo. You should decide whether or not this theme repo (in my case, the `hugo-theme-even` in themes folder) should be tracked to the original repo on Github, which you should not since you just clone the repo from others. In that case you need to clear index for the theme repo and keep it just tracked in your `my-blog` private repo.

Here's the question that arises, what should I do if I need to keep this theme updated with the original repo. The solution is to use `git submodule`, a powerful feature of Git, which can keep the parent repo and its summodule repos connected and isolated. Instead of `git clone` you install themes by `git submodule`:

```sh
git submodule add https://github.com/olOwOlo/hugo-theme-even.git themes/even
```

### Modify config.toml

Replace `config.toml` with the example config file provided by the theme `themes/even/exampleSite/config.toml` and modify necessary configs. Here are some examples.

```toml
baseURL = "https://<GITHUB_USERNAME>.github.io"
title = "XXXXXX"
googleAnalytics = "UA-XXXXXXXX-X" # for Google Analytics

# show 'xx Posts In Total' in archive page?
showArchiveCount = true

# mathjax
mathjax = true  # see https://www.mathjax.org/ 
mathjaxEnableSingleDollar = true  # use $...$ to render latex
mathjaxEnableAutoNumber = false   # auto number for formula

[params.busuanzi] # count web traffic by busuanzi 
  enable = true
  siteUV = true
  sitePV = true
  pagePV = true
```

## Write your first blog post

Writing blog posts is much more easier.

```sh
> hugo new posts/my-first-post.md
```

This will add the file `my-first-post.md` under the folder `content/posts`. You can also manually create new markdown file under that folder. In `my-first-post.md`, add:

```md
---
title: "My First Post"
date: 2020-11-06T23:19:48+08:00
draft: false
keywords: ["Hugo"]
description: "A 30 minutes guide"
tags: ["hugo"]
categories: ["howto"]
---

This is content. XXXX
```

Next preview your blog site (make sure your site works):

```sh
> hugo server -D
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

### images for posts

You may have pictures for your blog posts. Just put them in `my-blog/static/img/` and refer to them via

```md
![IMG_NAME](/img/img_example.png)
```

## Deploy Hugo as a Github Pages

Follow the instructions on [Hugo Docs](https://gohugo.io/hosting-and-deployment/hosting-on-github/#github-user-or-organization-pages) to deploy `my-blog` to a Github private repo and deploy the `public` folder which is automatically generated by `hugo` command onto a Github Pages repo. Here are the main steps.

### Prepare two repos

- One private repo (e.g. blog) for `my-blog`, the local Hugo website content and other source files.
- Another public repo, should be named `<GITHUB_USERNAME>.github.io`, for the fully rendered version of your Hugo website, i.e., the published blog (in `my-blog/public`)

### Upload repos

#### Blog repo

Make sure your blog website works locally (via `hugo server -D` as above) and then press `Ctrl+C` to kill the server.

Before we go any further, run

```sh
rm -rf public
```

to completely remove the public directory (if any).

Make sure your `<GITHUB_USERNAME>.github.io` repo is not empty (better add a README or .gitignore first) and then add it as a submodule into a directory named as `public` in your current site repo.

```sh
git submodule add -b master https://github.com/<GITHUB_USERNAME>/<GITHUB_USERNAME>.github.io.git public
```

Now run the command

```sh
hugo
```

to build your site to `public`. To push the `public` to your Github Pages repo, namely `<GITHUB_USERNAME>.github.io`, you should do these:

```sh
cd public
git add .
git commit -m 'building blog site'
git push origin master
```

If you receive an error message, you might need change `master` to `main` because GitHub have been using `main` instead of `master` since October 1, 2020.

#### Hugo site repo

Run the following commands:

```sh
git add .
git commit -m 'upload blog site source files'
git remote add origin git@github.com:<GITHUB_USERNAME>/blog.git
git push -u origin master
```

Again `master` should be replaced by `main` if you just create a new repository on GitHub where the branch name have been set `main` as default. Now that you have `master` branch in the local, branch renaming is needed. Just after `git commit` in the above commands, run

```sh
git branch -M main
```

to change your local branch name to `main` and then run `git remote` and `git push` commands.

If all goes well, you have two submodules in your site repo. `.gitmodules` shows like this:

```conf
[submodule "themes/even"]
  path = themes/even
  url = https://github.com/olOwOlo/hugo-theme-even.git
[submodule "public"]
  path = public
  url = git@github.com:<GITHUB_USERNAME>/<GITHUB_USERNAME>.github.io.git
```

Run `git push` command will not upload any file of these two submodules to your site upstream repo `blog` since submodules have their own upstream repo.

### Deploy script

By now you have accomplished the full steps to make a blog. You should access the blog in your browser with the url `https://<GITHUB_USERNAME>.github.io`.

However, let's have a recap. To publish a new post and keep your github repo up-to-date, your should have four steps:

1. Create new post file (*.md in /content/post/)
2. Build blog (`hugo` command)
3. Upload `public` (blog repo)
4. Upload `my-blog` (site repo)

Step 2 & 3 can be automated by a script, `deploy.sh` in `my-blog` directory.

```sh
#!/bin/sh

# If a command fails then the deploy stops
set -e

printf "\033[0;32mDeploying updates to GitHub...\033[0m\n"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public

# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site $(date)"
if [ -n "$*" ]; then
  msg="$*"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master
```

Change `master` to `main` in the last line if you encounter "*error: src refspec master does not match any*".

Now whenever you want to put something new in your blog, follow the three steps:

1. Create new post file (*.md in /content/posts/)
2. Run `zsh deploy.sh` or `./deploy.sh` in `my-blog` directory
3. Upload `my-blog` (site repo)

### Switch to another computer

You may ask: *what if I change computers?*

Not a big problem. All you have to do in your new computer is the first two parts of this post and to git clone your site repository `my-blog` from Github.

- Install Git
- Connecting Github with SSH
- Install Hugo
- Git clone repository

You'd better get all submodules repositories files. To do this, pass `--recurse-submodules` to the `git clone` command, it will automatically initialize and update each submodule in the repository, including nested submodules if any of the submodules in the repository have submodules themselves. (Ref: [Git - Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules))

```sh
git clone --recurse-submodules git@github.com:<GITHUB_USERNAME>/blog.git
```

## Conclusion

Setting up a blog with Hugo is easy and simple, and this post lays out the concise and complete steps, with just a few actions on Git that may take time to delve into the specifics.

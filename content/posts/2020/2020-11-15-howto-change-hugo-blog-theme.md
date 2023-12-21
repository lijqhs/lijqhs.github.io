---
title: "How To Change The Theme of A Hugo Blog"
date: 2020-11-15T23:30:00+08:00
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/beach-1866992_1920.jpg
thumbnailImagePosition: right
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/beach-1866992_1920.jpg
categories:
- blog
tags:
- hugo
- github
- git
---


A few days ago I [migrated this blog]({{< ref "/posts/2020/2020-11-06-howto-setup-blog-with-hugo-github-pages.md" >}}) to Hugo from Jekyll and employed a beautiful theme called [Even](https://github.com/olOwOlo/hugo-theme-even), a concise and powerful hugo theme, which fascinated me so much until I came across another graceful and exquisite hugo theme called [Tranquilpeak](https://github.com/kakawait/hugo-tranquilpeak-theme)<!--more-->, brought by [Thibaud Lepretre](https://github.com/kakawait) from the [original hexo theme](https://github.com/LouisBarranqueiro/hexo-theme-tranquilpeak).

So I decided to change the theme of this blog and I took the following steps to accomplished that, which was very easy and worth a share here.

{{< toc >}}

## Add another theme as a submodule

As previously mentioned in the [blog setup post]({{< ref "/posts/2020/2020-11-06-howto-setup-blog-with-hugo-github-pages.md" >}}), you can use command `git clone` or `git submodule` to add a blog theme from GitHub repository. It is highly recommended here that you always use `git submodule` to add a new theme, even though you want to customize the theme template by yourself, in which case you could first fork the theme repository and add submodule from your replica repo of the theme which should track all the modifications by you and can also keep updated with the original repo when necessary.

So just in case you want to make some changes someday, or just don't want to get the latest version from the original repo of which some newest fancy features may not be what you want.

You can fork a copy of `https://github.com/kakawait/hugo-tranquilpeak-theme.git` theme to your repository `https://github.com/<GITHUB_USERNAME>/hugo-tranquilpeak-theme.git` and then add submodule from the latter. Or you can simply add the orginal theme repo to a submodule by:

```bash
git submodule add https://github.com/kakawait/hugo-tranquilpeak-theme.git themes/tranquilpeak
```

## Change the blog settings

Backup `config.toml` in the root directory of your blog site to `config.toml_bak` and copy `config.toml` from `themes/tranquilpeak/exampleSite` to the root directory and do some configuration as the [user docs](https://github.com/kakawait/hugo-tranquilpeak-theme/blob/master/docs/user.md) tells you, in which theme should be set to `tranquilpeak`:

```conf
theme = "tranquilpeak"
```

After all settings are done, you can rebuild the blog now with the simple command `hugo` in the root directory or `hugo server -D` to preview your blog first. And then in the public directory you can commit all the changes and push up to the GitHub repo and a new blog is ready.

## Delete the submodule

Since you do not need the old theme anymore, you can delete it from you repository. 

*How to remove the old submodule?*

In the blog root directory, run the following commands to remove the theme even from your blog site:

```bash
git submodule deinit theme/even
git rm theme/even 
```

`git submodule deinit theme/even` deletes theme/even's entry from .git/config. This excludes `theme/even` from `git submodule update`, `git submodule sync` and `git submodule foreach` calls and deletes its local content (source). Also, **this will not be shown as change in your parent repository**. `git submodule init` and `git submodule update` will **restore** the submodule, again without commitable changes in your parent repository.

`git rm theme/even` will remove theme/even from the work tree and the `theme/even` directory. The files of `theme/even` directory will be gone as well as the submodules' entry in the `.gitmodules` file, that is, the following lines will disappear.

```conf
[submodule "themes/even"]
        path = themes/even
        url = https://github.com/olOwOlo/hugo-theme-even.git
```

## Reference

- [Git submodule](https://git-scm.com/docs/git-submodule)
- [Removing a submodule](https://riptutorial.com/git/example/2652/removing-a-submodule)

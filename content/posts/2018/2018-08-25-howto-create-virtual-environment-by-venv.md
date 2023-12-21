---
title: "How To Create Virtual Environments by venv"
date: 2018-08-25T20:11:48+08:00
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-worker.jpg
thumbnailImagePosition: left
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-worker.jpg
categories:
- dev
tags:
- python
- matplotlib
- mac
---


When I tested examples of the book **Deep learning from scratch**, a RuntimeError prompted told me matplotlib backend was not able to function correctly as my Python is not installed as a framework. <!--more-->The error message guided me to the Matplotlib FAQ ([Working with Matplotlib on OSX — Matplotlib 2.2.3 documentation](https://matplotlib.org/faq/osx_framework.html)).

The page shows `virtualenv` is to blame as the virtualenv creates a non framework build even if created from a framework build of Python. To solve the problem the best and the simplest way is to use the build-in virtual environment tool `venv` provided since Python 3.4.

{{< toc >}}

## Remove virtualenv

The previous [blog post]({{< ref "/posts/2018/2018-08-24-howto-install-python3-virtualenv-on-macbook.md" >}}) introduces how to depoly clean python working environments by `virtualenv`. Now because of the above problem I encountered with virtualenv I decided to abandon virtualenv in my MacBook. Instead `venv` might be a good replacement to create virtual environment. The shortage is there is no `workon` shortcut any more to switch between virtualenv seamlessly.
The restore steps are clearly. Remove settings related to virtualenv and virtualenvwrapper in `~/.profile`. After that my `.profile` looks just like this.

```bash
export CLICOLOR=1
export LSCOLORS=GxFxCxDxBxegedabagaced

# Setting PATH for Python 3.6
# The original version is saved in .profile.pysave
PATH=“/Library/Frameworks/Python.framework/Versions/3.6/bin:${PATH}”
export PATH

# set pip available only in virtual environment
export PIP_REQUIRE_VIRTUALENV=true

# create commands to override pip restriction.
# use `gpip` or `gpip3` to force installation of
# a package in the global python environment
gpip(){
   PIP_REQUIRE_VIRTUALENV="" pip "$@"
}
gpip3(){
   PIP_REQUIRE_VIRTUALENV="" pip3 "$@"
}

# set shortcut for activating venv
alias dl-env='source ~/Venv/dl-env/bin/activate'
alias code='cd ~/Documents/Code'
```

Then I uninstalled virtualenv and virtualenvwrapper which I should never need. And then just remove `.virtualenvs` directories used by virtualenv.
You can see `PIP_REQUIRE_VIRTUALENV` and gpip is still kept in the .profile file. That was meant to prevent myself mess up global python package list when using `pip install` or `pip3 install` which are now limited to take effect only in the virtual environment. To install packages in the global environment the command `gpip` or `gpip3` should be explicitly appointed. *Reference:*[Further Configuration of Pip and Virtualenv — The Hitchhiker’s Guide to Python](https://docs.python-guide.org/dev/pip-virtualenv/)

## Create virtual environments by venv

I created a new `dl-env` by venv in a different directory `Venv` where all virtual environments are settled down. Actually I could also use directory like `.virtualenvs` to make virtual environments folders containing installed packages invisible but I didn’t do that. Project files are now put in the `Code` sub-directory of `Documents` which I could navigate directly from Finder on my MacBook.

```bash
cd ~/Venv
python3 -m venv dl-env
```

In the settings in `.profile` I set two alias: `dl-env` for quick activation of virtual environment *dl-env* and `code` for frequently used command `cd ~/Documents/Code`. *Reference:*[macos - How do I create a terminal shortcut to this path? - Super User](https://superuser.com/questions/606308/how-do-i-create-a-terminal-shortcut-to-this-path)

## Update Python Path in VSCode

Python Path should be updated by new virtual environment python interpreter path.

```json
"python.pythonPath": "~/Venv/dl-env/bin/python",
```

## Run example in new virtual environment

After all those settings above I could run example again in the new virtual environment. First of all some packages like `numpy` `scipy` `matplotlib` are required.

```bash
$ dl-env # which activate venv dl-env
(dl-env) $ pip3 install numpy scipy matplotlib
(dl-env) $ python example-code.py
```

The RuntimeError never shows up and the result is exactly the same as the book says.
![gradient-2d](https://cdn.jsdelivr.net/gh/lijqhs/cdn@1.0/img/post/gradient-2d.png)
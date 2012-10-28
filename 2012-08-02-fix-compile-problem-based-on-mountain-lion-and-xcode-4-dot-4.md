---
layout: post
title: "Fix Compile Problem Based on Mountain Lion and Xcode 4.4"
date: 2012-08-02 11:36
meta: true
comments: true
categories:
- Mountain Lion
- Xcode
---

OS X Mountain Lion 作为一个比较成功的操作系统，短短几天下载量久已经突破了400万。
但是安装Xcode 4.4后，我们需要做一些额外的事情，才能使用Homebrew正常的安装一些packages。

{% img left half http://images.apple.com/osx/images/overview_hero1_title.png Mountain Lion %}

*安装Xcode + Command Lion Tools*

Xcode 4.4直接可以从 Mac app store上下载，大概1个多G，换成V2EX的DNS后下载速度会变得很快，大家可以尝试一下。
安装完成后打开Xcode，点击菜单 Xcode -> Preferences -> Downloads 然后找到 "Command Line Tools" 进行安装，
安装完成后关闭Xcode并打开终端。


*修复Homebrew和 GCC 4.2*
Mountain Lion安装完成后系统会自动将 `/usr/local`转成root权限， 所以我们需要在终端命令行中将权限做一个调整：

    $ sudo chown -R `whoami` /usr/local

接下来更新下 Homebrew:

    $ brew update

<!--more-->

如果你需要安装旧版本的ruby, 比如ruby 1.9.2, 1.8.7 或者REE, 你需要先安装 GCC 4.2.
Apple在在 Command Line Tools中并没有集成 GCC 4.2，所以我们需要通过Homebrew自行安装 GCC 4.2
    
    $ brew tap homebrew/dupes
    $ brew install apple-gcc42

GCC 4.2安装完成后， 编译过程中可能会出现gcc 4.2路径丢失的问题，如下

    compiling generator.c
    make: /usr/bin/gcc-4.2: No such file or directory
    make: *** [generator.o] Error 1

我们只要重新做个软链即可

    $ sudo ln -s /usr/local/bin/gcc-4.2 /usr/bin/gcc-4.2

*重新安装 X11*

如果你还在使用一些依赖于X11的应用，必须得再安装 X11，Apple已经在系统中移除了 X11的支持，但是我们还是可以直接从 [XQuartz][1] 安装，
我使用的版本是 2.7.2 release，下载完双击安装即可。


[1]: http://xquartz.macosforge.org/landing


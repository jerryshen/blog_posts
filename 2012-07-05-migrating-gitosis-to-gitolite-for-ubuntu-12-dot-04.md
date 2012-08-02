---
layout: post
title: "从 Gitosis 迁移到 Gitolite"
date: 2012-07-05 11:47
meta: true
comments: true
categories:
- git
- gitolite
- gitosis
---

之前Linode有个回馈用户的活动，新用户注册并购买linode可以得到100美金的虚拟货币，老用户添加一台新的linode可以免费使用3个月，手痒痒就添加了一台，并安装了ubuntu server最新的12.04，却发现Gitosis已经木有人维护了，也找不到相应的安装源了，只好舍弃，换到Gitolite，但是之前gitosis下的项目如何处理呢？答案当然是肯定的，
可以无缝迁移到新的gitolite上，下面来讲解下如何做这个迁移。

假设Gitosis在服务器A上， 我们要在服务器B安装Gitolite并且要把服务器A上的项目迁移到服务器B上。

首先需要安装Git和Gitolite

    $ sudo apt-get install git-core
    $ sudo apt-get install gitolite

安装完成后，接下来给gitolite创建一个用户

    sudo adduser \
        --system \
        --shell /bin/bash \
        --gecos 'git version control' \
        --group \
        --disabled-password \
        --home /home/git \
        git


<!--more-->

用户添加完成后需要切饿换到添加好的git用户上去，设置path变量，然后cd到git用户根目录下

    sudo su - git
    echo "PATH=$HOME/bin:$PATH" > ~/.bashrc
    cd

这个时候需要将你的本机的`ssh public key`，默认情况下文件名应为`id_rsa.pub`的文件拷贝到git用户根目录下，这里就不多说了使用scp即可，但是需要注意权限问题。
如果存在权限问题，可以通过`chown`命令来修改权限以便可以让git用户安全访问，需要在有权限的用户下进行。

    $ sudo chown git.git id_rsa.pub

假设在`/home/git/`目录下已经存在文件`id_rsa.pub`, 我们先将这个文件名改成以你名字命名的文件， 比如

    $ mv id_rsa.pub jerry.pub

然后在git用户的权限下运行以下脚本

    $ gl-setup jerry.pub

这个脚本会将会安装并初始化你的gitolite结构，安装过程中会弹出一个编辑器，需要做一些小修改

    # 将 $REPO_UMASK = 0077; 修改成 $REPO_UMASK = 0027;

执行完后将会生成以下的结构

    creating gitolite-admin...
    Initialized empty Git repository in /home/git/repositories/gitolite-admin.git/
    creating testing...
    Initialized empty Git repository in /home/git/repositories/testing.git/
    [master (root-commit) 94864dc] start
     2 files changed, 6 insertions(+)
     create mode 100644 conf/gitolite.conf
     create mode 100644 keydir/jerry.pub

回到本机，就可以将该gitolite的管理项目clone下来了

    $ git clone git@<server>:gitolite-admin.git

说回来，因为之前已经存在一个gitosis的admin项目，在这里需要做的其实很简单，将gitosis-admin中keydir目录内的所有成员的key拷贝到
'gitolite-admin/keydir'目录下即可。conf配置文件中可以加入之前旧项目的配置，[这里是配置文件的语法格式][1]，
还有[如何添加和移除用户][2]

然后去gitosis服务器，将repositories整个目录中除了gitosis-admin.git这个项目外都打包，并`scp`到gitolite服务器的`/home/git/`下
和gitolite已存在的目录进行合并， 这里也需要注意目录权限问题。

完成之后回到本机项目目录修改一下`.git/config`文件， 将url改成 `git@<server>:project.git`即可



[1]: http://sitaramc.github.com/gitolite/repos.html
[2]: http://sitaramc.github.com/gitolite/users.html





---
layout: post
title: "hosting your blog on linode using octopress"
date: 2012-06-23 12:27
meta: true
comments: true
categories:
- octopress
- linode
---

已经很长一段时间没有好好打理博客了，之前用Jekyll自己搭了个博客，想迁移到Octopress上，却一直因为一些事情所耽搁，趁端午期间，终于把这件拖了很久的事情给办了。

Octopress其实也是基于Jekyll的，只是不需要自己来写那么多扩展了，而且目前也有越来越多的插件支持Octopress。下面我来讲以下如何在linode上搭建你的Octopress博客。

我fork了octopress，[https://github.com/jerryshen/octopress](https://github.com/jerryshen/octopress)，大家可以直接clone到本地。

先来拆分下结构

1. [.rvmrc](https://github.com/jerryshen/octopress/blob/master/.rvmrc)：这个决定于你的机器用的是什么版本的ruby，但Octopress最低要求是1.9.2，我用的是1.9.3。
2. [_config.yml](https://github.com/jerryshen/octopress/blob/master/_config.yml)：这个文件里面都是一些可定制的配置信息，包括域名、博客名称还有第三方的用户名等等。
3. [.themes/blog](https://github.com/jerryshen/octopress/tree/master/.themes/blog)：这个是我借用了eDoctor的样式库并做了一些扩展。

<!--more-->

### 第一步：将我fork的Octopress安装到本地

前提是你的机器需要有ruby环境，这点就不多说了，git也是必须的。

    $ git clone git://github.com/jerryshen/octopress.git

除非世界末日，不然你将看到以下输出：

    Initialized empty Git repository in /PATH-TO-HOME/octopress/.git/
    remote: Counting objects: 95, done.
    remote: Compressing objects: 100% (83/83), done.
    remote: Total 95 (delta 4), reused 95 (delta 4)
    Receiving objects: 100% (95/95), 190.60 KiB | 76 KiB/s, done.
    Resolving deltas: 100% (4/4), done.

然后进入octopress目录安装相应的gem

    $ cd octopress
    $ bundle

接下来通过submodule安装eDoctor前端团队提供的主题样式

    $ git submodule init
    $ git submodule update

你会看到以下输出

    Initialized empty Git repository in /PATH-TO-HOME/octopress/.themes/blog/sass/_base/.git/
    remote: Counting objects: 35, done.
    remote: Compressing objects: 100% (23/23), done.
    remote: Total 35 (delta 17), reused 29 (delta 11)
    Receiving objects: 100% (35/35), 8.39 KiB, done.
    Resolving deltas: 100% (17/17), done.
    Submodule path '.themes/blog/sass/_base': checked out '141373944588b52f93ab10903001235d195a110a'

接下来就能安装博客了，指令将会拷贝`.themes/blog`下面的目录到根目录下

    $ rake install['blog']

以下信息的输出将会证明你所做的完全没有问题

    ## Copying blog theme into ./source and ./sass
    mkdir -p source
    cp -r .themes/blog/source/. source
    mkdir -p sass
    cp -r .themes/blog/sass/. sass
    mkdir -p source/_posts
    mkdir -p public

### Step2：配置你的博客信息

打开`_config.yml`文件，里面包含了博客的所有配置项，修改成你个人的配置即可，这里就不做过多说明了。

### Step3：创建一篇新文章

在本地使用markdown来写博客，这无疑是非常hack的方式。

    $ rake new_post['Hello World']

以上的rake指令会在`_posts`目录下生成一个markdown文件，用编辑器打开就能开始写文章了。写好后执行以下指令打开服务并浏览你的博客：

    $ rake generate
    $ rake preview

访问`http://localhost:4000`就能访问你的博客并且能看到你写的文章了，`rake generate`指令会将你的markdown格式的文章转成html格式。

### Step4：部署你的博客

我比较推荐在linode上部署你的博客，当然github本身也是可以搭建octopress的博客的，这个决定权在你。服务器上需要已经部署好web环境，可以参考我以前写的文章 [http://hansay.com/2011/12/03/deployment-for-ubuntu-server-with-rvm-nginx-passenger-and-postgresql-for-ruby-on-rails/](http://hansay.com/2011/12/03/deployment-for-ubuntu-server-with-rvm-nginx-passenger-and-postgresql-for-ruby-on-rails/)。

我们先来说下原理，发布博客到linode服务器上，其实就是把generate出来的public目录丢到服务器，也就是说服务器上只是用nginx将一个静态博客跑起来罢了。

修改`Rakefile`来使用`rsync`的方式发布博客，前提是需要将你的`public key`添加到服务器的`authorized_keys`中来避免输入密码的麻烦。

    ## -- Rsync Deploy config -- ##
    # Be sure your public key is listed in your server's ~/.ssh/authorized_keys file
    ssh_user       = "[YOUR USERNAME]@[YOUR DOMAIN]"
    ssh_port       = "22"
    document_root  = "~/PATH-TO-YOUR-PROJECT/"
    rsync_delete   = true
    deploy_default = "rsync"

保存完了后执行：

    $ rake deploy

修改nginx配置，这里我就不多说了，查下nginx配置文档即可，好好享受你的博客吧。











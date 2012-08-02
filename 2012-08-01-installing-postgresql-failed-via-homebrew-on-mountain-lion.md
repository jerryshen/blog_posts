---
layout: post
title: "Installing Postgresql Failed Via Homebrew on Mountain Lion"
date: 2012-08-01 10:31
meta: true
comments: true
categories:
- Mountain Lion
- Postgresql
---


Mountain Lion 出了之后确实有点小激动，当天就买下安装，新加的一些功能也比较喜欢，比如Message Center，
是历来几个版本中较好的一个。

{% img left half http://media.tumblr.com/tumblr_m7p8rzVYgu1qcfyao.jpg Mountain Lion %}

不过在安装postgresql的时候遇到了一些麻烦，安装老是失败

    Error: Failed executing: make install-world (postgresql.rb:67)

经过一系列的研究之后得出一下解决方案

    $ brew install postgres --without-ossp-uuid

Mountain Lion上的安装需要跳过 `ossp-uuid` 这个步骤,否则将无法正常安装. 安装完成后需要初始化一下数据库

    $ initdb /usr/local/var/postgres -Upostgres

这里我们用默认的postgres来初始化数据库，这步可能会碰到以下问题

    creating template1 database in /usr/local/var/postgres/base/1 ... FATAL:  could not create shared memory segment: Cannot allocate memory
    DETAIL:  Failed system call was shmget(key=1, size=2138112, 03600).
    HINT:  This error usually means that PostgreSQL's request for a shared memory segment exceeded available memory or swap space, or exceeded your kernel's SHMALL parameter.  You can either reduce the request size or reconfigure the kernel with larger SHMALL.  To reduce the request size (currently 2138112 bytes), reduce PostgreSQL's shared memory usage, perhaps by reducing shared_buffers or max_connections.

解决的方法是

    $ sudo sysctl -w kern.sysv.shmall=65536
    $ sudo sysctl -w kern.sysv.shmmax=16777216

或者直接加入到配置文件 `/etc/sysctl.conf`

    kern.sysv.shmall=65536
    kern.sysv.shmmax=16777216

接下来执行 `initdb /usr/local/var/postgres -Upostgres` 就没有问题了
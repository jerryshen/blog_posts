---
layout: post
title: "Integration of ActiveAdmin"
date: 2012-10-28 22:59
meta: true
comments: true
categories:
- activeadmin
- administration
---

如果你要做一个中小型的项目，你会觉得写一个后台是一个非常麻烦的事情，有以下几点

* Admin UI设计与布局
* 很多的重复工作，基本上都是CRUD
* 每换一个项目，都得重新做一次。。。

{% img left half http://activeadmin.info/images/activeadmin.png Active Admin %}

Active Admin 是一个后台界面框架，它能使你的后台搭建变得非常方便，当然界面也相当漂亮。

使用相当简单，按照[文档][1]就能很轻松的搞定，但是一些自定义的功能大家未必会深入去看，所以在这里我主要讲一些可
自定义的功能。[未完。。。待续]

<!--more-->

**一. Active Admin 多余的数据结构**

Active Admin会生成一些对一些项目本身无用的数据表，比如 `admin_users`, `comments` 等，
因为在一些项目中我们并不需要一个单独的表来存储管理员信息，一般都是和普通用户存一起，然后用`is_admin`字段来区分。

按照我写的需求，首先要把active admin生成出来的所有migrations通通手动删除，
然后给Active Admin作如下的修改:
  
{% codeblock lang:ruby app/controllers/application_controller.rb %}
class ApplicationController < ActionController::Base
  protect_from_forgery

  # 作为权限验证，提供Active Admin来调用
  def authenticate_active_admin_user!
    authenticate_user!

    redirect_to root_path, alert: 'Unauthorized Access!' \
      unless current_user.is_admin?
  end
end
{% endcodeblock %}

{% codeblock lang:ruby config/initializers/active_admin.rb %}
ActiveAdmin.setup do |config|
  # 这里的方法就是 application coontroller中自定义的权限验证方法
  config.authentication_method = :authenticate_active_admin_user!

  # 既然我们不用Active Admin自带的 admin user，
  # 那么当前用户的方法也应当改为默认的 `:current_user`
  config.current_user_method = :current_user

  # 同样的道理，登出路由也需要做相应改动
  config.logout_link_path = :destroy_user_session_path

  # Devise默认的登出方式是`delete`
  config.logout_link_method = :delete

  # 感觉这个功能有点多余，干脆直接关掉。。。
  config.allow_comments = false
end
{% endcodeblock %}

同时将 Active Admin 插件中的中文i18n文件拷贝到项目中，即可完成项目的国际化自定义。


[1]: http://activeadmin.info/documentation.html


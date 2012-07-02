---
layout: post
title: "让你的分享过程变得简单"
date: 2012-06-29 11:24
meta: true
comments: true
categories:
- ruby
- social network
---

社交网络的流行确实带动了很多社交应用的崛起，不论是国外这么好的大环境下还是在国内这种境况中，社交类型的网站遍地开花，数不胜数，就连传统行业的网站或者应用也都通通加入了社交这个圈子，可以想象，社交带来了一个质和量的双重飞跃。

前段时间写了个Rails小插件，方便一键分享，默认加入了新浪微博、腾讯微博、人人和豆瓣这些国内主流的社交网站，用户只需要调用一个helper就能把链接渲染到页面上，从而简化了你的分享流程，无需自己再写任何代码。项目地址是 [https://github.com/jerryshen/social_share_hub](https://github.com/jerryshen/social_share_hub)

{% img left half https://github.com/jerryshen/social_share_hub/raw/master/screenshorts/social_share_hub.png Social Share Hub! %}

其实这个插件很简单，所以只简单介绍下如何使用，当然你们也可以自己来扩展其他的社交网站

<!--more-->

### 插件的安装与配置

在`Gemfile`中加入gem:

{% codeblock lang:ruby %}
gem 'social_share_hub', '~> 0.0.3'
{% endcodeblock %}

安装并初始化插件的配置文件：

	$ bundle
	$ rails g social_share_hub:install

找到`config/initializers/social_share_hub.rb`，会有以下默认配置：

{% codeblock lang:ruby %}
SocialShareHub.configure do |config|
  config.allowed_sites = %w(weibo douban qq renren)
end
{% endcodeblock %}

默认是新浪微博、豆瓣、腾讯微博和人人，如果你不需要其中一个或者几个请自行删除。

### 如何使用

首先需要将相应的js和css文件引用到assets文件中：

{% codeblock lang:javascript app/assets/javascripts/application.js %}
//= require social_share_hub
{% endcodeblock %}

{% codeblock lang:css app/assets/stylesheets/application.css %}
*= require social_share_hub
{% endcodeblock %}

接下来通过一个helper就能把设置好的分享链接渲染到页面上了：

{% codeblock lang:ruby %}
<%= social_share_link("这里填写你要分享的文本", "图片绝对路径") %>
{% endcodeblock %}

很小的插件，但是可以让分享流程变得更简单。
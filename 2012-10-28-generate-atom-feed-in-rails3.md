---
layout: post
title: "Generate ATOM Feed in Rails3"
date: 2012-10-28 11:40
meta: true
comments: true
categories:
- Atom
---

{% img left half http://dribbble.s3.amazonaws.com/users/4598/screenshots/320237/excitedatom.jpg ATOM %}

个人博客中，经常会用到RSS订阅的功能，但如今，我比较推荐ATOM来完成这个功能，完全可以取代RSS.
因为ATOM的规范比RSS更加好，几乎每个feed reader都支持。

下面我来完成一个ATOM Feed的实例。我们的项目中需要有以下准备:

* `Post`类, 并且拥有以下属性
  * title
  * content
  * author
* `posts_controller.rb`, 我们将要在这个controller中加入有关feed的方法

我们将会用到 Ruby on Rails 自带的 ATOM Builder来实现这个功能，`atom_feed`非常简单，并且相当好用。

<!--more-->

1. 给 posts controller 添加 feed action
{% codeblock lang:ruby app/controllers/posts_controller.rb %}
def feed
  # this will be the name of the feed displayed on the feed reader
  @title = "FEED title"

  # the post entries
  @posts = Post.order("updated_at desc")

  # this will be our Feed's update timestamp
  @updated = @posts.first.updated_at unless @posts.empty?

  respond_to do |format|
    format.atom { render layout: false }

    # we want the RSS feed to redirect permanently to the ATOM feed
    format.rss { redirect_to feed_path(format: :atom), status: :moved_permanently }
  end
end
{% endcodeblock %}

2. 创建ATOM模板

{% codeblock lang:ruby app/views/posts/feed.atom.builder %}
atom_feed language: 'zh-CN' do |feed|
  feed.title @title
  feed.updated @updated

  @posts.each do |post|
    next if post.updated_at.blank?

    feed.entry( post ) do |entry|
      entry.url news_item_url(post)
      entry.title post.title
      entry.content post.content, type: 'html'

      # the strftime is needed to work with Google Reader.
      entry.updated(post.updated_at.strftime("%Y-%m-%dT%H:%M:%SZ")) 

      entry.author do |author|
        author.name entry.author)
      end
    end
  end
end
{% endcodeblock %}

3. 定义路由

我们可以将路由定义成这样 `http://www.example.com/feed`

{% codeblock lang:ruby config/routes.rb %}
match '/feed' => 'posts#feed',
      :as => :feed,
      :defaults => { :format => 'atom' }
{% endcodeblock %}

4. 给Layout添加feed链接

代码需要加在 `head`标签间

{% codeblock lang:ruby app/views/layouts/application.html.haml %}
  = auto_discovery_link_tag :atom, "/feed"
  = auto_discovery_link_tag :rss, "/feed.rss"
{% endcodeblock %}


你看，就是这么简单。
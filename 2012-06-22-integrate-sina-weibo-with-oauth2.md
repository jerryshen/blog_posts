---
layout: post
title: "利用OAuth2协议将网站接入到新浪微博"
date: 2012-06-22 14:35
meta: true
comments: true
categories:
- weibo
- OAuth2
- Devise

---

前段时间收到了新浪微博客开放平台发来的邮件，宣布要在9月份停止使用OAuth1协议，这点让我感到意外，以下是邮件原文：

{% blockquote 新浪微博开放平台敬上 %}
尊敬的开发者您好：
新浪微博开放平台预计2012年九月份停止旧版接口和OAuth1.0的使用，请尽快将您的应用迁移至新版接口和OAuth2.0。
新版接口更高效，包含更丰富的功能，为帮助您迁移，请参考以下链接：
...
希望您能够做好相关准备，技术问题可咨询 @微博API，由此造成诸多不便，请开发者给予谅解。
{% endblockquote %}

这个9月份的截止时间可谓是一记重磅炸弹，让所有还停留在Oauth1上的网站面临不得不面临升级的尴尬。

之前花了点时间把新浪微博的一整套流程都给研究清楚了，其实时间主要都花在研究他的审核机制了，OAuth2调用相对来说还是相当简单的。下面我来讲一下如何使用OAuth2做授权。

<!--more-->

开发之前其实需要做一些准备，比如创建新浪微博开放平台的帐号，其实也就是微博帐号，登录后访问[http://open.weibo.com](http://open.weibo.com/)，点击"网站接入"，会引导进入添加新网站表单，这里有一点需要注意，目前来说新浪微博的网站接入得验证域名的所有权了，所以必须得输入正确的域名而且要在网站首页添加相应的meta标签才能通过。完成之后可以不用立即提交审核，因为这个步骤需要提供备案密码，没有备案是通过不了的，但是验证域名的前提就是服务器的80端口能被访问，而且是解析好的域名，所以备案这关其实在这之前已经过了，没问题的情况下可以提交审核，一般在2-5个工作日之间。

网站接入后所得到的App Key 和 App Secret 需要保存起来，代码中将会使用到。

由于本人比较喜欢使用 `devise`, 所以OAuth2的集成也是基于devise的，在`Gemfile`加入相应的gem：

{% codeblock lang:ruby Gemfile %}
# Weibo support
gem 'devise'
gem 'omniauth-weibo-oauth2', '~> 0.2.0'
{% endcodeblock %}

在项目根目录执行 `bundle` 来安装相应的gem

使用 `devise`的朋友都应该清楚，`user.rb`模型中会默认存在功能模块配置，将模块做以下的修改：

{% codeblock lang:ruby app/models/user.rb %}
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :token_authenticatable, :encryptable, :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable,
         :registerable,
         :recoverable,
         :rememberable,
         :trackable,
         :validatable,
         :confirmable,
         :omniauthable, omniauth_providers: [:weibo]
end
{% endcodeblock %}

加入了以上devise的`omniauthable`的配置后, 就能通过devise自带的方法来调出可授权的第三方服务

    >> User.omniauth_providers
    => [:weibo]

到这里，我们可以来配置之前做好的网站接入了，新建一个`service.yml`来存放第三方的App Key和 App Secret：

{% codeblock lang:yaml app/config/services.yml %}
common: &common
  weibo:
    api_key: "这里填写你获取到的api key"
    api_secret: "这里填写你获取到的api secret"
    redirect_uri: "http://www.yourdomain.com/users/auth/weibo/callback"

production:
  <<: *common

development:
  <<: *common

test:
  <<: *common
{% endcodeblock %}

当然，还需要将此配置用到devise和omniauth的集成中：

{% codeblock lang:ruby app/config/initializers/devise.rb %}
SERVICES = YAML.load_file(Rails.root.join("config", "services.yml")).fetch(Rails.env)
Devise.setup do |config|
  # ==> OmniAuth
  # Add a new OmniAuth provider. Check the wiki for more information on setting
  # up on your models and hooks.
  config.omniauth :weibo, SERVICES['weibo']['api_key'], SERVICES['weibo']['api_secret']
end
{% endcodeblock %}

配置完成后我们就可以生成一个controller专门来处理回调的逻辑，我们以`authentications_controller.rb`为例：

    $ rails g controller authentications

既然我们用到了Devise的Omniauth模块，将`authenticationsc controller`做个集成即可，父类为`Devise::OmniauthCallbacksController`，请参考以下代码：

{% codeblock lang:ruby app/controllers/authentications_controller.rb %}
class AuthenticationsController < Devise::OmniauthCallbacksController
  def weibo
    # TODO
  end
end
{% endcodeblock %}

因为我们利用`authentications controller`集成了Devise自己的controller，所以我们需要来修改下默认的路由：

{% codeblock lang:ruby app/config/routes.rb %}
devise_for :users, controllers: {
  omniauth_callbacks: :authentications
}
{% endcodeblock %}

我们可以通过打印路由的方法来看看weibo的回调路由是否已经存在：

    $ rake routes |grep authentications
    user_omniauth_callback /users/auth/:action/callback(.:format) authentications#(?-mix:weibo)

不难看出，weibo的设置已经生效，剩下的事情其实就是来写回调逻辑了，就逻辑而言，想必都应该很清楚，会存在三种情况：

  1. 用户没有登录的情况下授权新浪微博：这个时候其实就是新用户注册，但是新浪微博客是不提供email的，在一些以email为必须字段的情况下，需要另外来处理，比如将授权信息存入session，然后跳转到让用户补充email的界面，必须让用户绑定一个email才能完成授权。这一点国内的其他网站也都差不多，基本都不会提供emial,不像facebook等国外主流网站。

  2. 用户已经登录，但是还未进行新浪微博帐号绑定：这步比较简单，只要将获得的授权信息和当前用户进行绑定即可。

  3. 用户未登录，但是之前已经授权过了：先要根据获得的收钱信息中的uid和provider取出来，数据库中一查便知。

请参考以下的代码：

{% codeblock lang:ruby app/controllers/authentications_controller.rb %}
class AuthenticationsController < Devise::OmniauthCallbacksController
  def weibo
    omniauth_process
  end
  protected
  def omniauth_process
    omniauth = request.env['omniauth.auth']
    authentication = Authentication.where(provider: omniauth.provider, uid: omniauth.uid.to_s).first

    if authentication
      set_flash_message(:notice, :signed_in)
      sign_in(:user, authentication.user)
      redirect_to root_path
    elsif current_user
      authentication = Authentication.create_from_hash(current_user.id, omniauth)
      set_flash_message(:notice, :add_provider_success)
      redirect_to authentications_path
    else
      session[:omniauth] = omniauth.except("extra")
      set_flash_message(:notice, :fill_your_email)
      redirect_to new_user_registration_url
    end
  end

  def after_omniauth_failure_path_for(scope)
    new_user_registration_path
  end
end
{% endcodeblock %}

{% codeblock lang:ruby app/models/authentication.rb %}
class Authentication < ActiveRecord::Base
  attr_accessible :user_id, :provider, :uid, :access_token

  belongs_to :user

  validates :provider, :uid, :access_token, presence: true
  validates :provider, uniqueness: { scope: :user_id }

  def self.create_from_hash(user_id, omniauth)
    self.create!(
      user_id:      user_id,
      provider:     omniauth.provider,
      uid:          omniauth.uid,
      access_token: omniauth.credentials.token
    )
  end
end
{% endcodeblock %}

代码堆砌完成，我们需要在登录页面或者注册页面加入微博授权的链接：

{% codeblock lang:haml %}
%ul
  %li= link_to '微博帐号登录', '/users/auth/weibo', class: 'weibo'
{% endcodeblock %}

本地测试的话可以添加一个host, 将域名绑定到本机，大家可以尝试以下，下次再讲下如何利用OAuth2来实现微博的主要接口。









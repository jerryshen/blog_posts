---
layout: post
title: "Run Wordpress on Nginx"
date: 2012-11-29 22:42
meta: true
comments: true
categories:
- nginx
- wordpress
---

在rails服务器上部署一个wordpress其实并没有想象中那么费事，只要简单安装几个模块即可，
充分的利用你的服务器资源，不要浪费（服务器租用太贵了😄）

假定服务器上已经有rails项目在跑了，也就是说已经安装有nginx，那么，一步步来(root用户权限)。


<!--more-->

先来更新一下系统

{% codeblock %}
apt get update
apt get upgrade
{% endcodeblock %}

Wordpress需要php的支持，我们先来安装php相关的包

{% codeblock %}
apt-get install php5-cli php5-cgi psmisc spawn-fcgi
cd /opt/
wget -O php-fastcgi-deb.sh http://library.linode.com/assets/554-php-fastcgi-deb.sh
mv /opt/php-fastcgi-deb.sh /usr/bin/php-fastcgi
chmod +x /usr/bin/php-fastcgi
wget -O init-php-fastcgi-deb.sh http://library.linode.com/assets/553-init-php-fastcgi-deb.sh
mv /opt/init-php-fastcgi-deb.sh /etc/init.d/php-fastcgi
chmod +x /etc/init.d/php-fastcgi
/etc/init.d/php-fastcgi start
update-rc.d php-fastcgi defaults
{% endcodeblock %}

还有一点需要注意的是php的主题或者插件需要php GD库的支持

{% codeblock %}
apt-get install php5-gd
/etc/init.d/php-fastcgi restart
/etc/init.d/nginx restart
{% endcodeblock %}

添加`fastcgi_params`文件

{% codeblock %}
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
# fastcgi_param  REDIRECT_STATUS    200;
{% endcodeblock %}

加入nginx配置

{% codeblock %}
server {
  listen 80;
  server_name domain.com;
  access_log /var/www/wordpress/logs/access.log;
  error_log /var/www/wordpress/logs/error.log;
  root /var/www/wordpress;

  location / {
    index index.html index.htm index.php;
  }

  location ~ \.php$ {
    include /path-to-your-nginx/fastcgi_params;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME /var/www/wordpress$fastcgi_script_name;
  }
}
{% endcodeblock %}

接下来就是创建数据库的事情了。。不一一讲述。

---
layout: post
category: "pi"
title:  "owncloud的nginx配置"
tags: [pi,owncloud]
summary: "owncloud最新版的官方安裝文檔是针对LAMP的，没有对nginx环境的安装说明，但owncloud8的文档里有提及到，这里记录一下，以方便查阅。"
---
其實owncloud是一个PHP应用，可以很容易地部署在任何一个web服務器中，根本就沒必要通過apt安裝。但owncloud9的官方文檔只有apache的配置文件，這對於很多使用nginx的开发者来说不直观。于是我Google了一下，发现owncloud8的文檔裏竟然就有nginx配置：

```

upstream php-handler {
  server 127.0.0.1:9000;
  #server unix:/var/run/php5-fpm.sock;
}

server {
  listen 80;
  server_name cloud.example.com;
  # enforce https
  return 301 https://$server_name$request_uri;
}

server {
  listen 443 ssl;
  server_name cloud.example.com;

  ssl_certificate /etc/ssl/nginx/cloud.example.com.crt;
  ssl_certificate_key /etc/ssl/nginx/cloud.example.com.key;

  # Add headers to serve security related headers
  add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;";
  add_header X-Content-Type-Options nosniff;
  add_header X-Frame-Options "SAMEORIGIN";
  add_header X-XSS-Protection "1; mode=block";
  add_header X-Robots-Tag none;

  # Path to the root of your installation
  root /var/www/owncloud/;

  # set max upload size
  client_max_body_size 10G;
  fastcgi_buffers 64 4K;

  # Disable gzip to avoid the removal of the ETag header
  gzip off;

  # Uncomment if your server is build with the ngx_pagespeed module
  # This module is currently not supported.
  #pagespeed off;

  index index.php;
  error_page 403 /core/templates/403.php;
  error_page 404 /core/templates/404.php;

  rewrite ^/.well-known/carddav /remote.php/carddav/ permanent;
  rewrite ^/.well-known/caldav /remote.php/caldav/ permanent;

  # The following 2 rules are only needed for the user_webfinger app.
  # Uncomment it if you're planning to use this app.
  #rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
  #rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json last;

  location = /robots.txt {
    allow all;
    log_not_found off;
    access_log off;
  }

  location ~ ^/(build|tests|config|lib|3rdparty|templates|data)/ {
    deny all;
  }

  location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
    deny all;
  }

  location / {

    rewrite ^/remote/(.*) /remote.php last;

    rewrite ^(/core/doc/[^\/]+/)$ $1/index.html;

    try_files $uri $uri/ =404;
  }
  location ~ \.php(?:$|/) {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_path_info;
    fastcgi_param HTTPS on;
    fastcgi_param modHeadersAvailable true; #Avoid sending the security headers twice
    fastcgi_pass php-handler;
    fastcgi_intercept_errors on;
  }

  # Adding the cache control header for js and css files
  # Make sure it is BELOW the location ~ \.php(?:$|/) { block
  location ~* \.(?:css|js)$ {
    add_header Cache-Control "public, max-age=7200";
    # Add headers to serve security related headers
    add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;";
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    # Optional: Don't log access to assets
    access_log off;
  }

  # Optional: Don't log access to other assets
  location ~* \.(?:jpg|jpeg|gif|bmp|ico|png|swf)$ {
    access_log off;
  }
}
```
## 如何使用
这个例子用的是HTTPS，改成HTTP只需把其中SSL部分配置去掉即可，其餘部分需按自己部署的實際環境修改。

## 另外值得一提的
上例中的php-handler采用upstream的方式來配置。有什麼特別呢？

通常一般人可能會直接在fastcgi_pass語句中直接寫具體的fcgi對接實現，例如tcp方式的127.0.0.1:9000或unix socket的socket文件路徑。當你只有一個vhost的時候，這種寫法當然是最快最簡單的。但當你有多個vhost時，一旦fpm的方式有變動，則需要手工改所有vhost配置文件的同一行配置。這樣的配置方式其維護成本是很高的。

當采用upstream的方式時，相當於定義了一個fcgi實現的變量，所有vhost只是使用這個變量。好處有：

1.要更改fpm的連接方式時，只需改一個upstream的配置即可。

2.可定義多個fpm的實例。例如針對你需要的各個PHP版本定義多個upstream,在使用時可靈活切換。示例配置如下：

```
upstream php5-handler {
#  server 127.0.0.1:9000;   # tcp socket方式
    server unix:/var/run/php5-fpm.sock;     # unix socket方式
}

upstream php7-handler {
#  server 127.0.0.1:9001;   # tcp socket方式
    server unix:/var/run/php7-fpm.sock;     # unix socket方式
}
```

由於開發需要，我通常會開兩個版本的PHP-FPM，如上例，一個是php5.x的，另一個是php7的，這樣某些不兼容php7的舊項目可以直接使用相同的php5.3的upstream，其他版本也能靈活切換。


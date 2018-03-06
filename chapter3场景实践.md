# nginx 场景实践

## 静态资源web服务

![](./images/static_resource.png)

nginx作为静态资源的http server其可以接收客户端类似于jpeg、html、flv 等静态资源的请求，然后直接通过静态文件的存储得到这些文件返回给客户端，这中方式是一种典型的比较高效的方式；这种场景经常会利用在 我们对静态资源的处理、请求、动静分离场景下面的应用

我们可以在请求一个网站的时候简单分为两类，一种是静态请求，一种是动态请求；而对于我们服务端，接收静态请求与动态请求时不一样的； 动态请求 我们时需要服务端的解释器进行一些比较复杂的逻辑运算，将对应的数据进行必要的封装，然后反馈给用户，这种数据时动态生成的； 对于那些不需要服务端动态生成的，即所请求的东西，在服务器上的文件目录中就可以直接找到的东西，则这种请求就是静态资源请求，文件就是静态资源文件

![](./images/static_r.png)

```bash
server {
    listen       80;
    server_name  116.62.103.228 jeson.imooc.com;
    
    sendfile on;
    #charset koi8-r;
    access_log  /var/log/nginx/log/static_access.log  main;

    
    location ~ .*\.(jpg|gif|png)$ {
        gzip on;
        gzip_http_version 1.1;
        gzip_comp_level 2;
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root  /opt/app/code/images;
    }

    location ~ .*\.(txt|xml)$ {
        gzip on;
        gzip_http_version 1.1;
        gzip_comp_level 1;
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root  /opt/app/code/doc;
    }

    location ~ ^/download {
        gzip_static on;
        tcp_nopush on;
        root /opt/app/code;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504 404  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```

### 静态资源服务场景CDN

> CDN有称为内容分发网络

## 代理服务

```bash
# fx_proxy.conf中

server {
    listen       80;
    server_name  localhost jeson.t.imooc.io;

    #charset koi8-r;
    access_log  /var/log/nginx/test_proxy.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    
    location ~ /test_proxy.html$ {
        proxy_pass http://127.0.0.1:8080;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```

```bash

# realserver.conf中
server {
    listen       8080;
    server_name  localhost jeson.t.imooc.io;

    #charset koi8-r;
    access_log  /var/log/nginx/server.access.log  main;

    location / {
        root   /opt/app/code2;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}



```

###  其它配置语法

> 头信息
 
 ![](./images/proxy_header.png)

 有时候我们需要将nginx作为代理服务器，但后端的server时需要读取一些头信息，而这些头信息时不准确的； 也就是在nginx做为访问控制这一层里面。remote_dir(没听清)这个信息就在后端的服务器里面，因为走了代理，所以后端就没办法读取对应的消息，此时就需要用到proxy_set_header 也就是发给后端的服务器里面 我们增加一个对应的头 将对应的信息 用新的这个头的方式 携带到后端 让后端能够读取到；这个时头信息的一个配置

 proxy_hide_header : 隐藏一些头不够后端访问到


> 超时：nginx作为代理到后端服务器中间的一个超时，这个超时 是一个tcp链接超时,当建立完链接之后，就有另外两个超时proxy_read_timeout、proxy_send_timeout

![](./images/timeout.png)

proxy_read_timeout : 已经建立好的链接的情况下面，在nginx作为代理和后端server之间让它会等待多长时间

proxy_send_timeout： 服务端请求完毕 发送给客户端的时间


### 补充配置

```bash
# fx_proxy.conf中

server {
    listen       80;
    server_name  localhost jeson.t.imooc.io;

    #charset koi8-r;
    access_log  /var/log/nginx/test_proxy.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    
    location ~ /test_proxy.html$ {
        proxy_pass http://127.0.0.1:8080;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```

## 负载均衡调度器SLB

## 动态缓存


```bash
# 利用nginx 查看本机因为nginx而启用的端口

netstat -luntp|grep nginx

```
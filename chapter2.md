# nginx基础

## 什么是nginx中间件

在我们的网站的后台往往存在很多的网站应用服务，对应的是我们的操作系统来驱动我们的硬件为我们提供相应的服务，应用之间的直接调用或者应用与操作系统间直接交互，会导致层次化的应用不够隔离，代码耦合程度高 ； 我们需要有一个东西来代理或处理我们的请求，让应用只负责业务的逻辑处理，这样的化就出现了中间件

## nginx的安装

http://nginx.org/en/linux_packages.html#stable

To set up the yum repository for RHEL/CentOS, create the file named /etc/yum.repos.d/nginx.repo with the following contents:

```bash

[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1

```

Replace “OS” with “rhel” or “centos”, depending on the distribution used, and “OSRELEASE” with “6” or “7”, for 6.x or 7.x versions, respectively.

利用 : yum list | grep nginx  可以查看已加源的列表

```bash
    yum install nginx 

    # nginx -v 安装的版本
    # nginx -V 编译的基本参数
```

## 基本使用

### 安装目录

> rpm 包管理
我们使用yum 来安装，其实都是装在一个由一个rpm包里面,对于linux的rpm管理器 只需要输入`rpm -ql  name` 就可以列出我们已经安装的name服务所对应的配置文件在我们操作系统上的目录

```bash
rpm -ql name 

# 查看打印出来的信息，配置文件基本上放在 /etc /usr /var 这三个目录当中

# /etc 存放软件的核心配置
```

### 编译参数

```bash
nginx -V 
# 用来查看nginx到底启用来那些模块

```

### Nginx的基本配置语法 

#### 主配置文件nginx.conf的基础配置语法

```bash
# /etc/nginx/nginx.conf文件中

# 设置nginx服务的系统的使用用户（一般情况下不用去变）
user  nginx;
# 工程进程数 与nginx的多进程管理有关系 io复用 我们会启用多个进程来，来增加其连接数的并发处理（配置的线程数一般与cpu的额核心数保持一致即可）
worker_processes  1;
# nginx的错误日志
error_log  /var/log/nginx/error.log warn;
# nginx服务启动的时候的pid
pid        /var/run/nginx.pid;


events {
    # 重要配置： 每个进程允许的最大链接数 （最大可设置成65535, 一般企业设置成10000左右，即可以满足需求） 是必须要优化的项
    worker_connections  1024;
}


http {
    # include 用来引入子配置文件
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # 定义日志的格式与类型
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    # 客户端与服务端的额超时的时间 65s
    keepalive_timeout  65;

    #gzip  on;
    # 类似于@import语法，包含了对应目录下的文件，也就是nginx会先去读取主配置文件，读到include字段之后，器会将对应目录下的文件也都读取到；因为默认/etc/nginx/conf.d/下面默认只有 default.conf文件

    # 所以nginx默认会有两个配置文件: 一个是nginx.conf一个是被include进来的default.conf文件

    include /etc/nginx/conf.d/*.conf; 
}

```

#### default.conf配置文件


```bash
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        # 要返回页面在系统中的目录位置
        root   /usr/share/nginx/html;
        # index指定默认返回的页面 返回root指定的目录中的某一个文件，没有index.html 则就去匹配index.htm
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

#### 常见http模块的主要配置方式

http 处于最外面一层，每个http都可以包含许多对外的服务，我们访问每一个网站使用的都是http协议，如：`htttp`://www.baidu.com 我们输入的请求地址 都是由http打头的，所以http都是放最外面的

server 每个server都用来配置一个独立的站点，在每个server里面配置当前server的属性，

```bash

http {
    # 每个server里面会出现多个location,我们对不同的location进行对应的配置
    server {
        listen 80;
        # 默认的是localhost 可配置为公网ip 与独立域名
        server_name localhost;
        # location是server_name下面的一个模块，其主要是控制着我们每一层的路径的访问，如 此时我们访问的是 http://www.baidu.com 访问的就是根路径 即 匹配'/' 最上面的一层路径，
        location / {
            root /user/share/nginx/html;
            index index.html index.htm;
        }

        # 配置错误页面的路径，当我们访问不到页面 或者服务服务端出现500 502 503 504 的返回状态码，我们统一定位到一个请求路径 /50x.html 上去， 返回给用户，
        error_page 500 502 503 504 /50x.html
        # 而路径 /50x.html  会匹配到下一个location 
        location = /50x.html {
            # root指的是我们的页面在系统中目录的路径， 也就是50x.html就存放在root指定的路径下面
            root /usr/share/nginx/html;
        }

    }
}

```

```bash
# 查看ip的系统信息 但看到是私有ip而不是公网ip
ip a 

# icanhazip.com 使你在任何地方知道你的公网IP地址
# icanhazip.com是一个网址，你在浏览器中输入这个网址，你就能得到你的公网IP地址了。
curl icanhazip.com

# 重启nginx 
systemctl restart nginx.service

# 柔和的重启
systemctl reload nginx.service

# 查看日志 因为我们要看那些最新加的东西，所以用到tail -f
# tail -f 可以实时的得到新追加到文件中的信息，常用来跟踪日志文件tail -f 
# tail -n 200 打印日志文件的最后200行

 tail -f /var/log/nginx/error.log


 # -t 查看nginx配置文件正确与否
 nginx -t
 # -c 指定配置文件的路径 进行前期配置语法的检查；
 nginx -t -c /etc/nginx/nginx.conf
 # 重新加载指定配置文件
 nginx -s reload -c /etc/nginx/nginx.conf

```

## nginx的基础知识

### http请求

nginx作为web server与http代理 处理的就是http的请求，http请求时建立在tcp基础之上的；

1. 每个http的请求主要包含了两个： 一个客户端发起request请求给服务端，一个是服务端给客户端一个response相应； 
2. 每一次的request与response 都会发送对应的http报文； request报文包括：请求行、请求头部、请求数据； response报文包括：状态行、消息报头、响应正文

```bash
# 利用curl进行测试
# curl 可以理解为我们的浏览器 不过在linux中 其是一个命令行 不能完成浏览器的渲染，所以我们不能看到内容，看到的只有返回给我们的html代码

# 返回的只有html页面代码，并不能看到返回与请求信息
curl http://www.baidu.com

# 加上-v参数，查看每次请求与返回的具体内容
# /dev/null 表示空设备，这里就是把日志记录到空设备里，就是不记录日志。
curl -v http://www.baidu.com > /dev/null


```

### Nginx的日志类型

nginx的日志主要包括error.log 与access_log

#### error.log日志

主要记录nginx处理http请求的错误的状态，以及nginx本身服务运行的错误的状态，这回按照不同的级别记录到error.log里面

#### access_log日志

主要记录nginx的每一次http请求的访问状态，主要用来分析每次请求与客户之间的交互，以及对行为的 一些分析，企业实际使用会比较多

nginx主要是依赖log_format配置来实现，nginx log里面记录了很多的信息，每一个信息都可以理解为对应nginx里面的变量，而log_format就是将这些变量组织到一起来，并记录到access_log里面去。配置语法如下：

```bash

# log_format的配置语法
# log_format 配置关键字
# name 格式的名字 可以自定义 类似变量名
Syntax: log_format name [escape=default|json] string ...;
# 默认的配置
Default: log_format combined "....";
# 指明对于 log_format只能配置在http这个大的模块下面，对于log_format配置的位置是有约束的，
Context: http

``` 
```bash
# /etc/nginx/niginx/conf中

log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    # main 指的就是上述log_format 语法中的 配置的格式名； 其与上面的`log_format main`是相互对应的，表示将会以'main'的格式要求 记录到access_log中去；
    access_log  /var/log/nginx/access.log  main;


```
### Nginx变量

####. HTTP请求变量 - arg_PARAMETER、http_HEADER、sent_http_HEADER

1. http_HEADER 是request请求里面的header  , HEADER 指的是请求头中的项 如：User-Agent Host 等，格式要变为`$http_user_agent  $http_host` 即大写变小写，横线变下划线；
2. sent_http_HEADER 是服务端返回给客户端的response的header


####. 内置变量 - Nginx内置的

> 文档地址 ： http://nginx.org/en/docs/http/ngx_http_log_module.html#access_log
> nginx所有变量的描述 http://nginx.org/en/docs/http/ngx_http_core_module.html#var_status


####. 自定义变量 - 自己定义



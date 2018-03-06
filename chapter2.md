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

```

## http请求




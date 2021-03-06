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
        #1 必须要使用到的跳转配置
        proxy_pass http://127.0.0.1:8080;
        include proxy_params;
    }

    #error_page  404               /404.html;

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
# proxy_params中

# 2.除非时后端返回301 且我们要进行一些改写 否则都将其配置为默认
proxy_redirect default;

# 3. 将nginx的代理向向后端的server发送信息的时候，所添加的头信息
proxy_set_header Host $http_host;
# 3.1 后端的server通过代理之后其实没法获取真实的用户ip的，往往要做用户访问限制，或实际上有常常需要获取用户的ip信息 ，这样我们就需要在前端将真实的用户ip信息 带到后端去 
proxy_set_header X-Real-IP $remote_addr;

# 4 链接超时的配置
proxy_connect_timeout 30;
proxy_send_timeout 60;
proxy_read_timeout 60;

# 5 缓冲区的一系列设置
proxy_buffer_size 32k;
proxy_buffering on;
proxy_buffers 4 128k;
proxy_busy_buffers_size 256k;
proxy_max_temp_file_size 256k;

```

![](./images/temp_file.png)

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

## 负载均衡调度器SLB

### nginx 负载均衡

![](../../images/负载均衡.png)

> 为什么需要负载均衡服务，

### SLB 模式
![](./images/slb.png)

一般调度节点与服务节点同处在一个逻辑单元里面，当海量信息请求过来之后 其请求的是调度节点，调度节点将服务请求转发给后端对应的服务节点； 服务节点将响应发送至调度节点，调度节点再去响应给客户；

![](./images/lsb_model.png) 

### 七层负载均衡与四层负载均衡
利用网络的模型 也就是osi的模型，可以分为两种概念，一种是四层负载均衡、一种是七层负载均衡；

所谓四层负载均衡就是`osi`模型中的`传输层` 传输层已经能支持tcp 协议的控制，所以器只需要对客户端的请求进行tcp\ip协议的包转发，就可以实现这个负载均衡，其好处是性能快，只需要对顶层进行应用处理，而不需要去进行一些比较复杂的逻辑，只需要进行包的转发就可以了；

与之对应的七层负载均衡就是osi模型中的应用层，所以其可以完成应用层面的协议的请求，如对于http ,其可以实现http信息的改写、头信息的改写、安全应用规则的控制 以及转发的规则，所以在应用层的服务里面，我们能做的更多；而nginx就是一个典型的七层负载均衡的slb

### nginx负载均衡的实现的原理

![](./images/负载均衡原理.png)

1. 其实nginx实现负载均衡的时候主要使用的就是proxy_pass，proxy_pass是代理模块的核心配置，其将所有的客户端请求代理，然后转发到后端的服务器上面
2. 但其实其并不是将请求转发到一台而是一组--一组服务池，我们称为upstream server; upstream server组中可以去定义所有服务的单元，如定义 服务1、服务2、服务3三个单元，它们三个单元都可以提供相同的类似的服务； 将它们三个放到同一个upstream组里面，在组里面其实现了对请求的分发； 这样当海量的用户请求过来的时候，upstream 这个模块就能不断的，将请求分发到不同的模块上面，从而实现负载均衡；
3. 所以proxy_pass 与 upsream server是实现负载均衡的两个核心的配置语法；

### upstream server的配置语法

```bash
Syntax: upstream name {...}
Default: ----
# upsream 必须得配置在http的下一层，也就是必须在server外面；
Context: http

```

### 实现负载均衡的实例

1. 我们准备了两台服务器，一台服务器单独做后端服务 提供真实的web服务，另外一台单独做负载均衡；

![](./images/aliyun_secure.png)

```bash
# 在opt/app/下面新建三个目录code1、code2、code3;在三个目录中分别存放了 三个不同的展示效果页面index.html

# 在 /etc/nginx/conf.d/新建三个server对应的：server1.conf server2.conf server3.conf  分别用来监听不同的端口  以表示建立起了单个端口不同的服务；server1--8001端口 server2---8002端口 server3---8003端口；对应的程序目录分别在code1 code2 code3上面

# 在浏览器或curl中使用公网ip加端口的方式进行访问，如自己的http://47.95.114.174:8001; 此时会发现页面是访问不到的，但利用netstat -luntp可以看到nginx在正常的监听； 归其原因是因为我们的aliyun配置了 防火墙的规则；以至于 我们的服务器无法收到客户端发送过来的请求 ；解决方式是修改阿里云安全组的规则 或iptables的规则；

# server1.conf
server {
    listen       8001;
    server_name  localhost;

    access_log  /var/log/nginx/log/server1.access.log  main;

    location / {
        root   /opt/app/code1;
        index  index.html index.htm;
    }

    error_page   500 502 503 504 404  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

# server2.conf
server {
    listen       8002;
    server_name  localhost;

    #charset koi8-r;
    access_log  /var/log/nginx/log/server2.access.log  main;

    location / {
        root   /opt/app/code2;
        index  index.html index.htm;
    }

    error_page   500 502 503 504 404  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}

# server3.conf
server {
    listen       8003;
    server_name  localhost;

    location / {
        root   /opt/app/code3;
        index  index.html index.htm;
    }

    error_page   500 502 503 504 404  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}



```

2. 我们会有三个服务对应不同的三个端口，接下来我们就要用负载均衡来配置不同的负载均衡的效果，我们用一个地址来代理负载到不同的服务器上面，我们就需要配置我们的前端的负载均衡服务器了；

```bash
# /etc/nginx/conf.d/upstream_test.conf 中

    # imooc是我们的名字是自定义的
    upstream dh_server {
        # 三个服务节点分别是8001、8002、8003节点
        server http://47.95.114.174:8001;
        server http://47.95.114.174:8002;
        server http://47.95.114.174:8003;
    }

server {
    listen       80;
    server_name  localhost jeson.t.imooc.io;

    access_log  /var/log/nginx/test_proxy.access.log  main;

    resolver  8.8.8.8;
    
    location / {
        # 在location这一级 将所有的请求都 proxy_pass到imooc也就是我们自定义的upstream组的name
        proxy_pass http://dh_server;
        include /etc/nginx/conf.d/proxy_params;
    }

   
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}


```

> 在浏览器页面不断刷新的时候，我们可以很好的看出来 负载均衡默认是轮询的，多次请求会在server1、server2、server3之间不停的轮询

3. 测试： 假设有一个服务器宕机了，不能服务了，那么剩下的两个节点是否还可以正常的去服务？在这里因为三个服务节点是利用后台一个nginx进程来开启的，所以我们不太好直接将进程关掉； 现在我们想将其中一个关掉
此时我们要去使用iptables的规则去实现这一点，将所有请求8002的端口都给drop掉，即不对外提供8002端口的服务

```bash
iptables -I INPUT -p tcp --dport 8002 -j DROP

# 我们在刷新页面的时候 就会不断的在server1与server3之间轮询，而server2就没有再展示，此时说明我们的负载均衡检测到server2是无法请求到的，就将server2这个服务节点直接下线掉了；
```

> iptables 日志 http://www.zsythink.net/archives/category/%E8%BF%90%E7%BB%B4%E7%9B%B8%E5%85%B3/%E9%98%B2%E7%81%AB%E5%A2%99/

### upstream其它的一些配置项

```bash
# backend 自定义的这一组虚拟服务的名字
upstream backend {
    # server 可以是ip的写法，也可以是域名的写法
    # 每一个server的后面 我们可以添加一些 小的配置参数，
    # weight 表示该节点的权重，对与轮询的这种方式而言，权重越大 轮询时分配到的几率就会越高；
    server backend1.example.com weight=5;
    server backend1.example.com:8080;
    server unix:/tmp/backend3;
    
    # backup 表示这是一个备份节点
    server backup1.example.com:8080 backup;
    server backup2.example.com:8080 backup;
}

```

![](./images/upstream_params.png)

* down 表示当前的节点不参与到负载均衡里面，也就是再里面没用 但是我们写在里面，不对外提供服务
* backup 也是表示不对外提供服务，但是他是在有其它成员存活的情况下，但同组的其它节点无法提供服务的时候，这个时候nackup就会被利用起来
* max_fails 代理服务器会对后端的服务做检查，若发现请求失败 或状态失败，就会将该节点标记成失败； 这样就会再一次去重试这一个服务器 就是再一次去检查； 当请求失败的次数用完之后，就会timeout  就是会休息一会；
* fail_timeout 服务器暂停的时间 默认是10s
* max_conns 限制每个server的最大链接数目，若发现某些后端的节点不均匀的情况下，如有的节点是四核的服务器，有的节点是24核的服务器，但是这些不均匀的节点 有被加在我们的同一组server里面；此时我们就可以去限制最大连接数，当请求数满了之后 ，就不会去发请求了；

### upstream的调度算法

![](./images/upstream_weight.png)

* 加权轮循： 默认的weight值为1

#### ip_hash

加权轮询与轮询都是基于请求来进行分配的，如果我们不想依赖请求，而是项保证对于一些cookoes session等信息会一直，也就是每一次用户的请求，如果基于请求来的 就会到不同的服务器上面，导致用户登陆的cookies信息验证会出现一些问题（掉线），这样的话 我们就需要另外一种方式--不基于请求的方式 来进行节点的分配；
具体实现，早先出现一种利用ip_hash的方式，ip_hash会基于用户的ip来计算用户的hash值，然后将每一个固定ip过来的请求 都会分配到同一个服务节点上去；这样就解决了 多次请求 会到达不同的服务器上面 这样一个问题， 所以ip_hash就出现了；

```bash
# /etc/nginx/conf.d/upstream_test.conf 中

    upstream dh_server {
        # 将ip_hash放到server节点上面；
        ip_hash;
        server http://47.95.114.174:8001;
        server http://47.95.114.174:8002;
        server http://47.95.114.174:8003;
    }

server {
    listen       80;
    server_name  localhost jeson.t.imooc.io;

    access_log  /var/log/nginx/test_proxy.access.log  main;

    resolver  8.8.8.8;
    
    location / {
        proxy_pass http://dh_server;
        include /etc/nginx/conf.d/proxy_params;
    }

   
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}


```

> 此时我们打开浏览器 去访问地址 会发现多次请求ip地址，请求始终会被分配到同一个server节点，这是ip_hash的策略，其会基于我们的$remote_addr来做hash, 若是同一个remote_addr过来其就会定位到同一台服务器上面去； 但是这种方式是也存在缺陷， remote_addr 如果是走代理这种方式的话，即前端再走一层remote_addr 则获取到的就不再是真实的remote_addr ; 这样我们就无法基于用户真实的ip来做对应的轮询了,这样用户请求的话始终会定位到一台机器上面去；； 更新方式是 走基于url_hash

#### url_hash 与 hash 关键数值

1. 配置语法

```bash
Syntax: hash key [consisitent]
Default: ---
Context: upstream
This directive appeared in verion 1.7.2

```

2. 实例演示

```bash
# 在opt/app/的三个目录code1、code2、code3;下分别新增3个html:url1.html url2.html url3.html  在三个目录中分别存放了 四个html文件

    upstream imooc {
        # $request_url 就是我们请求的参数 非域名的部分 http://47.95.114.174/url1.html 中的url1.html
        # 是基于$request_url这个自定义变量 来做hash 当每次请求的都是同一个 request_url的时候，其就会定位到同一台服务器上面
        #  再实际的场景里面我们的 $request_url会有很大一长串  http://47.95.114.174/url1.html?user=dfasdfasd&dfas=fasdfasdfa 会有很多的相关的信息，带到我们的服务器后台去； 
        # 我们想针对url中的某一个值来做hash也是可以的，我们只需要在服务端 加上一行判断语句，将对应的值用正则表达式提取出来，然后将自定义的变量进行hash
        hash $request_uri;
        server 116.62.103.228:8001;
        server 116.62.103.228:8002;
        server 116.62.103.228:8003;
    }

server {
    listen       80;
    server_name  localhost jeson.t.imooc.io;

    #charset koi8-r;
    access_log  /var/log/nginx/test_proxy.access.log  main;
    resolver  8.8.8.8;
    
    location / {
        proxy_pass http://imooc;
        include proxy_params;
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





## 动态缓存

缓存是为了减少后端的压力，让所有的请求都能集中在前端，在前端就能取得数据，

### 缓存的类型

![](./images/缓存类型.png)

1. 服务端缓存 若缓存出现在服务端则称为服务端缓存，最常见的就是用到一个redis存储一些key value型的数据，我们会将所有的数据放到一个单元里面，进行持久化或非持久化的存储

2. 代理缓存 若缓存集中在代理或中间件上面我们称之为代理缓存，是从服务端获取到的 然后在本地存储一份 并直接返回给客户端去使用

3. 客户端缓存 其实就在浏览器上面 数据都是从后台过来的，只不过给前台一份 这样用户就能自己访问自己

### nginx 作为代理缓存的流程

![](./images/代理缓存.png)

1. 首先客户端去请求nginx 如果第一次请求的时候,如果nginx本地没有缓存； 
2. 其会向服务器发送请求对应的数据，
3. 然后服务器返回对应的数据，nginx本地进行缓存
4. 然后返回给客户； 上述是在没有缓存的情况下

5. 当用户再次发起 同一种请求的时候，请求数据a ; 而nginx 本地已经有了a数据的缓存
6. 所以nginx就直接将数据返回给客户端，而无需再一次想服务器进行请求；

###  nginx代理缓存配置语法


1. proxy_cache_path 定义路径 目录空间大小 用来存放缓存文件

```bash
Syntax:	proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
Default:	—
Context:	http

```
2. proxy_cache配置语法

```bash
Syntax:	proxy_cache zone | off;
Default:	
proxy_cache off;
Context:	http, server, location

```

3. proxy_cache_valid缓存周期过期 
 
 ```bash
    # code 表示返回状态码
 Syntax:	proxy_cache_valid [code ...] time;
 Default:	—
 Context:	http, server, location
 
 ```

 4. proxy_cache_key 缓存的维度

 ```bash
 Syntax:	proxy_cache_key string;
 # $schema 协议 + servername + 请求参数
 # 将三者作为一个单独的key 作为一个纬度进行缓存
 Default: proxy_cache_key $scheme$proxy_host$request_uri;
 Context:	http, server, location
 
 ```

 ### nginx作为缓存服务的配置场景实例

```bash
    upstream imooc {
        server 116.62.103.228:8001;
        server 116.62.103.228:8002;
        server 116.62.103.228:8003;
    }
    # proxy_cache_path是最先需要配置的
    # /opt/app/cache 存放缓存文件的目录
    # levels 目录分级 一半建议就是1:2
    # keys_zone   我们开辟的zone空间的名字 在后面proxy_cache 配置项中的zone 就是这个空间名  10m 就开辟空间的大小10兆
    # max_size 目录控制最大的多大
    # inactive=60 inactive 表示不活跃的，即若在60分钟之内该缓存文件 未被请求过，则该缓存文件，就会被清理掉；
    # use_temp_path 用来存放临时文件的，一半建议将其关闭；若开启需要重新配置一个目录，和path配置的目录并行 会产生一定的问题；

    proxy_cache_path /opt/app/cache levels=1:2 keys_zone=imooc_cache:10m max_size=10g inactive=60m use_temp_path=off;

server {
    listen       80;
    server_name  localhost jeson.t.imooc.io;

    #charset koi8-r;
    access_log  /var/log/nginx/test_proxy.access.log  main;

    
    location / {
        # imooc_cache 前面proxy_cache_path中定义的zone 表示我们已经开启了缓存
        proxy_cache imooc_cache;
        # proxy_cache off;
        proxy_pass http://imooc;

        # 
        proxy_cache_valid 200 304 12h;
        proxy_cache_valid any 10m;
        # 
        proxy_cache_key $host$uri$is_args$args;
        # 
        add_header  Nginx-Cache "$upstream_cache_status";  
        
        # 应该在负载均衡中讲到：如果后端的其中的一台服务器，出现了、500、502、503、504或者不正常的头返回、出错的时候 我们就让其跳过这一台 去访问下一台；避免其中一台服务器宕机 会对前端服务产生影响；
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        include proxy_params;
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

### 补充 如何清理指定的缓存

* 利用rm -rf 清理缓存目录下的所有内容； 缺点是会将所有的缓存 都给清理掉

* 利用 第三方的扩展模块ngx_cache_purge purge(净化)
  需在学习nginx第三方模块的编译 安装之后使用

### 补充 如何让部分页面不缓存？

在某些场景里面 如登陆的页面 或者用户不能缓存的页面 那么我们是否能够让部分页面不缓存？ 使用proxy_no_cache 来配置 

```bash
# 用来配置那一部分url 是不用来做缓存的；一=一般是交互页面的url
Syntax:	proxy_no_cache string ...;
Default:	—
Context:	http, server, location

```


### 补充-如何去清理指定的缓存

* 利用rm -rf 缓存目录内容 清理全部缓存
* 利用第三方的扩展模块 ngx_cache_purge 清理指向的缓存 需要手动的编译与安装；后期会讲

### 如何让部分的页面不去缓存 proxy_no_cache

```bash
Syntax:	proxy_no_cache string ...;
Default:	—
Context:	http, server, location
# Defines conditions under which the response will not be saved to a cache. If at least one value of the string parameters is not empty and is not equal to “0” then the response will not be saved:
```


 ```bash
 Syntax:	proxy_cache_key string;
 # $schema 协议 + servername + 请求参数
 # 将三者作为一个单独的key 作为一个纬度进行缓存
 Default: proxy_cache_key $scheme$proxy_host$request_uri;
 Context:	http, server, location
 
 ```

 ### nginx作为缓存服务的配置场景实例

```bash
    upstream imooc {
        server 116.62.103.228:8001;
        server 116.62.103.228:8002;
        server 116.62.103.228:8003;
    }
    proxy_cache_path /opt/app/cache levels=1:2 keys_zone=imooc_cache:10m max_size=10g inactive=60m use_temp_path=off;

server {
    listen       80;
    server_name  localhost jeson.t.imooc.io;

    access_log  /var/log/nginx/test_proxy.access.log  main;

    #  $cookie_nocache 为自定义的变量 只要请求字符串的uri 匹配指定的规则，就设为1（非零 非空）
    if ($request_uri ~ ^/(url3|login|register|password\/reset)) {
        set $cookie_nocache 1;
    }
    #######
    
    location / {
        proxy_cache imooc_cache;
        proxy_pass http://imooc;

        proxy_cache_valid 200 304 12h;
        proxy_cache_valid any 10m;
        proxy_cache_key $host$uri$is_args$args;
        add_header  Nginx-Cache "$upstream_cache_status";  

        # 按照官方的文档，只要其中有一个变量的值部位0 或非空， 本次从后台过来的响应 就不会去缓存；
        proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
        proxy_no_cache $proxy_pragma $hhtp_authorization;
        ##
        
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        include proxy_params;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}



```


### nginx 大文件的分片请求

```bash
Syntax: slice size;
Default: slice 0;
Context: http,server,location;

```





```bash
# 利用nginx 查看本机因为nginx而启用的端口

netstat -luntp|grep nginx

```

> nginx大神博客 http://blog.sina.com.cn/openresty

> 网络模型 

http://www.toxingwang.com/linux-unix/linux-basic/1712.html

沟通就会有一些好的事情发生；

> 后台要做的事情----所谓动静分离

1. 静态资源托管 nginx
2. 交互程序 node java php
3. 上传与下载


反向代理 集中分布 负载均衡 动静分离

> mac 切换同一个程序的不同窗口

command + `

> 新建当前程序的窗口

command + n
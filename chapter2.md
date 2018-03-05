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

## 基本参数的使用
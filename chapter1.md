# 环境的准备

## 环境调试的确认

> 四项确认

1. 确认系统网络
2. 确认yum可以使用

yum list|grep gcc  (查找含有gcc名称的包)

3. 确认关闭iptables规则
4. 确认停用 selinux

> 两项基本安装

```bash
# 1 系统的基本库(省去安装其它工具时候的依赖)
yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake

# 2 安装一些基本的工具
yum -y install wget httpd-tools vim 

```

> 一次初始化

```bash
# 初始化自己的系统目录, 分别存放不同的东西
# app/ 存放代码
# downloads/ 从网站上下载的源码包
# logs/ 存放自定义的日志
# work/ 存放自定义的脚本
# backup/ 默认配置文件的备份 或者需要对一些东西进行修改备份的时候 
cd /opt;
mkdir app download logs work backup

```
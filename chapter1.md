# 环境的准备

## 环境调试的确认

> 四项确认

1. 确认系统网络
2. 确认yum可以使用

> yum list|grep gcc  (查找含有gcc名称的包) 以此来进行软件搜索

3. 确认关闭iptables规则

```bash
# 查看配置规则项
iptables -L 
iptables -t nat -L 
# 关闭iptables
iptables -F
iptables -t nat -F
```

4. 确认停用 selinux策略

```bash
# 查看是否已经开启
getenforce

# 关闭
setenforce 0

```

> 两项基本安装

```bash
# 1 系统的基本库(省去安装其它工具时候的依赖)
# -y 表示不需要进行确认
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

# /usr：系统级的目录，可以理解为C:/Windows/，/usr/lib理解为C:/Windows/System32。 /usr/local：用户级的程序目录，可以理解为C:/Progrem Files/。用户自己编译的软件默认会安装到这个目录下。/opt：用户级的程序目录，可以理解为D:/Software，opt有可选的意思，这里可以用于放置第三方大型软件（或游戏），当你不需要时，直接rm -rf掉即可。在硬盘容量不够时，也可将/opt单独挂载到其他磁盘上使用。
```

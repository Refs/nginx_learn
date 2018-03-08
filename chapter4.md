 # nginx 深入学习

 ## 动静分离 

 通过中间件将动态请求与静态请求分离。 目的对于服务端而言，分离资源可以减少不必要的请求消耗，较少请求延时；

![](./images/未作动静分离前.png)

 在没有做动态分离之前，一个请求首先回去请求中间件，然后由中间件将请求转发给中间框架，程序框架回去执行请求对应的一套运算逻辑，并通过调用数据库等去获取数据资源，然后返回；每一个请求都要经过这层层关系，现任会对后台的消耗，以及前台的响应时间都会差生影响；

![](./images/动静分离.png)

  那么对于静态请求，其实并不需要经过程序框架，我们通过中间件，直接从硬盘中拖取到这个资源，直接返回给用户就可以了； 

  只有那种不需要缓存的动态请求，交互程度比较高的，才需要去走这四层关系；

  ### nginx作动静分离中间件的场景
  
  ![](./images/tomcat动静分离.png)

tomcat主要负责jsp的所有的动态的请求，另外jpg、png等静态资源 我们利用nginx中间件 将其直接处理掉； 这样就通过nginx 将静态请求与动态请求分离开；



> linux 安装tomcat https://www.cnblogs.com/wangcMove/p/7606051.html

## jdk 的安装

> linux 安装jdk https://www.cnblogs.com/Dylansuns/p/6974272.html

由于各Linux开发厂商的不同,因此不同开发厂商的Linux版本操作细节也不一样,今天就来说一下CentOS下JDK的安装:

1. 在/usr/目录下创建java目录

```bash
[root@localhost ~]# mkdir/usr/java
[root@localhost ~]# cd /usr/java
```

2. 下载jdk,然后解压

```bash
[root@localhost java]# curl -O http://download.Oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.tar.gz 
[root@localhost java]# tar -zxvf jdk-7u79-linux-x64.tar.gz
```

3. 设置环境变量

```bash
[root@localhost java]# vi /etc/profile
```

在profile中添加如下内容:

```bash
#set java environment
JAVA_HOME=/usr/java/jdk1.7.0_79
JRE_HOME=/usr/java/jdk1.7.0_79/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

让修改生效:

```bash
[root@localhost java]# source /etc/profile
```

4.验证JDK有效性

```bash
[root@localhost java]# java -version
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
```

```bash
# linux 查看系统版本信息
lsb_release -a

```


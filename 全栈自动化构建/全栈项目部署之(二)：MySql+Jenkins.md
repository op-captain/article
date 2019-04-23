# 全栈项目部署之(二)：MySql+Jenkins

> Nginx+Nodejs+MySql+Vue+Github+Jenkins

## 背景
希望通过实际的操作，能更深入的了解与体验整个web项目的部署、发开、发布的完整流程

## 实现的目标

最终的实现目标是架构一套完整的前后端分离的web项目。具体的实现如下：

1. 能通过公网域名常访问到网站
2. 接口调用正常
3. 上线发布流程自动化、规范化

## 技术栈

- 后端使用 nodejs
- 静态资源管理 nginx
- 数据库使用 mySql
- 前端使用 vue
- 构建发布自动化 jenkins
- 代码托管使用 git

## 具体实现

### MySql

**1. 查看该操作系统上是否已经安装了mysql数据库**

    
    rpm -qa | grep mysql


**2. 移除CentOS默认的mysql-libs**

    
    yum remove mysql-libs

**3. 清空dbcache**


    yum clean dbcache

**4. 下载MySQL rpm安装包**


    wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
 

**5. 安装下载的rpm文件**。

这个rpm还不是mysql的安装文件，只是两个yum源文件，执行后，在/etc/yum.repos.d/ 这个目录下多出mysql-community-source.repo和mysql-community.repo


    rpm -ivh mysql-community-release-el6-5.noarch.rpm
    
**6. 查看是否已经有mysql可安装文件**


    yum repolist all | grep mysql
    
    
**7. 安装mysql-community-server（选项都用yes）**


    yum install mysql-community-server
    
**8. 启动**


    service mysqld start
    
    ==> Starting mysqld:                                           [  OK  ]
    
**9. mysql的root用户的密码**

默认是空的，所以我们需要及时用mysql的root用户登录（第一次回车键，不用输入密码），并修改密码。这个root账号是mysql的root账号，非Linux的root账号


    mysql -u root
    use mysql;
    
    //这里的分号，一定要加上。因为是sql语句。不然就失败
    update user set password=PASSWORD("这里输入root用户密码") where User='root';
    
    flush privileges; 

**10. 查看mysql是否自启动,并且设置开启自启动命令**


    chkconfig --list | grep mysqld
    chkconfig mysqld on
    
**11. mysql安全设置**

可以不执行这一步操作。直接开始使用数据库。如果执行了这一步的操作。那么远程连接数据库会失败。需要再做处理。处理方式见后面的 *踩坑说明*

    mysql_secure_installation

-   没设置过密码，就直接回车


    In order to log into MySQL to secure it, we'll need the current
    password for the root user.  If you've just installed MySQL, and
    you haven't set the root password yet, the password will be blank,
    so you should just press enter here.
    
    Enter current password for root (enter for none):
    

-   设置一个密码


    Setting the root password ensures that nobody can log into the MySQL
    root user without the proper authorisation.

    Set root password? [Y/n] y
    
- yes,来移除匿名用户


    By default, a MySQL installation has an anonymous user, allowing anyone
    to log into MySQL without having to have a user account created for
    them.  This is intended only for testing, and to make the installation
    go a bit smoother.  You should remove them before moving into a
    production environment.
    
    Remove anonymous users? [Y/n] y
    
- no,允许root远程连接库
    

    Normally, root should only be allowed to connect from 'localhost'.  This
    ensures that someone cannot guess at the root password from the network.
    

    Disallow root login remotely? [Y/n] n
    
- yes，删除测试库


    By default, MySQL comes with a database named 'test' that anyone can
    access.  This is also intended only for testing, and should be removed
    before moving into a production environment.
    
- yes,最后一步让刚才的设置生效，重启配置


    Reloading the privilege tables will ensure that all changes made so far
    will take effect immediately.

    Reload privilege tables now? [Y/n] y



**12. 查看mysql 默认端口号**


    //登录
    mysql mysql -u root -p //输入密码
    
    //查看端口号
    show global variables like 'port'
    
    //查看端口占用的程序 （查看所有netstat -anp）
    netstat -anp|grep 3306
    
**13. 常用命令**

    //进入库
    mysql -u root -p;
    use mysql;
    
    //重启
    service mysqld restart
    
    //启动
    service mysqld start
    
    //配置
    /etc/my.cnf 
    
    //连接
    mysql -h IP -u root -p 密码;


#### 踩坑说明    

- 无法远程连接mysql。

有四种可能，下面文章中有说明

    https://blog.csdn.net/qivan/article/details/79752574
    
    
- mysql没有开启远程连接问题

经过上面第11步的安全配置后，也是需要做如下处理。（执行语句记得加分号“;”结尾）
   
    mysql -uroot -p
    
    Enter password: 
    
    mysql> use mysql
    
    //下面root为用户名，mypasswd为密码 all为全部权限
    mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mypasswd' WITH GRANT OPTION;  
    
    //flush privileges;让我们刚才的操作立即生效
    mysql> flush privileges;  
    

- 安全组策略无法远程连接问题

具体的设置。需要登陆你的云服务控制台，在安全组里面添加响应的端口即可。

![avatar](https://i.loli.net/2019/04/22/5cbd6e570addc.png)
![avatar](https://i.loli.net/2019/04/22/5cbd6f2a41640.png)
![avatar](https://i.loli.net/2019/04/22/5cbd6f3301f25.png)

## Jenkins

**1. 安装JAVA JDK**

- 官网找到下载地址。


    https://www.oracle.com/technetwork/java/javase/downloads/index.html
  
-  进入下载面页

  
![avatar](https://i.loli.net/2019/04/23/5cbe775283753.png)

-  勾选列表上面的“Accept License Agreemen”。选择自己服务器对应的类型64位，还是x86。（如果不能下载或者文件大小下载完只有几K，就注册登录后再去下载）。


![avatar](https://i.loli.net/2019/04/23/5cbe775766997.png)
  
- 上传到服务器

使用scp方式，将下刚才下载的rpm传到服务器

    scp F:\jdk-8u211-linux-x64.rpm root@133.165.138.172:/root

    scp 本地文件 服务器账号@服务器IP : 服务器接收文件的地址
    
- 安装

然后命令直接安装即可（yum执行的是scp刚才传到服器/root目录下的 jdk-8u211-linux-x64.rpm）

    yum install jdk-8u211-linux-x64.rpm
    
    
**2. 安装Jenkins**

-       sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    
    
    
-       sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key


-       yum install jenkins


**3. 配置**

    
    vim /etc/sysconfig/jenkins

    #监听端口
    JENKINS_PORT="8080"
    
**4. 权限**

避免出现各种问题，这里直接使用root

    vim /etc/sysconfig/jenkins

    #修改配置
    $JENKINS_USER="root"

**5. 启动与重启**


    service jenkins restart
    
    service jenkins start
    
    ps -ef | grep jenkins
    
**6. 云服务器安全组添加8080端口规则**

如何添加，见上面mqsql的踩坑说明

    自定议    0.0.0.0/0    tcp:8080    允许   jenkins

    
**7. 访问**

访问jenkins地址 http//xxxxx:8080


### 踩坑说明

-  java-JDK的版本高了

运行service jenkins start后，报了如下问题，大概意意就是jdk的版本过高，我装的是12，而jenkins需要的是8-11。只能用yum remove * 删除重装


        SEVERE: Running with Java class version 56 which is not in the list of supported versions: [52, 55]. Run with the --enable-future-java flag to enable such behavior. See https://jenkins.io/redirect/java-support/
        java.lang.UnsupportedClassVersionError: 56.0
        	at Main.verifyJavaVersion(Main.java:174)
        	at Main.main(Main.java:142)
        
        Jenkins requires Java versions [8, 11] but you are running with Java 12 from /usr/java/jdk-12.0.1
        java.lang.UnsupportedClassVersionError: 56.0
        	at Main.verifyJavaVersion(Main.java:174)
        	at Main.main(Main.java:142)





    



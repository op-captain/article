# web项目部署之(一)：nodejs+nginx

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

### 服务器

**1. 购买云服务器一台。**

腾讯云、阿里云……等等，任选一个。学习使用，越便宜越好。

> 我买的是一个做活动的最基础款的腾讯云服务器。只做学习使用足够。一年费用几十块钱。配置：1 核 2 GB 1 Mbps 50G

**2. 服务器实例的操作系统把选择**

CentOS 6.9 64位

**3.管理员身分登录系统**

默认管理员账户是：root。使用的xShell工具连接并用管理员身分登录到服务器。


### nodeJS

**1. nvm安装**

nvm是一个node的版本管理工具。可以方便是的安装和管理node版本。执行以下命令：

    //更新yum
    yum update
    
    //安装构建工具
    yum groupinstall 'Development Tools'
    
    //安装构建工具时，需要从nvm的官方github存储库获取并执行安装脚本
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.1/install.sh | bash

    //安装完成后，会一个这个样提示,需要关闭重开终端，才能使用nvm
    ==> Close and reopen your terminal to start using nvm
    
    //重新用终端连接服务器后，执行命令。看到输出nvm相关的信息，就说明安装成功
    nvm -v
    
    ==> Node Version Manager……
    
    //输出版本列表
    nvm ls-remote
    
**2. nodejs安装**

而node的安装方法有很多种，源码安装、 yum安装、nvm安装。这里先择nvm来安装nodejs。

    //用刚才安装好的 nvm工具安装nodejs
    nvm install v10.15.3
    
    //查看node版本，输出版本号说明安装成功
    node -v
    
    ==> v10.15.3





### Nginx

**1. 安装nginx**

    //检查环境下有没有安装过低版本的
    rpm -qa|grep nginx
    ==>nginx-release-centos.noarch 0:7-0.el7.ngx
    
    //如果有，先卸载。
    yum remove nginx-release-centos.noarch 0:7-0.el7.ngx
    
    //查看有没有ngin的命令 ，可能是源码安装过的，若有也需要卸载掉
    whereis nginx
    
    //安装nginx
    yum install -y nginx
    
    //查看nginx安装路径, 其中配置文件所在目录 /etc/nginx
    whereis nginx
    ==>nginx: /usr/sbin/nginx /etc/nginx /usr/lib64/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz /usr/share/man/man3/nginx.3pm.gz

    
    //启动 成功会看到 如下输出 [ok]
    service nginx start
    ==>Starting nginx:                                            [  OK  ]
    
    
浏览器中输入服务器的域名或ip,可以看到有如下提示web页面：

*This page is used to test the proper operation of the nginx HTTP server after it has been installed. If you can read this page, it means that the web server installed at this site is working properly.*

到此nginx成功安装，和成功运行全部完成了



---
**2. 踩坑**

第一次安装，是根据网上文的章中第一步添加ngingx的yum源，第二步再进行nginx。按照上面的步骤操作试失败了。报错提示是需要有依赖项没安装。

    Requires: libcrypto.so.10(OPENSSL_1.0.2)(64bit)
    …………

于是赶紧查资料，在某文章中发现这样一段话：“centos 6.9默认的是nginx1.10版本，想安装nginx更高的版本可以下载nginx官网yum源进行安装”。

于是明白了问题所在：*我的centos正好是6.9，可以直接安装nginx1.10版本。只是如果需要安装更高版本的nginx，才需要添加nginx的yum源的。最后使用 yum remove nginx-release-centos.noarch 0:7-0.el7.ngx 移除。再yum install -y nginx。就成功安装了*

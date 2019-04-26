# web项目部署之(四)：jenkins部署 + nginx反向代理

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


## Nginx静态服务器配置

1. 多站点配置


    mkdir /etc/nginx/vhost

2.  创建站点配置文件与反向代理


- 创建配置文件


    vi /etc/nginx/vhost/test1.conf
    
    
-  写入下面配置

创建项目存放目录 /data/www/mock/dist


         server {
            listen       80;
            server_name  localhost;
            
            #charset koi8-r;
            
            #access_log  logs/host.access.log  main;
            
            location / {
                root   /data/www/mock/dist; //静态资源
                index  index.html index.htm;
            }
        
            error_page  404              /404.html;
            
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
        }

        
        
        
        server {
            listen 80;
            server_name localhost/api;
            location /{
                proxy_pass http://127.0.0.1:3000;反向代理
            }
        }
    
    
    
3. 引入站点配置文件


    vim /etc/nginx/nginx.conf
    在http下末尾添加 include /etc/nginx/vhost/*.conf;
    http {
        ...
        include /etc/nginx/vhost/*.conf;
    }
    
4. 重启nginx


    service nginx restart


## jenkins自动化部署

#### Jenkins和业务在不同的服务器

**1. 安装ssh插件**

首先，先在Jenkins上装一个插件Publish Over SSH，我们将通过这个工具实现服务器部署功能。


**2. 生成服务器上的密钥**

        cd ~
        
        ssh-keygen -t rsa
        
**3. 添加公钥**

将生成的公钥添加到需要发布的业务服务器的特定用户的.ssh/authorized_keys文件中。
        
**3. 插件配置**

系统管理-系统设置里找到Publish over SSH这一项。 重点参数说明：

    注意：Path to key 和 Key 二选一填写就可以了

    https://wiki.jenkins.io/display/JENKINS/Publish+Over+SSH+Plugin


    Passphrase：密码（key的密码，没设置就是空）
    Path to key：key文件（私钥）的路径
    Key：将私钥复制到这个框中(path to key和key写一个即可)
    
    SSH Servers的配置：
    SSH Server Name：标识的名字（随便你取什么）
    Hostname：需要连接ssh的主机名或ip地址（建议ip）
    Username：用户名
    Remote Directory：远程目录（上面nginx对应的目录）
    
    高级配置：
    Use password authentication, or use a different key：勾选这个可以使用密码登录，不想配ssh的可以用这个先试试
    Passphrase / Password：密码登录模式的密码
    Port：端口（默认22）
    Timeout (ms)：超时时间（毫秒）默认300000
    
    来源：https://juejin.im/post/5ad1980e6fb9a028c42ea1be
    
#### Jenkins和业务在同一个服务器

将构建立好的项目复制到nginx静态资源目录中，但jenkins用户操作其它目录是需要权限的。所以要先给权限。让jenkins以root或对应用户执行

打开配置文件

    vi /etc/sysconfig/jenkins 
    
1. 修改Jenkins配置文件 改为root


    #Unix user account that runs the Jenkins daemon 
    #Be careful when you change this, as you need to update
    #permissions of $JENKINS_HOME and /var/log/jenkins.
    
    JENKINS_USER="root"
    
2. 修改Jenkins相关文件夹用户权限


    chown -R root:root /var/lib/jenkins/
    chown -R root:root /var/cache/jenkins/
    chown -R root:root /var/log/jenkins/
    
    
3. 重启Jenkins（若是其他方式安装的jenkins则重启方式略不同


    service jenkins restart


4. 给jenkins用户添加相应文件的权限


    chown -R jenkins <path>

> 来源：https://www.jianshu.com/p/fa546f723724
    

    
5. 构建脚本增加删除和复制到指定目录


    npm install
    rm -rf dist
    npm run build
    rm -rf /data/www/mock/
    cp -a dist /data/www/mock/





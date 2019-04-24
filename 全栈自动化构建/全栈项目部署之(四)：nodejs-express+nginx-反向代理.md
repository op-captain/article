# web项目部署之(四)：nodejs-express+nginx-反向代理

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
    


         server {
            listen       80;
            server_name  localhost;
            
            #charset koi8-r;
            
            #access_log  logs/host.access.log  main;
            
            location / {
                root   /data/www/mock; //静态资源
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


# web项目部署之(三)：vue+github+jenkins

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

## Git

**1. 安装**

    yum info git
    
    yum install -y git
    

## Vue

**1. vue-cli新建立项目**

    https://cli.vuejs.org/zh/guide/installation.html

**2. 提交项目到github**

可以使用github、码云、coding、gitlib等任意的代码托管服务平台，这里使用的是github平台

    //git的入门教程
    https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000

## Jenkins

**1. 初始化**

- 浏览器访问jenkins的web地址 http//xxxxx:8080。
- 输入管理员密码，在服务器上用vim打开，网页上提示路径下的文件，复制密码粘贴输入,继续下一步。
- 安装需要的插件，选择推荐安装。

![image](https://i.loli.net/2019/04/23/5cbe819a27ffb.png)

- 创建一个管理员账户。

完成后会显示如下界面。

![image](https://i.loli.net/2019/04/23/5cbea000144e0.png)


**2. 全局添加node构建时，需要的环境**

- 安装node插件。在插件界面，搜索关键字 nodejs 
        

![image](https://i.loli.net/2019/04/23/5cbea47e337e6.png)

- 安装插件说明是如下的插件。安装完成后重启jenkins


    NodeJs
        NodeJS Plugin executes NodeJS script as a build step.


![image](https://i.loli.net/2019/04/23/5cbea47e0a66d.png)

- 【构建环境】的node选项，需要预先全局添加。位置：系统管理->全局工具配置->NodeJs下面。填写的nodejs版本和服务器一至。最下面保存按钮完成。

![image](https://i.loli.net/2019/04/23/5cbea60cae08f.png)

**3. 获取token**

- 进入github ,生成token（我已经生成过一次了，所以上面的提示是红色的按钮，可以点击，再次生成）


    Settings > Developer settings > Personal access tokens
    
![image](https://i.loli.net/2019/04/23/5cbeb4df65c1a.png)

    
**4. github插件的全局配置**

- 系统管理 –> 系统设置 –> GitHub –> Add GitHub Sever

![image](https://i.loli.net/2019/04/23/5cbeb714201ea.png)

- secret填上刚才 在github上面生成的token

![image](https://i.loli.net/2019/04/23/5cbeb72526102.png)


**5. 新建任务**

![image](https://i.loli.net/2019/04/23/5cbeaeaf3b753.png)

![image](https://i.loli.net/2019/04/23/5cbeaec78e5d3.png)

- 项目在github上的仓库URL地址，账号新增一个github的登录账号

![image](https://i.loli.net/2019/04/23/5cbeaed818278.png)

![image](https://i.loli.net/2019/04/23/5cbeaee893e0b.png)

- 选择刚才安装的插件和配置的全局nodejs环境，配置的github插件的token

![image](https://i.loli.net/2019/04/23/5cbeb85e81c3d.png)

- 选择shell

![image](https://i.loli.net/2019/04/23/5cbeaf0d28b0c.png)

说明：安装包，删除之前的静态目录，重新打包构建

![image](https://i.loli.net/2019/04/23/5cbeafde3aafa.png)


## gitHub勾子打通

经过上面的jenkins配置，现在就可以在github上面，通过webhook的配置，来和服务器的jenkins打通，通信。


**1. webhook的配置**

![image](https://i.loli.net/2019/04/23/5cbeb9527c7f0.png)

![image](https://i.loli.net/2019/04/23/5cbeb95346b72.png)

- Payload URL 注意这里的结尾要有 “/”


    http://你的IP：端口/github-webhook/
    
![image](https://i.loli.net/2019/04/23/5cbec08abd2f7.png)

- 成功发送消息给jenkins

![image](https://i.loli.net/2019/04/23/5cbec113317a7.png)

- jenkins收到消息后，自动构建

![image](https://i.loli.net/2019/04/23/5cbec2113a7e0.png)

![image](https://i.loli.net/2019/04/23/5cbec21e31bce.png)



    

### 踩坑说明

- jenkins 忘记密码


    https://blog.csdn.net/jlminghui/article/details/54952148
    
    





# web项目部署之(五)：express+mysql


## mysql
   

**1.连接数据库**

    https://jingyan.baidu.com/article/8065f87fa5917723312498c3.html

**2. workbench工具建库、建表**

- 建立一张用户表，表内的字段有：主键、uid、姓名、性别、年龄、头像

头像avator类型 varchar(10000)  要设置大一点。图片地址很长，太小了，新增操作会失败。

![image](https://i.loli.net/2019/04/25/5cc1098e920a7.png)


## express

1. 本地用nodejs搭建express项目

    http://www.expressjs.com.cn/starter/installing.html
    
2. 配置连接数据库

- conf/db 数据库连接的配置


    // MySQL数据库联接配置
    module.exports = {
    	mysql: {
    		host: '139.199.198.182', 
    		user: 'root',
    		password: '123qwe',
    		database:'mock', // 前面建的user表位于这个数据库中
    		port: 3306
    	}
    };

- crud/user 对表的增删改查的操作


    // 实现与MySQL交互
    var mysql = require('mysql');
    var dbconf = require('../conf/db');
    var _ = require('lodash');
    var sql = require('../sql/user');
    var utils = require('../public/javascripts/utils')
    
    // 使用连接池，提升性能
    var pool  = mysql.createPool(_.extend({}, dbconf.mysql));
    
    
    module.exports = {
    	add: function (param, res, next,callbackResponse) {
    		pool.getConnection(function(err, connection) {
    			var hash = utils.hash()
    			// 建立连接，向表中插入值
    			connection.query(sql.insert, [hash, param.name, param.sex, param.age, param.avator], function(err, result) {
    
    				if(result) {
    					result = {
    						code: 200,
    						msg:'增加成功'
    					};    
    				}
     
    				// 以json形式，把操作结果返回给前台页面
    				callbackResponse(res, result);
     
    				// 释放连接 
    				connection.release();
    			});
    		});
        }
    }

- sql/user    存放sql语句



    var user = {
    	insert:'INSERT INTO user(uid, name, sex, age, avator) VALUES(?,?,?,?,?)',
    	update:'update user set name=?, age=? where id=?',
    	delete: 'delete from user where id=?',
    	queryById: 'select * from user where id=?',
    	queryAll: 'select * from user'
    };
    
    module.exports = user;
    

- 工具方法生成uid的hash


    var crypto = require('crypto');
    
    module.exports = {
        hash:function(){
            var data;
            if(arguments.length === 0){
                var current_date = (new Date()).valueOf().toString();
                var random = Math.random().toString();
                data = current_date + random;
            }else{
                data = arguments[0]
            }
            var current_date = (new Date()).valueOf().toString();
            var random = Math.random().toString();
            return crypto.createHash('sha1').update(data).digest('hex');
        }
        
    }

    
![image](https://i.loli.net/2019/04/25/5cc10c741cbb9.png)

3. 路由接口

- 接口目录用版本结构 api/v1/xxx

![image](https://i.loli.net/2019/04/25/5cc10e5e3a8dc.png)

- 接口编写api/vi/user.js


    var express = require('express');
    var router = express.Router();
    
    var curdUser = require('../../crud/user');
    
    /* */
    router.post('/', function (req, res, next) {
    
        // 获取前台页面传过来的参数
        var param = req.body;
    
        // 向前台返回JSON方法的简单封装
        var jsonWrite = function (res, ret) {
            if(typeof ret === 'undefined') {
                res.json({
                    code:'1',
                    msg: '操作失败'
                });
            } else {
                res.json(ret);
            }
        };
    
        
    
        curdUser.add(param,res,next,jsonWrite)
    
    });
    
    module.exports = router;

**6. 将上面的express项目上传到 github**

    https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000


**7. 在服务器clone 刚才上传的express项目**

- 生成ssh


    https://www.cnblogs.com/horanly/p/6604104.html


- 找到服务器上面的ssh的公钥


    cd /root/.ssh
    
    cat id_rsa.pub 
    

- 复制公钥，去github设置里面，配置ssh

![image](https://i.loli.net/2019/04/25/5cc1123bf0ab9.png)

- 拉取项目


    git clone git@github.com:op-captain/mock.git
    
- 进入mock安装依赖模块

    npm install


**8. 安装pm2工具，管理nodejs服务**

clone完成后安装工具

    npm install -g pm2
    
**9. 启动express**

进入到项目根目录中，用pm2工具启动项目

    pm2 start ./bin/www --watch
    
    
![image](https://i.loli.net/2019/04/25/5cc114cee3c89.png)


![image](https://i.loli.net/2019/04/25/5cc114841287c.png)


**10 开启服务器安全组端口**

由于我的腾讯云服务开启了安全组，所以要增加一个 3000端口的规则

查看端口是否正常

    netstat -tunlp 

**11. 本地测试新增接口**

- 安装postman接口工具


- 发送接口

post类型，接口参数和express里面一样，也是就是和刚才用户表一至

![image](https://i.loli.net/2019/04/25/5cc11ffb57a3e.png)


数据库成功新增一条数据

![image](https://i.loli.net/2019/04/25/5cc126d1e3867.png)


至此nodejs操作数据已全部完成和调通。


## vue

最后一环，前端vue调用接口，通过nginx反向代理到express的接口。

接口请求不是nodejs服务端的3000端口，而是利用nginx的反向代理，请求的nginx静态资源地址

        //接口地址
         public url = "http://www.itokay.cn/api/v1/user";

整个测试请求的代码。vue+typescript的写法。用类的方式写vue组件

        <template>
        <div>
        <h1>测试接口</h1>
        <input @click="addUser" type="button" v-bind:value="content">
        </div>
        </template>
        
        <script lang="ts">
        import { Component, Prop, Vue } from 'vue-property-decorator';
        
        @Component
        export default class HelloWorld extends Vue {
          //接口地址
          public url = "http://www.itokay.cn/api/v1/user";
        
          //接口需要的参数
          public params:object = {
            "name":"Jon",
            "sex":1,
            "age":8,
            "avator":"http://tx.haiqq.com/uploads/allimg/170506/0G9454641-12.jpg"
          }
        
          //fetch参数
          protected option:object = {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(this.params)
          }
        
          //按钮文字
          protected content:string = "调用新增用户接口";
        
          //事件方法
          public async addUser(){
            let result = await fetch(this.url,this.option);
        
            result.json().then((data)=>{
              console.log(data);
            })
          }
        }
        </script>
    
线上项目成功调用接口

![image](https://i.loli.net/2019/04/25/5cc17ee141217.png)
    
    
## 踩坑说明

### 数据格式

application/x-www-form-urlencoded，body需要是a=1&b=2&c=3 格式，无法传递复杂多层次的对象

    //'Content-Type': 'application/json'
    
    fetch('/test/post', {
        headers: {
           "Content-Type": "application/x-www-form-urlencoded" 
        },
        method: 'POST',
        body:  "a=1&b=2&c=3"
            })

    
通过使用application/json ，可以直接传递JSON字符串，post复杂类型

    //"Content-Type": "application/json"
    
    fetch('/test/post', {
            headers: { 
                "Content-Type": "application/json"
            },
            method: 'POST',
            body:  JSON.stringify({a:1,b:{c:2,d:3}})
        })
        
        
### 跨域问题

    The 'Access-Control-Allow-Origin' header contains multiple values '*, *', but only one is allowed
    
原因是在nginx和express里面都设置了。去掉其中一个。我去掉的是nginx

    // express
    app.use(function(req, res, next) {
        res.header("Access-Control-Allow-Origin", "*");
        res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
        next();
    });
    
    
    //nginx
    location / {
       add_header Access-Control-Allow-Origin *;
       add_header Access-Control-Allow-Headers X-Requested-With;
       add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
        
    }






    



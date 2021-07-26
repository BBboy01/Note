# npm yarn命令对比

| npm                          | yarn                     |
| ---------------------------- | ------------------------ |
| npm init -y                  | yarn init -y             |
| npm install 包名 --save      | yarn add 包名            |
| npm uninstall react --save   | yarn remove react        |
| npm install react --save-dev | yarn add react --dev     |
| npm update --save            | yarn upgrade             |
| npm install -g @vue/cli      | yarn global add @vue/cli |

# 相对路径坑爹Tip

在项目中的任何一个地方的相对路径，都是相对于process.cwd()，即在哪一个文件夹启动的项目，process.cwd()就是哪个目录

# 环境变量坑爹Tip

node在环境变量中的值为键值对形式，而每个键所对应的值均为String类型，因此当为node环境变量赋值的时候都会默认转为String

```js
const isProduction = undefined
process.env.isProduction = isProduction
console.log(!!process.env.isProduction, typeof process.env.isProduction)  // true string
```

# Express

## 简易的搭建后台服务

```js
const express = require('express')

const app = express()

app.get('/', (req, res) => {
    res.send('ok')
})

app.listen(3000, () => {
    console.log('success')
})
```

## 静态资源的设置

```js
app.use(express.static("public")); 
```

## 不同类型请求的处理

### 单独处理

```js
app.get('/test', (req, res) => {
    res.send('get')
})

app.post('/test', (req, res) => {
    res.send('post')
})
```

### 路由的复用

```js
app.all('/test', (req, res) => {
    if (req.method === 'GET') {
        res.send('get')
    }
    if (req.method === 'POST') {
        res.send('post')
    }
})
```

### 动态路由

```js
app.get('/test/:id', (req, res) => {
    res.send(req.params.id)  // 通过 :id 的方式动态匹配该路由 通过 req.params.id 获取该值
})
```

## 对请求体中数据的解析与获取

```js
const express = require('express')

const app = express()

// 解析 application/json
app.use(express.json()); 
// 解析 application/x-www-form-urlencoded
// app.use(express.urlencoded({true}));  表示使用第三方qs模块解析 不使用内置的queryString
app.use(express.urlencoded());

app.post('/', (req, res) => {
    res.send(res.body)
})

app.listen(3000, () => {
    console.log('success')
})
```

## 重定向

```js
app.get('/log', (req, res) => {
    res.send('This is log')
})

app.post('/test', (req, res) => {
    res.redirct('/log')
})
```

## 路由

### 为所有路由添加前缀

```js
const Router = require('koa-router')
const router = new Router({prefix: '/api'})

router.get('/userInfo', async ctx => {  // /api/userInfo
    ctx.body = {code: 200, msg: 'ok'}
})
```

### 路由的抽离

```js
// server.js

const express = require('express')
const router = require('./router')

const app = express()

app.use(router)
// 解析 application/json
app.use(express.json()); 
// 解析 application/x-www-form-urlencoded
app.use(express.urlencoded());

app.post('/', (req, res) => {
    res.send(res.body)
})

app.listen(3000, () => {
    console.log('success')
})


// router.js

const express = require('express')

const router = express.Router()

router.get('/', (req, res) => {
    res.send('get')
})

module.exports = router
```

### 处理请求前的钩子函数

```js
// server.js

const { hookFun } = require('./hooks/hook.js')

app.use(hookFun, router)


// hooks/hook.js

function hookFun(req, res, next) {
    console.log('执行router的路由接口前先执行此钩子函数')
    
    if (true) {
        res.send('ok')
    }
    
    next()  // 执行app.use后面的代码
}

module.exports = {
    hookFun
}
```

### 处理请求后的钩子函数

```js
// server.js

const { hookFun } = require('./hooks/hook.js')

router.get('/data', (req, res, next) => {
    res.send('ok')
    next()
})

app.use(router, hookFun)


// hooks/hook.js

function hookFun(req, res) {
    console.log('执行router的路由接口后再执行此钩子函数')
    
    if (true) {
        res.send('ok')
    }
}

module.exports = {
    hookFun
}
```

### 中间件执行顺序原理

```js
function func1 (req, res, next) {
    console.log("func1") // 1
    next()
    console.log(11111111)  // 4
}

router.get("/get_data", (req, res, next) => {
    console.log("get_data")   //2
    // res.send("get_data");
    next()
    console.log(22222222)   // 3
})

function func2 (req, res, next) {
    console.log("func2")   // 5
}

app.use(func1, router, func2)

app.listen(3008, () => {
    console.log(`服务器已经启动，端口为：3008`)
})
```

## 模板渲染

### 过滤器函数

```js
// server.js

const express = require('express')
const path = require('path')
const template = require('art-template')

const app = express()

//第一个参数表示将使用模板引擎文件的后缀名，可以将html改为art,使用模板引擎的文件的后缀名也就需要是html
app.engine('html', require('express-art-template'));
//模板引擎的选项
app.set('view options', {
    debug: process.env.NODE_ENV !== 'production'
});
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'html');

// 过滤器函数配置
template.defaults.imports.filterFun = function(value){
    // value接收
    return value + 'dio'
}

app.get('/list', (req, res) => {
    
    data = {
        name: 'xxx',
        age: '0'
    }
    
    res.render('list', data)  // 第二个参数为给模板传递的数据
})


// views/list.html

/* a html page */
 {{ name | filterFun }}
/* a html page */
```

## Cookie

```js
const cookieParase = require('cookie-parser');
app.use(cookieParase());

// 设置
app.get('set_cookie', (req. res) => {
    res.cookie('name', 'nodejs', {maxAge: 1000*60*60})  // 过期时间 单位：ms
    res.cookie('age', 18)  // 未设置过期时间的会在浏览器关闭时清除
    
    res.send('set ok')
})

// 获取
app.get('get_cookie', (req. res) => {
    let name = req.cookies.name
    let age = req.cookies['age']
    
    res.send(name, age)
})
```

## Session

```js
const express = require('express')
const cookieSession = require('cookie-session')

const app = express()

app.use(cookieSession({
    name: 'my_session',
    keys: ['jcoxzijcoaijdiosajiodoi%&mkoc'],
    maxAge: 1000 * 60 * 60 *24 *2
}))

// 设置
app.get('set_session', (req. res) => {
    req.session['name'] = 'kunosiba'
    req.session['age'] = 12
    
    res.send('set ok')
})

// 获取
app.get('get_session', (req. res) => {
    let name = req.session.name
    let age = req.session['age']
    
    res.send(name, age)
})
```

## 操作MySQL

```js
// db.js

var mysql = require("mysql")
var pool = mysql.createPool({
    host:"localhost",
    user:"root",
    password:"",
    database:"qianduan_test"
})//数据库连接配置

function query(sql,callback){
    pool.getConnection(function(err,connection){
        connection.query(sql, function (err,rows) {
            callback(err,rows)
            connection.release()
        })
    })
}//对数据库进行增删改查操作的基础

exports.query = query


// server.js

const express = require('express');
const app = express();

// 1、引入数据库
const db = require('./db.js');

app.get('/get_data', (req, res) => {
    // console.log(db.query);
    
    // 2、进行数据库查询
    db.query("select * from areas limit 10",(err, data)=>{
        console.log("查完了");
        
        if(err) {
            console.log(err);
        }
        res.send(data);
    });
});

app.listen(3000, () => {
    console.log('Example app listening on port 3000!')
});
```

### 生成MySQL时间对象

```js
const moment = require('moment')

let dataNow = moment(new Date()).format('YYYY-MM-DD HH:mm:ss')
```

### ORM对象-关系映射

- ./db/node-orm/index.js 数据库连接设置

```js
// 数据库连接设置
let orm_config = {
    host: 'localhost',//数据库地址
    port:'3306',
    user: 'root',//用户名，没有可不填
    password: '',//密码，没有可不填
    database: 'qianduan_test'//数据库名称
}
```

- server.js

```js
const db = require("./db/nodejs-orm/index.js");

app.get("/",(req, res)=>{
    let Student = db.model("students");
    Student.find(['name', 'age'], (err,results)=>{
        res.send(results);
    });
})
```

#### 查询方法

```js
// 查询所有
Student.find((err,results)=>{
        res.send(results);
});

// 查询指定字段名
Student.find(['name', 'age'], (err,results)=>{
        res.send(results);
});

// 按条件查询(find、字符串参数)
Student.find("name='小月月'",(err,results)=>{  // 这里的引号里面书写的是where后面的字符串
    res.send(results);
});

// 分页查询(limit、对象参数 where\number\count)
Student.limit({where:"age>18",number:1,count:5},(err,results)=>{
    res.send(results);
});
```

#### 增加数据

```js
// 增加单条记录 insert、对象参数、属性就是列名
Student.insert({name:'赵云',age:20},(err,results)=>{
    res.send("增加成功");
});

// 同时增加多个数据 insert、数组参数、元素为对象、属性就是列名
let arr = [{name:'刘备'}, {name:"关羽"},{name:"张飞"}];
Student.insert(arr,(err,results)=>{
	res.send("增加成功");
});
```

#### 删除数据(物理删除)

```js
// 按条件删除记录 delete、字符串参数
Student.delete('age=0',(err,results)=>{
     res.send("删除成功");
});

// 清空表里面所有内容   delete、无参数
Test1.delete((err,results)=>{
        res.send("删除成功");
});
```

#### 修改数据

```js
// 修改所有学生name为小明   update、对象参数、属性就是列名
let Student = db.model("students");
Student.update({name:'小明'},(err,results)=>{
    console.log(results);
});

// 修改id为1的年龄为30     update、字符串参数1、对象参数2、属性就是列名
let Student = db.model("students");
Student.update('id=1',{age:30},(err,results)=>{
    res.send("修改成功");
});
```

#### 自定义执行sql语句

```js
let Student = db.model("students");
Student.sql('select * from students',(err,results)=>{
    res.send("执行成功");
});
```

### 封装handleDB

- db/nodejs-orm/index.js:

```js
const mysql = require('mysql')
// 数据库连接设置
let orm_config = {
    host: 'localhost',//数据库地址
    port: '3306',
    user: 'root',//用户名，没有可不填
    password: process.env.SQL_PWD,//密码，没有可不填
    database: 'news'//数据库名称
}

let options = {}
let tableSQL = ''
let isConnect = false

function Model (name, option) {
    this.name = name
    this.option = option
};

/**
* @description: 查询数据
* @param {} options：可选参数
* @param {Function} callback :（req,results）=>{}
*/
Model.prototype.find = function (options, callback) {
    if (!isConnect) {
        console.log(options.constructor)
        this.connect(err => {
            isConnect = true
            var str = ''
            if (!callback) {
                str = `select * from ${this.name}`
                callback = options
            } else if (options.constructor == Array) {
                str = `select ${options.join()} from ${this.name}`
            } else if (options.constructor == Object) {
                str = `select ${options.arr.join()} from ${this.name} where ${options.where}`
            } else {
                str = `select * from ${this.name} where ${options}`
            };
            //console.log(str);
            connection.query(str, (error, results, fields) => {
                callback(error, results, fields)
            })
            return this
        })
    } else {

        var str = ''
        if (!callback) {
            str = `select * from ${this.name}`
            callback = options
        } else if (options.constructor == Array) {
            str = `select ${options.join()} from ${this.name}`
        } else {
            str = `select * from ${this.name} where ${options}`
        };
        //console.log(str);
        connection.query(str, (error, results, fields) => {
            callback(error, results, fields)
        })
        return this
    }

}

/**
* @description: 分页查询
* @param {Object} options :   { where:查询条件, number: 当前页数 , count : 每页数量 }
* @return: 
*/
Model.prototype.limit = function (options, callback) {
    var str = ''
    if (!options.where) {
        str = `select * from ${this.name} limit ${(options.number - 1) * options.count},${options.count}`
    } else {
        str = str = `select * from ${this.name} where ${options.where} limit ${(options.number - 1) * options.count},${options.count}`
    };
    console.log(str)
    connection.query(str, (error, results, fields) => {
        callback(error, results, fields)
    })
    return this
}

/**
* @description: 插入数据
* @param {Object} obj:对象或者数组
* @param {Function} callback :（req,results）=>{}
*/
Model.prototype.insert = function (obj, callback) {
    if (!isConnect) {
        this.connect(err => {
            if (err) {
                throw err
            } else {
                connection.query(tableSQL, (error, results, fields) => {
                    if (Array.isArray(obj)) {
                        for (var i = 0; i < obj.length; i++) {
                            this.insertObj(obj[i], callback)
                        }
                    } else {
                        this.insertObj(obj, callback)
                    }
                })

            }
        })
    } else {
        if (Array.isArray(obj)) {
            for (var i = 0; i < obj.length; i++) {
                this.insertObj(obj[i], callback)
            }
        } else {
            this.insertObj(obj, callback)
        }
    }

}

Model.prototype.insertObj = function (obj, callback) {
    let keys = []
    let values = ''
    for (var key in obj) {
        keys.push(key)
        values += `"${obj[key]}",`
    };
    values = values.replace(/,$/, '')
    let str = `INSERT INTO ${this.name} (${keys.join()}) VALUES (${values})`
    connection.query(str, (error, results, fields) => {
        callback(error, results)
    })
}

/**
* @description: 更新数据
* @param {Object} option：可选参数 更新条件
* @param {Object} obj： 修改后的数据 
* @param {Function} callback :（req,results）=>{}
*/
Model.prototype.update = function (option, obj, callback) {
    let str = ''
    if (arguments.length == 2) {
        callback = obj
        obj = option
        str = `UPDATE ${this.name} SET `
        for (var key in obj) {
            str += `${key}='${obj[key]}', `
        };
        str = str.replace(/(, )$/, '')
    } else {
        str = `UPDATE ${this.name} SET `
        for (var key in obj) {
            str += `${key}='${obj[key]}', `
        };
        str = str.replace(/(, )$/, '')
        str += ` where ${option}`
    };

    console.log(str)
    connection.query(str, (error, results, fields) => {
        callback(error, results, fields)
    })
    return this

}

/**
* @description: 删除数据
* @param {Object} option：可选参数 删除条件
* @param {Function} callback :（req,results）=>{}
*/
Model.prototype.delete = function (option, callback) {
    var str = ''
    if (!callback) {
        str = `delete from ${this.name}`
        callback = option
    } else {
        str = `delete from ${this.name} where ${option}`
    };
    console.log(str)
    connection.query(str, (error, results, fields) => {
        callback(error, results, fields)
    })
    return this
}

/**
* @description: 执行sql语句
* @param {String} str : sql语句
* @param {Function} callback :（req,results）=>{}
*/
Model.prototype.sql = function (str, callback) {
    connection.query(str, (error, results, fields) => {
        callback(error, results, fields)
    })
    return this
}

/**
* @description: 删除model表格 （慎用！）
* @param {type} 
* @return: 
*/
Model.prototype.drop = function (callback) {
    connection.query(`DROP TABLE ${this.name}`, (error, results, fields) => {
        callback(error, results, fields)
    })
    return this
}

//连接检测
Model.prototype.connect = function (callback) {
    let p1 = new Promise((resolve, reject) => {
        connection.connect((err) => {
            if (err) {
                //console.log(err.stack);
                //console.log(err);//42000 数据库不存在  28000账号错误
                //console.log(err.sqlState);//42000 数据库不存在  28000账号错误
                reject(err)
            } else {
                resolve()
            }
        })
    })

    p1.then(() => {
        callback(null)
    }, err => {
        if (err.sqlState == 42000) {
            createDatabase(callback)
        } else if (err.sqlState == 28000) {
            callback('数据库账号或密码错误')
        } else {
            callback(err)
        }
    })
}

//创建数据库
let createDatabase = function (callback) {
    let p2 = new Promise((resolve, reject) => {
        connection = mysql.createConnection({
            host: options.host,//数据库地址
            port: options.port,//端口号
            user: options.user,//用户名，没有可不填
            password: options.password,//密码，没有可不填
        })
        connection.connect((err) => {
            //if (err) throw error;
            if (err) {
                reject(err)
            } else {
                resolve()
            }
        })
    })

    let p3 = new Promise((resolve, reject) => {
        connection.query(`CREATE DATABASE ${options.database}`, (err, results, fields) => {
            //if (error) throw error;
            if (err) {
                reject(err)
            } else {
                resolve()
            }

        })
    })

    let p4 = new Promise((resolve, reject) => {
        connection.query(`use ${options.database}`, (err, results, fields) => {
            if (err) {
                reject(err)
            } else {
                resolve()
            }
        })
    })

    let pAll = Promise.all([p2, p3, p4])

    pAll.then(() => {
        callback(null)
    }).catch((err) => {
        callback(err)
    })
}

let orm = {
    /**
    * @description:连接数据库
    * @param {String} host: 主机名 默认localhost
    * @param {Number} port: 端口号 默认3306
    * @param {String} user: 用户名 
    * @param {String} password: 密码 
    * @param {String} database: 数据库名称 默认og
    * @return: 
    */
    connect: function ({ host = 'localhost', port = 3306, user = '', password = '', database = 'og' }) {
        databaseName = database//全局存储当前数据库名称

        options = {
            host,//数据库地址
            port,//端口号
            user,//用户名，没有可不填
            password,//密码，没有可不填
            database//数据库名称
        }
        connection = mysql.createConnection(options)
    },
    /**
    * @description:创建model (表格模型对象)
    * @param {String} name:表格名称
    * @param {Object} options:表格数据结构
    * @return: Model对象：负责数据库增删改查
    */
    model: function (name, options) {
        let str = 'id int primary key auto_increment, '
        for (var key in options) {
            if (options[key] == Number) {
                str += `${key} numeric,`
            } else if (options[key] == Date) {
                str += `${key} timestamp,`
            } else {
                str += `${key} varchar(255),`
            }
        };
        str = str.replace(/,$/, '')
        //console.log(`CREATE TABLE ${name} (${str})`);
        //console.log(str);
        tableSQL = `CREATE TABLE ${name} (${str})`
        return new Model(name, options)
    }
}
orm.connect(orm_config)

module.exports = orm
```

- db/handleDB.js：

```js
const db = require("./nodejs-orm")

function handleDB(res,tableName,methodName,errorMsg, n1, n2){
    //数据库操作
    let Model = db.model(tableName);  // 表映射为模型, 参数是要操作的这个表的表名
    let results;  //results就收查询到的数据
    try{
        results = new Promise((resolve, reject)=>{
            // Model.find("id>=15",(err,data)=>{   //直接调用不封装
            if(!n1){
                Model[methodName]((err,data)=>{    //封装的时候使用这种格式 
                    if(err)reject(err);   // 失败的时候调用reject()
                    resolve(data);    //成功的时候调用resolve() 
                });
                return
            }
            //能够给执行到这里说明n1已经传进来了！
            if(!n2){
                Model[methodName](n1,(err,data)=>{    //封装的时候使用这种格式 
                    if(err)reject(err);   // 失败的时候调用reject()
                    resolve(data);    //成功的时候调用resolve() 
                });
                return
            }
            //能够给执行到这里说明n1和n2已经传进来了！
            Model[methodName](n1,n2,(err,data)=>{    //封装的时候使用这种格式 
                if(err)reject(err);   // 失败的时候调用reject()
                resolve(data);    //成功的时候调用resolve() 
            });               
        }) 
    }catch(err){
        console.log(err); // 给后台看到的
        res.send({errMsg:errorMsg}); //给前端送过去的
        return
    }
    return results
}

module.exports = handleDB
```

- 在项目中：

```js
const handleDB = require("./db/handleDB");

app.get("/", (req,res)=>{
	(async function(){
        //获取参数， 判空
        //数据库操作
        // let results = await Promise对象
        let results = await handleDB(res,"students", "find", "students数据库插入数据出错！");
        //后面我们在项目中操作数据库都可以使用上面这种做法
        //查询到的结果返回页面中去
        res.send(results);
    })();
})
```

## CSRF跨站请求伪造 

- CSRF请求伪造的示意图：

![CSRF攻击过程](./assets/CSRF攻击过程.png)

### CSRF防护思路

1. 请求转账页面的时候，服务器响应转账页面，在cookie中设置一个csrf_token值(随机48位字符串)。

2. 客户端在进行post请求的时候，在请求头中带上自定义的属性'X-CSRFToken' ，值为cookie中的csrf_token值。(要注意的是，此时的post请求，浏览器还会自发带着cookie中csrf_token到服务器。)

3. 服务器在接收到post请求的时候，首先验证响应头中的x-csrftoken值，和cookies中的csrf_token是不是一致，如果不一致，需要return，直接结束处理，不进行后续工作。

### 全局CSRF防护钩子函数

- 配置路由前配置钩子函数即可

```js
const router = express.Router();

router.all("/register",(res,req)=>{
    ....
})

function getRandomString(n) {
    let str = ''
    while(str.length<n){
        str+=Math.random().toString(36).substr(2)
    }
    return str.substr(str.length-n)
}

function csrfProtect(req, res, next){
    let method = req.method
    if(method=="GET"){
        let csrf_token = getRandomString(48);
        res.cookie('csrf_token', csrf_token);
        next() //执行跳转到下一个函数执行，即app.use(beforeReq,router)中的下一个
    }else if(method=="POST"){
        // 判断响应头中的x-csrftoken值，和cookies中的csrf_token进行对比
        console.log(req.headers["x-csrftoken"]);
        console.log(req.cookies["csrf_token"]);
        
        if((req.headers["x-csrftoken"] === req.cookies["csrf_token"])){
            console.log("csrf验证通过！");
            next()
        }else{
            res.send("csrf验证不通过!！");
            return
        }
    }
}

app.use(csrfProtect,router)
```

## 图片验证码创造

- utils/captcha/index.js

```js
// yarn add svg-captcha
var svgCaptcha = require('svg-captcha');

class Captcha{
    getCode(){
        var captcha = svgCaptcha.create({ 
            inverse: false, // 翻转颜色 
            fontSize: 48, // 字体大小 
            noise: 2, // 噪声线条数 
            width: 100, // 宽度 
            height: 40, // 高度 
            size: 4,// 验证码长度
            ignoreChars: '0o1i', // 验证码字符中排除 0o1i
        }); 
        return captcha
    }
}
 
// let captchaObj = new Captcha();
// let captcha = captchaObj.getCode();
// console.log(captcha.text)
// console.log(captcha.text.toLowerCase());
// console.log(String(captcha.data));    //将来要发送给客户端的数据

module.exports = Captcha
```

- routers/passport.js

```js
const router = require('express').Router()
const Captcha = require('../utils/captcha')

router.get('/passport/image_code/:float', (req, res) => {
    let captchaObj = new Captcha()
    let captcha = captchaObj.getCode()

    // captcha.text  // 图片验证码文本
    // captcha.data  // 图片验证码图片内容信息

    // 保存图片验证码文本到session
    req.session['imageCode'] = captcha.text

    res.setHeader('Content-Type', 'image/svg+xml')
    res.send(captcha.data)
})

module.exports = router
```

## 加密及编码

1. 为什么需要加密：

   ​	为了保证我们的数据能够在网络上进行安全的传输

2. 常见加密算法：

   - 不可逆算法：密码散列函数（英语：Cryptographic hash function），又译为加密散列函数、密码散列函数、加密散列函数，是散列函数的一种。它被认为是一种单向函数，也就是说极其难以由散列函数输出的结果，回推输入的数据是什么。这样的单向函数被称为“现代密码学的驮马”。这种散列函数的输入数据，通常被称为消息（message），而它的输出结果，经常被称为消息摘要（message digest）或摘要（digest）。

   - 可逆算法：

     - 对称加密：将信息使用一个密钥进行加密，解密时使用同样的密钥，同样的算法进行解密。

     - 非对称加密：又称公开密钥加密，是加密和解密使用不同密钥的算法，广泛用于信息传输中。

3. 项目中的密码学

   项目中用户身份信息的密码字段都是使用密文进行存储（单项散列函数处理）

   常见的单项散列函数处理方式有：   **md5**     和   **sha256**

   项目中使用双重md5加盐来对密码进行存储前的加密

   ```js
   const md5 = require('md5')
   const salt = 'dkl$UY(*Q&!#Jo))(!A^#1)U($MO0'
   
   let result = md5(md5(password) + salt)
   ```

### Base64

Encoding

```js
// plain-text string
const str = 'Base64 Encoding in Node.js';

// create a buffer
const buff = Buffer.from(str, 'utf-8');

// decode buffer as Base64
const base64 = buff.toString('base64');

// print Base64 string
console.log(base64);

// QmFzZTY0IEVuY29kaW5nIGluIE5vZGUuanM=
```

Decoding

```js
// Base64 encoded string
const base64 = 'QmFzZTY0IEVuY29kaW5nIGluIE5vZGUuanM=';

// create a buffer
const buff = Buffer.from(base64, 'base64');

// decode buffer as UTF-8
const str = buff.toString('utf-8');

// print normal string
console.log(str);

// Base64 Encoding in Node.js
```



### JWT

![jwt](./assets/jwt.png)

#### 对称签名加密

- 缺点：在分布式系统中，每一个子系统都需要获取到密钥，在拿到密钥后这个子系统既可以发布`Token`也可以验证`Token`，但是对于一些资源服务器来说，他们只需要有验证令牌的能力就足够了

---

- 生成token值

```js
const jwt = require('jsonwebtoken');
const salt = 'dkl$UY(*Q&!#Jo))(!A^#1)U($MO0'

// payload                     // 需要加密的内容
// secretOrPrivateKey          // 加密所用的字符串或者private.key密钥文件
// options                     // 参数列表
/*{
	algorithm: 'HS256',    // 加密使用的算法  如果不使用private.key 只能使用HS开头的算法
	expiresIn:'10s'		   // 有效时间 若加密内容不是对象 则无法设置有效时间  可以是数字(s)、字符串 如 60, "2 days", "10h", "7d". 
}*/
const token = jwt.sign({id:1,username:"zhangsan"},salt,{expiresIn: 60 * 60 * 2})
```

- 验证token

```js
const token = "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwidXNlcm5hbWUiOiJ6aGFuZ3NhbiIsImlhdCI6MTU4NjEyNTUwMywiZXhwIjoxNTg2MTMyNzAzfQ.Uwx-EzPq2c9oDJxfs0nCrWLAcTS89HxPBqTUbx91gwY"
try{
    let payload  = jwt.verify(token.replace("Bearer ", ""), salt);   //获取token中的数据(用户信息)
}catch(e){
    console.log("token已经过期")
}
```

> 如果使用了 `koa-jwt`，可以直接调用 `ctx.state.user` 来获取 payload 内容

```js
const jwtAuth = require('koa-jwt')

// 中间件的方式进行验证
router.get('/getUser-token', jwtAuth({ salt }), async ctx => {
    let payload = ctx.state.user
    ctx.body = {
        msg: 'ok',
        data: payload
    }
})
```

- 前端发送

![image-20210704155332932](assets/image-20210704155332932.png)

```js
axios.interceptors.request.use( config => {
    const token = window.localStorage.getItem('token')
        // 判断是否存在token
        if (token) {
            // token存在的话就在每个请求头上加上token
            // Bearer是JWT的认证头部信息
            config.headers.common["Authorization"] = "Bearer " + token
        }
    return config
}, err => {
    return Promise.reject(err)
})
```

#### 非对称签名加密

- 加密方法：`RS256`
- 私钥：用于发布令牌
- 公钥：用于验证令牌

- 使用openssl生成公、私钥：

```bash
openssl  # windows下使用Git Bash来使用openssl
genrsa -out private.key 1024
rsa -in private.key -pubout -out public.key
```

- 实例

```js
const jwt = require('jsonwebtoken');
const fs = require('fs')

const PRIVATE_KEY = fs.readFileSync('./private.key')
const PUBLIC_KEY = fs.readFileSync('./public.key')

// 签发token
router.post('/grant', ctx => {
    const user = {id: 10, name: 'dio'}
    const token = jwt.sign(user, PRIVATE_KEY, {
        expiresIn: 10,
        algorithm: "RS256"
    })
})

// 验证token
router.post('/verify', ctx => {
    const authorization = ctx.headers.authorization
    const token = authorization.replace("Bearer ", "")
    try{
    	const result  = jwt.verify(token, PUBLIC_KEY, { algorithm: ["RS256"] });
        ctx.body = result
    } catch (error) {
        ctx.body = "无效或过期的token"
    }
})
```

### 路由拦截

`koa-jwt` 提供了一个路由拦截的功能，用户拥有 Token 和未拥有 Token 可访问的接口权限不一样。比如，没有登录的用户只能访问 `/login` 和 `/register` 接口，而登录之后才能访问其他接口的地址，我们要在路由匹配前加入如下的中间件：

```js
/* 路由权限控制 */
app.use(jwtKoa({ secret: secret }).unless({
  // 设置login、register接口，可以不需要认证访问
  path: [
    /^\/api\/login/,
    /^\/api\/register/,
    /^((?!\/api).)*$/   // 设置除了私有接口外的其它资源，可以不需要认证访问
  ]
}));
```

> 如果在这一步检查出错，即当一个请求需要携带 Token 但未携带时，会暂停执行之后的所有中间件，但是可以通过设置 `passthrough: true` 来取消这一特性

## 获取前端传递的文件

```js
// pic_router.js
const multer = require('multer')

const upload = multer({ dest: 'public/news/upload/avatar' })  // sign the upload avatar path

router.post('/user/pic_info', upload.single('avatar'), (req, res) => {  // the param of 'avatar' is your name of input tag
    console.log(req.file)
    /**{
            fieldname: 'avatar',
            originalname: '四层板设计流程.png',
            encoding: '7bit',
            mimetype: 'image/png',
            destination: 'public/news/upload/avatar',
            filename: '2ebfb5f2984a4bcc5e403836d85810b5',
            path: 'public\\news\\upload\\avatar\\2ebfb5f2984a4bcc5e403836d85810b5',
            size: 990825
    }*/
    res.send(req.file)
})


// pic.html
<input type="file" name="avatar" class="input_file" id="upload_file" onchange="changepic(this)">
```

## CORS跨域请求

跨域，是指**浏览器不能执行其他网站的脚本**。它是**由浏览器的同源策略造成**的，是浏览器对JavaScript实施的安全限制。

同源策略限制了一下行为：

- Cookie无法读取
- DOM 和 JS 对象无法获取
- Ajax请求发送不出去

**同源是指，域名、协议、端口均为相同**

- 如何处理跨域带来的ajax问题？(解决跨域方案)

1. jsonp —— JSON Padding

2. 设置代理服务器  （正向代理，  反向代理）

3. 后端设置响应头Access-Control-Allow-Origin

```js
res.setHeader("Access-Control-Allow-Origin", "*")
```

### JSONP

- 处理使用ajax代码发起请求外，页面某些标签也会自动发起请求。我们可以利用script标签的src属性，来发起请求。

- jsonp 就是前端利用 script 在页面不刷新的情况下和服务器进行交互一种技术。拿 json 格式的数据去填充一个函数，英语：json with paddding a function 简称：jsonp

  ![跨域和jsonp](./assets/跨域和jsonp.png)

#### 普通模式使用

```html
<!-- frontend -->
<p><span id="sp1"></span>的年龄是<span id="sp2"></span></p>
<script>
    function callback(data){
    	console.log("执行了callback");
        $("#sp1").html(data.name)
        $("#sp2").html(data.age)
    }    
</script>
<!-- jsonp 原理，不会出现跨域-->
<script src="http://localhost:3001/get_data"></script>
```

```js
// backend
const express = require("express");
const app = express();
app.get("/get_data",(req, res)=>{   
    // 按照jsonp原理来响应:
    res.send('callback({name:"node", age:"11"})')
})
app.listen(3001, ()=>{
    console.log(`服务器已经启动，端口为：3001`);
})
```

#### express中应用jsonp

```html
<!-- frontend -->
<script>
    function foo(data){
    	console.log("执行了callback");
        $("#sp1").html(data.name)
        $("#sp2").html(data.age)
    }    
</script>
<script src="http://localhost:3001/get_data?callback=foo"></script>
<!--这里第一个callback是固定，=号后面的callback是我们回调函数的名字 -->
```

```js
// backend
const express = require("express");
const app = express();
app.get("/get_data",(req, res)=>{
    // express封装好的方法jsonp
    res.jsonp({name:"node", age:"11"})
})
app.listen(3001, ()=>{
    console.log('服务器已经启动，端口为：3001');
})
```

### 后端设置响应头

- 后端设置响应头：res.setHeader("Access-Control-Allow-Origin", "*")

- 前端代码无需任何设置

```js
const express = require("express");
const app = express();
app.get("/get_data",(req, res)=>{
    res.setHeader("Access-Control-Allow-Origin", "*")
    res.send({name:"node", age:"11"})
})
```

### 使用cors模块

```js
const cors = require("cors");
app.use(cors())
```

# Koa

> 当response.status(ctx.status)未设置时，Koa会自动将状态设置为**200**或**204(成功但内容为空)**

- Koa 应用程序是一个包含`一组中间件函数的对象`，它是按照类似堆栈的方式组织和执行的。

## 目录结构

```js
└─src

  ├─app      // 服务、数据库、错误处理等的一些配置

  ├─constants   // 全局中需要用到的常量

  ├─controller  // 处理路由匹配后的主要逻辑

  ├─middleware  // 路由中间件，对数据进行处理后交给controller

  ├─router    // 路由

  ├─service    // 操作数据库

  └─utils     // 一些工具方法
```

- 对于一些隐私的值（例如密码、端口号等），使用`dotenv`临时存放于环境变量中，声明于**.env**文件中

## 错误处理

```js
app.use(async ctx => {
  	const isLogin = false
    if (!isLogin) {
        ctx.app.emmit('error', new Error('you have not logged in'), ctx)
    	return
    }
    ctx.body = 'Hello World';
});

app.on('error', (err, ctx) => {
    ctx.status = 401
    ctx.body = err.message
})
```

## 路由中间件

- 安装 koa-router

```js
// 中间件
const Koa = require('koa');
const Router = require('koa-router');
const app = new Koa();

const router = new Router();

router.get("/", function (ctx, next){
    console.log("1111");
    next();
    console.log("2222");
    ctx.body = 'Hello World';
}, function(ctx, next){
    console.log("3333");
    next()
    console.log("4444");
    ctx.body = 'Hello World222';
}, function(ctx, next){
    console.log("5555");
});

app.use(router.routes());
```

### 路由钩子

```js
const Koa = require("koa");
const bodyParser = require("koa-bodyparser");
const router = require("./routers/note");
const koaRouter = require("koa-router")();

const app = new Koa();

async function test(ctx, next) {
  console.log("dio");
  await next();
  console.log("jojo");
}

koaRouter.get('/', ctx => {
    ctx.body = {msg: 'ok'}
})

app.use(test);
app.use(router.routes());

// 判断一个method是否支持
/**
* 当请求get 为正常请求
* 当请求put、delete、patch会报错：Method Not Allowed，状态码：405
* 当请求link、copy、lock会报错：Not Implemented，状态码：501
**/
app.use(koaRouter.allowedMethods());

app.use(bodyParser);
```

## 获取请求参数（GET参数）

```js
const Koa = require('koa');
const app = new Koa();
app.use(async (ctx) => {
    console.log(ctx.method); //获取请求方法
    console.log(ctx.url);    //获取请求URL
    console.log(ctx.query);  //获取解析的查询字符串对象
    console.log(ctx.querystring); //根据 ? 获取原始查询字符串
    console.log(ctx.headers);//获取请求头对象
    console.log(ctx.params);  // 根据动态路由获取参数
    ctx.body = ctx.url;
});
```

## 获取请求体（POST参数）

### 解析`json`、`urlencoded`数据

- 安装 `koa-bodyparser`


```js
const app = new Koa();
const bodyParser = require('koa-bodyparser');

app.use(bodyParser());

router.post("/", function (ctx){
    let post_params = ctx.request.body;
    ctx.body = post_params
});
```

### 解析`form-data`数据(不包含文件)

- 安装`koa-multer`

```js
const app = new Koa();
const multer = require('koa-multer');
const Router = require('koa-router');

const upload = multer();
const router = new Router({prefix: '/upload'});

// app.use(upload.any());  路由可以添加多个中间件 最好写到单独的路由中

router.post("/", upload.any(), (ctx, next) => {
    let formData = ctx.req.body;  // 此处数据需在ctx.req中获取而不在ctx.request
    ctx.body = formData
});
```

### 获取`form-data`中的文件

```js
const app = new Koa();
const multer = require('koa-multer');
const Router = require('koa-router');

const upload = multer({
    dest: './uploads/'  // 文件上传到的路径 如果没有该文件夹会自动创建
});
const router = new Router({prefix: '/upload'});

router.post("/avatar", upload.single('avatar'), (ctx, next) => {
    let fileData = ctx.req.file;
    ctx.body = {msg: 'upload success'}
});

router.post("/picture", upload.array('picture', 9), (ctx, next) => {
    let fileData = ctx.req.files;
    ctx.body = {msg: 'upload success'}
});
```

这两者的组合没什么问题，不过 `koa-multer` 和 `koa-route`（注意不是 `koa-router`） 存在不兼容的问题。

因此使用当使用`koa-route`时用`koa-body`代替（**其实也用不上**）

- app.js

```js
const koaBody = require('koa-body');
const app = new koa();
app.use(koaBody({
  multipart:true, // 支持文件上传
  encoding:'gzip',
  formidable:{
    uploadDir:path.join(__dirname,'public/upload/'), // 设置文件上传目录
    keepExtensions: true,    // 保持文件的后缀
    maxFieldsSize:2 * 1024 * 1024, // 文件上传大小
    onFileBegin:(name,file) => { // 文件上传前的设置
      // console.log(`name: ${name}`);
      // console.log(file);
    },
  }
}));
```

- `koa-body` 的基本参数

| 参数名     | 描述                                              | 类型             | 默认值         |
| :--------- | :------------------------------------------------ | :--------------- | :------------- |
| patchNode  | 将请求体打到原生 node.js 的`ctx.req`中            | Boolean          | `false`        |
| patchKoa   | 将请求体打到 koa 的 `ctx.request` 中              | Boolean          | `true`         |
| jsonLimit  | JSON 数据体的大小限制                             | String / Integer | `1mb`          |
| formLimit  | 限制表单请求体的大小                              | String / Integer | `56kb`         |
| textLimit  | 限制 text body 的大小                             | String / Integer | `56kb`         |
| encoding   | 表单的默认编码                                    | String           | `utf-8`        |
| multipart  | 是否支持 `multipart-formdate` 的表单              | Boolean          | `false`        |
| urlencoded | 是否支持 `urlencoded` 的表单                      | Boolean          | `true`         |
| text       | 是否解析 `text/plain` 的表单                      | Boolean          | `true`         |
| json       | 是否解析 `json` 请求体                            | Boolean          | `true`         |
| jsonStrict | 是否使用 json 严格模式，`true` 会只处理数组和对象 | Boolean          | `true`         |
| formidable | 配置更多的关于 `multipart` 的选项                 | Object           | `{}`           |
| onError    | 错误处理                                          | Function         | `function(){}` |
| stict      | 严格模式,启用后不会解析 `GET, HEAD, DELETE `请求  | Boolean          | `true`         |

- `formidable` 的相关配置参数

| 参数名         | 描述                                         | 类型     | 默认值                  |
| :------------- | :------------------------------------------- | :------- | :---------------------- |
| maxFields      | 限制字段的数量                               | Integer  | `1000`                  |
| maxFieldsSize  | 限制字段的最大大小                           | Integer  | `2 * 1024 * 1024`       |
| uploadDir      | 文件上传的文件夹                             | String   | `os.tmpDir()`           |
| keepExtensions | 保留原来的文件后缀                           | Boolean  | `false`                 |
| hash           | 如果要计算文件的 hash，则可以选择 `md5/sha1` | String   | `false`                 |
| multipart      | 是否支持多文件上传                           | Boolean  | `true`                  |
| onFileBegin    | 文件上传前的一些设置操作                     | Function | `function(name,file){}` |

- 获取文件上传后的信息

```js
router.post('/',async (ctx)=>{
  console.log(ctx.request.files);  // 上传后文件的信息
  console.log(ctx.request.body);  // 其他的表单字段
  ctx.body = JSON.stringify(ctx.request.files);
});
```

> 因为默认开启多个文件上传，因此 `ctx.request.files` 是一个对象，
>
> 而且是通过表单的 `name=photo` 属性作为对象的 key,值便是一个 File 对象，有用的字段如下：

| 字段名           | 描述             |
| :--------------- | :--------------- |
| size             | 文件大小         |
| path             | 文件上传后的目录 |
| name             | 文件的原始名称   |
| type             | 文件类型         |
| lastModifiedDate | 上次更新的时间   |

### Resize图片

- 需要用到`jimp`

```js
const path = require("path");
const Jimp = require("jimp");

Jimp.read(file.path).then((image) => {
    image.resize(1200, Jimp.AUTO).write(`${destPath}-large`);
    image.resize(640, Jimp.AUTO).write(`${destPath}-middle`);
    image.resize(320, Jimp.AUTO).write(`${destPath}-small`);
});
```

## 会话cookie

```js
// 设置和获取cookie
const Koa = require('koa');
const Router = require('koa-router');
const app = new Koa();
let router = new Router();

router.get("/", function (ctx){
    ctx.cookies.set('name', 'nodejs2020', {maxAge: 1000 * 3600});   // 设置cookie
    let cookie_name = ctx.cookies.get('name');   // 获取cookie
    ctx.body = cookie_name
});

app.use(router.routes());

app.listen(3000, ()=>{
    console.log("server is running at port 3000");
});
```

## 会话session

- 安装 koa-session
- 使用

```js
const session = require('koa-session');
app.keys = ['$%^&**%$#%$#&^*&)*&*&^$%#$%$&^&**'];  // 设置签名
app.use(session({maxAge:1000 * 3600}, app));
router.get("/", function (ctx){
    ctx.session.age = 11;    // 设置session
    ctx.body = ctx.session.age   // 获取session
});
```

- 完整

```js
// 设置和获取session
const Koa = require('koa');
const Router = require('koa-router');
const session = require('koa-session');
const app = new Koa();

const router = new Router();
app.keys = ['$%^&**%$#%$#&^*&)*&*&^$%#$%$&^&**'];
app.use(session({maxAge:1000 * 3600}, app));
router.get("/", function (ctx){
    ctx.session.age = 11;

    ctx.body = ctx.session.age
});

app.use(router.routes());

app.listen(3000, ()=>{
    console.log("server is running at port 3000");
});
```

### Redis代替内存存储session

> 登录demo

- 鉴权中间件

```js
// middleware/auth.js

module.exports = async (ctx, next) => {
    if (!ctx.session.userInfo) {
        ctx.body = {
            ok: 0,
            msg: 'user has not logged in'
        }
    } else {
        await next()
    }
}
```

- 逻辑与配置

```js
const Koa = require('koa');
const Router = require('koa-router');
const bodyParser = require('koa-bodyparser')
const session = require('koa-session');
const app = new Koa();
const router = new Router();

const redisStore = require('koa-redis')
const redis = require('redis')
const redisClient = redis.createClient(6379, 'localhost')
const wrapper = require('co-redis')  // promisify
const client = wrapper(redisClient)

app.keys = ['$%^&**%$#%$#&^*&)*&*&^$%#$%$&^&**'];

const SESSION_CONFIG = {
    key: 'bbboy.sess',  // 键名
	store: redisStore({ client })
}

app.use(bodyParser())
app.use(session(SESSION_CONFIG), app)

router.post('/login', async ctx => {
    const {body} = ctx.request
    // 登录逻辑 验证
    
    // 验证OK 设置session
    ctx.session.userInfo = body.username
	ctx.body = {
        ok: 1,
        msg: 'login success'
    }
})

router.post('/logout', async ctx => {
    delete ctx.session.userInfo
    ctx.body = {
        ok: 1,
        msg: 'logout success'
    }
})

router.get('/getUser', require('../middleware/auth.js'), async ctx => {
    ctx.body = {
        ok: 1,
        msg: 'get user info success',
        data: ctx.session.userInfo
    }
})
```

## cookie和session的缺点

- Cookie会被附加在每个HTTP请求中，所以无形中增加了流量（事实上某些请求时不需要的）
- Cookie是明文传输，所以存在安全性问题
- Cookie的大小限制是4KB，对于复杂的需求来说是不够的
- 对于浏览器以外的其他客户端（比如IOS和Android）访问服务器时不会自动携带cookie和session，必须手动的设置
- 对于分布式系统和服务器集群中难以保证其他系统也可以正确的解析session

## GitHub Oauth2 授权登录

- index.html

```html
<div id="app">
    <a href="/github/login">login with github</a>
</div>
```

- index.js

```js
const githubAuthorizationConfig = {
    client_id,
    client_secret
}

router.get('/github/login', async ctx => {
    let dataStr = (new Date()).valueOf()
    // 重定向到认证接口并配置参数
    let authorizationPath = "https://github.com/login/oauth/authorize"
    authorizationPath += `?client_id=${githubAuthorizationConfig.client_id}`
    // 转发到授权服务器
    ctx.redirect(authorizationPath)
})

router.get('/github/callback', async ctx => {
    const code = ctx.query.code
    const params = {
        client_id: githubAuthorizationConfig.client_id,
        client_secret: githubAuthorizationConfig.client_secret,
        code
    }
    let res = await axios.post("https://github.com/login/oath/access_token", params)
    const access_token = querystring.parse(res.data).access_token
    res = await axios.get(`https://api.github.com/user?access_token=${access_token}`)
    ctx.body = `
	<h1>Hello ${res.data.login}</h1>
	<img src="${res.data.avatar_url}" alt=""/>
	`
})
```



## 静态资源中间件

- 安装 koa-static
- 使用

```js
const static = require('koa-static');
const path = require('path')
app.use(static(path.join( __dirname,  'public')))
```

- 完整

```js
// 静态资源中间件
const Koa = require('koa');
const Router = require('koa-router');
const static = require('koa-static')
const path = require('path')
const app = new Koa();
app.use(static(path.join( __dirname,  'public')))

//通过http://localhost:3000/02.png访问到public文件夹下的02.png图片
let router = new Router();

router.get("/", function (ctx){
    ctx.body = "hello"
});

app.use(router.routes());

app.listen(3000, ()=>{
    console.log("server is running at port 3000");
});
```

## 操作数据库

或使用`sequelize`ORM操作

```js
// database.js
const mysql = require("mysql2");

const config = require("./config");

const connections = mysql.createPool({
  host: config.MYSQL_HOST,
  port: config.MYSQL_PORT,
  database: config.MYSQL_DATABASE,
  user: config.MYSQL_USER,
  password: config.MYSQL_PASSWORD,
});

module.exports = connections.promise();


// service.js
const connection = require("../app/database");

async checkResource(tableName, momentId, userId) {
    const statement = `SELECT * FROM ${tableName} WHERE id = ? AND user_id = ?;`;
    const [result] = await connection.execute(statement, [momentId, userId]);

    return result.length === 0 ? false : true;
}
```

## 模板引擎使用

- 安装   art-template 和  koa-art-template

  新建views文件夹，在views中新建login.html

```js
const render = require('koa-art-template');

render(app, {
  root: path.join(__dirname, 'views'),
  extname: '.html',
  debug: process.env.NODE_ENV !== 'production'
});

router.get("/", function (ctx, next){
    ctx.render("login")
})
```

- 完整代码：

```js
// art-template使用
const Koa = require('koa');
const path = require('path');
const Router = require('koa-router');
const render = require('koa-art-template');
const app = new Koa();

let router = new Router();

render(app, {
  root: path.join(__dirname, 'views'),
  extname: '.html',
  debug: process.env.NODE_ENV !== 'production'
});

router.get("/", function (ctx, next){
    ctx.render("login")
});

app.use(router.routes());

app.listen(3000, ()=>{
    console.log("server is running at port 3000");
});
```


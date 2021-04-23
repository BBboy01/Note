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

## 对请求体中数据的解析与获取

```js
const express = require('express')
const bodyParser = require('body-parser');

const app = express()

// 解析 application/json
app.use(bodyParser.json()); 
// 解析 application/x-www-form-urlencoded
app.use(bodyParser.urlencoded());

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

## 路由的抽离

```js
// server.js

const express = require('express')
const bodyParser = require('body-parser');
const router = require('./router')

const app = express()

app.use(router)
// 解析 application/json
app.use(bodyParser.json()); 
// 解析 application/x-www-form-urlencoded
app.use(bodyParser.urlencoded());

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

## 处理请求前的钩子函数

```js
// server.js

function hookFun(req, res, next) {
    if (true) {
        res.send('ok')
    }
    
    next()  // 执行app.use后面的代码
}

app.use(hookFun, router)
```


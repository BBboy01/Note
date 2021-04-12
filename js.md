# 预编译

预编译的时候做的事情：

- 1.创建了 AO 对象（供js引擎自己去访问）
- 2.找形参和变量的声明，作为 AO 对象那个的属性名，值是 undefined
- 3.实参和形参相统一
- 4.找函数声明 会覆盖变量的声明

```js
function fn (a, b) {
    console.log(a)  // function a() { }
    var a = 123  // 123
    console.log(a)  // 123
    console.log(c)  // function c() { }
    function a () { }
    console.log(d)  // undefined
    console.log(b)  // undefined
    var b = function () { }
    console.log(b)  // function() { }
    function c () { }
    console.log(c)  // function c() { }
}
fn(1, 2)


AO: {
    a: undefined -> 1 -> function a() { }
    c: undefined -> 2 -> function c() { }
    b: undefined
    d: undefined
}


// js的逐行解释执行
```

# this

- var s = function g () {}  g是只读的，且只能在函数内部访问

- this 当函数创建的时候(new) this指向当前函数的实例

- 简单的函数声明不能被new

  ```js
  var s = {
      a: function() {
          console.log(1)
      },
      b(){
          console.log(2)
      }
  }
  
  var f = s.a.bind(this)
  new f()  // 1
  
  var p = s.b.bind(this)
  new p()  // error
  ```

- es6简写的箭头函数不能被new

## 构造函数中

函数作为对象的方法被调用(谁调用指向谁 没人调用就指向window)

```js
var name = 222
var a = {
    name: 111,
    say: function () {
        console.log(this.name)
    }
}

var fun = a.say
fun()  // 222 fun.call(window)
a.say()  // 111 a.call(a)

var b = {
    name: 333,
    say: function (fun) {
        fun()
    }
}
b.say(a.say)  // 222  b(a.call(window))
b.say = a.say
b.say()  // 333 b.call(b)
```

## 箭头函数中

固定指向外层代码块 即指向父执行上下文

此处箭头函数本身与say平级以key：value的形式，也就是箭头函数本身所在的对象obj，而obj的父执行上下文是window，因此这里的this指向window

```js
var x = 111
var obj = {
    x: 22,
    say: () => {
        console.log(this.x)
    }
}
obj.say()  // 111
```

```js
var obj = {
    birth: 1990,
    getAge: function () {
        var b = this.birth
        console.log(b)  // 1990
        var fn = () => new Date().getFullYear() - this.birth  // this指向obj对象
        return fn()
    }
}
n = obj.getAge()
console.log(n)  // 31
```

# 深、浅拷贝

- 浅拷贝是创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是内存地址
- 深拷贝是将一个对象从内存中完整的拷贝一份出来，从堆内存中开辟一个新的区域存放新对象，且修改新对象不会影响到原对象

| js的拷贝 | 和原数据是否指向同一对象 |  第一层数据为一般数据类型  | 第一次数据不是一般数据类型 |
| :------: | :----------------------: | :------------------------: | :------------------------: |
|   赋值   |            是            |  改变会使原始数据一同改变  |  改变会使原始数据一同改变  |
|  浅拷贝  |            否            | 改变不会使原始数据一同改变 |  改变会使原始数据一同改变  |
|  深拷贝  |            否            | 改变不会使原始数据一同改变 | 改变不会使原始数据一同改变 |

# 防抖和节流

防抖:**指触发事件后在 n 秒内函数只能执行一次，如果在 n 秒内又触发了事件，则会重新计算函数执行时间。**

```js
function debounce(callback, time) {
            let timer
            return () => {
                if (timer) clearTimeout(timer)
                timer = setTimeout(callback, time)
            }
        }

fun = () => {
    console.log('Wryyy')
}

let debounceFun = debounce(fun, 1000)

document.getElementById('input').addEventListener('keyup', debounceFun)
```

节流:**指连续触发事件但是在 n 秒中只执行一次函数。**节流会稀释函数的执行频率。

```js
function throttle(func, wait) {
            let timeOut
            return () => {
                if (!timeOut) {
                    timeOut = setTimeout(() => {
                        func()
                        timeOut = null
                    }, wait)
                }
            }
        }

function func() {
    console.log('olololol')
}

document.getElementById('button').onclick = throttle(func, 1000)
```



# ES6新特性

## let const

### let 和 var的区别

- 不存在变量提升

  ```js
  console.log(c)  // undefined
  var c = 1
  console.log(a)  // 报未声明的错
  let a = 2
  ```

- 同一个作用域下不能重复定义同一个名称

  ```js
  var a = 1
  var a = 100
  console.log(a)  // 100
  let b =1
  let b =100  // 报重复定义的错
  ```

- 有着严格的作用域

  var:函数作用域 let:块级作用域

  ```js
  function fn() {
      var n = 10
      if (true) {
          var n = 100
      }
      console.log(n)
  }
  fn()  // 100
  
  function fn() {
      let n = 10
      if (true) {
          let n = 100
      }
      console.log(n)
  }
  fn()  // 10
  ```

### const

当声明的对象是基本类型时，该对象的值不可改变；当声明的对象是引用类型时，可以改变其内部的数据

```js
const a = 100
a = 10  // error

const r  // error 一定要初始化，不能只声明不赋值

const obj = {}
obj.name = 'me'
console.log(obj.name)  // 'me'
```

## 数据结构

### set

类似于数组 成员是唯一的

```js
const s = new Set()
s.add(3).add(2).add(3)
console.log(s)  // {3,2}

// 去重
var arr1 = [1, 2, 3, 2, 3, 5, 6, 4, 2]
var arr2 =  [...new Set(arr1)]
console.log(arr2)  // [1, 2, 3, 5, 6, 4]
```

### map

类似于对象

```js
const m = new Map()
m.set('name', 'dio').set('age', 18)
for (let [key, value] of m) {
    console.log(key, value)
}
```

# 时间循环机制

js中的异步操作 比如fetch setTimeout setInterval 压入到调用栈中的时候里面的消息会进入到消息队列中去 消息队列中会等到调用栈清空之后再执行

```js
function fun1() {
    console.log(1)
}
function fun2() {
    setTimeout(() => {
        console.log(2)
    }, 0)
    fun1()
    console.log(3)
}
fun2()

// 将 console.log(2) 压入消息队列中 -> 1 -> 3 -> 2
// 1 3 2
```

promise async await 的异步操作的时候会加入到微任务中去 会在调用栈清空的时候立即执行 微任务中的任务会比消息队列中的任务先执行  调用栈中加入的微任务会立即执行

```js
var p = new Promise(resolve => {
    console.log(4)
    resolve(5)
})
function func1() {
    console.log(1)
}
function func2() {
    setTimeout(() => {
        console.log(2)
    }, 0)
    func1()
    console.log(3)
    p.then(res => {
        console.log(res)
    })
}

// 4 -> 将console.log(2) 压入消息队列中 -> 1 -> 3 -> 将 console.log(res) 压入微任务中
// 4 1 3 5 2
```

# 单例

定义

- 只有一个实例
- 可以全局访问

主要解决:

- 一个全局使用的类
- 频繁的创建和销毁

何时使用:

- 当想控制实例的数目
- 当想节省系统化资源的时候

如何实现:

- 判断系统是否已经有这个单例 如果有则返回 没有则创建

单例模式的优点:

- 内存中只要一个实例
- 减少了内存的开销 尤其是频繁的创建和销毁实例(比如说是首页页面的缓存)

使用场景:

- 全局的缓存
- 弹窗

```js
let createLoginLayer = (function() {
    let div
    return () => {
        if (!div) {
            div = document.createElement('div')
            div.innerHTML = '这是弹窗'
            div.style.display = 'none'
            document.body.append(div)
        }
        return div
    }
})()

document.getElementById('button').onclick = () => {
    let loginLayer = createLoginLayer()
    loginLayer.style.display = 'block'
}
```




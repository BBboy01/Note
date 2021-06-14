## 组件是什么？类是什么？类被编译成什么？

- 组件指的是页面的一部分
- 类是一些功能的集合
- 类最后会被编译成函数

## 什么样的数据应该被放在Vuex中？

一般来说，为了各页面以及组件数据的独立，`store`中只存放**共享的数据**，但是这样却不利于整体维护和管理。由于各种父子传值以及组件自身的数据还有获取数据的各种方法过多，当一些页面的数据出现问题的时候，很难定位到底是哪个数据的问题，调用方法也不统一。因此从工程化的角度来说，应该将整个项目需要获取的数据全部都放到`store`中进行统一的管理。对于根据大、小项目来决定是否使用`Vuex`并不正确，项目最初可能确实只有几个页面，但是后期难免不会因为需求的增加而逐渐变成所谓的大项目。

## rem与em自适应

`rem`：根据html里的`font-size`来计算

`em`：根据父元素的`font-size`来计算

```html
html{ font-size: 50px; }
div{ height: 2rem; }
```

## 利用冒泡捕获机制

需求：给每一个访问的用户添加一个属性 `banned=true` ，此用户点击页面上的任何按钮或者元素都不可以相应原来的函数而是直接**alert**提示

```javascript
window.addEventListener('click', e => {
    if (banned === 'true') {
        e.stopPropagation()
    }
}, true)  // 捕获模式 默认为false
```

事件委托的场景经常会使用到冒泡机制

## Promise方法

`Promise.all()`

- 传入数组，数组里可以不全是`promise`对象
- 返回按输入数组顺序的结果的数组，如果其中有一个promise rejected，则只返回一个rejected的值，但是其他的promise正常执行

`Promise.race()`

- 传入数组，数组里可以不全是`promise`对象
- 返回输出数组中最先完成的promise的结果

`Promise.resolve()`

- 将传入的任何数据转换为`promise`对象

## React中事件调用顺序

- root originalCapture
  - 父元素 reactCapture
  - 子元素 reactCapture
  - 父元素 originalCapture
  - 子元素 originalCapture
  - 子元素 originalBubble
  - 父元素 originalBubble
  - 子元素 reactBubble
  - 父元素 reactBubble
- root  originalBubble
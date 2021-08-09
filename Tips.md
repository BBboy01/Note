## 将数组中`false、undefined、null、0`之类的隐式为`false`的类型的数据过滤掉

```js
const arr = [true, "aaa", null, "bbb", undefined, "ccc", false, "ddd", 0]
console.log(arr)  // [true, "aaa", null, "bbb", undefined, "ccc", false, "ddd"]

const newArr = arr.filter(Boolean)
console.log(newArr)  // [true, "aaa", "bbb", "ccc", "ddd"]
```

## js判断当前页面是否被切换

可以判断电脑桌面的切换、同一个浏览器中标签的切换，然而只要是非全屏状态的电脑应用切换不会触发

```js
window.addEventListener("visibilitychange",function(){
　　console.log(document.hidden)
    console.log(document.visibilityState)
});
```

| document.hidden          | document.visibilityState |
| ------------------------ | ------------------------ |
| true:表示被隐藏，不可见  | hidden:表示隐藏状态      |
| false:表示未被隐藏，可见 | visible:表示是可见状态   |

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

## flex布局中的参数

- flex为`flex-grow、flex-shrink、flex-basic`的缩写，默认值为 0 1 auto
  - flex: auto (1 1 auto)
  - flex: 0 (0 0 0)
  - flex: 1 (1 1 auto)
  - flex: none (0 0 auto)

- flex-grow
  - 当父元素flex方向为row时子元素没占满父元素宽度时
    - flex-grow默认为0，表示该子元素不自适应增长宽度使其占有剩余宽度
    - flex-grow为1，表示该子元素宽度自适应占据剩余宽度
    - 计算规则：所有子元素中flex-grow的值加起来表示剩余宽度被分多少份，子元素自身的flex-grow的值表示占据的份数
- flex-shrink
  - 当父元素flex方向为row时子元素占满父元素宽度或累加宽度超过父元素宽度时
    - flex-shrink默认为1，表示该子元素自适应缩减宽度
    - flex-shrink为0，表示该子元素保持自身宽度
    - 计算规则：父元素宽度 - flex-shrink为0的元素 = X ，flex-shrink不为0的元素的原本总宽度 Y ， flex-shrink不为0的元素的个数为要压缩的份数 M , flex-shrink不为0的元素的自身的宽度为 C ，Y - X = Z为需要shrink沾满的宽度，C - Z / M = L 为此时该元素的宽度
- flex-auto
  - 为auto时计算该子元素的宽度
  - 为0时不计算该子元素的宽度
- order
  - 表示该元素的权重，值越小权重越高，排列时权重大的在前面

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

## 关于React为什么使用JSX来渲染

JSX是`React.createElement`的语法糖

- 约束性相比于模板更小
- 更符合HTML书写习惯
- 相比于模板不会引入太多的概念和语法

## 关于setState

### 异步or同步

当有更新时，React中会先创建一个`update`并通过`enqueueUpdate`将本次更新加入到当前`fiber`的更新链表中

- 17版本以前：React中会根据当前执行的上下文来判断当前情况下是同步的还是异步的

> react为同步模式，在事件处理函数里`setState`更新是**批量**的，或者说是**异步**
>
> 但是如果`setState`在`setTimeout`、`setInterval`、`addEventListener`等等中的回调中，或者在`setState`本身的回调中执行，更新就是同步的
>
> 如果想在`setTimeout`中实现批量更新，需要用`batchedUpdates`包裹才可以

- 17版本以后：

> ​	react为并发模式，无论在哪`setState`都是批量异步执行的

### 数据的合并

当使用`setState`时，如果更新状态中只返回了需要修改的数据，不需要修改的数据也会保留，因为源码中使用的`Object.assign({}, this.state, { message: 'Thanks for your back' })`来更新状态

```jsx
this.state = {
    message: 'Hello World',
    name: 'coder'
}

changeState(){
    this.setState({
        message: 'Thanks for your back'
    })
}

// 此时 state状态为{ message: 'Hello World', name: 'coder' }
```

### 本身的合并

源码中，有一个`do while`循环，每一次从当前`fiber`的更新链表中取出下一个更新执行，即多次执行`getStateFromUpdate`，而此方法会拿到前一次的值和当前传入的值使用`Object.assign({}, prevState, partialState)`来进行合并，然而此时的`prevState`一直为`{ counter: 0 }`，即不论执行多少次都是在执行`Object.assign({}, {counter: 0 }, {counter: 1 })`

```jsx
this.state = {
    counter: 0
}

changeState(){
    this.setState({
        counter: this.state.counter + 1
    })
    this.setState({
        counter: this.state.counter + 1
    })
    this.setState({
        counter: this.state.counter + 1
    })
}

// 此时 state状态为{ counter: 1 }
```

当`setState`中的参数为`function`时，则会执行该方法，并传递前一次的值返回给`partialState`

```jsx
changeState(){
    this.setState((prevState) => {
        return {
            counter: prevState.counter + 1
        }
    })
    this.setState((prevState) => {
        return {
            counter: prevState.counter + 1
        }
    })
}

// 此时 state状态为{ counter: 2 }
```

## React在diff时的情况

### 情况一：对比不同类型的元素

- 当节点为不同的元素，React会拆卸原有的树，并且建立起新的树

  - 当一个元素从`<a>`变成`<img>`，从`<Article>`变成`<Comment>`，或从`<Button>`变成`<div>`都会触发一个完整的重建流程
  - 当卸载一颗树时，对应的DOM节点也会被销毁，组件实例将执行`componentWillUnmount()`方法
  - 当建立一颗新的树时，对应的DOM节点会被创建以及插入到DOM中，组件实例将执行`componentWillMount()`方法，紧接着执行`componentDidMount()`方法

- 例如如下代码

  - React会销毁`Counter`组件并且重新装载一个新的组件，而不会对`Counter`进行复用

  ```jsx
  // from
  <div>
      <Counter />
  </div>
  
  // to
  <span>
      <Counter />
  </span>
  ```

### 情况二：对比同一类型的元素

- 当对比两个相同类型的React元素时，React会保留DOM节点，仅比对及更新有改变的属性

- 例如如下代码

  - 通过比对这两个元素，React知道只需要修改DOM元素上的`className`属性

  ```jsx
  <div className="before" title="stuff" />
  <div className="after" title="stuff" />
  ```

- 例如如下代码

  - 当更新`style`属性时，React仅更新有所改变的属性
  - 通过对比这两个元素，React知道只需要修改DOM元素上的`color`样式，无需修改`fontWeight`

  ```jsx
  <div style={{color: 'red', fontWeight: 'bold'}} />
  <div style={{color: 'green', fontWeight: 'bold'}} />
  ```

- 如果时同类型的组件元素：

  - 组件会保持不变，React会更新该组件的`props`，并调用`componentWillReceiveProps()`和`componentWillUpdate()`方法，接着调用`render()`方法，`diff`算法将在之前的结果以及更新的结果中进行递归

### 情况三：对子节点进行递归

- 在默认条件下，当递归DOM节点的子元素时，React会同时遍历两个子元素列表；当产生差异时，生成一个`mutation`

  > 前面两个比较是完全相同的，所以不会产生`mutation`，最后一个比较，产生一个`mutation`，将其插入到新的DOM树中即可

  ```jsx
  <ul>
  	<li>first</li>
      <li>second</li>
  </ul>
  
  <ul>
  	<li>first</li>
      <li>second</li>
      <li>third</li>
  </ul>
  ```

- 但是如果是在中间插入一条数据

  > React会对每一个子元素产生一个`mutation`，而不是保持`<li>1</li>`和`<li>2</li>`不变，这种低效的比较方式会带来一定的性能问题

  ```jsx
  <ul>
  	<li>1</li>
      <li>2</li>
  </ul>
  
  <ul>
      <li>3</li>
  	<li>1</li>
      <li>2</li>
  </ul>
  ```

**keys的优化**

1. 方式一：在最后位置插入数据，这种情况有无`key`意义不大
2. 方式二：在前面插入数据，在没有`key`的情况下所有的`li`都需要进行修改

- 当子元素拥有`key`时，React使用`key`来匹配原有树上的子元素以及最新树上的子元素
- `key`的注意事项：
  - `key`在当前队列中应该是唯一的
  - `key`不要使用随机数（随机数在下一次render时会重新生成一个数字，导致`key`全部变更进而全部修改）
  - 不要使用`index`作为`key`，这样是没有性能优化的

## React更新

- 流程 ：`props/state`改变 -> `render`函数重新执行 -> 产生新的DOM树 -> 新旧DOM树进行`diff` -> 计算出差异进行更新 -> 更新到真实的DOM

- 问题：父组件状态改变，子组件尽管并未依赖父组件所修改的状态，但是却也跟着重新执行`render`
- 解决方法（类组件中）：
  - React源码中会检查是否实现了`shouldComponentUpdate`函数，若未实现则默认返回`true`，即需要更新，而在子组件中实现`shouldComponentUpdate`函数则会调用该函数，该函数需返回`boolean`值，`true`表示跟着父组件一起更新，`false`表示不更新。
  - 而子组件在实现的时候不继承`React.Component`，而是继承`React.PureComponent`时，React中`PureComponent`会绑定一个类属性`isPureReactComponent = true`，因此在检查的时候会执行`shallowEqual`对`oldProps`、`newProps`和`oldState`、`newState`进行浅层比较，如果相等则返回`false`，即不更新。不使用深层比较是为了性能考虑
- 函数组件中：
  - 使用`memo`包裹该函数组件

## express和Koa处理异步

express中`next`的实现只返回了一个普通函数，所以对于获取异步请求结果并不方便

Koa中`next`的实现是返回`Promise`的，所以可以`await next()`来等待异步调用有结果了再往下执行

## display为none时的样式

当display为none时为元素添加的所有样式都不会生效，因此通过display:none控制元素动画时当元素消失的时候不会执行相应的动画，建议通过元素的取消挂载来实现消失时的动画。并且animation属性不会继承，如果不想覆盖，可以在需要添加元素外包裹一层为其添加动画

## 前端项目在git操作之前执行命令

需要使用到`Husky`，安装`yarn add husky`

```json
// package.json

"husky": {
    "hooks": {
      "pre-commit": "yarn test:nowatch && yarn lint",
      "pre-push": "yarn test:nowatch && yarn lint"
    }
}
```


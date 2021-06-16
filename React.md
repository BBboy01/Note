## HOC(高阶组件)

Higher-Order Components就是一个函数，传给它一个组件，它返回一个新的组件。

```jsx
const NewComponent = higherOrderComponent(YourComponent)
```

比如，我们想要我们的组件通过自动注入一个版权信息。

```jsx
// withCopyright.js 定义一个高阶组件
import React, { Component, Fragment } from 'react'

const withCopyright = (WrappedComponent) => {
  return class NewComponent extends Component {
    render() {
      return (
        <Fragment>
          <WrappedComponent />
          <div>&copy;版权所有 千锋教育 2019 </div>
        </Fragment>
      )
    }
  }
}
export default withCopyright
```

```jsx
// 使用方式
import withCopyright from './withCopyright'

class App extends Component {
  render () {
    return (
  		<div>
        <h1>Awesome React</h1>
        <p>React.js是一个构建用户界面的库</p>
      </div>
  	)
  }
}
const CopyrightApp = withCopyright(App)
```

这样只要我们有需要用到版权信息的组件，都可以直接使用withCopyright这个高阶组件包裹即可。

在这里要讲解在CRA 中配置装饰器模式的支持。

## useCallback 记忆函数

在类组件中，我们经常犯下面这样的错误：

```jsx
class App {
    render() {
        return <div>
            <SomeComponent style={{ fontSize: 14 }} doSomething={ () => { console.log('do something'); }}  />
        </div>;
    }
}
```

这样写有什么坏处呢？一旦 App 组件的 props 或者状态改变了就会触发重渲染，即使跟 SomeComponent 组件不相关，**由于每次 render 都会产生新的 style 和 doSomething（因为重新render前后， style 和 doSomething分别指向了不同的引用）**，所以会导致 SomeComponent 重新渲染，倘若 SomeComponent 是一个大型的组件树，这样的 Virtual Dom 的比较显然是很浪费的，解决的办法也很简单，将参数抽离成变量。

```jsx
const fontSizeStyle = { fontSize: 14 };
class App {
    doSomething = () => {
        console.log('do something');
    }
    render() {
        return <div>
            <SomeComponent style={fontSizeStyle} doSomething={ this.doSomething }  />
        </div>;
    }
}
```

在类组件中，我们还可以通过 this 这个对象来存储函数，而在函数组件中没办法进行挂载了。所以函数组件在每次渲染的时候如果有传递函数的话都会重渲染子组件。

```jsx
function App() {
  const handleClick = () => {
    console.log('Click happened');
  }
  return <SomeComponent onClick={handleClick}>Click Me</SomeComponent>;
}
```

> 这里多说一句，一版把**函数式组件理解为class组件render函数的语法糖**，所以每次重新渲染的时候，函数式组件内部所有的代码都会重新执行一遍。所以上述代码中每次render，handleClick都会是一个新的引用，所以也就是说传递给SomeComponent组件的props.onClick一直在变(因为每次都是一个新的引用)，所以才会说这种情况下，函数组件在每次渲染的时候如果有传递函数的话都会重渲染子组件。

而有了 useCallback 就不一样了，你可以通过 useCallback 获得一个记忆后的函数。

```jsx
function App() {
  const memoizedHandleClick = useCallback(() => {
    console.log('Click happened')
  }, []); // 空数组代表无论什么情况下该函数都不会发生改变
  return <SomeComponent onClick={memoizedHandleClick}>Click Me</SomeComponent>;
}
```

老规矩，第二个参数传入一个数组，数组中的每一项一旦值或者引用发生改变，useCallback 就会重新返回一个新的记忆函数提供给后面进行渲染。

这样只要子组件继承了 PureComponent 或者使用 React.memo 就可以有效避免不必要的 VDOM 渲染。
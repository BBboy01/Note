# 数组更新检测

Vue将被侦听的数组的变更方法进行了包裹，所以它们也会触发视图更新：

- push()
- pop()
- unshift()
- shift()
- splice()
- sort()
- reverse()

上面这些方法会直接修改原数组，但是某些方法会生成新的数组而不改变原数组：

- filter()
- slice()
- concat()

# v-model中的修饰符

- `.lazy`修饰符

默认情况下，v-model再进行双向绑定时绑定的是`input`事件，因此输出的内容和绑定的内容会一直保持同步

当使用`v-model.lazy="xxx"`时，会将绑定的事件变为`change`事件，只有在提交时才会触发（例如 回车）

- `.number`修饰符

默认情况下v-model绑定的值在输入的时候会变为`str`类型

当使用`v-model.number="xxx"`时，会将输入的内容从最开始的连续数字部分截取转化为`number`类型

- `.trim`修饰符

可以删除输入值前后的空格

# props接收值

当接收对象为`Object`类型时如果要设置默认值，需要转换成函数调用的形式，原因类似vue2中`data`的设置也是函数调用

```js
props: {
	message: {
		type: String,
		required: true
	},
        
    code: {
        type: Number,
		default: 200
    },
	
	req: {
		type: Object,
		default() {
			return {msg: "ok"}
		}
	},
    
    // 自定义验证函数
    status: {
        // 这个值必须是下面状态中的其中一个
        validator(value) {
            return ["success", "warning", "danger"].includes(value)
        }
    }
}
```

# eventBus

`yarn add mitt`

定义全局事件总线

```js
// eventBus.js
import mitt from "mitt"

const emitter = mitt()
// 也可以定义多个总线
// export emitter2 = mitt()
// export emitter3 = mitt()

export default emitter
```

触发总线中的事件

```js
import emitter from "eventBus.js"

emitter.emit("event", {msg: "params"})
```

监听总线中的事件

```js
import emitter from "eventBus.js"

emitter.on("event", (param) => {
    console.log(param)  // {msg: "params"}
})

// 监听所有事件
emitter.on("*", (type, param) => {
    console.log(type, param)  // event {msg: "params"}
})
```

取消事件监听

```js
import emitter from "eventBus.js"

// 取消所有事件
emitter.all.clear()

function foo () {}
emitter.on("event", foo)
emitter.off("event", foo)
```

# 作用域插槽

当插槽中的元素需要展示插入组件中的数据时，由于数据作用域问题而不能直接访问

插槽组件`ShowNames`

```vue
<template>
	<div>
        <template v-for="(item, index) in names" :key="index">
			<slot :item="item" :index="index"></slot>
		</template>
    </div>
</template>

names: {
	type: Array,
	default: () => {}
}
```

插槽元素

```vue
<show-names>
	<template v-slot:default="slotProps">
    	<button>
            {{ slotProps.item }} - {{ slotProps.index }}
        </button>
    </template>
</show-names>
```

# 动态组件

在不使用路由的情况下，根据一些条件来动态渲染组件

```vue
<component :is="currentComponent"></component>


components: {
	Home,
	Index,
	About
}

currentComponent = ""
tabs = ["home", "index", "about"]
```

# KeepAlive组件缓存

Vue默认切换组件的时候每次都会对组件进行创建挂载和卸载操作，同时组件的状态也被重置，因此当某些组件需要保持状态或者需要频繁切换时，可以使用`keep-alive`

|  属性   |           类型            |                             描述                             |
| :-----: | :-----------------------: | :----------------------------------------------------------: |
| include | string \| RegExp \| Array |                  只有名称匹配的组件会被缓存                  |
| exclude | string \| RegExp \| Array |               任何名称匹配的组件都不会进行缓存               |
|   max   |     number \| string      | 最多可以缓存多少组件。当达到这个值时，缓存组件中最近没有被访问到的组件会被销毁 |

`inculde`和`exclude`都可以使用 逗号分隔字符串、正则表达式或一个数组 来表示

**匹配首先检查组件自身的`name`属性**

```vue
<keep-alive>
	<component :is="currentComponent"></component>
</keep-alive>

<keep-alive include="home, about">
	<component :is="currentComponent"></component>
</keep-alive>


components: {
	Home,
	Index,
	About
}

currentComponent = ""
tabs = ["home", "index", "about"]
```

# 缓存组件切换时的生命周期

当对组件进行缓存后，组件的`created`和`unmounted`生命周期都不会再触发

而当想要在缓存组件切换的时候进行操作时，可以使用`activated`和`deactivated`生命周期来监听

# 代码分包

通常在对Vue项目进行打包的时候，只会生成两个js文件（`app.js`、`chunk-hash.js`），也就意味着一个项目中所有的组件以及所有的自定义的函数都会打包到`app.js`中，极大的加长了首屏加载时间

分包则是基于`webpack`对于`import`语法的特殊处理

```js
import("./utils/math").then(res => {
    res.random()
})
```

Vue中想要对组件进行分包处理`defineAsyncComponent`

```js
import { defineAsyncComponent } from 'vue'
import LoadingComponent from './LoadingComponent.vue'
import ErrorComponent from './ErrorComponent.vue'

const AsyncHomeComponentMethod1 = defineAsyncComponent(() => import("./Home.vue"))
const AsyncHomeComponentMethod2 = defineAsyncComponent({
	loader: () => import("./Home.vue"),
    loadingComponent: LoadingComponent,  // 当异步组件未加载出来的时候所展示的占位组件
    errorComponent: ErrorComponent,  // 当异步组件加载失败的时候所显示的组件
    delay: 2000  // 如果在 delay 时间后异步组件还未加载完，就显示 loadingComponent
})
```

# 动画

`trasition`组件

## 过度动画基本使用

```vue
<transition name="dio">
	<button>show me</button>
</transition>

<style>
	.dio-enter-from,
    .dio-leave-to {
        opcity: 0;
    }
    
    .dio-enter-to,
    .dio-leave-from {
        opacity: 1;
    }
    
    .dio-enter-active,
    .dio-leave-active {
        transition: opacity 2s ease;
    }
</style>
```

## css动画基本使用

```vue
<transition name="dio">
	<div>show me</div>
</transition>

<style>
    .dio-enter-active {
        animation: bounce 1s ease;
    }
    
    .dio-leave-active {
        animation: bounce 1s ease reverse;
    }
    
    @keyframe bounce {
        0% {
            transform: scale(0);
        }
        50% {
            transform: scale(1.2);
        }
        100% {
            transform: scale(1);
        }
    }
</style>
```

## 当插入或删除包含`transition`组件中的元素时，Vue会做如下操作：

1. 自动检测`style`标签中是否对相应的类添加相应的css过度动画，如果有，则会在恰当的时机添加或删除类名
2. 如果`transition`组件提供了Javascript钩子函数，这些钩子函数将会在恰当的时机被调用
3. 如果没有找到Javascript钩子函数以及css过度动画，则DOM的插入、删除操作会立即执行

![过渡动画生命周期](https://v3.cn.vuejs.org/images/transitions.svg)

-  `v-enter-from`：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的下一帧移除。
- `v-enter-active`：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，在元素被插入之前生效，在过渡/动
  画完成之后移除。这个类可以被用来定义进入过渡的过程时间，延迟和曲线函数。
- `v-enter-to`：定义进入过渡的结束状态。在元素被插入之后下一帧生效(与此同时v-enter-from 被移除)，在过渡动画完成之后移除。
- `v-leave-from`：定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。
- `v-leave-active`：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数。
- `v-leave-to`：离开过渡的结束状态。在离开过渡被触发之后下一帧生效(与此同时v-leave-from 被删除)，在过渡动画完成之后移除

## 当多个元素切换时需要显示动画时，可以通过`mode`属性来设置先结束的动画

- `in-out`: 进入的动画结束后再执行离开的动画
- `out-in`: 离开的动画结束后再执行进入的动画

```vue
<transition name="dio" mode="out-in">
	<h2 class="title" v-if="isShow">
        Hello World
    </h2>
    <h2 v-else>
        How are you
    </h2>
</transition>
```

## 如果需要当第一次展示页面的时候动画生效，可以添加`appear`属性

```vue
<transition name="dio" mode="out-in" appear>
	<h2 class="title" v-if="isShow">
        Hello World
    </h2>
    <h2 v-else>
        How are you
    </h2>
</transition>
```

## 自定义动画类

- enter-from-class
- enter-active-class
- enter-to-class
- leave-from-class
- leave-active-class
- leave-to-class

```vue
// 以 Animate.css 为例
<transition enter-active-class="animate__animated animate__fadeInDown"
            leave-active-class="animate__animated animate__flipInY">
	<h2 v-if="isShow">
        Hello Wrold
    </h2>
</transition>

<style>
    .animate__flipInY {
        animation-direction: reverse;
    }
</style>
```

## Javascript钩子动画

- @before-enter="beforeEnter"
- @enter="enter"
- @after-enter="afterEnter"
- @before-leave="beforeLeave"
- @leave="leave"
- @afterLeave="afterLeave"

## 结合GSAP使用

```vue
<template>
  <div class="app">
    <div><button @click="isShow = !isShow">显示/隐藏</button></div>

    <transition @enter="enter"
                @leave="leave"
                :css="false">  // 跳过css检测，使css动画失效
      <h2 class="title" v-if="isShow">Hello World</h2>
    </transition>
  </div>
</template>

<script>
  import gsap from 'gsap';

  export default {
    data() {
      return {
        isShow: true,
      }
    },
    methods: {
      enter(el, done) {
        console.log("enter");
        gsap.from(el, {
          scale: 0,
          x: 200,
          onComplete: done
        })
      },
      leave(el, done) {
        console.log("leave");
        gsap.to(el, {
          scale: 0,
          x: 200,
          onComplete: done
        })
      }
    }
  }
</script>

<style scoped>
  .title {
    display: inline-block;
  }
</style>
```

## 为列表添加增、删动画`transition-group`

使用`transition-group`有如下的特点：
- **它不会渲染一个元素的包裹器，但是你可以指定一个元素并以`tag attribute`进行渲染**
- 过渡模式不可用，因为我们不再相互切换特有的元素，即`mode`属性
- 内部元素总是需要提供唯一的key attribute 值
- CSS 过渡的类将会应用在内部的元素中，而不是这个组/容器本身

```vue
<template>
  <div>
    <button @click="addNum">添加数字</button>
    <button @click="removeNum">删除数字</button>
    <button @click="shuffleNum">数字洗牌</button>

    <transition-group tag="p" name="why">  // 以 p 标签包裹
      <span v-for="item in numbers" :key="item" class="item">
        {{item}}
      </span>
    </transition-group>
  </div>
</template>

<script>
  import _ from 'lodash';

  export default {
    data() {
      return {
        numbers: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9],
        numCounter: 10
      }
    },
    methods: {
      addNum() {
        // this.numbers.push(this.numCounter++)
        this.numbers.splice(this.randomIndex(), 0, this.numCounter++)
      },
      removeNum() {
        this.numbers.splice(this.randomIndex(), 1)
      },
      shuffleNum() {
        this.numbers = _.shuffle(this.numbers);
      },
      randomIndex() {
        return Math.floor(Math.random() * this.numbers.length)
      }
    },
  }
</script>

<style scoped>
  .item {
    margin-right: 10px;
    display: inline-block;
  }

  .why-enter-from,
  .why-leave-to {
    opacity: 0;
    transform: translateY(30px);
  }

  .why-enter-active,
  .why-leave-active {
    transition: all 1s ease;
  }

  .why-leave-active {
    position: absolute;
  }

  // 对剩余元素添加动画
  .why-move {
    transition: transform 1s ease;
  }
</style>
```

# :data-index属性

为元素添加`dataset`属性，并将值赋给该属性

```vue
<template v-for="(item, index) in arr">
	<h1 :data-index="index">
    	Hello World
	</h1>
</template>

console.log(el.dataset.index)
```




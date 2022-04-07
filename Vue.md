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

# v-model

- `.lazy`修饰符

默认情况下，v-model再进行双向绑定时绑定的是`input`事件，因此输出的内容和绑定的内容会一直保持同步

当使用`v-model.lazy="xxx"`时，会将绑定的事件变为`change`事件，只有在提交时才会触发（例如 回车）

- `.number`修饰符

默认情况下v-model绑定的值在输入的时候会变为`str`类型

当使用`v-model.number="xxx"`时，会将输入的内容从最开始的连续数字部分截取转化为`number`类型

- `.trim`修饰符

可以删除输入值前后的空格

使用自定义修饰符时会向子组件中传入 `modelModifiers` 的 `props`：

```typescript
// 父组件
<child-component v-mode.diy="callback" v-mode:title.dio="callback2"/>

// 子组件
type PropsType = {
  modelModifiers?: {
    diy: boolean
  },
  titleModifiers?: {
		dio: boolean
	}
}

const propsData = defineProps<PropsType>()
```

`v-model` 是 `props` 和 `emits` 的语法糖，默认值为：

|       | Vue2  |       Vue3        |
| :---: | :---: | :---------------: |
| props | value |    modelValue     |
| emits | input | update:modelValue |

`v-mode` 同时可以自定义 `props` 和 `emits` 的名称：`v-mode:name=callback`，子组件中 `props: name`，`emits: update:name`

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

## attrs

当给组件传递自定义属性时，未在组件的 `props` 中声明该属性，则该属性会被添加到组件的 `arrts` 属性中，并且是**非响应式的**

## ctx.expose

当子组件中的 `setup` 中返回了一些组件的方法，在父组件中，可以使用 `ref` 的方式获取到组件实例，从而可以通过该实例**直接调用子组件的所有方法**，然而这是可以被避免的：

在子组件 `setup` 的第二个参数 `ctx` 中有 `expose` 方法，使用该方法可以指定哪些方法可以直接原名绑定到当前组件实例中，可以传递数组：`expose: ['method1', 'method2']`

在 `<script setup>` 语法中，子组件的方法是默认关闭的，即无法在父组件中通过 `ref` 的方式直接访问，需要通过 `defineExpose` 方法进行暴露：`defineExpose({method1, method2})`

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
<!-- 以 Animate.css 为例 -->
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



# watch的使用

- `watch`需要侦听特定的数据源，并在回调函数中执行副作用

- 默认情况下是惰性的，只有当被侦听的数据发生改变的时候才会执行回调
- 与`watchEffect`相比，`watch`允许我们：
  - 懒执行（第一次不会直接运行）
  - 更具体的说明哪些状态发生改变时，触发侦听器的执行
  - 访问侦听状态变化前后的值（`watchEffect`访问不到）

```js
const info = reactive({name: "dio", age: "21"})
const slogan = ref("The World")

const changeInfo = () => {
    info.name = "jojo"
    slogan.value = "pinjiaku"
}

// 1.传入一个reactive对象 获取到的newVal, oldVal都是reactive对象 直接传reactive对象监听不到oldVal
watch(info, (newVal, oldVal) => {
    console.log(newVal, oldVal)  // Proxy {name: "jojo", age: "21"}  Proxy {name: "jojo", age: "21"}
})
watch(() => ({ ...info }), (newVal, oldVal) => {
    console.log(newVal, oldVal)  // {name: "jojo", age: "21"}  {name: "dio", age: "21"}
})

// 2.传入一个函数
watch(() => info, (newVal, oldVal) => {
    console.log(newVal, oldVal)  // jojo  dio
})

// 3.传入Array对象
watch([() => ({ info }), slogen], ([newInfoVal, newSloganVal], [oldInfoVal, oldSloganVal]) => {
	console.log(newInfoVal, newSloganVal, oldInfoVal, oldSloganVal)
    // {name: "jojo", age: "21"} pinjiaku  {name: "dio", age: "21"} The World
})
```

# Not Found匹配路由

```js
{
    path: "/:pathMatch(.*)",
    component: () => import("../pages/NotFound.vue")
}
```

`/:pathMetch(.*)*`会将路径按`/`分割放到params数组中

# `router-link`自定义`tag`

默认情况下，`router-link`下的元素在渲染的时候是`a`标签，之前可以通过`tag="xxx"`来设置渲染的标签类型，但是在`vue-router`4以后不再支持此方式，改为`slot`的方式，使得内容更加灵活，甚至支持组件

```vue
<!-- props: href  跳转的链接 -->
<!-- props: isActive  是否处于激活状态 -->
<!-- props: isExecActive  是否处于精确的激活状态 -->
<!-- props: navigate  导航函数 -->
<router-link to="/home" v-slot="props" custom>
	<strong @click="props.navigate">link 1</strong>
	<div>link 2</div>
</router-link>
```

# `router-view`获取当前匹配的组件

```vue
<router-view v-slot="props">
	{{ props.Component }}
</router-view>
```

- 为组件切换添加动画

```vue
<router-view v-slot="props">
    <transition name="dio">
		<component :is="props.Component"/>    
    </transition>
</router-view>

<style>
	.dio-enter-from,
    .dio-leave-to {
        opacity: 0;
    }
    
    .dio-enter-to,
    .dio-leave-from {
        opacity: 1;
    }
    
    .dio-enter-active,
    .dio-leave-active {
        transition: opacity 1s ease;
    }
</style>
```

# 动态添加路由

```js
const categoryRoutes = {
    path: "/category",
    component: () => import("../pages/Category.vue")
}

// 添加一级路由
router.addRoute(categoryRoutes)

// 添加二级路由，以`name`查找
router.addRoute("home", {
    path: "category",
    component: () => import("../pages/Category.vue")
})
```

# `mapState`在`CompositionAPI`中的使用

`mapGetters`类似

`mapState`中返回的对象在使用时会调用`this.$store`来访问`store`中的数据

```js
import { useStore, mapState } from "vuex"

setup() {
    const store = useStore()
    const storeFuncs = mapState(["counter", "name", "age"])
    
    const storeState = {}
    Object.keys(storeFuncs).forEach((fnKey) => {
        const fn = storeFuncs[fnKey].bind({$store: store})
        storeState[fnKey] = computed(fn)
    })
    
    return {
        ...storeState
    }
}
```

# Vuex中各个方法接收的参数

- `getters`:	  state, getters
- `mutations`:  state, payload
- `actions`:      context, payload  context: { commit, dispatch, getters, rootGetters, state, rootState }

# 环境变量的注入

在项目根目录中添加`.env.development`、`.env.production`、`.env.production`，其中`NODE_ENV`、`BASE_URL`、`VUE_APP_xxx`可以通过`webpack.DefinePlugin`注入

```yaml
VUE_APP_BASE_URL=https://bbboy.org/api
```

使用

```js
console.log(process.env.VUE_APP_BASE_URL)  // https://bbboy.org/api
```


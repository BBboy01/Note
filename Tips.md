## 安装未向苹果付费认证的软件

```bash
brew install --cask $application-name --no-quarantine
```

## 获取安装时使用的包管理器类型

当使用 `npm install` `yarn install` `pnpm install` 时，会在环境变量中创建对应包管理器的环境变量

- yarn

```json
{
  "npm_config_registry": "https://registry.yarnpkg.com",
  "npm_execpath": "/usr/local/lib/node_modules/yarn/bin/yarn.js",
  "npm_config_user_agent": "yarn/1.22.11 npm/? node/v16.13.2 darwin arm64"
}
```

- npm

```json
{
  "npm_config_metrics_registry": "https://registry.npmjs.org/",
  "npm_execpath": "/opt/homebrew/lib/node_modules/npm/bin/npm-cli.js",
  "npm_config_user_agent": "npm/8.5.5 node/v16.13.2 darwin arm64 workspaces/false"
}
```

可以看到 `npm_config_user_agent` 的对应的值的第一项表明了当前安装时所使用的包管理器，然后就可以通过 `process.env.npm_config_user_agent` 获取到当前使用的包管理器了

package.json 中的 preinstall 钩子函数只会在退出状态码不为 0 的时候退出执行，`only-allow` 库的实现原理也就显而易见了

## **使用**`rem`时获取屏幕宽度

```js
document.documentElement.style.fontSize =
  document.documentElement.clientWidth / 37.5 + "px";
// 37.5 是因为设计稿通常是以 iphone6 设计的，iphone6 的宽度为 375px
// 屏幕宽度为 375 的时候， 1rem = 10px
// 屏幕宽度为 441 的时候， 1rem = 11.76px
// 这样就实现了屏幕越大像素值越大，完成了适配
```

## 获取物理像素

```js
const screenWidth = screen.width;
const screenHeight = screen.height;
const dpr = window.devicePixelRatio;
const physicalWidth = screenWidth * dpr;
const physicalHeight = screenHeight * dpr;
```

## CSS 选择器优先级

![specifishity](https://gitee.com/RealBBboy/mark-down-images-repo/raw/master/NoteImg/specifishity.png)

## `load`和`ready`的区别

```js
document.addEventListener("load", () => {
  // 页面资源加载完才会执行，包括图片、视频等
});
document.addEventListener("DOMContentLoaded", () => {
  // DOM渲染完即可，此时图片、视频可能还没有加载完
});
```

## new 对象时进行的操作

1. 在函数中获取到该函数的(例)`Foo.prototype`对象（默认有`constructor: Foo`属性且`enumerable: false`，函数创建时便已经产生）
2. 构造函数内部的`this`指向该对象
3. 执行函数内部的代码（函数体代码）
4. 如果构造函数没有返回非空对象，则返回创建出来的新对象

## 使用正则注意需要嵌套量词的正则

例如 `/^(\d+\s?)+$/.test('362183621897539402476478z')`，看似再普通不过的正则然而浏览器却会因为**其嵌套的量词匹配进行递归处理**，很容易造成性能问题

```js
(362 183621897539402476478)z
(362 18362189753940247647)(8)z
(362 1836218975394024764)(78)z
(362 1836218975394024764)(7)(8)z
(362 183621897539402476)(478)z
(362 183621897539402476)(47)(8)z
(362 183621897539402476)(4)(7)(8)z
   .
   .
   .
(3)(6)(2)( )(1)(8)(3)(6)(2)(1)(8)(9)(7)(5)(3)(9)(4)(0)(2)(4)(7)(6)(4)(7)(8)z
```

解决：

- 换一种写法 `/^(\d+\s+)*\d+$/`
- 使用环视避免回溯 `/^(?:(?=(\d+))\1\s?)*$/`

```js
const regReg = /\/[^\/]+\/g?(?=[,;\.])/;

/dsjahio/g;
/djsaok/.test(hdas);
dhsaoi.replace(/dshai/g, dhsao);
```

## CSS 百分比相对的目标

- 相对父元素：`width` `height`(`position: absolue/fixed` 时相对祖先定位元素)
- 相对父元素宽度：`margin` `padding`
- 相对自身：`translate`

## `webpack` 自动导入模块(vue2 环境)

`require.context(path, withChildPath, fileNameReg)`

- path: String 需要导入模块的目录
- withChildPath: Boolean 是否包含子目录
- fileNameReg: RegExp 需要导入模块的文件名的正则

```js
const requireComponents = require.context(
  "./modules",
  false,
  /\.(vue|js|jsx|ts|tsx)$/,
);
// requireComponents.keys()  // 返回匹配到的文件名
const components = {};
requireComponents.keys().forEach((filePath) => {
  components[
    _.upperFirst(
      _.camelCase(
        filePath.replace(/\.(vue|js|jsx|ts|tsx)/, "").replace("/.//", ""),
      ),
    )
  ] = requireComponents(filePath).default; // 以大驼峰式组件名对应模块中默认导出的内容
});

export default components;
```

## 添加锚点

原理：改变 URL 的 hash 值时不会刷新页面，同时可以监听到 `hashchange` 的事件

```js
window.addEventListener(
  "hashchange",
  function (e) {
    console.log(e.oldURL);
    console.log(e.newURL);
  },
  false,
);
```

- 通过点击 `a 标签`

```html
<a href="#a01">页面滚动到 id = 'a01' 的元素的位置</a>

<p>练习1</p>
<p id="a01">练习2</p>
<p>练习3</p>
```

- 通过 `a 标签` 的 name 属性直接改变 URL 的 hash 值

```html
<!-- bbboy.top#dio -->
<p>练习1</p>
<p>练习2</p>
<a name="dio"></a>
<p>练习3</p>
<p>练习4</p>
<p>练习5</p>
<p>练习6</p>
```

- 通过标题标签 `h1 h2 ...` 的 id 属性直接改变 URL 的 hash 值

```html
<!-- bbboy.top#dio -->
<p>练习1</p>
<p>练习2</p>
<h1 id="dio"></h1>
<p>练习3</p>
<p>练习4</p>
<p>练习5</p>
<p>练习6</p>
```

## 子域名共享父域名中的 Cookie

由于浏览器为了保护用户安全而建立的 CORS 协议，一般情况下违反了该协议时发送的跨域请求会被浏览器拦截，但是当两个域名处于父子的关系时，通过后端设置响应头 `Set-Cookie: name=value; domain=topDomain.com`，即设置 `domain` 字段为顶级域名时，子域名可以共享父域名的 Cookie

![顶级/二级/三级域名&父/子域名](https://i0.hdslb.com/bfs/article/ace3488847dc77e77cc167985312bc54213ece40.png@900w_261h_progressive.webp)

- 有「二级域名」 能读取设置了domain为顶级域名或者自身的cookie，不能读取其他二级域名domain的cookie。例如：要想cookie在多个二级域名中共享，需要设置domain为顶级域名，这样就可以在所有二级域名里面获取到这个cookie的值了
- 「顶级域名」 只能获取到domain设置为顶级域名的cookie，domain设置为其他子级域名的无法获取

## 隐式转换与显式转换

### Number([val])

- 一般用于浏览器的隐式转换中
  - 数学运算
  - isNaN 检测
  - == 比较
- 规则
  - 字符串转换为数字：空字符串变为 0，如果出现任何非有效数字字符，结果都是 NaN
  - 把布尔转换为数字：true -> 1，false -> 0
  - null -> 0，undefined -> NaN
  - Symbol 无法转换为数字，并且会报错
  - BigInt 去除 'n' （超过安全数字的，会按照科学计数法处理）
  - 把对象转换为数字的方法：
    - 如果对象存在 `Symbol.toPrimitive` 方法，则先调用该方法
    - 如果对象的 `valueOf` 方法获取到的值是原始值，则直接转换
    - 调用对象的 `toString` 方法转换为字符串，再把字符串基于 `Number` 转换为数字

#### parseInt([val], [radix]) parseFloat([val])

- 一般用于手动转换
- 规则：
  - [val] 的值必须是一个字符串，如果不是则先转换为字符串
  - 然后从字符串左边第一个字符开始找基于 [radix] 合法的数字字符，把找到的所有的有效数字字符按照 [radix] 转换为数字（如果没有则返回 NaN）
  - 遇到一个非有效数字字符，不论后面是否还有有效数字字符都不再接着找
  - parseFloat 可以多识别一个小数点

```js
const arr = [27, 2, 0, "0013", "14px", 123, 0022];
const res = arr.map(parseInt); // [27, NaN, 0, 1, 1, 38, 1]
/**
 * 27 -> parseInt(27, 0) -> parseInt(27, 10) -> 17
 * 2 -> parseInt(2, 1) -> parseInt('', 1) -> NaN
 * 0 -> parseInt(0, 2) -> 0
 * '0013' -> parseInt('0013', 3) -> parseInt('001', 1) -> 1
 * '14px' -> parseInt('14px', 4) -> parseInt('14', 4) -> 1
 * 123 -> parseInt(123, 5) -> 38
 * 0022 -> parseInt(0022, 6) -> parseInt(parseInt(parseInt(0022, 8), 10), 6) -> 1
 */
```

### Srting([val])

- 转换规则：
  - 拿字符串包起来
  - 特殊：Object.prototype.toString
- 出现情况：
  - String([val]) 或者 [val].toString()
  - '+' 除数学运算外，还可能表示字符串拼接
  - 如果属于字符串拼接，则按照 `Symbol.toPrimitive -> valueOf -> toString` 的规则转化
    - 其中 Symbol.toPrimitive 方法会传入 `string、number、default` 三种参数，根据当前进行的是加法还是字符串拼接决定
    - 有两边，一边是字符串
    - 有两边，一边是对象
    - 只出现在左边

```js
// for example the following code
value1 + value2;
// is equivalent to
toPrimitive(value1) + toPrimitive(value2);
// where toPrimitive is
function toPrimitive(val) {
  if (isPrimitive(val)) return val;
  if (isObject(val)) {
    let v = val.valueOf();
    if (isPrimitive(v)) return v;
  }
  let s = val.toString();
  if (isPrimitive(s)) return s;
  throw new TypeError();
}

// while in a typed language, add will simply be several hardware instructions
```

```js
const result =
  100 + true + 21.2 + null + undefined + "Tencetn" + [] + null + 9 + false;
console.log(result); // 'NaNTencentnull9false'
```

### Boolean([val])

- 转换规则：除了 `0 NaN '' null undefined` 为 false，其余的都是 true

### "==" 比较

- "==" 相等，两边数值类型不同，需要先转为相同类型，然后再进行比较
  - 对象 == 字符串，对象转字符串 `Symbol.toPrimitive -> valueOf -> toString`
  - null == undefined -> true **null/undefined 和其他任何值都不相等**
  - null === undefined -> false
  - 对象 == 对象 比较的是堆内存的地址
  - NaN == NaN -> false
  - 除了以上情况，只要两边的类型不一致，剩下的都是转换为数字，然后再进行比较的
  - === 绝对相等，如果两边类型不同则直接就是 false，不会转换数据类型

## `position: fixed` 的位置 `top、bottom、left、right`

当元素的 `position` 属性为 `fixed` 时，会创建层叠上下文，此时该元素的位置 `top、bottom、left、right` 是[相对于祖先中第一个属性 `transform || perspective || filter` 不是 `none` 的元素进行定位的](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position#fixed)

## `Array.fill` 对于非基础数据类型

对于非基础类型，直接使用 `fill` 填入，会是相同的地址引用。
建议改为 `Array(5).fill().map(_ => [])`

## `performance` 中的各种时间以及指标监控

```javascript
const {
  fetchStart,
  connectStatr,
  connectEnd,
  requestStart,
  resonseStart,
  responseEnd,
  domLoading,
  domInteractive,
  domContentLoadedEventStart,
  domContentLoadedEventEnd,
  loadEventStart,
} = performance.timing;

const time = {
  connectTime: connectEnd - connectStart, // 连接时间
  ttfbTime: responseStart - requestStart, // 首字节到达时间
  responseTime: responseEnd - reponseStart, // 响应的读取时间
  parseDOMTime: loadEventStart - domLoading, // DOM 解析时间
  DOMContentLoadedTime: domContentLoadedEventEnd - domContentLoadedEventStart, // DOM 加载时间
  timeToInteractiveTime: domInteractive - fetchStart, // 首次可交互时间
  loadTime: loadEventStart - fetchStart, // 完整的加载时间
};

// FMP 所对监控的元素需要有 `elementtiming= meaningful` 的属性

let FMP, LCP;

new PerformanceObserver((entryList, observer) => {
  const perfEntry = entryList[0];
  FMP = perfEntry;
  observer.disconnect();
}).observe({ entryTypes: ["element"] }); // 观察页面中的意义元素

new PerformanceObserver((entryList, observer) => {
  const perfEntry = entryList[0];
  LCP = perfEntry;
  observer.disconnect();
}).observe({ entryTypes: ["largest-contentful-paint"] }); // 观察页面中的z元素
```

## 监听错误

`Promise` 错误

如果未在 `reject` 里指定信息，则 `PromiseResult -> message` 会提示具体错误信息，否则只会在 `PromiseResult` 中显示 `reject` 的信息

```javascript
window.addEventListener("unhandledrejection", (e) => e, true);
```

一般错误

```javascript
window.addEventListener("error", (e) => e, true);
```

## js判断当前页面是否被切换

可以判断电脑桌面的切换、同一个浏览器中标签的切换，然而只要是非全屏状态的电脑应用切换不会触发

```js
window.addEventListener("visibilitychange", function () {
  console.log(document.hidden);
  console.log(document.visibilityState);
});
```

| document.hidden          | document.visibilityState |
| ------------------------ | ------------------------ |
| true:表示被隐藏，不可见  | hidden:表示隐藏状态      |
| false:表示未被隐藏，可见 | visible:表示是可见状态   |

- 监听页面可见性变化事件

```js
document.addEventListener("visibilitychange", function () {
  // 用户离开了当前页面
  if (document.visibilityState === "hidden") {
    document.title = "页面不可见";
  }

  // 用户打开或回到页面
  if (document.visibilityState === "visible") {
    document.title = "页面可见";
  }
});
```

## Ajax 请求放在 mounted 中的原因(vue2)

- 异步请求放在 created 中（流程混乱）：
  - created -> API 请求 -> 组件重新渲染
  -                -> mounted -> 组件首次渲染
- 异步请求放在 mounted 中（流程清晰）：
  - created -> mounted -> 组件首次渲染 -> API请求 -> 组件重新渲染
    created 和 mounted 的执行过程是同步的，也就是说只有 created 和 mounted 中的同步代码执行完毕后才会执行 created 中的异步代码。

## `Object`方法

### `assign`

将对象浅层拷贝到指定对象中，如果重复则覆盖

```js
const a = Object.assign({}, { a: 1, b: 2 }); // {a: 1, b: 2}
Object.assign(a, { a: 1, b: 4, c: 3 }); // {a: 1, b: 4, c: 3}

// 同时会改变原对象
const b = { a: 1, b: 2 };
Object.assign(b, { c: 3 }) === b; // true
```

### `create`

`Object.create(obj1, optionalProps)`

创建一个对象，使其上一层`__proto__`指向`obj1`

`optionalProps`为可选的，不填写为`null`，可以填写属性描述符来为该对象声明自身的值

当只想要一个对象存储数据, 不需要长长的原型链上的一堆方法, 则可以使用 `Object.create(null)` 来创建一个没有任何方法的对象,节约内存开销

```js
const father = Object.create(Object.prototype, {
  a: {
    value: 1,
    configurable: true,
    enumerable: true,
  },
  b: {
    value: 2,
    configurable: true,
    enumerable: true,
  },
});

console.dir(father);
/*
    a: 1
    b: 2
    [[Prototype]]: Object  // Object.prototype
*/

const child = Object.create(father, {
  c: {
    value: 3,
    configurable: true,
    enumerable: true,
  },
  d: {
    value: 4,
    configurable: true,
    enumerable: false,
  },
});

console.dir(child);
/*
    c: 3
    d: 4
    [[Prototype]]: Object  // father
        a: 1
        b: 2
        [[Prototype]]: Object  // Object.prototype
*/

let result = [];

for (let i in child) {
  result.push(i);
}

console.log(result); // [ 'c', 'a', 'b' ]
console.log(Object.keys(child)); // ['c']
console.log(Object.keys(father)); // ['a', 'b']
```

### `new`

复制指定的对象(引用复制)，和`obj1 = obj2`效果一样

`new Object(obj1)`

```js
const a = {
  a: 1,
  b: 2,
  c: {
    d: 3,
    e: 4,
  },
};

const b = new Object(a);

b === a; // true
```

### `getPrototypeOf`

获取对象的原型

```js
const a = { b: 1, c: 2, d: 3 };

Object.getPrototypeOf(a) === a.__proto__; // true
Object.getPrototypeOf(a) === Object.prototype; // true
```

### `setPrototypeOf`

设置对象的原型

```js
const a = { b: 1, c: 2, d: 3 };

Object.setPrototypeOf(a, { e: 4, f: 5 });
/*
    b: 1
    c: 2
    d: 3
    [[Prototype]]: Object
        e: 4
        f: 5
        [[Prototype]]: Object  // Object.prototype
*/
```

### `getOwnProperty`

获取自己身上非继承的属性

```js
const a = { b: 1, c: 2, d: 3 };

Object.setPrototypeOf(a, { e: 4, f: 5 });

Object.getOwnPropertyNames(a); // ["b", "c", "d"]
```

### `seal`

封闭对象：可读、可写，不可修改、不可删除

```js
const a = { b: 1, c: 2, d: 3 };

Object.seal(a);

a.e = 1; // {b: 1, c: 2, d: 3}
delete a.b; // {b: 1, c: 2, d: 3}
a.b = 2; // {b: 2, c: 2, d: 3}
```

### `freeze`

冻结对象：可读，不可写、不可修改、不可删除

```js
const a = { b: 1, c: 2, d: 3 };

Object.freeze(a);

a.e = 1; // {b: 1, c: 2, d: 3}
delete a.b; // {b: 1, c: 2, d: 3}
a.b = 2; // {b: 1, c: 2, d: 3}
```

### `isExtensible`

对象是否可扩展

```js
const a = { b: 1, c: 2, d: 3 };

Object.isExtensible(a); // true
Object.freeze(a);
Object.isExtensible(a); // false
```

`preventExtensions`

禁止对象属性扩展（可删除）

```js
const a = { b: 1, c: 2, d: 3 };

Object.preventExtensions(a);
a.e = 2; // {b: 1, c: 2, d: 3}

delete a.b; // {c: 2, d: 3}
```

### `hasOwnProperty`

判断对象自有属性是否有对应的属性

`in`会判断对象整个原型链上的所有属性

```js
const a = { b: 1, c: 2, d: 3 };
Object.setPrototypeOf(a, { e: 4, f: 5 });

a.hasOwnProperty("b"); // true
a.hasOwnProperty("e"); // false
```

## 去重

`indexOf`只会返回第一个找到的下标

```js
const a = [1, 2, 3, 2, 1, 2, 3, 3, 4, 6, 3, 8, 4, 1, 9, 6, 4, 0];

function uniqueArr(array) {
  return array.filter((item, index) => {
    return array.indexOf(item) === index;
  });
}

uniqueArr(a);
```

## `vscode debug mode`

|          |                  launch                  |                                       attach                                        |
| :------: | :--------------------------------------: | :---------------------------------------------------------------------------------: |
|   作用   | vscode负责启动程序并给程序搭配一个调试器 | 为一个已经在运行且暂不支持调试的程序（通常是一个正在运行的web应用程序）加一个调试器 |
| 使用场景 |                 日常开发                 |                          远程调试、本地开发的一些特殊情况                           |

## `history`对象常见属性和方法

- 属性
  - length 会话中的记录条数
  - state 当前保留的状态值
- 方法
  - back 返回上一页，等价于`history.go(-1)`
  - forward 前进下一页，等价于`history.go(1)`
  - go 加载历史中的某一页
  - pushState 打开一个指定的地址
  - replaceState 打开一个新的地址，并且使用replace，即不会产生新的历史记录

## `generator`解决回调地狱

```js
function requestData(url) {
  return new Promise((resolve, reject) => {
    resolve();
  });
}

function* getData() {
  const res1 = yield requestData("why");
  const res2 = yield requestData(res1 + "1");
  const res3 = yield requestData(res2 + "2");
  const res4 = yield requestData(res3 + "3");
  const res5 = yield requestData(res4 + "4");
  console.log(res5);
}

function execGen(genFn) {
  function exec(res) {
    const result = genFn.next(res);
    if (result.done) return result.value;
    result.value.then((r) => {
      exec(result.value);
    });
  }
  exec();
}

execGen(getData);
```

## HTML元素设置自定义属性以及JS获取

- 设置 以`data-`开头

```html
<div id="dio" data-sizing="12"></div>
```

- 获取 从`dataset`获取

```js
const el = document.querySelector("#dio");
el.dataset.sizing; // 12
```

## HTML 获取元素 `data-xxx` 中的内容

一般在元素自身的 `data-xxx` 属性可以在该元素自身的伪元素中直接通过 CSS 的 `attr(data-xxx)` 获取到其内容来设置 `content` 属性(尽可在伪元素中使用)

```html
<section class="section" data-dio="the content of section">
  Common Element
</section>
```

```css
.section::after {
  content: arrt(data-dio);
  background-color: #333;
}
```

## JS监听元素CSS `transition` 结束

```js
const el = document.querySelecyor("div");
el.addEventListener("transitionend", (e) => console.log("transition end"));
```

## CSS中的 `@import` 机制

@import 会将加载的 CSS 应用到样式文件的最顶层，即层叠优先级最高

对于请求阻塞问题，请求阻塞并不是 @import 的问题，而是 CSS 本身的问题，页面渲染的性质要求样式必须提前加载完毕，且要保证顺序

至于重复请求，对于同一个页面，只要不存在 N 个 CSS 指向同一个 CSS 模块的场景，@import 就不会有这个的问题

```css
/* style.css */
html {
  background-color: red;
}
```

```css
/* index.css */
@import "style.css";

html {
  height: 100vh;
  background-color: blue;
}
```

```html
<link rel="stylesheet" href="index.css" id="style" />
```

最终编译的效果为

```css
/* index.css */
html {
  background-color: red;
}
html {
  height: 100vh;
  background-color: blue;
}
```

同时支持 layer 指定 layer 名称

```html
@import './zxx.lib.css' layer(lay);
```

对于第三方样式文件，则可以使用 layer 指定名称的方式来调整优先级

```html
layer lay1, lay2;
@import './lay1.css' layer(lay1);
@layer lay2 {

}
</ 此时 lay1 的层叠优先级比 lay2 高，即 lay2 中的样式优先级更高 -->
```

## CSS 全局属性关键字

- `initial` 设置该样式为 `CSS协会推荐` 的样式
- `inherite` 设置该样式继承自父样式
- `unset` 清除不能继承的样式，通常与 `all` 联用重置样式，使其与 `span` 的效果一样
- `revert` 设置该样式为浏览器对该元素默认的样式

## 读取图片 Blob 实现图片预览

- URL.createObjectURL(file)

```javascript
let fileBlobPath;
try {
  fileBlobPath = URL.createObjectURL(file);
} catch (e) {
  console.log(e);
}
```

- FileReader.readAsDataURL(file)

```javascript
const fileReader = new FileReader();
fileReader.readAsDataURL(file);
fileReader.addEventListener("load", () => {
  const fileBase64String = fileReader.result;
});
```

- 异同：
  - 返回值：
    - FileReader.readAsDataURL(file) 可以得到一段 base64 的字符串
    - URL.createObjectURL(file) 可以得到当前文件的一个内存 URL
  - 执行机制：
    - FileReader.readAsDataURL(file) 通过回调的形式返回结果，异步执行
    - URL.createObjectURL(file) 直接返回，直接执行
  - 内存清理
    - FileReader.readAsDataURL(file) 依照 JS 垃圾回收机制自动从内存中清理
    - URL.createObjectURL(file) 存在于当前 document 内，清除方式只有 unload 事件或 revokeObjectURL 手动清除

## 数组扁平化

- `Array.prototype.concat`会将多个变量中的一层数组展开并添加到目标中

```js
const arr = [1, 2, 3, 4, [2, 5, 1, [2, 31, 5], 2, 7], 9, 1, [0, 1], 8];

function flatten(array) {
  let arr = array || [];
  let finalArr = [];

  arr.forEach((item) => {
    if (isArray(item)) {
      finalArr = finalArr.concat(flatten(item));
    } else {
      finalArr.push(item);
    }
  });

  return finalArr;

  function _isArray(obj) {
    if (Object.prototype.toString.call(obj) === "[object Array]") return true;
    return false;
  }
}
```

- 数组内置方法 `flat`

```js
// Array.prototype.flat(n)  n 表示需要拍平多少层 如果不论多少层都直接拍平可以使用 Infinity
[1, 2, 3, 4, [2, 5, 1, [2, 31, 5], 2, 7], 9, 1, [0, 1], 8].flat(Infinity); // [1, 2, 3, 4, 2, 5, 1, 2, 31, 5, 2, 7, 9, 1, 0, 1, 8]
```

- 递归实现

```js
Array.prototype.flatten = function () {
  var self = this;
  var toStr = Object.prototype.toString;
  if (toStr.call(self) !== "[object Array]") {
    throw new Error("only support Array type");
  }
  // var finalArr = []
  // self.forEach(function (item) {
  //     toStr.call(item) === '[object Array]'
  //         ? finalArr = finalArr.concat(item.flatten())
  //      : finalArr.push(item)
  // })
  return self.reduce(function (prev, curr) {
    return prev.concat(
      toStr.call(curr) === "[object Array]" ? curr.flatten() : curr,
    );
  }, []);

  // return finalArr
};
```

## rem 与 em 区别

`rem`：根据html里的`font-size`来计算

`em`：根据父元素的`font-size`来计算

```html
html{ font-size: 50px; } div{ height: 2rem; }
```

## flex布局中的参数

- flex为 `flex-grow、flex-shrink、flex-basic` 的缩写，默认值为 0 1 auto，`flex-basic: auto` 表现为 `min-content`

  - `flex: auto` (1 1 auto)
  - `flex: none` (0 0 auto)
  - `flex: initial` (0 1 auto)
  - `flex: 1` (flex-grow: 1)
  - `flex: 1px` (flex-basis: 1px)
  - `flex: 1 1` (flex-grow: 1; flex-shrink: 1)
  - `flex: 1 1px` (flex-grow: 1; flex-basis: 1)

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
window.addEventListener(
  "click",
  (e) => {
    if (banned === "true") {
      e.stopPropagation();
    }
  },
  true,
); // 捕获模式 默认为false
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

## 上传文件指定文件的类型

```html
<input type="file" accept=".png,.jpeg,.jpg" />
```

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
- root originalBubble

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

> ​ react为并发模式，无论在哪`setState`都是批量异步执行的

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

## display=none 与 visibility=hidden

- display=none 时，隐藏的元素不会占据之前的位置；而 visibility=hidden 时，隐藏的元素会占据之前的位置
- 当 visibility=hidden 时，子元素可以通过设置 visibility=visible 显示；而 display=none 时则无法通过任何途径显示子元素
- visibility=hidden 不会影响计数器的结果(例如 `ul -> li` )，而 display=none 会使计数器直接忽略这个元素
- display=none 无法通过 animation 设置动画，而 visibility=hidden 可以(建议通过高性能的 opacity 来实现隐藏显示动画)

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

## 迭代器

为各种不同的数据结构提供统一的访问机制，任何数据结构只要部署`Iterator`接口，就可以完成遍历操作(`for of`循环)，依次处理该数据结构的所有成员

- 拥有`next`方法用于依次遍历数据结构的成员
- 每次遍历返回的结果是一个对象：`{ done: Boolean, value: any }`
  - done 记录是否遍历完成
  - value 当前遍历的结果

拥有`Symbol.iterator`属性的数据结构，被称为可被遍历的

- Array
- 部分`类数组`：arguments/NodeList/HTMLCollection...
- String
- Set
- Map
- generator object
- ...

对象默认不具备`Symbol.iterator`，属于不可被遍历的数据结构

```js
class Iterator {
  constructor(assemble) {
    this.assemble = assemble;
    this.index = 0;
  }

  next() {
    if (this.index > this.assemble.length - 1)
      return { done: true, value: undefined };
    return { done: false, value: this.assemble[this.index++] };
  }
}

const arr = [1, 2, 3, 4];
const iter = new Iterator(arr);

console.log(iter.next()); // { done: false, value: 1 }
console.log(iter.next()); // { done: false, value: 2 }
console.log(iter.next()); // { done: false, value: 3 }
console.log(iter.next()); // { done: false, value: 4 }
console.log(iter.next()); // { done: true, value: undefined }
```

## `for in`和`for of`的区别

`for in`的循环机制

- 优先迭代数字属性

- 无法迭代`Symbol`的私有属性

- 无法迭代`enumerable=false`的`key`

- 遍历完私有属性后会遍历公有属性（性能差）只能遍历第一层的`key`

  ```js
  Object.property.AA = 20;

  const obj = {
    1: "dio",
    name: "jojo",
    [Symbol("AA")]: 10,
  };

  for (let key in obj) {
    // 避免遍历到公有属性 然而依旧有性能问题
    if (!obj.hasOwnProperty(key)) break;
    console.log(key);
  }

  // 获取所有私有属性的名字
  let keys = Object.getOwnPropertyNames(obj); // Object.keys(obj)
  keys = keys.concat(Object.getOwnPropertySymbols(obj));
  keys.forEach((key) => {
    conslole.log(key, obj[key]);
  });
  ```

`for of`的循环机制

- 自动调用对象的`Symbol.iterator`方法获取到一个`itor`对象，每次获取值则调用该对象的`next`方法

  ```js
  let arr = [1, 2, 3, 4, 5];

  for (let item of arr) {
    console.log(item); // 1 2 3 4 5
  }

  arr[Symbol.iterator] = function () {
    let index = -2;

    return {
      next() {
        if (index > this.length - 1) return { done: true, value: undefined };
        return { done: false, value: this[(index += 2)] };
      },
    };
  };

  for (let item of arr) {
    console.log(item); // 1 3 5 undefined
  }
  ```

## `instanceof`的原理

调用`instanceof`时会调用对象的`Symbol.hasInstance`方法

```js
class Fn {
  constructor() {
    this.x = Symbol.for("Fn.x");
  }

  static [Symbol.hasInstance](obj) {
    return obj.x && obj.x === Symbol.for("Fn.x");
  }
}

const f = new Fn();

// obj instanceof Ctor => Ctor[Symbol.hasInstance](obj)
console.log(f instanceof Fn); // true

const arr = [1, 2, 3];
// 此时尽管修改了 arr 的原型，但是会调用自定义的 Symbol.hasInstance 方法来判断
Object.setPrototypeOf(arr, Fn.prototype);
console.log(arr instanceof Fn); // false
```

## `npm` 命令行进行版本控制

```bash
# 增加一个修复版本号: 1.0.1 -> 1.0.2 (自动更改 package.json 中的 version 字段)
npm version patch

# 增加一个小的版本号: 1.0.1 -> 1.1.0 (自动更改 package.json 中的 version 字段)
npm version minor

# 将更新后的包发布到 npm 中
npm publish
```

## 对象和数字比较的隐式转换

对象 == 数字 默认把对象转换为数字

1. 首先看对象是否有`obj[Symbol.toPrimitive]`方法，有则按该方法执行
2. 没有则使用`obj.valueOf()`验证是否为原始值类型
3. 如过不是原始值类型，则调用`obj.toString()`变为字符串
4. 将字符串变为数字

```js
const a = {
  i: 0,
};

a[Symbol.toPrimitive] = function () {
  return ++this.i;
};

console.log(a == 1 && a == 2 && a == 3); // true
```

## `Element-Plus`组件按需引入时的问题

当使用按需引入时，子组件不能从`element-plus`中导入，需要在对应的包中导入

```js
import { ElButton, ElTabs, ElForm, ElInput } from "element-plus";
import { ElTabPane } from "element-plus/lib/components/tabs";
import { ElFormItem } from "element-plus/lib/components/form";
```

## `z-index`与元素层级

`z-index`只影响同一级元素

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      div {
        width: 100px;
        height: 100px;
      }
      .red {
        position: relative;
        background-color: red;
        z-index: 10;
      }
      .blue {
        position: absolute;
        top: 20px;
        left: 20px;
        background-color: blue;
        z-index: 30;
      }
      .yellow {
        position: absolute;
        top: 40px;
        left: 40px;
        background-color: yellow;
        z-index: 50;
      }
      .black {
        position: absolute;
        top: 80px;
        left: 10px;
        background-color: black;
        z-index: 20;
      }
    </style>
  </head>
  <body>
    <div class="red">
      <div class="blue"></div>
      <div class="yellow"></div>
    </div>
    <div class="black"></div>
  </body>
</html>
```

元素可见从上到下依次为`黑 -> 黄 -> 蓝 -> 红`

# loader与plugin

- loader 是用于处理**特定的模块类型**进行转换
- plugin 可以用于执行**更加广泛的任务**，比如打包优化、分包、资源管理、环境变量注入等

# 查看主流浏览器browserslist

通常对浏览器适配时要查询，例如css加前缀`yarn add browserslist -D`

配置

```json
// package.json

browserslist: [
    ">1%",  // 使用量大于1%的浏览器
    "last 2 version",  // 这些浏览器最新发布的两个版本
    "not dead"  // 24个月内有更新的浏览器
]
```

```json
// .browserslistrc

>1%
last 2 version
not dead
```

所有结果都是基于`caniuse.com/usage-table`提供的数据

# 配置了多个loader但是想让其中一个每次都最先执行

一种规则的多个loader的执行顺序为从后往前，但是可以通过设置`enforce`属性来使某些loader不依赖书写的顺序执行

loader的执行是通过`pitchLoader`和`normalLoader`来执行，pitchLoader阶段对`loaderIndex++`，`normalLoader`阶段对`loaderIndex--`

- 默认所有的`loader`都是`normal`
- `pitchLoader`执行顺序：post -> inline -> normal -> pre
- `normalLoader`执行顺序：pre-> normal -> inline-> post

```js
rules: [
    {
        test: /\.js$/,
        loader: "loader1"
    },
    {
        test: /\.js$/,
        loader: "loader2",
        enforce: "pre"
    },
    {
        test: /\.js$/,
        loader: "loader3"
    },
    {
        test: /\.js$/,
        loader: "loader4"
    },
]

resolveLoader: {  // 配置查找模块的路径
    modules: ["node_modules", "./myLoaders"]
}
```



# 对css进行兼容处理

> **行内样式**

## 加前缀

`yarn add postcss autoprefixer postcss-loader -D`

```js
module: {
    rules: [
        test: /\.css/,
        use: [
             "style-loader",  // 将css插入到index.html中
             "css-loader",  // 处理css文件
            // 将需要做兼容处理的css加前缀
             {
                 loader: "postcss-loader",
                 options: {
					postcssOptions: {
                        plugins: [
                            "autoprefixer"  // require("autoprefixer")
                        ]
                    }
                 }
             }
        ]
    ]
}
```

## 对现代的css进行处理（例如：color: #12345678 => rgba(12,23,56,78)），并且可以代替`autoprefixer`添加前缀

`yarn add postcss-preset-env -D`

```js
plugins: [
    // "autoprefixer"
    "postcss-preset-env"
]
```

## 对配置进行抽离

```js
// webpack.config.js

use: [
    "style-loader",
    "css-loader",
    "postcss-loader"
]
```

```js
// postcss.config.js

module.export = {
    plugins: [
        require("postcss-preset-env")
    ]
}
```

## 当css中使用了`@import`语法时，新的css需要重新使用之前的loader进行处理

```js
use: [
    "style-loader",
    {
        loader: "css-loader",
        options: {
            importLoaders: 1  // 表示需要重新被后面的几个loader重新处理
        }
    },
    "postcss-loader"
]
```

# 将css代码抽离到文件中

`yarn add mini-css-extract-plugin -D`

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin")

test: "/\.css/",
use: [
    isProduction ? miniCssExtractPlugin.loader : "style-loader",
    "css-loader"
]

plugins: [
    new MiniCssExtractPlugin({
        filename: "css/[name].[chunkhash:6].css"
    })
]
```

# 对css进行压缩

`yarn add css-minimizer-webpack-plugin -D`

```js
const CssMinimizerWebpackPlugin = require("css-minimizer-webpack-plugin")

plugins: [
    new CssMinimizerWebpackPlugin()
]
```

# 对文件进行处理

`yarn add file-loader -D`

## 图片

### webpack5之前

- `file-loader`对`require(url)`、`img.src = url`和`import imgUrl from url`加载的图片进行处理，最终复制到`build`文件夹中并使用md4对文件内容进行计算得到的值进行图片的重命名

```js
{
    test: "/\.(png|jpe?g|gif|svg)/",
    use: [
        {
            loader: "file-loader",
        }
    ]
}
```

使用`placeholder`对打包的文件名进行自定义，同时指定输出的目录

```js
{
    loader: "file-loader",
	options: {
        // name: "[name].[chunkhash:6].[ext]",
        // outputPath: "img"  // 会将图片重命名后复制到 build/img 中
        name: "img/[name].[chunkhash:6].[ext]"
    }
}
```

- `url-loader`对小图片进行`base64`编码，大图片正常复制并重命名

`yarn add url-loader -D`

```js
{
    loader: "url-loader",
	options: {
        name: "img/[name].[hash:6].[ext]",
		limit: 100 * 1024  // 单位：Byte 对小于100kb的图片进行base64编码
    }
}
```

### webpack5之后

直接使用资源模块类型( asset module type )来代替上面的loader

- asset/resource 发送一个单独的文件并导出URL。之前通过file-loader实现

```js
{
    test: /\.(png|jpe?g|gif|svg)/,
    type: "asset/resource"
}
```

指定输出文件目录

```js
/**
output: {
    filename: "build.js",
	path: path.resolve(__dirname, "build"),
    assetModuleFilename: "img/[name].[hash:6][ext]"
}
*/

{
    test: /\.(png|jpe?g|gif|svg)/,
    type: "asset/resource",
    generator: {
        filename: "img/[name].[hash:6][ext]"
    }
}
```

- asset/inline 导出一个资源的data URI。之前通过url-loader实现(无法指定输出目录，因为图片直接使用base64输出到js中了)

```js
{
    test: /\.(png|jpe?g|gif|svg)/,
    type: "asset/inline"
}
```

- asset 配置实现小图片使用base64，大图片直接复制

```js
{
    test: /\.(png|jpe?g|gif|svg)/,
    type: "asset",
    generator: {
        filename: "img/[name].[hash:6][ext]"
    },
    parser: {
        dataUrlCondition: {
            maxSize: 100 * 1024
        }
    }
}
```

## 字体

```js
{
    test: /\.(ttf|eot|woff2?)$/i,
    type: "asset/source",  // 保留原始文件不做处理
    generator: {
        filename: "font/[name].[hash:6][ext]"
    }
}
```

## 每次打包的时候对build(output中的path)里的文件进行清除

`yarn add clean-webpack-plugin -D`

```js
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.export = {
    entry: "./src/main.js",
    output: {
        filename: "build.js",
        path: path.resolve(__dirname, "build")
    },
    plugins: [
        new CleanWebpackPlugin()
    ]
}
```

## 自动为打包后的文件夹中添加`index.html`

`yarn add html-webpack-plugin -D`

```js
// 使用自定义的index.html模板(基于ejs)
const HtmlWebpackPlugin = require('html-webpack-plugin')

plugins: [
    new HtmlWebpackPlugin({
        title: "assign your web title",
        // 指定模板文件，也可以不指定使用自动生成的
        template: "public/index.html",
        // 当文件没有发生任何改变时直接使用之前的缓存
        cache: true,
        // 是否压缩代码 默认为true
        minify: isProduction ? {
            // 是否移除注释
            removeComments: true,
            // 是否移除冗余的属性 例如input标签默认type为text不用再声明type=text
            removeRedundantAttributes: false,
            // 是否移除空属性
            removeEmptyAttributes: true,
            // 是否移除空白
            collapseWhitespace: true
        } : false
    })
]
```

## 定义全局变量

webpack内置插件`DefinePlugin`

```js
const { DefinePlugin } = require('webpack')

plugins: [
    new DefinePlugin({
        BASE_URL: '"./"'
    })
]
```

## 将指定目录里的文件拷贝到打包后的目录中

`yarn add copy-webpack-plugin -D`

```js
const { CopyWebpackPlugin } = require('copy-webpack-plugin')

plugins: [
    new CopyWebpackPlugin({
        patterns: [
        	{
                from: "public",
                globOptions: {
                    ignore: [
                        "**/index.html",
                        "**/.DS_Store"
                    ]
                }
            }
        ]
    })
]
```

# 使用babel处理

## JS

`yarn add babel-loader @babel/core -D`

```js
{
    test: "\.js$",
    use: [
        {
            loader: "babel-loader",
            options: {
                plugins: [
                    "@babel/plugin-transform-arrow-functions",
                    "@babel/plugin-transform-block-scoping"
                ]
            }
        }
    ]
}
```

- 根据`.browserslistrc`中对浏览器的配置进行js转换适配

`yarn add @babel/preset-env -D`

```js
options: {
    presets: [
        "@babel/preset-env"
    ]
}
```

在配置文件中

```js
// babel.config.js

module.export = {
    presets: [
        "@babel/preset-env"
    ]
}
```

## Polyfill

`yarn add core-js regenerator-runtime -S`

一种补丁，当用到一些新的语法时(例如：Promise、Generator、Symbol等以及一些实例方法Array.prototype.includes等)低版本的浏览器不认识会报错，这时便可以使用polyfill来声明，最后会在bebel转换后的代码中添加上这些新方法的实现

- `useBuiltIns`

```js
// babel.config.js

module.export = {
    presets: [
        ["@babel/preset-env", {
            // false "usage" "entry"
            useBuiltIns: "usage",
            corejs: 3
        }]
    ]
}
```

> 配置的时候需要指定安装的`corejs`的版本，不指定默认使用2的版本
>
> useBuiltIns的值：
>
> - `false` 不使用
> - `usage` 匹配的js文件中用到的进行打补丁
> - `entry` 根据目标浏览器(`.browserslist`)所支持的特性无论是否用到都打补丁
> - 当在用到`usage`和`entry`时建议在webpack中配置`exclude: /node_modules/`来避免第三方包已经打过补丁但是自己又打会造成一些冲突
> - 当在使用到`entry`时需要在打包的入口文件（例如：`index.js`）中导入`import "core-js/stable"` `import "regenerator-runtime/runtime"`

- `plugin-transform-runtime`

`yarn add @babel/plugin-transform-runtime -D`

和`useBuiltIns`相比同样也是配置使用补丁的策略，区别在于useBuiltIns会全局配置，而`plugin-transform-runtime`只会局部作用，这就意味着当你开发第三方库的时候如果使用了`useBuiltIns`就会污染使用者的代码环境

```js
// babel.config.js

module.export = {
    presets: [
        "@babel/preset-env"
    ],
    plugins: [
        ["@babel/plugin-transform-runtime", {
            corejs: 3
        }]
    ]
}
```

因为使用到了`corejs`3的版本，同时需要安装`yarn add @babel/runtime-corejs3 -S`

## TS

`yarn add ts-loader`

需要有`tsconfig.json`，如果使用`babel-loader`则不需要安装`ts-loader`，`ts-loader`会对ts代码进行检错，`babel-loader`会对ts的代码进行ployfill但不检错。建议使用的时候先用`tsc --noEmit`进行类型检测，然后再使用`babel-loader`进行转换

```js
{
    test: /\.ts$/,
    exclude: /node_modules/,
    use: {
        // 本质上依赖tsc
        loader: "babel-loader"
    }
}
```

更推荐使用`preset-typescript`

`yarn add @babel/preset-typescript -D`

```js
// babel.config.js

module.export = {
    presets: [
        ["@babel/preset-env"],
        ["@babel/preset-typescript"]
    ],
}
```

# 配置`webpack-dev-serve`为热更新

`yarn add webpack-dev-serve -D`

```js
devServr: {
    hot: true
}
```

使用：`webpack serve`

一般文件需要对单独需要热更新的文件进行配置，而`react、vue`之类的框架提供了HMR(`vue-loader、@pmmmwh/react-refresh-webpack-plugin react-refresh`)

# publicPath的作用

## output中

在打包之后的静态资源前面进行一个路径的拼接，大多配置为`./`

```js
output: {
    path: path.resolve(__dirname, 'build'),  // -> js/build
    publicPath: "./"
}
"./" + "js/build"
```

## devServer中

本地资源所存放的目录，默认为`/`

```js
devServer: {
    hot: true,
    publicPath: "/"
}
```

建议两处的`publicPath`的配置一样

# devServer中的其他配置

## contentBase

指定静态引入的资源的查找目录，默认为当前文件夹

## port

指定服务监听的端口，默认为8080

## open

默认为`false`，为`true`时当项目编译完成会自动打开浏览器

## compress

设置是否为静态文件开启gzip压缩，默认为false

# entry的路径

entry的路径是相对于contex的，不指定contex的路径时默认为启动webpack时命令所在的目录

# optimization对代码进行分割（可复用代码进行抽离）

```js
optimization: {
    // 分包后的文件名 开发环境下默认为named 生产环境下默认为deterministic
    // natural  自然数 不推荐
    // named  文件路径_文件名_output中的filename  可以在output中声明 chunkFilename: "[name].chunk.js"
    // deterministic  生成id，针对相同的文件生成的id是不变的
    chunkIds: "deterministic",
    splitChunks: {
        // async 异步导入
        // initial 同步导入
        // all 异步/同步导入
        chunks: "all",
        // 最小尺寸，如果包的大小未达到minSize则不拆分
        minSize: 20000,
        // 将大于maxSize的包拆分成不小于minSize的包
        maxSize: 30000,
        // 表示被导入的包最少被引入了几次
        minChunks: 1,
        // 将匹配到的所有文件一起打包到指定文件中
        cacheGroups: {
            vendor: {
                test: /[\\/]node_modules[\\/]/,
                filename: "[id]_vendors.js"
            }
        }
    }
}
```

# Prefetch和Preload

预获取和预加载。在声明异步import时，可以提前获取资源文件

```js
import(
/* webpackPrefetch: true */
"./component"
).then(( { default: component } ) => { console.log(component) })
```

prefetch会在加载完所有需要立即加载的文件后等待浏览器空闲的时候向服务器发送请求来获取资源文件，然后放入缓存中

preload会在父chunk加载的同时并行加载

# 对于第三方包不进行打包而是改为引入cdn

先在webpack中排除第三方库的打包

```js
externals: {
    // 第二个参数为第三方库导出的全局对象
    lodash: "_",
    dayjs: "dayjs"
}
```

然后再在`index.html`的模板文件中手动引入第三方库的cdn

```js
plugins: [
    new HtmlWebpackPlugin({
        title: "assign your web title",
        template: "public/index.html"
    })
]
```

# 对打包的文件进行作用域提升

默认情况下webpack打包会有很多作用域，无论是从最开始的代码开始执行，还是加载一个模块，都需要执行一系列的函数，而作用域提升后可以将函数合并到一个模块中来运行，使得webpack打包后的代码更小，运行更快

这个插件是基于对ESM的静态分析，因此导包的时候如果想使用作用域提升进行优化就尽量使用`import`的语法静态导入

```js
plugins: [
    new webpack.optimeze.ModuleConcatenationPlugin()
]
```

# TreeShaking

## JS

### usedExports

- 将optimization中的usedExports设置为true，同时需要配合Terser。此时打包时便会标记未使用的函数
- 打包后会发现没有使用到的代码不会被添加到打包的文件中

### sideEffects

- 用于告知webpack compiler哪些模块是有副作用的
- 如果sideEffects为false，那么对于一些未使用到的import在打包时便会删除
- 对于css文件，可以再在loader中单独为其设置`sideEffects: true`

## CSS

`yarn add purge-css-plugin -D`

```js
const PurgeCssPlugin = require("purge-css-plugin")
const glob = require("glob")

plugins: [
    new PurgeCssPlugin({
        // 需要做treeshaking的css代码的路径
        path: glob.sync(`${__dirname}/src/**/*`, {nodir: true}),
        // 添加白名单
        safeList: function() {
            return {
                standard: ["body", "html"]
            }
        }
    })
]
```

# 压缩

`yarn add compression-webpack-plugin -D`

```js
const CompressionWebpackPlugin = require("compression-webpack-plugin")

plugins: [
    new CompressionWebpackPlugin()
]
```

# 对自定义第三方包进行打包发布

```js
output: {
    // 适配AMD，ESM，CommonJS(无module)，CommonJS2(有module) 这样会在打包的文件中对导出的东西进行规范的判断然后统一导出
    libraryTarget: "umd",
    // 库的名称 webpack会根据名称对齐导出的内容进行遍历然后统一导出
    library: "myUtils",
    // 全局对象
    globalObject: "this"
}
```


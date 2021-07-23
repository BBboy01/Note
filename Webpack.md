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

# 对css进行兼容处理

## 加前缀

`yarn add postcss autoprefixer postcss-loader -D`

```js
module: {
    {
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
    }
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

# 对文件进行处理

`yarn add file-loader -D`

## 图片

### webpack5之前

- `file-loader`对`require(url)`、`img.src = url`和`import imgUrl from url`加载的图片进行处理，最终复制到`build`文件夹中并使用md4对文件内容进行计算得到的值进行图片的重命名

```js
{
    test: /\.(png|jpe?g|gif|svg)/,
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
        // name: "[name].[hash:6].[ext]",
        // outputPath: "img"  // 会将图片重命名后复制到 build/img 中
        name: "img/[name].[hash:6].[ext]"
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

自动为打包后的文件夹中添加`index.html`

`yarn add html-webpack-plugin -D`

```js
// 使用自定义的index.html模板(基于ejs)
const HtmlWebpackPlugin = require('html-webpack-plugin')

plugins: [
    new HtmlWebpackPlugin({
        title: "assign your web title",
        template: "public/index.html"
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

`yarn add copy-webpack-plugin`

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

# 使用babel处理js

`yarn add babel @babel/core-loader -D`

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

- 根据`.broswerslistrc`中对浏览器的配置进行js转换适配

`yarn add @babel/preset-env -D`

```js
options: {
    preset: [
        "@babel/preset-env"
    ]
}
```


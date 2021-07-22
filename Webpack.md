## 查看主流浏览器browserslist

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


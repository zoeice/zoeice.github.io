---
layout:     post
title:      Webpack配置详解
subtitle:   一些简单的配置说明
description: 
date:        2019-07-06 08:30:00
author:     zoeice
image:      http://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-webpack-config.jpg
optimized_image: http://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-webpack-config.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   Webpack
tags:
    - Webpack
---

# 介绍
Webpack 是一个模块打包器。
在Webpack看来, 前端的所有资源文件(js/json/css/img/less/…)都会作为模块处理

它可以做到：
1. 捆绑 ES 模块、CommonJS 和 AMD 模块（甚至它们的组合）。
2. 可以创建在运行时异步加载的单个包或多个块（以减少初始加载时间）。
3. 在编译期间解决依赖关系，从而减少运行时大小。
4. 加载器可以在编译时预处理文件，例如 TypeScript 到 JavaScript，Handlebars 字符串到编译函数，图像到 Base64 等。
5. 高度模块化的插件系统，可以满足您的应用程序需要的任何其他需求。

Webpack 支持所有符合 ES5 的浏览器（不支持 IE8 及以下版本）

### 安装

用npm安装：
```bash
npm install webpack webpack-cli -D
```
或者用yarn安装：
```bash
yarn add webpack webpack-cli -D
```
这样就将 webpack 放入 devDependencies 依赖中。

### 入口起点(entry)
默认入口起点是 `./src/index.js`，也可以在 `webpack.config.js` 中配置 entry 来指定一个或多个入口起点。

简单写法：
```js
module.exports = {
    // 指定入口起点
    entry: './src/app.js',
};
```

完整写法：
```js
module.exports = {
    entry: {
        main: './src/app.js'
    }
};
```

### 输出(output)
output属性指定 webpack 输出它创建的 bundle 的位置，以及如何命名。
输出文件的默认值是：`./dist/main.js`，生成的其他文件默认放置在 `./dist` 文件夹中。
也可以在 `webpack.config.js` 中配置 output。
```js
const path = require('path');

module.exports = {
    entry: './src/app.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'my-webpack.bundle.js',
    }
};
```

### loader
webpack 默认只能识别 JavaScript 和 JSON 文件。
`loader` 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 `module`，以供应用程序使用，以及被添加到依赖图中。
通过 `loader` 能让你是用很多新的功能。

`loader` 的规则有两个属性：
- `test` 属性，识别出哪些文件会被转换（这里是用正则表达式来约定配置，这里的正则表达式不需要额外加引号）。
- `use` 属性，定义出在进行转换时，应该使用哪个 `loader`。

在 `webpack.config.js` 中配置 loader。
```js
const path = require('path');

module.exports = {
  output: {
    filename: 'my-webpack.bundle.js',
  },
  module: {
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
};
```

以上配置中，对一个单独的 `module` 对象定义了 `rules` 属性，里面包含两个必须属性：`test` 和 `use`。

### 插件(plugin)
插件相比预设可以执行范围更广的任务。包括：打包优化，资源管理，注入环境变量。

如果插件和预设同时处理某段代码，会按这个规则：
- 插件在 预设(preset) 前运行。
- 插件顺序从前往后排列。

想要使用一个插件，你只需要 require() 它，然后把它添加到 plugins 数组中。

在 `webpack.config.js` 中配置 `plugin`。
```js
module.exports = {
    plugins: [new PluginA()],
};
```

### 模式(mode)
可以将 mode 参数设为 `development`, `production` 或 `none` 之中的一个，你可以启用 webpack 内置在相应环境下的优化。其默认值为 production。

在 `webpack.config.js` 中配置 `mode`。
```js
module.exports = {
  mode: 'production',
};
```

# 应用
### 打包less资源
安装less相关库

```bash
npm install css-loader style-loader less-loader less -D
```

在 `webpack.config.js` 中配置 loader

```js
const path = require('path');

module.exports = {
  module: {
    rules: [
        {
            // 检查文件是否以.less结尾（检查是否是less文件）
            test: /\.less$/, 
            use: [  
                // 创建style标签，添加上js中的css代码
                'style-loader', 
                 // 将css以commonjs方式整合到js文件中
                'css-loader',
                // 将less文件解析成css文件
                'less-loader'  
            ]
        }
    ],
  },
};
```

### js转换：配置Babel
Babel 是一个Javascript编译器，可以让你不用考虑浏览器兼容问题，直接使用 ES6/7/8/9等 写代码。
babel可以做这些事：
- 语法转换
- 通过 Polyfill 方式在目标环境中添加缺失的特性
- 源码转换

使用babel一般会安装这几个库:
- babel-loader 用于让 webpack 处理 babel
- babel-core 可以看做编译器，这个库知道如何解析代码
- babel-preset-env 这个库可以根据环境的不同转换代码

用npm安装：
```bash
npm i --save-dev babel-loader babel-core babel-preset-env
```
或者用yarn安装
```bash
yarn add -D babel-loader babel-core babel-preset-env
```

接下来更改 webpack-config.js 中的代码
```js
module.exports = {
    module: {
        rules: [
            {
                // js 文件才使用 babel
                test: /\.js$/,
                // 使用哪个 loader
                use: {
                    loader: "babel-loader",
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
                // 不包括路径
                exclude: /node_modules/
            }
        ]
    }
}
```

配置 Babel 有很多方式，这里使用 `.babelrc` 文件管理, 放在项目根目录。
```js
{
    // Babel 的预设（preset）可以被看作是一组 Babel 插件和/或 options 配置的可共享模块。
    // Preset数组 是逆序排列的（从后往前执行）
    "presets": ["babel-preset-env"]
}
```

### js兼容性处理：配置polyfill
解决babel只能转换部分低级语法的问题(如：let/const/解构赋值…)，引入polyfill可以转换高级语法(如:Promise…)， 但是会转换全部的高级语法，可以通过cire-js实现按需引入来解决这个问题。

```bash
npm install @babel/polyfill
npm install core-js
```

在 `webpack.config.js` 中配置 loader

```js

module.exports = {
  module: {
    rules: [
        {
            test: /\.js$/,
            exclude: /(node_modules)/,
            use: {
            loader: 'babel-loader',
            options: {
                presets: [
                    [
                        '@babel/preset-env',
                        {
                            // 按需引入需要使用polyfill
                            useBuiltIns: 'usage',  
                            // 解决warn
                            corejs: { version: 3 }, 
                            // 指定兼容性处理哪些浏览器
                            targets: { 
                                "chrome": "58",
                                "ie": "9",
                            }
                        }
                    ]
                ],
                // 开启babel缓存
                cacheDirectory: true, 
            }
            }
        },
    ],
  },
};
```

### 处理图片
通过 url-loader 处理图片
url-loader是对象file-loader的上层封装，使用时需配合file-loader使用。

```bash
npm install file-loader url-loader -D
```

在 `webpack.config.js` 中配置 loader

```js

module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpg|gif)$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        // 8kb --> 8kb以下的图片会base64处理
                        limit: 8192, 
                        // 决定文件本地输出路径
                        outputPath: 'images', 
                        // 决定图片的url路径
                        publicPath: '../build/images',  
                        // 修改文件名称 [hash:8] hash值取8位. [ext] 文件扩展名
                        name: '[hash:8].[ext]' 
                    }
                }
            },
        ],
    },
};
```

### 打包html文件
需要用html-webpack-plugin这个库实现， 注意不要在html中引入任何css和js文件。

```bash
npm install html-webpack-plugin -D
```

在 `webpack.config.js` 中配置 `plugin`。
```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
// 用于访问内置插件
const webpack = require('webpack'); 

module.exports = {
    module: {
        rules: [{ test: /\.txt$/, use: 'raw-loader' }],
    },
    plugins: [new HtmlWebpackPlugin(
        { 
            // 以当前文件为模板创建新的HtML(1. 结构和原来一样 2. 会自动引入打包的资源)
            template: './src/index.html' 
        }
    )],
};
```
在上面的示例中，`html-webpack-plugin`插件 为应用程序生成一个 HTML 文件，并自动将生成的所有 bundle 注入到此文件中

要过还要处理打包html里的图片资源，加上这个：
```bash
npm install html-loader -D
```

在 `webpack.config.js` 中配置 `loader`。
```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(html)$/,
                use: {
                    loader: 'html-loader'
                }
            }
        ]
    }
}
```

### 打包其他资源（字体，音视频等）
在 `webpack.config.js` 中配置 `loader`。
```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(eot|svg|woff|woff2|ttf|mp3|mp4|avi)$/,  // 处理其他资源
                loader: 'file-loader',
                options: {
                    outputPath: 'media',
                    name: '[hash:8].[ext]'
                }
            }
        ]
    }
}
```

### 自动编译打包
dev安装依赖库：`webpack-dev-server`
```bash
npm install webpack-dev-server -D
```
在 `webpack.config.js` 中配置
```js
module.exports = {
    devServer: {
        // 自动打开浏览器
        open: true, 
        // 启动gzip压缩
        compress: true, 
        // 端口号
        port: 3000, 
    }
}
```

因为构建工具以build为根目录，需要改下url-loader的配置:
`publicPath: '../build/images/'` 改为 `publicPath: 'images/'`

示例如下
```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpg|gif)$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        // 8kb --> 8kb以下的图片会base64处理
                        limit: 8192, 
                        // 决定文件本地输出路径
                        outputPath: 'images', 
                        // 决定图片的url路径
                        publicPath: 'images/',  
                        // 修改文件名称 [hash:8] hash值取8位. [ext] 文件扩展名
                        name: '[hash:8].[ext]' 
                    }
                }
            },
        ],
    },
    devServer: {
        // 自动打开浏览器
        open: true, 
        // 启动gzip压缩
        compress: true, 
        // 端口号
        port: 3000, 
    }
};
```

因为`webpack-dev-server`指令才能启动`devServer`配置, 因此修改package.json里的scripts:
```json
"scripts": {
    "start": "webpack-dev-server"
}
```

### 热更新（HMR）
热更新是webpack提供的最有用的功能之一。它允许在运行时更新变化的所有类型的模块，而无需完全刷新。

示例如下
```js
module.exports = {
    entry: ['./src/app.js', './src/index.html']
    devServer: {
        // 运行项目的目录
        contentBase: resolve(__dirname, 'build'), 
        // 自动打开浏览器
        open: true, 
        // 启动gzip压缩
        compress: true, 
        // 端口号
        port: 3000,
        // 开启热模替换功能 HMR，从 webpack-dev-server v4.0.0 开始，热模块替换是默认开启的。
        hot: true
    }
};
```

### TypeScript
首先，执行以下命令安装 TypeScript compiler 和 loader：
```bash
npm install -D typescript ts-loader
```

在工程个目录加一个配置文件：`tsconfig.json`, 配置如下：
```json
{
    "compilerOptions": {
        // 输出目录
        "outDir": "./dist/",
        // 没有任何隐含的。 打开后在Typescript无法推导类型时会提醒错误
        "noImplicitAny": true,
        // 模块。配置项有：CommonJS，UMD，AMD，System，ESNext，ES2020， ES2015/ES6，node16/ nodenext，None。
        "module": "es6",
        // 目标。和运行环境有关，如果在旧的环境，建议设置的低一点
        "target": "es5",
        // 声明导入jsx和jsxs工厂函数的模块说明符
        "jsx": "react",
        // 检查JS。打开后，将在 JavaScript 文件中报告错误
        "allowJs": true,
        // 模块分辨率。可选项有：'node'，'node12'，'classic'
        "moduleResolution": "node"
    }
}
```
完整的配置项可以参考：[TSConfig参考](https://www.typescriptlang.org/tsconfig)

配置 webpack 处理 TypeScript：
webpack.config.js
```js
const path = require('path');

module.exports = {
    entry: './src/index.ts',
    module: {
        rules: [
            {
                test: /\.tsx?$/,
                use: 'ts-loader',
                exclude: /node_modules/,
            },
        ],
    },
    resolve: {
        extensions: ['.tsx', '.ts', '.js'],
    },
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist'),
    },
};
```
---
title: webpack基础
date: 2020-12-31 11:36:21
categories: Webpack
tags:
- webpack
permalink: webpack-basic
---
#### 一、安装
##### 1.初始化：
```shell
npm init -y
```

##### 2.安装webpack：
```shell
npm install webpack webpack-cli -D
```
<!--more-->

#### 二、配置
##### 1.babel-loader(高版本js转译为低版本)
a.安装：
```shell
npm install babel-loader -D
```

b.安装babel 7相关配置：https://juejin.im/post/6844904008679686152
```shell
npm install @babel/core @babel/preset-env @babel/plugin-transform-runtime -D
npm install @babel/runtime @babel/runtime-corejs3
```

c.新建webpack.config.js
```javascript
//webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                use: ['babel-loader'],
                exclude: /node_modules/ //排除 node_modules 目录
            }
        ]
    }
}
```

d.创建.babelrc
```json
{
    "presets": ["@babel/preset-env"],
    "plugins": [
        [
            "@babel/plugin-transform-runtime",
            {
                "corejs": 3
            }
        ]
    ]
}
```

##### 2.配置环境变量(分development/production)
a.安装cross-env
```shell
npm install cross-env -D
```

b.package.json添加script
```json
{
    "scripts": {
        "dev": "cross-env NODE_ENV=development webpack",
        "build": "cross-env NODE_ENV=production webpack"
    }
}
```

c.webpack.config.js添加获取env
```javascript
const isDev = process.env.NODE_ENV === 'development';
```

##### 3.配置mode(自动启用不同环境的配置优化)
```javascript
//webpack.config.js
const isDev = process.env.NODE_ENV === 'development';

modue.exports = {
    //...
    mode: isDev ? 'development' : 'production'
    plugins: []
}
```

##### 4.HTMLWebpackPlugin(自动生成入口html，动态插入script标签)
a.安装
```shell
npm install html-webpack-plugin -D
```

b.新建模版文件template.ejs
```html
<!DOCTYPE html>
    <!-- Date: <%= htmlWebpackPlugin.options.banner.date %> --> 
    <!-- Branch: <%= htmlWebpackPlugin.options.banner.branch %> --> 
    <!-- Tag: <%= htmlWebpackPlugin.options.banner.tag %> -->
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
        <meta name="apple-mobile-web-app-capable" content="yes">
        <meta name="apple-mobile-web-app-status-bar-style" content="black">
        <meta name="format-detection" content="telephone=no">
        <meta http-equiv="cleartype" content="on">
        <meta name="renderer" content="webkit">
        <title><%= htmlWebpackPlugin.options.title || '' %></title>
        <script>window.webpackPublicPath = '';</script> 
    </head>

    <body ontouchstart>
        
        <div id="app"></div>
        <% if (htmlWebpackPlugin.options.IS_EXTERNALS !== false) { %>
    <script crossorigin src="https://assets.gaiaworkforce.com/libs/jquery/3.3.1/jquery.min.js"></script>
    <script crossorigin src="https://assets.gaiaworkforce.com/libs/sjcl/1.0.8/sjcl.min.js"></script>
<% } %> 
    </body>
</html>
```

c.修改webpack.config.js
```javascript
//首先引入插件
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ROOT_PATH = path.resolve(__dirname);

module.exports = {
    //...
    plugins: [
        new HtmlWebpackPlugin({
            template: `${ROOT_PATH}/template.ejs`,
            filename: 'index.html', //打包后的文件名
            minify: {
                removeAttributeQuotes: false, //是否删除属性的双引号
                collapseWhitespace: false, //是否折叠空白
            },
            // hash: true //是否加上hash，默认是 false
        })
    ]
}
```

##### 5.配置webpack-dev-server
a.安装
```shell
npm install webpack-dev-server -D
```

b.修改package.json
```json
"scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server",
    "build": "cross-env NODE_ENV=production webpack"
},
```

c.修改webpack.config.js
```javascript
//webpack.config.js
module.exports = {
    //判断env是否为development
    devServer: {
        port: '3000', //默认是8080
        host: '0.0.0.0', //局域网都可访问
        open: true,
        quiet: false, //默认不启用
        inline: true, //默认开启 inline 模式，如果设置为false,开启 iframe 模式
        stats: "errors-only", //终端仅打印 error
        overlay: false, //默认不启用
        clientLogLevel: "silent", //日志等级
        compress: true, //是否启用 gzip 压缩
        disableHostCheck: true,
    }
}
```

##### 6.devtool(source map: https://awdr74100.github.io/2020-04-02-webpack-devtool/)
```javascript
//webpack.config.js
module.exports = {
    devtool: 'cheap-module-eval-source-map' //开发环境下使用
}
```

##### 7.处理样式文件
a.安装
```shell
npm install style-loader less-loader css-loader postcss-loader autoprefixer less -D
```

b.修改webpack.config.js
```javascript
//webpack.config.js
module.exports = {
    //...
    module: {
        rules: [
            {
                test: /\.(le|c)ss$/,
                use: ['style-loader', 'css-loader', {
                    loader: 'postcss-loader',
                    options: {
                        postcssOptions: {
                            plugins: function () {
                                return [
                                    require('autoprefixer')({
                                        "overrideBrowserslist": [
                                            ">0.25%",
                                            "not dead"
                                        ]
                                    })
                                ]
                            }
                        },
                    }
                }, 'less-loader'],
                exclude: /node_modules/
            }
        ]
    }
}
```

##### 8.抽离CSS为独立的文件
a.安装
```shell
npm install mini-css-extract-plugin -D
```

b.修改webpack.config.js
```javascript
//webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/[name].css' //个人习惯将css文件放在单独目录下
        })
    ],
    module: {
        rules: [
            {
                test: /\.(le|c)ss$/,
                use: [
                    MiniCssExtractPlugin.loader, //替换之前的 style-loader
                    'css-loader', {
                        loader: 'postcss-loader',
                        options: {
                            plugins: function () {
                                return [
                                    require('autoprefixer')({
                                        "overrideBrowserslist": [
                                            "defaults"
                                        ]
                                    })
                                ]
                            }
                        }
                    }, 'less-loader'
                ],
                exclude: /node_modules/
            }
        ]
    }
}
```

##### 9.配置(.browserlistrc: https://github.com/browserslist/browserslist)
a.新建
```shell
last 2 version
> 0.25%
not dead
```

b.修改webpack.config.js
```javascript
//webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
module.exports = {
    //...
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/[name].css' 
        })
    ],
    module: {
        rules: [
            {
                test: /\.(c|le)ss$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader', {
                        loader: 'postcss-loader',
                        options: {
                            plugins: function () {
                                return [
                                    require('autoprefixer')()
                                ]
                            }
                        }
                    }, 'less-loader'
                ],
                exclude: /node_modules/
            },
        ]
    }
}
```

##### 10.压缩CSS
a.安装
```shell
npm install optimize-css-assets-webpack-plugin -D
```

b.配置
```javascript
const OptimizeCssAssetsPlugin = require("optimize-css-assets-webpack-plugin");

module.exports = {
    //...
    plugins: [
        new OptimizeCssAssetsPlugin()
    ]
}
```

c.解决css source-map不生成的[问题](https://awdr74100.github.io/2020-07-06-webpack-optimizecssassetswebpackplugin/)
```javascript
new OptimizeCssAssetsPlugin({
      cssProcessorOptions: {
        map: {
          inline: false,
          annotation: true,
        },
      },
    })
```

##### 11.图片/字体文件处理
a.安装file-loader、url-loader
```shell
npm install file-loader url-loader -D
```

b.修改webpack.config.js
```javascript
//webpack.config.js
module.exports = {
    //...
    modules: {
        rules: [
            {
                test: /\.(png|jpg|gif|jpeg|webp|svg|eot|ttf|woff|woff2)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            limit: 8192, //8K
                            esModule: false,
                            name: '[name]_[hash:6].[ext]'
                        }
                    }
                ],
                exclude: /node_modules/
            }
        ]
    }
}
```

##### 12.处理html中的本地图片
```html
<!-- index.html -->
<img src="<%= require('./thor.jpeg') %>" />
```

##### 13.配置output路径
```javascript
//webpack.config.js
module.exports = {
    output: {
        path: path.resolve(__dirname, 'dist'), //必须是绝对路径
        filename: '[name].[chunkhash:8].bundle.js',
        publicPath: '/' //通常是CDN地址
    }
}
```

##### 14.每次打包前清空dist目录
a.安装
```shell
npm install clean-webpack-plugin -D
```

b.配置
```javascript
//webpack.config.js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    //...
    plugins: [
        //不需要传参数喔，它可以找到 outputPath
        new CleanWebpackPlugin() 
    ]
}
```

##### 15.拷贝无需webpack处理的文件
a.安装
```shell
npm install copy-webpack-plugin -D
```

b.配置
```javascript
const CopyWebpackPlugin = require("copy-webpack-plugin");

plugins: [
	new CopyWebpackPlugin({
      patterns: [
        {
          from: "public/js/*.js",
          to: path.resolve(__dirname, "dist", "js"),
          flatten: true,
        },
      ],
    }),
]
```

##### 16.多页应用打包
```javascript
//webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
    entry: {
        index: './src/index.js',
        login: './src/login.js'
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[hash:6].js'
    },
    //...
    plugins: [
        new HtmlWebpackPlugin({
            template: './public/index.html',
            filename: 'index.html', //打包后的文件名
            chunks: ['index'], //只引入index.js
        }),
        new HtmlWebpackPlugin({
            template: './public/login.html',
            filename: 'login.html', //打包后的文件名
            chunks: ['login'], //只引入login.js
        }),
    ]
}
```

##### 17.resolve配置
a.alias别名
```javascript
//webpack.config.js
module.exports = {
    //....
    resolve: {
        alias: {
            'react-native': '@my/react-native-web' //这个包名是我随便写的哈
        }
    }
}
```

b.使用
```javascript
import { View, ListView, StyleSheet, Animated } from 'react-native';
```

c.extensions
```javascript
resolve: {
        alias: {
            'react-native': '@my/react-native-web' //这个包名是我随便写的哈
        },
       	extensions: ['.js', '.jsx', '.ts', '.tsx']
}
```

##### 18.不同环境使用不同配置文件
a.安装
```shell
npm install webpack-merge -D
```

b.配置
```javascript
const { merge } = require("webpack-merge");
const baseWebpackConfig = require("./webpack.config.base");

module.exports = merge(baseWebpackConfig, {
  mode: "development",
  devtool: "cheap-module-eval-source-map",
  devServer: {
    port: "3000", //默认是8080
    quiet: false, //默认不启用
    inline: true, //默认开启 inline 模式，如果设置为false,开启 iframe 模式
    stats: "errors-only", //终端仅打印 error
    overlay: false, //默认不启用
    clientLogLevel: "silent", //日志等级
    compress: true, //是否启用 gzip 压缩
  },
});
```

c.修改package.json
```json
"scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server --open --config=webpack.config.dev.js",
    "build": "cross-env NODE_ENV=production webpack --config=webpack.config.prod.js"
  },
```

##### 19.dev-server代理设置
a.新建server.js
```javascript
//server.js
let express = require('express');

let app = express();

app.get('/user', (req, res) => {
    res.json({name: '刘小夕'});
});

app.listen(4000);
```

b.index.js添加请求
```javascript
//需要将 localhost:3000 转发到 localhost:4000（服务端） 端口
fetch("/api/user")
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(err => console.log(err));
```

c.修改webpack配置
```javascript
//webpack.config.js
module.exports = {
    //...
    devServer: {
        proxy: {
            '/api': {
                target: 'http://localhost:4000',
                pathRewrite: {
                    '/api': ''
                }
            }
        }
    }
}
```

参考：

[带你深度解锁Webpack系列(基础篇)](https://juejin.cn/post/6844904079219490830)

[带你深度解锁Webpack系列(进阶篇)](https://juejin.cn/post/6844904084927938567)

[带你深度解锁Webpack系列(优化篇)](https://juejin.cn/post/6844904093463347208)

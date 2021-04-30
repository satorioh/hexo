---
title: Babel 基础
date: 2021-04-30 11:01:17
categories: Babel
tags:
- babel
permalink: babel-basic
---
#### 一、功能
- 语法转换：将ES6版本的代码转为ES5等向后兼容的JS代码，从而可以运行在低版本浏览器或其它环境中
- 补齐API：通过 Polyfill 的方式在目标环境中添加缺失的特性
<!--more-->
polyfill：广义上讲是为环境提供不支持的特性的一类文件或库，狭义上讲是polyfill.js文件以及@babel/polyfill这个npm包

#### 二、相关依赖包
- @babel/core：核心包，babel版本通知指代它
- @babel/cli：命令行转码工具，如果我们使用命令行进行Babel转码就需要安装它
- @babel/preset-env：提供了ES6转换ES5的语法转换规则，以及polyfill的部分引入功能
- @babel/runtime：提供辅助函数模块
- @babel/plugin-transform-runtime：自动替换辅助函数
- @babel/runtime-corejs3：这个npm包里除了包含Babel做语法转换的辅助函数，也包含了core-js的API转换函数

#### 三、配置文件
##### 1.概念
配置文件推荐用.js格式，方便做逻辑处理

配置文件总结起来就是配置plugins和presets这两个数组，我们分别称之为插件数组和预设数组

plugins插件数组和presets预设数组是有顺序要求的。如果两个插件或预设都要处理同一个代码片段，那么会根据插件和预设的顺序来执行。规则如下：
- 插件比预设先执行
- 插件执行顺序是插件数组从前向后执行
- 预设执行顺序是预设数组从后向前执行

示例：
```javascript
{
    "presets": [
      [
        "@babel/preset-env", // 名称
        {
          "useBuiltIns": "entry" // 参数
        }
      ]
    ]
  }
```

##### 2.preset和plugin选择
Babel官方的preset，一般的react项目实际可能会用到的就只有3个:
- @babel/preset-env
- @babel/preset-react
- @babel/preset-typescript
  

  目前比较常用的插件只有@babel/plugin-transform-runtime

#### 四、补齐API的方式
Babel V7.4.0 版本开始，@babel/polyfill 已经被废弃，官方推荐单独安装 core-js 和 regenerator-runtime 模块
```javascript
npm install --save core-js@3 regenerator-runtime
```

##### 1.从前端入口文件引入
```javascript
import '@babel/polyfill';
// 或者
import "core-js/stable";
import "regenerator-runtime/runtime";
```

##### 2.从webpack配置引入
```javascript
const path = require('path');
  module.exports = {
    entry: ['@babel/polyfill', './a.js'], 
      // 或者['core-js/stable', 'regenerator-runtime/runtime', './a.js']
    output: {
      filename: 'b.js',
      path: path.resolve(__dirname, '')
    },
    mode: 'development'
  };
```

##### 3.从预设或插件引入（可实现polyfill部分引入，不污染全局，具体参考下文）

#### 五、@babel/preset-env介绍
##### 1.作用
主要作用是对我们所使用的、并且目标浏览器中缺失的功能进行代码转换和加载 polyfill

##### 2.参数项
(1) targets: 作用和写法与 [browserslist](https://github.com/browserslist/browserslist) 一样
```javascript
module.exports = {
    presets: [["@babel/env", {
      targets: {
        "chrome": "58",
        "ie": "11"
      }
    }]],
    plugins: []
}
```
说明：如果我们对targets参数项进行了设置，那么就不使用browserslist的配置。如不设置targets，那么就使用browserslist的配置。如果targets不配置，browserslist也没有配置，那么@babel/preset-env就对所有ES6语法转换成ES5的

正常情况下，我们推荐使用browserslist的配置而很少单独配置@babel/preset-env的targets

(2) useBuiltIns: 主要和polyfill的行为有关，取值可以是"usage" 、 "entry" 或 false(默认)
- false: 全部引入polyfill
- entry: 需要我们在项目入口处手动引入polyfill；会引入目标环境缺失的API模块
- usage: 不需要在项目入口处手动引入polyfill；会引入目标环境缺失的且代码中用到的API模块

(3) corejs: 取值可以是2(默认)或3，只有useBuiltIns设置为'usage'或'entry'时，才会生效

#### 六、@babel/plugin-transform-runtime介绍
##### 1.作用
移除重复的辅助函数、以不污染全局的方式引入polyfill(主要是给开发JS库或npm包等的人用的)

##### 2.作用与示例
(1)自动移除语法转换后内联的辅助函数（inline Babel helpers），使用@babel/runtime/helpers里的辅助函数来替代

a.安装依赖
```
npm install --save @babel/runtime // 提供辅助函数模块
npm install --save-dev @babel/plugin-transform-runtime // 自动替换辅助函数
```

b.配置
```javascript
{
  "presets": [
    [
      "@babel/preset-env"
    ]
  ],
  "plugins": [
    "@babel/plugin-transform-runtime"
  ]
}
```

(2)API转换：当代码里使用了core-js的API，自动引入@babel/runtime-corejs3/core-js-stable/，以此来替代全局引入的core-js/stable，避免全局污染

a.安装依赖
```
npm install --save @babel/runtime-corejs3
```

b.配置
```javascript
{
    "presets": [
      "@babel/env"
    ],
    "plugins": [
      ["@babel/plugin-transform-runtime", {
        "corejs": 3
      }]
    ]
}
```

(3)API转换：当代码里使用了Generator/async函数，自动引入@babel/runtime/regenerator，以此来替代全局引入的regenerator-runtime/runtime，避免全局污染（此功能默认开启）

##### 3.安装与配置规则总结
- 如果你不需要对core-js做API转换，那就安装@babel/runtime并把corejs配置项设置为false即可。

- 如果你需要用core-js 2做API转换，那就安装@babel/runtime-corejs2并把corejs配置项设置为2即可。

- 如果你需要用core-js 3做API转换，那就安装@babel/runtime-corejs3并把corejs配置项设置为3即可。

参考文章：

- [Babel教程](https://www.jiangruitao.com/babel/)
- [不容错过的 Babel7 知识](https://juejin.cn/post/6844904008679686152)
- [99% 开发者没弄明白的 babel 知识](https://developer.aliyun.com/article/783477)

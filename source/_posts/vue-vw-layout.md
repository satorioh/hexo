---
title: 【Vue】移动端适配之vw解决方案
date: 2018-06-13 16:37:21
categories: VUE
tags:
- vue
- 前端
permalink: vue-vw-layout
---
主要参考了大漠老师的 [《再聊移动端页面的适配》](https://www.w3cplus.com/css/vw-for-layout.html)和 [《如何在Vue项目中使用vw实现移动端适配》](https://www.w3cplus.com/mobile/vw-layout-in-vue.html)，目前vw移动端适配的相关文章都比较类似，在这里做个记录<!--more-->

#### 一、建立vue项目
#### 二、安装并配置PostCss插件
1.安装
```shell
npm i postcss-aspect-ratio-mini postcss-px-to-viewport postcss-write-svg postcss-cssnext postcss-viewport-units cssnano cssnano-preset-advanced --save-dev
```

2.配置PostCss插件

在根目录的.postcssrc.js：
```js
// https://github.com/michael-ciniawsky/postcss-load-config

module.exports = {
  "plugins": {
    "postcss-import": {},//解决@import引入路径问题
    "postcss-url": {},//该插件主要用来处理文件，比如图片文件、字体文件等引用路径的处理
    "postcss-aspect-ratio-mini": {},
    "postcss-write-svg": {//移动端1px解决方案 
      utf8: false
    },
    "postcss-cssnext": {},
    "postcss-px-to-viewport": {
      viewportWidth: 750, // 视窗的宽度，对应的是我们设计稿的宽度，一般是750
      viewportHeight: 1334, //视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置著作权归作者所有。
      unitPrecision: 3, // 指定`px`转换为视窗单位值的小数位数（很多时候无法整除）著作权归作者所有。
      viewportUnit: 'vw', // 指定需要转换成的视窗单位，建议使用vw
      selectorBlackList: ['.ignore','.hairlines'], // 指定不转换为视窗单位的类，可以自定义，可以无限添加,建议定义一至两个通用的类名著作权归作者所有。
      minPixelValue: 1, // 小于或等于`1px`不转换为视窗单位，你也可以设置为你想要的值著作权归作者所有。
      mediaQuery: false // 允许在媒体查询中转换`px`
    },
    "postcss-viewport-units": {}, //给css添加content的属性，配合viewport-units-buggyfill库解决适配问题
    "cssnano": {
      preset: "advanced",
      autoprefixer: false,
      "postcss-zindex": false
    }
  }
}
```

#### 三、引入viewport-units-buggyfill解决兼容问题
1.在index.html中引入
```html
<script src="//g.alicdn.com/fdilab/lib3rd/viewport-units-buggyfill/0.6.2/??viewport-units-buggyfill.hacks.min.js,viewport-units-buggyfill.min.js"></script>
    <script>
      window.onload = function () { 
        window.viewportUnitsBuggyfill.init({ hacks: window.viewportUnitsBuggyfillHacks });
      }
    </script>
```

#### 四、问题解决
1.img图片不显示：

全局引入
```css
img { content: normal !important; }
```

2.与第三方UI库兼容问题：

使用postcss-px-to-viewport-opt，然后使用exclude配置项，具体参考 [Vue+ts下的移动端vw适配（第三方库css问题）](https://zhuanlan.zhihu.com/p/36913200)

参考链接：

[再聊移动端页面的适配](https://www.w3cplus.com/css/vw-for-layout.html)

[如何在Vue项目中使用vw实现移动端适配](https://www.w3cplus.com/mobile/vw-layout-in-vue.html)

[Vue+ts下的移动端vw适配（第三方库css问题）](https://zhuanlan.zhihu.com/p/36913200)
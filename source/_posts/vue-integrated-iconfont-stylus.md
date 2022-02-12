---
title: 【Vue】使用iconfont和stylus
date: 2018-05-29 16:50:52
categories: VUE
tags:
- vue
- 前端
permalink: vue-integrated-iconfont-stylus
---
#### 一、使用iconfont
1.在 [iconfont](http://www.iconfont.cn)新建项目，添加需要的图标
2.选择“Unicode”，并下载至本地
3.解压后，将4个字体文件和iconfont.css放到src/assets/styles下
<!--more-->
4.根据情况，修改iconfont.css中的字体文件路径
5.在main.js中引入iconfont.css
```js
import '@/assets/styles/iconfont.css'
```
6.在[iconfont](http://www.iconfont.cn)中复制unicode代码，然后在模板中使用iconfont
```html
<span class="iconfont">&#xe632;</span>
```

#### 二、使用stylus
1.安装依赖
```bash
npm install stylus --save
npm install stylus-loader --save
```

2.在模板样式中使用
```html
<style lang="stylus" scoped></style>
```

3.使用变量

(1)在src/assets/styles下创建varibles.styl文件
(2)写入需复用的变量
```css
$bgColor = #00bcd4
```
(3)配置路径别名：在build/webpack.base.conf.js中，找到resolve选项，修改如下
```js
resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
      'styles': resolve('src/assets/styles'),
    }
  }
```
(4)修改完后，需重启server，然后`npm run dev`

(5)在模板样式中引入varibles.styl，然后使用变量
```css
@import '~styles/varibles.styl'

.header
  display: flex
  line-height: .86rem
  background : $bgColor
```
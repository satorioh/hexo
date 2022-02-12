---
title: 【Vue】keep-alive性能优化与activated生命周期钩子
date: 2018-06-03 16:27:29
categories: VUE
tags:
- vue
- 前端
permalink: vue-keepalive-activated-lifecycle
---
#### 一、keep-alive介绍
引用官网：
> keep-alive 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们...当组件在 keep-alive 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。
<!--more-->

#### 二、使用
在App.vue中添加keep-alive，缓存后端返回的json数据，减少xhr请求
```vue
<template>
  <div id="app">
    <keep-alive>
      <router-view/>
    </keep-alive>
  </div>
</template>
```

#### 三、配合activated生命周期钩子
当路由跳转回页面时，判断城市是否改变，从而发送xhr请求
```vue
import { mapState } from 'vuex'

data () {
    return {
      lastCity: ''
    }
  },
computed: {
    ...mapState(['city'])
  },
methods: {
    getHomeInfo () {
      axios.get('/api/index.json?city=' + this.city)
        .then(this.getHomeInfoSuccess)
    },
  },
mounted () {
    this.lastCity = this.city
    this.getHomeInfo()
  },
activated () {
    if (this.lastCity !== this.city) {
      this.lastCity = this.city
      this.getHomeInfo()
    }
  }
```

#### 四、排除某些组件
使用exclude标签，来排除某个组件，不缓存，这样mounted钩子也可以正常使用。

app.vue:
```vue
<template>
  <div id="app">
    <keep-alive exclude="Detail">
      <router-view/>
    </keep-alive>
  </div>
</template>
```
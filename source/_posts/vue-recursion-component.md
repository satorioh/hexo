---
title: 【Vue】递归组件
date: 2018-06-05 16:14:17
categories: VUE
tags:
- vue
- 前端
permalink: vue-recursion-component
---
组件是可以在它们自己的模板中调用自身的。不过它们只能通过 name 选项来做这件事
<!--more-->
#### 一、数据结构
```js
list: [{
        title: '成人票',
        children: [{
          title: '成人三馆联票'
        }, {
          title: '成人五馆联票'
        }]
      }, {
        title: '学生票'
      }, {
        title: '儿童票'
      }, {
        title: '特惠票'
      }]
```

#### 二、递归组件
```vue
<template>
  <div>
    <div class="item" v-for="(item,index) of list" :key="index">
      <div class="item-title border-bottom"><span class="item-title-icon"></span>{{item.title}}</div>
      <div v-if="item.children">
        <detail-list :list="item.children" class="item-children"></detail-list>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DetailList',
  props: {
    list: Array
  }
}
</script>
```
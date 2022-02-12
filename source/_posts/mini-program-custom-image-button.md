---
title: 【微信小程序】自定义image button组件
date: 2019-04-12 14:56:26
categories: 小程序
tags:
- 微信小程序
- 前端
permalink: mini-program-custom-image-button
---
##### imageButtonCmp.wxml
```html
<!--components/image-button/imageButtonCmp.wxml-->
<button bind:getuserinfo="onGetUserInfo"
        open-type='{{openType}}'  plain='{{true}}'
        class="container">
  <slot name="img"></slot>
</button>
```
<!--more-->

##### imageButtonCmp.wxss
```css
.container{
  padding: 0 !important;
  border:none !important;
}
```

##### imageButtonCmp.js
```javascript
// components/image-button/imageButtonCmp.js
Component({
  options:{
    multipleSlots: true
  },
  /**
   * 组件的属性列表
   */
  properties: {
    openType:{
      type: String
    }
  },

  /**
   * 组件的初始数据
   */
  data: {

  },

  /**
   * 组件的方法列表
   */
  methods: {
    onGetUserInfo(e){
      this.triggerEvent('getuserinfo', e.detail, {})
    },
  }
})
```

##### 使用
```html
<v-button class="share-btn" open-type="share">
    <image class="share" slot="img" src="/images/icon/share.png" />
</v-button>

<v-button wx:if="{{!authorized}}" open-type="getUserInfo" class="avatar-position" bind:getuserinfo="onGetUserInfo">
    <image slot="img" class="avatar" src="/images/my/my.png" />
</v-button>
```

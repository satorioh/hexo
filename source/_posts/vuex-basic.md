---
title: 【Vue】Vuex的基本使用
date: 2018-06-03 14:29:59
categories: VUE
tags:
- vue
- 前端
permalink: vuex-basic
---
#### 一、安装
`npm install vuex --save`

#### 二、配置
1.在src下新建store文件夹，新增index.js文件

2.在index.js中写入配置代码，如下:
<!--more-->
```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
export default new Vuex.Store({

})
```

3.在main.js中导入
```js
import store from './store'

new Vue({
  el: '#app',
  router,
  store,  //导入
  components: { App },
  template: '<App/>'
})
```

#### 三、在组件中使用
1.在methods中使用
```vue
methods: {
    handlerCityClick (city) {
      this.$store.dispatch('changeCity', city)
      this.$router.push('/')
    }
  }
```

2.在store/index.js中添加对应的state、actions和mutations
```js
export default new Vuex.Store({
  state: {
    city: '北京'
  },
  actions: {
    changeCity (ctx, city) {
      ctx.commit('changeCity', city)
    }
  },
  mutations: {
    changeCity (state, city) {
      state.city = city
    }
  }
})
```

#### 四、优化拆分store/index.js
在store文件夹下分别新增state.js、actions.js、mutations.js，将index.js分拆

以actions.js为例:
```js
export default {
  changeCity (ctx, city) {
    ctx.commit('changeCity', city)
  }
}
```

index.js:
```js
import Vue from 'vue'
import Vuex from 'vuex'
import state from './state'
import actions from './actions'
import mutations from './mutation'

Vue.use(Vuex)
export default new Vuex.Store({
  state,
  actions,
  mutations
})
```
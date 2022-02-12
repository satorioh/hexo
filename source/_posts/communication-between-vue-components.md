---
title: 【Vue】组件间通信
date: 2018-07-03 11:06:16
categories: VUE
tags:
- vue
- 前端
permalink: communication-between-vue-components
---
### 一、父组件->子组件
**1.属性绑定**

(1)父组件
```vue
<template>
  <div id="app">
    <hello-world :message="message"></hello-world>
  </div>
</template>

<script>
data () {
    return {
      message: 'father message'
    }
  }
</script>
```
<!--more-->
(2)子组件
```vue
<template>
  <div>{{fatherMsg}}</div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: ['message'],
  data () {
    return {
      fatherMsg: this.message
    }
  }
}
</script>
```

**2.调用父链（$parent，$root**）

(1)父组件
```vue
<template>
  <div id="app">
    <hello-world></hello-world>
  </div>
</template>

<script>
data () {
    return {
      message: 'father message'
    }
  }
</script>
```
(2)子组件
```vue
<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      fatherMsg: this.$parent.message
    }
  }
}
</script>
```

### 二、子组件->父组件
**1.事件绑定**

(1)子组件
```vue
<script>
export default {
  name: 'HelloWorld',
  mounted () {
    this.$emit('init', 'child message')
  }
}
</script>
```
(2)父组件
```vue
<template>
  <div id="app">
    <hello-world @init="handleInit"></hello-world>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld'
export default {
  name: 'App',
  components: {
    'hello-world': HelloWorld
  },
  methods: {
    handleInit (msg) {
      console.log(msg)
    }
  }
}
</script>
```
**2.子组件索引**

(1)子组件
```vue
<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      message: 'child message'
    }
  }
}
</script>
```
(2)父组件
```vue
<template>
  <div id="app">
    <hello-world ref="child1"></hello-world>
  </div>
</template>

mounted () {
    console.log(this.$refs.child1.message)
  }
```

### 三、非父子组件间
**1.Bus中央通信**

(1)设置Bus组件
```vue
<script>
import Vue from 'vue'
export default new Vue()
</script>
```
(2)组件1发送事件
```vue
<template>
    <button @click="handleClick">brother</button>
</template>

<script>
import Bus from './Bus'
export default {
  name: 'brother',
  methods: {
    handleClick () {
      Bus.$emit('change', 'brother1 message')
    }
  }
}
</script>
```
(3)组件2监听事件
```vue
<template>
  <div>message:{{message}}</div>
</template>

<script>
import Bus from './Bus'
export default {
  name: 'HelloWorld',
  data () {
    return {
      message: ''
    }
  },
  created () {
    var _this = this
    Bus.$on('change', function (msg) {
      _this.message = msg
    })
  }
}
</script>
```

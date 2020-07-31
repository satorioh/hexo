---
title: 【学习小结】JSONP原理与实现
date: 2018-01-30 11:22:04
categories: 跨域
tags:
- jsonp
- 跨域
permalink: jsonp-learning-summary
---
### 一、原理
我的理解：因为script标签（类似的还有img，iframe）不受同源策略限制，故可以用于跨域数据的获取，这是根本原因。

客户端设置一个script，其url指向支持JSONP的后端API，再声明一个callback函数，函数名需与url中?callback=字段值相同。当包含特定url的script载入时，会向后端API发起请求，后端会将数据包裹进一个函数的参数里，当然这个函数的名称是根据前端url参数自动生成的，前端script载入完成则自动调用已有的callback函数，对接受到的数据（此刻是函数参数）做出处理，从而实现了跨域。<!--more-->

### 二、实现
#### 1.JS实现
```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>test</title>
</head>
<body>
<script>
  "use strict";
  let test = function (data) {//返回数据后的处理函数
    console.log(data);
  };

  let url = "http://cache.video.iqiyi.com/jp/avlist/202861101/1/?callback=test";//支持JSONP的API

  let script = document.createElement("script");
  script.setAttribute("src",url);
  document.getElementsByTagName("body")[0].appendChild(script);
</script>
</body>
</html>
```

#### 2.jQuery实现
```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>test</title>
</head>
<body>
<script src="jquery.js"></script>
<script>
  "use strict";
  $(function () {
    $.ajax({
      url: "http://cache.video.iqiyi.com/jp/avlist/202861101/1/",
      dataType: "jsonp",
      jsonp: "callback",
      success: function (data) {
        console.log(data);
      },
      error: function(){
        alert("fail");
      }
    })
  })
</script>
</body>
</html>
```
注：jQuery的callback函数名是随机生成的，返回的数据直接放入success函数处理，很方便。

### 三、JSONP与AJAX的区别
我的理解：

1.AJAX基于xhr对象实现，而JSONP是通过script标签动态加载后端js实现，原理不同
2.AJAX支持get/post，而JSONP因为需要在script的url中传参，所以只支持get

### 四、JSONP优缺点
优点：极好的浏览器兼容性

缺点：存在脚本注入的安全隐患

参考：

[【原创】说说JSON和JSONP，也许你会豁然开朗，含jQuery用例](http://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html)

[5 分钟彻底明白 JSONP](https://tonghuashuo.github.io/blog/jsonp.html)

[JSONP原理优缺点(只能GET不支持POST)](https://www.jianshu.com/p/9a5a6f853fa8)

---
title: 【JS】解决跨域的几种方法
date: 2018-06-15 14:18:36
categories: 跨域
tags:
- js
- 前端
permalink: cors-methods
---
### 一、何为同源
所谓"同源"指的是"三个相同"：
- 协议相同
- 域名相同
- 端口相同

> http://www.example.com/dir2/other.html：同源
http://example.com/dir/other.html：不同源（域名不同）
http://v2.www.example.com/dir/other.html：不同源（域名不同）
http://www.example.com:81/dir/other.html：不同源（端口不同）
<!--more-->

### 二、限制范围
如果非同源，如下行为会受到限制：
- Cookie、LocalStorage 和 IndexDB 无法读取
- DOM 无法获得
- AJAX 请求不能发送

跨域类型：这里暂且分为两种：主域相同，子域不同的跨域和完全跨域

### 三、主域相同，子域不同的跨域
#### 1.共享Cookie：设置document.domain

**方法a:**
```javascript
// A页面：http://w1.example.com/a.html
document.domain = 'example.com';
document.cookie = "test1=hello";

// B页面：http://w2.example.com/b.html
document.domain = 'example.com';
var allCookie = document.cookie;
```

**方法b:**

服务器在设置Cookie的时候，指定Cookie的所属域名为一级域名，比如.example.com，这样的话，二级域名和三级域名不用做任何设置，都可以读取这个Cookie
```
Set-Cookie: key=value; domain=.example.com; path=/
```

#### 2.获取DOM、变量：设置document.domain + iframe
```html
<!--父窗口：(http://www.domain.com/a.html)-->
<iframe id="iframe" src="http://child.domain.com/b.html"></iframe>
<script>
    document.domain = 'domain.com';
    var user = 'admin';
</script>
```

```javascript
// 子窗口：(http://child.domain.com/b.html)
<script>
    document.domain = 'domain.com';
    // 获取父窗口中变量
    alert('get js data from parent ---> ' + window.parent.user);
</script>
```

### 四、完全跨域
##### 1.通信：片段标识符 + iframe
片段标识符（fragment identifier）指的是，URL的#号后面的部分，比如http://example.com/x.html#fragment的#fragment。如果只是改变片段标识符，页面不会重新刷新

```javascript
// 父窗口可以把信息，写入子窗口的片段标识符
var src = originURL + '#' + data;
document.getElementById('myIFrame').src = src;
```
```javascript
// 子窗口通过监听hashchange事件得到通知
window.onhashchange = checkMessage;

function checkMessage() {
  var message = window.location.hash;
  // ...
}
```
```javascript
// 子窗口也可以改变父窗口的片段标识符
parent.location.href= target + "#" + hash;
```

##### 2.通信：window.postMessage
用法：postMessage(data,origin)方法接受两个参数

data： html5规范支持任意基本类型或可复制的对象，但部分浏览器只支持字符串，所以传参时最好用JSON.stringify()序列化。

origin： 协议+主机+端口号，也可以设置为"*"，表示可以传递给任意窗口，如果要指定和当前窗口同源的话设置为"/"。

message事件的事件对象event，提供以下三个属性：
- event.source：发送消息的窗口
- event.origin: 消息发向的网址
- event.data: 消息内容

```html
<!--a.html：(http://www.domain1.com/a.html-->
<iframe id="iframe" src="http://www.domain2.com/b.html" style="display:none;"></iframe>
<script>
    var iframe = document.getElementById('iframe');
    iframe.onload = function() {
        var data = {
            name: 'aym'
        };
        // 向domain2传送跨域数据
        iframe.contentWindow.postMessage(JSON.stringify(data), 'http://www.domain2.com');
    };

    // 接受domain2返回数据
    window.addEventListener('message', function(e) {
        if (e.origin !== 'http://www.domain2.com') return;
        alert('data from domain2 ---> ' + e.data);
    }, false);
</script>
```

```javascript
// b.html：(http://www.domain2.com/b.html)
<script>
    // 接收domain1的数据
    window.addEventListener('message', function(e) {
        if (e.origin !== 'http://www.domain1.com') return;
        alert('data from domain1 ---> ' + e.data);

        var data = JSON.parse(e.data);
        if (data) {
            data.number = 16;

            // 处理后再发回domain1
            window.parent.postMessage(JSON.stringify(data), 'http://www.domain1.com');
        }
    }, false);
</script>
```
##### 3.LocalStorage: window.postMessage
```javascript
// 子窗口接收消息并操作localStorage
window.onmessage = function(e) {
  if (e.origin !== 'http://bbb.com') return;
  var payload = JSON.parse(e.data);
  switch (payload.method) {
    case 'set':
      localStorage.setItem(payload.key, JSON.stringify(payload.data));
      break;
    case 'get':
      var parent = window.parent;
      var data = localStorage.getItem(payload.key);
      parent.postMessage(data, 'http://aaa.com');
      break;
    case 'remove':
      localStorage.removeItem(payload.key);
      break;
  }
};
```
```javascript
// 父窗口发送消息并操作子
var win = document.getElementsByTagName('iframe')[0].contentWindow;
var obj = { name: 'Jack' };
// 存入对象
win.postMessage(JSON.stringify({key: 'storage', method: 'set', data: obj}), 'http://bbb.com');
// 读取对象
win.postMessage(JSON.stringify({key: 'storage', method: "get"}), "*");
window.onmessage = function(e) {
  if (e.origin != 'http://aaa.com') return;
  // "Jack"
  console.log(JSON.parse(e.data).name);
};
```

##### 4.Ajax: 通过jsonp跨域
只能发GET请求，原理可参考之前的 [这篇](https://roubin.me/jsonp-learning-summary/) ，原生js实现：
```js
<script>
    var script = document.createElement('script');
    script.type = 'text/javascript';

    // 传参并指定回调执行函数为onBack
    script.src = 'http://www.domain2.com:8080/login?user=admin&callback=onBack';
    document.head.appendChild(script);

    // 回调执行函数
    function onBack(res) {
        alert(JSON.stringify(res));
    }
 </script>
```
<!--more-->

jQuery实现：
```js
$.ajax({
    url: 'http://www.domain2.com:8080/login',
    type: 'get',
    dataType: 'jsonp',  // 请求方式为jsonp
    jsonpCallback: "onBack",    // 自定义回调函数名
    data: {}
});
```

##### 5.Ajax: 跨域资源共享（CORS）
普通跨域请求：只要服务端设置Access-Control-Allow-Origin即可，前端无须设置，若要带cookie请求：前后端都需要设置。

原生ajax实现：
```js
var xhr = new XMLHttpRequest(); // IE8/9需用window.XDomainRequest兼容

// 前端设置是否带cookie
xhr.withCredentials = true;

xhr.open('post', 'http://www.domain2.com:8080/login', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.send('user=admin');

xhr.onreadystatechange = function() {
    if (xhr.readyState == 4 && xhr.status == 200) {
        alert(xhr.responseText);
    }
};
```

jQuery ajax实现：
```js
$.ajax({
    ...
   xhrFields: {
       withCredentials: true    // 前端设置是否带cookie
   },
   crossDomain: true,   // 会让请求头中包含跨域的额外信息，但不会含cookie
    ...
});
```

参考链接：

[浏览器同源政策及其规避方法 - 阮一峰](https://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)

[前端常见跨域解决方案（全）](https://segmentfault.com/a/1190000011145364)

[10种跨域解决方案](https://juejin.im/post/5e948bbbf265da47f2561705)
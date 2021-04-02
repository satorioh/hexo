---
title: CORS请求总结
date: 2020-07-31 13:45:16
categories: 跨域
tags:
- 跨域
- cors
permalink: cors-summary
---
#### 一、简介
CORS需要浏览器和服务器同时支持，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信
<!--more-->
#### 二、简单请求
##### 1.满足条件
> (1) 请求方法是以下三种方法之一：
> - HEAD
> - GET
> - POST
>
> (2) HTTP的请求头信息不超出以下几种字段：
> - Accept
> - Accept-Language
> - Content-Language
> - Last-Event-ID
> - Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

##### 2.处理流程
(1)浏览器在请求头中，添加`Origin`字段，用来说明本次请求来自哪个源(协议 + 域名 + 端口)

(2)如果`Origin`字段指定的域名在服务器许可范围内，服务器返回的响应头，会多出几个字段：
```shell
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```

a.Access-Control-Allow-Origin：该字段是必须的。它的值要么是请求时Origin字段的值，要么是一个*，表示接受任意域名的请求

b.Access-Control-Allow-Credentials: 可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。同时，前端必须在AJAX请求中打开withCredentials属性：
```javascript
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;
```
如果要发送Cookie，`Access-Control-Allow-Origin`就不能设为星号，必须指定明确的、与请求网页一致的域名

`Access-Control-Allow-Credentials`也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。

c.Access-Control-Expose-Headers: 可选。跨域请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。上面的例子指定，getResponseHeader('FooBar')可以返回FooBar字段的值

#### 三、非简单请求
##### 1.预检请求
跨域请求的非简单请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight），"预检"请求用的请求方法是`OPTIONS`，预检请求通过后，浏览器才会发出正式的XMLHttpRequest请求，否则就报错

##### 2.处理流程
(1)浏览器发现，跨域请求是一个非简单请求，就自动发出一个"预检"请求，包含`Origin`、`Access-Control-Request-Method和Access-Control-Request-Headers

```shell
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
a.Origin: 表示请求来自哪个源

b.Access-Control-Request-Method: 该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是PUT

c.Access-Control-Request-Headers: 该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段，上例是X-Custom-Header

(2)服务器收到"预检"请求以后，检查Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段

如果未通过，服务器会返回一个正常的HTTP回应，但是没有任何CORS相关的头信息字段。这时，浏览器就会认定，服务器不同意预检请求，就抛出错误，被xhr的onerror回调捕获：
```shell
XMLHttpRequest cannot load http://api.alice.com.
Origin http://api.bob.com is not allowed by Access-Control-Allow-Origin
```

如果预检请求通过，服务器会回应如下字段：
```shell
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```

a.Access-Control-Allow-Methods: 该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求

b.Access-Control-Allow-Headers: 如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段

c.Access-Control-Allow-Credentials: 是否发送Cookie

d.Access-Control-Max-Age: 该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求

(3)一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个Origin头信息字段:
```shell
PUT /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
X-Custom-Header: value
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
服务器的回应，也都会有一个Access-Control-Allow-Origin头信息字段：
```shell
Access-Control-Allow-Origin: http://api.bob.com
Content-Type: text/html; charset=utf-8
```

#### 四、与JSONP比较
优点：JSONP只支持GET请求，CORS支持所有类型的HTTP请求
缺点：不支持老式浏览器，不支持CORS的服务器无法使用

#### 五、常见问题
##### 1.fetch设置mode: no-cors拿不到数据？
答：fetch设置no-cors依旧受跨域策略限制，只是浏览器不再报错，并且即使后端添加了相关header，也拿不到response

##### 2.前端跨域请求设置cookie后，报cors错误？
答：需要同时满足如下三个条件(后端Set-Cookie也是)：
- 后端Response header 有 Access-Control-Allow-Credentials: true
- 后端Response header的Access-Control-Allow-Origin不能是*，要明确指定
- 前端fetch 加上 credentials: 'include'

##### 3.无法获取跨域请求中设置的自定义header？
答：后端需要带上Access-Control-Expose-Headers这个字段

##### 4.使用PUT(非简单请求)报错？
答：如果前端要使用GET、HEAD以及POST以外的HTTP method发送请求的话，后端的preflight response header必须有Access-Control-Allow-Methods并且指定合法的method，preflight才会通过，浏览器才会把真正的request发送出去



参考链接：

[跨域资源共享 CORS 详解 -- 阮一峰](https://www.ruanyifeng.com/blog/2016/04/cors.html)

[CORS 完全手册之CORS 详解](https://mp.weixin.qq.com/s/y8e1HLNzbLLYWSeMnT-xSA)

[Fetch Living Standard](https://fetch.spec.whatwg.org/)

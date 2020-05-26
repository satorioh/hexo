---
title: 跨tab页通信(同源/跨域)
date: 2020-05-21 17:34:37
categories: JavaScript
tags:
- 跨域
- tab
permalink: cross-browser-tab-communication
---
#### 一、获取句柄 + postMessage
必须有一个页面（如A页面）可以获取另一个页面（如B页面）的window对象，这样才可以完成通信
```javascript
// parent.html
const childPage = window.open('child.html', 'child')

childPage.onload = () => {
	childPage.postMessage('hello', location.origin)
}

// child.html
window.onmessage = evt => {
	// evt.data
}
```
<!--more-->

#### 二、localStorage + storageEvent
同源的两个tab页面可以通过共享localStorage的方式通信
```javascript
// A 页面
window.addEventListener("storage", function(ev){
    if (ev.key == 'message') {
        // removeItem同样触发storage事件，此时ev.newValue为空
        if(!ev.newValue)
            return;
        var message = JSON.parse(ev.newValue);
        console.log(message);
    }
});

function sendMessage(message){
    localStorage.setItem('message',JSON.stringify(message));
    localStorage.removeItem('message');
}

// 发送消息给B页面
sendMessage('this is message from A');
```
```javascript
B 页面

window.addEventListener("storage", function(ev){
    if (ev.key == 'message') {
        // removeItem同样触发storage事件，此时ev.newValue为空
        if(!ev.newValue)
            return;
        var message = JSON.parse(ev.newValue);
        // 发送消息给A页面
        sendMessage('message echo from B');
    }
});

function sendMessage(message){
    localStorage.setItem('message',JSON.stringify(message));
    localStorage.removeItem('message');
}
```

#### 三、iframe + postMessage
通过在这两个tab页嵌入同一个iframe页实现“桥接”，最终完成两个不相关的tab页通信
```
tab A -----> iframe A[bridge.html]
                     |
                     |
                    \|/
             iframe B[bridge.html] ----->  tab B
```
```javascript
// tab A:

// 向弹出的tab页面发送消息
window.sendMessageToTab = function(data){
    // 由于[#J_bridge]iframe页面的源文件在vstudio服务器中，因此postMessage发向“同源”
    document.querySelector('#J_bridge').contentWindow.postMessage(JSON.stringify(data),'/');
};

// 接收来自 [#J_bridge]iframe的tab消息
window.addEventListener('message',function(e){
    let {data,source,origin}  = e;
    if(!data)
        return;
    try{
        let info = JSON.parse(JSON.parse(data));
        if(info.type == 'BSays'){
           console.log('BSay:',info);
        }
    }catch(e){
    }
});

sendMessageToTab({
    type: 'ASays',
    data: 'hello world, B'
})
```
```javascript
// bridge.html

window.addEventListener("storage", function(ev){
    if (ev.key == 'message') {
        window.parent.postMessage(ev.newValue,'*');
    }
});

function message_broadcast(message){
    localStorage.setItem('message',JSON.stringify(message));
    localStorage.removeItem('message');
}

window.addEventListener('message',function(e){
    let {data,source,origin}  = e;
    // 接受到父文档的消息后，广播给其他的同源页面
    message_broadcast(data);
});
```
```javascript
// tab B

window.addEventListener('message',function(e){
    let {data,source,origin}  = e;
    if(!data)
        return;
    let info = JSON.parse(JSON.parse(data));
    if(info.type == 'ASays'){
        document.querySelector('#J_bridge').contentWindow.postMessage(JSON.stringify({
            type: 'BSays',
            data: 'hello world echo from B'
        }),'*');
    }
});

// tab B主动发送消息给tab A
document.querySelector('button').addEventListener('click',function(){
    document.querySelector('#J_bridge').contentWindow.postMessage(JSON.stringify({
        type: 'BSays',
        data: 'I am B'
    }),'*');
})
```

参考：

[跨浏览器tab页的通信解决方案尝试](https://cloud.tencent.com/developer/article/1061884)

[跨页面通信的各种姿势](https://juejin.im/post/59bb7080518825396f4f5177)

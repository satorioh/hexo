---
title: 【JS】async/await
date: 2019-01-26 20:47:32
categories: JavaScript
tags:
- js
- 前端
permalink: async-await
---
#### 示例：
```javascript
// 将解决方法封装到 async 函数中
async function solution() {
   
    // 等待第一个 HTTP 请求并打印出结果
    console.log(await rp('http://example.com/'));
 
   
    // 创建两个 HTTP 请求，不等它们执行完 —— 让他们同时执行
    const call2Promise = rp('http://example.com/');  // Does not wait!
    const call3Promise = rp('http://example.com/');  // Does not wait!
 
    
    // 创建完以后 —— 等待它们都执行完
    const response2 = await call2Promise;
    const response3 = await call3Promise;
 
    console.log(response2);
    console.log(response3);
}
 
 
// 调用这一 async 函数
solution().then(() => console.log('Finished'));
```
<!--more-->

#### 错误处理：
```javascript
async function f() {
    try {
        const promiseResult = await Promise.reject('Error');
    } catch (e){
        console.log(e);
    }
}
```

#### 循环：
```javascript
async function asyncAwaitLoops () {
  const api = new Api()
  const user = await api.getUser()
  const friends = await api.getFriends(user.id)
  for (let friend of friends) {
    let moreFriends = await api.getFriends(friend.id)
    console.log('asyncAwaitLoops', moreFriends)
  }
}
```

#### 使用数组方法实现并行：
```javascript
async function asyncAwaitLoopsParallel () {
  const api = new Api()
  const user = await api.getUser()
  const friends = await api.getFriends(user.id)
  const friendPromises = friends.map(friend => api.getFriends(friend.id))
  const moreFriends = await Promise.all(friendPromises)
  console.log('asyncAwaitLoopsParallel', moreFriends)
}
```

```javascript
async function getLotsOfUserDataFaster () {
  try {
    const userPromises = Array(10).fill(getUserInfo())
    const users = await Promise.all(userPromises)
    console.log('getLotsOfUserDataFaster', users)
  } catch (err) {
    console.error(err)
  }
}
```

#### 等效于Promise.all的并行处理：
```javascript
async function A2() {
  let res = [];
  let reqs = [createPromise(), createPromise(), createPromise()];
  for (let i = 0; i< reqs.length; i++) {
    res[i] = await reqs[i];
  }
  console.log('Data', res);
}
```
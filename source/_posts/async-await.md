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

#### 一、执行顺序
##### 1.继发执行
```javascript
// 示例一
let foo = await getFoo();
let bar = await getBar();

// 示例二
async function logInOrder(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    console.log(await response.text());
  }
}
```
##### 2.并发执行
```javascript
// 示例一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 示例二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;

// 示例三
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}

// 示例四
async function logInOrder(urls) {
  // 并发读取远程URL
  const textPromises = urls.map(async url => {
    const response = await fetch(url);
    return response.text();
  });

  // 按次序输出
  for (const textPromise of textPromises) {
    console.log(await textPromise);
  }
}

// 示例五
async function A2() {
  let res = [];
  let reqs = [createPromise(), createPromise(), createPromise()];
  for (let i = 0; i< reqs.length; i++) {
    res[i] = await reqs[i];
  }
  console.log('Data', res);
}

// 示例六
async function asyncAwaitLoopsParallel () {
  const api = new Api()
  const user = await api.getUser()
  const friends = await api.getFriends(user.id)
  const friendPromises = friends.map(friend => api.getFriends(friend.id))
  const moreFriends = await Promise.all(friendPromises)
  console.log('asyncAwaitLoopsParallel', moreFriends)
}

// 示例七
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

#### 二、错误处理：
```javascript
async function f() {
    try {
        const promiseResult = await Promise.reject('Error');
    } catch (e){
        console.log(e);
    }
}
```
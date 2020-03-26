---
title: 【JS】常用函数
date: 2018-12-30 20:31:54
categories: JavaScript
tags:
- js
- 前端
permalink: js-util-functions
---
#### 1.数组求和
```javascript
function sum(arr){
  return arr.reduce((prev,val) => prev + val)
}
```
<!--more-->

#### 2.二维数组拉平
```javascript
function flat(arr) {
  return arr.reduce((pre, value) => {
    return Array.isArray(value) ? [...pre, ...flat(value)] : [...pre, value]
  }, [])

function flattenArray(arr) {
  const flattened = [].concat(...arr)
  return flattened.some(item => Array.isArray(item)) ? 
    flattenArray(flattened) : flattened
}
```

#### 3.对象数组-求某个属性的最大值
```javascript
function keyMax(arr, key) {
  return Math.max.apply(Math, arr.map( o => o[key]))
}
```

#### 4.对象数组-求包含某个属性最大值的对象
```javascript
function keyMaxObj(arr,key) {
  return arr.reduce((p,v) => p[key] < v[key] ? v : p)
}
```

#### 5.对象数组-根据对象某个属性值做对象排序(1:升序 0:降序)
```javascript
function compare(key, sortType) {
  return function(obj1,obj2){
          let a = obj1[key]
          let b = obj2[key]
          if (sortType) {
            return a - b     //升序
          } else {
            return b - a     //降序
          }
        }
}
objArray.sort(compare('value',1))
```

#### 6.普通数组去重
```javascript
function unique(arr){
  return Array.from(new Set(arr))
}
```

#### 7.根据某个属性做对象数组去重
```javascript
function  objArrayUnique(objArray,key){
    let hash = {}
    objArray = objArray.reduce(function(item, next) {
      hash[next[key]]?'':hash[next[key]] = true && item.push(next)
      return item
    }, [])
    return objArray
  }
```

#### 8.构建树
```javascript
/**
   * 将一维的扁平数组转换为多层级对象
   * @param  {[type]} list 一维数组，数组中每一个元素需包含id和parent_id两个属性
   * @sort 父节点在数组中的位置需要在子节点前，即 节点3必须排在节点3-2之前(可调用上面的compare方法)
   * @type 选择children为数组或对象
   * @return {[type]} tree 多层级树状结构
   */
buildTree: function (list, parentKey){
    let temp = {};
    let tree = {};
    for(let i in list){
      temp[list[i].id] = list[i];
    }
    for(let i in temp){
      if(temp[i][parentKey]) {
        if(!temp[temp[i][parentKey]].children) {
          temp[temp[i][parentKey]].children = new Array();
          // if (type === 'Object') temp[temp[i][parentKey]].children = new Object();
        }
        temp[temp[i][parentKey]].children.push(temp[i]);
        // temp[temp[i][parentKey]].children[temp[i].id] = temp[i];
      } else {
        tree[temp[i].id] =  temp[i];
      }
    }
    return tree;
  }
```

#### 9.多个对象数组-根据某个属性求并集
```javascript
function inUnion(list1 = [], list2 = [], key) {
    let arr = list1.concat(list2)
    return objArrayUnique(arr, key)
  }
```

#### 10.多个对象数组-根据某个属性求交集
```javascript
function inBoth(list1, list2, key) {
  var result = [];
  for (var i = 0; i < list1.length; i++) {
    var item1 = list1[i],
    found = false;
    for (var j = 0; j < list2.length && !found; j++) {
      found = item1[key] === list2[j][key];
    }
    if (found) {
      result.push(item1);
    }
  }
  return result;
}
```

#### 11.格式化时间戳
```javascript
function formatDate(timestamp, type = 'ymd') {
    let date = new Date(timestamp)
    let y = date.getFullYear()
    let m = date.getMonth() + 1
    m = m < 10 ? ('0' + m) : m
    let d = date.getDate()
    d = d < 10 ? ('0' + d) : d
    let h = date.getHours()
    h = h < 10 ? ('0' + h) : h
    let i = date.getMinutes()
    i = i < 10 ? ('0' + i) : i
    let s = date.getSeconds()
    s = s < 10 ? ('0' + s) : s

    switch (type) {
      case 'ymd': {
        return `${y}-${m}-${d}`
      }
      case 'ymdhi':{
        return `${y}-${m}-${d} ${h}:${i}`
      }
      default:{
        return `${y}-${m}-${d} ${h}:${i}:${s}`
      }
    }
  }
```

#### 12.yyyy-MM-dd转换成时间戳的格式
```javascript
function dateToTimestamp(dateStr){
    var newstr = dateStr.replace(/-/g,'/')   
    return Date.parse(new Date(newstr))
  }
```

#### 13.树状结构转扁平数组
```javascript
function nestedToFlat(children, parent) {
  var arr = []
  for (var i = 0; i < children.length; i++) {
    arr.push({ id: children[i].id,  isChecked: children[i].isChecked, parent: parent })
    if (children[i].children) arr = arr.concat(nestedToFlat(children[i].children, children[i].id))
  }
  return arr
},
```

#### 14.随机颜色
```javascript
function randomColor () {
  let rgb = []
  for (let i = 0; i < 3; i++) {
    rgb[i] = Math.round(255 * Math.random())
  }
  return 'rgb(' + rgb.join(',') + ')'
},
```

#### 15.检查是否是整数
```javascript
function isInteger(x) { 
  return (x ^ 0) === x 
}
```

#### 16.分页函数（一维数组转二维）
```javascript
function pages (arr, size) {
  const pages = []
  arr.forEach((item, index) => {
    const page = Math.floor(index / size)
    if (!pages[page]) {
      pages[page] = []
    }
    pages[page].push(item)
  })
  return pages
}
```

#### 17.全屏函数
```javascript
function launchFullScreen(element) {
    if (element.requestFullscreen) {
        element.requestFullscreen();
    } else if (element.mozRequestFullScreen) {
        element.mozRequestFullScreen();
    } else if (element.webkitRequestFullscreen) {
        element.webkitRequestFullscreen();
    } else if (element.msRequestFullscreen) {
        element.msRequestFullscreen();
    }
}
launchFullScreen(document.documentElement) //使用
```

#### 18.退出全屏函数
```javascript
function exitFullscreen() {
    if (document.exitFullscreen) {
        document.exitFullscreen();
    } else if (document.mozCancelFullScreen) {
        document.mozCancelFullScreen();
    } else if (document.webkitExitFullscreen) {
        document.webkitExitFullscreen();
    }  
}
```

#### 19.检测变量是否是对象
```javascript
function isObject(x) {
    return Object.prototype.toString.call(x) === '[object Object]';
}
```

#### 20.模拟sleep
```javascript
function sleep(milliSeconds) { // 毫秒
    const startTime = new Date().getTime();
    while (new Date().getTime() < startTime + milliSeconds);
}
```

#### 21.使用Blob实现点击下载
```javascript
function saveBlobFile(name, type, data) {
    if (data !== null && navigator.msSaveBlob) return navigator.msSaveBlob(new Blob([data], { type: type }), name);
    let a = $("<a style='display: none;'/>");
    let url = window.URL.createObjectURL(new Blob([data], { type: type }));
    a.attr('href', url);
    a.attr('download', name);
    $('body').append(a);
    a[0].click();
    window.URL.revokeObjectURL(url);
    a.remove();
}
```

#### 22.等待某个dom加载后修改内容
```javascript
function setLoadableTip(tip) {
    let timer = setInterval(() => {
        let $loadingText = $('.hr1-loading-text');
        if ($loadingText.length > 0) {
            $loadingText.text(i18next.t(tip));
            clearInterval(timer);
        }
    }, 100);
}
```

#### 23.银行卡号隐藏
```javascript
function formatBankAccount(account) {
    const pattern = /^(\d{4})(\d{4})+(\d+)$/;
    return account.replace(pattern, '$1 **** **** $3');
}
```

#### 24.字符串补足到指定长度
```javascript
function rightPad(str, targetLength, padChar) {
    return str + Array(targetLength - str.length + 1).join(padChar);
}
```

---
title: 【微信小程序】HTTP封装与使用示例
date: 2019-02-19 09:57:12
categories: 小程序
tags:
- 微信小程序
- 前端
permalink: mini-program-http-encapsulation
---
#### config.js
```javascript
const config = {
  api_base_url: 'http://bl.7yue.pro/v1/',
  appkey:'AbhC31IG7ruCDp57'
}

export {config}
```
<!--more-->

### 一、callback封装：
#### util/http.js
```javascript
import {
  config
} from '../config.js'

const tips = {
  1: '抱歉，出现了一个错误',
  1007: 'url路径错误',
  1005: '不正确的开发者key',
  3000: '该期内容不存在'
}

class HTTP {
  request(params) {
    if (!params.method) {
      params.method = 'GET'
    }
    wx.request({
      url: config.api_base_url + params.url,
      method: params.method,
      data: params.data,
      header: {
        'content-type': 'application/json',
        'appkey': config.appkey
      },
      success: (res) => {
        let code = res.statusCode.toString()
        if (code.startsWith('2')) {
          params.success && params.success(res.data)
        } else {
          let errorCode = res.data.error_code
          this._showError(errorCode)
        }
      },
      fail: (err) => {
        this._showError(1)
      }
    })
  }

  _showError(errorCode) {
    if (!errorCode) {
      errorCode = 1
    }
    wx.showToast({
      title: tips[errorCode],
      icon: 'none',
      duration: 2000
    })
  }
}

export {
  HTTP
}
```

#### models/classic.js
```javascript
import { HTTP } from '../util/http.js'

class ClassicModel extends HTTP {
  constructor() {
    super()
  }

  getLatest(callBack){
    this.request({
      url:'classic/latest',
      success:(data) => {
        callBack(data)
      }
    })
  }
}

export {ClassicModel}
```
#### models/like.js
```javascript
import { HTTP } from '../util/http.js'

class LikeModel extends HTTP {
  constructor() {
    super()
  }

  like(behavior, artId, type){
    let url = behavior == 'like' ? 'like' : 'like/cancel'
    this.request({
      url:url,
      method: 'POST',
      data: {
        art_id: artId,
        type: type
      },
    })
  }
}

export {LikeModel}
```

#### page/classic/classic.js
```javascript
import {ClassicModel} from '../../models/classic.js'
import {LikeModel} from '../../models/like.js'
let classicModel = new ClassicModel()
let likeModel = new LikeModel()

Page({

  /**
   * 页面的初始数据
   */
  data: {
    classicData:null,
    first:false,
    last:true,
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    classicModel.getLatest((data)=>{
      this.setData({
        classicData:data
      })
    })
  },

  handleLike (e) {
    let behavior = e.detail.behavior
    likeModel.like(behavior, this.data.classicData.id, this.data.classicData.type)
  },
  /**
   * 生命周期函数--监听页面初次渲染完成
   */
  onReady: function () {

  },

  /**
   * 生命周期函数--监听页面显示
   */
  onShow: function () {

  },

  /**
   * 生命周期函数--监听页面隐藏
   */
  onHide: function () {

  },

  /**
   * 生命周期函数--监听页面卸载
   */
  onUnload: function () {

  },

  /**
   * 页面相关事件处理函数--监听用户下拉动作
   */
  onPullDownRefresh: function () {

  },

  /**
   * 页面上拉触底事件的处理函数
   */
  onReachBottom: function () {

  },

  /**
   * 用户点击右上角分享
   */
  onShareAppMessage: function () {

  }
})
```

### 二、Promise封装：
#### util/http-p.js
```javascript
import {
  config
} from '../config.js'

const tips = {
  1: '抱歉，出现了一个错误',
  1007: 'url路径错误',
  1005: '不正确的开发者key',
  3000: '该期内容不存在'
}

class HTTP {
  request({url, data={}, method='GET'}){
    return new Promise((resolve, reject)=>{
      this._request(url, resolve, reject, data, method)
    })
  }

  _request(url, resolve, reject, data={}, method='GET') {
    wx.request({
      url: config.api_base_url + url,
      method: method,
      data: data,
      header: {
        'content-type': 'application/json',
        'appkey': config.appkey
      },
      success: (res) => {
        const code = res.statusCode.toString()
        if (code.startsWith('2')) {
          resolve(res.data)
        } else {
          reject()
          const errorCode = res.data.error_code
          this._showError(errorCode)
        }
      },
      fail: (err) => {
        reject()
        this._showError(1)
      }
    })
  }

  _showError(errorCode) {
    if (!errorCode) {
      errorCode = 1
    }
    const tip = tips[errorCode]
    wx.showToast({
      title: tip ? tip : tips[1],
      icon: 'none',
      duration: 2000
    })
  }
}

export {
  HTTP
}
```

#### models/book.js
```javascript
import { HTTP } from '../util/http-p.js'

class BookModel extends HTTP {
	getHotList () {
		return this.request({
			url: '/book/hot_list',
		})
	}

	getMyBookCount () {
		return this.request({
			url: '/book/favor/count',
		})
	}
}

export {
	BookModel
}
```

#### book/book.js
```javascript
import {BookModel} from '../../models/book.js'
const bookModel = new BookModel()

Page({

  /**
   * 页面的初始数据
   */
  data: {

  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    bookModel.getHotList().then(res =>{
      console.log(res)
    })
  },
  })
```


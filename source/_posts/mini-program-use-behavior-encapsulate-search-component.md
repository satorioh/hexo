---
title: 【微信小程序】使用behavior封装search组件
date: 2019-04-11 10:34:33
categories: 小程序
tags:
- 微信小程序
- 前端
permalink: mini-program-use-behavior-encapsulate-search-component
---
behaviors 是用于组件间代码共享的特性，类似于一些编程语言中的“mixins”或“traits”。
每个 behavior 可以包含一组属性、数据、生命周期函数和方法，组件引用它时，它的属性、数据和方法会被合并到组件中，生命周期函数也会在对应时机被调用。每个组件可以引用多个 behavior 。 behavior 也可以引用其他 behavior<!--more-->

##### pagination.js
```javascript
const paginationBev = Behavior({
	data:{
		dataArray:[],
		total:null,
		noneResult:false,
		loading:false
	},

	methods:{
		setMoreData(dataArray){
			const tempArray = this.data.dataArray.concat(dataArray)
			this.setData({
				dataArray:tempArray
			})
		},

		getCurrentStart(){
			return this.data.dataArray.length
		},

		setTotal(total){
			this.data.total = total
			if (total == 0){
				this.setData({
					noneResult:true
				})
			}
		},

		hasMore(){
			if(this.data.dataArray.length >= this.data.total){
				return false
			} else{
				return true
			}
		},

		initialize(){
			// this.data.dataArray = []
			this.setData({
				dataArray:[],
				noneResult:false,
				loading:false
			})
			this.data.total = null
		},

		isLocked(){
			return this.data.loading
		},

		locked(){
			// this.data.loading = true
			this.setData({
				loading:true
			})
		},

		unLocked(){
			// this.data.loading = false
			this.setData({
				loading:false
			})
		},
	}
})

export {paginationBev}
```

##### searchCmp.js
```javascript
// components/search/searchCmp.js
import { KeywordModel } from "../../models/keyword";
import { BookModel } from "../../models/book";
import { paginationBev } from "../behaviors/pagination";

const keywordModel = new KeywordModel()
const bookModel = new BookModel()

Component({
  behaviors:[paginationBev],
  /**
   * 组件的属性列表
   */
  properties: {
    more:{
      type:String,
      observer:'loadMore'
    }
  },

  /**
   * 组件的初始数据
   */
  data: {
    historyWords:[],
    hotWords:[],
    searching:false,
    q:'',
    loadingCenter:false,
  },

  attached(){
    this.setData({
      historyWords: keywordModel.getHistory()
    })

    keywordModel.getHot().then(res => {
      this.setData({
        hotWords: res.hot
      })
    })
  },

  /**
   * 组件的方法列表
   */
  methods: {
    onCancel(e){
      this.initialize()
      this.triggerEvent('cancel',{},{})
    },

    onConfirm(e){
      this._showResult()
      this._showLoadingCenter()
      // this.initialize()
      const q = e.detail.value || e.detail.text
      this.setData({
        q
      })
      bookModel.search(0, q).then(res =>{
          this.setMoreData(res.books)
          this.setTotal(res.total)
          keywordModel.addToHistory(q)
          this._hideLoadingCenter()
        })
    },

    onDelete(e){
      this.initialize()
      this._closeResult()
    },

    loadMore(){
      if(!this.data.q){
        return
      }
      if(this.isLocked()){
        return
      }
      console.log('load more')
      // const length = this.data.dataArray.length
      if(this.hasMore()){
        // this.data.loading = true
        this.locked()
        bookModel.search(this.getCurrentStart(), this.data.q).then(res =>{
          this.setMoreData(res.books)
          // this.data.loading = false
          this.unLocked()
        }, ()=>{
          this.unLocked()
        })
      }
    },

    _showResult(){
      this.setData({
        searching: true
      })
    },

    _closeResult(){
      this.setData({
        searching: false,
        q:''
      })
    },

    _showLoadingCenter(){
      this.setData({
        loadingCenter:true
      })
    },

    _hideLoadingCenter(){
      this.setData({
        loadingCenter:false
      })
    },
  }
})

```

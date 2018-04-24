---
title: 【Angular】服务归纳
date: 2018-04-24 14:46:28
categories: Angular
tags:
- angular
- 前端
permalink: angular-service-summary
---
### 一、基本配置
#### 1.创建服务并在根模块注册
`ng g service stock-info`
`providers: [StockInfoService]`

#### 2.配置服务类逻辑
```typescript
import { Injectable } from '@angular/core';

@Injectable()
export class StockInfoService {

  constructor() { }
  getStock(){
    return new Stock(1,'IBM');
  }
}

export class Stock{
  constructor(public id:number,public name:string){}
}
```

#### 3.在组件中注入服务
```typescript
export class AppComponent implements OnInit {
  title = 'app';
  private stock:Stock;

  constructor(private stockInfo:StockInfoService) {//依赖注入
  }
  ngOnInit(){
    this.stock = this.stockInfo.getStock();
  }

}
```
注意：
- 服务是否可注入，取决于其是否在根模块的providers中声明
- 而@Injectable修饰符表示其他的组件或服务，可以注入到此服务
- 一般优先在根模块中声明，这样对模块包含的所有组件都可见；在组件中声明的话，则只对该组件及其子组件可见

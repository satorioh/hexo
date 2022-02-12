---
title: 【Angular】HTTP通信
date: 2018-05-07 09:49:51
categories: Angular
tags:
- angular
- 前端
permalink: angular-http
---
#### 一、在根模块导入HttpModule
```typescript
import {HttpModule } from "@angular/http";
imports: [
    HttpModule
  ],
```
<!--more-->

#### 二、在组件中导入Http服务
```typescript
import { Http } from '@angular/http';
import "rxjs/Rx";
import {Observable} from "rxjs";

export class HttpComponent implements OnInit {

  dataSource:Observable<any>;
  stocks = [];
  constructor(public http:Http) {
    this.dataSource =  this.http.get('/stock').map(response => response.json());
   }

  ngOnInit() {
   this.dataSource.subscribe(data => this.stocks = data);
  }

}
```

#### 三、使用proxy实现跨域调用API，方便开发
##### 1.在客户端新增proxy.conf.json
```json
{
    "/api":{
        "target":"http://localhost:8000"
    }
}
```

##### 2.修改http组件中的url，前面增加api
```typescript
constructor(public http:Http) {
    this.dataSource =  this.http.get('/api/stock').map(response => response.json());
   }
```

##### 3.server端修改响应url，前面增加api
```typescript
import * as express from "express";

const app = express();

app.get('/',(request,response) => response.send("这里是首页!"));
app.get('/api/stock',(request,response) => response.json(stocks));
app.get('/api/stock/:id',(request,response) => response.json(stocks.find((stock) => stock.id == request.params.id )));
const server = app.listen(8000, 'localhost', ()=>{
    console.log("服务器已启动");
});

export class Stock {
    constructor(
        public id:number,
        public name:string,
        public price:number,
        public rating:number,
        public desc:string,
        public categories:Array<string>
    ){}
}

const stocks: Stock[] = [
    new Stock(1, '第一只股票', 1.99, 3.5, '第1只股票，大牛股', ['IT', '互联网']),
    new Stock(2, '第二只股票', 2.99, 4.5, '第2只股票，大牛股', ['IT', '金融']),
    new Stock(3, '第三只股票', 3.99, 3.5, '第3只股票，大牛股', ['金融', 'IT']),
    new Stock(4, '第四只股票', 4.99, 1.5, '第4只股票，大牛股', ['金融', '互联网']),
    new Stock(5, '第五只股票', 5.99, 2.5, '第5只股票，大牛股', ['IT']),
    new Stock(6, '第六只股票', 6.99, 3.5, '第6只股票，大牛股', ['IT', '互联网']),
    new Stock(7, '第七只股票', 7.99, 2.5, '第7只股票，大牛股', ['互联网']),
    new Stock(8, '第八只股票', 8.99, 1.5, '第8只股票，大牛股', ['金融']),
];
```

##### 4.在package.json的启动脚本start中添加proxy配置
```json
"scripts": {
    "ng": "ng",
    "start": "ng serve --proxy-config proxy.conf.json",
    "build": "ng build --prod",
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e"
  },
```

#### 四、使用async管道接收并展示数据
##### 1.模板修改为：
```html
<p>
  股票信息
</p>
<ul>
  <li *ngFor="let stock of stocks | async">
      {{stock.name}}
  </li>
</ul>
```

##### 2.组件类修改为：
```typescript
export class HttpComponent implements OnInit {

  stocks:Observable<any>;
  
  constructor(public http:Http) {
    this.stocks =  this.http.get('/api/stock').map(response => response.json());
   }

  ngOnInit() {}
}
```

#### 五、增加Request Header内容
```typescript
import { Http , Headers} from '@angular/http';

export class HttpComponent implements OnInit {

  stocks:Observable<any>;
  
  constructor(public http:Http) {
    let myHeaders:Headers = new Headers();
    myHeaders.append("Authorization","Basic 123456");
    this.stocks =  this.http.get('/api/stock',{headers:myHeaders}).map(response => response.json());
   }

  ngOnInit() {}
}
```
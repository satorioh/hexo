---
title: 【Angular】响应式编程与自定义管道
date: 2018-04-25 17:18:08
categories: Angular
tags:
- angular
- 前端
permalink: angular-rxjs-pipe
---
### 实现功能：根据用户输入内容，过滤显示匹配的信息

#### 1.创建自定义pipe
`ng g pipe stock/stockFilter`
<!--more-->
stock-filter.pipe.ts
```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'stockFilter'//使用时的名称
})
export class StockFilterPipe implements PipeTransform {

  transform(list: any[], field: string, keyword: string): any {
    if(!field || !keyword){
      return list;
    }
    return list.filter(item =>{
      let itemValue = item[field].toLowerCase();
      return itemValue.indexOf(keyword) >= 0;
    })
  }
}
```

#### 2.根模块中导入表单、响应式、pipe组件
app.module.ts
```typescript
import {FormsModule, ReactiveFormsModule } from '@angular/forms';
import { StockFilterPipe } from './stock/stock-filter.pipe';

declarations: [StockFilterPipe],
imports:[ FormsModule,ReactiveFormsModule]
```

#### 3.在模板中添加FormControl标签、应用管道
stock-manager.component.html

```html
<input [formControl]="nameFilter" type="text" name="table_search" class="form-control pull-right" placeholder="股票名称">

<tr *ngFor="let stock of stocks | stockFilter: 'name' : keyword ; let i = index">
        <td>{{i +1}}</td>
        <td>{{stock.name}}</td>
        <td>{{stock.price}}</td>
        <td><app-star [rating]="stock.rating"></app-star></td>
        <td>
          <a class="btn btn-warning btn-xs" (click)="update(stock)"><span class="glyphicon glyphicon-pencil"></span>修改</a>
          <a class="btn btn-danger btn-xs"><span class="glyphicon glyphicon-remove"></span>删除</a>
        </td>
      </tr>
```

#### 4.组件类中使用rxjs监控用户输入值的变化
stock-manager.component.ts
```typescript
import { FormControl } from '@angular/forms';
import 'rxjs/Rx';

export class StockManageComponent implements OnInit {
  private stocks: Array<Stock>;
  private nameFilter:FormControl = new FormControl();
  private keyword:string;

  constructor(private router: Router, private stockService: StockService) {
  }

  ngOnInit() {
    this.stocks = this.stockService.getStocks();
    this.nameFilter.valueChanges
    .debounceTime(500)
    .subscribe(value => this.keyword =value);
  }

  create() {
    this.router.navigateByUrl('/stock/0');
  }

  update(stock) {
    this.router.navigateByUrl('/stock/' + stock.id);
  }

}
```
---
title: 【Angular】分页组件
date: 2018-05-09 09:17:54
categories: Angular
tags:
- angular
- 前端
permalink: angular-pagination-component
---
#### 一、流程说明
- record.service负责从后端server获取数据
- 父组件table.component通过依赖注入，从record.service订阅实时变更的数据
- 子组件pagination.component通过@Input，获取父组件传入的分页相关变量值，从而在模板中显示正确的分页信息
- 然后根据用户点击的page值，子组件通过@Output属性，将page值传回父组件
- 父组件依据page值将数组数据分片展示
<!--more-->

#### 二、相关代码
##### 1.record.service.ts
```typescript
import {EventEmitter, Injectable} from '@angular/core';

@Injectable()
export class RecordService {
  records = [];
  recordChange: EventEmitter<any> = new EventEmitter();

  constructor() {
  }

  getRecord() {
    return this.records;
  }

  setRecord(arr) {
    this.records = arr;
    this.recordChange.emit(this.records);
  }
}
```

##### 2.pagination.component.html
```html
<div class="row">
  <div class="col-sm-5">
    <div style="padding-top: 8px;">
      <span>第 {{ getMin() }} 到 {{ getMax() }} 条，</span>
      <span>共 {{ total }} 条({{ totalPages() }}页)</span>
    </div>
  </div>
  <div class="col-sm-7">
    <ul class="pagination pagination-sm no-margin pull-right" *ngIf="total > 0">
      <li class="paginate_button previous" [class.disabled]="page === 1"><a href="javascript:void(0);" (click)="onPrev()">上一页</a></li>
      <li class="paginate_button" *ngFor="let pageNum of getPages()" (click)="onPage(pageNum)" [class.active]="pageNum === page"><a href="javascript:void(0);">{{pageNum}}</a></li>
      <li class="paginate_button next" [class.disabled]="lastPage()"><a href="javascript:void(0);" (click)="onNext()">下一页</a></li>
    </ul>
  </div>
</div>
```

##### 3.pagination.component.ts
```typescript
import {Component, EventEmitter, Input, OnInit, Output} from '@angular/core';

@Component({
  selector: 'app-pagination',
  templateUrl: './pagination.component.html',
  styleUrls: ['./pagination.component.css']
})
export class PaginationComponent implements OnInit {
  @Input() page: number; // 传入用户点击的页数
  @Input() total: number; // 数据总个数
  @Input() perPage: number; // 每页展示的数据个数
  @Input() pagesToShow: number; // 需要显示的分页个数

  @Output() goPrev = new EventEmitter();
  @Output() goNext = new EventEmitter();
  @Output() goPage = new EventEmitter<number>();

  constructor() {}

  ngOnInit() {}

  // 当前页的最小编号
  getMin(): number {
    return ((this.perPage * this.page) - this.perPage) + 1;
  }

  // 当前页的最大编号
  getMax(): number {
    let max = this.perPage * this.page;
    if (max > this.total) {
      max = this.total;
    }
    return max;
  }

  onPage(n: number): void {
    this.goPage.emit(n);
  }

  onPrev(): void {
    this.goPrev.emit();
  }

  onNext(): void {
    this.goNext.emit();
  }

  totalPages(): number {
    return Math.ceil(this.total / this.perPage) || 0;
  }

  lastPage(): boolean {
    return this.perPage * this.page >= this.total;
  }

  getPages(): number[] {
    const c = Math.ceil(this.total / this.perPage);
    const p = this.page || 1;
    const pagesToShow = this.pagesToShow || 3;
    const pages: number[] = [];
    pages.push(p);
    const times = pagesToShow - 1;
    for (let i = 0; i < times; i++) {
      if (pages.length < pagesToShow) {
        if (Math.min.apply(null, pages) > 1) {
          pages.push(Math.min.apply(null, pages) - 1);
        }
      }
      if (pages.length < pagesToShow) {
        if (Math.max.apply(null, pages) < c) {
          pages.push(Math.max.apply(null, pages) + 1);
        }
      }
    }
    pages.sort((a, b) => a - b);
    return pages;
  }
}
```

##### 4.table.component.html
```html
<div class="box" *ngIf="records.length > 0">
  <!-- /.box-header -->
  <div class="box-body">
    <table class="table table-bordered">
      <tr>
        <th style="width: 10px">#</th>
        <th>工号</th>
        <th>姓名</th>
        <th>日期</th>
        <th>上班时间</th>
        <th>上班地点</th>
        <th>下班时间</th>
        <th>下班地点</th>
        <th>出勤时数</th>
      </tr>
      <tr *ngFor="let record of recordsGroup;let i = index;">
        <td>{{(i+1)+(page-1)*limit}}</td>
        <td>{{record.cwid}}</td>
        <td>{{record.ccname}}</td>
        <td>{{record.cdate}}</td>
        <td>{{record.cintime}}</td>
        <td>{{record.cinpos}}</td>
        <td>{{record.cofftime}}</td>
        <td>{{record.coffpos}}</td>
        <td>{{record.chour}}</td>
      </tr>
    </table>
  </div>
  <!-- /.box-body -->
  <div class="box-footer clearfix">
    <app-pagination (goPage)="goToPage($event)"
                    (goNext)="onNext()"
                    (goPrev)="onPrev()"
                    [pagesToShow]="3"
                    [page]="page"
                    [perPage]="limit"
                    [total]="total"></app-pagination>
  </div>
</div>
<span style="color: red;">{{noRecodesInfo}}</span>
```

##### 5.table.component.ts
```typescript
import {Component, OnInit} from '@angular/core';
import {RecordService} from '../services/record.service';

@Component({
  selector: 'app-table',
  templateUrl: './table.component.html',
  styleUrls: ['./table.component.css']
})
export class TableComponent implements OnInit {

  records = [];
  recordsGroup = [];
  hasRelatedRecords = false;
  noRecodesInfo: string;

  // pagination input varible
  total = 0;
  page = 1;
  limit = 10;

  constructor(public recordService: RecordService) {
  }

  ngOnInit() {
    this.recordService.recordChange.subscribe(data => {
      this.records = data;
      if (this.records.length > 0) {
        this.noRecodesInfo = '';
        this.total = this.records.length;
        this.recordsGroup = this.records.slice(0, this.limit);
      } else {
        this.noRecodesInfo = '查无相关记录！';
      }
    });
  }

  getRecordsGroup() {
    this.recordsGroup = this.records.slice((this.page - 1) * this.limit, this.page * this.limit);
  }

  goToPage(n: number): void {
    this.page = n;
    this.getRecordsGroup();
  }

  onNext(): void {
    if (this.page === Math.ceil(this.total / this.limit)) {
      return;
    }
    this.page++;
    this.getRecordsGroup();
  }

  onPrev(): void {
    if (this.page === 1) {
      return;
    }
    this.page--;
    this.getRecordsGroup();
  }

}
```

参考链接：
[Create a pagination component in Angular 4](http://www.bentedder.com/create-a-pagination-component-in-angular-4/)
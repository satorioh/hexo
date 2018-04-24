---
title: 【Angular】路由传参
date: 2018-04-23 11:27:29
categories: Angular
tags:
- angular
- 前端
permalink: angular-routing-params
---
### 一、path传参
#### 1.路由配置（app-routing.module.ts）
```typescript
import {NgModule} from '@angular/core';
import {Routes, RouterModule} from '@angular/router';
import {AppComponent} from './app.component';
import {LoginComponent} from './login/login.component';
import {UserComponent} from './user/user.component';

const routes: Routes = [
  {path: '', component: LoginComponent},
  {path: 'login', component: LoginComponent},
  {path: 'user/:id', component: UserComponent} //id段传参
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
```
<!--more-->
#### 2.跳转链接配置(app.component.html)
```html
<a [routerLink]="['/login']">登陆页</a>
<a [routerLink]="['/user',1]">用户中心</a>
<router-outlet></router-outlet>
```
或使用指令跳转配置
```html
<a (click)="toUser()">用户中心</a>
```
```typescript
export class AppComponent {
  title = 'app';
  constructor(private router:Router){}
  toUser(){
    this.router.navigate(['/user',1]);//or this.router.navigateByUrl(['/user/1'])
  }
}
```

#### 3.目的地组件类接收参数（user.component.ts）
```typescript
export class UserComponent implements OnInit {
  id: string;

  constructor(private activatedRoute: ActivatedRoute) {
  }//实例化

  ngOnInit() {
    this.id = this.activatedRoute.snapshot.params['id'];
    console.log(this.id);
  }

}
```

### 二、query传参
#### 1.跳转设置queryParams参数（user.component.html）
```html
<a [routerLink]="['/user']" [queryParams]="{id:1,name:'user1'}">用户1</a>
```
或者
```typescript
this._router.nagivate(['/user'],{queryParams:{{id:1,name:'user1'}}});
this._router.navigateByUrl('/user?id=1&name=user1')
```
#### 2.目的地组件类接收参数（使用参数订阅）
```typescript
export class UserComponent implements OnInit {
  id: any;
  constructor(private activatedRoute:ActivatedRoute) {
  }

  ngOnInit() {
    this.activatedRoute.queryParams.subscribe(params  => {
      this.id = params['id'];
      console.log(params);
    });
  }
}
```

### 三、路由配置中data传参
#### 1.路由配置（app-routing.module.ts）
```typescript
{path: 'user', component: UserComponent, data: [{id: 1}]}
```
#### 2.目的地组件类接收参数
```typescript
export class UserComponent implements OnInit {
  id: any;

  constructor(private activatedRoute: ActivatedRoute) {
  }

  ngOnInit() {
    this.id = this.activatedRoute.snapshot.data[0]['id'];
    console.log(this.id);
  }
}
```

### 四、使用参数订阅解决同一页面内传参
#### 1.链接跳转（user.component.html）
```html
<a [routerLink]="['/user',1]">用户1</a>
<a [routerLink]="['/user',2]">用户2</a>
```
#### 2.本页面组件类设置参数订阅（user.component.ts）
```typescript
export class UserComponent implements OnInit {
  id: any;
  constructor(private activatedRoute: ActivatedRoute) {
  }

  ngOnInit() {
    this.activatedRoute.params.subscribe(params  => {
      this.id = params['id'];
      console.log(this.id);
    });
  }
}
```

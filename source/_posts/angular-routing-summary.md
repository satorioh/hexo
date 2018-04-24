---
title: 【Angular】路由归纳
date: 2018-04-24 08:55:27
categories: Angular
tags:
- angular
- 前端
permalink: angular-routing-summary
---
### 一、概念

| 名词            | 解释                                                                                |
|:---------------|:-----------------------------------------------------------------------------------|
| Routes         | 路由配置，保存着哪个URL对应展示哪个组件，以及在哪个RouterOutlet中展示组件                     |
| RouterOutlet   | 在HTML中标记路由内容呈现位置的占位符                                                     |
| Router         | 负责在运行时执行路由的对象，可以通过调用其navigate()和navigateByUrl()方法来导航到一个指定的路由 |
| RouterLink     | 在HTML中声明路由导航用的指令                                                           |
| ActivatedRoute | 当前激活的路由对象，保存着当前路由的信息，如路由地址、路由参数等                                       |
<!--more-->
### 二、路由基本配置
#### 1.设置路由配置文件
##### （1）创建项目时生成：
`ng new xxx --routing`会自动生成app-routing.module.ts文件
##### （2）手动添加：可将路由信息手动添加到app.module.ts根模块中，使之生效

#### 2.添加路由策略
以添加到根模块为例：
```typescript
import {RouterModule, Routes} from '@angular/router';

const routes: Routes = [
  {path: '', redirectTo: '/dashboard', pathMatch: 'full'},
  {path: 'dashboard', component: DashboardComponent},
  {path: 'stock', component: StockManageComponent}
];

imports: [
    BrowserModule,
    RouterModule.forRoot(routes)
  ],
```

#### 3.在模板中使用路由跳转
```typescript
<a [routerLink]="['/dashboard']">首页</a>
<a [routerLink]="['/stock']">股票管理</a>
<router-outlet></router-outlet>
```

### 三、路由传参
参考 [此篇文章](https://roubin.me/angular-routing-params/)

### 四、重定向路由、默认路由、404路由
```typescript
const routes: Routes = [
  {path: '', redirectTo: '/dashboard', pathMatch: 'full'},
  {path: 'dashboard', component: DashboardComponent},
  {path: 'stock', component: StockManageComponent},
  {path: '**', component: Code404Component}
];
```

### 五、子路由
#### 1.设置子路由策略
```typescript
{path: 'user', component: UserComponent,
  children:[
    {path: '', component: UserInfoComponent},
    {path: 'usermanager', component: UserManagerComponent}
  ]
}
```

#### 2.在父组件模板中使用子路由跳转(user.component.html)
```html
<a [routerLink]="['./usermanager']">用户管理</a>
<router-outlet></router-outlet>
```

### 六、辅助（附属）路由
#### 1.添加路由信息
```typescript
{path: '', component: UserInfoComponent},
{path: 'usermanager', component: UserManagerComponent, outlet:'aux'}
```
#### 2.在模板中使用
```html
<a  [routerLink]="[{outlets:{aux:'usermanager'}}]">用户管理</a>
<router-outlet></router-outlet>
<router-outlet name="aux"></router-outlet>
```

### 七、路由守卫（拦截）
#### 1.CanActivate:处理导航到某路由的情况
##### （1）生成CanActivateGuard服务，并导入根模块的providers中
`ng g service can-activate-guard`
`providers: [CanActivateGuardComponent]`

##### (2)配置CanActivateGuard服务类逻辑
```typescript
@Injectable()
export class CanActivateGuardComponent implements CanActivate {
  canActivate(){
    let hasPermission:boolean = Math.random() < 0.5;
    if(!hasPermission){
      console.log("您无权访问用户中心");//false
    }
    return hasPermission;//true
  }
}
```
##### (3)应用到相关路由上
```typescript
{path: 'user', component: UserComponent,canActivate:[CanActivateGuardComponent]}
```

#### 2.CanDeactivate:处理从当前路由离开的情况
##### (1）生成CanDeactivateGuard服务，并导入根模块的providers中
`ng g service can-deactivate-guard`
`providers: [CanDeactivateGuardService]`

##### (2)配置CanDeactivateGuard服务类逻辑
```typescript
@Injectable()
export class CanDeactivateGuardService implements CanDeactivate<any> {
  canDeactivate(component: any){
    if(component.isModified()){//调用类实例的方法
      return true;
    } else {
      return false;
    }
  }
}
```

##### (3)应用到相关路由上
```typescript
 {path: 'user', component: UserComponent,
  canActivate:[CanActivateGuardComponent],
  canDeactivate:[CanDeactivateGuardService]}
```

#### 3.Resolve:确认数据获取成功后，再激活路由，进入组件
##### (1）生成resolve服务，并导入根模块的providers中
##### (2) 配置服务类的逻辑
```typescript
import { Resolve, ActivatedRouteSnapshot, RouterStateSnapshot, Router } from "@angular/router";
import { Data } from "../cn/cn.component";
import { Injectable } from "@angular/core";

@Injectable()
export class DataResolve implements Resolve<Data>{
    constructor(private router: Router){}

    resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot) {
        let id = route.params['id'];
        if(id == 1) {
            return new Data(1, 'IBM');
        } else {
            this.router.navigate(['/home']);
            return undefined;
        }
    }
}
```
##### （3）配置路由应用resolve
```typescript
{  
    path: 'home', 
    component: HomeComponent,
    resolve: {
      data: DataResolve
    }  
  },
```
##### (4)目标组件类获取数据
```typescript
export class HomeComponent implements OnInit {
  private info: any;

  constructor(private routeInfo: ActivatedRoute) { }

  ngOnInit() {
    this.routInfo.data.subscribe((data: {info: Data}) => {
      this.info = data.info;
    });
  }
}
```

### 八、根据不同路由，显示不同标题
```typescript
import 'rxjs/add/operator/filter';

export class ContentComponent implements OnInit {
  pageTitle = '';
  pageDesc = '';
  constructor(public router:Router) {
    
  }

  ngOnInit() {
    this.router.events.filter(event => event instanceof NavigationEnd)
                          .subscribe((event:NavigationEnd)=>{
                            if(event.url == '/dashboard'){
                              this.pageTitle = '这里是首页';
                              this.pageDesc = '天天赚钱';
                            }else if(event.url.startsWith('/stock')){
                              this.pageTitle = '股票信息管理';
                              this.pageDesc = '各方信息汇总';
                            }
                          })
  }

}
```

参考链接：
[Angular2 路由](http://www.xinxiaoyang.com/programming/2017-11-30-angular-router/)

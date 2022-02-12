---
title: 【Angular】响应式表单的使用
date: 2018-05-04 09:25:25
categories: Angular
tags:
- angular
- 前端
permalink: angular-reactive-forms
---
#### 一、在根模块中引入依赖
```typescript
import {FormsModule, ReactiveFormsModule } from '@angular/forms';

imports: [
    FormsModule,
    ReactiveFormsModule
  ],
```
<!--more-->
#### 二、在组件类中声明formModel
```typescript
formModel:FormGroup;
```

#### 三、在组件模板form处添加属性，使用ng接管
```html
<form [formGroup]="formModel" class="form-horizontal"></form>
```

#### 四、在组件类中构建数据表单模型
```typescript
export class StockFormComponent implements OnInit {
  formModel:FormGroup;
  stockId:number;
  stock:Stock;
  categories = ["IT","互联网","金融"];
  constructor(
    private activatedRoute: ActivatedRoute, 
    private stockService:StockService,
    private router:Router
  ) { }

  ngOnInit() {
    this.stockId = this.activatedRoute.snapshot.params['id'];
    this.stock = this.stockService.getStock(this.stockId);

    let fb = new FormBuilder;
    this.formModel = fb.group({
      name:[this.stock.name, [Validators.required, Validators.minLength(3)]],
      price:[this.stock.price, [Validators.required]],
      desc:[this.stock.desc],
      categories:fb.array([
        new FormControl(this.stock.categories.indexOf(this.categories[0])!== -1),
        new FormControl(this.stock.categories.indexOf(this.categories[1])!== -1),
        new FormControl(this.stock.categories.indexOf(this.categories[2])!== -1)
      ])
    });
  }

  cancel(){
    this.router.navigateByUrl('/stock');
  }

  save(){
      const chineseCategories = [];
          for(let i = 0; i< 3;i++){
            if(this.formModel.value.categories[i]){
              chineseCategories[i] = this.categories[i];
            }
          }
          this.formModel.value.categories = chineseCategories;
          this.formModel.value.rating = this.stock.rating;
          console.log(this.formModel.value);
          //this.router.navigateByUrl('/stock');
  }
}
```

#### 五、在前台模板中绑定对应的数据模型
```html
<form [formGroup]="formModel" class="form-horizontal">
    <div class="box-body">
      <div class="form-group">
        <label for="name" class="col-sm-2 control-label">股票名称</label>

        <div class="col-sm-10">
          <input formControlName="name" type="email" class="form-control" id="name" placeholder="股票名称">
        </div>
      </div>
      <div class="form-group">
        <label for="price" class="col-sm-2 control-label">股票价格</label>

        <div class="col-sm-10">
          <input formControlName="price" type="text" class="form-control" id="price" placeholder="股票价格">
        </div>
      </div>
      <div class="form-group">
        <label class="col-sm-2 control-label">股票评级</label>

        <div class="col-sm-10">
          <app-star [(rating)]="stock.rating"></app-star>
        </div>
      </div>

      <div class="form-group">
        <label for="desc" class="col-sm-2 control-label">股票描述</label>

        <div class="col-sm-10">
          <textarea formControlName="desc" name="" class="form-control" id="desc" rows="5"></textarea>
        </div>
      </div>

      <div class="form-group">
        <label class="col-sm-2 control-label">股票类型</label>
        <div class="col-sm-10">
          <div class="row" formArrayName="categories">
            <div class="col-sm-2" *ngFor="let category of categories;let i = index">
                <div class="checkbox">
                    <label>
                      <input [formControlName]="i" type="checkbox">{{category}}
                    </label>
                  </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- /.box-body -->
    <div class="box-footer">
      <button (click)="cancel()" type="submit" class="btn btn-default">取消</button>
      <button (click)="save()" type="submit" class="btn btn-info pull-right">保存</button>
    </div>
    <!-- /.box-footer -->
  </form>
```

#### 六、模板绑定校验器，并给出错误提示信息
```html
<div class="form-group" [class.has-error]="formModel.hasError('required','name') || formModel.hasError('minlength','name')">
        <label for="name" class="col-sm-2 control-label">股票名称</label>

        <div class="col-sm-10">
          <input formControlName="name" type="email" class="form-control" id="name" placeholder="股票名称">
          <span class="help-block" [class.hidden]="!formModel.hasError('required','name')">股票名称是必填项</span>
        <span class="help-block" [class.hidden]="!formModel.hasError('minlength','name')">请至少输入三个字</span>
        </div>
      </div>
      <div class="form-group" [class.has-error]="formModel.hasError('required','price')">
        <label for="price" class="col-sm-2 control-label">股票价格</label>

        <div class="col-sm-10">
          <input formControlName="price" type="number" class="form-control" id="price" placeholder="股票价格">
          <span class="help-block" [class.hidden]="!formModel.hasError('required','price')">股票价格是必填项</span>
        </div>
      </div>
```

#### 七、自定义校验器
```typescript
categories:fb.array([
        new FormControl(this.stock.categories.indexOf(this.categories[0])!== -1),
        new FormControl(this.stock.categories.indexOf(this.categories[1])!== -1),
        new FormControl(this.stock.categories.indexOf(this.categories[2])!== -1)
      ], this.categoriesSelectValidator)
      
      categoriesSelectValidator(control:FormArray){
            let valid:boolean = !control.controls.every(function(item){
              return item.value === false;
            })
            if(valid){
              return null;
            }else{
              return {categoriesLength:true}
            }
        }
```

绑定前台模板
```html
<div class="form-group" [class.has-error]="formModel.hasError('categoriesLength','categories')">
        <label class="col-sm-2 control-label">股票类型</label>
        <div class="col-sm-10">
          <div class="row" formArrayName="categories">
            <div class="col-sm-2" *ngFor="let category of categories;let i = index">
                <div class="checkbox">
                    <label>
                      <input [formControlName]="i" type="checkbox">{{category}}
                    </label>
                  </div>
            </div>
            <span class="help-block" [class.hidden]="!formModel.hasError('categoriesLength','categories')">请至少选择一个类型</span>
          </div>
        </div>
      </div>
```

#### 八、细节完善
##### 1.当表单中存在错误时，保存按钮不能点击
```html
<button (click)="save()" type="submit" class="btn btn-info pull-right" [disabled]="formModel.invalid">保存</button>
```

##### 2.表单初始时，根据状态字段实现不立即报错
```html
<div class="form-group" [class.has-error]="formModel.get('name').touched && (formModel.hasError('required','name') || formModel.hasError('minlength','name'))">
        <label for="name" class="col-sm-2 control-label">股票名称</label>

        <div class="col-sm-10">
          <input formControlName="name" type="email" class="form-control" id="name" placeholder="股票名称">
          <span class="help-block" [class.hidden]="formModel.get('name').untouched || !formModel.hasError('required','name')">股票名称是必填项</span>
        <span class="help-block" [class.hidden]="!formModel.hasError('minlength','name')">请至少输入三个字</span>
        </div>
      </div>
```

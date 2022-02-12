---
title: Typescript 泛型
date: 2021-03-05 11:03:17
categories: Typescript
tags:
- typescript
permalink: typescript-generic
---
#### 一、工具泛型
##### 1.Partial
作用：将一个接口的所有属性设置为可选状态
```
type Partial<T> = {
    [P in keyof T]?: T[P]
}
```
<!--more-->
自定义DeepPartial
```
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends Object ? DeepPartial<T[P]> : T[P];
}
```

##### 2.Required
作用：将接口中所有可选的属性改为必须的
```
type Required<T> = {
    [P in keyof T]-?: T[P]
}
```

##### 3.Record
作用：以K中类型为键，每个都返回T类型
```
type Record<K extends keyof any, T> = {
    [P in K]: T
}
```
例子：
```
type petsGroup = 'dog' | 'cat' | 'fish';
interface IPetInfo {
    name:string,
    age:number,
}

type IPets = Record<petsGroup, IPetInfo>;

const animalsInfo:IPets = {
    dog:{
        name:'dogName',
        age:2
    },
    cat:{
        name:'catName',
        age:3
    },
    fish:{
        name:'fishName',
        age:5
    }
}
```

##### 4.Pick
作用：提取接口的某几个属性
```
type Pick<T, K extends keyof T> = {
    [P in K]: T[P]
}
```
例子：
```
interface Todo {
  title: string
  completed: boolean
  description: string
}

type TodoPreview = Pick<Todo, "title" | "completed">

const todo: TodoPreview = {
  title: 'Clean room',
  completed: false
}
```

##### 5.Exclude
作用：取出T在U中不存在的属性
```
type Exclude<T, U> = T extends U ? never : T
```
例子：
```
interface Worker {
  name: string
  age: number
  email: string
  salary: number
}

interface Student {
  name: string
  age: number
  email: string
  grade: number
}


type ExcludeKeys = Exclude<keyof Worker, keyof Student>
// 'salary'
```

##### 6.Omit
作用：忽略T中的K属性
```
type Omit<T, K extends keyof any> = Pick<
  T, Exclude<keyof T, K>
>
```
例子：
```
interface Todo {
  title: string
  completed: boolean
  description: string
}

type TodoPreview = Omit<Todo, "description">

const todo: TodoPreview = {
  title: 'Clean room',
  completed: false
}
```

##### 7.Extract
作用：取交集
```
type Extract<T, U> = T extends U ? T : never;
```
例子：
```
interface Worker {
  name: string
  age: number
  email: string
  salary: number
}

interface Student {
  name: string
  age: number
  email: string
  grade: number
}


type CommonKeys = Extract<keyof Worker, keyof Student>
// 'name' | 'age' | 'email'
```

##### 8.NonNullable
作用：从T中排除null和undefined
```
type NonNullable<T> = T extends null | undefined ? never : T;
```
例子：
```
type example = NonNullable<string | number | undefined>
// type example = string | number
```

##### 9.Readonly
作用：将所有属性设置为只读
```
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```
自定义DeepReadonly
```
type DeepReadonly<T> = {
  readonly [P in keyof T]: DeepReadonly<T[P]>;
}

const a = { foo: { bar: 22 } }
const b = a as DeepReadonly<typeof a>
b.foo.bar = 33 // Cannot assign to 'bar' because it is a read-only property.ts(2540)
```

参考：

[你需要知道的 TypeScript 高级类型](https://mp.weixin.qq.com/s/yzdeYmlkszplTPAeyixaBQ)
[TypeScript 高级特性](https://jishuin.proginn.com/p/763bfbd3969c)
[TypeScript 中提升幸福感的10 个高级技巧](https://segmentfault.com/a/1190000039030887)
[TS 一些工具泛型的使用及其实现](https://zhuanlan.zhihu.com/p/40311981)
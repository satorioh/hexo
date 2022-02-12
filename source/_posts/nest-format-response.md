---
title: Nestjs 使用拦截器、异常过滤器统一response格式
date: 2021-09-14 13:15:52
categories: Nest.js
tags:
- nestjs
permalink: nest-format-response
---
格式举例：
```javascript
// 成功返回
{
  code: 200,
  data: {
    // 详情类
    info: { 
      // 返回数据
    },

    // 列表类
    list: [],

    pagination: {
      total: 100,
      pageSize: 10,
      pages: 10,
      page: 1,
    },
  },
  message: "请求成功"
},

// 失败返回
{
  code: 400,
  message: "查询失败",
}
```
<!--more-->

##### 1.创建拦截器，统一成功返回的response格式
```shell
nest g in transform interceptor
```

##### 2.拦截器代码
```typescript
// src/interception/transform.interception.ts

import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map((data) => ({
        code: 200,
        data,
        message: 'success',
      })),
    );
  }
}
```

##### 3.创建异常过滤器，统一失败返回的response格式
```shell
nest g f httpExecption filter
```

##### 4.异常过滤器代码
```typescript
// src/filter/http-execption.filter.ts

import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExecptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const status = exception.getStatus();
    const message = exception.message;

    response.status(status).json({
      code: status,
      message,
    });
  }
}
```

##### 5.在 main.ts 中全局使用拦截器、异常过滤器
```typescript
// src/main.ts

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { HttpExceptionFilter } from './filters/http-execption.filter';
import { TransformInterceptor } from './interceptor/transform.interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalInterceptors(new TransformInterceptor())
  app.useGlobalFilters(new HttpExceptionFilter())

  await app.listen(3000);
}
bootstrap();
```

##### 6.修改service中的方法
```typescript
// src/modules/article/article.service.ts

import { Injectable, NotFoundException } from '@nestjs/common';

async getOne(idDTO: IdDTO) {
    const { id } = idDto
    const articleDetail = await this.articleRepository
      .createQueryBuilder('article')
      .where('article.id = :id', { id })
      .getOne();

    if (!articleDetail) {
      throw new NotFoundException('找不到文章');
    }

    return {
      info: articleDetail,
    };
  }
```

参考文章：

[使用拦截器、异常过滤器实现统一返回格式](https://juejin.cn/post/6992098170540916773)

[Nestjs-Exception filters](https://docs.nestjs.com/exception-filters)

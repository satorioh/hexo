---
title: Nestjs 使用class-validator实现输入验证
date: 2021-09-14 16:43:57
categories: Nest.js
tags:
- nestjs
permalink: nest-input-validation
---
##### 1.安装依赖
```shell
npm i --save class-validator class-transformer
```
<!--more-->

##### 2.启用全局验证
```typescript
// src/main.ts

import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { HttpExceptionFilter } from './filters/http-execption.filter';
import { TransformInterceptor } from './interceptor/transform.interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(new ValidationPipe())
  app.useGlobalInterceptors(new TransformInterceptor())
  app.useGlobalFilters(new HttpExceptionFilter())

  await app.listen(3000);
}
bootstrap();
```

##### 3.修改异常过滤器，返回验证的错误信息
```typescript
// src/filter/http-exception.filter.ts

import { ExceptionFilter, Catch, ArgumentsHost, HttpException, Logger } from '@nestjs/common';
import { Request, Response } from 'express';
import { execPath } from 'process';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const status = exception.getStatus();
    const message = exception.message

    const exceptionResponse: any = exception.getResponse()
    let validatorMessage = exceptionResponse
    if (typeof validatorMessage === 'object') {
      validatorMessage = exceptionResponse.message[0]
    }

    response
      .status(status)
      .json({
        code: status,
        message: validatorMessage || message,
      });
  }
}
```

##### 4.工具类增加正则
```typescript
// src/utils/index.ts
// 非 0 正整数
export const regPositive: RegExp = /^[1-9]\d*$/
```

##### 5.修改dto
```typescript
// src/modules/article/dto/id.dto.ts

import { IsNotEmpty, Matches } from "class-validator";
import { regPositive } from "src/utils/regex.util";

export class IdDTO {
    @Matches(regPositive, {message: '请输入有效 id'})
    @IsNotEmpty({message: 'id 不能为空'})
    readonly id: number
}
```

##### 6.修改controller
```typescript
@Get('detail')
async getOne(@Query() params: IdDTO) {
    return this.articleService.getOne(params);
}
```

参考文章：

[使用class-validator+类验证器实现表单验证](https://juejin.cn/post/6992098387713589284)

[Nestjs-Validation](https://docs.nestjs.com/techniques/validation)



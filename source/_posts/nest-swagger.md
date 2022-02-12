---
title: Nestjs 使用 Swagger 生成接口文档
date: 2021-09-15 13:52:43
categories: Nest.js
tags:
- nestjs
permalink: nest-swagger
---
##### 1.安装依赖
```shell
npm install --save @nestjs/swagger swagger-ui-express
```
<!--more-->

##### 2.配置swagger
```typescript
// src/main.ts

import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { HttpExceptionFilter } from './filters/http-exception.filter';
import { TransformInterceptor } from './interceptor/transform.interceptor';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(new ValidationPipe())
  app.useGlobalInterceptors(new TransformInterceptor())
  app.useGlobalFilters(new HttpExceptionFilter())

  const options = new DocumentBuilder()
    .setTitle('blog-serve')
    .setDescription('接口文档')
    .setVersion('1.0')
    .build();
  const document = SwaggerModule.createDocument(app, options);
  SwaggerModule.setup('swagger-doc', app, document);

  await app.listen(3000);
}
bootstrap();
```

##### 3.设置nest-cli.json
```json
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "plugins": [
      {
        "name": "@nestjs/swagger",
        "options": {
          "introspectComments": true
        }
      }
    ]
  }
}
```

##### 4.修改dto，添加文档说明
```typescript
import { IsNotEmpty, IsOptional } from 'class-validator';
import { IdDTO } from './id.dto';

export class ArticleEditDTO extends IdDTO {
  /**
   * 文章标题
   * @example 美丽的大海
   */
  @IsOptional()
  @IsNotEmpty({ message: '请输入文章标题' })
  readonly title?: string;

  /**
   * 文章描述
   * @example 给你讲述美丽的大海
   */
  @IsOptional()
  @IsNotEmpty({ message: '请输入文章描述' })
  readonly description?: string;

  /**
   * 文章内容
   * @example 美丽的大海，你是如此美丽
   */
  @IsOptional()
  @IsNotEmpty({ message: '请输入文章内容' })
  readonly content?: string;
}
```

##### 5.查看
```shell
localhost:3000/swagger-doc
```

参考文章：

[使用 Swagger 生成文档](https://juejin.cn/post/6992098225020911629)

[Nestjs-CLI Plugin](https://docs.nestjs.com/openapi/cli-plugin)

[Nestjs-Open API](https://docs.nestjs.com/openapi/introduction)

[Nestjs-Swagger sample](https://github.com/nestjs/nest/tree/master/sample/11-swagger)

[blog-serve](https://github.com/huihuipan/blog-serve)

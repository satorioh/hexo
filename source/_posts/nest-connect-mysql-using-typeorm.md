---
title: Nestjs 使用 TypeORM 连接 mysql
date: 2021-09-13 15:42:02
categories: Nest.js
tags:
- nestjs
- typeorm
- mysql
permalink: nest-connect-mysql-using-typeorm
---
##### 1.安装依赖
```shell
npm install --save @nestjs/typeorm typeorm mysql2
```
<!--more-->

##### 2.在app.module.ts 中引入 MySQL 的连接模块
方法：1
```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ArticleModule } from './modules/article/article.module';
import { TypeOrmModule } from '@nestjs/typeorm'

@Module({
  imports: [
    // 使用 TypeORM 配置数据库
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      entities: ["dist/modules/**/*.entity{.ts,.js}"],
      synchronize: true, //生产环境不要使用
    }),
    ArticleModule
  ],
  controllers: [AppController],
  providers: [AppService],
})

export class AppModule {}
```

方法：2
在根目录创建ormconfig.json
```json
{
  "type": "mysql",
  "host": "localhost",
  "port": 3306,
  "username": "root",
  "password": "root",
  "database": "test",
  "entities": ["dist/modules/**/*.entity{.ts,.js}"],
  "synchronize": true
}
```
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
    imports: [TypeOrmModule.forRoot()],
})
export class AppModule {}
```

##### 3.创建实体 Entity
```typescript
import { 
  Entity, 
  Column, 
  PrimaryGeneratedColumn, 
  UpdateDateColumn,
  CreateDateColumn,
  VersionColumn,
} from 'typeorm';

@Entity()
export class Article {
    // 主键id
    @PrimaryGeneratedColumn()
    id: number;
  
    // 创建时间
    @CreateDateColumn()
    createTime: Date
  
    // 更新时间
    @UpdateDateColumn()
    updateTime: Date
  
    // 软删除
    @Column({
      default: false
    })
    isDelete: boolean
  
    // 更新次数
    @VersionColumn()
    version: number
  
    // 文章标题
    @Column('text')
    title: string;

    // 文章描述
    @Column('text')
    description: string;

    // 文章内容
    @Column('text')
    content: string;
}
```

##### 4.在article.module中声明repository
```typescript
// article.module.ts
import { Module } from '@nestjs/common';
import { ArticleService } from './article.service';
import { ArticleController } from './article.controller';
import { Article } from './entity/article.entity';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forFeature([Article]),
  ],
  providers: [ArticleService],
  controllers: [ArticleController]
})

export class ArticleModule {}
```

##### 5.在article.service中注入repository
```typescript
// article.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Article } from './entity/article.entity';

@Injectable()
export class ArticleService {
    constructor(
        @InjectRepository(Article)
        private articleRepository: Repository<Article>,
    ) {}

    async findAll(): Promise<User[]> {
        return this.articleRepository.find();
    }
}
```

参考文章：

[Nestjs-Database](https://docs.nestjs.com/techniques/database#typeorm-integration)

[使用TypeORM+Mysql实现数据持久化](https://juejin.cn/post/6992097780487929870)


---
title: Nestjs 使用 mongoose 连接 mongodb
date: 2021-09-09 10:54:11
categories: Nest.js
tags:
- nestjs
- mongodb
permalink: nest-connect-mongodb-using-mongoose
---
##### 1.安装依赖
```shell
npm install mongoose @nestjs/mongoose --save
npm install @types/mongoose --save-dev
```
<!--more-->

##### 2.在app.module.ts 中引入 Mongoose 的连接模块
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UserModule } from './server/user/user.module';

@Module({
  imports: [MongooseModule.forRoot('mongodb://localhost/test'), UserModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

##### 3.创建schema，定义数据表的格式
```typescript
// user.schema.ts
import { Schema } from 'mongoose';

export const userSchema = new Schema({
  _id: { type: String, required: true },
  user_name: { type: String, required: true },
  password: { type: String, required: true },
});
```

##### 4.定义interface
```typescript
import { Document } from 'mongoose';

export interface User extends Document {
  readonly _id: string;
  readonly user_name: string;
  readonly password: string;
}
```

##### 5.修改userModule
```typescript
// user.module.ts
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import { userSchema } from './user.schema';

@Module({
  imports: [MongooseModule.forFeature([{ name: 'Users', schema: userSchema }])],
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}
```

##### 6.创建userService
```typescript
// user.service.ts
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { CreateUserDTO, EditUserDTO } from './user.dto';
import { User } from './user.interface';

@Injectable()
export class UserService {
  constructor(@InjectModel('Users') private readonly userModel: Model<User>) {}

  /** 查找所有用户 */
  async findAll(): Promise<User[]> {
    return this.userModel.find();
  }

  /** 查找单个用户 */
  async findOne(_id: string): Promise<User> {
    return this.userModel.findById(_id);
  }

  /** 添加单个用户 */
  async addOne(body: CreateUserDTO): Promise<void> {
    await this.userModel.create(body);
  }

  /** 编辑单个用户 */
  async editOne(_id: string, body: EditUserDTO): Promise<void> {
    await this.userModel.findByIdAndUpdate(_id, body);
  }

  /** 删除单个用户 */
  async deleteOne(_id: string): Promise<void> {
    await this.userModel.findByIdAndDelete(_id);
  }
}
```

##### 7.配置userController
```typescript
import {
  Controller,
  Body,
  Delete,
  Get,
  Param,
  Post,
  Put,
} from '@nestjs/common';
import { CreateUserDTO, EditUserDTO } from './user.dto';
import { User } from './user.interface';
import { UserService } from './user.service';

interface UserResponse<T = unknown> {
  code: number;
  data?: T;
  message: string;
}

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get('users')
  async findAll(): Promise<UserResponse<User[]>> {
    return {
      code: 200,
      data: await this.userService.findAll(),
      message: 'success',
    };
  }

  @Get(':_id')
  async findOne(@Param('_id') _id: string): Promise<UserResponse<User>> {
    return {
      code: 200,
      data: await this.userService.findOne(_id),
      message: 'success',
    };
  }

  @Post()
  async addOne(@Body() body: CreateUserDTO): Promise<UserResponse> {
    await this.userService.addOne(body);
    return {
      code: 200,
      message: 'success',
    };
  }

  @Put(':_id')
  async editOne(
    @Param('_id') _id: string,
    @Body() body: EditUserDTO,
  ): Promise<UserResponse> {
    await this.userService.editOne(_id, body);
    return {
      code: 200,
      message: 'Success.',
    };
  }

  @Delete(':_id')
  async deleteOne(@Param('_id') _id: string): Promise<UserResponse> {
    await this.userService.deleteOne(_id);
    return {
      code: 200,
      message: 'Success.',
    };
  }
}
```

参考文章：

[写给前端的 Nest.js 教程——10分钟上手后端接口开发](https://juejin.cn/post/6885751452015263758)

[Nestjs-Mongo](https://docs.nestjs.com/techniques/mongodb)

[How To Build a Blog with Nest.js, MongoDB, and Vue.js](https://www.digitalocean.com/community/tutorials/how-to-build-a-blog-with-nest-js-mongodb-and-vue-js)

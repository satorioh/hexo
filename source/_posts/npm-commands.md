---
title: npm常用命令
date: 2020-02-22 20:48:04
categories: NPM
tags:
- npm
permalink: npm-commands
---
#### 1.常用
```
npm -v                          // 查看版本
npm -l                          // 显示各个命令简单用法
npm config list                 // 查看npm配置信息
npm cache clean                 // 删除缓存目录下的所有数据
npm view <package> version      // 查看 package 的最新的版本信息
npm ls <package> (-g)           // 查看本地安装的 package 版本
npm update package              // 更新本地 package
```
<!--more-->

#### 2.配置
```
npm config list                           // 查看npm信息；注意：此命令不是查看所有参数配置
npm config ls -l                          // 可查看 npm 的所有配置

npm config set <key> <value> [--global]   // 设置指定参数
npm config get <key>                      // 获取现有参数值
npm config delete <key>                   // 删除指定参数，此时参数值会变为默认值
npm config edit                           // 编辑全量的npm配置文件（.npmrc）
```

#### 3.缓存
```
npm cache clean                  // 删除缓存目录下的所有数据
```

#### 4.查看远程包信息
```
npm view <package> versions      // 查看 package 的所有版本信息
npm view <package> version       // 查看 package 的最新的版本信息
npm view <package> dependencies  // 查看包的依赖关系
```

#### 5.查看本地包信息
```
npm ls <package> (-g)           // 查看本地安装的 package 版本
```

#### 6.安装
```
npm install package --save [-S]     // 安装运行时依赖包
npm install package --save-dev [-D] // 安装开发时依赖包
npm install package -g              // 全局安装 package
npm install package@version         // 安装指定版本 package
```

#### 7.卸载
```
npm uninstall package          // 卸载本地package
npm uninstall -g package       // 卸载全局package
```

#### 8.更新升级
```
npm outdated                  // 列出未升级到最新版本的依赖
npm update package            // 更新本地package
npm update package -g         // 更新全局package
```

#### 9.查看安装目录
```
npm root      // 查看本地安装目录
npm root -g   // 查看全局安装目录
```

#### 10.版本管理
- ~: 当安装依赖时获取到有新版本时，安装到 x.y.z 中 z 的最新的版本。即保持主版本号、次版本号不变的情况下，保持修订号的最新版本。
- ^: 当安装依赖时获取到有新版本时，安装到 x.y.z 中 y 和 z 都为最新版本。 即保持主版本号不变的情况下，保持次版本号、修订版本号为最新版本。

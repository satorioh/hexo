---
title: 使用Gulp压缩CSS/JS
date: 2017-06-30 17:16:03
categories: 工具
tags:
- gulp
permalink: gulp-compress-css-js
---
# 一、安装
### 1.安装gulp
    npm install -g gulp
### 2.检查gulp 版本
    gulp -v
<!--more-->
### 3.在项目文件夹下安装gulp
    npm install --save-dev gulp

#     二、压缩JS
### 1.安装gulp-uglify模块
    npm install gulp-uglify
### 2.在项目根目录创建gulpfile.js文件
### 3.在gulpfile.js文件中写入代码
```javascript
// 获取 gulp
var gulp = require('gulp');
// 获取 uglify 模块（用于压缩 JS）
var uglify = require('gulp-uglify');

// 压缩 js 文件
// 在命令行使用 gulp script 启动此任务
gulp.task('jscompress', function() {
    // 1. 找到文件
   return gulp.src('js/1.js')
    // 2. 压缩文件
        .pipe(uglify())
        // 3. 另存压缩后的文件
        .pipe(gulp.dest('dist/js'));
});
```
### 4.命令行中切换到gulpfile.js所在的目录，执行如下命令开始压缩
    gulp script
### 5.在gulpfile.js中添加自动监视任务
```javascript
// 在命令行使用 gulp auto 启动此任务
gulp.task('auto', function () {
    // 监听文件修改，当文件被修改则执行 script 任务
    gulp.watch('js/1.js', ['jscompress']);
});
```
### 6.在gulpfile.js中设置默认任务（即命令行中输入gulp执行的任务）
```javascript
// 使用 gulp.task('default') 定义默认任务
// 在命令行使用 gulp 启动 script 任务和 auto 任务
gulp.task('default', ['auto']);
```

# 三、压缩CSS
### 1.安装gulp-clean-css模块
    npm install gulp-clean-css
### 2.在gulpfile.js文件中添加相应任务
```javascript
// 获取 cleancss 模块（用于压缩 CSS）
var cleanCSS = require('gulp-clean-css');

// 压缩 css 文件
// 在命令行使用 gulp csscompress 启动此任务
gulp.task('csscompress', function() {
    // 1. 找到文件
  return  gulp.src('css/my.css')
    // 2. 压缩文件
        .pipe(cleanCSS())
        // 3. 另存压缩后的文件
        .pipe(gulp.dest('dist/css'));
});
```
### 3.修改添加对应的自动监视任务
```javascript
// 在命令行使用 gulp auto 启动此任务
gulp.task('auto', function () {
    // 监听文件修改，当文件被修改则执行 script 任务
    gulp.watch('js/1.js', ['jscompress']);
    gulp.watch('css/my.css', ['csscompress']);
});
```

# 四、重命名文件
### 1.安装gulp-rename模块
    npm install gulp-rename
### 2.修改gulpfile.js内代码
```javascript
// 获取 gulp
var gulp = require('gulp');
// 获取 uglify 模块（用于压缩 JS）
var uglify = require('gulp-uglify');
// 获取 cleancss 模块（用于压缩 CSS）
var cleanCSS = require('gulp-clean-css');
var rename = require("gulp-rename");
var gutil = require('gulp-util');//错误信息输出

// 压缩 js 文件
// 在命令行使用 gulp jscompress 启动此任务
gulp.task('jscompress', function() {
    // 1. 找到文件
   return gulp.src('js/1.js')
       .pipe(rename({suffix: '.min'}))
    // 2. 压缩文件
        .pipe(uglify())
        .on('error', function (err) { gutil.log(gutil.colors.red('[Error]'), err.toString()); })//错误信息输出
        // 3. 另存压缩后的文件
        .pipe(gulp.dest('dist/js'));
});
// 压缩 css 文件
// 在命令行使用 gulp csscompress 启动此任务
gulp.task('csscompress', function() {
    // 1. 找到文件
   return gulp.src('css/my.css')
        .pipe(rename({suffix: '.min'}))
    // 2. 压缩文件
        .pipe(cleanCSS())
        // 3. 另存压缩后的文件
        .pipe(gulp.dest('dist/css'));
});

// 在命令行使用 gulp auto 启动此任务
gulp.task('auto', function () {
    // 监听文件修改，当文件被修改则执行 script 任务
    gulp.watch('js/1.js', ['jscompress']);
    gulp.watch('css/my.css', ['csscompress']);
});


// 使用 gulp.task('default') 定义默认任务
// 在命令行使用 gulp 启动 script 任务和 auto 任务
gulp.task('default', ['auto']);
```
# 五、增加ES6支持
### 1.安装gulp-babel模块
    npm install --save-dev babel-core gulp-babel babel-preset-es2015
### 2.修改文件
```javascript
// 获取 gulp
var gulp = require('gulp');
// 获取 uglify 模块（用于压缩 JS）
var uglify = require('gulp-uglify');
// 获取 cleancss 模块（用于压缩 CSS）
var cleanCSS = require('gulp-clean-css');
var rename = require("gulp-rename");
var gutil = require('gulp-util');
var babel = require('gulp-babel');

// 压缩 js 文件
// 在命令行使用 gulp jscompress 启动此任务
gulp.task('jscompress', function() {
    // 1. 找到文件
   return gulp.src('js/1.js')
       .pipe(rename({suffix: '.min'}))
       //babel转码
       .pipe(babel({
           presets: ['es2015']
       }))
    // 2. 压缩文件
        .pipe(uglify())
       .on('error', function (err) { gutil.log(gutil.colors.red('[Error]'), err.toString()); })
        // 3. 另存压缩后的文件
        .pipe(gulp.dest('js'));
});
// 压缩 css 文件
// 在命令行使用 gulp csscompress 启动此任务
gulp.task('csscompress', function() {
    // 1. 找到文件
   return gulp.src('css/my.css')
        .pipe(rename({suffix: '.min'}))
    // 2. 压缩文件
        .pipe(cleanCSS())
        // 3. 另存压缩后的文件
        .pipe(gulp.dest('css'));
});

// 在命令行使用 gulp auto 启动此任务
gulp.task('auto', function () {
    // 监听文件修改，当文件被修改则执行 script 任务
    gulp.watch('js/1.js', ['jscompress']);
    gulp.watch('css/my.css', ['csscompress']);
});


// 使用 gulp.task('default') 定义默认任务
// 在命令行使用 gulp 启动 script 任务和 auto 任务
gulp.task('default', ['auto']);


```


参考资料：

[https://github.com/nimojs/gulp-book](https://github.com/nimojs/gulp-book)
[http://www.cnblogs.com/Tzhibin/p/4318457.html](http://www.cnblogs.com/Tzhibin/p/4318457.html)
[https://stackoverflow.com/questions/34398338/uglification-failed-unexpected-character](https://stackoverflow.com/questions/34398338/uglification-failed-unexpected-character)
---
title: Morgan 和 Winston
date: 2021-08-20 10:52:13
categories: Node
tags:
- morgan
- winston
permalink: morgan-winston
---
### 一、[Morgan](https://github.com/expressjs/morgan)
#### 1.使用
```javascript
const morgan = require("morgan");
const Logger = require("../models/logger");

const stream = {
  write: (message) => Logger.http(message),
};

const morganMiddleware = morgan(
  ":method :url :status :res[content-length] - :response-time ms",
  { stream }
);

module.exports = morganMiddleware;

// 使用中间件
app.use(morganMiddleware);
```
<!--more-->

### 二、[Winston](https://github.com/winstonjs/winston)
#### 1.使用
```javascript
const winston = require("winston");

const isDevelopment = process.env.NODE_ENV === "development";

const levels = {
  error: 0,
  warn: 1,
  info: 2,
  http: 3,
  debug: 4,
};

const colors = {
  error: "red",
  warn: "yellow",
  info: "green",
  http: "magenta",
  debug: "white",
};

winston.addColors(colors);

const format = isDevelopment
  ? winston.format.combine(
      winston.format.timestamp({ format: "YYYY-MM-DD HH:mm:ss:ms" }),
      winston.format.colorize({ all: true }),
      winston.format.printf(
        (info) => `${info.timestamp} ${info.level}: ${info.message}`
      )
    )
  : winston.format.combine(
      winston.format.timestamp({ format: "YYYY-MM-DD HH:mm:ss:ms" }),
      winston.format.printf(
        (info) => `${info.timestamp} ${info.level}: ${info.message}`
      )
    );

const transports = [new winston.transports.Console()];

const Logger = winston.createLogger({
  level: "debug",
  levels,
  format,
  transports,
});

module.exports = Logger;
```

参考文章：

[Nodejs日志系统指南](https://juejin.cn/post/6938741721308069895)


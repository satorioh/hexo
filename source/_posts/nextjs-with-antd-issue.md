---
title: Next.js 搭配 AntD 问题总结
date: 2021-09-01 17:16:23
categories: Next.js
tags:
- next.js
- antd
permalink: nextjs-with-antd-issue
---
#### 一、AntD
##### 1.[Warning: useLayoutEffect does nothing on the server](https://github.com/ant-design/ant-design/issues/30396)
解决：AntD降级到4.15.6

##### 2.[Warning: findDOMNode is deprecated in StrictMode](https://github.com/ant-design/ant-design/issues/26136)
解决：next.config.js中reactStrictMode设为false
<!--more-->

#### 二、Next.js
##### 1.[next js chunkloaderror loading chunk node_modules next_dist_client_dev_noop js failed](https://stackoverflow.com/questions/65858803/vercel-next-js-uncaught-in-promise-chunkloaderror-loading-chunk-0-failed)
解决：删除.next文件夹

##### 2.[Next.js API routes (and pages) should support reading files](https://github.com/vercel/next.js/issues/8251)
解决：[配置使用import导入](https://blog.vulcanjs.org/how-to-set-configuration-variables-in-next-js-a81505e43dad?gi=7ce8100a35fb)

##### 3.如何使用pm2启动
解决：
```json
"scripts": {
  "dev": "next dev",
  "build": "next build && PORT=4000 npm start",
  "start": "next start",
  "deploy": "pm2 start npm --name mlcNext --log-type=json --log-date-format 'YYYY-MM-DD HH:mm:ss' -- run build"
}
```

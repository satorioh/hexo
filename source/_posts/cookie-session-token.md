---
title: Cookie、Session 和 Token
date: 2021-05-31 16:38:41
categories: JavaScript
tags:
- js
permalink: cookie-session-token
---
### 一、Cookie
1.作用：用于在连接时证明客户端的身份

2.设置：服务端Set-Cookie方式
<!--more-->

3.相关字段：
- Expires 设置 cookie 的过期时间（时间戳），这个时间是客户端时间
- Max-Age 设置 cookie 的保留时长（秒数），同时存在 Expires 和 Max-Age 的话，Max-Age 优先
- Domain 设置生效的域名，默认就是当前域名，不包含子域名
- Path 设置生效路径，/ 全匹配
- Secure 设置 cookie 只在 https 下发送，防止中间人攻击
- HttpOnly 设置禁止 JavaScript 访问 cookie，防止XSS
- SameSite 设置跨域时不携带 cookie，防止CSRF

### 二、Session
#### 1.存储方式
可以储存在客户端，也可以存储在服务器

- 客户端存储：服务器通过set-cookie把信息写入浏览器cookie，在下次浏览器发送请求时，信息又通过 cookie 原样带回来，所以服务器什么东西都不用存。缺点是无法即时取消

- 服务端存储：将sessionID存储在客户端cookie中，在客户端发送请求时，拿到sessionId，再使用 id 在服务端 store 中获取 session 信息。

#### 2.后台是如何通过 SessionID 知道是哪个用户呢？
- 数据库存储关联：将 SessionID 与数据信息关联，存储在 Redis、Mysql 等数据库中；
- 数据加密直接存储：比如 JWT 方式，用户数据直接从 SessionID 值解密出来（此方式时 Cookie 名称以 Token 居多）

#### 3.服务端存储的缺点：
- 多集群支持: 当网站采用集群部署的时候，会遇到多台web服务器之间如何做session共享的问题。因为session是由单个服务创建，处理请求的服务器可能不是创建session的服务器，那么该服务器就无法拿到之前放入到session中的登录凭证之类的信息
- 性能差：当流量高峰期时，由于每个请求的用户信息都需要存储在数据库中，对资源会是一种负担
- 低扩展性：当扩容服务端的时候，session store也需要扩容。这会占用额外的资源和增加复杂性

### 三、Token
本质上 token 的功能就是和 session id 一模一样。区别在于，session id 一般存在 cookie 里，自动带上；token 一般是要你主动放在请求中，例如设置请求头的 Authorization 为 bearer:<access_token>

#### 1.JWT
客户端储存 session 信息的一种方式。

用户访问需要授权的连接时，可以把 token 放在 cookie，也可以在请求头带上 Authorization: Bearer <token>。（手动放在请求头不受 CORS 限制，不怕 CSRF）

缺点：

- 无法废弃: 在签发后，在到期之前会始终有效，无法中途废弃。
- 性能差: session方案中，cookie需要携带的sessionId是一个很短的字符串。但是由于jwt是无状态的，需要携带一些必要的信息，体积会比较大。
- 安全性：jwt中的payload是base64编码的，没有加密，因此不能存储敏感数据
- 续签: 传统的cookie续签方案都是框架自带的，session有效期30分钟，30分钟内如果有访问，有效期被刷新至30分钟。如果要改变jwt的有效时间，就需要签发新的jwt。一种方案是每次请求都更新jwt，这样性能太差了；第二种方案为每个jwt设置过期时间，每次访问刷新jwt的过期时间，就失去了jwt无状态的优势了

参考文章：

[前后端接口鉴权全解](https://ssshooter.com/2021-02-21-auth/)
[JSON Web Token 入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
[搞懂 JWT 这个知识点](https://mp.weixin.qq.com/s/9SC6Dtgkfvq1ehOq76bpbA)

---
title: Cookie、Session 和 Token
date: 2021-05-31 16:38:41
categories: JavaScript
tags:
- js
permalink: cookie-session-token
---
### 一、Cookie
#### 1.作用
用于在连接时证明客户端的身份

#### 2.设置
服务端Set-Cookie方式、客户端document.cookie
<!--more-->

#### 3.相关字段：
- Expires 设置 cookie 的过期时间（时间戳），这个时间是客户端时间
- Max-Age 设置 cookie 的保留时长（秒数），同时存在 Expires 和 Max-Age 的话，Max-Age 优先
- Domain 设置生效的域名，默认就是当前域名，不包含子域名
- Path 设置生效路径，/ 全匹配
- Secure 设置 cookie 只在 https 下发送，防止中间人攻击
- HttpOnly 设置禁止 JavaScript 访问 cookie，防止XSS
- SameSite 设置跨域时不携带 cookie，防止CSRF

#### 4.缺点
- 受CORS限制
- 存在CSRF风险
- 大小有限制，最大为 4kb

### 二、Session
#### 1.存储方式
可以储存在客户端，也可以存储在服务器，或者redis(推荐)

- 客户端存储：服务器通过set-cookie把信息写入浏览器cookie，在下次浏览器发送请求时，信息又通过 cookie 原样带回来，所以服务器什么东西都不用存。缺点是无法即时取消

- 服务端存储：将sessionID存储在客户端cookie中，在客户端发送请求时，拿到sessionId，再使用 id 在服务端 store 中获取 session 信息。

- redis存储：集中存储时，对分布式服务、性能比较好

#### 2.后台是如何通过 SessionID 知道是哪个用户呢？
- 数据库存储关联：将 SessionID 与数据信息关联，存储在 Redis、Mysql 等数据库中；
- 数据加密直接存储：比如 JWT 方式，用户数据直接从 SessionID 值解密出来（此方式时 Cookie 名称以 Token 居多）

#### 3.服务端存储的缺点：
- 多集群支持: 当网站采用集群部署的时候，会遇到多台web服务器之间如何做session共享的问题。因为session是由单个服务创建，处理请求的服务器可能不是创建session的服务器，那么该服务器就无法拿到之前放入到session中的登录凭证之类的信息
- 性能差：当流量高峰期时，由于每个请求的用户信息都需要存储在数据库中，对资源会是一种负担
- 低扩展性：当扩容服务端的时候，session store也需要扩容。这会占用额外的资源和增加复杂性

#### 4.Session更新
- 每次请求都刷新: 开发一个 middleware（默认情况下所有请求都会经过该 middleware），如果校验Session有效，就更新Session的expires: 当前时间+过期时间
- 快过期时再刷新：频繁更新 session 会影响性能，如果某个用户一直在操作，同一个 sessionID 可能会长期有效，如果相关 cookie 泄露，可能导致比较大的风险，可以在生成 sessionID 的同时生成一个 refreshID，在 sessionID 过期之后使用 refreshID 请求服务端生成新的 sessionID（这个方案需要前端判断 sessionID 失效，并携带 refreshID 发请求，refreshID只能使用一次)。

#### 5.单设备登录
有些情况下，只允许一个帐号在一个端下登录，如果换了一个端，需要把之前登录的端踢下线（默认情况下，同一个帐号可以在不同的端下同时登录的）。

这时候可以借助一个服务保存用户唯一标识和 sessionId 值的对应关系，如果同一个用户，但 sessionId 不一样，则不允许登录或者把之前的踢下线(删除旧 session )。

### 三、Token
本质上 token 的功能就是和 session id 一模一样。区别在于，session id 一般存在 cookie 里，自动带上；token 一般是要你主动放在请求中，例如设置请求头的 Authorization 为 bearer:<access_token>
所以 token 更适合一次性的命令认证，设置一个比较短的有效期

#### 1.JWT
客户端储存 session 信息的一种方式。

用户访问需要授权的连接时，可以把 token 放在 cookie，也可以在请求头带上 Authorization: Bearer <token>。（手动放在请求头不受 CORS 限制，不怕 CSRF）

缺点：

- 无法废弃: 在签发后，在到期之前会始终有效，无法中途废弃。
- 性能差: session方案中，cookie需要携带的sessionId是一个很短的字符串。但是由于jwt是无状态的，需要携带一些必要的信息，体积会比较大。
- 安全性：jwt中的payload是base64编码的，没有加密，因此不能存储敏感数据
- 续签: 传统的cookie续签方案都是框架自带的，session有效期30分钟，30分钟内如果有访问，有效期被刷新至30分钟。如果要改变jwt的有效时间，就需要签发新的jwt。一种方案是每次请求都更新jwt，这样性能太差了；第二种方案为每个jwt设置过期时间，每次访问刷新jwt的过期时间，就失去了jwt无状态的优势了

#### 2.JWT token校验
当 server 收到浏览器传过来的 token 时，它会首先取出 token 中的 header + payload，根据密钥生成签名，然后再与 token 中的签名比对，如果成功则说明签名是合法的，即 token 是合法的。而且你会发现 payload 中存有我们的 userId，所以拿到 token 后直接在 payload 中就可获取 userid，避免了像 session 那样要从 redis 去取的开销

#### 3.JWT token更新
可以同时生成 JWT Token 与 Refresh Token，其中 Refresh Token 的有效时间较长，长于 JWT Token，这样当 JWT Token 过期之后，使用  Refresh Token 获取新的 JWT Token 与 Refresh Token。

#### 4.适合的场景
设置较短的过期时间，更适合做一些一次性的安全认证，用来做下载链接、邮箱验证、或短时间内的一次性验证业务是非常好的

参考文章：

[前后端接口鉴权全解](https://ssshooter.com/2021-02-21-auth/)

[JSON Web Token 入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)

[搞懂 JWT 这个知识点](https://mp.weixin.qq.com/s/9SC6Dtgkfvq1ehOq76bpbA)

[常见登录鉴权方案](https://75.team/post/common-login-authencation-scheme)

[你管这破玩意儿叫Token](https://zhuanlan.zhihu.com/p/373092817)

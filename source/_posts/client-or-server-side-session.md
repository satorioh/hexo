---
title: 关于 Client Side Session 和 Server Side Session 的思考
date: 2021-09-18 14:27:41
categories: 架构
tags:
- session
- jwt
permalink: client-or-server-side-session
---
#### 一、本文中的定义(我的理解)
##### 1.Server Side Session
以cookie+session模式为代表，sessionData存储在数据库，生成的sessionId存储在redis/数据库，并通过set-cookie种到客户端

##### 2.Client Side Session
以jwt为代表，sessionData存储在数据库，生成的token存储在客户端，以实现服务端无状态
<!--more-->

#### 二、基本流程
##### 1.cookie+session的流程
> (1)用户使用账号/密码登录，服务器去数据库验证，验证通过后，服务器生成sessionId(关联userId，过期时间)，set-cookie保存到客户端，同时将sessionId(关联userId，过期时间)存储到redis
> 
> (2)客户端每次请求自动带上cookie中的sessionId，服务端校验，校验通过返回请求的资源
> 
> (3)校验不通过或者sessionId过期了，返回401，客户端跳转登录页

##### 2.jwt的流程
> (1)用户使用账号/密码登录，服务器去数据库验证，验证通过后，服务器根据secret密钥生成access_token，返回access_token给客户端
> 
> (2)客户端收到access_token后，将其存储在local storage，之后每次请求，都在header的Authorization中带上access_token
> 
> (3)服务端对请求中的access_token校验（签名、过期时间），验证通过，则返回资源响应
> 
> (4)签名验证未通过直接返回401，客户端跳转登录页

#### 三、过期续签策略
##### 1.cookie+session的续签
> (1)每次请求刷新ttl，未请求则一段时间后sessionId过期，重新登录，性能略差
>
> (2)生成sessionId的同时，再多生成一个refreshId，前者过期时间短，后者过期时间较长，sessionId过期后，查询refreshId，如未过期，则直接颁发新的sessionId，如果refreshId过期，则需要重新登录

##### 2.jwt的续签
> (1)每次请求都颁发新token，未请求则一段时间后token过期，重新登录，性能差；或者在redis中记录ttl，但就不是无状态的了
> 
> (2)生成access_token的同时，再多生成一个refresh_token，前者过期时间短，后者过期时间较长，access_token过期后，使用refresh_token获取新的access_token，如果refresh_token也过期，则需要重新登录；两个token可以都寸客户端，但不太安全；或者refresh_token存redis，但就不是无状态的了

#### 四、适用场景
##### 1.cookie+session
需要对在线账号数量、设备数量做限制的，比如同一时间同一账号只能有一个登录

##### 2.jwt
安全认证，用来做下载链接、邮箱验证、或短时间内的一次性验证业务、可以多处同时在线的

#### 五、本质
> 以 JWT 自身的设计逻辑来说，靠的是『算法』验证，而非『数据』验证，这注定是一种『离线验证』方式。
> 
> 各类试图为 JWT 加入吊销机制的方式，实际上都需要引入『数据』，不管这个『数据』是什么形式存在的，都势必变为『在线验证』方式。
> 
> 各类优化的本质，还是将『数据』做压缩和分发，尽可能让这个『数据』去中心化。
> 
> jwt 本来设计之初就是无状态的，就是"算法验证而非数据验证"，是因为有用户下线需求这些，才搞出了refresh token，token黑名单这些。(这些都需要靠数据验证)

参考文章：

[有关 Session 的那些事儿](https://blog.by24.cn/archives/about-session.html)

[有关 JWT 能否真正实现无状态](https://www.v2ex.com/t/757882)

[（译）别再使用 JWT 作为 Session 系统！](https://learnku.com/articles/22616)

[讲真，别再使用JWT了！](https://cloud.tencent.com/developer/article/1464231)

[Implementing refresh tokens using JWT](https://wanago.io/2020/09/21/api-nestjs-refresh-tokens-jwt/)

[Using server-side sessions instead of JSON Web Tokens](https://wanago.io/2021/06/07/api-nestjs-server-side-sessions-instead-of-json-web-tokens/)

[express-session设置session详解](https://cloud.tencent.com/developer/article/1467247)

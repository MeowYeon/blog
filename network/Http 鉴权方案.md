# Http 鉴权方案
## Http 鉴权方案

### Http Basic认证
- 如果有未带Authorization的请求，服务器返回WWW-Authenticate头部的401请求，具体格式为：
WWW-Authenticate: Basic realm=""
- 浏览器接到响应后，弹框提示用户输入
- 浏览器发送一个带Authrization头部的请求，具体格式为：
Authorization: Basic [base64(user:passwd)]
- 服务端取Authorization头部，然后校验

登录成功之后，浏览器并不主动清除用户信息

### Cookie-session
- 浏览器初次请求服务器，服务器返回响应头Set-Cookie，具体格式为：
Set-Cookie: SessionID=sid
- session 保存在服务端，该session对应一批会话信息
- 浏览器再次请求带Cookie头部，具体格式为：
Cookie: SessionID=sid
- 服务端通过该session获取到会话信息，确保用户已经成功登录
- cookie session都会有过期，过期后就需要再次登录了

同样，cookie也会有泄露的风险，也会被截取重放

### Token-JWT
- 浏览器初次请求服务之后，服务端返回JWT令牌
- 浏览器后续请求都携带Authorization头部，具体格式为：
Authorization: Bearer token
- 服务端接到请求后，对token进行校验，防止token篡改

### OAuth
待续

## 参考链接
[前后端常见的几种鉴权方式](https://www.lishuaishuai.com/nodejs/1167.html?soure=jj)
[JWT 官方介绍](https://jwt.io/introduction)  
[阮一峰 JWT教程](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)  
[Cookie 介绍](https://segmentfault.com/a/1190000004556040)  
[Cookie 详细](https://www.cnblogs.com/ajianbeyourself/p/4900140.html)  
[Session Cookie 过期理解](https://blog.csdn.net/u010002184/article/details/89413166)  
[阮一峰 OAuth介绍](https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)

## 时间线
> 2020.12.17 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

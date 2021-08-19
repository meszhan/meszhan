# JWT

### 什么是JWT

> 基于JSON的、用于在网络上声明某种主张的token。JWT一般由三部分组成：header、payload和signature
>
> + 头信息：指定JWT使用的签名算法，例如HMAC-SHA256
> + payload：JWT的意图，比如令牌生成时间和登录信息
> + 未签名的令牌由base64url编码的header和payload拼接而成，signature通过私有的key计算而成
>
> JWT通常被用作保护服务端的资源，客户端通常将JWT通过HTTP的Authorization header发送给服务端，服务端使用自己的key计算、验证签名判断JWT是否可信

### 优点

#### 易于水平扩展

> 使用了token后，服务端不需要处理session，在做分布式扩展的时候，不需要处理session复制

#### 防护CSRF攻击

> CSRF攻击就是利用cookie-session漏洞的攻击
>
> 比如你在某银行站点bank.com转账，转账完成后没有退出站点，又访问了一个恶意网站，在该网站上隐藏了一个转账的表单，通过某个按钮来吸引点击提交表单或者使用JavaScript自动提交，尽管无法通过js直接盗用cookie，但是向某一个网站发送请求的时候会被自动发送
>
> 如果将JWT通过HTTP header发送给服务端而不是cookie自动发送，可以有效的防护CSRF。服务端代码在完成认证后，在HTTP response的header中返回JWT，前端将JWT存储在localStorage中或HttpOnly为false的cookie
>
> 发送请求时，取出JWT，通过header发送给服务端通过认证。但是这对于XSS攻击太友好了，它可以直接通过JavaScript访问JWT，然后通过表单发送到其他站点

### 缺点

#### 更多的空间占用

> 考虑cookie的4KB限制，和localStorage的XSS攻击

#### 更不安全

> localStorage的XSS攻击

#### 无法作废已经颁布的token

> 所有的认证信息都在JWT中，由于服务端不保存任何状态，因此即使token被盗取，也无法作废，除非token主动过期

#### 不易应对数据过期

> 因为无法作废已经颁布的token，在token过期前，payload不会改变

### 如何正确使用JWT

> + 绝对不使用localStorage存储token，使用HttpOnly为true的cookie，因此cookie只能通过自动回传
> + 在JWT中加入一个随机值作为CSRF token，将该token保存在cookie中并可访问，将该token作为header传回，服务端认证时，从JWT中取出CSRF token与header中的CSRF token比较
> + 考虑cookie的4KB大小限制，JWT中只存放需要的认证信息，其他信息数据库获取




































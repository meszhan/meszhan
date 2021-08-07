# CSP内容安全策略

https://juejin.cn/post/6844903841238876174

### 背景

#### 同源策略

相同协议、主机、端口

#### 跨站脚本攻击XSS

### CSP内容安全策略

> 开发者明确告诉客户端（制定比较严格的策略和规则），哪些外部资源可以加载和执行，即使攻击者发现漏洞，也没办法注入脚本

### 启用CSP

#### 通过HTTP头信息的Content-Security-Policy字段

```
Response Headers:
	content-security-policy:
```

#### 通过网页的meta标签

```htmls
<meta http-equiv="Content-Security-Policy" content="script-src 'self'">
```

我们在页面上引入一个CDN，但是meta的content只设置为script-src 'self'，会导致cdn无法加载

### 常见配置

> 该策略允许加载同源的图片、脚本、AJAX和CSS资源，并组织加载其他任何资源、

```
default-src 'none';script-src 'self';connect-src 'self';img-src 'self';style-src 'self'
```

### CSP的意义

> + 防止XSS的攻击
>
>   CSP实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单
>
>   它的实现和执行全部交由浏览器完成，开发者只需要提供配置
>
> + 增强网页安全性
>
>   攻击者即使发现了漏洞，也无法注入脚本，除非控制了一台列入了白名单的主机

### CSP的分类

+ Content-Security-Policy配置好并启用后，不符合CSP的外部资源会被阻止加载
+ Content-Security-Policy-Report-Only表示不执行限制选项，只是记录违反限制的行为，需要与report-url选项配合使用

### CSP的使用

#### **HTTP Header上使用**

```
"Content-Security-Policy:" 策略
"Content-Security-Policy-Report-Only:" 策略
```

#### **HTML上使用**

```
<meta http-equiv="content-security-policy" content="策略">
<meta http-equiv="content-security-policy-report-only" content="策略">
```

> Meta标签与HTTP头只是形式不同，但作用是一致的。如果同时定义，优先采用HTTP头中的定义
>
> 如果用户浏览器已经为当前文档执行了一个CSP策略，跳过Meta的定义
>
> 如果Meta标签缺少content属性同样跳过

### 策略怎么写

```
// 限制所有的外部资源，都只能从当前域名加载
Content-Security-Policy: default-src 'self'

// default-src 是 CSP 指令，多个指令之间用英文分号分割；多个指令值用英文空格分割
Content-Security-Policy: default-src https://host1.com https://host2.com; frame-src 'none'; object-src 'none'  

// 错误写法，第二个指令将会被忽略
Content-Security-Policy: script-src https://host1.com; script-src https://host2.com

// 正确写法如下
Content-Security-Policy: script-src https://host1.com https://host2.com

// 通过report-uri指令指示浏览器发送JSON格式的拦截报告到某个地址
Content-Security-Policy: default-src 'self'; ...; report-uri /my_amazing_csp_report_parser; 
```

```
{  
  "csp-report": {  
    "document-uri": "http://example.org/page.html",  
    "referrer": "http://evil.example.com/",  
    "blocked-uri": "http://evil.example.com/evil.js",  
    "violated-directive": "script-src 'self' https://apis.google.com",  
    "original-policy": "script-src 'self' https://apis.google.com; report-uri http://example.org/my_amazing_csp_report_parser"  
  }  
} 
```

#### 常用CSP指令
































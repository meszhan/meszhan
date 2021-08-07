# Source Map

> Source Map是将编译、打包、压缩后的代码映射回源码的过程，打包压缩后的代码不具有高可读性，想要映射到源码上就需要source map
>
> map文件只要不打开开发者工具，浏览器就不会加载

### 线上环境处理方案

+ Hidden-source-map：借助第三方错误监控平台Sentry使用
+ nosources-source-map：只会显示具体行数和查看源代码的错误栈，比sourcemap的安全性高
+ sourcemap：通过nginx设置将.map文件只对公司内网开放

> 要避免在生产中使用inline-和eval-的sourcemap，因为会增加bundle大小，降低性能


# Fetch

>  Fetch API提供了一个用于获取资源包括跨网络的接口

### 概念和用法

> Fetch提供了Request和Response对象以及其他与网络请求有关东西的通用定义。允许它们在未来需要的任何地方使用，无论是Service Worker、缓存API还是其他处理或修改请求和响应的类似事物，还是可能需要生成响应的热和类型的用力和编程方式

> 还定义了相关的概念，例如CORS和HTTP Origin的标头语义，取代了在其他地方的单独语义

> 要发出请求并获取资源，使用WindowOrWorkerGlobalScopr.fetch()方法，它在多个接口中实现，特别是Window和WorkerGlobalScope中，它几乎可以在您想要获取资源的任何上下文中使用

> fetch方法采用一个强制性参数，即要获取资源的路径。返回一个Promise解析Response为该请求的，只要服务器响应标头，即使服务器响应是HTTP错误状态。可以选择传入一个init选项对象作为第二个参数

> 一旦Response被检索到，有很多方法可以用于确定主体内容，并处理响应
>
> 可以使用Request和Reponse构造函数直接创建请求和响应，但并不推荐如此

#### 与jQuery的区别

+  即使fetch发送请求接收到的响应式HTTP 404或500，它都不会拒绝，而是会正常解析，并且只会在网络故障或任何阻止请求完成的情况下才会拒绝
+ fetch除非设置初始化init选项，否则不会发送跨域cookie

#### 中止提取

### 获取接口

#### WindowOrWorkerGlobalScope.fetch()

> 用于获取资源的方法

#### Headers

> 表示响应或请求标头，允许查询它们并根据结果采取不同的操作

#### Request

> 代表一个资源请求

#### Response

> 表示对资源的响应








































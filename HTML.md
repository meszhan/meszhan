# HTML

### HTML DOM

在HTML文档对象模型DOM中每个东西都是节点：

+ 文档本身document就是一个文档对象
+ 所有HTML元素都是元素节点
+ 所有HTML属性都是元素节点
+ 插入到HTML元素文本是文本节点
+ 注释是注释节点

#### addEventListener添加事件

参数列表

+ event：事件名，没有on前缀
+ function：回调函数，事件对象会作为第一个参数传入
+ useCapture：表示事件是否在捕获阶段执行，false表示在冒泡阶段执行，默认为false

##### 移除事件removeEventListener

```javascript
document.getElementById("myDIV").addEventListener("mousemove", myFunction);
function myFunction() 
{
    document.getElementById("demo").innerHTML = Math.random();
}
function removeHandler() 
{
    document.getElementById("myDIV").removeEventListener("mousemove", myFunction);
}
```

#### appendChild添加子元素

向节点的子节点列表末尾添加新的子节点，如果已经存在，从文档树中删除，并重新插入新位置

如果是DocumentFragment节点，不直接插入，而是把子节点按序插入当前节点的childNodes数组的末尾，所以不会影响父子关系

#### childNodes和children

+ childNodes返回Node的数组，包括了文本节点和注释节点，也就是所有类型的Nodes

+ children返回Element的集合，只包括元素节点，也就是nodeType为1的Nodes

+ childNodes和children都是LIVE类型的，改变子节点或元素个数，length属性会马上改变

+ children是Element的属性，childNodes是Node的属性，不过因为Element继承自Nodes，因此Element同样可以访问childNodes

  但是Node却不能访问children属性（undefined）

Element继承了Node类，也就是说Element是Node多种类型的一种，只有当NodeType为1时，Node才是Element，另外Element扩展了Node，拥有id、class和children等属性

##### nodeType的值

+ 元素elment：1
+ 属性attr：2
+ 文本text：3
+ 注释comments：8
+ 文档document：9

```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Demo</title>
</head>
<body>
    <div id="test">
        <p>One</p>
        <P>Two</p>
    </div>
    <script>
        var oDiv=document.getElementById("test");
        console.log(oDiv.children[0] instanceof Node);        //true
        console.log(oDiv.children[0] instanceof Element);    //true

        console.log(oDiv.childNodes[0] instanceof Node);    //true
        console.log(oDiv.childNodes[0] instanceof Element);    //false

        console.log(typeof oDiv.childNodes[0].children);    //undefined
        console.log(typeof oDiv.childNodes[0].childNodes);    //object
    </script>
</body>
</html>
```

#### classList返回元素的类名，作为DOMTokenList对象

+ add：添加一个或多个类名
+ contains：返回boolean，指定类名是否存在
+ item：返回索引值对应的类名
+ remove：移除一个或多个类名，不存在的类名会报错
+ toggle：在元素中切换类名，如果类名存在则移除并返回false，不存在则添加类名并返回true

#### 其他属性

+ nodeName返回元素的标记名（大写）

+ nodeType返回元素的节点类型

+ nodeValue返回元素的节点值

### HTML语义化

#### 语义化的好处

+ 在没有CSS的情况下，也能呈现出较好的内容结构

+ 提高代码可读性，代码结构更容易理解
+ 有利于SEO：爬虫依赖标签来确定关键字的权重，可以与搜索引擎良好的沟通，帮助爬虫抓取更多有效信息
+ 方便其他设备对页面进行解析

#### 语义化标签

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/11/29/16eb53662136c6de~tplv-t2oaga2asx-watermark.awebp)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/11/29/16eb536c55aa8b0e~tplv-t2oaga2asx-watermark.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4870901ed8324467974eace62aece41d~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73501a58918a45e5aa73af3162913283~tplv-k3u1fbpfcp-watermark.awebp)

##### 常用语义化标签

+ h1-h6：页面标题

+ header：定义页面的介绍展示区域，包括网站logo、导航、全站链接以及搜索框
+ nav元素：定义页面的导航链接部分区域
+ main：定义页面主要内容，一个页面只能使用一次
+ article：定义页面独立的内容，可以有自己的header、footer，内部可以嵌套article，专用于博客文章
+ section：标记文档的各个部分
+ aside：定义与主要内容相关的内容块，显示为侧边栏
+ footer：文档的底部区域
+ small：为不重要的内容定义小字体

### Preload和Prefetch优化

#### prefetch

prefetch叫链接预取，是一种浏览器机制，利用浏览器空闲时间来下载或预取用户在不久的将来可能访问的文档。网页向浏览器提供一组预取提示，并在浏览器完成当前页面加载后静默的拉取指定的文档并将其存储在缓存中。当用户访问其中一个预取文档时，可以快速的从浏览器缓存中得到。

> 浏览器通过标签实现预加载,rel="prefetch"被称为Resource-Hints资源提示，辅助浏览器进行资源优化的指令

```html
<head>
    ...
    <link rel="prefetch" href="static/img/ticket_bg.a5bb7c33.png">
    ...
</head>
```

通过prefetch预取到资源后，如果页面中需要再次请求该资源，network也会增加一次新的访问请求，但是请求status虽然也是200，却有一个特殊标记—prefetch cache，表明请求来自prefetch缓存

#### preload

> preload与prefetch都属于浏览器的Resource-Hints，辅助浏览器进行资源优化，但是被翻译为预加载

rel属性为preload能够让我们在HTML页面中元素内部书写一些声明式的资源获取请求，可以指定哪些资源在页面记载完成后即刻需要。对于即刻需要的资源，在页面加载的生命周期早期阶段开始获取，在浏览器的主渲染机制介入前进行预加载。preload使得资源可以更早的得到加载并可用，且不易阻塞页面的初步渲染，进而提升性能。

也就是通过标签显示的声明一个高优先级的资源，强制浏览器提前请求资源，同时不阻塞文档正常的资源加载onload

```javascript
<head>
    ...
    <link rel="preload" as="font" href="<%= require('/assets/fonts/AvenirNextLTPro-Demi.otf') %>" crossorigin>
    <link rel="preload" as="font" href="<%= require('/assets/fonts/AvenirNextLTPro-Regular.otf') %>" crossorigin>
    ...
</head>
```

#### 总结

+ preload和prefetch本质都是预加载，先记载后使用，将加载与使用解耦

+ preload与prefetch不会阻塞页面的onload

+ preload声明当前页面的关键资源，强制浏览器尽快加载，prefetch声明将来可能用到的资源，在浏览器空闲时加载

+ preload加载字体资源时必须设置crossorigin属性，否则会导致重复向服务器请求

  如果不设置crossorigin属性，浏览器会采用匿名模式CORS去preload，导致两次请求无法共用缓存

### 行内元素和块级元素

#### 区别

+ 默认情况下，行内元素不会以新的一行开始，块级元素会新起一行

  行内元素当一行内放不下时才会换行，宽度随着元素内容变化，块级元素宽度自动填满父元素宽度

+ 块级元素可以设置width、height属性，行内元素设置无效

+ 块级元素所有方向的padding和margin都有效，行内元素只有水平方向的padding和margin有效

+ 块级元素可以包含行内元素和块级元素，行内元素不能包含块级元素

+ 居中设置

  + 行内元素：利用text-align和line-height
  + 块级元素：水平方向需要设置填满父元素宽度，然后利用margin:0 auto，垂直方向需要利用line-height

```css
margin:0 aut0;
height:30px;
line-height:30px
```

#### 常见的行内元素和块级元素

##### 行内元素

<span>` `<a>` `<lable>` `<strong>` `<b>` `<small>` `<abbr>` `<button>` `<input>` `<textarea>` `<select>` `<code>` `<img>` `<br>` `<q>` `<i>` `<cite>` `<var>` `<kbd>` `<sub>` `<bdo>

##### 块级元素

+ h1-h6
+ div
+ p
+ li
+ form
+ header
+ hr
+ ol
+ article
+ canvas
+ video
+ audio

### HTML中href与src的区别

+ src指向物件的来源地址，是引入，在img、script和iframe等元素上使用。href是超文本引用，指向需要连接的地方，与页面有关联

  是引用，在link和a元素上使用。src通常是拿取，href是引用

+ 可替换元素上使用src，src属性仅仅嵌入当前资源到当前文档定义的位置。href用于在涉及到文档和外部的资源之间建立一种关系

  href指定网络资源到位置，从而在当前元素或者当前文档和由当前属性定义的需要的锚点或资源直接定义一个链接或关系

+ 在请求src资源时会将指向的资源下载并应用到文档内，例如js脚本、img图片和iframe元素。当浏览器解析到该元素时，会暂停其他资源到下载和处理，直到将该资源加载、编译和执行完毕，图片和iframe同样，类似于将所指向的资源嵌入当前标签。

  所以一般会将js脚本放在底部而不是头部，但是当浏览器解析href时，会识别该文档为css文件，会下载并且不会停止对当前文档的处理

  也就是说css文件并不会阻塞文档的解析，所以我们推荐使用link而不是@import，因为@import添加的样式在页面载入后加载，可能会导致页面因重新渲染而闪烁




















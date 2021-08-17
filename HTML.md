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






































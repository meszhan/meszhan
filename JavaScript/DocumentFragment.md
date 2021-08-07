# DocumentFragment

### 引言

> 一般动态创建html元素，我们都是在js中createElement，然后appendChild。但是如果要添加大量的元素，会导致大量的重绘和回流。因此我们需要一个缓存区域来保存创建的节点，一次性的添加到父节点上。

### 作用

> 使用document.createDocumentFragment可以创建一个文档碎片，把所有的新节点附加在上面，然后一次性的把文档碎片的内容添加到DOM树上，这样只需要一次刷新

### 支持的方法和属性

#### 方法

+ cloneNode
+ removeChild
+ replaceChild

#### 属性

+ attributes
+ childNodes
+ firstChild
+ nodeType
+ nodeValue
+ nodeName
+ parentNode：null
+ 等等

#### 动态创建html节点的方法

```
createAttribute(name) // 用指定name创建特性节点
createComment(text) // 创建带文本的注释节点
createDocumentFragment() // 创建文档碎片节点
createElement(tagname) // 创建标签名为tagname的节点
createTextNode(text) //创建包含文本text的文本节点
```

> 在所有节点类型中，只有文档碎片节点在文档中没有相应的标记
>
> DOM规定文档碎片是一种轻量级文档，可以包含和控制节点，但是不会占用额外的资源，也就是它没有实际的标签或元素，不会影响其他节点的父子关系，类似于vue中的template、react中的Fragments

#### w3c

> DocumentFragment节点不属于文档树，继承的parentNode属性总是null
>
> 重要行为：当把一个DocumentFragment节点插入文档树时，插入的不是DocumentFragment自身，而是它的子孙节点
>
> 相当于DocumentFragment实际上是用来占位的，暂时存放一次插入文档的节点
>
> 同样，因为DocumentFragment节点不属于文档树，因此把创建的节点添加到该对象时，并不会导致页面的回流

### DEMO

```html
<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<title></title>
	</head>

	<body>
		<div id="main1"></div>
		<div id="main2"></div>
	</body>
	<script type="text/javascript">
		var d1 = new Date();
		//创建十个段落,常规的方式
		for(var i = 0; i < 1000; i++) {
			var p = document.createElement("p");
			var oTxt = document.createTextNode("段落" + i);
			p.appendChild(oTxt);
			document.getElementById("main1").appendChild(p);
		}
		var d2 = new Date();
		console.log("第一次创建需要的时间:" + (d2.getTime() - d1.getTime()));

		//使用了createDocumentFragment()的程序
		var d3 = new Date();
		var pFragment = document.createDocumentFragment();
		for(var i = 0; i < 1000; i++) {
			var p = document.createElement("p");
			var oTxt = document.createTextNode("段落" + i);
			p.appendChild(oTxt);
			pFragment.appendChild(p);
		}
		document.getElementById("main2").appendChild(pFragment);
		var d4 = new Date();
		console.log("第2次创建需要的时间:" + (d4.getTime() - d3.getTime()));
	</script>
</html>
```

### Q & A

### 如何在面试中回答此问题

#### 先回答什么是DocumentFragment

> DocumentFragment被称为文档碎片或文档片段，它不属于文档树document，也不会继承parentNode属性
>
> 它的用处来源于它的一个重要行为：**占位**
>
> 在把文档碎片插入document中时，插入的不是DocumentFragment，而是它内部的所有子孙节点

#### 作用

> 因为它不属于document文档树，因此如果我们把创建的节点添加到DocumentFragment内部，不会导致页面的的回流
>
> 这样我们就可以利用它暂时存放需要插入大量的节点，最后再插入DocumentFragment，由于它只起占位的作用，因此插入后不会影响节点间的继承关系，而且回流次数也从n次变为1次
>
> 除了插入外，更新节点也可以使用它，我们可以把document文档树中的节点移到DocumentFragment中操作，这样节点的一次性更新不会导致页面的重渲染，在所有节点更新完后，重新利用DocumentFragment插入即可，回流次数从n次变为2次（移除一次，插入一次）






















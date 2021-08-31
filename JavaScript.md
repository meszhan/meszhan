# JavaScript

## 语法型面试题

### 原型和原型链

每个构造函数constructor都有一个原型对象prototype，原型对象都包含一个指向构造函数的指针，而实例instance都包含一个指向原型对象是内部指针

```javascript
let Foo = function () {};

console.log(Foo.prototype.constructor === Foo); // true

let foo = new Foo();
// proto是隐式原型，prototype是显式原型
console.log(foo.__proto__ === Foo.prototype); // true
```

如果试图引用对象或实例instanhce的某个属性，会首先在对象内部寻找该属性，只至找不到，然后才在该对象的原型上去找该属性

```javascript
function Father() {
  this.property = true;
}
Father.prototype.getFatherValue = function () {
  return this.property;
};
function Son() {
  this.sonProperty = false;
}
//继承 Father
Son.prototype = new Father(); //Son.prototype被重写,导致Son.prototype.constructor也一同被重写
console.log(Son.prototype.constructor)// Father
Son.prototype.getSonVaule = function () {
  return this.sonProperty;
};
var instance = new Son();
console.log(instance.getFatherValue()); // true
```

#### 确定原型和实例的关系

##### instanceof操作符

```javascript
console.log(instance instanceof Father);// true
console.log(instance instanceof Son);// true
console.log(instance instanceof Object);// true
```

##### isPrototypeOf方法

```javascript
console.log(Father.prototype.isPrototypeOf(instance));// true
console.log(Son.prototype.isPrototypeOf(instance));// true
console.log(Object.prototype.isPrototypeOf(instance));// true
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9bb70f2afec41bdb7129721416367e4~tplv-k3u1fbpfcp-watermark.awebp)

### 虚拟列表

#### 为什么需要使用虚拟列表

假设我们需要展示10000条记录，如果同时将10000条记录渲染到页面中，可以看看花费了多少时间

```html
<button id="button">button</button><br>
<ul id="container"></ul>  
```

```javascript
document.getElementById('button').addEventListener('click',function(){
    // 记录任务开始时间
    let now = Date.now();
    // 插入一万条数据
    const total = 10000;
    // 获取容器
    let ul = document.getElementById('container');
    // 将数据插入容器中
    for (let i = 0; i < total; i++) {
        let li = document.createElement('li');
        li.innerText = ~~(Math.random() * total)
        ul.appendChild(li);
    }
    console.log('JS运行时间：',Date.now() - now);
    setTimeout(()=>{
      console.log('总运行时间：',Date.now() - now);
    },0)

    // print JS运行时间： 38
    // print 总运行时间： 957 
  })
```

在点击按钮时，会向页面中加入一万条记录，通过控制台输出，可以很明显观察到js运行时间和渲染完成的时间的差异

+ 在JS的事件循环中，当JS引擎所管理的主线程执行栈中的事件以及所有微任务事件全部执行完后才会触发渲染线程对页面进行渲染
+ 第一个console在页面渲染之前触发，得到的是大致的JS运行时间
+ 第二个console放在setTimeout中，在渲染完成后的下一次Event Loop中执行

当然我们也可以打开Chrome的Performance分析工具查看，大部分下的时间开销都发生在

+ Recalculate Style：样式计算，浏览器根据css选择器计算哪些元素应该应用哪些规则，确定每个元素具体的样式
+ Layout：布局，知道哪些元素应用哪些规则后，浏览器开始计算它占据的空间大小和在屏幕上的位置

#### 虚拟列表的定义

https://juejin.cn/post/6844903982742110216

虚拟列表：按需显示，只对可视区域进行渲染，对非可见区域中的数据不渲染或部分渲染，从而达到极高的渲染性能

假设有10000条记录需要同时渲染，屏幕的可见区域高度为500px，列表项高度为50px，则我们此时在屏幕上最多只能看见10个列表项，那么在首次渲染时只需加载10条

当滚动发生时，可以通过计算当前滚动值得到在屏幕可见区域应该显示的列表项

假设滚动发生，滚动条距离顶部1500px，则我们可得知在可见区域内的列表项为第4到13项

#### 实现

虚拟列表的实现，就是在首屏记载的时候，只加载可视区域内列表项，当滚动时，动态计算获得可视区域内的列表项，并将不在可视区域内的列表项删除

+ 计算当前可视区域起始列表项索引startIndex
+ 计算结束列表项索引endIndex
+ 计算当前可视区域的数据，并渲染到页面中
+ 计算startIndex对应的数据在整个列表中的偏移位置

因为是对可视区域进行渲染，所以为了保证列表容器高度并可以正常触发滚动，我们需要

```html
<div class="infinite-list-container">
    <div class="infinite-list-phantom"></div>
    <div class="infinite-list">
      <!-- item-1 -->
      <!-- item-2 -->
      <!-- ...... -->
      <!-- item-n -->
    </div>
</div>
```

+ infinite-list-container为可视区域的容器
+ infinite-list-phantom为容器内的占位，高度为总列表高度，形成滚动条
+ infinite-list为列表项的渲染区域

#### 列表项动态高度

实际应用中，当列表项中包含文本之类的可变内容，会导致列表项的高度并不相同

#### 面向未来

前面我们使用了监听scroll事件的方式来触发可视区域中数据的更新，当滚动发生后，scroll事件会频繁触发，造成大量的重复计算

可以使用IntersectionObserver替换监听scroll事件，IntersectionObserver可以监听目标元素是否出现在可视区域内，并在监听的回调事件中执行可视区域数据的更新，并且IntersectionObserver的监听回调函数是异步触发，不会随着目标元素的滚动触发，节约性能

#### 遗留问题

如果列表项中包含图片，并且列表高度会收到图片影响，因为图片会发送网络请求，因此无法保证在获取列表项真实高度时图片是否已经完成加载，导致计算不准确。

可以使用ResizeObserver来监听列表项内容区域的高度改变，实时获取每一列表项的高度。

### JSON.stringify的真正用法

https://juejin.cn/post/6844904016212672519

#### 特性一

对于undefined、任意函数以及symbol三个特殊值分别作为对象属性值、数组元素、单独的值时返回不同的结果

##### 作为对象属性值

```javascript
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  fn: function() {
    return true;
  }
};
JSON.stringify(data); // 输出：？

// "{"a":"aaa"}"
```

undefined、任意函数以及symbol作为对象属性值时stringify将跳过它们进行序列化

##### 作为数组元素

```
JSON.stringify(["aaa", undefined, function aa() {
    return true
  }, Symbol('dd')])  // 输出：？

// "["aaa",null,null,null]"
```

undefined、任意函数以及symbol作为数组元素时stringify将它们序列化为null

##### 单独序列化

```javascript
JSON.stringify(function a (){console.log('a')})
// undefined
JSON.stringify(undefined)
// undefined
JSON.stringify(Symbol('dd'))
// undefined
```

undefined、任意函数以及symbol作为单独的值进行序列化是时stringify将它们序列化为undefined

#### 特性二

非数组对象的属性不能保证以特定顺序出现在序列化后的字符串中

```javascript
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  fn: function() {
    return true;
  },
  d: "ddd"
};
JSON.stringify(data); // 输出：？
// "{"a":"aaa","d":"ddd"}"

JSON.stringify(["aaa", undefined, function aa() {
    return true
  }, Symbol('dd'),"eee"])  // 输出：？

// "["aaa",null,null,null,"eee"]"
```

因为stringify序列化时会忽略一些特殊的值，所以不能保证序列化后的字符串以特定顺序出现

#### 特性三

转换值如果有toJSON函数，该函数返回什么值，序列化的结果就是什么值，并且忽略其他属性值

```javascript
JSON.stringify({
    say: "hello JSON.stringify",
    toJSON: function() {
      return "today i learn";
    }
  })
// "today i learn"
```

#### 特性四

JSON.stringify会正常序列化Date类型的值

```javascript
JSON.stringify({ now: new Date() });
// "{"now":"2019-12-08T07:42:11.973Z"}"
```

#### 特性五

NaN和Infinity格式的数值及null都会被当作null

```javascript
JSON.stringify(NaN)
// "null"
JSON.stringify(null)
// "null"
JSON.stringify(Infinity)
// "null"
```

#### 特性六

布尔值、数字、字符串的包装对象在序列化过程中会自动转为对应的原始值

```javascript
JSON.stringify([new Number(1), new String("false"), new Boolean(false)]);
// "[1,"false",false]"
```

#### 特性七

其他类型的对象，例如Map/Set/WeakMap/WeakSet，只会序列化可枚举的属性

```javascript
// 不可枚举的属性默认会被忽略：
JSON.stringify( 
    Object.create(
        null, 
        { 
            x: { value: 'json', enumerable: false }, 
            y: { value: 'stringify', enumerable: true } 
        }
    )
);
// "{"y":"stringify"}"
```

#### 特性八

我们经常会用到stringify实现深拷贝，但是对于循环引用问题，stringify无法处理从而报错

```javascript
// 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。 
const obj = {
  name: "loopObj"
};
const loopObj = {
  obj
};
// 对象之间形成循环引用，形成闭环
obj.loopObj = loopObj;

// 封装一个深拷贝的函数
function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj));
}
// 执行深拷贝，抛出错误
deepClone(obj)
/**
 VM44:9 Uncaught TypeError: Converting circular structure to JSON
    --> starting at object with constructor 'Object'
    |     property 'loopObj' -> object with constructor 'Object'
    --- property 'obj' closes the circle
    at JSON.stringify (<anonymous>)
    at deepClone (<anonymous>:9:26)
    at <anonymous>:11:13
 */
```

#### 特性九

所有以Symbol为key的属性都会被忽略掉，即使repalcer参数中强制指定包含它们

```javascript
JSON.stringify({ [Symbol.for("json")]: "stringify" }, function(k, v) {
    if (typeof k === "symbol") {
      return v;
    }
  })

// undefined
```

replacer是stringify的第二个参数

#### 第二个参数replacer

replacer参数有两种形式，可以是一个函数或数组。作为函数时，有两个参数key和value，函数类似于map和filter等数组方法的回调函数，对每一个属性值都执行一次该函数。如果replacer是一个数组，数组的值将被序列化为JSON字符串的属性名

##### 打破九大特性的大多数特性

```javascript
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  fn: function() {
    return true;
  }
};
// 不用 replacer 参数时
JSON.stringify(data); 

// "{"a":"aaa"}"
// 使用 replacer 参数作为函数时
JSON.stringify(data, (key, value) => {
  switch (true) {
    case typeof value === "undefined":
      return "undefined";
    case typeof value === "symbol":
      return value.toString();
    case typeof value === "function":
      return value.toString();
    default:
      break;
  }
  return value;
})
// "{"a":"aaa","b":"undefined","c":"Symbol(dd)","fn":"function() {\n    return true;\n  }"}"
```

replacer被传入函数时，第一个参数不是对象的第一个键值对，而是空字符串作为key值，value值是整个对象的键值对

```javascript
const data = {
  a: 2,
  b: 3,
  c: 4,
  d: 5
};
JSON.stringify(data, (key, value) => {
  console.log(value);
  return value;
})
// 第一个被传入 replacer 函数的是 {"":{a: 2, b: 3, c: 4, d: 5}}
// {a: 2, b: 3, c: 4, d: 5}   
// 2
// 3
// 4
// 5
```

##### 实现对象的map函数

```javascript
// 实现一个 map 函数
const data = {
  a: 2,
  b: 3,
  c: 4,
  d: 5
};
const objMap = (obj, fn) => {
  if (typeof fn !== "function") {
    throw new TypeError(`${fn} is not a function !`);
  }
  return JSON.parse(JSON.stringify(obj, fn));
};
objMap(data, (key, value) => {
  if (value % 2 === 0) {
    return value / 2;
  }
  return value;
});
// {a: 1, b: 3, c: 2, d: 5}
```

replacer作为数组时，数组的值就代表了将被序列化为JSON字符串的属性名

```javascript
const jsonObj = {
  name: "JSON.stringify",
  params: "obj,replacer,space"
};

// 只保留 params 属性的值
JSON.stringify(jsonObj, ["params"]);
// "{"params":"obj,replacer,space"}" 
```

#### 第三个参数space

space参数用来控制结果字符串里的间距

```javascript
const tiedan = {
  name: "弹铁蛋同学",
  describe: "今天在学 JSON.stringify()",
  emotion: "like shit"
};
JSON.stringify(tiedan, null, "🐷");
// 接下来是输出结果
// "{
// 🐷"name": "弹铁蛋同学",
// 🐷"describe": "今天在学 JSON.stringify()",
// 🐷"emotion": "like shit"
// }"
JSON.stringify(tiedan, null, 2);
// "{
//   "name": "弹铁蛋同学",
//   "describe": "今天在学 JSON.stringify()",
//   "emotion": "like shit"
// }"
```

+ space如果是数字，会在序列化时，每一级别比上一级别多这个数字的空格数量，最多10个空格
+ space如果是字符串，每一级别比上一级别多缩进该字符串或该字符串的前10个字符

### null和undefined的区别

首先，null和undefined都是基本数据类型，即使typeof不这么认为

```javascript
console.log(typeof null) // 'object'
console.log(typeof undefined) // 'undefined'
```

#### undefined

undefined：按照字面上意思就是未定义的值，什么情况变量会未定义

```javascript
/*
	* 声明变量但不赋值
*/
var foo;
console.log(foo);

/*
	* 访问对象上不存在的属性或者未定义的变量
*/
console.log(Object.foo);
console.log(typeof demo);

/*
	* 函数定义了形参，但是没有传递实参
*/
function fn(a){
    console.log(a);
}
fn();

/*
	* 使用void对表达式求值
*/
console.log(void 0);
console.log(void false);
console.log(void []);
console.log(void null);
console.log(void function fn(){});
```

ES明确规定void操作符对任何表达式求值都返回undefined，这和函数执行操作后没有返回值作用相同

Js中的函数执行后都会有返回值，如果没有return或return;，默认返回undefined

#### null

空值，表示对象被人为的置为空对象，而非undefined，注意，null值在栈中的变量不会指向堆中的内存对象，因为它没有

+ 将来会用来 保存对象的变量，置为null，不希望被其他类型值占用
+ 当某个变量不再被需要时，设为null解除引用

#### typeof null

```javascript
console.log(typeof null) // 'object'
```

+ null表示一个空指针，代表的就是一个空对象，只不过在堆中没有内存

+ JavaScript中的值是由一个表示类型的标签和实际数据值决定的，因为null代表空指针，为0x00，

  null的类型标签为0，与对象的类型标签为0相同，会被识别为object

### 绑定this的方法

#### 引言

+ call、apply的实现和区别
+ bind依托于call或apply的实现，以及和它们的区别
+ 箭头函数绑定this

#### call

```javascript
Function.prototype.myCall=function(){
    let [context,...args]=[...arguments]
    if(typeof window!=='undefined')
        context=context||window;
    else
        context=context||global
    context.fn=this;
    context.fn(...args);
    delete context.fn;
};
```

#### apply

```javascript
Function.prototype.myApply=function(){
    let [context,...args]=[...arguments]
    if(typeof window!=='undefined')
        context=context||window;
    else
        context=context||global
    context.fn=this;
    context.fn(args);
    delete context.fn;
};
```

#### bind

```javascript
Function.prototype.myBind=function(){
    let [context,...args]=[...arguments]
    if(typeof window!=='undefined')
        context=context||window;
    else
        context=context||global
    const fn=this;
    return function(){
        fn.myApply(context,[...args,...arguments]);
    };
};
```

#### 箭头函数绑定this

箭头函数的this始终指向函数定义时的this，很明显于普通函数内部的this的指向不同（指向执行时的）

箭头函数中没有this绑定，必须通过查找作用域链来决定this，如果被非箭头函数包含，则this绑定最近一层非箭头函数的this

```javascript
var name = "windowsName";

var a = {
  name: "Cherry",

  func1: function () {
    console.log(this.name);
  },

  func2: function () {
    setTimeout(() => {
      this.func1();
    }, 100);
  },
};

a.func2(); // Cherry
```

箭头函数的这种行为与我们寻常了解到的setTimeout回调函数内部的this完全不同，寻常的setTimeout回调函数内部的this都会指向window或global，但是箭头函数因为在定义时指定this，它会捕获func2函数内部的this

#### 在函数内部使用_this或that

利用_this或that可以很轻松的固定this，这一点在Axios等异步请求中非常有用，因为异步请求中有大量的回调，甚至是以前的回调地狱问题

```javascript
var name = "windowsName";

var a = {
  name: "Cherry",

  func1: function () {
    console.log(this.name);
  },

  func2: function () {
    // _tgi
    var _this = this;
    setTimeout(function () {
      _this.func1();
    }, 100);
  },
};

a.func2(); // Cherry
```

#### 区别

+ call和apply只是传入的参数形式不同，call是参数列表，而apply是数组

+ call、apply和bind不同的是，前者返回执行的结果，而bind返回一个绑定了this的函数，不过需要说明的是，返回的函数仍然可以被绑定this到其他对象上

#### 测试

```javascript
const obj={
    name: 'zzg',
    age: 20
};

const print=function(){
    console.log(this.name,this.age);
}

global.name='wax';
global.age=20;

Function.prototype.myCall=function(){
    let [context,...args]=[...arguments]
    if(typeof window!=='undefined')
        context=context||window;
    else
        context=context||global
    context.fn=this;
    context.fn(...args);
    delete context.fn;
};

Function.prototype.myApply=function(){
    let [context,...args]=[...arguments]
    if(typeof window!=='undefined')
        context=context||window;
    else
        context=context||global
    context.fn=this;
    context.fn(args);
    delete context.fn;
};

Function.prototype.myBind=function(){
    let [context,...args]=[...arguments]
    if(typeof window!=='undefined')
        context=context||window;
    else
        context=context||global
    const fn=this;
    return function(){
        fn.myApply(context,[...args,...arguments]);
    };
};

const myPrint=print.myBind(obj);
print.myCall(null);
```

### new新建实例的原理

#### 四个步骤及实现

> 联想原型和原型链

+ 创建空对象
+ 链接原型链
+ 执行构造函数，注意改变this指向（call、apply）
+ 返回结果（优先返回构造函数返回的对象，次之返回obj）

```javascript
const myNew=function(){
    const obj={};
    const [constructor,...args]=[...arguments];
    obj.__proto__=constructor.prototype;
    const result=constructor.apply(obj,args);
    console.log(obj)
    return typeof result==='object' ? result : obj;
}
```

#### 如何函数确定是否被new了

> new.target属性可以确定

##### 必须实例化的构造函数

```javascript
const Person=function(name,age){
    if(!new.target) throw new Error('该构造函数必须new');
    this.name=name;
    this.age=age;
};
```

##### 不能独立使用，必须继承才能使用

```javascript
class Parent {
    constructor() {
        if (new.target) throw Error('该构造函数不能被实例化。')
    }
}

class Son extends Parent {
    constructor(name) {
        super()
        this.name = name
    }
}
```

#### 测试代码

```javascript
const Person=function(name,age){
    this.name=name;
    this.age=age;
};

const myNew=function(){
    const obj={};
    const [constructor,...args]=[...arguments];
    obj.__proto__=constructor.prototype;
    const result=constructor.apply(obj,args);
    console.log(obj)
    return typeof result==='object' ? result : obj;
}

console.log(myNew(Person,'zzg',20));
```

### Function.length

length属性表示函数期待的参数数量，但是只包括第一个具有默认值的参数之前的参数，且不计入rest参数

+ Writable: false，表示不可写
+ Enumerable: false，不可枚举
+ Configurable: true，可配置

````javascript
console.log(Function.length); // 1

console.log((function()        {}).length); /* 0 */
console.log((function(a)       {}).length); /* 1 */
console.log((function(a, b)    {}).length); /* 2 */

console.log((function(...args) {}).length);
// 0, arguments对象只有在函数内部才能使用，用来表示实际传给函数的参数

console.log((function(a, b = 1, c) {}).length);// 1
// 只有a会被计算，b因为有默认值，后面的所有参数不会记入
````

### ES继承

#### 原型链继承

将子类构造函数的原型对象链接到父类的某个实例上

##### 缺点

+ 每个实例对引用类型的修改都会被其他实例共享
+ 在创建Child实例时，无法向Parent穿餐，Child实例没法自定义自己的属性

```javascript
function Parent() {
  this.name = "arzh";
}

Parent.prototype.getName = function () {
  console.log(this.name);
};

function Child() {}

//主要精髓所在
Child.prototype = new Parent();
Child.prototype.constructor = Child;

var arzhChild = new Child();

arzhChild.getName(); // 'arzh'
```

#### 借用构造函数继承

在子类的构造函数中调用父类构造函数

##### 优点

+ 解决引用类型属性的问题
+ 子类可以向父类传参

##### 缺点

+ 无法复用父类的公共函数

  因为只是相当于把子类构造函数中的this放到父类中跑了一遍，只能获取通过this定义的属性，不能获取父类父类原型上的方法

+ 每次子类都早实例都会执行一次父类的函数

```javascript
function Parent() {
  this.names = ["arzh", "arzh1"];
}

function Child() {
  Parent.call(this);
}

var arzhChild2 = new Child();
arzhChild2.names.push("arzh2");
console.log(arzhChild2.names); //[ 'arzh', 'arzh1', 'arzh2' ]

var arzhChild3 = new Child();
arzhChild3.names.push("arzh3");
console.log(arzhChild3.names); //[ 'arzh', 'arzh1', 'arzh3' ]
```

#### 组合继承

结合原型式继承和构造函数继承

##### 优点

+ 解决引用类型数据问题
+ 子类可以向父类传参
+ 可以实现父类方法复用

##### 缺点

+ 会执行两次父类构造函数

```javascript
function Parent(name) {
  this.name = name;
  this.body = ["foot", "hand"];
}

function Child(name, age) {
  Parent.call(this, name);
  this.age = age;
}

Child.prototype = new Parent();
Child.prototype.constructor = Child;

var arzhChild1 = new Child("arzh1", "18");
arzhChild1.body.push("head1");
console.log(arzhChild1.name, arzhChild1.age); //arzh1 18
console.log(arzhChild1.body); //[ 'foot', 'hand', 'head1' ]

var arzhChild2 = new Child("arzh2", "20");
arzhChild2.body.push("head2");
console.log(arzhChild2.name, arzhChild2.age); //arzh2 20
console.log(arzhChild2.body); //[ 'foot', 'hand', 'head2' ]
```

#### 原型式继承

把传入的对象链接到创建对象的原型上，实现继承，类似于原型链继承

##### 缺点

+ 对于引用类型的数据会共享

```javascript
function createObj(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
var person = {
  name: "arzh",
  body: ["foot", "hand"],
};
var person1 = createObj(person);
var person2 = createObj(person);

console.log(person1); //arzh
person1.body.push("head");
console.log(person2); //[ 'foot', 'hand', 'head' ]
```

#### 寄生式继承

> 

```javascript
function createEnhanceObj(o) {
  //代替原型式继承的createObj
  var clone = Object.create(o);
  clone.getName = function () {
    console.log("arzh");
  };
  return clone;
}
```

#### 寄生组合式继承

在构造函数继承的基础上，将子类构造函数链接到父类构造函数通过Object.create构造出的新对象上

```javascript
function Parent(name) {
  this.name = name;
}

Parent.prototype.getName = function () {
  console.log(this.name);
};

function Child(color) {
  Parent.call(this, "arzh");
  this.color = color;
}

Child.prototype = Object.create(Parent.prototype); //创建父类原型的一个副本,把副本赋值给子类原型
Child.prototype.constructor = Child;

var arzhChild = new Child("red");
console.log(arzhChild.name); // 'arzh'
```

### CommonJS

#### require

Nodejs遵循CommonJS规范，使用require关键字加载模块

#### require重复引入的问题

在使用require引入模块时，多个代码文件多次引入相同的模块会不会造成重复？

Nodejs中默认先从缓存中加载模块，一个模块被加载一次之后，就会在缓存中维持一个副本，如果遇到重复加载的模块会直接提取缓存中的副本，在任何时候每个模块只在缓存中有一个实例

#### require同步加载模块

同步

+ 一个作为公共依赖的模块，最好一次性加载出来，同步更好
+ 模块的个数是有限的，而且Nodejs在require时会自动缓存已经加载的模块，再加上访问的都是本地文件，IO开销可以忽略

#### require的缓存策略

Nodejs会自动缓存require引入的文件，下次再引入不需要经过文件系统而是直接从缓存读取

不过这种缓存式经过文件路径定位的，即使两个完全相同的文件，位于不同路径下，也会在缓存中维持两份

```javascript
// 可以通过以下获取缓存中的文件
console.log(require.cache)
```

#### exports与module.exports的区别

##### js文件启动时

node执行一个文件时，会给这个文件生成一个exports和module对象，module有一个exports属性

```javascript
exports = module.exports = {};
```

##### require加载模块

```javascript
//koala.js
let a = '程序员成长指北';

console.log(module.exports); //能打印出结果为：{}
console.log(exports); //能打印出结果为：{}

exports.a = '程序员成长指北哦哦'; //这里辛苦劳作帮 module.exports 的内容给改成 {a : '程序员成长指北哦哦'}

exports = '指向其他内存区'; //这里把exports的指向指走

//test.js

const a = require('/koala');
console.log(a) // 打印为 {a : '程序员成长指北哦哦'}
```

require导出的内容是module.exports指向的内存，不是exports的

#### 实际例子

```javascript
// ./a.js
let count = 1;

setCount = () => {
    count++;
}

setTimeout(() => {
    console.log('a', count)// a 2
}, 1000);

module.exports = {
    count,
    setCount
}

//b.js
const obj = require('./a.js');

obj.setCount();// count=2

console.log('b', obj.count)// b 1

setTimeout(() => {
    console.log('b next', obj.count);
}, 2000);
```

+ 运行b模块时，先加载a模块，会执行setTimeout，到时间后把回调函数扔进宏任务队列

+ 调用setCount，这里注意，setCount内部改变的是a模块内部的count变量，因为count是普通类型，导出的是count的拷贝，因此即使将count增1，输出仍为1

+ 执行setTimeout，扔进队列

+ 执行a模块中的回调，输出count为2，因为setCount被调用了

+ 执行b模块中的回调，输出count为1，没有发生改变

### 垃圾回收GC

#### GC

GC即Garbage Collection，程序在运行过程中会产生很多垃圾，这些垃圾是程序不用的内存或者以前用过的以后不会再使用的空间，GC就是负责回收产生的垃圾的，它工作在引擎（如V8）内部。

#### 垃圾的产生以及如何回收

JavaScript的引用数据类型是保存在堆内存中的，然后再栈内存中保存一个对堆内存的实际对象的引用。所以JavaScript中对引用数据类型的操作都是操作对象的应用而不是实际的对象。

```javascript
let test = {
  name: "isboyjc"
};
test = [1,2,3,4,5]
```

在上面我们声明了一个变量test，引用了一个对象，接着我们为test变量重新赋值了一个数组对象，那么之前都是对象引用关系就被丢弃了

没有了引用关系，也就是无用的对象，需要被回收

#### 垃圾回收策略

在JavaScript内存管理中有一个概念叫可达性，就是那些以某种方式可访问或可用的值被存储在内存中，反之不可访问则需回收

所以我们需要做的就是发现这些不可达的对象并清理。常见的回收算法有：

+ 标记清除
+ 引用计数

#### 标记清除

最常用的垃圾回收算法，目前大多数的JavaSCript引擎都在使用。

##### 标记阶段

从出发点遍历内存中的所有对象去打标记，这个出发点可以是全局的Window对象、文档树DOM等根对象

+ 垃圾收集器在运行时给内存中所有遍历加上标记，假设内存中所有对象都是垃圾，全标记为0
+ 然后从各个根对象开始遍历，把不是垃圾的节点标记为1
+ 清理所有标记为0的垃圾，销毁并回收占用的内存空间
+ 把所有内存中对象标记修改为0，等待下一轮垃圾回收

##### 优点

+ 实现简单，只需要一个二进制位就可以标记

##### 缺点

+ 在清除之后，剩余的内存位置不变，导致空闲内存空间不连续，出现内存碎片，出现内存分配的问题
+ 内存分配速度慢，即使是First-fit策略，操作仍然是O(n)操作，最坏情况可能每次都要遍历到最后，同时因为对片花，大对象的分配效率会更慢

##### 内存分配

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12247ac3d8f249a5ab85b9b40ba1147b~tplv-k3u1fbpfcp-watermark.awebp)

假设我们新建对象内存时需要大小为size，由于空闲内存是间断的不连续的，就需要对空闲内存列表进行一次单向遍历找出大于等于size的块才能为其分配

+ First-fit：找到大于等于size的块立即返回
+ Best-fit：遍历整个空闲列表，返回大于等于size的最小分块
+ Worst-fit：遍历整个空闲列表，找到最大的分块，切成两部分，一部分为size大小并返回

三种策略中，Worst-fit空间利用率看起来更合理，但实际上切分之后会造成更多的小块，形成内存碎片，所以不推荐使用

对于First-fit和Best-fit来说，考虑到分配的速度和效率，First-fit更快。

##### 标记整理算法

在标记结束后，将活着的对象向内存的一端移动，清理掉边界内存

#### 引用计数算法

把对象是否不再需要定义为对象有没有被其它对象引用，如果没有引用指向该对象，对象将被GC回收

+ 当声明了一个变量并将一个引用类型赋值给该变量时，这个值的引用次数加1
+ 如果同一个值被赋给另一个变量，引用加1
+ 如果该变量的值被其它值覆盖，引用减1
+ 当值的引用次数变为0时，说明没有变量在使用，值不能被访问，回收该空间，GC会在运行时清理掉引用次数为0的值占用的内存

```javascript
function test(){
  let A = new Object()
  let B = new Object()
  
  A.b = B
  B.a = A
}
```

##### 循环引用的问题

对象A和B各自的属性相互引用，按照引用计数的策略，引用数量都是2，但是在test函数在执行完后，对象A和B都应被回收

如果用标记清除的思想来看，当函数结束后，两个对象都不在作用域中，A和B都会被当作非活动对象清除，但是引用计数就不会释放掉，造成大量无用内存占用。

> 在 IE8 以及更早版本的 IE 中，`BOM` 和 `DOM` 对象并非是原生 `JavaScript` 对象，它是由 `C++` 实现的 `组件对象模型对象（COM，Component Object Model）`，而 `COM` 对象使用 引用计数算法来实现垃圾回收，所以即使浏览器使用的是标记清除算法，只要涉及到 ` COM` 对象的循环引用，就还是无法被回收掉，就比如两个互相引用的 `DOM` 对象等等，而想要解决循环引用，需要将引用地址置为 `null` 来切断变量与之前引用值的关系，如下

```javascript
// COM对象
let ele = document.getElementById("xxx")
let obj = new Object()

// 造成循环引用
obj.ele = ele
ele.obj = obj

// 切断引用关系
obj.ele = null
ele.obj = null
```

在IE9以及以后的BOM和DOM对象都改成了JS对象，避免了循环引用的问题

##### 优点

+ 可以在引用计数为0时立即回收垃圾
+ 标记清除算法需要每隔一段时间进行一次，在JS脚本中线程比阿虚暂停一段时间执行GC
+ 标记清除需要遍历堆里的活动以及非活动对象来清除，而引用计数只需要在引用时计数

##### 缺点

+ 需要一个计数器，而且我们无法预估引用数量的上限
+ 无法解决循环引用问题

#### V8对GC的优化

https://juejin.cn/post/6981588276356317214

### 函数声明

#### 表达式声明

```javascript
var a=function(){
  
}
```

#### 直接声明

```javascript
function a(){
  
}
```

#### 差别

+ var虽然会对变量a进行提升，但是并不会提升表达式，实际提升的部分只有var a;，因此此时a为undefined

  初次之外，在非严格模式下，var声明的变量会被挂到window对象上，因此window.a===a

+ 非严格模式下，直接声明的函数也会被挂到window对象上

```javascript
a();// a is not a function
var a = function () {};

window.a===a;// true

a();// 不会报错
function a(){
  
}
window.a===a;// true
```

#### JavaScript预解析

JavaScript解析过程分为两个阶段，分别是编译阶段和执行阶段

##### 编译阶段

编译阶段就是JavaScript的预解析阶段，解释器会将JavaScript脚本转为字节码

**变量提升**

创建一个当前执行环境下的活动对象AO，将var声明的变量挂到AO上并赋值为undefined，同时将function定义的函数也添加到AO中

```javascript
if(true){
  var a=1;
  var b=2;
}

function A1(){

}

function B1(){
  
}
```

**函数声明与函数表达式在预解析阶段的区别**

解析器会对function定义的函数在代码开始执行之前对实行函数进行变量提升，所以函数声明前调用函数不会在执行期间报错，

但是函数表达式使用var声明，解析器会对其进行变量提升，并赋值为undefined，然后执行期间，将变量指向一个function，因此在指向之前调用会报错

**function覆盖**

若定义两个同名函数，在预解析期间后面会覆盖前面

```javascript
AA();   // 输出 I am AA_2;
function AA(){
   console.log('I am AA_1');
};

AA();  // 输出 I am AA_2;
function AA(){
  console.log('I am AA_2');
}
```

**预解析把变量或函数解析道其运行时对环境中**

解析器将变量提升到执行环境对应的AO中，并不一定是window

**预解析分段进行**

按照script标签分块预解析

##### 执行阶段

借助执行环境把字节码转为机器码，并按顺序执行

#### Q&A

```javascript
AA(); // 输出 AA_2

function AA(){
  console.log(' AA_2 ');
}

var AA = function(){
  console.log(' AA_1 ');
}

AA(); //输出 AA_1
```

+ 首先，会将var的声明和function声明向上提，因此此时AA指向AA_2

+ 然后开始向下执行，调用输出AA_2

+ 然后找到表达式声明，指向AA_1，再次调用，输出AA_1

##### 变量提升和函数提升的优先级

在JavaScript中，函数是一等公民，函数声明的优先级最高，会被提升至当前作用域顶端，而var声明对a会被函数声明所覆盖

```javascript
console.log(typeof a);
var a=5;
console.log(a);

function a(){
    
}
```

+ 函数a会被先提升上去，所以第一次应该输出函数a也就是function
+ 然后对变量a进行赋值操作，得到5

### 判断数组

#### typeof

只能判断Function以及基本数据类型

#### instanceof

```javascript
arr instanceof Array
```

#### 原型链

一般情况下，除了undefined和null，都能使用constructor判断类型

但是，会出现问题，因为原型对象prototype是可以手动修改的

```javascript
> arr.___proto__.constructor===Array
>
> arr.constructor===Array
```

#### Array.isArray

原理也是通过Object.prototype.toString判断对象的内部属性[[class]]是否为“Array”

```javascript
Array.isArray
```

#### Object.prototype.toString

```javascript
Object.prototype.toString.call(arr)==='[object Array]'
```

#### 使用instanceof和constructor的局限性

使用和声明都必须在当前页面，比如父页面引用了子页面，在子页面中声明了一个Array，将其赋值给父页面的一个变量，那么此时原型链的判断得到false

+ Array属于引用型数据，传递过程中只传递引用地址
+ 每个页面的Array原生对象所引用的地址不同，在子页面声明的Array所对应的构造函数是子页面的Array对象；父页面判断使用的Array不等于子页面的Array

### 浮点数精度丢失问题

#### 引出问题

```javascript
console.log(0.1+0.2);
> 0.30000000000000004
```

#### Number类型

回顾一下Js中的七大基本类型：Number、String、Boolean、undefined、null、Symblo、Bigint

可以看到js中的数字类型只有Number类型，也就是其他类似Java中的double双精度浮点型，不区分浮点型和整数型

Number的四种进制表示法：二进制、十进制、八进制、十六进制

Number类型使用**IEEE754**格式表示整数和浮点值

+ 二进制：0B或0b，后接1或0表示二进制数
+ 十进制数：默认0-9

#### IEEE754

64位二进制表示一个数字，64位=1位符号位+11位指数位+52位小数位

+ 符号位：表示正负，0为正，1为负，-1^0=1，-1^1=-1
+ 指数位：用科学计数法表示数值大小一般都是二进制的，表示2^n
+ 小数位：科学计数法前面的值，默认所有数值都转为1.xxxx格式，可以省略一位小数位，存储更多数字内容，但是会丢失精度

#### 精度丢失

本质：将浮点数转换为IEEE754标准的二进制过程中出现丢失

小数转换为二进制：乘二取整法

```
0.1*2=0.2======取出整数部分0

0.2*2=0.4======取出整数部分0

0.4*2=0.8======取出整数部分0

0.8*2=1.6======取出整数部分1

0.6*2=1.2======取出整数部分1

0.2*2=0.4======取出整数部分0

0.4*2=0.8======取出整数部分0

0.8*2=1.6======取出整数部分1

0.6*2=1.2======取出整数部分1

接下来会无限循环

0.2*2=0.4======取出整数部分0

0.4*2=0.8======取出整数部分0

0.8*2=1.6======取出整数部分1

0.6*2=1.2======取出整数部分1

所以0.1转化成二进制是：0.0001 1001 1001 1001…（无限循环）

0.1 => 0.0001 1001 1001 1001…（无限循环）
```

#### 解决方法

+ 考虑到每次浮点数运算的偏差较小，可以对结果进行指定精度的四舍五入

  ```
  parseFloat(result.toFixed(12));
  ```

+ 将浮点数转为整数运算，再做除法，可能会溢出

+ 将浮点数转为字符串，模拟实际运算过程

### 箭头函数和普通函数的区别

#### this指向问题

+ 箭头函数没有原型prototype，所遇你本身没有this

+ this指向在定义的时候继承外层第一个普通函数的tihs，与执行时的位置无关

   被继承的普通函数的this指向改变，箭头函数的this指向会跟着改变

+ 不能直接修改箭头函数的this指向

  不能用call、apply或bind修改箭头函数的this指向，但是可以通过修改外层普通函数的this间接修改

+ 箭头函数外层如果没有普通函数，在严格模式和非严格模式下this都会指向全局对象window或global

```javascript
/** 第1条 */
let a = () => {
  console.log(this.b);
};
console.log(a.prototype);// undefined

/** 第2条 */
function bar() {
  let a = () => {
    console.log(this);
  };
  a();
}

bar(); // global/window
bar.call({ a: 1 }); // { a: 1 }

/** 第3条 */
a.call({ b: 1 }); // undefined

/** 第4条 */

let b = () => {
  console.log(this);
};

b();// {}

```

#### 箭头函数的arguments

+ 如果箭头函数外层没有普通函数，使用arguments会报错
+ 如果有普通函数，箭头函数的arguments会继承普通函数的arguments

```javascript
function bar() {
  console.log(arguments); // ['外层第二个普通函数的参数']
  bb("外层第一个普通函数的参数");
  function bb() {
    console.log(arguments); // ["外层第一个普通函数的参数"]
    let a = () => {
      console.log(arguments, "arguments继承this指向的那个普通函数"); // ["外层第一个普通函数的参数"]
    };
    a("箭头函数的参数"); // this指向bb
  }
}
bar("外层第二个普通函数的参数");
```

##### rest获取函数不定参数

+ rest参数可以用在箭头函数和普通函数中
+ 更加灵活
+ 可读性更好，不会突然出现arguments
+ rest是一个真正的数组，可以使用数组的API。arguments只是一个类数组的对象，需要用Array.from转为数组或用扩展运算符

**注意**

+ rest必须是函数的最后一位参数
+ 函数的length属性不包括rest参数

```javascript
let a = (first, ...abc) => {
  // arguments.push(0); // arguments.push is not a function
  arguments = [...arguments];
  // arguments = Array.from(arguments);
  console.log(first, abc); // 1 [2, 3, 4]
};
a(1, 2, 3, 4);

let b = (first, ...rest, three) => {
    console.log(first, rest,three); // 报错：Rest parameter must be last formal parameter
};
b(1, 2, 3, 4);

// (function(...a) {}).length  // 0
(function(a, ...b) {}).length  // 1
```

#### new调用箭头函数会报错

无论箭头函数的this指向哪里，使用new调用箭头函数都会报错，因为箭头函数不能作为constructor，也没有prototype

```javascript
let a = () => {};
let b = new  a(); // a is not a constructor
```

**箭头函数不支持new.target**

new.target是ES6新引入的属性，普通函数如果通过new调用，new.target会返回该函数的引用，可以确定构造函数是否被new调用

+ 箭头函数this指向全局对象时，内部的new.target会报错
+ 箭头函数的this指向全局函数，new.target指向该普通函数的引用

```javascript
let a = () => {
  // console.log(new.target); // 报错：new.target 不允许在这里使用
};
// a();

new bb();
function bb() {
  let a = () => {
    console.log(new.target); // 指向函数bb：function bb(){...}
  };
  a();
}
```

#### 箭头函数的参数不能有相同

```javascript
function func1(a, a) {
  console.log(a, arguments); // 2 [1,2]
}

var func2 = (a, a) => {
  console.log(a); // 报错：在此上下文中不允许重复参数名称
};
func1(1, 2);
func2(1, 2);
```

#### 注意事项

+ 如果返回对象字面量，需要加括号，或用return返回，不能直接返回对象字面量，因为花括号会被解析为语句块而不是对象

### JavaScript中的隐式转换

先来看一道题

```javascript
const a = {
  i: 1,
  toString: function () {
    return a.i++;
  }
}
if (a == 1 && a == 2 && a == 3) {
  console.log('hello world!');// 'hello world'
}
```

为什么会出现上面的结果，明明并没有对a进行赋值，a却可以同时等于1、2、3

#### 数据类型

+ 基本数据类型
  + undefined
  + null
  + string
  + number
  + boolean
  + symbol
  + bigint
+ 复杂数据类型object
  + Function
  + Array
  + RegExp
  + Date
  + 等

#### 三种隐式转换类型

Js中出现隐式转换最多的运算符是+和==

+ +因为既可以字符串想加，也可以数字相加，例如0+'1' ='01'
+ ==与===不同，它会在两边类型不同时，隐式的转换两边的数据
+ -*/这些运算符，因为只能针对number类型运算，所以所有数据的转换的结果都朝着number

##### 三种隐式转换

+ 值转为原始值，ToPrimitive
+ 值转为数字，ToNumber
+ 值转为字符串，ToString

#### ToPrimitive转为原始值

```
ToPrimitive(input, PreferredType?)
```

其中input是要转换的值，PreferredType是可选参数，既可以是number也可以是string，它只是一个转换标志，转换后的结果不一定是参数所指的类型，但转换结果一定是原始值或报错

##### 标记为number

1. 如果输入的值已经是一个原始值，则直接返回它
2. 否则，如果输入的值是一个对象，则调用该对象的valueOf()方法，
   如果valueOf()方法的返回值是一个原始值，则返回这个原始值。
3. 否则，调用这个对象的toString()方法，如果toString()方法返回的是一个原始值，则返回这个原始值。
4. 否则，抛出TypeError异常。

##### 标记为string

1. 如果输入的值已经是一个原始值，则直接返回它
2. 否则，调用这个对象的toString()方法，如果toString()方法返回的是一个原始值，则返回这个原始值。
3. 否则，如果输入的值是一个对象，则调用该对象的valueOf()方法，
   如果valueOf()方法的返回值是一个原始值，则返回这个原始值。
4. 否则，抛出TypeError异常。

##### 默认表示

因为PreferredType是可选参数，而我们日常的使用一般也不会使用到，因此它有自己的默认规则：

+ 对象为Date类型，PreferredType被设置为String

  返回一个GMT格式的字符串

```javascript
console.log(new Date().toString());
// Tue Aug 24 2021 09:35:32 GMT+0800 (China Standard Time) 
```

+ 否则被设置为number

#### valueOf和toString方法

对象类型的数据一定存在valueOf方法和toString方法，因为它们被定义在原型链的最顶层Object.prototype（null除外）

##### valueOf

1. number、boolean、string三种构造函数生成的基础值的对象形式，通过valueOf会变成对应的原始值

```javascript
var num = new Number('123');
num.valueOf(); // 123

var str = new String('12df');
str.valueOf(); // '12df'

var bool = new Boolean('fd');
bool.valueOf(); // true
```

2. 对于Date类型，valueOf返回毫秒时间戳

```javascript
var a = new Date();
console.log(a.valueOf()); // 1629769189180
```

3. 除此之外，valueOf方法都会返回对象本身

```javascript
var a = new Array();
console.log(a.valueOf() === a); // true

var b = new Object({});
console.log(b.valueOf() === b); // true
```

##### toString

对于js常见的内置对象：Date、Array、Math、Number、Boolean、String、RegExp、Function

这些对象都封装了自己的toString方法

```javascript
Number.prototype.hasOwnProperty('toString'); // true
Boolean.prototype.hasOwnProperty('toString'); // true
String.prototype.hasOwnProperty('toString'); // true
Array.prototype.hasOwnProperty('toString'); // true
Date.prototype.hasOwnProperty('toString'); // true
RegExp.prototype.hasOwnProperty('toString'); // true
Function.prototype.hasOwnProperty('toString'); // true

var num = new Number('123sd');
num.toString(); // 'NaN'

var str = new String('12df');
str.toString(); // '12df'

var bool = new Boolean('fd');
bool.toString(); // 'true'

var arr = new Array(1,2);
arr.toString(); // '1,2'

var d = new Date();
d.toString(); // "Wed Oct 11 2017 08:00:00 GMT+0800 (中国标准时间)"

var func = function () {}
func.toString(); // "function () {}"
```

除了上面的对象和通过Object实例化的对象外，其他对象都会返回该对象的类型

```javascript
var obj = new Object({});
obj.toString(); // "[object Object]"

Math.toString(); // "[object Math]"
```

上面的现象说明了，我们在判断对象的时候为什么不直接用对象本身的toString方法，而是要用Object.prototype.toString.call去调用，因为对象本身的toString方法很容易被重写

#### ToNumber转为数字

+ undefined——》NaN
+ null——〉+0
+ 布尔值：true转1，false转0
+ 字符串：将字符串解析为数字，空字符串转为0
+ 对象：先进行ToPrimitive得到原始值，再进行ToNumber转为数字

#### ToString转为字符串

+ undefined：'undefined'
+ null：'null'
+ 布尔值：转'true'或'false'
+ 数字：直接转
+ 对象：先进行ToPrimitive转为得到原始值，再进行toString转字符串

##### 例子

```javascript
({} + {}) = ?
  /**
两个对象的值进行+运算符，肯定要先进行隐式转换为原始类型才能进行计算。
1、进行ToPrimitive转换，由于没有指定PreferredType类型，{}会使默认值为Number，进行ToPrimitive(input, Number)运算。
2、所以会执行valueOf方法，({}).valueOf(),返回的还是{}对象，不是原始值。
3、继续执行toString方法，({}).toString(),返回"[object Object]"，是原始值。
故得到最终的结果，"[object Object]" + "[object Object]" = "[object Object][object Object]"
*/
  
* {} = ?
  /**
1、首先*运算符只能对number类型进行运算，故第一步就是对{}进行ToNumber类型转换。
2、由于{}是对象类型，故先进行原始类型转换，ToPrimitive(input, Number)运算。
3、所以会执行valueOf方法，({}).valueOf(),返回的还是{}对象，不是原始值。
4、继续执行toString方法，({}).toString(),返回"[object Object]"，是原始值。
5、转换为原始值后再进行ToNumber运算，"[object Object]"就转换为NaN。
故最终的结果为 2 * NaN = NaN
*/
```

#### ==隐式转换

```javascript
比较运算 x==y, 其中 x 和 y 是值，返回 true 或者 false。这样的比较按如下方式进行：

1、若 Type(x) 与 Type(y) 相同， 则

    1* 若 Type(x) 为 Undefined， 返回 true。
    2* 若 Type(x) 为 Null， 返回 true。
    3* 若 Type(x) 为 Number， 则
  
        (1)、若 x 为 NaN， 返回 false。
        (2)、若 y 为 NaN， 返回 false。
        (3)、若 x 与 y 为相等数值， 返回 true。
        (4)、若 x 为 +0 且 y 为 −0， 返回 true。
        (5)、若 x 为 −0 且 y 为 +0， 返回 true。
        (6)、返回 false。
        
    4* 若 Type(x) 为 String, 则当 x 和 y 为完全相同的字符序列（长度相等且相同字符在相同位置）时返回 true。 否则， 返回 false。
    5* 若 Type(x) 为 Boolean, 当 x 和 y 为同为 true 或者同为 false 时返回 true。 否则， 返回 false。
    6*  当 x 和 y 为引用同一对象时返回 true。否则，返回 false。
  
2、若 x 为 null 且 y 为 undefined， 返回 true。
3、若 x 为 undefined 且 y 为 null， 返回 true。
4、若 Type(x) 为 Number 且 Type(y) 为 String，返回比较 x == ToNumber(y) 的结果。
5、若 Type(x) 为 String 且 Type(y) 为 Number，返回比较 ToNumber(x) == y 的结果。
6、若 Type(x) 为 Boolean， 返回比较 ToNumber(x) == y 的结果。
7、若 Type(y) 为 Boolean， 返回比较 x == ToNumber(y) 的结果。
8、若 Type(x) 为 String 或 Number，且 Type(y) 为 Object，返回比较 x == ToPrimitive(y) 的结果。
9、若 Type(x) 为 Object 且 Type(y) 为 String 或 Number， 返回比较 ToPrimitive(x) == y 的结果。
10、返回 false。
```

+ x和y类型相同时，无类型转换，注意NaN不与任何值相等即可，NaN!==NaN

+ 不相同时
  + x、y为number或string时，转换为number比较
  + 有boolean类型时，boolean转为number比较
  + 一个Object类型，一个string或number类型，将object类型转为原始值再比较

##### 例子

```javascript
var a = {
  valueOf: function () {
     return 1;
  },
  toString: function () {
     return '123'
  }
}
true == a // true;
首先，x与y类型不同，x为boolean类型，则进行ToNumber转换为1,为number类型。
接着，x为number，y为object类型，对y进行原始转换，ToPrimitive(a, ?),没有指定转换类型，默认number类型。
而后，ToPrimitive(a, Number)首先调用valueOf方法，返回1，得到原始类型1。
最后 1 == 1， 返回true。
```

```javascript
[] == !{}
//
1、! 运算符优先级高于==，故先进行！运算。
2、!{}运算结果为false，结果变成 [] == false比较。
3、根据上面第7条，等式右边y = ToNumber(false) = 0。结果变成 [] == 0。
4、按照上面第9条，比较变成ToPrimitive([]) == 0。
    按照上面规则进行原始值转换，[]会先调用valueOf函数，返回this。
   不是原始值，继续调用toString方法，x = [].toString() = ''。
   故结果为 '' == 0比较。
5、根据上面第5条，等式左边x = ToNumber('') = 0。
   所以结果变为： 0 == 0，返回true，比较结束。

```

```javascript
const a = {
  i: 1,
  toString: function () {
    return a.i++;
  }
}
if (a == 1 && a == 2 && a == 3) {
  console.log('hello world!');
}
```

```javascript
console.log(true + true); // 2
console.log(new Date() - new Date()); // 0
// 两个数组的引用地址不同
console.log([1] == [1]); // false
// 有时候{}会被识别为语句块而不是空对象
console.log({}.valueOf());
console.log(1 + {}); // 1[object object]
console.log({} + 1); // [objet object]1
```

### 强制类型转换

强转方式包括Number()、parseInt()、parseFloat()、toString()、String()、Boolean()

#### Number

+ 布尔值，true转换为1，false转换为0

+ 数字返回自身。

+ null返回0

+ undefined返回NaN

+ 如果是字符串
  + 如果字符串中只包含数字(或是0x/0X开头数字可以有正负)将其转换为十进制
  + 如果字符串中包含有效的浮点格式，转化为浮点数
  + 如果是空字符串，转换为0
  + 如果不是以上的，返回NaN

+ 如果是Symbol，抛出错误

+ 如果是对象，并且部署了[Symbol.toPrimitive]，调用对象的valueOf()方法，然后根据前面的规则转换返回的值，如果转换的结果是NaN,那么调用对象的toString方法，再次按前面的顺序返回对应的值。

```javascript
Number(true);        // 1
Number(false);       // 0
Number('0111');      //111
Number(null);        //0
Number('');          //0
Number('1a');        //NaN
Number(-0X11);       //-17
Number('0X11')       //17
```

#### Boolean类型

Undefined、null、false、''、0、NaN都被转为false

```javascript
Boolean(0)          //false
Boolean(null)       //false
Boolean(undefined)  //false
Boolean(NaN)        //false
Boolean(1)          //true
Boolean(13)         //true
Boolean('a')       //true
```

#### parseInt()

```javascript
parseInt("1234blue");   //returns   1234
parseInt("0xA");   //returns   10
parseInt("22.5");   //returns   22
parseInt("blue");   //returns   NaN
```

##### 设置进制

```javascript
parseInt("AF",   16);   //returns   175
parseInt("10",   2);   //returns   2
parseInt("10",   8);   //returns   8
parseInt("10",   10);   //returns   10
```

#### parseFloat

```javascript
parseFloat("1234blue");   //returns   1234.0
parseFloat("0xA");   //returns   NaN
parseFloat("22.5");   //returns   22.5
parseFloat("22.34.5");   //returns   22.34
parseFloat("0908");   //returns   908
parseFloat("blue");   //returns   NaN
```

#### 如何判定!x

所有的!(假值)都是true，假值都是基本数据类型，分别有''、false、0、null、undefined、NaN，其他引用数据类型例如[]或{}都开辟了内存空间，都是真值，所以![]和!{}都为假

```javascript
// 都为true
console.log(!'');
console.log(!null);
console.log(!undefined);
console.log(typeof NaN);// number
console.log(!NaN);
console.log(!0);
console.log(!false);
// 都为false
console.log(![]);
console.log(!{});
```

## 手撕型面试题

### flat平铺数组

#### toString实现

因为arr调用toString方法会删除所有中括号，最后得到的都是1,2,3,4,5的形式

```javascript
let flatString=function(arr){
    return arr.toString().split(',').map(val=>parseInt(val));
}
```

#### reduce实现

```javascript
let flatReduce=function(arr){
    return arr.reduce((val,cur)=>val.concat(Array.isArray(cur) ? flatReduce(cur) :cur),[]);
}

console.log(flatReduce([1,2,[3,[4]]]));
```

#### 递归

```javascript
let flatTraverse=function(arr){
    let result=[];
    arr.forEach(val=>{
        if(Array.isArray(val)){
            result.push(...flatTraverse(val));
        }else{
            result.push(val);
        }
    });
    return result;
}
```

#### 扩展运算符

```javascript
let flatExpand=function(arr){
    while(arr.some(val=>Array.isArray(val))){
      // concat连接的时候会逐个连接并自动展开数组
        arr=[].concat(...arr);
    }
    return arr;
}
```

### 防抖和节流

#### 引言

+ 要学会自己实现防抖和节流，深刻了解两者的使用场景
+ 谈到了防抖和节流，我们就不得不继续谈一下有关前端优化的部分，另开专题了解

#### 防抖

> 在事件被触发n秒后执行回调，如果在n秒内被再次触发，n秒清零重新计时

```javascript
/*
    * 防抖
    * @params {Function} fn
    * @params {Number} delay
*/
const debounce=function(fn,delay){
    let timer=null;
    return function(){
        let that=this;
        if(timer){
            clearTimeout(timer);
        }
        timer=setTimeout(()=>{
            // 这里要改变this指向
            fn.call(that,...arguments);
        },delay);
    }
}

const callback=function(){
    console.log(this.name);
}

global.name='wax';

const fn=debounce(callback,1000);
console.log(global.name);
fn('zzg');
```

#### 节流

> 在一个单位时间内，只能触发一次函数

```javascript
/*
    * 节流
    * @params {Function} fn
    * @params {Number} delay
*/
const throttle=function(fn,delay){
    let timer=null;
    return function(){
        if(timer)
            return;
        timer=setTimeout(()=>{
            fn(...arguments);
        },delay);
    };
};

const callback=function(num){
    console.log(num);
}

global.name='wax';

const fn=debounce(callback,1000);
fn(1);
fn(2);
fn(3);
```

#### 应用场景

##### 防抖

+ 输入框搜索的联想功能，用户在输入值的时候，用防抖来节约请求，其实节流也是可以的
+ window触发resize时，不断调整浏览器窗口大小会不断触发resize事件

##### 节流

+ 鼠标不断点击触发，mousedown
+ 监听滚动事件

### 深拷贝和浅拷贝

#### 深拷贝

##### JSON.parse和JSON.stringify

+ 不能复制正则、function、Symbol
+ 循环引用报错
+ 相同的引用会被重复复制：指向同一个对象的属性会被重复创建

```javascript
let obj = {         
    reg : /^asd$/,
    fun: function(){},
    syb:Symbol('foo'),
    asd:'asd'
}; 
let cp = JSON.parse(JSON.stringify(obj));
```

##### 递归属性

```javascript
const deepClone=function(obj){
    const types={
        'date':Date,
        'regexp':RegExp
    };
    if(obj===null || typeof obj!=='object') return obj;
    for(let key in types)
        if(obj instanceof types[key])
            return new types[key]();
    let cloneObj=new obj.constructor();
    for(let key in obj)
        if(obj.hasOwnProperty(key))
            cloneObj[key]=deepClone(obj[key]);
    return cloneObj;
}
```

##### lodash

> _.cloneDeep

```javascript
const _=require('lodash');
const obj1 = {
    a: 1,
    b: { f: { g: 1 } },
    c: [1, 2, 3]
};
const obj2 = _.cloneDeep(obj1);
console.log(obj1.b.f === obj2.b.f);// false
```

#### 浅拷贝

##### Object.assign()

> 可以把任意多个源对象的可枚举属性拷贝到目标对象然后返回

```javascript
let obj1 = { person: {name: "kobe", age: 41},sports:'basketball' };
let obj2 = Object.assign({}, obj1);
obj2.person.name = "wade";
obj2.sports = 'football'
console.log(obj1); // { person: { name: 'wade', age: 41 }, sports: 'basketball' }
```

##### lodash

> _.clone

```javascript
var _ = require('lodash');
var obj1 = {
    a: 1,
    b: { f: { g: 1 } },
    c: [1, 2, 3]
};
var obj2 = _.clone(obj1);
console.log(obj1.b.f === obj2.b.f);// true
```

##### 展开运算符

```javascript
let obj1 = { name: 'Kobe', address:{x:100,y:100}}
let obj2= {... obj1}
obj1.address.x = 200;
obj1.name = 'wade'
console.log('obj2',obj2) // obj2 { name: 'Kobe', address: { x: 200, y: 100 } }
```

##### Array.prototype.concat

```javascript
let arr = [1, 3, {
    username: 'kobe'
    }];
let arr2 = arr.concat();    
arr2[2].username = 'wade';
console.log(arr); //[ 1, 3, { username: 'wade' } ]
```

##### Array.prototype.slice

```javascript
let arr = [1, 3, {
    username: ' kobe'
    }];
let arr3 = arr.slice();
arr3[2].username = 'wade'
console.log(arr); // [ 1, 3, { username: 'wade' } ]
```

## 区别型面试题

### Array.from与扩展运算符...的区别

#### 扩展运算符

任何定义了Iterator接口的对象都可以用扩展运算符转为真正的数组，比如Set和Map、Generator函数

扩展运算符对于没有实现iterator接口的，不能转

#### Array.from

Array.from方法用于将两类对象转为真正的数组：

+ 类似数组的对象Array-like Object
+ 可遍历对象，包括Set和Map

能用扩展运算符转的一定能用Array.from

```javascript
const obj = { a: 1, b: 2 };
let arr = [...obj]; // TypeError: Cannot spread non-iterable object
// 其实最终想要的是 [{a:1},{b:2}]

let a = { ...obj };
// 这里 不报错，原因是 只有[...变量] 这种方式，也就是最后要的是数组的情况下，
//  才会去调用iterator接口。如果不是数组，是对象的话，不会调用的。
a = { a: 1, b: 2 }; // 这里相当于浅拷贝
```

##### 替代品slice

slice方法是浅拷贝

```javascript
let toArray=(()=>{
    Array.from ? Array.from : obj=>[].slice.call(obj);
})
```

##### 使用

```javascript
console.log(Array.from({length:3}));// [undefined,undefined,undefined]
console.log(Array.from({length:3},()=>1));// [1,1,1]
console.log(Array.from([1,2,3],(x)=>2*x));// [2,4,6]
```

#### Array.of

将一组值转为数组，弥补了数组构造函数Array的不足

Array方法只有在参数不少于两个时才会返回参数组成的新数组，参数一个以下时指的都是数组的长度

```javascript
console.log(Array(3));// [,,]
console.log(Array());// []
console.log(Array(1,2));// [1,2]
```

Array.of始终返回参数值组成的数组，如果没有参数返回空数组

##### ES5实现

```javascript
let ArrayOf=function(){
    return [].slice.call(arguments);
}
```

#### 总结

+ 扩展运算符如果想把对象转为数组，这个对象必须实现iterator接口
+ Array.from可以兼容扩展运算符，并可以将类数组对象（有length属性）转为数组

### 基本数据类型和引用数据类型

+ 对于基本类型来说，复制变量复制的是变量的值，与原来的变量并无关系；对于引用类型，复制变量只是复制引用地址而已，两者除了在栈中的地址不同之外，但指向的对象是同一个，因此修改会相互影响
+ 因为引用数据占据的空间会比较大且不固定，因此存储在堆内存中不会影响程序性能，但是引用数据的内存地址依然存储在栈中；基本类型的数据因为大小固定，存储在栈中

## 细节问题

###  for循环中能用const定义变量吗

https://blog.biossun.xyz/variable-declarations-in-for-semantics/

```javascript
for(const i=0;i<5;i++){// Assignment to constant variable. 
    console.log(i)
}
```

从上面结果可以看出，不可以用const定义变量i这是因为i是常量，不可以有除初始化之外的赋值

#### for语句规范

在规范文档中，所有语句相关内容都定义在 *[第 13 章 Statements and Declarations](http://www.ecma-international.org/ecma-262/#sec-ecmascript-language-statements-and-declarations)* 中

`for` 语句的执行逻辑定义在 [13.7.4.7 Runtime Semantics: LabelledEvaluation](http://www.ecma-international.org/ecma-262/6.0/#sec-for-statement-runtime-semantics-labelledevaluation) 这一节中。

##### 使用var声明循环变量的for语句

<img src="https://p2.biossun.xyz/blog/biossun/2021/01/1.png" style="zoom:50%;" />

+ 在使用var声明循环变量时，会直接在for语句所在的作用域中执行初始化表达式，循环变量自然就会被声明到该作用域中；
+ 若初始化表达式正常执行完
+ 开始执行循环体

##### 使用let或const声明

<img src="https://p2.biossun.xyz/blog/biossun/2021/01/2.png" style="zoom:50%;" />

+ 基于`for`语句所在的作用域创建一个新的子作用域，称为loopEnv
+ 在`loopEnv`创建所有循环变量
+ 将`loopEnv`设置为当前作用域
+ 在`loopEnv`中执行初始化表达式，为循环变量赋于初值
+ 执行循环体，并在执行结束后还原当前作用域

在第10步时，如果使用`let`而非`const`声明循环变量，则会额外创建一个`perIterationLets`结构，其中会包含所有循环变量的绑定名称，该结构会连同其数据一并被传入循环体执行逻辑中。而在使用`cosnt`时，该结构被声明为空，和使用`var`声明循环变量时一样。

##### `perIterationLets`结构的作用

循环体执行逻辑

<img src="https://p2.biossun.xyz/blog/biossun/2021/01/3.png" style="zoom:50%;" />

可以看到在每次循环中，大体的处理逻辑就是：

+ 首先执行测试表达式，若返回`false`则退出循环
+ 执行循环体语句，根据执行结果判断是否退出循环
+ 最后执行增量表达式

在循环开始之前，一起每次循环最后执行增量表达式之前，都会拿`perIterationLet`去执行一个创建每次迭代环境的逻辑

<img src="https://p2.biossun.xyz/blog/biossun/2021/01/4.png" style="zoom:50%;" />

这个逻辑的目的就是未来下次循环创建作用域环境

+ 若`perIterationLets`中有值，会基于当前作用域的父级创建一个新的子作用域，称为`interationEnv`

  当前作用域在循环开始之前时`loopEnv`，在每次循环中最后执行增量表达式之前时，是当次循环对应的`interationEnv`，它们的父作用域就是for语句所在的作用域

+ 在`interationEnv`被创建之后，会根据`perIterationLets`在其中创建所有循环变量，并从当前作用域中复制循环变量的值

这么做的目的就是为了在每次循环时，都会有一个单独的作用域用于绑定此次循环变量的值

```javascript
const array = ['a', 'b', 'c']

for (let i = 0; i < array.length; i++) {
	setTimeout(() => {
		console.log(`for: index: ${i}, value: ${array[i]}`)
	}, 100)
}
```

<img src="https://p2.biossun.xyz/blog/biossun/2021/01/5.png" style="zoom:50%;" />

如果`perIterationLets`为空，也就是在使用`var`或`const`声明循环变量时，则不会创建`interationEnv`。那么若是使用`var`声明循环变量，在整个循环语句执行过程中，当前作用域将一直都是for语句所在的作用域，若是使用`const`，则一直都是`loopEnv`

+ 使用`var`声明变量时，因为`var`只有函数作用域，因此for语句不需要再创建`interationEnv`来存放这些变量
+ 使用`const`时，因为常量值不可变，没必要创建`interationEnv`，仅需要提供一个`loopEnv`来存放

#### `for-in`语句

<img src="https://p2.biossun.xyz/blog/biossun/2021/01/6-1.png" style="zoom:50%;" />

`for-in`和`for-of`的逻辑是一样的，都是先执行迭代表达式获取迭代器，然后执行循环体。循环体执行逻辑中的条件分支较多，但与循环变量声明类型相关的处理逻辑是在第g和h两步中

<img src="https://p2.biossun.xyz/blog/biossun/2021/01/7-1.png" style="zoom:50%;" />

若是使用`var`声明循环变量，或只是赋值给当前作用域中的某个变量，不会做额外的事情；但若是使用`let`或`const`声明循环变量，会为每次循环创建一个单独的作用域，其父级作用域为当前for语句所在的作用域环境。该作用域中会创建循环变量，并使用从迭代器中获取的下一个迭代项的值作为其初始值，在这个过程中并不会修改循环变量，因此可以在`for-of`语句中使用const声明循环变量

#### 性能


































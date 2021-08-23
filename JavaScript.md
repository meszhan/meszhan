# JavaScript

## 语法型面试题

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

### 在函数内部使用_this或that

> 利用_this或that可以很轻松的固定this，这一点在Axios等异步请求中非常有用，因为异步请求中有大量的回调，甚至是以前的回调地狱问题

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

### 区别

+ call和apply只是传入的参数形式不同，call是参数列表，而apply是数组

+ call、apply和bind不同的是，前者返回执行的结果，而bind返回一个绑定了this的函数，不过需要说明的是，返回的函数仍然可以被绑定this到其他对象上

### 测试

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

requir导出的内容时module.exports指向的内存，不是exports的

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
+ 

### JavaScript中的隐式转换

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




































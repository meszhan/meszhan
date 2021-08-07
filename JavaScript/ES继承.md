# ES继承

### 原型链继承

> 将子类构造函数的原型对象链接到父类的原型链上
>
> 缺点：
>
> + 每个实例对引用类型的修改都会被其他实例共享
> + 在创建Child实例时，无法向Parent穿餐，Child实例没法自定义自己的属性

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

### 借用构造函数继承

> 在子类的构造函数中调用父类构造函数
>
> 优点：
>
> + 解决引用类型属性的问题
> + 子类可以向父类传参
>
> 缺点：
>
> + 无法复用父类的公共函数
>
>   因为只是相当于把子类构造函数中的this放到父类中跑了一遍，只能获取通过this定义的属性，不能获取父类父类原型上的方法
>
> + 每次子类都早实例都会执行一次父类的函数

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

### 组合继承

> 结合原型式继承和构造函数继承
>
> 优点：
>
> + 解决引用类型数据问题
> + 子类可以向父类传参
> + 可以实现父类方法复用
>
> 缺点：
>
> + 会执行两次父类构造函数

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

### 原型式继承

> 把传入的对象链接到创建对象的原型上，实现继承，类似于原型链继承
>
> 缺点：
>
> + 对于引用类型的数据会共享

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

### 寄生式继承

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

### 寄生组合式继承

> 在构造函数继承的基础上，将子类构造函数链接到父类构造函数通过Object.create构造出的新对象上

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
































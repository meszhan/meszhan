# JavaScript内置对象

### 内置对象

> String、Number、Boolean、Object、Function、Array、Date、RegExp、Error
>
> 上面的内置构造函数分别也都对应了常见的基本数据类型和引用数据类型

```javascript
var strP = "i am iverson";
console.log(typeof strP); // "string"
console.log(strP instanceof String); //false

var strObj = new String("我是海贼猎人索隆");
console.log(typeof strObj);// Object
console.log(strObj instanceof String); // true;
```

> 看出了什么，虽然typeof显示strP是一个string，但是instanceof不承认啊
>
> 正解就是：只有对象才配拥有原型链，string只是一个基本数据类型而已，除非使用new String构造的字符串对象而不是简单的字面量字符串

### []和.对象的访问

+ .属性访问；属性名要满足基本的变量规范
+ [] 键访问：可以接受任意的utf-8/Unicode字符串作为属性名

### 检测存在

```javascript
var myObj={
	name:"李白打野换酒钱"
};
console.log("name" in myObj);
//true;

console.log("age" in myObj);
//false;

console.log(myObj.hasOwnProperty("name"));
//true

console.log(myObj.hasOwnProperty("age"));
//false;

for(var k in myObj){
	console.log("k",k,myObj[k])
}
//k name 李白打野换酒钱

console.log(Object.keys(myObj));
//["name"]

console.log(Object.getOwnPropertyNames(myObj));
//["name"]
```

> + in操作符：检查属性是否在对象以及对象的原型链上，参考for...in...
> + hasOwnProperty：只检查对象本身
> + Object.keys：返回一个数组，包括所有可枚举属性
> + Object.getOwnPropertyNames：返回一个包含所有属性的数组，无论是否可枚举enumerable

### for...of遍历

> for...of循环首先向被访问的对象请求一个迭代器对象，然后通过迭代器对象的next方法来遍历所有返回值

```javascript
//for of 循环
var myArr = [1,2,3];
for(var v of myArr){
  console.log(v);
}
//1,2,3


//迭代器对象
var myArr1 = [1,2,3];
var it = myArr1[Symbol.iterator]();
it.next();//{value:1,done:false};
it.next();//{value:2,done:false};
it.next();//{value:3,done:false};
it.next();//{done:true};

//书中对象使用迭代器的例子

var myObject={
	a:2,
  b:3
}

/*
	* 劫持Symbol.iterator属性
	* 利用闭包的特性调用next返回后面的值
*/
Object.defineProperty(myObject,Symbol.iterator,{
  enumerable:false,
  writable:false,
  configurable:true,
  value:function(){
    var o = this;
    var idx = 0;
    var ks = Object.keys(o);
    return {
      next:function(){
        return {
          value:o[ks[idx++]],
          done:(idx > ks.length)
        }
      }
    }
  }
});

var it = myObject[Symbol.iterator]();
it.next();//{value:2,done:false};
it.next();//{value:3,done:false};
it.next();//{done:true};

for(var t of myObject){
  console.log(t);
};
//2,3
```




































# JavaScript

## 语法型面试题



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




































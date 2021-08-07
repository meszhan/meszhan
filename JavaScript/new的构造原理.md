# new新建实例的原理

### 引言

### 四个步骤及实现

> 联想原型和原型链

+ 创建空对象
+ 链接原型链
+ 执行构造函数，注意改变this指向（call、apply）
+ 返回结果（优先返回构造函数返回的对象，次之返回obj）

```
const myNew=function(){
    const obj={};
    const [constructor,...args]=[...arguments];
    obj.__proto__=constructor.prototype;
    const result=constructor.apply(obj,args);
    console.log(obj)
    return typeof result==='object' ? result : obj;
}
```

### 如何函数确定是否被new了

> new.target属性可以确定

#### 必须实例化的构造函数

```javascript
const Person=function(name,age){
    if(!new.target) throw new Error('该构造函数必须new');
    this.name=name;
    this.age=age;
};
```

#### 不能独立使用，必须继承才能使用

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

### 测试代码

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


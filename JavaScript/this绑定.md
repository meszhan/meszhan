# 绑定this的方法

### 引言

> + call、apply的实现和区别
> + bind依托于call或apply的实现，以及和它们的区别
> + 箭头函数绑定this

### call

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

### apply

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

### bind

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

### 箭头函数绑定this

> 箭头函数的this始终指向函数定义时的this，很明显于普通函数内部的this的指向不同（指向执行时的）
>
> 箭头函数中没有this绑定，必须通过查找作用域链来决定this，如果被非箭头函数包含，则this绑定最近一层非箭头函数的this

```javascript
    var name = "windowsName";

    var a = {
        name : "Cherry",

        func1: function () {
            console.log(this.name)     
        },

        func2: function () {
            setTimeout( () => {
                this.func1()
            },100);
        }

    };

    a.func2()     // Cherry
```

> 箭头函数的这种行为与我们寻常了解到的setTimeout回调函数内部的this完全不同，寻常的setTimeout回调函数内部的this都会指向window或global，但是箭头函数因为在定义时指定this，它会捕获func2函数内部的this

### 在函数内部使用_this或that

> 利用_this或that可以很轻松的固定this，这一点在Axios等异步请求中非常有用，因为异步请求中有大量的回调，甚至是以前的回调地狱问题

```
    var name = "windowsName";

    var a = {

        name : "Cherry",

        func1: function () {
            console.log(this.name)     
        },

        func2: function () {
        		// _tgi
            var _this = this;
            setTimeout( function() {
                _this.func1()
            },100);
        }

    };

    a.func2()       // Cherry
```

### 区别

> call和apply只是传入的参数形式不同，call是参数列表，而apply是数组
>
> call、apply和bind不同的是，前者返回执行的结果，而bind返回一个绑定了this的函数，不过需要说明的是，返回的函数仍然可以被绑定this到其他对象上



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














































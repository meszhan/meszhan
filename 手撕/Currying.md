# Currying

> 函数Currying：将能够接受多个参数的函数转化为接收单一参数的函数，并且返回接收余下参数且返回结果的新参数的技术

### 作用

+ 参数复用
+ 提前返回
+ 延迟执行

```javascript
const curry=function(fn){
    return function curriedFn(){
        let [...args]=[...arguments];
        if(args.length<fn.length){
            return function(){
                return curriedFn(...args,...arguments);
            }
        }
        else
            return fn(...args);
    }
}

const sum=function(a,b,c){
    return a+b+c;
};

console.log(curry(sum)(1)(2,3));
```


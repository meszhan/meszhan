# CommonJS

### require

> Nodejs遵循CommonJS规范，使用require关键字加载模块

#### require重复引入的问题

> 在使用require引入模块时，多个代码文件多次引入相同的模块会不会造成重复？
>
> Nodejs中默认先从缓存中加载模块，一个模块被加载一次之后，就会在缓存中维持一个副本，如果遇到重复加载的模块会直接提取缓存中的副本，在任何时候每个模块只在缓存中有一个实例

#### require加载模块时同步还是异步

> 同步
>
> + 一个作为公共依赖的模块，最好一次性加载出来，同步更好
> + 模块的个数是有限的，而且Nodejs在require时会自动缓存已经加载的模块，再加上访问的都是本地文件，IO开销可以忽略

#### require的缓存策略

> Nodejs会自动缓存require引入的文件，下次再引入不需要经过文件系统而是直接从缓存读取
>
> 不过这种缓存式经过文件路径定位的，即使两个完全相同的文件，位于不同路径下，也会在缓存中维持两份

可以通过以下获取缓存中的文件

```javascript
console.log(require.cache)
```

### exports与module.exports的区别

#### js文件启动时

> mode执行一个文件时，会给这个文件生成一个exports和module对象，module有一个exports属性

```javascript
exports = module.exports = {};
```

#### require加载模块

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

> requir导出的内容时module.exports指向的内存，不是exports的

### 实际例子

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

> 运行b模块时，先加载a模块，会执行setTimeout，到时间后把回调函数扔进宏任务队列
>
> 然后调用setCount，这里注意，setCount内部改变的是a模块内部的count变量，因为count是普通类型，导出的是count的拷贝，因此即使将count增1，输出仍为1
>
> 然后执行setTimeout，扔进队列
>
> 然后执行a模块中的回调，输出count为2，因为setCount被调用了
>
> 最后执行b模块中的回调，输出count为1，没有发生改变
























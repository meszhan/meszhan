# Function

### Function.length

> length属性表示函数期待的参数数量，但是只包括第一个具有默认值的参数之前的参数
>
> + Writable:false，表示不可写
> + Enumerable:false，不可枚举
> + Configurable:true，可配置

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


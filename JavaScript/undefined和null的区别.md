# null和undefined的区别

> 首先，null和undefined都是基本数据类型，即使typeof不这么认为

```javascript
console.log(typeof null) // 'object'
console.log(typeof undefined) // 'undefined'
```

### undefined

> Undefined：按照字面上意思就是未定义的值，什么情况变量会未定义

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

> ES明确规定void操作符对任何表达式求值都返回undefiend，这和函数执行操作后没有返回值作用相同
>
> Js中的函数执行后都会有返回值，如果没有return或return;，默认返回undefined

### null

> 空值，表示对象被人为的置为空对象，而非undefined，注意，null值在栈中的变量不会指向堆中的内存对象，因为它没有

> + 将来会用来 保存对象的变量，置为null，不希望被其他类型值占用
> + 当某个变量不再被需要时，设为null解除引用

#### typeof null

```javascript
console.log(typeof null) // 'object'
```

> + null表示一个空指针，代表的就是一个空对象，只不过在堆中没有内存
>
> + JavaScript中的值是由一个表示类型的标签和实际数据值决定的，因为null代表空指针，为0x00，
>
>   null的类型标签为0，与对象的类型标签为0相同，会被识别为object




















































# Symbol

#### Symbol.species

> Symbol.species是个函数值属性，被构造函数用以创建派生对象
>
> species访问器属性允许子类覆盖对象的默认构造函数

> 如果想在扩展数组类MyArray上返回Array对象，例如使用map方法返回默认的构造函数时，希望这些方法能够返回父级的Array对象，取代MyArray对象

```javascript
class MyArray extends Array {
  // 覆盖 species 到父级的 Array 构造函数上
  static get [Symbol.species]() { return Array; }
}
var a = new MyArray(1,2,3);
var mapped = a.map(x => x * x);

console.log(mapped instanceof MyArray); // false
console.log(mapped instanceof Array);   // true
```


```javascript
const reduce = function (fn, init) {
    if (!Array.isArray(this)) { throw new Error('数组调用') }
    if (!fn || arguments.length<2) { throw new Error('arguments error') };
    const arr = this;
    arr.forEach((value, index, _arr) => {
        init = fn(init, value, index, _arr);
    });
    return init;
};

Array.prototype.reduce = reduce;

console.log([1, 2, 3].reduce((a, b) => a + b,0));
```


# 数组的dfs

> fn([['a', 'b'], ['n', 'm'], ['0', '1']]) => ["an0", "an1", "am0", "am1", "bn0", "bn1", "bm0", "bm1"]

```javascript
const fn = function (targets) {
    let _targets = [...targets];
    if (_targets.length === 1) {
        return _targets.shift();
    }
    let results = _targets.shift();
    while (_targets.length > 0) {
        let items = _targets.shift();
        let len = results.length;
        for (let i = 0; i < len; i++) {
            let item = results.shift();
            results.push(...items.map(val => item + val))
        }
    }
    return results;
}

console.log(fn([['m', 'n'], ['a', 'b'], ['0', '1']]));
```


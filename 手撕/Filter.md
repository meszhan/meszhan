# Array.prototype.filter

````javascript
Array.prototype.filter = function (filterFunction) {
    const targets = this;
    if (!Array.isArray(targets)) {
        throw new Error('Filter function can only be called by array.');
    }
    const results = [];
    targets.forEach((val) => {
        if (filterFunction(val)) {
            results.push(val);
        }
    });
    return results;
};

const result = [{ value: 1 }, { value: 2 }, { value: 3 }].filter(val => val.value >= 3);
console.log(result);
````


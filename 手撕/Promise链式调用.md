# Promise链式调用

> u.console("lunch").setTimeout(3000).console("breakfast").setTimeout(3000).console("dinner");

```javascript
let U = function () {
    this.p = new Promise((resolve) => resolve());
}

U.prototype.setTimeout = function (delay) {
    this.p = this.p.then(() => {
        return new Promise((resolve) => setTimeout(() => resolve(), delay));
    });
    return this;
}

U.prototype.console = function (str) {
    this.p = this.p.then(() => {
        return new Promise((resolve) =>{
            console.log(str);
            resolve();
        });
    });
    return this;
}

let u = new U();
u.console("lunch").setTimeout(3000).console("breakfast").setTimeout(3000).console("dinner");
```

### 红绿灯切换

```javascript
let U = function () {
    this.p = new Promise((resolve) => resolve());
}

U.prototype.setTimeout = function (delay) {
    this.p = this.p.then(() => {
        return new Promise((resolve) => setTimeout(() => resolve(), delay));
    });
    return this;
}

U.prototype.console = function (str) {
    this.p = this.p.then(() => {
        return new Promise((resolve) =>{
            console.log(str);
            resolve();
        });
    });
    return this;
}

let u = new U();

setInterval(() => {
  u=u.console("red").setTimeout(3000).console("green").setTimeout(3000).console("yellow").setTimeout(3000);
}, 0);
```


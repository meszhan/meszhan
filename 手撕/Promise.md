# Promise

```javascript
const Promise = function (executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;

    this.onResolvedCallbacks = [];
    this.onRejectedCallbacks = [];

    let resolve = (value) => {
        if (this.state === 'pending') {
            this.state = 'fullfilled';
            this.value = value;
            this.onResolvedCallbacks.forEach(fn => fn());
        }
    }

    let reject = (reason) => {
        if (this.state === 'pending') {
            this.state = 'rejected';
            this.reason = reason;
            this.onRejectedCallbacks.forEach(fn => fn());
        }
    }

    try {
        executor(resolve, reject);
    } catch (error) {
        reject(error);
    }
}

Promise.prototype.then = function (onResolved, onRejected) {
    onResolved = onResolved ? onResolved : v => v;
    onRejected = onRejected ? onRejected : error => { throw error };
    if (this.state === 'fullfilled')
        onResolved(this.value);
    if (this.state === 'rejected') {
        onRejected(this.reason)
    }
    if (this.state === 'pending') {
        this.onResolvedCallbacks.push(() => resolve(this.value));
        this.onRejectedCallbacks.push(() => reject(this.reason));
    }
}

const p = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(1);
    }, 1000);
});

p.then(console.log);
```

### Promise.allSettled

```javascript
Promise.allSettled = function (promises) {
  let result = [];
  return new Promise((resolve, reject) => {
    promises.forEach((val) =>
      val
        .then(
          (value) => {
            result.push(value);
          },
          (reason) => {
            result.push(reason);
          }
        )
        .finally(() => {
          if (result.length === promises.length) {
            resolve(result);
          }
        })
    );
  });
};


let p1=new Promise((resolve,reject)=>{
    setTimeout(reject,1000,'p1');
});

let p2=new Promise((resolve)=>{
    setTimeout(resolve,1000,'p2');
});

let p3=new Promise((resolve,reject)=>{
    setTimeout(reject,1000,'p3');
});

Promise.allSettled([p1,p2,p3]).then(val=>console.log(val));
```


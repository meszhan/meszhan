# Scheduler

```javascript
class Scheduler {
    constructor(limit, timeout, tryLimit) {
        this.waitingTasks = [];
        this.runningTasks = [];
        this.tryCount = new Map();
        this.limit = limit;
        this.timeout = timeout;
        this.tryLimit = tryLimit;
    }

    addTask(fn) {
        if (!this.tryCount.get(fn)) {
            this.tryCount.set(fn, 0);
        }
        if (this.runningTasks.length >= this.limit) {
            this.waitingTasks.push(fn);
        } else {
            this.runTask(fn);
        }
    }

    runTask(fn) {
        this.runningTasks.push(fn);
        let p = new Promise((resolve, reject) => {
            setTimeout(() => {
                resolve();
            }, 1000);
        }).then(() => {
            this.runningTasks = this.runningTasks.filter((val) => val === fn);
            if (this.waitingTasks.length > 0) {
                this.runTask(this.waitingTasks.shift());
            }
        });
        Promise.race([
            p,
            new Promise((resolve, reject) => {
                setTimeout(() => {
                    reject();
                }, this.timeout);
            }),
        ])
            .then(() => {
                fn();
            })
            .catch(() => {
                console.log(this.tryCount);
                this.tryCount.set(fn, this.tryCount.get(fn) + 1);
                if (this.tryCount.get(fn) >= this.tryLimit) {
                    this.tryCount.delete(fn);
                    console.log(fn, "重试次数过多或超时");
                    return;
                }
                this.addTask(fn);
            });
    }
}

let scheduler = new Scheduler(2, 500, 3);
scheduler.addTask(() => console.log("h"));
scheduler.addTask(() => console.log("he"));
scheduler.addTask(() => console.log("hel"));
scheduler.addTask(() => console.log("hell"));
```


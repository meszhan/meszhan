# 节流Throttle

```javascript
/*
    * 节流
    * @params {Function} fn
    * @params {Number} delay
*/
const throttle=function(fn,delay){
    let timer=null;
    return function(){
        if(timer)
            return;
        timer=setTimeout(()=>{
            fn(...arguments);
        },delay);
    };
};

const callback=function(num){
    console.log(num);
}

global.name='wax';

const fn=debounce(callback,1000);
fn(1);
fn(2);
fn(3);
```


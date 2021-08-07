# 防抖Debounce

```javascript
/*
    * 防抖
    * @params {Function} fn
    * @params {Number} delay
*/
const debounce=function(fn,delay){
    let timer=null;
    return function(){
        let that=this;
        if(timer){
            clearTimeout(timer);
        }
        timer=setTimeout(()=>{
            // 这里要改变this指向
            fn.call(that,...arguments);
        },delay);
    }
}

const callback=function(){
    console.log(this.name);
}

global.name='wax';

const fn=debounce(callback,1000);
console.log(global.name);
fn('zzg');
```


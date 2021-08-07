# setTimeout实现setInterval

```javascript
const setInterval=(fn,delay)=>{
    setTimeout(()=>{
        fn('setInterval');
        setInterval(fn,delay);
    },delay);
}

setInterval(console.log,1000);
```


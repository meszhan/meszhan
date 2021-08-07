# Array.prototype.map

```javascript
Array.prototype.map=function(fn){
    const targets=this;
    if(!Array.isArray(targets)){
        throw new Error('Map function must be called by an Array');
    }
    let init=[];
    targets.reduce((temp,val,index,arr)=>{
        init.push(fn(val,index,arr));
        return init;
    },init);
    return init;
};

console.log([1,2,3,4,5].map((val,index,arr)=>[val,index,arr]));
```


# 打乱数组顺序

```javascript
const shuffleArray=function(targets){
    let len=targets.length;
    for(let i=0;i<len;i++){
        let ran=Math.floor(Math.random()*(i+1));
        [targets[ran],targets[i]]=[targets[i],targets[ran]];
    }
    return targets;
}

console.log(shuffleArray([1,2,3,4,5,6,7])); 
```


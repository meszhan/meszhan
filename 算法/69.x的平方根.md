# x的平方根

### 暴力遍历

```javascript
var mySqrt = function(x) {
    if(x<=1){
        return x;
    }
    for(let i=1;i<x;i++){
        if(i*i<=x&&(i+1)*(i+1)>x){
            return i;
        }
    }
};
```

### 二分查找

```javascript
var mySqrt = function(x) {
    let left=0,right=x,result=-1;
    while(left<=right){
        let mid=left+Math.floor((right-left)/2);
        if(mid*mid<=x){
            result=mid;
            left=mid+1;
        }else{
            right=mid-1;
        }
    }
    return result;
};
```


































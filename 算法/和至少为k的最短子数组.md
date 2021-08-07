# 和至少为k的最短子数组

https://leetcode-cn.com/problems/shortest-subarray-with-sum-at-least-k/solution/he-zhi-shao-wei-k-de-zui-duan-zi-shu-zu-by-leetcod/

```javascript
var shortestSubarray = function(nums, k) {
    let len=nums.length;
    if(len<0){
        return -1;
    }
    let deque=[0];// 维护一个和队列
    nums.forEach((val,index)=>{
        deque.push(val+deque[index]);
    });
    // 存储对于每个y，找出满足deque[y]-deque[x]>=K的最大的x
    let monoq=[];
    let ans=len+1;
    for(let y=0;y<deque.length;++y){
        // 从后往前找，如果前面的大于等于当前的，说明不可能是，因为K是正整数
        while(monoq.length>0&&deque[y]<=deque[monoq[monoq.length-1]]){
            monoq.pop();
        }
        // 从前往后找，找出所有满足条件的x，并逐渐求最小值覆盖，并从头部去掉
        while(monoq.length>0&&deque[y]>=deque[monoq[0]]+k){
            ans=Math.min(ans,y-monoq.shift());
        }
        monoq.push(y);
    }
    return ans<len+1 ? ans : -1;
};
```




























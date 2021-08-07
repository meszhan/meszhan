# 数组中的第K个最大元素

https://leetcode-cn.com/problems/kth-largest-element-in-an-array/

```javascript
var findKthLargest = function(nums, k) {
    let swap=function(nums,i,j){
        let temp=nums[i];
        nums[i]=nums[j];
        nums[j]=temp;
    }
    let randomPartition=function(nums,left,right){
        // 获取一个left和right之间随机的下标
        let i=Math.floor(Math.random()*(right-left+1))+left;
        // 交换两个值
        swap(nums,i,right);
        return partition(nums,left,right);
    }
    let partition=function(nums,left,right){
        // 大的放右边，小的放左边
        let x=nums[right],i=left-1;
        for(let j=left;j<right;j++){
            if(nums[j]<=x){
                swap(nums,++i,j);
            }
        }
        // 把基准放到中间位置去 
        swap(nums,i+1,right);
        return i+1;
    }
    let quickSelect=function(nums,left,right,index){
        // 得到分治后的下标q
        let q=randomPartition(nums,left,right);
        // q刚好为index，直接返回
        if(q===index){
            return nums[q];
        }else{
            // 如果小于index，递归右部分，大于则递归左部分
            return q<index ? quickSelect(nums,q+1,right,index) : quickSelect(nums,left,q-1,index);
        }
    }
    // 快排，从0开始到nums.length-1最后一个，需要找到下标为nums.length-k的值
    return quickSelect(nums,0,nums.length-1,nums.length-k);
};
```


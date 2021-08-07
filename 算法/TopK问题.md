# TopK问题

https://github.com/sisterAn/JavaScript-Algorithms/issues/73

> 经典的TopK问题：最大或最小的K个数，前K个高频元素，第K个最大或最小元素

### 直接排序

> 直接对已有数组进行排序，后取前k个数字

```javascript
var getLeastNumbers = function(arr, k) {
    return arr.sort((a,b)=>a-b).slice(0,k);
};
```

#### 复杂度分析

> + 时间复杂度：O(blogs)
> + 空间复杂度：O(logn)
>
> 在V8引擎7.0版本之前，数组长度小于10时，sort方法使用插入排序，否则使用快速排序
>
> 在7.0之后，舍弃了快速排序，因为它不稳定，最坏情况下会降级为O(n2)
>
> 使用一种混合排序算法：TimSort，在数据量小的子数组中使用插入排序，再使用归并排序将有序的子数组合并，时间复杂度O(nlogn)

### 冒泡

> 因为只需要求出数组中的第K个最大元素，因此不需要整体排序
>
> 可以使用冒泡排序，将最大的数从底部冒出来

```javascript
function bubbleSort(targets,k){
    bubble(targets,k);
    return targets.slice(targets.length-k);
}

function bubble(targets,k){
    for(let i=0;i<k;i++){
        let flag=false;
        for(let j=0;j<targets.length-i-1;j++){
            if(targets[j]>targets[j+1]){
                swap(targets,j,j+1);
                flag=true;
            }
        }
        // 说明没有发生数据交换，已经排好序了
        if(!flag) break;
    }
}

function swap(targets,i,j){
    let temp=targets[i];
    targets[i]=targets[j];
    targets[j]=temp;
}

console.log(bubbleSort([1,3,2,1,4],3));
```

#### 复杂度分析

> + 时间复杂度：最好O(n)，即已经排好序了，平均O(n*k)
> + 空间复杂度：O(1)

### 最小堆或最大堆

```javascript
function buildHeap(targets, heapSize) {
  for (let i = Math.floor(heapSize / 2); i >= 1; i--) {
    heapify(targets, heapSize, i);
  }
}

function heapify(targets, heapSize, i) {
  while (true) {
    let maxIndex = i;
    if (2 * i <= heapSize && targets[2 * i] > targets[i]) {
      maxIndex = 2 * i;
    }
    if (2 * i + 1 <= heapSize && targets[2 * i + 1] > targets[maxIndex]) {
      maxIndex = 2 * i + 1;
    }
    swap(targets, i, maxIndex);
    if (maxIndex === i) break;
    i = maxIndex;
  }
}

function swap(targets, i, j) {
  let temp = targets[i];
  targets[i] = targets[j];
  targets[j] = temp;
}

function topK(targets, k) {
  buildHeap(targets, k);
  for (let i = k + 1; i < targets.length; i++) {
    if (targets[1] > targets[i]) {
      swap(targets, i, 1);
      heapify(targets, k, 1);
    }
  }
}
let targets=[,1,2,3,4,8,7,6,5,4];
topK(targets,7);
console.log(targets.slice(1,8));
```

#### 复杂度分析

> + 时间复杂度：遍历剩余的数组元素O(n)，堆化O(logk)
> + 空间复杂度：O(k)

#### 优势

> 动态数组TopK

### 快速选择

#### 快排

> + 创建两个指针分别指向数组的最左端和最右端
> + 在数组中任意取一个元素作为基准
> + 左指针右移，遇到比基准大的停止
> + 右指针左移，遇到比基准小的停止，交换左右指针指的元素
> + 重复3，4，直到左指针超过右指针，此时小的在左边，大的在右边
> + 递归二分

```
function quickSort(targets){
    quick(targets,0,targets.length-1);
    return targets;
}

function quick(targets,left,right){
    if(left<right){
        let index=partition(targets,left,right);
        if(left<index-1){
            quick(targets,left,index-1);
        }
        if(index<right){
            quick(targets,index,right);
        }
    }
}

function partition(targets,left,right){
    let base=targets[right];
    while(left<=right){
        while(targets[left]<base){
            left++;
        }
        while(targets[right]>base){
            right--;
        }
        if(left<=right){
            swap(targets,left,right);
            left++;
            right--;
        }
    }
    return left;
}

function swap(targets,i,j){
    let temp=targets[i];
    targets[j]=targets[i];
    targets[i]=temp;
}

console.log(quickSort([1,3,2,5,3,4,1]));
```
































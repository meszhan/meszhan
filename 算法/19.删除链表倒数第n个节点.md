# 删除链表倒数第n个节点

https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/

### 双指针

> 让first指针先走n步，那么当first到达时，second指针其实是指向倒数第n个节点的前两个
>
> 因此second.next跨越倒数第n个节点指向second.next.next

```javascript
var removeNthFromEnd = function(head, n) {
    let dummy=new ListNode(0,head);
    let first=head,second=dummy;
    for(let i=0;i<n;i++){
        first=first.next;
    }
    while(first){
        first=first.next;
        second=second.next;
    }
    second.next=second.next.next;
    return dummy.next;
};
```

### 栈

### 计算链表长度


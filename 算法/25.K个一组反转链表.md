# K个一组翻转链表

https://leetcode-cn.com/problems/reverse-nodes-in-k-group/

> + 类似于反转链表，在整个链表头部添加一个新的节点用来做反转
> + 从hair节点出发，走k步得到k长的节点（不包括hair），局部翻转
> + 局部翻转：prev要指向最后一个节点的next，其他类似
> + 

```javascript
let reverse = function (head, tail) {
    /**
        * 正常情况下，prev应该指向null，但这只是因为单个的反转链表的最后一个节点的next为null
        * 对于k个一组的链表来说，头节点在翻转后应该指向尾节点的next，也就是需要继续连接下一个k组
     */
    let prev = tail.next;
    // p指向头部
    let p = head;
    // p没有到尾部
    while (prev !== tail) {
        let next = p.next;
        p.next = prev;
        prev = p;
        p = next;
    }
    // 此时tail作为首节点，p作为k组中的尾节点
    return [tail, head];
}
var reverseKGroup = function (head, k) {
    // 新建一个节点放在链表最前面
    let hair = new ListNode(0);
    hair.next = head;
    // 保存当前组的前一个节点
    let pre = hair;
  // 下一组首节点不为空
    while (head) {
        // tail指向下一个k个的前一个节点，因为k个要从这里开始遍历
        let tail = pre;
        // 走k步，tail到达k个的最后一个
        for (let i = 0; i < k; i++) {
            tail = tail.next;
            // tail为空，说明到达最后一个节点且数量不足k个，不进行翻转
            if (!tail) {
                return hair.next;
            }
        }
        // 保存tail的下一个节点
        let next = tail.next;
        // 翻转k个
        [head, tail] = reverse(head, tail);
        // head连接到前面
        pre.next = head;
        // tail连接到后面
        tail.next = next;
        // pre后移到当前组尾部
        pre = tail;
        // head后移到下一组首节点
        head = tail.next;
    }
    // 返回首节点
    return hair.next;
};
```

### 递归

```javascript
 let reverse=function(head,tail){
        let prev=null;
        // p指向头部
        let p=head;
        // p没有到尾部
        while(p!==tail){
            let next=p.next;
            p.next=prev;
            prev=p;
            p=next;
        }
        // 此时tail作为首节点，p作为k组中的尾节点
        return prev;
    }
var reverseKGroup = function(head, k) {
    // 到达链表末尾或只剩一个节点时，直接返回
    if(head===null||head.next===null){
        return head;
    }
    // 尾节点移动
    let tail=head;
    // 移动k个位置，到达当前组的后一个节点
    for(let i=0;i<k;i++){
        // 不足k个直接返回
        if(tail===null){
            return head;
        }
        tail=tail.next;
    }
    // 得到翻转后的首节点
    let newHead=reverse(head,tail);
    // 当前组的首节点的next指向下一组的新首节点
    head.next=reverseKGroup(tail,k);
    return newHead;
};
```


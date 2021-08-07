# 手写diff算法

### Vue2的diff算法

```javascript
/**
 * diff 过程:
 *   diff 优化：做了四种假设，假设新老节点开头结尾有相同节点的情况，一旦命中假设，就避免了一次循环，以提高执行效率
 *             如果不幸没有命中假设，则执行遍历，从老节点中找到新开始节点
 *             找到相同节点，则执行 patchVnode，然后将老节点移动到正确的位置
 *   如果老节点先于新节点遍历结束，则剩余的新节点执行新增节点操作
 *   如果新节点先于老节点遍历结束，则剩余的老节点执行删除操作，移除这些老节点
 */
function updateChildren(parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    // 老节点的开始索引
    let oldStartIdx = 0
    // 新节点的开始索引
    let newStartIdx = 0
    // 老节点的结束索引
    let oldEndIdx = oldCh.length - 1
    // 新节点的结束索引
    let newEndIdx = newCh.length - 1
    // 第一个老节点
    let oldStartVnode = oldCh[0]
    // 最后一个老节点
    let oldEndVnode = oldCh[oldEndIdx]
    // 第一个新节点
    let newStartVnode = newCh[0]
    // 最后一个新节点
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, vnodeToMove, refElm

    // removeOnly是一个特殊的标志，仅由 <transition-group> 使用，以确保被移除的元素在离开转换期间保持在正确的相对位置
    const canMove = !removeOnly

    if (process.env.NODE_ENV !== 'production') {
        // 检查新节点的 key 是否重复
        checkDuplicateKeys(newCh)
    }

    // 遍历新老两组节点，只要有一组遍历完（开始索引超过结束索引）则跳出循环
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        if (isUndef(oldStartVnode)) {
            // 如果节点被移动，在当前索引上可能不存在，检测这种情况，如果节点不存在则调整索引
            oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
        } else if (isUndef(oldEndVnode)) {
            oldEndVnode = oldCh[--oldEndIdx]
        } else if (sameVnode(oldStartVnode, newStartVnode)) {
            // 老开始节点和新开始节点是同一个节点，执行 patch
            patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
            // patch 结束后老开始和新开始的索引分别加 1
            oldStartVnode = oldCh[++oldStartIdx]
            newStartVnode = newCh[++newStartIdx]
        } else if (sameVnode(oldEndVnode, newEndVnode)) {
            // 老结束和新结束是同一个节点，执行 patch
            patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
            // patch 结束后老结束和新结束的索引分别减 1
            oldEndVnode = oldCh[--oldEndIdx]
            newEndVnode = newCh[--newEndIdx]
        } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
            // 老开始和新结束是同一个节点，执行 patch
            patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
            // 处理被 transtion-group 包裹的组件时使用
            canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
            // patch 结束后老开始索引加 1，新结束索引减 1
            oldStartVnode = oldCh[++oldStartIdx]
            newEndVnode = newCh[--newEndIdx]
        } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
            // 老结束和新开始是同一个节点，执行 patch
            patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
            canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
            // patch 结束后，老结束的索引减 1，新开始的索引加 1
            oldEndVnode = oldCh[--oldEndIdx]
            newStartVnode = newCh[++newStartIdx]
        } else {
            // 如果上面的四种假设都不成立，则通过遍历找到新开始节点在老节点中的位置索引

            // 找到老节点中每个节点 key 和 索引之间的关系映射 => oldKeyToIdx = { key1: idx1, ... }
            if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
            // 在映射中找到新开始节点在老节点中的位置索引
            idxInOld = isDef(newStartVnode.key)
                ? oldKeyToIdx[newStartVnode.key]
                : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
            if (isUndef(idxInOld)) { // New element
                // 在老节点中没找到新开始节点，则说明是新创建的元素，执行创建
                createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
            } else {
                // 在老节点中找到新开始节点了
                vnodeToMove = oldCh[idxInOld]
                if (sameVnode(vnodeToMove, newStartVnode)) {
                    // 如果这两个节点是同一个，则执行 patch
                    patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
                    // patch 结束后将该老节点置为 undefined
                    oldCh[idxInOld] = undefined
                    canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
                } else {
                    // 最后这种情况是，找到节点了，但是发现两个节点不是同一个节点，则视为新元素，执行创建
                    createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
                }
            }
            // 老节点向后移动一个
            newStartVnode = newCh[++newStartIdx]
        }
    }
    // 走到这里，说明老节点或者新节点被遍历完了
    if (oldStartIdx > oldEndIdx) {
        // 说明老节点被遍历完了，新节点有剩余，则说明这部分剩余的节点是新增的节点，然后添加这些节点
        refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
        addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
        // 说明新节点被遍历完了，老节点有剩余，说明这部分的节点被删掉了，则移除这些节点
        removeVnodes(oldCh, oldStartIdx, oldEndIdx)
    }
}
```

### 手撕diff

```javascript
/**
 * @name: 
 * @param {*} parentElm 父节点
 * @param {*} oldCh 老dom树
 * @param {*} newCh 新dom树
 * @param {*} insertedVnodeQueue 插入的节点队列
 * @param {*} removeOnly 有关transition-group
 * @return {*}
 */
function diff(parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    // 首先diff算法是从dom节点列表的两端开始，因此需要定义四个下标，分别指向老子树和新子树
    let oldStartIdx=0,oldEndIdx=oldCh.length-1,newStartIdx=0,newEndIdx=newCh.length-1;
    // 老节点中节点key与节点下标idx的映射
    let oldToIdx;
    // 在非生产环境下，需要检查节点的key是否重复
    if(process.env.NODE_ENV!=='production'){
        // 检查key是否重复
    }
    // 表示新老节点列表都没遍历完成
    while(oldStartIdx<=oldEndIdx&&newStartIdx<=newEndIdx){
        if(oldCh[oldStartIdx]==undefined){
            // 这里主要是因为被key比较过了
            oldStartIdx++;
        }else if(old[oldEndIdx]==undefined){
            oldEndIdx--;
        }
        else if(sameNode(oldCh[oldStartIdx],newCh[newStartIdx])){
            // patch过程

            // 指针右移
            oldStartIdx++;
            newStartIdx++;
        }else if(sameNode(oldCh[oldEndIdx],newCh[newEndIdx])){
            // patch过程
            
            // 指针左移
            oldEndIdx--;
            newEndIdx--;
        }else if(sameNode(oldCh[oldStartIdx],newCh[newEndIdx])){
            // patch过程

            // 指针右移，左移
            oldStartIdx++;
            newEndIdx--;
        }else if(sameNode(oldCh[oldEndIdx],newCh[newStartIdx])){
            // patch过程

            // 指针左移，右移
            oldEndIdx--;
            newStartIdx++;
        }else{
            // 比较key
            if(oldToIdx==undefined){
                // 创建映射关系
                /**
                 * 为什么不一开始就创建，因为在这个时候创建有一些dom已经完成了比较，不需要映射关系
                 * 可以降低内存和计算负担
                 */
            }
            // 在老dom中寻找新dom的key对应的节点
            let key;
            if(key==undefined){
                // key不存在，就是新创建的元素
            }else{
                // key存在，判断是否是相同节点
                if(sameNode()){
                    // patch过程
                    // 将新节点在老节点中的位置对应的dom置为undefined
                }else{
                    // 不是同一个节点，创建新节点
                }
            }
            // 不管是那种情况，新节点下标都要后移
            newStartIdx++;
        }
    }
    // 走到这里，说明老节点和新节点有一个已经遍历完成
    if(oldStartIdx>oldEndIdx){
        // 老节点走完了，为新节点添加剩余的节点
    }else if(newStartIdx>newEndIdx){
        // 新节点走完了，移除多余的老节点
    }
}

function sameNode(oldVnode,newVnode){
    return oldVnode===newVnode;
}
```




































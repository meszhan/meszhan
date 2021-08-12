# patch

https://juejin.cn/post/6964141635856760868#heading-10

### 前言

前面说到了，当组件更新的时候，实例化渲染watcher是传递的updateComponent会被执行

```javascript
const updateComponent = () => {
  // 执行 vm._render() 函数，得到 虚拟 VNode，并将 VNode 传递给 vm._update 方法，接下来就该到 patch 阶段了
  vm._update(vm._render(), hydrating)
}
```

首先执行vm._render函数，得到组件的VNode，并将VNode传递给vm._update方法，最后进入patch阶段

### 历史

### 目标

深入理解Vue的patch阶段，理解其diff算法的原理

### 源码解读

#### 入口

> /src/core/instance/lifecycle.js

```javascript
const updateComponent = () => {
  // 执行 vm._render() 函数，得到 虚拟 VNode，并将 VNode 传递给 vm._update 方法，接下来就该到 patch 阶段了
  vm._update(vm._render(), hydrating)
}
```

#### vm._update

> /src/core/instance/lifecycle.js

```javascript
/*
	* 页面首次渲染和后续更新的入口位置_update，也是patch的入口位置
*/
export function lifecycleMixin (Vue: Class<Component>) {
  Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    //  页面挂载点，真实的元素
    const prevEl = vm.$el
    // 旧的VNode
    const prevVnode = vm._vnode
    const restoreActiveInstance = setActiveInstance(vm)
    // 新的VNode
    vm._vnode = vnode
    // Vue.prototype.__patch__ is injected in entry points
    // based on the rendering backend used.
    if (!prevVnode) {
      // initial render
      // 老的VNode不存在，表示首次渲染，初始化页面走这里
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
      // 响应式数据更新时，更新页面走这里
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    restoreActiveInstance()
    // update __vue__ reference
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
    // updated hook is called by the scheduler to ensure that children are
    // updated in a parent's updated hook.
  }

  Vue.prototype.$forceUpdate = function () {
    const vm: Component = this
    if (vm._watcher) {
      vm._watcher.update()
    }
  }

  Vue.prototype.$destroy = function () {
    const vm: Component = this
    if (vm._isBeingDestroyed) {
      return
    }
    callHook(vm, 'beforeDestroy')
    vm._isBeingDestroyed = true
    // remove self from parent
    const parent = vm.$parent
    if (parent && !parent._isBeingDestroyed && !vm.$options.abstract) {
      remove(parent.$children, vm)
    }
    // teardown watchers
    if (vm._watcher) {
      vm._watcher.teardown()
    }
    let i = vm._watchers.length
    while (i--) {
      vm._watchers[i].teardown()
    }
    // remove reference from data ob
    // frozen object may not have observer.
    if (vm._data.__ob__) {
      vm._data.__ob__.vmCount--
    }
    // call the last hook...
    vm._isDestroyed = true
    // invoke destroy hooks on current rendered tree
    vm.__patch__(vm._vnode, null)
    // fire destroyed hook
    callHook(vm, 'destroyed')
    // turn off all instance listeners.
    vm.$off()
    // remove __vue__ reference
    if (vm.$el) {
      vm.$el.__vue__ = null
    }
    // release circular reference (#6759)
    if (vm.$vnode) {
      vm.$vnode.parent = null
    }
  }
}
```

#### vm._patch_

> /src/platforms/web/runtime/index.js

```javascript
// 在 Vue 原型链上安装 web 平台的 patch 函数
Vue.prototype.__patch__ = inBrowser ? patch : noop
```

#### patch

> /src/platforms/web/runtime/patch.js

```javascript
// patch 工厂函数，为其传入平台特有的一些操作，然后返回一个 patch 函数
export const patch: Function = createPatchFunction({ nodeOps, modules })
```

#### nodeOps

> /src/platforms/web/runtime/node-ops.js

```javascript
/*
	* web平台的DOM操作 API
*/
// 创建标签名为tagName的元素节点
export function createElement (tagName: string, vnode: VNode): Element {
  // 创建元素节点
  const elm = document.createElement(tagName)
  if (tagName !== 'select') {
    return elm
  }
  // false or null will remove the attribute but undefined will not
  // 如果是select元素，设置multiple属性
  if (vnode.data && vnode.data.attrs && vnode.data.attrs.multiple !== undefined) {
    elm.setAttribute('multiple', 'multiple')
  }
  return elm
}
// 创建带命名空间的元素节点
export function createElementNS (namespace: string, tagName: string): Element {
  return document.createElementNS(namespaceMap[namespace], tagName)
}

// 创建文本节点
export function createTextNode (text: string): Text {
  return document.createTextNode(text)
}
// 创建注释节点
export function createComment (text: string): Comment {
  return document.createComment(text)
}
// 在指定节点前插入节点
export function insertBefore (parentNode: Node, newNode: Node, referenceNode: Node) {
  parentNode.insertBefore(newNode, referenceNode)
}
// 移除指定子节点
export function removeChild (node: Node, child: Node) {
  node.removeChild(child)
}
// 添加子节点
export function appendChild (node: Node, child: Node) {
  node.appendChild(child)
}
// 返回指定节点的父节点
export function parentNode (node: Node): ?Node {
  return node.parentNode
}
// 返回指定节点的下一个兄弟节点
export function nextSibling (node: Node): ?Node {
  return node.nextSibling
}
// 返回指定节点的标签名
export function tagName (node: Element): string {
  return node.tagName
}
// 为指定节点设置文本
export function setTextContent (node: Node, text: string) {
  node.textContent = text
}
// 为节点设置指定的scopeId属性，属性值为’‘
export function setStyleScope (node: Element, scopeId: string) {
  node.setAttribute(scopeId, '')
}
```

#### modules

> /src/platforms/web/runtime/modules 和 /src/core/vdom/modules

平台特有的一些操作，比如：attr、class、style、event等，还有核心的directive和ref，它们会向外暴露一些特有的方法，比如create、activate、update、remove、destroy，这些方法在patch阶段会被调用，从而做相应的操作，比如创建attr、指令等。

#### createPatchFunction

> /src/core/vdom/patch.js

```javascript
/**
 * vm.__patch__
 *   1、新节点不存在，老节点存在，调用 destroy，销毁老节点
 *   2、如果 oldVnode 是真实元素，则表示首次渲染，创建新节点，并插入 body，然后移除老节点
 *   3、如果 oldVnode 不是真实元素，则表示更新阶段，执行 patchVnode
 */
function patch(oldVnode, vnode, hydrating, removeOnly) {
  // 如果新节点不存在，老节点存在，则调用 destroy，销毁老节点
  if (isUndef(vnode)) {
    if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
    return
  }

  let isInitialPatch = false
  const insertedVnodeQueue = []

  if (isUndef(oldVnode)) {
    // 新的 VNode 存在，老的 VNode 不存在，这种情况会在一个组件初次渲染的时候出现，比如：
    // <div id="app"><comp></comp></div>
    // 这里的 comp 组件初次渲染时就会走这儿
    // empty mount (likely as component), create new root element
    isInitialPatch = true
    createElm(vnode, insertedVnodeQueue)
  } else {
    // 判断 oldVnode 是否为真实元素
    const isRealElement = isDef(oldVnode.nodeType)
    if (!isRealElement && sameVnode(oldVnode, vnode)) {
      // 不是真实元素，但是老节点和新节点是同一个节点，则是更新阶段，执行 patch 更新节点
      patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
    } else {
      // 是真实元素，则表示初次渲染
      if (isRealElement) {
        // 挂载到真实元素以及处理服务端渲染的情况
        // mounting to a real element
        // check if this is server-rendered content and if we can perform
        // a successful hydration.
        if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
          oldVnode.removeAttribute(SSR_ATTR)
          hydrating = true
        }
        if (isTrue(hydrating)) {
          if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
            invokeInsertHook(vnode, insertedVnodeQueue, true)
            return oldVnode
          } else if (process.env.NODE_ENV !== 'production') {
            warn(
              'The client-side rendered virtual DOM tree is not matching ' +
              'server-rendered content. This is likely caused by incorrect ' +
              'HTML markup, for example nesting block-level elements inside ' +
              '<p>, or missing <tbody>. Bailing hydration and performing ' +
              'full client-side render.'
            )
          }
        }
        // 走到这儿说明不是服务端渲染，或者 hydration 失败，则根据 oldVnode 创建一个 vnode 节点
        // either not server-rendered, or hydration failed.
        // create an empty node and replace it
        oldVnode = emptyNodeAt(oldVnode)
      }

      // 拿到老节点的真实元素
      const oldElm = oldVnode.elm
      // 获取老节点的父元素，即 body
      const parentElm = nodeOps.parentNode(oldElm)

      // 基于新 vnode 创建整棵 DOM 树并插入到 body 元素下
      createElm(
        vnode,
        insertedVnodeQueue,
        // extremely rare edge case: do not insert if old element is in a
        // leaving transition. Only happens when combining transition +
        // keep-alive + HOCs. (#4590)
        oldElm._leaveCb ? null : parentElm,
        nodeOps.nextSibling(oldElm)
      )

      // 递归更新父占位符节点元素
      if (isDef(vnode.parent)) {
        let ancestor = vnode.parent
        const patchable = isPatchable(vnode)
        while (ancestor) {
          for (let i = 0; i < cbs.destroy.length; ++i) {
            cbs.destroy[i](ancestor)
          }
          ancestor.elm = vnode.elm
          if (patchable) {
            for (let i = 0; i < cbs.create.length; ++i) {
              cbs.create[i](emptyNode, ancestor)
            }
            // #6513
            // invoke insert hooks that may have been merged by create hooks.
            // e.g. for directives that uses the "inserted" hook.
            const insert = ancestor.data.hook.insert
            if (insert.merged) {
              // start at index 1 to avoid re-invoking component mounted hook
              for (let i = 1; i < insert.fns.length; i++) {
                insert.fns[i]()
              }
            }
          } else {
            registerRef(ancestor)
          }
          ancestor = ancestor.parent
        }
      }

      // 移除老节点
      if (isDef(parentElm)) {
        removeVnodes([oldVnode], 0, 0)
      } else if (isDef(oldVnode.tag)) {
        invokeDestroyHook(oldVnode)
      }
    }
  }

  invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
  return vnode.elm
}
```




























































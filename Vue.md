# Vue

### Vue和React的区别

#### 相同点

+ 都使用虚拟dom
+ 使用组件化思想，流程基本一致
+ 都是响应式，提倡单向数据流

Vue和React都是使用Virtual DOM+diff算法，不管是Vue的模版还是React的类组件和函数组件，底层最终都是为了生成render函数服务，render函数执行后返回VNode来描述真实DOM。当每一次UI更新时，总会根据render函数生成最新的VNode，然后根老的VNode进行比较，使用diff算法真正更新真实DOM。因为虚拟DOM是JS对象结构，在JS引擎中，而真实DOM在浏览器的渲染引擎中，操作虚拟DOM的代价比真实DOM小得多

##### 虚拟DOM的好处

+ 减少直接操作DOM，屏蔽了底层DOM操作，减少频繁的更新DOM，数据驱动视图更新
+ 为函数式编程提供可能
+ 提供跨平台，可以渲染到web之外的平台，比如RN或Weex

#### 不同点

##### 核心思想

+ Vue早期定位是尽可能的降低前端开发的门槛，所以Vue推崇灵活易用渐进式开发、数据可变（可以直接修改data）、双向绑定
+ React推崇函数式编程，也就是纯组件，数据不可变（不能直接修改state，要用新的state替换旧的）和单向数据流，函数式编程的好处是无副作用且可测试性（纯函数）

##### 响应式原理不同

+ Vue
  + 依赖收集、自动优化、数据可变
  + 递归监听data的所有属性，直接修改
  + 数据改变时，自动找到依赖的组件重新渲染
+ React
  + 基于状态的，需要手动优化，数据不可变，需要setState驱动新的state替换老的state
  + 数据改变时，以组件为根目录，默认全部重新渲染

##### diff算法不同

两者流程思维相似，都基于两个假设，将算法复杂度降为O(n)

+ 不同的组件产生不同的DOM结构，当type不同时，对应DOM操作就是直接销毁老的DOM，创建新的DOM
+ 同一层次的一组子节点，通过唯一的key区分

差别：

+ Vue基于snabbdom库，使用双向链表，边对比边更新DOM
+ React主要使用diff队列保存需要更新哪些DOM，得到patch树后再统一操作批量更新DOM

![](https://user-images.githubusercontent.com/6310131/58315009-41998700-7e43-11e9-8c52-438adad9b23b.png)

##### 事件机制不同

+ Vue
  + 原生事件使用标准的web事件
  + 自定义事件机制用于组件之间通信，基于发布订阅模式
+ React
  + 包装原生事件，将所有事件都冒泡到顶层document监听，然后合成事件下发，可以跨端使用事件机制
  + React组件上无事件，父子组件通信使用props

##### 流程图

+ Vue

  <img src="https://user-images.githubusercontent.com/6310131/58315972-1dd74080-7e45-11e9-94bc-b494d41ae61c.png" style="zoom:200%;" />

+ React

  ![](https://user-images.githubusercontent.com/6310131/58316112-6b53ad80-7e45-11e9-8b2a-d31bfaf269aa.png)

### 为什么不用index或随机数作为key

快速总结：

+ 使用index作为key会造成VNode的错误复用
+ 使用random作为key，导致因为两次生成的随机数都不一样导致无法复用任何节点

#### 例子

```vue
<template>
  <div id="app">
    <div v-for="(item, index) in data" :key="index">
      <Child />
      <button @click="handleDelete(index)">删除这一行</button>
    </div>
  </div>
</template>

<script>

export default {
  name: "App",
  components: {
    Child: {
      template: '<span>{{name}}{{Math.floor(Math.random() * 1000)}}</span>',
      props: ['name']
    }
  },
  data() {
    return {
      data: [
        { name: "小明" },
        { name: "小红" },
        { name: "小蓝" },
        { name: "小紫" },
      ]
    };
  },
  methods: {
    handleDelete(index) {
      this.data.splice(index, 1);
    },
  }
};
</script>
```

在这里面，即使我们删除的不是最后一行，但是每次都是最后一行被删除了，所以我们需要进一步了解一下diff的流程

#### diff算法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc6584989d934b57ba795f1904122ef1~tplv-k3u1fbpfcp-watermark.awebp)

通常来说，Vue的diff流程就是patchVNode，其中updateChildren就是同层比较，也就是比较新旧节点树

Vue会声明新旧四个指针变量，分别记录新旧VNode同层数组的首尾索引，通过索引的移动，根据顺序依次比较新旧VNode，如果不能命中sameVnode，则将oldVnode.key维护为一个map，继续查询是否包含newVnode.key，如果可以根据key命中sameVnode，则递归的执行patchVnode。如果无法命中，说明没有可以复用的Vnode，创建新的dom节点

如果newVnode的首尾指针首先相遇，说明newVnode已经遍历完成，直接移除oldVnode多余部分，若oldVnode的首尾指针先相遇，说明oldVnode已经遍历完成，直接新增newVnode的多余部分。

#### 使用index作为key导致的问题

##### sameVnode方法

```javascript
function sameVnode (a, b) {
    return (
      a.key === b.key &&
      a.asyncFactory === b.asyncFactory && (
        (
          a.tag === b.tag &&
          a.isComment === b.isComment &&
          isDef(a.data) === isDef(b.data) &&
          sameInputType(a, b)
        ) || (
          isTrue(a.isAsyncPlaceholder) &&
          isUndef(b.asyncFactory.error)
        )
      )
    )
  }
```

对于用index作为key来说，不管我们删除数组中的哪条数据，数组都会重新排序，从而生成新的index

例如对于数组['1','2','3','4']，此时对应下标关系为：

+ ‘1’——0
+ ‘2’——1
+ ‘3’——2
+ ‘4’——3

如果我们删除3后，得到新的下标关系

+ ‘1’——0
+ ‘2’——1
+ ‘3‘——2

从上面可以看到，我们本来是想删除3但是实际删除的却是4，这是因为diff算法在根据key判断的时候0、1、2都满足了sameVnode，而此时因为新的dom树已经遍历完成，会将老的vnode数组的剩余项目丢弃

#### 使用index作为key导致的没问题

为什么开发中使用index很少出现问题？

```vue
<template>
  <div id="app">
    <div v-for="(item, index) in data" :key="index">
      <Child :name="`${item.name}`" />
      <button @click="handleDelete(index)">删除这一行</button>
    </div>
  </div>
</template>
```

我们给Child组件添加了一个props，回顾一下diff更新流程图

组件类型的Vnode，在patchVnode过程中会执行prePatch函数，给组件的propsData重新赋值，从而触发setter重新渲染子组件

##### 文本类型节点直接覆盖

```vue
<template>
  <div id="app">
    <div v-for="(item, index) in data" :key="index">
      <span>{{item.name}}</span>
      <button @click="handleDelete(index)">删除这一行</button>
    </div>
  </div>
</template>
```

我们可以看到上面并没有子组件，也就不存在props更新的问题，但是index作为key依然不会产生问题

因为文本类型的节点会直接覆盖

##### 为什么Child组件中的span不会直接覆盖

对于组件类型的vnode来说，只有在父组件里修改子组件的props后，才会引起子组件的重新render








































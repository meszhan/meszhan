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






























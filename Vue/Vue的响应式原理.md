# 数据响应式原理

### **响应式的对象**

> 首先，我们需要知道在组件中需要对谁做响应式处理。
>
> 很明显，对于一个组件实例来说，需要做响应式处理的肯定是属性（数据），因为所有的重新渲染都是由数据变化所引起的。
>
> 那么具体的是哪些属性或数据会被观察呢？
>
> + data对象或data函数返回的对象
> + data对象中的对象
> + props
>
> 为什么没有computed、watcher、inject和provide？
>
> + inject同上
> + computed和watch都是直接通过Watcher进行监控
> + provide提供的本身就是自己的已经观察的data

### **数据劫持**

> Vue2中的Object.defineProperty，这个方法会劫持所有key的getter和setter

### **响应式策略**

> getter中的具体行为？
>
> **依赖收集**
>
> 如果触发了getter，说明这个组件模版或某个调用中有对于这个组件的依赖，而我们又知道每一个组件实例都对应了一个Watcher实例。
>
> 因此在这里会进行依赖收集，在被触发了get的key的dep中添加此组件对应的Watcher，同时在Watcher中添加dep（目的是主动取消监听），表示Watcher在监听key的变更
>
> setter中的具体行为？
>
> + 观察新的被set的val
> + 通知watcher更新

### **data的数据响应式**

> + 调用observe方法为data对象打上__ob__标记后创建一个Observer实例
> + 如果对象中还有嵌套对象，会递归创建Observer实例，所以我们总可以看到data中定义的所有对象下面都会有__ob__属性表明其被观察
> + 在Observer实例中，会有一个Dep实例，用于发布事件
> + 如果观察的是数组，会对数组的7个方法进行增强，然后递归观察数组中的对象

### **computed的数据响应式**

> + 首先，根据Watcher实例的创建要求，获取到vm实例、getter方法（key对应的val或val的get属性值）、懒执行的配置项
> + 为computed配置项中的每个key添加了的对应的watcher实例后，就可以直接调用get方法获取返回值了

### **watch的数据响应式**

> 为每个handler都创建一个Watcher，而不是每个key，因为一个key可以有多个watch
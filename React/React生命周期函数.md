# React生命周期函数

https://juejin.cn/post/6914112105964634119

### 引言

> React从v16.3开始，对生命周期的钩子函数进行渐进式调整，废弃和新增了一些生命周期的钩子函数
>
> + 新旧生命周期函数对比
> + 分析为什么废弃旧的钩子函数
> + 详解新的生命周期使用场景
> + 实例

### 新旧生命周期对比

#### 旧的生命周期

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e12b2e35c8444f19b795b27e38f4c149~tplv-k3u1fbpfcp-watermark.image" style="zoom:40%;" />

> 分析上图，React组件的整个生命周期主要分为挂载、更新、卸载
>
> + 挂载：componentWillMount=>rener=>componentDidMount
> + 更新： componentWillReceiveProps=>shouldComponentUpdate=>componentWillUpdate=>render=>componentDidUpdate
> + 卸载： componentWillUnmount

#### 新的生命周期

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7d8676f379d4d96bbf0ebd9a8528594~tplv-k3u1fbpfcp-watermark.image" style="zoom:50%;" />

> 分析上面的生命周期，同样分为挂载、更新和卸载
>
> + 挂载：constructor=>getDerivedStateFromProps=>render=>componentDidMount
> + 更新：getDerivedStateFromProps=>shouldComponentUpdate=>rener=>getSnapshotBeforeUpdate=>componentDidUpdate
> + 卸载：componentWillUnMount

#### 对比

> 对比新旧生命周期
>
> + 废弃了旧的componentWillMount、componentWillReceiveProps、componentWillUpdate函数
> + 新增了getDerivedStateFromProps和getSnapshotBeforeUpdate

### 分析废弃原因

> 在旧的版本中，更新过程是同步的，因此如果主线程被占用，会导致页面的性能问题，因此Facebook发布了React Filber
>
> 机制：React Filber利用浏览器requestIdleCallback将可中断的任务进行分片处理，每个小片运行时间都很短，唯一的线程就不会独占
>
> ​			这在操作系统中叫时间片轮转调度
>
> 因为React Filber Reconciliation这个过程有可能暂停然后继续执行，所以挂载和更新之前的生命周期钩子有可能不执行或执行多次

### 详解新的生命周期

#### constructor

> constructor会在React组件挂载之前被调用，在为React.Component子类实现构造函数时，应在其他语句之前调用super()
>
> super会将父类的this对象继承给子类

React构造函数用于两种情况：

+ 初始化函数内部的state
+ 为事件处理函数绑定实例bind

> 如果不初始化state或不绑定方法，就不需要写constructor，只需要设置this.state
>
> 不能在constructor内部调用this.setState，因为此时render还未执行，DOM节点还未挂载

#### static getDerivedStateFromProps(nextProps, prevState)

> 在调用render方法之前调用，在初始化和后续的update过程中都会调用

**返回一个对象来更新state，如果返回null不更新任何内容**

> 参数：第一个参数为即将更新的props，第二个参数为上一个状态的state，可以比较props和state来加一些限制条件，防止无用的state更新
>
> getDerivedStateFromProps时一个静态函数，不能使用this，只能做一些无副作用的操作

#### render

> render方法时class组件唯一必须实现的方法，用于渲染dom，必须返回react-dom
>
> 不要在render里setState，因为会触发死循环

#### componentDidMount

> componentDidMount在组件挂载后立即调用，此时最应该发送网络请求、启用事件监听方法，可以在钩子函数里调用setState

#### shouldComponentUpdate(nextProps,nextState)

> shouldComponentUpdate在组件更新前调用，可以控制组件是否更新，返回true时组件更新，返回false不更新
>
> + nextProps：即将更新的props值
> + nextState：更新后的state值
> + 可以根据props和state的比较决定是否更新
>
> 不建议在shouldComponentUpdate中进行深层比较火使用JSON.stringify，这样十分影响效率和性能
>
> 同样不能调用setState，会导致循环调用更新
>
> 可以使用内置PureComponent组件替代

#### getSnapshotBeforeUpdate(prevProps, prevState)

> getSnapshotBeforeUpdate在最近一次的渲染输出被提交前调用，也就是render之后和组件挂载之前调用
>
> 可以使组件在DOM更新之前捕获一些信息例如滚动位置
>
> 这个生命周期返回的任何值都会作为参数传递给componentDidUpdate，如果不需要传递，返回null

#### componentDidUpdate(prevProps, prevState, snapshot)

> componentDidUpdate会在更新后被立即调用，首次渲染不会执行
>
> + prevProps：上一次props值
> + prevState：上一次state值
> + snapshot：参数传递

#### componentWillUnmount()

> 在组件即将被卸载后销毁时调用
>
> 可以用于取消请求、移除监听事件、清理DOM元素、清理定时器

### 父子组件生命周期执行顺序探索

#### 子组件修改自身状态

> 如果不修改父组件的状态，不会触发父组件的任何生命周期函数
>
> + getDerivedStateFromProps
> + shouldComponentUpdate
> + render
> + getSnapshotBeforeUpdate
> + componentDidUpdate

#### 修改父组件中传入给子组件的props

> 会影响到子组件的生命周期
>
> + getDerivedStateFromProps
> + shouldComponentUpdate
> + render
> + 子getDerivedStateFromProps
> + 子shouldComponentUpdate
> + 子render
> + 子getSnapshotBeforeUpdate
> + getSnapshotBeforeUpdate
> + 子componentDidUpdate
> + componentDidUpdate

#### 卸载子组件

> 需要更新父组件
>
> + getDerivedStateFromProps
> + shouldComponentUpdate
> + render
> + getSnapshotBeforeUpdate
> + 子componentWillUnmount
> + componentDidUpdate

#### 重新挂载子组件

> 同样会引起父组件的更新
>
> + getDerivedStateFromProps
> + shouldComponentUpdate
> + render
> + 子constructor
> + 子getDerivedStateFromProps
> + 子render
> + getSnapshotBeforeUpdate
> + 子componentDidMount
> + componentDidUpdate






































# React

### React生命周期函数

https://juejin.cn/post/6914112105964634119

#### 引言

React从v16.3开始，对生命周期的钩子函数进行渐进式调整，废弃和新增了一些生命周期的钩子函数

+ 新旧生命周期函数对比
+ 分析为什么废弃旧的钩子函数
+ 详解新的生命周期使用场景
+ 实例

#### 新旧生命周期对比

##### 旧的生命周期

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e12b2e35c8444f19b795b27e38f4c149~tplv-k3u1fbpfcp-watermark.image" style="zoom:40%;" />

分析上图，React组件的整个生命周期主要分为挂载、更新、卸载

+ 挂载：componentWillMount=>rener=>componentDidMount
+ 更新： componentWillReceiveProps=>shouldComponentUpdate=>componentWillUpdate=>render=>componentDidUpdate
+ 卸载： componentWillUnmount

##### 新的生命周期

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7d8676f379d4d96bbf0ebd9a8528594~tplv-k3u1fbpfcp-watermark.image" style="zoom:50%;" />

分析上面的生命周期，同样分为挂载、更新和卸载

+ 挂载：constructor=>getDerivedStateFromProps=>render=>componentDidMount
+ 更新：getDerivedStateFromProps=>shouldComponentUpdate=>rener=>getSnapshotBeforeUpdate=>componentDidUpdate
+ 卸载：componentWillUnMount

##### 对比

对比新旧生命周期

+ 废弃了旧的componentWillMount、componentWillReceiveProps、componentWillUpdate函数
+ 新增了getDerivedStateFromProps和getSnapshotBeforeUpdate

#### 分析废弃原因

在旧的版本中，更新过程是同步的，因此如果主线程被占用，会导致页面的性能问题，因此Facebook发布了React Filber

机制：React Filber利用浏览器requestIdleCallback将可中断的任务进行分片处理，每个小片运行时间都很短，唯一的线程就不会独占

​			这在操作系统中叫时间片轮转调度

因为React Filber Reconciliation这个过程有可能暂停然后继续执行，所以挂载和更新之前的生命周期钩子有可能不执行或执行多次

#### 详解新的生命周期

##### constructor

constructor会在React组件挂载之前被调用，在为React.Component子类实现构造函数时，应在其他语句之前调用super()

super会将父类的this对象继承给子类

React构造函数用于两种情况：

+ 初始化函数内部的state
+ 为事件处理函数绑定实例bind

如果不初始化state或不绑定方法，就不需要写constructor，只需要设置this.state

不能在constructor内部调用this.setState，因为此时render还未执行，DOM节点还未挂载

##### static getDerivedStateFromProps(nextProps, prevState)

在调用render方法之前调用，在初始化和后续的update过程中都会调用

**返回一个对象来更新state，如果返回null不更新任何内容**

参数：第一个参数为即将更新的props，第二个参数为上一个状态的state，可以比较props和state来加一些限制条件，防止无用的state更新

getDerivedStateFromProps时一个静态函数，不能使用this，只能做一些无副作用的操作

##### render

render方法时class组件唯一必须实现的方法，用于渲染dom，必须返回react-dom

不要在render里setState，因为会触发死循环

##### componentDidMount

componentDidMount在组件挂载后立即调用，此时最应该发送网络请求、启用事件监听方法，可以在钩子函数里调用setState

##### shouldComponentUpdate(nextProps,nextState)

shouldComponentUpdate在组件更新前调用，可以控制组件是否更新，返回true时组件更新，返回false不更新

+ nextProps：即将更新的props值
+ nextState：更新后的state值
+ 可以根据props和state的比较决定是否更新

不建议在shouldComponentUpdate中进行深层比较火使用JSON.stringify，这样十分影响效率和性能

同样不能调用setState，会导致循环调用更新

可以使用内置PureComponent组件替代

##### getSnapshotBeforeUpdate(prevProps, prevState)

getSnapshotBeforeUpdate在最近一次的渲染输出被提交前调用，也就是render之后和组件挂载之前调用

可以使组件在DOM更新之前捕获一些信息例如滚动位置

这个生命周期返回的任何值都会作为参数传递给componentDidUpdate，如果不需要传递，返回null

##### componentDidUpdate(prevProps, prevState, snapshot)

componentDidUpdate会在更新后被立即调用，首次渲染不会执行

+ prevProps：上一次props值
+ prevState：上一次state值
+ snapshot：参数传递

##### componentWillUnmount()

在组件即将被卸载后销毁时调用

可以用于取消请求、移除监听事件、清理DOM元素、清理定时器

#### 父子组件生命周期执行顺序探索

##### 子组件修改自身状态

如果不修改父组件的状态，不会触发父组件的任何生命周期函数

+ getDerivedStateFromProps
+ shouldComponentUpdate
+ render
+ getSnapshotBeforeUpdate
+ componentDidUpdate

##### 修改父组件中传入给子组件的props

会影响到子组件的生命周期

+ getDerivedStateFromProps
+ shouldComponentUpdate
+ render
+ 子getDerivedStateFromProps
+ 子shouldComponentUpdate
+ 子render
+ 子getSnapshotBeforeUpdate
+ getSnapshotBeforeUpdate
+ 子componentDidUpdate
+ componentDidUpdate

##### 卸载子组件

需要更新父组件

+ getDerivedStateFromProps
+ shouldComponentUpdate
+ render
+ getSnapshotBeforeUpdate
+ 子componentWillUnmount
+ componentDidUpdate

##### 重新挂载子组件

同样会引起父组件的更新

+ getDerivedStateFromProps
+ shouldComponentUpdate
+ render
+ 子constructor
+ 子getDerivedStateFromProps
+ 子render
+ getSnapshotBeforeUpdate
+ 子componentDidMount
+ componentDidUpdate

### React Class组件中请求可以在componentWillMount中发起吗

首先，在React16之后，已经废除了componentWillMount钩子函数（一同废除的还有componentWillUpdate和componentWillReceiveProps），原因是React Filber利用了浏览器的requestIdleCallback机制对可中断的任务进行分时间片处理，这会导致挂载前和更新前的钩子函数不执行或执行多次

+ 如果是SSR会拿不到数据
+ componentWillMount方法的调用在constructor之后，在render之前，在这方法里的代码调用setState方法不会触发重渲染，所以一般不会用来加载数据
+ 一般从服务端请求得到的数据都会与组件上的数据加载有关，所以应该在componentDidMount中发起请求

### React Class组件和React Hook的区别

https://juejin.cn/post/6844904179136200712

#### React Hooks解决的问题

+ 函数组件中不能拥有自己的state，在hooks之前函数组件是无状态的，都是通过props来获取父组件的状态，但是hooks提供了useState维护函数组件内部的状态
+ 函数组件不能监听组件的生命周期，useEffect聚合了多个生命周期函数
+ class组件中生命周期复杂
+ class组件难以复用，需要使用HOC或render props

#### Hook的好处

+ 写法上更加简洁，不需要constructor和setState改变状态，useState就包含了状态的初始化和setter
+ 业务代码更加聚合，使用class组件经常会出现一个逻辑的代码分布在两个生命周期内，但是useEffect可以聚合代码，比如创建和销毁定时器
+ 逻辑复用更加方便，class组件的复用采用render props和HOC，这样会增加组件树的层数和渲染负担，hooks提供了自定义的hooks来复用逻辑

#### 常见Hook

**useState**

维护组件的状态

**useEffect**

用useEffect可以模仿生命周期，还可以聚合逻辑代码

+ componentDidMount：将第二个参数设为空数组[]，代表不监听任何属性的更新
+ componentDidUpdate：不传入第二个参数，监听所有属性的更新
+ componentWillUnmount：useEffect传入的函数返回一个函数，在组件销毁时执行

**useMemo/useCallback**

useMemo和useCallback主要用于减少组件的更新次数，优化组件性能

+ useMemo接受一个回调函数以及依赖项，只有依赖项变化时才会重新执行回调函数
+ useCallback接收一个回调函数以及依赖项，并且返回该回调函数的memorize版本，只有在依赖项重新变化时才会产生新的memorize版本

在优化组件性能时，class组件一般使用React.PureComponent，PureComponent会在shouldUpdate进行一次浅比较，判断是否需要更新；针对函数组件一般使用React.memo。

但是需要注意的是，React Hooks的每一次渲染更新都是独立的，也就是说这一次的状态和上一次的状态即使是同一个变量甚至是同一个值，它们也是不同的，因此即使子组件使用了React.memo，但是因为父组件会给子组件传递props，而父组件传递过去的props是会随着渲染而变化的，因此子组件也会重新渲染，导致React.memo失效

使用useCallback后，只有当useCallback的依赖项发生变化时，才会生成memorize函数，所以改变父组件的状态时不会引起子组件的变化

**useRef**

useRef类似于React.createRef

useRef返回一个可变的ref对象，current属性被初始化为传入的参数

```javascript
const node=useRef(null)
<input ref={node}/>
```

这样就可以通过node.current属性访问到该DOM元素

useRef创建的对象在组件的整个生命周期内都保持不变，每次渲染函数组件时，返回的ref对象都是同一个

React.createRef每次重新渲染组件都会创建ref

**useReducer**

类似于redux中的reducer

useReducer传入一个计算函数和初始化state，通过返回的state可以访问状态，通过dispatch可以修改状态

**useContext**

通过useContext可以方便获取上层组件提供的context

**useLayoutEffect**

useEffect在全部渲染完毕后执行，useLayoutEffect会在浏览器layout之后，painting之前执行

会阻塞DOM，可以用它读取DOM布局并同步触发重渲染

#### React中的HOC和自定义Hook的优缺点

+ HOC就是把一个组件当作参数传入，再返回一个新的组件，会导致组件的层次变深
+ 自定义Hook可以不使用HOC的情况下进行复用

### useState和useEffect运行原理

#### useState

```javascript
function App() {
    const [n,setN] = React.useState(0);
    return (
        <div className="App">
            <p>{n}</p>
            <p>
                <button onClick={() => setN(n + 1)}+1</button>
            </p>
        </div>
        );
ReactDOM.render(<App />, rootElement);
```

看上面的代码，

+ 首次渲染时，会调用App函数，返回一个虚拟DOM，产生一个真实DOM
+ 点击按钮后，调用setN，此时因为状态改变，触发组件更新，App函数再次被调用
+ 使用diff算法和patch过程更新虚拟真实DOM
+ 很明显App方法每次更新都会被调用，因此useState会被多次调用，那么每次调用有什么区别呢

### Redux

Redux是JavaScript状态容器，提供可预测化的状态管理

#### 介绍

##### 动机

JavaScript需要管理比任何时候都要多的state状态，state包括服务器响应、缓存数据、本地生成尚未持久化到服务器的数据，也包括UI的状态，比如激活的路由、被选中的标签，是否显示动态加载动效或分页器

管理不断变化的state非常困难，如果一个model的变化会引起另一个model变化，那么view变化时可能引起对应model以及另一个model的变化，可能会引起另一个view的变化，state逐渐变得不受控制

##### 核心概念

Redux本身很简单，当使用普通对象来描述应用的state时，

```
{
  todos: [{
    text: 'Eat food',
    completed: true
  }, {
    text: 'Exercise',
    completed: false
  }],
  visibilityFilter: 'SHOW_COMPLETED'
}
```

这个对象就是一个model，区别是它没有setter方法，不能随意的修改它

如果想要更新state的数据，需要发起一个action，Action是一个普通的JavaScript对象

```javascript
{ type: 'ADD_TODO', text: 'Go to swimming pool' }
{ type: 'TOGGLE_TODO', index: 1 }
{ type: 'SET_VISIBILITY_FILTER', filter: 'SHOW_ALL' }
```

强制使用action来描述所有变化的好处就是可以清晰的知道应用中到底发生了什么

action是描述发生了什么的指示器，为了把action和state串起来，开发一些函数，也就是reducer

reducer只是一个接收state和action并返回新的state的函数

对于大的应用来说，我们需要编写很多小函数来分别管理state的一部分

```
function visibilityFilter(state = 'SHOW_ALL', action) {
  if (action.type === 'SET_VISIBILITY_FILTER') {
    return action.filter;
  } else {
    return state;
  }
}

function todos(state = [], action) {
  switch (action.type) {
  case 'ADD_TODO':
    return state.concat([{ text: action.text, completed: false }]);
  case 'TOGGLE_TODO':
    return state.map((todo, index) =>
      action.index === index ?
        { text: todo.text, completed: !todo.completed } :
        todo
   )
  default:
    return state;
  }
}
```

再开发一个reducer调用这两个reducer，进而来管理整个应用的state

#### 三大原则

#### 先前技术

#### 生态系统

#### 示例












































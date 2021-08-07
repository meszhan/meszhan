# 面试小tips

### React Class组件中请求可以在componentWillMount中发起吗

> 首先，在React16之后，已经废除了componentWillMount钩子函数（一同废除的还有componentWillUpdate和componentWillReceiveProps），原因是React Filber利用了浏览器的requestIdleCallback机制对可中断的任务进行分时间片处理，这会导致挂载前和更新前的钩子函数不执行或执行多次

> + 如果是SSR会拿不到数据
> + componentWillMount方法的调用在constructor之后，在render之前，在这方法里的代码调用setState方法不会触发重渲染，所以一般不会用来加载数据
> + 一般从服务端请求得到的数据都会与组件上的数据加载有关，所以应该在componentDidMount中发起请求

### React Class组件和React Hook的区别

https://juejin.cn/post/6844904179136200712

#### React Hooks解决的问题

> + 函数组件中不能拥有自己的state，在hooks之前函数组件是无状态的，都是通过props来获取父组件的状态，但是hooks提供了useState维护函数组件内部的状态
> + 函数组件不能监听组件的生命周期，useEffect聚合了多个生命周期函数
> + class组件中生命周期复杂
> + class组件难以复用，需要使用HOC或render props

#### Hook的好处

+ 写法上更加简洁，不需要constructor和setState改变状态，useState就包含了状态的初始化和setter
+ 业务代码更加聚合，使用class组件经常会出现一个逻辑的代码分布在两个生命周期内，但是useEffect可以聚合代码，比如创建和销毁定时器
+ 逻辑复用更加方便，class组件的复用采用render props和HOC，这样会增加组件树的层数和渲染负担，hooks提供了自定义的hooks来复用逻辑

#### 常见Hook

**useState**

> 维护组件的状态

**useEffect**

> 用useEffect可以模仿生命周期，还可以聚合逻辑代码
>
> + componentDidMount：将第二个参数设为空数组[]，代表不监听任何属性的更新
> + componentDidUpdate：不传入第二个参数，监听所有属性的更新
> + componentWillUnmount：useEffect传入的函数返回一个函数，在组件销毁时执行

**useMemo/useCallback**

> useMemo和useCallback主要用于减少组件的更新次数，优化组件性能
>
> + useMemo接受一个回调函数以及依赖项，只有依赖项变化时才会重新执行回调函数
> + useCallback接收一个回调函数以及依赖项，并且返回该回调函数的memorize版本，只有在依赖项重新变化时才会产生新的memorize版本
>
> 在优化组件性能时，class组件一般使用React.PureComponent，PureComponent会在shouldUpdate进行一次浅比较，判断是否需要更新；针对函数组件一般使用React.memo。
>
> 但是需要注意的是，React Hooks的每一次渲染更新都是独立的，也就是说这一次的状态和上一次的状态即使是同一个变量甚至是同一个值，它们也是不同的，因此即使子组件使用了React.memo，但是因为父组件会给子组件传递props，而父组件传递过去的props是会随着渲染而变化的，因此子组件也会重新渲染，导致React.memo失效
>
> 
>
> 使用useCallback后，只有当useCallback的依赖项发生变化时，才会生成memorize函数，所以改变父组件的状态时不会引起子组件的变化

**useRef**

> useRef类似于React.createRef
>
> useRef返回一个可变的ref对象，current属性被初始化为传入的参数

```
const node=useRef(null)
<input ref={node}/>
```

> 这样就可以通过node.current属性访问到该DOM元素
>
> useRef创建的对象在组件的整个生命周期内都保持不变，每次渲染函数组件时，返回的ref对象都是同一个
>
> React.createRef每次重新渲染组件都会创建ref

**useReducer**

> 类似于redux中的reducer
>
> useReducer传入一个计算函数和初始化state，通过返回的state可以访问状态，通过dispatch可以修改状态

**useContext**

> 通过useContext可以方便获取上层组件提供的context

**useLayoutEffect**

> useEffect在全部渲染完毕后执行，useLayoutEffect会在浏览器layout之后，painting之前执行
>
> 会阻塞DOM，可以用它读取DOM布局并同步触发重渲染

#### React中的HOC和自定义Hook的优缺点

> + HOC就是把一个组件当作参数传入，再返回一个新的组件，会导致组件的层次变深
> + 自定义Hook可以不使用HOC的情况下进行复用

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

> 看上面的代码，
>
> + 首次渲染时，会调用App函数，返回一个虚拟DOM，产生一个真实DOM
> + 点击按钮后，调用setN，此时因为状态改变，触发组件更新，App函数再次被调用
> + 使用diff算法和patch过程更新虚拟真实DOM
> + 很明显App方法每次更新都会被调用，因此useState会被多次调用，那么每次调用有什么区别呢






























































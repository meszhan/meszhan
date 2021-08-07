# Redux

> Redux是JavaScript状态容器，提供可预测化的状态管理

### 介绍

#### 动机

> JavaScript需要管理比任何时候都要多的state状态，state包括服务器响应、缓存数据、本地生成尚未持久化到服务器的数据，也包括UI的状态，比如激活的路由、被选中的标签，是否显示动态加载动效或分页器
>
> 管理不断变化的state非常困难，如果一个model的变化会引起另一个model变化，那么view变化时可能引起对应model以及另一个model的变化，可能会引起另一个view的变化，state逐渐变得不受控制

#### 核心概念

> Redux本身很简单，当使用普通对象来描述应用的state时，

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

> 这个对象就是一个model，区别是它没有setter方法，不能随意的修改它
>
> 如果想要更新state的数据，需要发起一个action，Action是一个普通的JavaScript对象

```javascript
{ type: 'ADD_TODO', text: 'Go to swimming pool' }
{ type: 'TOGGLE_TODO', index: 1 }
{ type: 'SET_VISIBILITY_FILTER', filter: 'SHOW_ALL' }
```

> 强制使用action来描述所有变化的好处就是可以清晰的知道应用中到底发生了什么
>
> action是描述发生了什么的指示器，为了把action和state串起来，开发一些函数，也就是reducer
>
> reducer只是一个接收state和action并返回新的state的函数
>
> 对于大的应用来说，我们需要编写很多小函数来分别管理state的一部分

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

> 再开发一个reducer调用这两个reducer，进而来管理整个应用的state

#### 三大原则

#### 先前技术

#### 生态系统

#### 示例












































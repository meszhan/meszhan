# MVC、MVVM、MVP的异同

https://juejin.cn/post/6844903480126078989

>  首先我们可以观察到它们的共同点，都有MV（Model-View），而不同点在于C控制器、VM（View-Model）和P（Presenter）

### Model&View

#### Model

> Model层封装了和应用程序的业务逻辑相关的数据以及对数据的处理方法
>
> 比如我们将需要用到的数值变量封装在Model中，并定义add、sub、getVal三种操作数值方法

```javascript
var myapp = {}; // 创建这个应用对象

myapp.Model = function() {
    var val = 0; // 需要操作的数据

    /* 操作数据的方法 */
    this.add = function(v) {
        if (val < 100) val += v;
    };

    this.sub = function(v) {
        if (val > 0) val -= v;
    };

    this.getVal = function() {
        return val;
    };
};
```

#### View

> View作为视图层，主要负责数据的展示

```javascript
myapp.View = function() {

    /* 视图元素 */
    var $num = $('#num'),
        $incBtn = $('#increase'),
        $decBtn = $('#decrease');

    /* 渲染数据 */
    this.render = function(model) {
        $num.text(model.getVal() + 'rmb');
    };
};
```

> 在上面，我们通过Model&View完成了数据从模型层到视图层到逻辑。但是对于一个应用程序而言，我们还需要响应用户的操作，同步的更新VIew和Model。
>
> 因此我们在MVC中引入了控制器Controller，用它定义用户界面对用户输入的响应方式，连接模型和视图，用于控制应用程序的流程，处理用户行为和数据上的改变

### MVC

> MVC允许在不改变视图的情况喜爱改变视图对用户输入对用户输入的响应方式，用户对View的操作交给了Controller处理
>
> 在Controller中响应View的事件调用Model的接口对数据进行操作，一旦Model发生变化便通知相关视图进行更新
















































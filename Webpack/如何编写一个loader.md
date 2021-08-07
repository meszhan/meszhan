# 如何编写一个loader

https://segmentfault.com/a/1190000021657031

### Loader

> 在了解如何开发一个loader时，我们首先要了解一下loader

#### 作用

> + 用于对模块的源代码进行转换，可以使我们在import或load加载模块时预处理文件，因此loader类似于其他构建工具中的task
>
> + 可以将文件从不同的语言转换为JS或将内联图像转换为data URL
> + 再举一个例子，可以在JS模块中import CSS文件，因为文件会被作处理，将对应的源代码包装为style标签插入到head中

#### 示例

> 不管是使用上面loader，都需要使用npm安装，比如css-loader或ts-loader

#### 使用loader

+ 配置方式：在webpack.config.js中的rules中指定loader

  这里需要注意，loader的配置执行是从右到左或从下到上的，也就是会先执行css-loader，再执行style-loader

  ```
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  
  module.exports = {
      module: {
          rules: [
              // {
              //     test: /\.js|jsx?$/, loader: "babel-loader", exclude: /node_modules/, options: {
              //         plugins: ['syntax-dynamic-import'],
              //         // 解析js中的jsx必须配置
              //         presets: ['@babel/preset-env', '@babel/preset-react']
              //     },
              // },
              // { test: /\.tsx?$/, loader: "awesome-typescript-loader" },
              { enforce: "pre", test: /\.js$/, loader: "source-map-loader" },
              { test: /\.css?$/, use: ['style-loader', 'css-loader'] },
              { test: /\.js?$/, use:path.resolve(__dirname,'./src/loaders/loader.js')},
          ]
      },
  };
  ```

+ 内联方式：在每个import语句中指定loader

  使用!将资源中的laoder分开，每个部分都会相对于当前目录解析

  ```javascript
  import Styles from 'style-loader!css-loader?modules!./styles.css';
  ```

  通过为内联import添加前缀，可以覆盖配置中的所有loader、preLoader和postLoader

  + !前缀，禁用所有已配置的normal loader普通loader

  + !!前缀，禁用所有已配置的laoder：preLoader、loader和postLoader

  + -！前缀，禁用所有已配置的preLoader和loader，但是不禁用postLoaders

    ```javascript
    import Styles from '-!style-loader!css-loader?modules!./styles.css';
    ```

    选项可以传递查询的参数，例如?key=value&foo=bar

#### loader特性

+ 支持链式调用
+ loader可以是同步或异步的
+ loader运行在Nodejs中，能够执行任何操作
+ loader可以通过options对象配置
+ 插件可以为loader带来更多的特性
+ loader能够产生额外的任意文件

### 最简单的loader

> 当只有一个loader应用于资源文件时，接收源码作为参数，输出转换后的js代码

```javascript
module.exports=function loader(source){
    console.log('source------------------------');
    return source;
}
```

> 上面是一个最简单的loader，只是接收源码并返回，而输出仅仅只是为了调试而已

#### 配置自定义的loader

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    module: {
        rules: [
            { test: /\.js?$/, use:path.resolve(__dirname,'./src/loaders/loader.js')},
        ]
    },
};
```

### 带有pitch的loader

> pitch时loader上的一个方法，它的作用是阻断loader链

```javascript
module.exports=function loader(source){
    console.log('source------------------------');
    return source;
}

module.exports.pitch=function(){
    console.log('pitch-----------------------');
}
```

> pitch方法并不是必须的
>
> 但是如果有pitch，loader的执行会分为：pitch阶段和normal execution阶段。webpack会先从左到右执行loader链中的每个loader上的pitch方法，然后从右到左执行loader链中的每个loader的普通loader方法

假如我们配置了如下的loader链

```
use:['loader1','loader2','loader3']
```

那么真实的执行过程将是，当然这是假设所有loader都有pitch方法

<img src="https://segmentfault.com/img/remote/1460000021657034" style="zoom:100%;" />

在上述过程中，如果任何pitch有返回值，则loader链条将会被阻断，webpack将会跳过所有的pitch和loader直接进入上一个loader

的normal execution

假设在loader2中pitch返回了一个字符串，那么loader链会发生阻断

![](https://segmentfault.com/img/remote/1460000021657038)

### 简版的style-loader

> Style-loader一般不会单独使用，而是跟css-loader一起使用，当然css-loader总会比style-loader先调用
>
> 即将css-loader的输出作为style-loader的输入

css-loader的返回值一般是一个js模块

```javascript
// 打印 css-loader 的返回值

// Imports
var ___CSS_LOADER_API_IMPORT___ = require("../node_modules/css-loader/dist/runtime/api.js");
exports = ___CSS_LOADER_API_IMPORT___(false);
// Module
exports.push([module.id, "\nbody {\n    background: yellow;\n}\n", ""]);
// Exports
module.exports = exports;
```

这个模块在运行时上下文中执行后返回css代码，style-loader的作用就是将css代码转为style标签插入到html的head中

#### 设计思路

> 根据上面的介绍，我们已经有了具体的思路
>
> + style-loader需要返回一个脚本，这个脚本执行时会创建一个style标签，将css代码赋给style标签，然后将style标签插入到html的head中
> + 所以最困难的就是如何获取到css代码，因为css-loader的返回值只能在运行时的上下文中执行，而执行loader是在编译阶段，也就是说css-loader的返回值在style-loader中并没有什么用
> + 换一个思路：使用获取css代码的表达式，在运行时获取css
> + 在处理css的loader中去调用inline-loader require css文件，会产生循环执行loader的问题，所以需要利用pitch方法，让style-loader在pitch阶段返回脚本，跳过剩下的laoder，同时需要内联前缀!!的帮助

pitch方法有3个参数：

+ remainingRequest：loader链中排到自己后面的loader，实际上是先执行的，以及资源文件的绝对路径以!作为连接符组成的字符串
+ precedingRequest：loader链中排在自己前面的loader的绝对路径以！作为连接符组成的字符串
+ data：每个loader中存放在上下文中的固定字段，用于pitch给loader传递数据

可以利用remainingRequest参数获取loader链的剩余部分






























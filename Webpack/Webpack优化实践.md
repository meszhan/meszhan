# Webpack优化实践

> 一切不以实践验证的优化都是扯淡，干！

## 优化构建速度

### 多线程构建Thread-loader

#### 安装

```
npm i thread-loader --save-dev
```

#### 用法

把这个loader放置在其他loader之前，放置在这个laoder之后多loader就会在一个单独的worker池中运行

在worker池中运行的loader是受限制的：

+ 这些loader不能产生新文件
+ 这些loader不能使用定制的loader API（通过插件）
+ 这些loader无法获取webpack1的选项设置

每个worker都是一个单独的600ms限制的node.js进程，同时跨进程的数据交换也会被限制，因此应该只在比较耗时的loader中使用

#### 示例

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        include: path.resolve("src"),
        use: [
          "thread-loader",
          "expensive-loader"
        ]
      }
    ]
  }
}
```

#### 配置

```javascript
use: [
  {
    loader: "thread-loader",
    // 有同样配置的 loader 会共享一个 worker 池(worker pool)
    options: {
      // 产生的 worker 的数量，默认是 cpu 的核心数
      workers: 2,

      // 一个 worker 进程中并行执行工作的数量
      // 默认为 20
      workerParallelJobs: 50,

      // 额外的 node.js 参数
      workerNodeArgs: ['--max-old-space-size', '1024'],

      // 闲置时定时删除 worker 进程
      // 默认为 500ms
      // 可以设置为无穷大， 这样在监视模式(--watch)下可以保持 worker 持续存在
      poolTimeout: 2000,

      // 池(pool)分配给 worker 的工作数量
      // 默认为 200
      // 降低这个数值会降低总体的效率，但是会提升工作分布更均一
      poolParallelJobs: 50,

      // 池(pool)的名称
      // 可以修改名称来创建其余选项都一样的池(pool)
      name: "my-pool"
    }
  },
  "expensive-loader"
]

```

#### 预热

可以通过预热worker池来防止启动worker时的高延时

这会启动池内的最大数量的worker并把指定的模块载入node.js的模块缓存中

```javascript
const threadLoader = require('thread-loader');

threadLoader.warmup({
  // pool options, like passed to loader options
  // must match loader options to boot the correct pool
}, [
  // modules to load
  // can be any module, i. e.
  'babel-loader',
  'babel-preset-es2015',
  'sass-loader',
]);
```

### 缩小打包作用域

+ exclude/include确定loader规则范围
+ resolve.modules指明第三方模块的绝对路径，减少不必要的查找
+ resolve.extensions尽可能减少后缀尝试的可能性
+ noParse对完全不需要解析的库进行忽略（不去解析但仍然会打包到bundle中，注意被忽略掉的文件里不应该包含import、require、define等模块化语句）
+ IgnorePlugin完全排除模块
+ 合理使用alias

### 充分利用缓存提升二次构建速度

在性能开销较大的loader之前开启cache-loader，将结果缓存在磁盘里

+ babel-loader开启缓存
+ terser-webpack-plugin开启缓存
+ 使用cache-loader或hard-source-webpack-plugin注意thread-loader和cache-loader一起使用时，要先放cache-loader接着是thread-loader然后是heavy-loader

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.ext$/,
        use: [
          'cache-loader',
          ...loaders
        ],
        include: path.resolve('src')
      }
    ]
  }
}
```

### DLL

使用DllPlugin进行分包，使用DllRefrencePlugin对manifest.json引用，让一些基本不会改动的代码先打包成静态资源，避免反复编译浪费时间

## 优化打包体积

### 压缩代码

+ uglifyjs-webpack-plugin开启parallel参数，不支持ES6

+ terser-webpack-plugin

+ 多进程并行压缩

+ 通过mini-css-extract-plugin提取chunk中的css代码到单独文件，通过optimize-css-assets-webpack-plugin插件，压缩css

+ 使用html-webpack-externals-plugin将基础包通过CDN引入，不打入bundle中

+ 使用SplitChunksPlugin进行分离

+ 分离一些可以用CDN引入到基础包，例如vue

+ TreeShaking

  打包过程中检测工程中没有引用过的模块并进行标记，在资源压缩时从bundle中去掉，不过只对ESM生效，因此尽量使用ESM的import和export

+ Scope hoisting

  构建后的代码存在大量的闭包，造成体积增大，运行代码时创建的函数作用域变多，内存开销变大。Scope hoisting将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量防止变量名冲突






























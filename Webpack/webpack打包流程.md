# webpack打包可流程

https://juejin.cn/post/6844903728735059976

### 什么是webpack

> webpack是一个JavaScript应用程序的静态模块打包器（module bundler），在webpack处理应用程序时，会从打包入口entry开始递归的构建一个依赖关系图（dependency graph），其中包含了应用程序的每一个模块，并将这些模块打包成一个或多个bundle。
>
> webpack就像一条生产线，要经过一系列处理流程后才能将源文件转换成输出结果。插件就像是一个插入到生产线中的一个功能，在特定的时候对生产线上的资源做处理。
>
> webpack通过Tapable来组织复杂的生产线。它在运行过程中会广播事件，插件plugin只需要监听它关心的事件，就能加入到生产线中来改变生产线的运作。webpack的事件流机制保证了插件的有序性，使得系统的扩展性很好。

### webpack核心概念

#### Entry入口

> entry point是webpack打包的起点模块，也是内部依赖图构建的开始模块，它会递归的·找到entry直接或间接依赖的模块，在依赖项被处理后，最后输出到bundles的文件夹中

#### output出口

> output属性告诉webpack在哪里输出它创建的bundles，以及如何命名文件

#### module

> 在webpack中，一切文件皆为模块

#### chunk

> 代码块，一个Chunk由多个模块组合而成，用于代码合并与分割

#### loader

> wepack只能处理JavaScript文件，而loader让webpack可以处理其他文件，例如.css、.vue。
>
> 不同的loader可以将不同类型的文件转换为webpack能够处理的有效模块

#### plugin

> 与loader被用于转换某些类型的模块不同，插件用于执行范围更广的任务
>
> 插件的范围可以包括从打包优化和压缩，一直到重新定义环境中的变量，它可以用来处理各种各样的任务

### webpack构建流程

+ 初始化参数：从配置文件和shell语句中读取和合并参数，得到最后的webpack配置项
+ 编译：用得到的配置项初始化Compiler对象，加载所有配置的插件，执行对象的run方法开始编译
+ 确定入口：根据entry找出所有入口（入口可以不止一个）
+ 编译模块：从入口出发，调用不同loader处理不同模块，递归编译得到编译后的结果和依赖关系
+ 输出资源：根据入口和模块之间的依赖关系，组装成一个个Chunk，再把每个Chunk转换为单独的文件输出，根据配置的输出，写入到文件系统中

> 以上过程中，webpack会在特定的时间点广播出一些特定的时间，插件在监听到感兴趣的事件时执行特定的逻辑。
>
> 而且插件可以调用webpack提供的API改变webpack运行结果

### 原理解读

#### 源头webpack.js

````javascript
/*
	* 用webpack.config.js（或者自定义的配置文件）覆盖webpack的默认配置
*/
options = new WebpackOptionsDefaulter().process(options);
/*
	* 控制整个编译的流程，options.context是整个文件夹的绝对路径，已经在前面默认配置了
	* 配置compiler编译器，将NodeEnviromentPlugin的hooks以及自定义插件plugins的钩子挂到compiler上
	* 挂入之后出发environment的一些钩子
	* 比如在解析文件之前，要用到文件系统，就是将inputFileSystem挂载到compiler上，然后通过compiler控制哪些插件需要这个功能，派发
*/
compiler = new Compiler(options.context);
compiler.options = options;
new NodeEnvironmentPlugin().apply(compiler);
···省略自定义插件的绑定
compiler.hooks.environment.call();
compiler.hooks.afterEnvironment.call();
/*
	* 进一步对options做处理，将配置在compiler中激活
	* 这个类很重要，因为compiler中的激活了许多钩子，同时在一些钩子上挂上tap了函数
*/
compiler.options = new WebpackOptionsApply().process(options, compiler);
````

> NodeEnvironmentPlugin

```javascript
class NodeEnvironmentPlugin {
	apply(compiler) {
		compiler.inputFileSystem = new CachedInputFileSystem(
			new NodeJsInputFileSystem(),
			60000
		);
		//....
		compiler.hooks.beforeRun.tap("NodeEnvironmentPlugin", compiler => {
			if (compiler.inputFileSystem === inputFileSystem) inputFileSystem.purge();
		});
	}
}
module.exports = NodeEnvironmentPlugin;
```

#### Compiler.js

> Compiler.js真正控制webpack的打包流程

**constructor**

> 从constructor开始解析Compiler，它定义了很多钩子分别对应了流程的各个阶段，在这些钩子上可以挂上我们的插件

+ beforeRun
+ run
+ beforeCompile
+ compile
+ make
+ afterCompile
+ shouldEmit
+ emit
+ afterEmit
+ done

**Compiler.run**

> 首先触发beforeRun这个async钩子，在这个钩子中绑定读取文件的对象
>
> 然后是run这个async钩子，这里主要处理缓存的模块，减少的编译的模块，加速编译速度
>
> 最后调用Compiler.compile()进行编译

```javascript
this.hooks.beforeRun.callAsync(this, err => {
    ....
	this.hooks.run.callAsync(this, err => {
	    ....
      /*
      	* 在complie编译结束后，回调run中名为onCompiled的函数
      	* 将编译后的内容生成文件
      */
		this.compile(onCompiled);
		....
	});
	....
});
```

> onCompiled
>
> + 首先shouldEmit判断是否编译成功
> + 调用Compiler.emitAssets

```javascript
if (this.hooks.shouldEmit.call(compilation) === false) {
	...
	this.hooks.done.callAsync(stats, err => {
		...
    }
    return
	
}
this.emitAssets(compilation, err => {
    ...
    if (compilation.hooks.needAdditionalPass.call()) {
    ...
	    this.hooks.done.callAsync(stats, err => {});
    };
})
```

**Compiler.compile**

> + 定义params并传入hooks.compile钩子中，params就是模块工厂
>
> + 常用的是normalModuleFactory，将这个工厂传入后，方便之后的插件或钩子操作模块
>
>   钩子想要和程序产生联系，比如在compiler中添加内容，就需要将Compiler传入钩子中，否则没有暴露给插件的接口
>
> + beforeCompile：新建了Compilation
>
> + hooks.make：正式启动编译，

```javascript
const params = this.newCompilationParams();
this.hooks.beforeCompile.callAsync(params, err => {
    ...
	this.hooks.compile.call(params);
	const compilation = this.newCompilation(params);
    this.hooks.make.callAsync(compilation, err => {
    	...
        compilation.finish();
        ompilation.seal(err => {
			...	
			this.hooks.afterCompile.callAsync(compilation, err => {
			    ...
			    此处是回调函数，这个函数主要用于将编译成功的代码输出
    			...
			});
		});
	});
});
```

### 面试时如何回答

> Webpack是一个静态模块打包工具，它用来为前端的应用程序构建一个模块依赖关系图，或者是打包。
>
> 首先它会像大多数工具和框架一样，从配置文件或命令行中读取和合并参数，得到最终的配置项。
>
> 然后根据得到的配置项初始化Compiler也就是编译器对象，加载所有配置的插件，执行对象的run方法开始编译
>
> 编译过程会从配置中的entry开始递归的处理模块之间的依赖关系，因为Webpack只能识别js文件，所以碰到非js文件，就必须用对应的Loader对模块进行翻译
>
> 编译产生的模块依赖关系图，将模块组装成一个个chunk，再把chunk转换为文件输出到output配置中
































# Webpack

### 为什么需要Webpack

+ 在过去由于HTTP/1.1受网络延迟和浏览器并行请求限制等原因，将资源合并压缩有助于减少HTTP请求从而提升页面性能
+ JavaScript本身并没有提供模块系统。虽然在Nodejs中提供了CommonJS的模块化机制，但是在浏览器环境下并不支持。
+ Webpack不仅可以对静态资源打包压缩，还实现了一套模块化系统，还有各种loaer、plugin可以对打包的代码polyfill，编译转换less、vue和jsx等浏览器无法识别等文件格式，提供了HMR热更新机制来提高开发效率

### Webpack的缺点

+ webpack在打包时需要构建模块依赖图，因此需要递归遍历所有模块，随着项目体积增长，构建时间也会线性的增长，启动时间也会延长
+ 当我们修改文件后，webpack需要重新打包
+ 由于webpack在打包后会对源码进行编译和压缩，代码的可读性大大降低，因此在开发环境下需要开启SourceMap进行调试，但是会降低打包速度
+ 随着项目体积增大，打包出来的包也越来越大，过大的包会延长首屏加载时间，可以使用TreeShaking、Lazy-loading、CommonsChunkPlugin

### 为什么不需要Webpack了

+ HTTP2中实现了多路复用和首部压缩，解决了1.1中的1队头阻塞的问题
+ 浏览器开始逐一支持ESM，浏览器可以解析imports，无需自己实现模块化的优化，可以通过ESM实现按需加载，不需要打包

### Vite为什么使用esbuild进行编译

+ 快：esbuild使用Golang编写，大量使用并行操作们可以充分利用CPU资源，esbuild没有使用第三方依赖，可以高效利用内存
+ 缺点：
  + 虽然有loader但是没有插件机制
  + esbuild没有热更新

### Vite的缺陷

+ Vite在开发环境使用原生的ESM，但是在生产环境下仍然需要使用rollup打包。因为嵌套的imports导致需要发送大量的网络请求，即使使用了HTTP2.0，但依然无法避免效率低下的问题。所以为了记载性能，最好都bundle一遍
+ Vite更像是一个开发工具，而不是一个用于生产环境的构建工具。开发环境使用ESM解析模块，生产环境下使用rollup打包，导致本地和线上环境代码不一致

### 与Vite对比

#### 打包优先级

+ Webpack会先打包，然后启动开发服务器（dev server），请求服务器时直接给予打包结果。
+ Vite直接启动开发服务器，请求哪个模块时再对该模块进行实时编译

现代浏览器本身就支持ESM，会自动向依赖的Module发出请求，Vite充分利用这一点，将开发环境下的模块文件作为浏览器要执行的文件，而不是像webapck一样进行打包合并

#### 按需编译

因为vite在启动时不需要打包，也就不需要分析模块的依赖、不需要实时编译，启动速度非常快。当浏览器请求某个模块时，再根据需要对模块内容进行编译。按需动态编译的方式极大的缩减了编译时间，项目越复杂、模块越多，Vite的优势就越明显。

#### 热更新

在HMR方面，当改动一个模块后，只需俄昂浏览器重新请求该模块即可，不需要像webpack一样把该模块的相关依赖模块全部编译一次，效率更高。

#### 生产环境

当需要打包到生产环境时，Vite利用传统的rollup进行打包。Vite的优势主要体现在开发阶段。由于Vite利用的是ESM的按需加载，因此在代码中不能使用CommonJS

## HMR热更新原理

https://juejin.cn/post/6844904008432222215

https://juejin.cn/post/6939678015823544350

### webpack热更新

Hot Module Replacement，简称HMR，无需完全刷新整个页面从而更新模块。HMR可以有效节省开发时间，提升开发效率

#### 刷新

+ 页面刷新：不保留页面状态，直接window.location.reload()
+ 基于WDS也就是Webpack-dev-server的模块热替换，只需要局部刷新页面上发生变化的模块，同时也可以保留当前的页面状态

### webpack的编译构建过程

项目启动后，进行构建打包，控制台会输出构建过程，也会为当前构建生成一个hash值。

然后，在后面每次修改代码保存后，都会出现Compiling触发新的编译，会生：

+ 新的hash值
+ 新的json文件
+ 新的js文件

Hash值代表每次编译的标识，其次可以根据生成文件名可以发现，上次编译输出的Hash值会作为本次编译新生成的文件标识，同样，本次编译输出的Hash值会作为下次热更新的标识

#### json文件

```json
{
	c:{
    index:true,
  },
  h:'一段hash值'
}
```

h代表本次新生成的Hash值，用来作为下次文件热更新请求的前缀，c表示当前热更新文件对应的是index模块

#### js文件

里面是重新编译后打包的代码，但是如果没有任何代码改动保存文件，控制台也会输出编译打包信息，但只会包含：

+ 新的Hash值
+ 新的json文件

也就是说没有新的js文件，因为没有改动任何代码，同时浏览器中的json文件的c值也会为空

```json
{
  c:{},
  h:'本次构建新生成的Hash值'
}
```

### 热更新实现原理

#### webpack-dev-server启动本地服务

```javascript
// node_modules/webpack-dev-server/bin/webpack-dev-server.js

// 生成webpack编译主引擎 compiler
let compiler = webpack(config);

// 启动本地服务
let server = new Server(compiler, options, log);
server.listen(options.port, options.host, (err) => {
    if (err) {throw err};
});

// node_modules/webpack-dev-server/lib/Server.js
class Server {
    constructor() {
        this.setupApp();
        this.createServer();
    }
    
    setupApp() {
        // 依赖了express
    	this.app = new express();
    }
    
    createServer() {
        this.listeningApp = http.createServer(this.app);
    }
    listen(port, hostname, fn) {
        return this.listeningApp.listen(port, hostname, (err) => {
            // 启动express服务后，启动websocket服务
            this.createSocketServer();
        }
    }                                   
}
```

+ 启动webpack，生成compiler示例，compiler上有很多方法，比如可以启动webpack的所有编译工作，以及监听本地文件变化
+ 使用express框架启动本地server，让浏览器可以请求本地的静态资源
+ 本地server启动后，启动websocket服务，通过websocker可以建立本地服务和浏览器的双向通信，这样就可以在本地文件变化时，通知浏览器热更新代码

#### 修改webpack.config.js的entry配置

启动本地服务前，调用updateCompiler(this.compiler)方法

+ 获取websocket客户端代码路径
+ 根据配置获取webpack热更新代码路径

```javascript
// 获取websocket客户端代码
const clientEntry = `${require.resolve(
    '../../client/'
)}?${domain}${sockHost}${sockPath}${sockPort}`;

// 根据配置获取热更新代码
let hotEntry;
if (options.hotOnly) {
    hotEntry = require.resolve('webpack/hot/only-dev-server');
} else if (options.hot) {
    hotEntry = require.resolve('webpack/hot/dev-server');
}

```

修改后的webpack入口配置

```javascript
// 修改后的entry入口
{ entry:
    { index: 
        [
            // 上面获取的clientEntry
            'xxx/node_modules/webpack-dev-server/client/index.js?http://localhost:8080',
            // 上面获取的hotEntry
            'xxx/node_modules/webpack/hot/dev-server.js',
            // 开发配置的入口
            './src/index.js'
    	],
    },
}      

```

可以看到，在入口处新增了2个文件，它们会被一同打包到bundle文件中，也就是线上的runtime环境

##### webpack-dev-server/client/index.js

这个文件用于websocket，应为websocket是双向通信。在WDS初始化过程中，启动本地服务端的websocket，那么在客户端浏览器也需要支持websocket，但是又不应该让开发者去实现，因此就会将支持客户端websocket通信的代码通过改变entry从而添加到打包后的代码中

##### webpack/hot/dev-server.js

检查更新逻辑

#### 监听webpack编译结束

修改entry配置后，调用setupHooks方法，监听webpack编译完成的事件

```javascript
// node_modules/webpack-dev-server/lib/Server.js
// 绑定监听事件
setupHooks() {
    const {done} = compiler.hooks;
    // 监听webpack的done钩子，tapable提供的监听方法
    done.tap('webpack-dev-server', (stats) => {
        this._sendStats(this.sockets, this.getStats(stats));
        this._stats = stats;
    });
};
```

在监听到webpack一次编译结束后，就会调用sendStats方法通过websocket给浏览器通知，浏览器就可以拿到最新编译的一次的Hash值了

```javascript
// 通过websoket给客户端发消息
_sendStats() {
    this.sockWrite(sockets, 'hash', stats.hash);
    this.sockWrite(sockets, 'ok');
}
```

#### webpack监听文件变化

每次修改代码，都会触发文件重新编译，因此webpack必须监听本地代码的变化，主要通过setupDevMiddleware实现

这个方法主要执行webpack-dev-middleware库，wds只负责启动服务和前置准备工作，所有文件相关的操作都由wdm完成

包括本地文件的编译、输出和监听

```javascript
// node_modules/webpack-dev-middleware/index.js
compiler.watch(options.watchOptions, (err) => {
    if (err) { /*错误处理*/ }
});

// 通过“memory-fs”库将打包后的文件写入内存
setFs(context, compiler); 
```

+ 调用了compiler.watch方法，不得不说compiler十分强大
  + 首先对本地文件代码进行编译打包，也就是常说的webpack编译流程
  + 在编译结束后，开启对本地文件的监听，当文件发生变化时，重新编译
  + 编译完成后继续监听本地文件变化，原理是监听文件的生成时间，也就是传统的Last-Modified，因为如果对文件内容进行hash的话会十分耗费内存资源

+ 执行setFs方法，主要目的就是将编译后的文件打包到内存，所以我们其实在运行的时候不会在dist中拿到最新的代码，因为打包后的文件都会被放到内存中，访问内存的代码比文件系统中快很多，也可以减少代码写入的开销

#### 浏览器接收到热更新通知

在监听到文件变化后，就会触发重新编译，在编译结束后，就需要调用sendStats方法通过websocket给浏览器发送通知，检查是否需要热更新。上面我们知道sendStats方法先后触发了hash事件和ok事件

我们来看一下运行在浏览器的代码吧

##### 注入的socket代码

```javascript
// webpack-dev-server/client/index.js
var socket = require('./socket');
var onSocketMessage = {
    hash: function hash(_hash) {
        // 更新currentHash值
        status.currentHash = _hash;
    },
    ok: function ok() {
        sendMessage('Ok');
        // 进行更新检查等操作
        reloadApp(options, status);
    },
};
// 连接服务地址socketUrl，?http://localhost:8080，本地服务地址
socket(socketUrl, onSocketMessage);

function reloadApp() {
	if (hot) {
        log.info('[WDS] App hot update...');
        
        // hotEmitter其实就是EventEmitter的实例
        var hotEmitter = require('webpack/hot/emitter');
        hotEmitter.emit('webpackHotUpdate', currentHash);
    } 
}
```

Socket方法建立websocket和服务端的连接，并注册了2个监听事件

+ hash事件，更新最新一次打包生成的hash值
+ ok事件，浏览器端进行热更新检查

热更新检查事件调用reloadApp方法，但是它又利用了nodejs中的EventEmitter，发出webpackHotUpdate消息，而不是直接进行热更新

因为socket具有它自己的职责，而其他事情还是要交回给webpack去做

##### 热更新代码

```javascript
// node_modules/webpack/hot/dev-server.js
var check = function check() {
    module.hot.check(true)
        .then(function(updatedModules) {
            // 容错，直接刷新页面
            if (!updatedModules) {
                window.location.reload();
                return;
            }
            
            // 热更新结束，打印信息
            if (upToDate()) {
                log("info", "[HMR] App is up to date.");
            }
    })
        .catch(function(err) {
            window.location.reload();
        });
};

var hotEmitter = require("./emitter");
hotEmitter.on("webpackHotUpdate", function(currentHash) {
    lastHash = currentHash;
    check();
});
```

webpack监听到webpackHotUpdate事件后获取最新的hash值，然后调用module.hot.update方法，它来自于HotModuleReplacementPlugin

#### HotModuleReplacementPlugin

因为热更新代码发生在浏览器端，因此对于热更新的逻辑也必须在浏览器端运行，HotModuleReplacementPlugin也会将自己的逻辑代码打包到bundle中，这些是基于Tapable事件流和plugin事件捕获机制。

#### module.hot.check开始热更新

+ 利用上一次保存到hash值，调用hotDownloadManifest发送xxx/ha sh.hot-update.json的ajax请求
+ 请求获取热更新模块以及下次热更新的hash标识，并进入热更新准备阶段

```javascript
hotAvailableFilesMap = update.c; // 需要更新的文件
hotUpdateNewHash = update.h; // 更新下次热更新hash值
hotSetStatus("prepare"); // 进入热更新准备状态
```

+ 调用hotDownloadUpdateChunk发送js文件的请求，这里采用JSONP方式

```javascript
function hotDownloadUpdateChunk(chunkId) {
    var script = document.createElement("script");
    script.charset = "utf-8";
    script.src = __webpack_require__.p + "" + chunkId + "." + hotCurrentHash + ".hot-update.js";
    if (null) script.crossOrigin = null;
    document.head.appendChild(script);
 }
```

**为什么要用JSONP**

因为JSONP获取的代码可以直接执行

#### hotApply热更新模块替换

##### 删除过期的模块，也就是需要替换的模块

通过hotUpdate可以找到旧的模块

```javascript
var queue = outdatedModules.slice();
while (queue.length > 0) {
    moduleId = queue.pop();
    // 从缓存中删除过期的模块
    module = installedModules[moduleId];
    // 删除过期的依赖
    delete outdatedDependencies[moduleId];
    
    // 存储了被删掉的模块id，便于更新代码
    outdatedSelfAcceptedModules.push({
        module: moduleId
    });
}
```

##### 将新的模块添加到modules中

````javascript
appliedUpdate[moduleId] = hotUpdate[moduleId];
for (moduleId in appliedUpdate) {
    if (Object.prototype.hasOwnProperty.call(appliedUpdate, moduleId)) {
        modules[moduleId] = appliedUpdate[moduleId];
    }
}
````

##### 通过__webpack_require__执行相关模块的代码

```javascript
for (i = 0; i < outdatedSelfAcceptedModules.length; i++) {
    var item = outdatedSelfAcceptedModules[i];
    moduleId = item.module;
    try {
        // 执行最新的代码
        __webpack_require__(moduleId);
    } catch (err) {
        // ...容错处理
    }
}
```

### 总结

#### 热更新流程

+ webpack在首次构建完成后，将打包后的文件传输给本地启动的Bundle Server，所以我们可以通过localhost:8080去访问它

  但是注意打包后的文件其实是放在内存中的，所以我们无法在dist目录下的到最新的打包文件。

  同时在启动Bundle Server构建的时候，还会启动一个HMR Server专门用于提供热更新服务，也会通过修改entry配置向bundle中注入线上的HMR Runtime运行环境，主要包括提供与HMR Server通信的websocket和提供热模块替换

+ HMR Server

  启动一个express服务，然后创建一个websocket服务并监听

+ 什么时候HMR Server与HMR Runtime通信

  webpack基于Tapable事件流实现，会在编译构建过程中广播很多事件，我们需要捕获compiler.hooks.done钩子，表示在webpack编译完成后触发，向Runtime发送hash和ok事件，浏览器接收到事件后更新最新一次构建的hash，并检查更新模块

+ 开启热更新后的代码会比不开的时候多了HMR Runtime代码

+ 浏览器接收到更新通知后，调用hot.check方法发起http请求分别请求一个json文件（最新一次构建的hash以及所有更新的模块），然后根据json文件再次请求最新的模块替换过期的模块并进行热模块替换






























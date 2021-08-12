# Vue全局API

https://juejin.cn/post/6952643167715852319

### 目标

**深入了解以下API的实现原理**

+ Vue.use
+ Vue.mixin
+ Vue.component
+ Vue.filter
+ Vue.directive
+ Vue.extend
+ Vue.set
+ Vue.delete
+ Vue.nextTick

### 源码解读

> Vue的众多全局API的实现大部分都放在/src/core/global-api目录下，入口文件在/src/core/global-api/index.js

### 入口

> /src/core/global-api/index.js

```javascript
/*
	* 初始化Vue的众多全局API，比如：
	* 默认配置Vue.config
	* 工具方法Vue.util.xx
	* Vue.set、Vue.delete、Vue.nextTick、Vue.observable
	* Vue.options.components、Vue.options.directives、Vue.options.filters、Vue.options._base
	* Vue.use、Vue.extend、Vue.mixin、Vue.component、Vue.directive、Vue.filter
*/
export function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  // Vue的众多配置项
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)

  /*
  	* 暴露一些工具方法，不要轻易使用这些工具方法，除非很清楚以及知道使用的风险
  */
  Vue.util = {
    // 警告日志
    warn,
    // 类似选项合并
    extend,
    // 合并选项
    mergeOptions,
    // 设置响应式
    defineReactive
  }

  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  // 响应式方法
  Vue.observable = <T>(obj: T): T => {
    observe(obj)
    return obj
  }

  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // 把Vue构造函数挂到_base上
  Vue.options._base = Vue

  // 在Vue.options.components中添加内置组件，比如keep-alive
  extend(Vue.options.components, builtInComponents)

  initUse(Vue)
  initMixin(Vue)
  initExtend(Vue)
  initAssetRegisters(Vue)
}
```

#### Vue.use

> /src/core/global-api/use.js

```javascript
/*
	* 定义Vue.use，负责为Vue安装插件
	* 1. 判断插件是否已经被安装，如果安装pass
	* 2. 安装插件，执行插件的install方法
*/
export function initUse (Vue: GlobalAPI) {
  Vue.use = function (plugin: Function | Object) {
    // 安装过的插件列表
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    // 防止重复安装
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // Vue构造函数放在第一个参数位置
    const args = toArray(arguments, 1)
    args.unshift(this)
    if (typeof plugin.install === 'function') {
      // plugin是对象，执行install方法安装插件
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      // 直接执行plugin方法安装插件
      plugin.apply(null, args)
    }
    // 在插件列表中添加新增插件
    installedPlugins.push(plugin)
    return this
  }
}
```

#### Vue.mixin

> /src/core/global-api/mixin.js

```javascript
/*
	* 定义Vue.mixin，负责全局混入选项，影响之后所有创建的Vue实例，这些实例会合并全局混入的选项
	* @params {*} mixin Vue配置对象
	* @returns 返回Vue实例
*/
export function initMixin (Vue: GlobalAPI) {
  Vue.mixin = function (mixin: Object) {
    // 合并mixin到Vue的默认配置项
    this.options = mergeOptions(this.options, mixin)
    return this
  }
}
```

#### mergeOptions

> /src/core/util/options.js

```javascript
/*
	* 合并两个选项，出现相同配置项时，子选项覆盖父选项配置
*/
export function mergeOptions (
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  if (process.env.NODE_ENV !== 'production') {
    checkComponents(child)
  }

  if (typeof child === 'function') {
    child = child.options
  }
	// 标准化props、inject、directive选项
  normalizeProps(child, vm)
  normalizeInject(child, vm)
  normalizeDirectives(child)

  /*
  	* 处理原始child对象上的extends和mixins，分别执行mergeOptions，将继承的选项合并到parent
  	* mergeOptions处理过的对象会有_base属性
  */
  if (!child._base) {
    if (child.extends) {
      parent = mergeOptions(parent, child.extends, vm)
    }
    if (child.mixins) {
      for (let i = 0, l = child.mixins.length; i < l; i++) {
        parent = mergeOptions(parent, child.mixins[i], vm)
      }
    }
  }

  const options = {}
  let key
  // 比那里父选项
  for (key in parent) {
    mergeField(key)
  }
  // 遍历子选项，如果父选项不存在配置，合并，否则跳过
  // 因为父子拥有同一个属性的情况下，在上面处理父选项时已经处理过了，用的子选项的值
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  // 合并选项，child优先级高于parent
  function mergeField (key) {
    // strat是合并策略函数，如何解决key冲突，则child会覆盖parent
    const strat = strats[key] || defaultStrat
    // 值为如果child存在则优先使用，否则使用parent
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
}
```

#### Vue.component、Vue.filter、Vue.directive

> /src/core/global-api/assets.js

```javascript
const ASSET_TYPES = ['component', 'directive', 'filter'];

/*
	* 定义Vue.component、Vue.filter、Vue.directive方法
	* 这三个方法事情类似，都是在this.options.xx上存放对应的配置
	* 比如Vue.component(compName,{XX})结果是this.options.components.compName=组件构造函数
	* ASSET_TYPES=['components','directive','filter']
*/
export function initAssetRegisters (Vue: GlobalAPI) {
  /**
   * Create asset registration methods.
   */
  
  /*
  	* 比如Vue.component(name,definition)
  	* @params {*} id name
  	* @params {*} definition 组件构造函数或者配置对象
  	* returns 返回组件构造函数
  */
  ASSET_TYPES.forEach(type => {
    Vue[type] = function (
      id: string,
      definition: Function | Object
    ): Function | Object | void {
      if (!definition) {
        return this.options[type + 's'][id]
      } else {
        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production' && type === 'component') {
          validateComponentName(id)
        }
        if (type === 'component' && isPlainObject(definition)) {
          // 如果组件配置中存在name，则使用，否则直接使用id
          definition.name = definition.name || id
          // extend就是Vue.extend,这是的definition旧变成了组件构造函数，使用时可直接new Definition
          definition = this.options._base.extend(definition)
        }
        if (type === 'directive' && typeof definition === 'function') {
          definition = { bind: definition, update: definition }
        }
    // this.options.components[id]=definition
    // 在实例化时通过mergeOptions将全局注册的组件合并到每个组件的配置对象的components中
        this.options[type + 's'][id] = definition
        return definition
      }
    }
  })
}
```

#### Vue.extend

> /src/core/gobal-api/extend.js

```javascript
export function initExtend (Vue: GlobalAPI) {
  /**
   * Each instance constructor, including Vue, has a unique
   * cid. This enables us to create wrapped "child
   * constructors" for prototypal inheritance and cache them.
   */
  Vue.cid = 0
  let cid = 1

  /*
  	* 基于Vue扩展子类，该子类同样支持进一步扩展
  	* 扩展时可以传递一些默认配置，就像Vue也会有一些默认配置
  	* 默认配置如果和基类有冲突会进行选项合并
  */
  Vue.extend = function (extendOptions: Object): Function {
    extendOptions = extendOptions || {}
    const Super = this
    const SuperId = Super.cid
    /*
    	* 利用缓存，如果存在直接返回缓存中的构造函数
    	* 什么情况喜爱可以使用这个缓存？
    	* 如果多次调用Vue.extend时使用了同一个配置项，就会启用缓存
    */
    const cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {})
    if (cachedCtors[SuperId]) {
      return cachedCtors[SuperId]
    }

    const name = extendOptions.name || Super.options.name
    if (process.env.NODE_ENV !== 'production' && name) {
      validateComponentName(name)
    }

    // 定义Sub构造函数，和Vue一样
    const Sub = function VueComponent (options) {
      // 初始化
      this._init(options)
    }
    // 原型继承的方式继承Vue
    Sub.prototype = Object.create(Super.prototype)
    Sub.prototype.constructor = Sub
    Sub.cid = cid++
    // 选项合并，合并Vue的配置项到自己的配置项
    Sub.options = mergeOptions(
      Super.options,
      extendOptions
    )
    // 记录自己的基类
    Sub['super'] = Super

    // 初始化props，将props代理到Sub.prototype._props对象上
    // 在组件内通过_props访问
    if (Sub.options.props) {
      initProps(Sub)
    }
    // 初始化computed
    // 组件内可以通过this.computedKey访问
    if (Sub.options.computed) {
      initComputed(Sub)
    }

    // 定义三个静态方法，允许在Sub基础上进一步构造子类
    Sub.extend = Super.extend
    Sub.mixin = Super.mixin
    Sub.use = Super.use

    // 定义三个静态方法
    ASSET_TYPES.forEach(function (type) {
      Sub[type] = Super[type]
    })
    // 递归组件的原理，如果组件设置了name属性，将自己注册到自己的components选项中
    if (name) {
      Sub.options.components[name] = Sub
    }

    // 在扩展时保留对基类选项的引用
    // 稍后在实例化时，检查Super的选项是否有更新
    Sub.superOptions = Super.options
    Sub.extendOptions = extendOptions
    Sub.sealedOptions = extend({}, Sub.options)

    // 缓存
    cachedCtors[SuperId] = Sub
    return Sub
  }
}
function initProps (Comp) {
  const props = Comp.options.props
  for (const key in props) {
    proxy(Comp.prototype, `_props`, key)
  }
}

function initComputed (Comp) {
  const computed = Comp.options.computed
  for (const key in computed) {
    defineComputed(Comp.prototype, key, computed[key])
  }
}
```

#### Vue.set

> /src/core/global-api/index.js

```javascript
Vue.set=set
```

> /src/core/observer/index.js

```javascript
/*
	* 通过Vue.set或者this.$set方法给target指定key并设为val
	* 如果target为对象，并且key原本不存在，则为新key设置响应式，然后执行依赖通知
*/
export function set (target: Array<any> | Object, key: any, val: any): any {
  if (process.env.NODE_ENV !== 'production' &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(`Cannot set reactive property on undefined, null, or primitive value: ${(target: any)}`)
  }
  // 如果target是数组，且下标有效（在数组长度范围内），通过splice方法响应式更新
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key)
    target.splice(key, 1, val)
    return val
  }
  // 更新对象已有的属性，直接更新即可，会自动触发setter，然后通知依赖他的watcher
  if (key in target && !(key in Object.prototype)) {
    target[key] = val
    return val
  }
  const ob = (target: any).__ob__
  // 不能项Vue实例或者$data添加动态添加响应式属性，vmCount用处之一
  // this.$data的ob.vmCount=1，表示根组件，其他子组件的vm.vmCount=0
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid adding reactive properties to a Vue instance or its root $data ' +
      'at runtime - declare it upfront in the data option.'
    )
    return val
  }
  // target不是响应式对象，新属性会被设置，但是不做响应式处理
  if (!ob) {
    target[key] = val
    return val
  }
  // 给对象定义新属性，同definereactive方法设置响应式，并触发依赖更新
  defineReactive(ob.value, key, val)
  ob.dep.notify()
  return val
}
```

#### Vue.delete

> /src/core/global-api/index.js

```javascript
Vue.delete=del
```

> /src/core/observer/index.js

```javascript
/*
	* 通知Vue.delete或者vm.$delete删除target对象的指定key
	* 数组通过splice方法实现，对象通过delete运算符删除指定key，并执行依赖同志1
*/
export function del (target: Array<any> | Object, key: any) {
  if (process.env.NODE_ENV !== 'production' &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(`Cannot delete reactive property on undefined, null, or primitive value: ${(target: any)}`)
  }
  // 数组通过splice方法删除指定下标元素
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.splice(key, 1)
    return
  }
  const ob = (target: any).__ob__
  // 避免删除Vue实例的属性或$data的数据
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid deleting properties on a Vue instance or its root $data ' +
      '- just set it to null.'
    )
    return
  }
  // 属性不存在
  if (!hasOwn(target, key)) {
    return
  }
  // 删除对象属性
  delete target[key]
  if (!ob) {
    return
  }
  // 执行依赖通知
  ob.dep.notify()
}
```

#### Vue.nextTick

> /src/core/global-api/index.js

```javascript
Vue.nextTick=nextTick
```

> /src/core/util/next-tick.js

```javascript
const callbacks = []
/**
 * 完成两件事：
 *   1、用 try catch 包装 flushSchedulerQueue 函数，然后将其放入 callbacks 数组
 *   2、如果 pending 为 false，表示现在浏览器的任务队列中没有 flushCallbacks 函数
 *     如果 pending 为 true，则表示浏览器的任务队列中已经被放入了 flushCallbacks 函数，
 *     待执行 flushCallbacks 函数时，pending 会被再次置为 false，表示下一个 flushCallbacks 函数可以进入
 *     浏览器的任务队列了
 * pending 的作用：保证在同一时刻，浏览器的任务队列中只有一个 flushCallbacks 函数
 * @param {*} cb 接收一个回调函数 => flushSchedulerQueue
 * @param {*} ctx 上下文
 * @returns 
 */
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  // 用 callbacks 数组存储经过包装的 cb 函数
  callbacks.push(() => {
    if (cb) {
      // 用 try catch 包装回调函数，便于错误捕获
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    // 执行 timerFunc，在浏览器的任务队列中（首选微任务队列）放入 flushCallbacks 函数
    timerFunc()
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```

### Q & A

#### Vue.use(plugin)做了什么？

> 






























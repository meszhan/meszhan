# Loader和Plugin的区别

### Loader

> Loader本质上是一个函数，在该函数中对接受的内容进行转换，返回转换后的结果。因为Webpack只能识别JavaScript文件，因此Loader就是一个翻译的函数，对其他类型的资源进行转译的预处理工作
>
> Loader一般在module.rules中配置，作为模块的解析规则，类型为数组（因为我们需要不同的loader处理不同的资源）
>
> 数组的每一项都是对象，包含test类型的文件、loader、options等属性

### Plugin

> Plugin是插件，给予事件流框架Tapable，插件可以扩展Webpack的功能，在Webpack运行生命周期中会广播出许多事件，Plugin通过监听这些事件，在合适的时机通过Webpack提供的API改变输出的结果
>
> 在module.plugins中配置，类型为数组，每一项都是Plugin的实例，参数通过构造函数传入
































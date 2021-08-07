# 如何提高webpack构建速度

#### 多入口时，使用CommonsChunkPlugin

#### DllPlugin和DllRefrencePlugin

> 把我们需要打包的第三方库打包成一个js文件和json文件，这个json文件中会映射每个打包的模块地址和id，DllRefrencePlugin通过读取这个json文件来使用打包的这些模块

#### 在loader中配置include和exclude

#### resolve.modules

> 指定webpack解析模块时应该搜索的目录

#### 优化babel-loader

> babel-loader处理大量的JS，JSX等JS模块，通过内置的缓存配置来缓存执行结果

#### cache-loader

> 缓存loader的执行结果到本地磁盘或数据库中，但是因为设计IO操作，可能带来更大的内存空间
>
> 所以只在需要编译大量模块的loader中使用cache-loader

#### 多线程打包thread-loader

> 利用nodejs的worker pool机制，

#### externals

> 把某些模块在打包过程中剔除，比如一些用CDN引入的模块










































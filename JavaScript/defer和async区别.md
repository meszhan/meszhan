# defer和asycn的区别

### 引言

> defer和async都是异步执行外链js的策略，这样js的加载不会阻塞页面的渲染和资源的加载

### defer

> 脚本的执行在页面解析完后，会按照原本的顺序执行，在document的DOMContentLoaded事件之前执行

### async

> 在脚本下载结束后立即执行，因此无法保证脚本执行顺序，不能保证js脚本的依赖关系正确，会在window的load事件之前执行

### 区别

> + 联想到为什么把CSS的link放前面，把script标签放后面
>
> + 两个事件的区别：onload和domcontentloaded

### load和DOMContentLoaded

#### load

> load仅用于检测一个完全加载的页面，当一个质押u你及其依赖资源已加载完成时，触发load事件

#### DOMContentLoaded

> 当初始的HTML文档被完全加载和解析完成后，DOMContentLoaded事件被触发，无需等待样式表、图像和子框架完成加载

### link和script

> script会阻塞后续内容的解析，但是link标签不会阻塞DOM的构建，CSSOM的构建可以和DOM同步进行
>
> 但是Render Tree的构建会受到DOM和CSSOM的限制
































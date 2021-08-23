# CSS

### 水平垂直居中

### transform、transition、animation

#### transition

用于设置元素的样式过渡，元素从某个属性的值过渡到这个属性的另一个值，是单个属性的状态转变，但是这种转变需要条件来触发。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>transition</title>
  <style>
    #box {
      height: 100px;
      width: 100px;
      background: green;
      transition: transform 1s ease-in 1s;
    }

    #box:hover {
      transform: rotate(180deg) scale(.5, .5);
    }
  </style>
</head>
<body>
  <div id="box"></div>
</body>
</html>
```

在上述样式中，我们为transition设置了transform属性的过渡，并由hover事件触发，transition属性从左到右分别对应了

+ transition-property：过渡的属性
+ transition-duration：过渡花费的时间，注意如果不指定过渡时间，transition没有效果，因为默认为0
+ transition-timing-function：过渡效果的时间曲线
+ transition-delay：过渡延时

多项transition属性设置

```css
div
{
transition: width 2s, height 2s, transform 2s;
-webkit-transition: width 2s, height 2s, -webkit-transform 2s;
}  
```

##### 缺点

+ 需要特定的事件触发，例如hover、focus、checked，不能在网页加载时自动发生
+ 一次性过渡，不能重复发生，除非重复触发
+ 只能定义开始和结束状态，不能定义中间状态，也就是只有两个状态

#### animation

animation是对transition属性的扩展，弥补了transition的很多不足之处，animation是由很多个transition效果的叠加，可操作性也更强

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <title>animation</title>
  <style>
    .box {
      height: 100px;
      width: 100px;
      border: 15px solid black;
      animation: changebox 1s ease-in-out 1s infinite alternate running forwards;
    }

    .box:hover {
      animation-play-state: paused;
    }

    @keyframes changebox {
      10% {
        background: red;
      }
      50% {
        width: 80px;
      }
      70% {
        border: 15px solid yellow;
      }
      100% {
        width: 180px;
        height: 180px;
      }
    }
  </style>
</head>

<body>
  <div class="box"></div>
</body>

</html>
```

上述由@keyframes定义了一个关键帧动画，百分比定义了在动画时间到达某个百分比的时候发生什么变化

animation属性的拆解：

+ Animation-name： 调用keyframes定义的动画，必须与keyframes名称一致
+ Animation-duration：动画持续时间
+ animation-timing-function：速度效果的速度曲线
+ animation-delay：动画延时多久开始
+ animation-iteration-count：动画播放次数，可以设置为infinite无限播放
+ animation-direction：动画播放方向，normal（时间轴顺序）、reverse（反序）、alternate（轮流，来回往复进行）、alternate（反序交替）
+ animation-play-state：动画播放状态，控制动画的暂停和继续，running继续、paused暂停
+ animation-fill-mode：动画结束后元素的样式，none（回到动画没开始时的状态）、forwards（停留在动画结束时的状态）、backwards（回到第一帧的状态）、both（轮流应用forwards和backwards）

##### 优点

+ keyframes提供了更多的控制

### 选择器和优先级

### 属性继承

#### 字体系列属性

- font：组合字体
- font-family：规定元素的字体系列
- font-weight：设置字体的粗细
- font-size：设置字体的尺寸
- font-style：定义字体的风格
- font-variant：设置小型大写字母的字体显示文本，这意味着所有的小写字母均会被转换为大写，但是所有使用小型大写字体的字母与其余文本相比，其字体尺寸更小。
- font-stretch：允许你使文字变宽或变窄。所有主流浏览器都不支持。
- font-size-adjust：为某个元素规定一个 aspect 值，字体的小写字母 "x" 的高度与"font-size" 高度之间的比率被称为一个字体的 aspect 值。这样就可以保持首选字体的 x-height。

#### 文本系列属性

- text-indent：文本缩进
- text-align：文本水平对齐
- line-height：行高
- word-spacing：增加或减少单词间的空白（即字间隔）
- letter-spacing：增加或减少字符间的空白（字符间距）
- text-transform：控制文本大小写
- direction：规定文本的书写方向
- color：文本颜色

#### 元素可见性

visibility

#### 表格布局属性

caption-side、border-collapse、border-spacing、empty-cells、table-layout

#### 列表属性

list-style-type、list-style-image、list-style-position、list-style

#### 生成内容属性

quotes

#### 光标属性

cursor

#### 页面样式属性

page、page-break-inside、windows、orphans

#### 声音样式属性

speak、speak-punctuation、speak-numeral、speak-header、speech-rate、volume、voice-family、pitch、pitch-range、stress、richness、azimuth、elevation

#### 特殊的属性继承

a标签字体颜色不能被继承，h1-h6标签字体大小不能被继承，因为存在默认值

### link和@import的区别

#### 从属关系区别

+ @import是 CSS 提供的语法规则，只有导入样式表的作用；

+ link是HTML提供的标签，不仅可以加载 CSS 文件，还可以定义 RSS、rel 连接属性等。

#### 加载顺序区别

+ 加载页面时，link标签引入的 CSS 被同时加载
+ @import引入的 CSS 将在页面加载完毕后被加载

#### 兼容性区别

+ @import是 CSS2.1 才有的语法，故只可在 IE5+ 才能识别
+ link标签作为 HTML 元素，不存在兼容性问题

#### DOM可控性区别

+ 可以通过 JS 操作 DOM ，插入link标签来改变样式
+ 由于 DOM 方法是基于文档的，无法使用@import的方式插入样式

### 脱离文档流的操作

#### float

使用float脱离文档流后，其他盒子会无视这个元素，但其他盒子的文本依然会为这个元素让出位置，环绕在元素周围

#### absolute

绝对定位，脱离文档流后，相对于该元素的非static父元素定位

#### fixed

完全脱离文档流，相对于浏览器窗口左上角定位

### 清除浮动

### BFC块级格式上下文

#### 什么是BFC

BFC：Block Formatting Context，块级格式上下文

BFC决定了元素如何对内容进行定位，以及与其他元素的关系和相互作用，当涉及到可视化布局时，BFC提供了一个环境，HTML在这个环境中按照一定的规则布局

BFC是一个完全独立的空间，让空间里的子元素不会影响到外面的布局

#### 如何触发BFC

+ overflow不为visible，因为它会影响到外部！！！
+ position：absolute/fixed
+ display：inline-block/table-cell
+ html根元素

### BFC的规则

+ BFC内部的块级元素在垂直方向连续排列
+ BFC是页面中的一个隔离的独立容器，里面的内容不会影响到外面的
+ 垂直方向的距离由margin决定，属于同一个BFC的两个相邻标签会发生重叠，因此可以用于防止margin重叠
+ 计算BFC高度时，会计算浮动元素，因此可以用于清除浮动

#### BFC解决的问题

##### 解决高度塌陷

##### Margin重叠

##### 两栏布局

### animation动画

#### 一个三角形从透明到实体，持续1s，然后匀速前进5s，然后变大持续2s

首先，题目中描述了多个动画，因此对于某个元素需要定义多个animation，以逗号（,）分隔，然后考虑具体实现细节

+ 从透明到实体：opacity属性从0到1，第一个keyframe，持续1s，animation-duration属性，而且需要保留最后一帧，因此需要animation-fill-mode属性设为forwards表示用最后一帧填充
+ 然后匀速前进5s：因为有“然后”，所以需要设置animation-delay属性为1s，表示等opacity动画执行完后执行，匀速前进需要控制动画的执行函数animation-timing-function设置为linear表示线性即匀速，其他属性类似
+ 然后变大持续2s：这里可以既可以控制width，也可以控制height，也可以用transform属性，不过效果是不一样的
  + transform属性：利用scale或scaleX或scaleY都可以，但是这种扩张是基于中心点不变实现的，会改变四个角的位置
  + 控制width和height不会改变左上角的位置

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>Page Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="stylesheet" type="text/css" media="screen" href="main.css" />
    <script src="main.js"></script>
    <style>
      .animation-sample {
        width: 200px;
        height: 200px;
        margin-top: 100px;
        background-color: red;
        animation: opacity 1s forwards, move 5s linear 1s forwards,
          scale 2s linear 6s forwards;
      }
      @keyframes opacity {
        from {
          opacity: 0;
        }
        to {
          opacity: 1;
        }
      }
      @keyframes move {
        from {
          margin-left: 0px;
        }
        to {
          margin-left: 1000px;
        }
      }
      @keyframes scale {
        from {
          transform: scale(1);
        }
        to {
          transform: scale(2);
        }
      }
    </style>
  </head>
  <body>
    <div class="animation-sample"></div>
  </body>
</html>

```

#### 试探多动画情形下的属性配置问题

> 多动画的属性配置既可以都写在animation属性中，也可以像animation中一样，用逗号分隔开写在不同的animation控制属性中

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>Page Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="stylesheet" type="text/css" media="screen" href="main.css" />
    <script src="main.js"></script>
    <style>
      .animation-sample {
        width: 200px;
        height: 200px;
        margin-top: 100px;
        background-color: red;
        animation: opacity, move, scale;
        animation-timing-function: linear, linear, linear;
        animation-delay: 0s, 1s, 6s;
        animation-duration: 1s, 5s, 2s;
        animation-fill-mode: forwards, forwards, forwards;
      }
      @keyframes opacity {
        from {
          opacity: 0;
        }
        to {
          opacity: 1;
        }
      }
      @keyframes move {
        from {
          margin-left: 0px;
        }
        to {
          margin-left: 1000px;
        }
      }
      @keyframes scale {
        from {
          transform: scale(1);
        }
        to {
          transform: scale(2);
        }
      }
    </style>
  </head>
  <body>
    <div class="animation-sample"></div>
  </body>
</html>
```

#### 多动画中的属性缺失问题

```

```

### 形状绘制

#### 三角形

```html
<div class="test"></div>

<style>
    .test{
        width: 0%;
        height: 0%;
        border: 100px solid transparent;
        border-right: 100px solid red;
    }
</style>
```

#### 梯形

```html
<div class="test"></div>

<style>
    .test{
        width: 200px;
        height: 200px;
        border: 100px solid transparent;
        border-right: 100px solid red;
    }
</style>
```

#### 扇形

```html
<div class="test"></div>

<style>
    .test{
        width: 0;
        border-radius: 100px;
        border: 100px solid transparent;
        border-top-color: red;
    }
</style>
```

#### 箭头

```html
<div class="test"></div>

<style>
    body{
        margin:0%;
        padding: 0%;
    }
    .test{
        width: 0%;
        height: 0%;
        border: 100px solid transparent;
        border-right: 100px solid red;
    }
    .test::after{
            content: '';
            position: absolute;
            left: 5px;
            top: 0px;
            border: 100px solid;
            border-color: transparent #fff transparent transparent;
        }
</style>
```

#### 椭圆

```html
<div class="test"></div>

<style>
    .test{
        width: 200px;
        height: 100px;
        border-radius: 100px / 50px;
        background-color: red;
    }
</style>
```

### CSS GPU硬件加速——Web性能优化

https://fed.taobao.org/blog/2016/04/26/performance-composite/

CSS硬件加速又叫做GPU加速，利用GPU进行渲染，从而减少CPU操作进行优化的方案。由于GPU中的transform等CSS属性不会触发repaint，所以可以提高网页性能。

#### 动画与帧

![](https://lz5z.com/assets/img/css3_gpu_speedup.png)

上面大概描述了浏览器在每一帧内做的事情：

+ JavaScript：一般来说我们可能性会使用JavaScript实现一些动画效果和DOM操作，这些操作会影响到页面布局
+ Style计算样式：根据CSS选择器，对每个DOM元素匹配对应的CSS样式，确定每个DOM元素的CSS样式规则
+ Layout布局：计算每个元素最终在屏幕上显示的大小和位置。因为页面上的元素布局是相对的，所以任意一个元素的位置发生变化，都会导致其他元素发生变化，也就是回流或重排。例如父元素的宽度会影响到子元素的宽度甚至是更深层的元素布局
+ Paint绘制：填充像素，在多个层上绘制元素的文字、颜色、图像和边框等，即一个DOM元素的所有可视效果，这个绘制过程会在多个层上完成
+ Composite渲染层合并：因为DOM元素的绘制是在多个层上进行的，在所有层都完成绘制后，浏览器会按照合理的顺序合并图层然后显示到屏幕上，对于有位置重叠的元素的页面，一旦合并顺序出错，会导致元素显示异常

#### 浏览器渲染原理

在浏览器中，页面内容是存储为由 Node 对象组成的树状结构，也就是 DOM 树。每一个 HTML element 元素都有一个 Node 对象与之对应，DOM 树的根节点永远都是 Document Node。

![](https://img.alicdn.com/tps/TB1VFRDMXXXXXahXpXXXXXXXXXX-814-320.png_720x720.jpg#alt=)

##### 从Nodes到LayoutObjects

DOM树中的每个Node节点都有对应的LayoutObject，LayoutObject描述了如何在屏幕上绘制节点内容

##### 从 LayoutObjects 到 PaintLayers

一般来说，拥有相同的坐标空间的 LayoutObjects，属于同一个渲染层（PaintLayer）。PaintLayer 最初是用来实现 [stacking contest（层叠上下文）](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context)，以此来保证页面元素以正确的顺序合成（composite），这样才能正确的展示元素的重叠以及半透明元素等等。因此满足形成层叠上下文条件的 LayoutObject 一定会为其创建新的渲染层，当然还有其他的一些特殊情况，为一些特殊的 LayoutObjects 创建一个新的渲染层，比如 `overflow != visible` 的元素。根据创建 PaintLayer 的原因不同，可以将其分为常见的 3 类：

- NormalPaintLayer
  - 根元素（HTML）
  - 有明确的定位属性（relative、fixed、sticky、absolute）
  - 透明的（opacity 小于 1）
  - 有 CSS 滤镜（fliter）
  - 有 CSS mask 属性
  - 有 CSS mix-blend-mode 属性（不为 normal）
  - 有 CSS transform 属性（不为 none）
  - backface-visibility 属性为 hidden
  - 有 CSS reflection 属性
  - 有 CSS column-count 属性（不为 auto）或者 有 CSS column-width 属性（不为 auto）
  - 当前有对于 opacity、transform、fliter、backdrop-filter 应用动画

- OverflowClipPaintLayer
  - overflow 不为 visible

- NoPaintLayer
  - 不需要 paint 的 PaintLayer，比如一个没有视觉属性（背景、颜色、阴影等）的空 div。

满足以上条件的 LayoutObject 会拥有独立的渲染层，而其他的 LayoutObject 则和其第一个拥有渲染层的父元素共用一个。

#### 动画与图层

浏览器在获取render tree后，渲染树中包含大量的渲染元素，每个渲染元素被分到一个图层中，每个图层都会被加载到GPU形成渲染纹理。GPU中的transform不会触发repaint，最终使用transform的图层会由独立的合成器进程进行处理

渲染过程：

+ 生成render tree
+ 渲染元素
+ 图层
+ GPU渲染
+ 浏览器复合图层
+ 生成最终的屏幕图像

<img src="https://lz5z.com/assets/img/css3_gpu_lauer_borders.png" style="zoom:50%;" />

在GPU渲染过程中，一些元素因为符合某些规则，被提升为独立的层，独立出来之后，就不会影响到其它DOM的布局，所以我们可以将经常变换的DOM主动提升到独立的层，就可以在浏览器的一帧运行中，减少Layout和Paint的时间。

#### 创建独立图层

+ transform属性
+ video元素
+ WebGL或canvas
+ Flash
+ opacity动画

#### 开启GPU加速

+ transform
+ opacity
+ filter
+ will-change

#### 问题

+ 开启过多的硬件加速会耗费更多的内存
+ GPU渲染会影响字体的抗锯齿效果，因为CPU和GPU有不同的渲染机制








































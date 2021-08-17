# CSS

### 水平垂直居中

### transform和transition

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
































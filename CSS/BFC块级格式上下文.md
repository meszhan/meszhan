# 块级格式上下文

### 什么是BFC

> BFC：Block Formatting Context，块级格式上下文
>
> BFC决定了元素如何对内容进行定位，以及与其他元素的关系和相互作用，当涉及到可视化布局时，BFC提供了一个环境，HTML在这个环境中按照一定的规则布局
>
> BFC是一个完全独立的空间，让空间里的子元素不会影响到外面的布局

### 如何触发BFC

+ overflow不为visible，因为它会影响到外部！！！
+ position：absolute/fixed
+ display：inline-block/table-cell
+ html根元素

### BFC的规则

> + BFC就是一个块级元素，块级元素在垂直方向连续排列
> + BFC是页面中的一个隔离的独立容器，里面的标签不会影响到外面的
> + 垂直方向的距离由margin决定，属于同一个BFC的两个相邻标签会发生重叠，因此可以用于防止margin重叠
> + 计算BFC高度时，会计算浮动元素，因此可以用于清除浮动

### BFC解决的问题

#### 解决高度塌陷

#### Margin重叠

#### 两栏布局


























































# CSS Animation动画实现

### 一个三角形从透明到实体，持续1s，然后匀速前进5s，然后变大持续2s

> 首先，题目中描述了多个动画，因此对于某个元素需要定义多个animation，以逗号（,）分隔，然后考虑具体实现细节
>
> + 从透明到实体：opacity属性从0到1，第一个keyframe，持续1s，animation-duration属性，而且需要保留最后一帧，因此需要animation-fill-mode属性设为forwards表示用最后一帧填充
> + 然后匀速前进5s：因为有“然后”，所以需要设置animation-delay属性为1s，表示等opacity动画执行完后执行，匀速前进需要控制动画的执行函数animation-timing-function设置为linear表示线性即匀速，其他属性类似
> + 然后变大持续2s：这里可以既可以控制width，也可以控制height，也可以用transform属性，不过效果是不一样的
>   + transform属性：利用scale或scaleX或scaleY都可以，但是这种扩张是基于中心点不变实现的，会改变四个角的位置
>   + 控制width和height不会改变左上角的位置

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








































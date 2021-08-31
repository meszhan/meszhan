# Q & A

### Vue的created生命周期中改变为data中的对象添加属性

```vue
<template>
	<div>
    {{a.b}}
  </div>
</template>

<script>
export default{
  data(){
    return {
			a:{}
    }
  },
  created(){
    this.a.b=1;
  },
  mounted(){
    this.a.b=2;
  }
}
</script>

// Vue3之前：1
// Vue3：2
```

### arguments对象和函数参数的关系

```javascript
let side = function (arr) {
  arr[0] = arr[2];
};

let f1 = function (a, b, c = 3) {
  c = 10;
  side(arguments);
  console.log(arguments); // 1 1 1
  console.log(a, b, c); // 1 1 10
};

let f2 = function (a, b, c) {
  c = 10;
  side(arguments);
  console.log(arguments); // 10 1 10
  console.log(a, b, c); // 10 1 10
};

f1(1, 1, 1);
f2(1, 1, 1);
```

这里主要看arguments中的数和函数参数是不是一一对应关系

+ 在f1中，因为给参数c传了默认值，所以c和arguments中的形参没有对应关系，修改c的值不会反馈到arguments中
+ 在f2中，arguments中的参数和c是对应的关系，对c的修改会反应到arguments中


























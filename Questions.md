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






























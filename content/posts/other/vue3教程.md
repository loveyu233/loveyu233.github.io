---
title: "Vue3"
date: 2023-05-29T19:16:59+08:00
author: ["loveyu"]
draft: false
categories: 
- Vue3
- 前端
- other
tags: 
- Vue3
- 前端
---

# Vite创建vue+ts

``npm create vite@latest ``

```shell
~ > npm create vite@latest                                                  
Need to install the following packages:
  create-vite@4.3.2
Ok to proceed? (y)
✔ Project name: … vite-project
✔ Select a framework: › Vue
✔ Select a variant: › TypeScript
Scaffolding project in /Users/huangzhenyu/logs/vite-project...
Done. Now run:
  cd vite-project
  npm install
  npm run dev
```



# 组合式Api

>使用``setup`` ,按需引入不再和vue2一样全部导入了

```vue
<script setup lang="ts">
</script>
```

>导入vue文件直接使用无需注册

```vue
<script setup lang="ts">
	import HelloWorld from "./components/HelloWorld.vue";
</script>

<template>
  <HelloWorld/>
</template>
```



# 响应式

>要实现响应式数据,不能直接赋值,需要使用``ref``和``reactive``
>
>``ref``: 主要为基本类型, 也可以为对象,当为对象是ref还是会调佣reactive
>
>``reactive``: 主要为对象

```vue
<template>
  <HelloWorld ref="hello"/>
</template>

<script setup lang="ts">
  let name:String = ref("test")
  let obj = reactive({
    a: "adad",
    b: 13
});
  // 获取HelloWorld标签,变量名称必须和标签的ref值一样
  let hello = ref()
  
  // 解构一个
  let a1 = toRef(obj, "a");
  console.log(a1);

  // toRefs 用来解构对象里面的值
  let {a, b} = toRefs(obj);

  console.log(a, b);
</script>
```



# 计算属性

```vue
<script setup lang="ts">
	let user = reactive({
    firstName: "张",
    lastName: "三"
});
let name = computed({
    get() {
        return user.firstName + "-" + user.lastName;
    },
    set(newValue: string) {
        [user.firstName, user.lastName] = newValue.split("-");
    }
});
  
  // watchEffect中使用的响应式数据发生改变时就会执行
  const stop = watchEffect((before) => {
    console.log(watchTest);
    before(() => {
        console.log("before先执行");
    });
}, {
    // post 组件加载完后执行
    // pre  组件更新前执行
    // sync 强制效果始终同步触发
    flush: "post"
});
  // 执行则会关闭这个监听
  stop()
</script>
```



# 父给子组件传递数据

``父组件.vue``

```vue
<template>
    <PropsEmit :str="str" @on_click="getName" :a="a"/>
</template>
```

``子组件.vue``

```vue
defineProps<{
  str: {
      type: String,
      default: "abc"
  },
  a: {
      a: String,
      b: Number
  },
  b: {
      type: String,
      default: "default"
  }
}>();
// 第一种接受方法回调方式
let emits = defineEmits(["on_click"]);
      
// 第二种,优点是可以指定参数类型
let emits = defineEmits<{
    (e: "on_click", name: string)
}>();
      
// 执行回调函数
emits("on_click", "hello");
```



# 子给父组件传递数据

``子组件.vue``

```vue
<script setup lang="ts">
	const de = ref("Hello from child component");

  const updateData = () => {
      de.value = "Updated data from child component";
      console.log("updateData");
  };

  defineExpose({
      de,
      updateData
  });
</script>
```

``父组件.vue``

```vue
<template>
    <PropsEmit ref="childRef"/>
</template>
<script setup lang="ts">
  let childRef = ref();
  // 必须在组件挂载完毕后才能调用子组件传递的数据,使用.value的方式获取变量或函数回调
  onMounted(() => {
      console.log(childRef.value.de);
      childRef.value.updateData();
      console.log(childRef.value.de);
  });
</script>
```

# 案例: 实现实时瀑布流

``app.vue``

```vue
<template>
	<Demo1 :list="list"/>
</template>
<script setup lang="ts">
import Demo1 from "./components/Demo1.vue";
const list = [
    {
        height: 300,
        background: "red"
    },
    {
        height: 400,
        background: "pink"
    }...多个
];
</script>
```

``Demo1.vue``

```vue
<template>
    <div class="wraps">
        <div
                class="items"
                v-for="item in waterList"
                :style="{
                           background:item.background,
                            height:item.height+'px',
                            left:item.left+'px',
                            top:item.top+'px'
                        }"
        ></div>
    </div>
</template>

<script setup lang="ts">
import {onBeforeUnmount, onMounted, reactive, ref} from "vue";

type L = {
    height: number,
    background: string,
    left: number,
    top: number
}
const props = defineProps<
    {
        list: L[]
    }
>();
// 存储div的长宽高和颜色,实际页面渲染使用的
let waterList = reactive<L[]>([]);
const init = () => {
    // 用来存储计算当前这一列的长度
    let heightList: number[] = [];
    // 获取当前body的宽度
    const x = document.body.clientWidth;
    // 设置默认div的宽度为130
    const width = 130;
    // 向下取整,得到当前窗口大小可以容纳多少列
    const colum = Math.floor(x / width);
    // 遍历得到每一个div的属性值
    for (let i = 0; i < props.list.length; i++) {
        // 如果 i < column , 说明这i个为第一行展示
        if (i < colum) {
            // 因为已经设置了div为绝对定位,所以这里要单独为每一个div设置一个left值和top值
            props.list[i].left = i * width;
            props.list[i].top = 10;
            // 把设置完成的元素添加到waterList
            waterList.push(props.list[i]);
            // 记录第一行的当前高度
            heightList.push(props.list[i].height);
            // else: 说明这些元素不是第一行的,需要再次设置其位置
        } else {
            // current: 记录最小的高度; index: 记录最小高度的索引值
            let current = heightList[0];
            let index = 0;
            // 遍历获取到heightList里面最小的高度值
            heightList.forEach((item, i) => {
                if (item < current) {
                    current = item;
                    index = i;
                }
            });
            // 设置最小的元素的left和top
            props.list[i].left = index * width;
            props.list[i].top = current + 20;
            // 重新把值添加到heightList
            heightList[index] += props.list[i].height + 20;
            // 元素添加到waterList渲染到页面
            waterList.push(props.list[i]);
        }
    }
};
// 先调用一次
init();
onMounted(() => {
    // 挂载中监听当前页面的宽度,实现跟随当前页面宽度实时变化
    window.addEventListener("resize", init);
});
onBeforeUnmount(() => {
    // 页面销毁时取消监听
    window.removeEventListener("resize", init);
});
</script>

<style scoped>

.wraps {
    position: relative;
    height: 100%;
}

.items {
    position: absolute;
    width: 120px;
}
</style>
```


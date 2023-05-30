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



# 全局组件

>在``main.ts``中注册的组件可以在全局使用

``main.ts``

```tsx
import HelloWorld from "./components/HelloWorld.vue";

App.component("helloWorld", HelloWorld);
```



# 递归组件

``app.vue``

```vue

<template>
    <HelloWorld :data="data"/>
</template>

<script setup lang="ts">
import HelloWorld from "./components/HelloWorld.vue";
import {reactive} from "vue";

interface C {
    name: string,
    check: boolean,
    children?: C[]
}

let data = reactive<C[]>([{
    name: "1",
    check: false,
    children: [{
        name: "1-1",
        check: false
    }, {
        name: "1-2",
        check: false
    }]
}, {
    name: "2",
    check: false
}, {
    name: "3",
    check: false
}, {
    name: "4",
    check: false,
    children: [{
        name: "4-1",
        check: false,
        children: [{
            name: "4-1-1",
            check: false
        }]
    }, {
        name: "4-2",
        check: false
    }]
}]);

</script>
```



``helloWorld.vue``

>使用:``<HelloWorld v-if="item?.children?.length" :data="item?.children as C[]"/>`` 

```vue
<script setup lang="ts">
interface C {
    name: string,
    check: boolean,
    children?: C[]
}

const props = defineProps<{
    data: C[]
}>();

const e = (item) => {
    console.log(item.name);
};
</script>

<template>
    <div
            class="tree"
            v-for="item in props.data"
            :key="item.name"
            @click.stop="e(item)"
    >
        <input type="checkbox" v-model="item.check"><span>{{ item.name }}</span>
        <HelloWorld v-if="item?.children?.length" :data="item?.children as C[]"/>
    </div>
</template>

<style scoped>
.tree {
    margin-left: 20px;
}
</style>
```



# 动态组件

>``<component :is="comId"/>`` 其中``is``指定的即为展示的组件

## 性能调优

>``markRaw``:reactive对象中对于不需要深层代理的可以使用markRow声明
>
>``shallowRef``:同理

分别创建A,B,C三个vue组件

``app.vue``

```vue
<template>
    <div style="display: flex">
        <div
                v-for="(item,index) in data"
                class="item"
                :class="[active == index ? 'active':'']"
                @click="swatchActive(item,index)"
        >
            {{ item.name }}
        </div>
    </div>
    <component :is="comId"/>
</template>

<script setup lang="ts">
import AVue from "./components/AVue.vue";
import BVue from "./components/BVue.vue";
import CVue from "./components/CVue.vue";
import {markRaw, reactive, ref, shallowRef} from "vue";

let comId = shallowRef(AVue);

let active = ref(0);

let data = reactive([
    {
        name: "AVue",
        com: markRaw(AVue)
    }, {
        name: "BVue",
        com: markRaw(BVue)
    }, {
        name: "CVue",
        com: markRaw(CVue)
    }
]);

const swatchActive = (item, index) => {
    active.value = index;
    comId.value = item.com;
};
</script>

<style scoped>
.item {
    border: 1px black solid;
    padding: 10px;
    margin: 10px;
    cursor: pointer;
}

.active {
    background-color: skyblue;
}
</style>

```



# 插槽

>语法:``v-slot可以简写为#`` 如果有``:`` 可以省略不写
>
>匿名插槽: ``v-slot``==``#`` 绑定到``<slot></slot>`` 
>
>具名插槽: ``v-slot:slotName`` == ``#slotName`` 绑定到``<slot name="slotName"></slot>`` 
>
>作用域插槽: ``v-slot:slotName="slotDataName"`` == ``#slotName=[slotDataName]`` 绑定到``<slot name="slotName" :slotDataName="data"></slot>``  

``AVue.vue``

```vue
<template>
    <div style="display: flex;width: 900px">
        <div>
            <slot></slot>
        </div>
        <div>
            <slot name="center"></slot>
        </div>
        <div>
            <slot name="bottom" :data="data"></slot>
        </div>
    </div>
</template>

<script setup lang="ts">
import {reactive} from "vue";

let data = reactive(["A", "B", "C"]);
</script>

<style scoped>
div {
    width: 300px;
    height: 300px;
    color: red;
    border: 1px solid aqua;
}
</style>
```

``App.vue``

```vue
<template>
    <AVue>
        <template v-slot>
            <div>
                匿名插槽
            </div>
        </template>
        <template v-slot:center>
            <div>
                具名插槽
            </div>
        </template>
        <template v-slot:bottom="data">
            <div>
                作用域插槽(获取子组件传递的数据): {{ data }}
            </div>
        </template>
        <template v-slot:[slotName]>
            <div>
                这是动态插槽,会覆盖原本插槽的数据,只能作用于具名插槽
            </div>
        </template>
    </AVue>
  {{ slotName }}
    <button @click="slotName === 'center' ? slotName='bottom' : slotName='center'">动态插槽切换</button>
</template>

<script setup lang="ts">
import AVue from "./components/AVue.vue";
import {ref} from "vue";

let slotName = ref("center");
</script>

<style scoped>

</style>
```



# 异步组件

>在大型应用中，我们可能需要将应用分割成小一些的代码块 并且减少主包的体积
>
>这时候就可以使用异步组件
>
>默认打包的话是在一个js文件中,使用异步组件的js会单独在一个js文件,在需要的时候才会引入,以此来减少js文件体积做到优化的目的



## await

>在setup语法糖里面 使用await
>
>``<script setup> 中可以使用顶层 await。结果代码会被编译成 async setup()``

```vue
<script setup>
  const post = await fetch(`/api/post/1`).then(r => r.json())
</script>
```



## defineAsyncComponent

>异步获取组件,父组件引用子组件 通过defineAsyncComponent加载异步配合import 函数模式便可以分包

```vue
<script setup lang="ts">
import { reactive, ref, markRaw, toRaw, defineAsyncComponent } from 'vue'
 
const Dialog = defineAsyncComponent(() => import('../../components/Dialog/index.vue'))
 
//完整写法
const AsyncComp = defineAsyncComponent({
  // 加载函数
  loader: () => import('./Foo.vue'),
 
  // 加载异步组件时使用的组件
  loadingComponent: LoadingComponent,
  // 展示加载组件前的延迟时间，默认为 200ms
  delay: 200,
 
  // 加载失败后展示的组件
  errorComponent: ErrorComponent,
  // 如果提供了一个 timeout 时间限制，并超时了
  // 也会显示这里配置的报错组件，默认值是：Infinity
  timeout: 3000
})
```



## suspense

>配合defineAsyncComponent使用
>
><suspense> 组件有两个插槽。它们都只接收一个直接子节点。default 插槽里的节点会尽可能展示出来。如果不能，则展示 fallback 插槽里的节点。

```vue
<Suspense>
  <template #default>
   	需要展示的组件
  </template>

  <template #fallback>
    default插槽里面的组件没有展示前会展示这里的组件
  </template>
</Suspense>
```



# 传送组件

>使用``Teleport``标签包裹的才可以被传送,``to="标签选择器"`` 设置传送到那个下面,``disabled="布尔值"`` 设置是否传送,true不传送,false传送

``默认,也就是没有使用Teleport``

![loveyu 4](https://www.loveyu.asia//img/loveyu 4.png)

``使用``

```vue
<template>
    <Teleport to="body" :disabled="true">
        <AVue/>
    </Teleport>
</template>
```

![loveyu 5](https://www.loveyu.asia//img/loveyu 5.png)





# 缓存组件

>include: 缓存那些组件
>
>exclude:不缓存那些组件
>
>max:最大缓存几个,旧的不常用的

```vue
<template>
    <KeepAlive :include="[]" :exclude="[]" :max="10">
        <AVue v-if="flag"/>
        <BVue v-else/>
    </KeepAlive>
    <button @click="flag=!flag">切换</button>
</template>
```

>使用KeepAlive会有新的两个声明周期,这两个生命周期只有被缓存的组件才会有

```vue
<script setup lang="ts">
import {onActivated, onDeactivated} from "vue";

onActivated(() => {
    console.log("KeepAlive初始化");
});
onDeactivated(() => {
    console.log("KeepAlive切换");
});
</script>
```

# 过渡

## 基本使用

>``<transition name="demo">``: css中的动画样式必须以这个name值为开头
>
>``.demo-enter-from, .demo-leave-to``: 显示前和消失后
>
>``.demo-enter-active, .demo-leave-active``: 显示过程中和消失过程中
>
>``.demo-enter-to, .demo-leave-from ``: 显示后,消失前

```vue
<template>
    <button @click="flag = !flag">显示/隐藏</button>
    <transition name="demo">
        <div class="d" v-if="flag"></div>
    </transition>

</template>

<script setup lang="ts">
import {ref} from "vue";

let flag = ref<boolean>(true);

</script>

<style scoped>
.d {
    width: 300px;
    height: 300px;
    background-color: aqua;
}

.demo-enter-from, .demo-leave-to {
    width: 0;
    height: 0;
}

.demo-enter-active, .demo-leave-active {
    transition: 1s all linear;
}

.demo-enter-to, .demo-leave-from {
    width: 300px;
    height: 300px;
}
</style>
```



## 自定义类名

>自定义指定过渡动画的类名, 主要是为了使用第三方过滤动画库

```vue
<transition
  enter-active-class=""
  enter-from-class=""
  enter-to-class=""
  leave-active-class=""
  leave-from-class=""
  leave-to-class=""
>
  <div class="d" v-if="flag"></div>
</transition>
```



## 使用animate.css

>官网: https://animate.style/
>
>下载: npm i animate.css
>
>引入: import "animate.css";
>
>``:duration="{enter:1000,leave:2000}"`` : 设置动画时长,直接使用``:duration="2000"`` 则显示和消失都是2000,单位为毫秒
>
>``新版本必须在原有类名称上加上前缀animate__animated `` 

```vue
<transition
    enter-active-class="animate__animated animate__fadeIn"
    leave-active-class="animate__animated animate__fadeOut"
    :duration="{enter:1000,leave:2000}"
>
<div class="d" v-if="flag"></div>
</transition>
```



## transition 生命周期8个

>结合``https://greensock.com/``使用

```vue
<transition
    @before-enter="beforeEnter" 				//对应enter-from
    @enter="enter"											//对应enter-active
    @after-enter="afterEnter"						//对应enter-to
    @enter-cancelled="enterCancelled"		//显示过度打断
    @before-leave="beforeLeave"					//对应leave-from
    @leave="leave"											//对应enter-active
    @after-leave="afterLeave"						//对应leave-to
    @leave-cancelled="leaveCancelled"		//离开过度打断   
>
    <div class="d" v-if="flag"></div>
</transition>

<script setup>
const beforeEnter = (el: Element) => {
  console.log('进入之前from', el);
}
const Enter = (el: Element,done:Function) => {
  console.log('过度曲线');
  setTimeout(()=>{
     done()
  },3000)
}
const AfterEnter = (el: Element) => {
  console.log('to');
}
</script>
```



## 页面载入时触发

>必须有``appear``,不设置值默认为true

```vue
appear
appear-active-class=""
appear-from-class=""
appear-to-class=""
```



>结合动画库使用

```vue
<transition
    appear
    appear-active-class="animate__animated animate__backOutDown"
>
<div class="d" v-if="flag"></div>
</transition>
```



## 过渡列表

> 当需要对一个for循环的元素添加动画时就适合使用
>
> 使用方式和之前的一模一样
>
> ``tag``: 外层包裹的标签名,不写就没有
>
> ``class``: 包裹标签名的class

```vue
<transition-group tag="div" class="">
		<div v-for="...">
  		...
  	</div>
</transition-group>
```

## 位移动画

>``move-class``: 当transition-group里面元素发生位移改变时的动画

```vue
<transition-group
    tag="div"
    class="items"
    move-class="mc"
>
<div v-for="item in list" :key="item.id" class="item">
    {{ item.number }}
</div>
</transition-group>


.mc {
    transition: all 1s;
}
```





# 案例

## 实现实时瀑布流

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



## 实现点击div添加背景色

>``.active {
>    background-color: skyblue;
>}`` : 提前声明active的样式
>
>``let active = ref(0);``: 声明变量,默认展示索引为0的
>
>`` :class="[active == index ? 'active':'']"``: 动态绑定class名称
>
>`` @click="swatchActive(index)"``: 添加点击事件
>
>``const swatchActive = (index) => {
>    active.value = index;
>};``: 修改active值为新的index值

```vue
<template>
    <div style="display: flex">
        <div
                v-for="(item,index) in data"
                class="item"
                :class="[active == index ? 'active':'']"
                @click="swatchActive(index)"
        >
            {{ item.name }}
        </div>
    </div>
</template>

<script setup lang="ts">
import {markRaw, reactive, ref, shallowRef} from "vue";

let active = ref(0);

let data = reactive([
    {
        name: "AVue"
    }, {
        name: "BVue"
    }, {
        name: "CVue"
    }
]);

const swatchActive = (index) => {
    active.value = index;
};
</script>

<style scoped>
.item {
    border: 1px black solid;
    padding: 10px;
    margin: 10px;
    cursor: pointer;
}

.active {
    background-color: skyblue;
}
</style>

```



## 骨架屏

>使用异步组件实现
>
>分别需要三个组件: 骨架屏组件,实际展示数据组件,父组件用来引用展示

``骨架屏组件.vue``

```vue
<template>
    <div class="main">
        <div class="top">
            <div class="left">

            </div>
            <div class="right">

            </div>
        </div>
        <div class="bottom">

        </div>
    </div>
</template>

<script setup lang="ts">
</script>

<style scoped>
.main {
}

.top {
    display: flex;
}

.top .left {
    width: 50px;
    height: 50px;
    border-radius: 50%;
    background-color: darkgray;
}

.right {
    height: 10px;
    width: 50px;
    background-color: darkgray;
    margin-left: 20px;
}

.bottom {
    margin-top: 30px;
    height: 10px;
    width: 100px;
    background-color: darkgray;
}
</style>
```

``实际展示数据组件.vue``

```vue
<template>
    <div class="main">
        <div class="top">
            <div class="left">
                <img :src="data.url">
            </div>
            <div class="right">
                {{ data.name }} -- {{ data.age }}
            </div>
        </div>
        <div class="bottom">
            {{ data.desc }}
        </div>
    </div>
</template>

<script setup lang="ts">
import {axios} from "../server/axios.ts";

interface D {
    "data": {
        "name": string,
        "age": number,
        "url": string,
        "desc": string
    };
}

const {data} = await axios.get<D>("./data.json");
</script>

<style scoped>
.main {
}

.top {
    display: flex;
}

.top .left img {
    width: 50px;
    height: 50px;
}

.right {
    margin-left: 20px;
}

.bottom {
    margin-top: 30px;
}
</style>
```

``app.vue``

```vue
<template>
    <Suspense>
        <template #default>
            <SyncVue/>
        </template>
        <template #fallback>
            <AVue/>
        </template>
    </Suspense>
</template>

<script setup lang="ts">
import {defineAsyncComponent} from "vue";
import AVue from "./components/AVue.vue";

const SyncVue = defineAsyncComponent(() => import("./components/Sync.vue"));

</script>
```

``axios.ts``

```ts
export const axios = {
    get<T>(url: string): Promise<T> {
        return new Promise((resolve) => {
            const xhr = new XMLHttpRequest();
            xhr.open("GET", url);
            xhr.onreadystatechange = () => {
                if (xhr.readyState == 4 && xhr.status == 200) {
                    setTimeout(() => {
                        resolve(JSON.parse(xhr.responseText));
                    }, 2000);
                }
            };
            xhr.send(null);
        });
    }
};
```



## 位移动画

```vue
<template>
    <button @click="random">random</button>
    <transition-group
            tag="div"
            class="items"
            move-class="mc"
    >
        <div v-for="item in list" :key="item.id" class="item">
            {{ item.number }}
        </div>
    </transition-group>
</template>

<script setup lang="ts">
import {ref} from "vue";
import _ from "lodash";

interface T {
    id: number,
    number: number
}

let list = ref(Array.apply(null, {length: 81}).map((_, index) => {
    return {
        id: index,
        number: (index % 9) + 1
    };
}));

const random = () => {
    list.value = _.shuffle(list.value);
};

setInterval(() => {
    random();
}, 1000);

</script>

<style scoped>
.items {
    display: flex;
    flex-wrap: wrap;
    width: calc(30px * 10);
}

.item {
    width: 30px;
    height: 30px;
    border: 1px solid #ccc;
}

.mc {
    transition: all 1s;
}
</style>
```

















# 其他

## 问号操作符

```js
变量?.属性	// 如果这个属性不存在不会报错而是返回undefined,undefined隐式转换就是false
这个只能用于undefined和null,对于其他则无效
// 配合双问号使用
变量?.属性 ?? 返回值	// 当这个属性不存在就返回指定的返回值
```


















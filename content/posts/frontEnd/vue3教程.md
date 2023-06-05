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



>解决ts不能识别vue文件

`` vite-env.d.ts ``

```ts
/// <reference types="vite/client" />
declare module "*.vue" {
    import { DefineComponent } from "vue"
    const component: DefineComponent<{}, {}, any>
    export default component
}
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



# 依赖注入,组件数据传输

>只能传递给有关系的组件,也就是使用过的
>
>例如:app使用a,a使用b

```ts
父组件传递值: provide(value,key)	如果不想让别的组件修改这个key可以使用 readonly(color)
后代组件接收值: let color = inject<ref<string>>(value, ref("默认值"));
如果父组件没有设置readonly,后代组件就可以修改值:color.value = "red";直接修改即可
```

``app.vue``

```vue
<template>
    <input v-model="color" value="red" type="radio">红色
    <input v-model="color" value="green" type="radio">绿色
    <input v-model="color" value="black" type="radio">黑色
    <div class="div">

    </div>
    <AVue/>
</template>

<script setup lang="ts">
import {provide, readonly, ref} from "vue";
import AVue from "./components/AVue.vue";


let color = ref<string>("red");
provide("color", readonly(color));


</script>

<style scoped>
.div {
    width: 200px;
    height: 200px;
    background-color: v-bind(color);
}
</style>

```

``a.vue``

```vue
<template>
    <div>
        AVue
    </div>
    <div class="div">

    </div>
    <BVue/>
</template>

<script setup lang="ts">
import {inject, Ref} from "vue";
import BVue from "./BVue.vue";

let color = inject<Ref<string>>("color");
</script>
<style scoped>
.div {
    width: 200px;
    height: 200px;
    background-color: v-bind(color);
}
</style>

```

``b.vue``

```vue
<template>
    <div>
        BVue
    </div>
    <div class="div">

    </div>
    <button @click="change">修改颜色</button>
</template>

<script setup lang="ts">

import {inject, Ref} from "vue";

let color = inject<Ref<string>>("color");

const change = () => {
    color!.value = "red";
};
</script>
<style scoped>
.div {
    width: 200px;
    height: 200px;
    background-color: v-bind(color);
}
</style>

```



# 双向数据流

>defineProps和defineEmits实现修改父组件传递的值并返回
>
>``传递给子组件的数据如果子组件想要修改并返回值就必须使用v-model不能省略这个前缀``
>
>`` v-model:text="text" 如果省略前缀就会导致子组件修改的值无法传递给父组件``
>
>子组件修改数据的语法:
>
>``let emits = defineEmits(["update:modelValue", "update:text"]);``
>
>必须是``update:``开头
>
>使用:``emits("update:text", target.value);`` 第二个参数为要修改后的值

``父组件.vue``

```vue
<template>
    <button @click="b = !b">change</button>
    <h1>App -- {{ b }} -- {{ text }}</h1>
    <AVue v-model="b" v-model:text="text"/>
</template>

<script setup lang="ts">

import AVue from "./components/AVue.vue";
import {ref} from "vue";

let b = ref<boolean>(true);
let text = ref<string>("hello");
</script>
```

``子组件``

```vue
<template>
    <h1>AVue -- {{ modelValue }} -- {{ text }}</h1>
    <button @click="change">AVue修改</button>
    <input :value="text" @input="textChange">
</template>

<script setup lang="ts">
let props = defineProps<{
    modelValue: boolean,
    text: string
}>();
let emits = defineEmits(["update:modelValue", "update:text"]);
const change = () => {
    emits("update:modelValue", !props.modelValue);
};
const textChange = (e: Event) => {
    let target = e.target as HTMLInputElement;
    emits("update:text", target.value);
};
</script>
```



# 自定义指令

>自定义指令的名称必须是``v自定义名称``,例如:``vDemo``, 多个单词用驼峰方式命名
>
>自定义指令语法:``const vDemo: Directive = {生命周期钩子}`` 
>
>``Directive<HTMLButtonElement, string> `` Directive接受两个类型,分别为参数一的和参数二的类型
>
>每个生命周期钩子都有参数:``el,dir``:为别为:``使用自定义组件的元素,使用自定义组件携带的数据`` 
>
>如果只需要mount和update的话不需要声明别的函数,直接赋值一个函数即可,在mount和update时就会执行这个函数
>
>```ts
>const vDemo: Directive = (el: HTMLElement, dir: DirectiveBinding) => {
>    console.log(el);
>    console.log(dir);
>};
>```
>
>

```vue
<template>
    <h1 v-demo.qwe:asdas="{name:'zxc'}">helloWorld</h1>
</template>

<script setup lang="ts">
import {Directive, DirectiveBinding} from "vue";

const vDemo: Directive = {
    created() {

    },
    beforeMount() {

    },
    mounted(el: HTMLElement, dir: DirectiveBinding) {
        console.log(el);
        console.log(dir);
        console.log(dir.modifiers);
        console.log(dir.value.name);
    },
    updated() {

    },
    beforeUpdate() {

    },
    beforeUnmount() {

    },
    unmounted() {

    }
};
</script>
```



# 自定义hook

>把重复使用的函数给抽离出来,在每个需要的组件里面再引用,到达代码的复用
>
>自定义hook一般以``usr``开头,再接名称

``hook/index.ts``

```ts
export default function (data: string): string {
    return data + "====";
}
```



``*.vue``

```vue
<template>

</template>

<script setup lang="ts">
import h from "./hook/index.ts";

console.log(h("demo"));
</script>

<style scoped>
```



# 全局属性

## 添加全局属性

>注意点:
>
>1. 要先声明出app ``const app = createApp(App);``
>2. 添加语法: ``app.config.globalProperties.$env = "dev"``
>3. 如果添加的是方法则用对象方式
>4. 必须给添加的变量声明出来``declare module "@vue/runtime-core"`` ,否则编辑器会提示错误但运行没问题的
>5. 必须最后再挂载``app.mount("#app");`` ,不然全局属性无法添加成功

``main.ts``

```ts
import {createApp} from "vue";
import App from "./App.vue";

const app = createApp(App);

app.config.globalProperties.$env = "dev";
app.config.globalProperties.$fun = {
    f(str: string): string {
        return `test - ${str}`;
    }
};
type F = {
    f(str: string): string
}
declare module "@vue/runtime-core" {
    export interface ComponentCustomProperties {
        $env: string,
        $fun: F;
    }
}

app.mount("#app");
```



## 使用

>注意点:
>
>1. 可以使用插值语法直接获取全局属性``<div>{{ $env }}</div>`` 
>2. 可以使用``getCurrentInstance`` 获取到全局属性
>   1. 必须在onMounted钩子里面才可以成功获取到属性
>3. 使用``解构``的方式可以更快的获取到属性 ``const {appContext: {config: {globalProperties}}} = getCurrentInstance();`` 
>   1. ``globalProperties.属性名称`` 
>4. 不使用``解构``的方式
>   1. ``let app = getCurrentInstance();
>      console.log(app?.proxy?.$env);
>      console.log(app?.proxy?.$fun.f("sad"));`` 
>   2. vue已经给属性添加到``proxy``中,所以可以直接读取到

``*.vue``

```vue
<template>
    <div>{{ $env }}</div>
    <div>{{ $fun.f("adadada") }}</div>
</template>

<script setup lang="ts">
import {getCurrentInstance, onMounted} from "vue";

onMounted(() => {
    const {appContext: {config: {globalProperties}}} = getCurrentInstance();
    console.log(globalProperties.$env);
    console.log(globalProperties.$fun.f("asd"));
});
</script>
```



# 自定义插件

``loaing/index.vue``

```vue
<template>
    <div v-if="isShow">
        loading...
    </div>
</template>

<script setup lang="ts">
import {ref} from "vue";

let isShow = ref<boolean>(false);

const show = () => isShow.value = true;
const hide = () => isShow.value = false;
defineExpose({
    show,
    hide,
    isShow
});
</script>

<style scoped>
div {
    background-color: black;
    opacity: 0.8;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    color: white;
}
</style>
```

``loading/index.ts``

```ts
import {App, createVNode, render} from "vue";
import Loading from "./index.vue";

export default {
    // 对象形式写插件必须有install方法
    install(app: App) {
        // 把Loading组件转为虚拟节点
        const vnode = createVNode(Loading);
        // 把这个虚拟节点挂载到body下
        render(vnode, document.body);
        // 添加全局属性让其他组件可以调用
        app.config.globalProperties._loading = {
            isShow: vnode.component?.exposed?.isShow,
            show: vnode.component?.exposed?.show,
            hide: vnode.component?.exposed?.hide
        };
    }
};

type show = {
    isShow: string,
    show: () => void,
    hide: () => void
}
// 声明,不写编辑器报错但不影响运行,写了编辑器就不会报错了
declare module "@vue/runtime-core" {
    export interface ComponentCustomProperties {
        _loading: show;
    }
}
```

## 使用插件

>1. ``main.ts``里面use使用一下
>
>2. 其他组件
>
>   1. 访问全局属性必须在``onMounted``钩子函数中才可以调用
>
>   2. ```vue
>      onMounted(() => {
>          const ins = getCurrentInstance();
>          ins?.proxy?._loading.show();
>          setTimeout(() => {
>              ins?.proxy?._loading.hide();
>          }, 2000);
>      });
>      ```
>
>      

``main.ts``

```ts
import {createApp} from "vue";
import App from "./App.vue";
import Loading from "./components/loading/index.ts";

const app = createApp(App);
app.use(Loading);
app.mount("#app");
```



# 样式穿透

## deep

``AVue.vue``

```vue
<template>
    <div>
        第一层
        <input>
    </div>
</template>
```

``App.vue``

```vue
<template>
    <AVue class="avue"/>
</template>

<script setup lang="ts">
import AVue from "./components/AVue.vue";
</script>

<style scoped>
/*直接添加样式只能添加到组件的最外层*/
.avue {
    color: red;
    background-color: aqua;
}

/*如果不添加deep是无法添加上样式*/
.avue :deep(input) {
    background-color: aquamarine;
}
</style>
```

## slotted

>插件中使用插槽,直接对插槽内的元素添加样式是无法生效的,必须添加``slotted``才可以生效

``*.vue``

```vue
<template>
    <div>
        插槽
        <slot>

        </slot>
    </div>
</template>

<script setup lang="ts">

</script>

<style scoped>
:slotted(input) {
    background-color: red;
}
</style>
```



## global

>``scoped``设置里面直接设置的样式只在本组件生效,添加``global``可以让这个样式在全局中都生效

``*.vue``

```vue
<style scoped>
:global(h1) {
    background-color: skyblue;
}
</style>
```

## v-bind

>让css样式的值设置为一个ts里面的变量
>
>基本类型可以直接设置,对象形式需要用引号设置``'style.color'`` 

``*.vue``

```vue
<script setup lang="ts">
import AVue from "./components/AVue.vue";
import {reactive, ref} from "vue";

let color = ref("red");

let style = reactive({
    "color": "red"
});
</script>

<style scoped>
.div {
    color: v-bind('style.color');
}
</style>
```

## module

>使用``module`` 可以动态的给元素添加样式``:class="[$style.div,$style.bo]"`` 
>
>单个样式不用写成数组形式

``*.vue``

```vue
<template>
    <div :class="[$style.div,$style.bo]">
        asas
    </div>
</template>

<script setup lang="ts">

</script>

<style module>
.div {
    width: 100px;
    height: 100px;
    color: red;
}

.bo {
    border: 1px solid red;
}
</style>
```

## module自定义名称

>``module="xy"``: 设置自定义的值,后边使用的时候就不能使用默认的``$style`` 而是使用``xy``自定义的值代替

``*.vue``

```vue
<template>
    <div :class="[xy.div,xy.bo]">
        asas
    </div>
</template>

<script setup lang="ts">

</script>

<style module="xy">
.div {
    width: 100px;
    height: 100px;
    color: red;
}

.bo {
    border: 1px solid red;
}
</style>
```

## useCssModule

>``useCssModule``: 获取样式列表
>
>如果module没有设置值则直接module()即可,如果module设置了值则需要在参数中指定值

``*.vue``

```vue
<template>
    <div :class="[xy.div,xy.bo]">
        asas
    </div>
</template>

<script setup lang="ts">
import {useCssModule} from "vue";

const um = useCssModule("xy");
/*
{div: '_div_1ukg8_2', bo: '_bo_1ukg8_8'}bo: "_bo_1ukg8_8"div: "_div_1ukg8_2"[[Prototype]]: Object
 */
console.log(um);
</script>

<style module="xy">
.div {
    width: 100px;
    height: 100px;
    color: red;
}

.bo {
    border: 1px solid red;
}
</style>
```



# 添加环境变量

## 添加

>创建`` .env.development `` 和 `` .env.production`` 两个文件,分别为开发环境和生产环境

`` .env.development ``

```txt
VITE_TEST = DEVELOPMENT
```

`` .env.production`` 

```txt
VITE_TEST = PRODUCTION
```

## 使用

>``import.meta.env);`` 可以获取到全部的变量,其中包括设置的
>
>根据运行的环境不同分别读取不同环境下的变量文件

```ts
console.log(import.meta.env);
```

## vite获取变量

>先安装: ``npm i --save-dev @types/node``

``vite.config.ts``

>先解构出``mode``
>
>再使用``loadEnv``获取自定义环境变量

```ts
import {defineConfig, loadEnv} from "vite";
import vue from "@vitejs/plugin-vue";

export default defineConfig(({mode}: any) => {
    console.log(loadEnv(mode, process.cwd()));
    return {
        plugins: [vue()]
    };
});
```



# 自定义组件

## 原生方法

``btn.js``

```js
class Btn extends HTMLElement {
    constructor() {
        super();
        const shaDom = this.attachShadow({mode: "open"})
        
        // 第一种方法
        // this.p = this.h("p")
        // this.p.innerText = "测试"
        // this.p.setAttribute("style", "width:200px;height:200px;border:1px solid red")
        // shaDom.appendChild(this.p)

        // 第二种方法
        this.template = this.h("template")
        this.template.innerHTML = `
            <style>
                div{
                    width: 200px;
                    height: 200px;
                    border: 1px solid red;
                }
            </style>
            <div>测试233</div>
        `
        shaDom.appendChild(this.template.content.cloneNode(true))
    }

    h(el) {
        return document.createElement(el)
    }
  
      /**
     * 生命周期
     */
    //当自定义元素第一次被连接到文档 DOM 时被调用。
    connectedCallback() {
        console.log('我已经插入了！！！嗷呜')
    }

    //当自定义元素与文档 DOM 断开连接时被调用。
    disconnectedCallback() {
        console.log('我已经断开了！！！嗷呜')
    }

    //当自定义元素被移动到新文档时被调用
    adoptedCallback() {
        console.log('我被移动了！！！嗷呜')
    }

    //当自定义元素的一个属性被增加、移除或更改时被调用
    attributeChangedCallback() {
        console.log('我被改变了！！！嗷呜')
    }
}

window.customElements.define("xy-btn", Btn)
```

``index.html``

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="btn.js"></script>
</head>
<body>
  <xy-btn></xy-btn>
</body>
</html>
```

![loveyu-5792241](https://www.loveyu.asia//img/loveyu-5792241.png)

## vue中使用

``vite.config.ts``

```ts
import {defineConfig} from "vite";
import vue from "@vitejs/plugin-vue";

export default defineConfig(({mode}: any) => {
    return {
      	// 添加配置
        plugins: [vue({
            template: {
                compilerOptions: {
                    // 以 xy- 的都会跳过组件检测
                    isCustomElement: (tag) => tag.includes("xy-")
                }
            }
        })]
    };
});

```

``custom-vue.ce.vue``

>必须是``.ce.vue``结尾, 接受参数时,参数直接在标签上携带

```vue
<template>
    <div class="div">
        <h1>
            CUSTOM
        </h1>
    </div>
</template>

<script setup lang="ts">
defineProps<{
    obj: any
}>();
</script>

<style scoped>
.div {
    width: 200px;
    height: 200px;
    border: 1px solid red;
}
</style>
```

``app.vue``

```vue
<template>
    <xy-btn :obj="JSON.stringify(obj)"></xy-btn>
</template>

<script setup lang="ts">
import customVueCe from "./components/custom-vue.ce.vue";
import {defineCustomElement, reactive} from "vue";

const btn = defineCustomElement(customVueCe);
window.customElements.define("xy-btn", btn);

let obj = {
    name: "张三"
};

</script>

<style module="xy">

</style>
```

``携带参数:`` 

![loveyu-5793242](https://www.loveyu.asia//img/loveyu-5793242.png)



# pinia

>注册

``main.ts``

```ts
import {createPinia} from "pinia";

const app = createApp(App);

app.use(createPinia());

```

``store/index.ts``

>``export const useTestStore = defineStore("nameSpace1", { state:()=>{return {} }, getters:{},actions:{} }`` 
>
>可以声明多个,pinia不需要显示的注册,只需要声明即可

```ts
import {defineStore} from "pinia";


type User = {
    name: string,
    age: number
}
const login = (): Promise<User> => {
    return new Promise<User>(resolve => {
        setTimeout(() => {
            resolve({
                name: "哈哈哈",
                age: 13123
            });
        }, 2000);
    });
};
export const useTestStore = defineStore("nameSpace1", {
    state: () => {
        return {
            name: "张三",
            age: 19,
            user: {}
        };
    },
    getters: {
        getUser(): string {
            return `name:${this.name} --- age:${this.age}`;
        }
    },
    actions: {
        updateName(name: string) {
            this.name = name;
        },
        async login() {
            const res = await login();
            this.user = res;
        }
    }
});

```



``*.vue``

>使用

```vue
<template>
    <h1>origin: {{ Test.name }} --- {{ Test.age }} --- {{ Test.user }}</h1>
    <h1>解构: {{ name }} --- {{ age }}</h1>
    <h1>getters: {{ Test.getUser }}</h1>
    <button @click="change">change</button>
    <button @click="reset">reset重置到原始值</button>
</template>

<script setup lang="ts">
import {useTestStore} from "./store";
import {storeToRefs} from "pinia";

const Test = useTestStore();

/*
修改state值的五种方式
1.       Test.age = 123; Test.name = "啊哈哈哈";
2.       Test.$patch({age: 666, name: "啊哈哈哈"});
3.       Test.$patch((state) => {
          state.age = 9999;
          state.name = "阿莎哈哈";
         });
4. 必须全部修改   Test.$state = {
        age: 123,
        name: "asadsdsa"
    };
5.在action里面定义方法,然后调用  Test.updateName("哈哈哈哈");
6.通过storeToRefs解构为响应式数据 let {name, age} = storeToRefs(Test);
 */

let {name, age} = storeToRefs(Test);
const change = () => {
    // Test.age++;
    Test.login();
};

const reset = () => {
    Test.$reset();
};

Test.$subscribe((mutation, state) => {
    console.log(mutation, state);
});
Test.$onAction((args) => {
    args.after(() => {
        console.log("after");
    });
    console.log(args);
});

</script>

<style module="xy">

</style>
```



## 自定义持久化插件

``localStore.ts``

```ts
import {PiniaPluginContext} from "pinia";
import {toRaw} from "vue";

type O = {
    key?: string
}
export const piniaPlugin = (options: O) => {
    return (content: PiniaPluginContext) => {
        const {store} = content;
        const data = getLocalStore(`${options?.key ?? "default"}-${store.$id}`);
        store.$subscribe(() => {
            setLocalStore(`${options?.key ?? "default"}-${store.$id}`, toRaw(store.$state));
        });
        return {
            ...data
        };
    };

};

const setLocalStore = (key: string, value: any) => {
    localStorage.setItem(key, JSON.stringify(value));
};

const getLocalStore = (key: string) => {
    return localStorage.getItem(key) ? JSON.parse(localStorage.getItem(key) as string) : {};
};
```

``main.ts``

>``store.use``即可使用

```ts

const app = createApp(App);
const store = createPinia();


store.use(piniaPlugin({
    key: "xy"
}));


app.use(store);
```



# Vue-Router

## 安装

```shell
npm i vue-router
```

## 声明

``router/index.ts``

```ts
import {createRouter, createWebHashHistory, RouteRecordRaw} from "vue-router";

const routes: Array<RouteRecordRaw> = [{
    path: "/a",
    name: "aVue",
    component: () => import("../components/A.vue")
}, {
    path: "/b",
    name: "bVue",
    component: () => import("../components/B.vue")
}
];
export default createRouter({
    history: createWebHashHistory(),
    routes
});
```

## use使用

``main.ts``

```ts
import {createApp} from "vue";
import App from "./App.vue";
import router from "./router";

const app = createApp(App);
app.use(router);
app.mount("#app");
```



## 显示/跳转

>``<router-view></router-view>`` 放到哪.就显示在哪
>
>不留下历史记录:
>
>声明式: ``<router-link replace to="/a">AVue</router-link>`` 添加replace
>
>编程式:``vueRouter.replace`` 使用``vueRouter.replace({...})``

``app.vue``

```vue
<template>
    <h1>App</h1>
    <div>
        <router-link to="/a">AVue</router-link>
        <router-link to="/b">BVue</router-link>
    </div>
    <div>
        <router-link :to="{name:'aVue'}">AVue</router-link>
        <router-link :to="{name:'bVue'}">BVue</router-link>
    </div>
    <div>
        <button @click="toPage1('/a')">AVue</button>
        <button @click="toPage1('/b')">BVue</button>
    </div>
    <div>
        <button @click="toPage2('aVue')">AVue</button>
        <button @click="toPage2('bVue')">BVue</button>
    </div>

    <router-view></router-view>
</template>

<script setup lang="ts">
import {useRouter} from "vue-router";

const vueRouter = useRouter();

const toPage1 = (url: string) => {

    // vueRouter.push(url);

    vueRouter.push({
        path: url
    });
};
const toPage2 = (url: string) => {
    vueRouter.push({
        name: url
    });
};
</script>

<style scoped>

</style>
```



## 前进后退

```ts
const go = () => {
    vueRouter.go(1);
};

const back = () => {
    vueRouter.back();
};
```



## 参数传递

``index.json``

```json
{
  "data": [
    {
      "id": 1,
      "name": "脚踩老坛酸菜",
      "price": 100
    },
    {
      "id": 2,
      "name": "伤肺火腿肠",
      "price": 300
    },
    {
      "id": 3,
      "name": "翡翠玉石",
      "price": 200
    }
  ]
}
```



### query

``传递.vue``

```vue
<template>
    <table>
        <tr v-for="item in data" :key="item.id">
            <td>{{ item.id }}</td>
            <td>{{ item.name }}</td>
            <td>{{ item.price }}</td>
            <td>
                <button @click="info(item)">详情</button>
            </td>
        </tr>
    </table>
</template>

<script setup lang="ts">
import {data} from "../data/idnex.json";
import {useRouter} from "vue-router";

type Item = {
    id: number,
    name: string,
    price: number
}

const router = useRouter();
const info = (item: Item) => {
    router.push({
        path: "/b",
        query: item
    });
};

</script>

<style scoped>

</style>
```

``接收.vue``

```vue
<template>
    <h1>详情</h1>
    <h2>ID: {{ route.query.id }}</h2>
    <h2>Name: {{ route.query.name }}</h2>
    <h2>Price: {{ route.query.price }}</h2>
    <button @click="back">返回</button>
</template>

<script setup lang="ts">
import {useRoute, useRouter} from "vue-router";

const route = useRoute();
const router = useRouter();
const back = () => {
    router.push("/");
};
</script>

<style scoped>

</style>
```



## params

>新版本已经不支持直接传递参数,详细:https://github.com/vuejs/router/blob/main/packages/router/CHANGELOG.md#414-2022-08-22
>
>使用params必须在url上声明才可以

``router/index.ts``

>``path: "/b/:id",`` 在路径后边声明

```tsx
import {createRouter, createWebHashHistory, RouteRecordRaw} from "vue-router";

const routes: Array<RouteRecordRaw> = [{
    path: "/",
    name: "aVue",
    component: () => import("../components/A.vue")
}, {
    path: "/b/:id",
    name: "bVue",
    component: () => import("../components/B.vue")
}
];
export default createRouter({
    history: createWebHashHistory(),
    routes
});
```

传递

>``params: {id: item.id}`` 必须和path声明的一致

```tsx
const info = (item: Item) => {
    router.push({
        name: "bVue",
        params: {id: item.id}
    });
};
```

接收

```ts
const route = useRoute();
route.params.id
```



## 子路由/重定向/别名

>``children``: 主路由,页面跳转时必须携带上父路由
>
>``redirect``: 重定向,当页面跳转到这个路由时会重定向到指定的路由,写法可以说字符串也可以是对象
>
>``alias``: 别名,可以起多个,这些别名都指向一个组件

```ts

const routes: Array<RouteRecordRaw> = [{
    path: "/",
    name: "aVue",
    redirect: "/a",
    alias: ["/root", "/root1", "/root2"],
    component: () => import("../components/A.vue"),
    children: [{
        path: "/a",
        name: "aVue",
        component: () => import("../components/A.vue")
    }, {
        path: "/b",
        name: "aVue",
        component: () => import("../components/A.vue")
    }]
}, {
    path: "/b/:id",
    name: "bVue",
    component: () => import("../components/B.vue")
}
];
```



## 前置守卫后置守卫

```ts
/*
to: Route， 即将要进入的目标 路由对象；
from: Route，当前导航正要离开的路由；
next(): 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed (确认的)。
next(false): 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 from 路由对应的地址。
next('/') 或者 next({ path: '/' }): 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。
*/
router.beforeEach((to, form, next) => {
    console.log(to, form);
    next()
})


const whileList = ['/']
router.beforeEach((to, from, next) => {
    let token = localStorage.getItem('token')
    //白名单 有值 或者登陆过存储了token信息可以跳转 否则就去登录页面
    if (whileList.includes(to.path) || token) {
        next()
    } else {
        next({
            path:'/'
        })
    }
})
```



## 路由元信息

>获取: ``直接.mate.key`` 即可

```ts
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      component: () => import('@/views/Login.vue'),
      meta: {
        title: "登录"
      }
    },
    {
      path: '/index',
      component: () => import('@/views/Index.vue'),
      meta: {
        title: "首页",
      }
    }
  ]
})

// 声明类型,不然调用的时候编辑器会报错	
declare module 'vue-router' {
  interface RouteMeta {
    title?: string
  }
}
```



## 添加路由过渡效果

>先下载``animate``css

添加meta信息,用来设置不同的过渡效果

```ts
declare module 'vue-router'{
     interface RouteMeta {
        title:string,
        transition:string,
     }
}
 
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      component: () => import('@/views/Login.vue'),
      meta:{
         title:"登录页面",
         transition:"animate__fadeInUp",
      }
    },
    {
      path: '/index',
      component: () => import('@/views/Index.vue'),
      meta:{
         title:"首页！！！",
         transition:"animate__bounceIn",
      }
    }
  ]
})
```



``*.vue``

```vue
    <router-view #default="{route,Component}">
        <transition  :enter-active-class="`animate__animated ${route.meta.transition}`">
            <component :is="Component"></component>
        </transition>
    </router-view>
```



## 动态路由

>主要就是:``router.addRoute({ path: '/about', component: About })`` 
>
>请求后端返回数据,然后使用addRouter添加路由
>
>如果添加与现有途径名称相同的途径，会先删除路由，再添加路由：

```ts
// addRoute有返回值调用就会删除
const removeRoute = router.addRoute(routeRecord)
removeRoute() // 删除路由如果存在的话


// 删除路由,当路由被删除时，所有的别名和子路由也会被同时删除
router.removeRoute('about')

router.hasRoute()：检查路由是否存在。
router.getRoutes()：获取一个包含所有路由记录的数组。

													//这儿不能使用@
component: () => import(`../views/${v.component}`)
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



## 使用自定义指定实现权限校验

>一般用于对按钮的鉴权,后端返回用户的权限,然后根据用户的权限来判断是否要显示这个按钮

```vue
<template>
    <button v-button-auth="'shop:buttonInsert'">添加</button>
    <button v-button-auth="'shop:buttonDelete'">删除</button>
    <button v-button-auth="'shop:buttonUpdate'">修改</button>
</template>

<script setup lang="ts">
import {Directive} from "vue";

localStorage.setItem("userid", "zs");

let authList = [
    "zs:shop:buttonInsert",
    "zs:shop:buttonDelete",
    "zs:shop:buttonUpdate"
];

let userid = localStorage.getItem("userid");
const vButtonAuth: Directive<HTMLButtonElement, string> = (el, {value}) => {
    if (!authList.includes(userid + ":" + value)) {
        el.style.display = "none";
    }
};
</script>

<style scoped>
button {
    width: 50px;
    height: 50px;
    border: 1px solid #ccc;
    margin-left: 50px;
}
</style>
```



## 使用自定义指令实现拖拽移动元素

>只要给div添加v-move自定义指令即可实现随意拖拽移动

```vue
<template>
    <div class="box" v-move>
        <div class="header"></div>
        <div>内容</div>
    </div>
</template>

<script setup lang="ts">
import {Directive, DirectiveBinding} from "vue";

const vMove: Directive<any, void> = (el: HTMLElement, dir: DirectiveBinding) => {
    let element: HTMLDivElement = el.firstElementChild as HTMLDivElement;
    const elMove = (e: MouseEvent) => {
        // el.offsetLeft 返回当前元素距离某个父辈元素左边缘的距离
        // e.clientX 返回鼠标距离浏览器边框的距离
        // 相减获取鼠标距离当前元素边框的距离
        let x = e.clientX - el.offsetLeft;
        let y = e.clientY - el.offsetTop;
        const move = (e: MouseEvent) => {
            // 相减获取元素移动后的位置
            el.style.left = e.clientX - x + "px";
            el.style.top = e.clientY - y + "px";
        };
        // 添加鼠标移动事件
        document.addEventListener("mousemove", move);
        // 添加鼠标松开事件
        document.addEventListener("mouseup", () => {
            // 取消鼠标移动事件
            document.removeEventListener("mousemove", move);
        });
    };
    // 添加鼠标按下事件
    element.addEventListener("mousedown", elMove);
};
</script>

<style>

.box {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 200px;
    height: 200px;
    border: 1px solid #ccc;
}

.header {
    height: 20px;
    background: black;
    cursor: move;
}
</style>
```



## 使用自定义指令实现图片懒加载

>``IntersectionObserver``: 监视方法

```vue
<template>
    <div>
        <img v-for="i in arr" alt="a" v-lazy="i">
    </div>
</template>

<script setup lang="ts">
import {Directive} from "vue";
// 获取指定目录下的指定多个文件,eager: true 设置为静态加载,不设置默认为懒加载
let imageList: Record<string, { default: string }> = import.meta.glob("./images/*.*", {eager: true});
// 获取到路径
let arr = Object.values(imageList).map(v => v.default);
let vLazy: Directive<HTMLImageElement, string> = async (el, dir) => {
    // 导入默认的图片
    const icon = await import("./assets/vue.svg");
    // 设置默认图片
    el.src = icon.default;
    // 设置监听回调函数
    const observer = new IntersectionObserver((entries) => {
        // intersectionRatio: 返回这个元素的可视比例
        let intersectionRatio = entries[0].intersectionRatio;
        // 当这个元素的可视比列大于0说明已经能看到这个元素了,设置为实际要展示的图片即可
        if (intersectionRatio > 0) {
            // 凸显效果
            setTimeout(() => {
                // 设置src为实际要载入的图片
                el.src = dir.value;
            }, 1000);
            // 取消监听元素
            observer.unobserve(el);
        }
    });
    // 设置监听哪一个元素
    observer.observe(el);
};
</script>

<style scoped>
img {
    width: 400px;
    height: 500px;
}
</style>
```



## 使用自定义hook实现把img转为base64

``hook/index.ts``

```ts
import {onMounted} from "vue";

type options = {
    el: string
}
export default function (options: options): Promise<{ baseUrl: string }> {
    return new Promise(resolve => {
        onMounted(() => {
            // 获取到元素
            let img: HTMLImageElement = document.querySelector(options.el) as HTMLImageElement;
            // 当元素加载完毕
            img.onload = () => {
                resolve({
                    baseUrl: base(img)
                });
            };
        });
        const base = (img: HTMLImageElement) => {
            // 创建canvas用来对图片转base64
            let canvas: HTMLCanvasElement = document.createElement("canvas");
            let ctx = canvas.getContext("2d");
            canvas.width = img.width;
            canvas.height = img.height;
            // drawImage: 给img的图片绘制上去
            ctx?.drawImage(img, 0, 0, img.width, img.height);
            // toDataURL: 转为base64
            return canvas.toDataURL("image/png");
        };
    });
}
```

``*.vue``

```vue
<template>
    <img src="./assets/vue.svg" id="img">
</template>

<script setup lang="ts">
import toBaseUrl from "./hook/index.ts";

toBaseUrl({el: "img"}).then(res => {
    console.log(res.baseUrl);
});
</script>
```

## 实现自定义hook和自定义指令监听dom的宽高变化并上传到npm

```shell
1. 创建文件夹: V-RESIZE-XY
2. 创建文件夹: src
3. 创建ts文件: src/index.ts
4. 创建ts声明文件: V-RESIZE-XY/index.d.ts
5. 创建vite配置文件: V-RESIZE-XY/vite.config.js
6. 执行命令:
	// 初始化npm项目
	npm init
	// 初始化ts项目
	tsc --init
	// 安装vue和vite, 必须使用-D,因为这是给vue项目使用的所以不需要要再次安装vue和vite
	npm i vue -D
	npm i vite -D
7. npm账户命令:
	// 登录账号 
	npm adduser
	// 验证是否登录
	npm who am i
8. 发布到npm
	npm publish
```

``index.ts``

```ts
/*
ResizeObserver: 主要监听元素宽高变化
MutationObserver: 监听子集的变化和属性的变化以及增删改查
IntersectionObserver: 监听可视区域的变化
 */
import {App} from "vue";

function useResize(el: HTMLElement, callback: Function) {
    let resize = new ResizeObserver((entries) => {
        callback(entries[0].contentRect);
    });
    resize.observe(el);
}

const install = (app: App) => {
    app.directive("resize", (el, binding) => {
        useResize(el, binding.value);
    });
};

useResize.install = install;

export default useResize;
```

``index.d.ts``

```ts
import {App} from "vue";

declare const useResize: {
    (el: HTMLElement, callback: Function): void
    install: (app: App) => void;
};

export default useResize;
```

``vite.config.js``

```js
import {defineConfig} from "vite";

export default defineConfig({
    build: {
        lib: {
            entry: "src/index.ts",
            name: "useResize"
        },
        rollupOptions: {
            external: ['vue'],
            output: {
                globals: {
                    useResize: "useResize"
                }
            }
        }
    }
})
```

``package.json``

>main: requery引入需要
>
>module: import引入需要
>
>files: 配置上传的文件

```json
{
  "name": "v-resize-xy",
  "version": "1.0.0",
  "description": "使用hook或指令监听dom元素宽高的变化",
  "main": "dist/v-resize-xy.umd.js",
  "module": "dist/v-resize-xy.mjs",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "vite build"
  },
  "files": [
    "dist",
    "index.d.ts"
  ],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "vite": "^4.3.9",
    "vue": "^3.3.4"
  }
}
```

### 在vue项目中引用

```shell
npm i v-resize-xy
```

>hook使用

```vue
<template>
    <div class="test">
        asdasda
    </div>
</template>

<script setup lang="ts">
import useResize from "v-resize-xy";
import {onMounted} from "vue";

// 必须要在onMounted才能使用
onMounted(() => {
    useResize(document.querySelector(".test"), (res) => {
        console.log(res);
    });
});
</script>

<style scoped>
.test {
    width: 100px;
    height: 100px;
    border: 1px solid red;
    overflow: hidden;
    resize: both;
}
</style>
```

>指令使用
>
>需要先在``main.ts``中使用一下这个插件,然后在其他组件中就可以直接使用这个指令

``main.ts``

```ts
import {createApp} from "vue";
import App from "./App.vue";
import useResize from "v-resize-xy";

createApp(App).use(useResize).mount("#app");
```

``*.vue``

```vue
<template>
    <div class="test" v-resize="callback">
        asdasda
    </div>
</template>

<script setup lang="ts">
const callback = (res) => {
    console.log(res);
};
</script>
```





![loveyu](https://www.loveyu.asia//img/loveyu-20230601102614346.png)





# 其他

## 问号操作符

```js
变量?.属性	// 如果这个属性不存在不会报错而是返回undefined,undefined隐式转换就是false
这个只能用于undefined和null,对于其他则无效
// 配合双问号使用
变量?.属性 ?? 返回值	// 当这个属性不存在就返回指定的返回值
```



## 非空操作符

```js
color!.value = "red";
变量!.属性 = 新值
```



## css绑定变量

>``v-bind(color)``: 值为变量名称

```vue
<template>
    <div class="div">

    </div>
</template>

<script setup lang="ts">
import {ref} from "vue";

let color = ref<string>("red");

</script>

<style scoped>
.div {
    width: 200px;
    height: 200px;
    background-color: v-bind(color);
}
</style>
```



## Event类型

>参数为``e: Event`` 指定类型,可以获取target但是无法获取input元素的value,所以需要转一下类型
>
>``let target = e.target as HTMLInputElement;`` 即可``target.value`` 

```ts
const textChange = (e: Event) => {
    let target = e.target as HTMLInputElement;
    emits("update:text", target.value);
};
```



## 批量载入静态资源

```ts
// eager: true 设置为静态载入,不设置默认为懒加载
let imageList: Record<string, { default: string }> = import.meta.glob("./images/*.*", {eager: true});
```



## http-server

```shell
npm install http-server -g
http-server -p 端口号
```



## 性能优化

``https://xiaoman.blog.csdn.net/article/details/126811832?spm=1001.2014.3001.5502`` 



## 定义@别名代表src

``vite.config.ts``

>可以定义多个`` "@": fileURLToPath(new URL("./src", import.meta.url))`` 

```ts
export default defineConfig(({mode}: any) => {
    console.log(loadEnv(mode, process.cwd()));
    return {
        resolve: {
            alias: {
                "@": fileURLToPath(new URL("./src", import.meta.url))
            }
        }
    };
});

```


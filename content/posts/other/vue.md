---
title: "Vue2"
date: 2022-12-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- Vue2
- 前端
- other
tags: 
- vue2
- 前端
---

# 指令

|                             语法                             |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                          {{ key }}                           |                           插值句法                           |
|              v-bind:value="key" / :value="key"               |                           单向绑定                           |
|             v-model:value="key" / v-model="key"              |                 双向绑定,只能用于有value值的                 |
|     @keydown / @keyup / @keydown/keyup.键位名称.键位名称     |                           键盘事件                           |
| @click.prevent / @click.stop / @click.once / @click.capture / @click.self / @click.passive | 默认事件 / 冒泡 / 只执行一次 / 捕获模式 / event.target是当前元素在执行 / 不等待事件处理完 |
|              v-if / v-else-if / v-else / v-show              | v-if 条件不成立则没有这个元素, v-show则是使用display隐藏这个元素 |
|       v-for="(key, value/index) in data" :key="唯一值"       |                   key里面的值必须是唯一的                    |
|              v-text / v-html / v-once / v-cloak              |        文本 / 解析html / 只执行一次 / 渲染完毕后删除         |
|                          ref="key"                           |                     获取 this.#refs.key                      |
|              this.$nextTick(function () { ... }              |                下一次dom更新时执行的回调方法                 |

# 计算属性

```js
computed: {
  // 计算的值
  info: {
    	// 获取这个值时调用
      get() {
          return ...
      },
      // 设置值时调用,value为设置的值
      set(value) {
          ...
      }
  }
}

// 如果只读不修改的话可以简写为:
computed: {
  info() {
      return ...
  }
}
```



# 监视属性

```js
watch: {
  name: {
      // immediate页面打开直接调用一次,用来初始化
      immediate: true,
      handler(newdata, olddata) {
          console.log("newData: ", newdata, "  oldData: ", olddata)
      }
  },
  // 监视里面某一个属性可以写: "number.a" 必须加上引号不然格式不对,默认不写引号是因为简写
  number: {
      // 开启深度监视,否则无法直接监视number里面的属性变化
      deep: true,
      handler(newDta, oldData) {
          console.log(newDta, " ", oldData)
      }
  },
  // 当只有handler时简写:
  demo(newData, oldData) {
      console.log(newData, oldData)
      // 这里必须是箭头方式的,因为只有箭头的没有this向外招,招到demo,demo的this是vue
      // 不使用箭头的this就是wind
      setTimeout(() => {
          console.log("")
      }, 1000)
  }
}
```

# 表单绑定

```html
<body>
<div id="root">
    <form @submit.prevent="submit">
        <!--     v-model.number 把输入转为数字   -->
        账号:<input type="number" v-model.number="userinfo.phone"><br><br>
        <!--     v-model.trim 去掉前后空格   -->
        密码:<input type="password" v-model.trim="userinfo.password"><br><br>
        性别:<input type="radio" name="sex" value="1" v-model="userinfo.sex">男
        <input type="radio" name="sex" value="0" v-model="userinfo.sex">女<br><br>
        爱好:<input type="checkbox" value="a" v-model="userinfo.hobby">A
        <input type="checkbox" value="b" v-model="userinfo.hobby">B
        <input type="checkbox" value="c" v-model="userinfo.hobby">C<br><br>
        所属:<select v-model="userinfo.city">
        <option value="">选择</option>
        <option value="北京">北京</option>
        <option value="上海">上海</option>
    </select><br><br>
        <!--    v-model.lazy 当失去焦点的时候再收集    -->
        其他信息:<textarea v-model.lazy="userinfo.other"></textarea><br><br>
        <input type="checkbox" v-model="userinfo.agree">接受阅读协议<a>用户协议
    </a><br><br>
        <button>提交</button>
    </form>
</div>
</body>
<script>
    new Vue({
        el: "#root",
        data: {
            userinfo: {
                phone: "",
                password: "",
                sex: "",
                hobby: [],
                city: "",
                other: "",
                agree: false
            }
        },
        methods: {
            submit() {
                console.log(JSON.stringify(this.userinfo))
            }
        }
    })
</script>
```



# 过滤器

```html
<body>
<div id="root">
    <!--    过滤器只能用在插值语法或者v-bind-->
    <h1>当前时间戳: {{ time }}</h1>

    <h1>计算属性 {{ filterTime }}</h1>
    <h1>过滤器无参 {{ time | timePar }}</h1>
    <h1>过滤器带参 {{ time | timePar("YYYY-MM") }}</h1>
    <h1>多个过滤器结合使用 {{ time | timePar | jiequ }}</h1>
    <h1>全局过滤器: {{ time | timePar | quanju }}</h1>
</div>
</body>
<script>
    //    全局过滤器
    Vue.filter("quanju", function (value) {
        return value.slice(0, 1)
    })
    new Vue({
        el: "#root",
        data: {
            time: 1681548595292
        },
        computed: {
            filterTime() {
                return dayjs(this.time).format("YYYY-MM-DD HH:mm:ss")
            }
        },
        filters: {
            timePar(value, fomt = "YYYY-MM-DD HH:mm:ss") {
                return dayjs(this.time).format(fomt)
            },
            jiequ(value) {
                return value.slice(0, 4)
            }
        }
    })
</script>
```



# 自定义指令

```html
<body>
<div id="root">
    <h1>n: {{ n }}</h1>
    <!--    使用自定义指令,作用就是把n*10后展示-->
    <h1>n*10: <span v-big="n"></span></h1>
    <!--   页面打开就获取焦点 -->
    <input v-fbind:value="n">
    <button @click="n++">n+1</button>
    <h1>如果指定名称是多个单词则需要用 ' - ' 分割,例如: big-number, 创建时: 'big-number'(){...}</h1>
</div>
</body>
<script>
    //    全局指令
    Vue.directive('big1', function (element, bindalue) {
        //     内容
    })
    Vue.directive('big2', {
        bind(element, bindalue) {
            //     内容
        },
        inserted(element, bindalue) {
            //     内容
        },
        update(element, bindalue) {
            //     内容
        }
    })
    new Vue({
        el: "#root",
        data: {
            n: 1,
        },
        directives: {
            // 精简写法,相当于bind + update
            big(element, bindalue) {
                // this 为 wind 不是vm
                element.innerText = bindalue.value * 10
            },
            fbind: {
                // 指令与元素绑定后
                bind(element, bindalue) {
                    element.value = bindalue.value
                },
                // 指令所在元素被插入页面后
                inserted(element, bindalue) {
                    element.focus()
                },
                // 模版重新解析时调用
                update(element, bindalue) {
                    element.value = bindalue.value
                }
            }
        }
    })
</script>
```



# props 父组件给子组件传值

```js
// 父组件传值:
<!--        直接使用 name=""传递的是字符串 使用 :age="" 绑定形式传递的是语法格式-->
<HelloWorld name="张三" :age="11"></HelloWorld>


// 子组件接收:

// 简写
// props: ["name", "age"]

// 限制传递数据的类型
// props: {
//     name: String,
//     age: Number
// }

// 详细写法
props: {
    name: {
        type: String,   // 限制类型
        required: true, // 是否为必须的值
    },
    age: {
        type: Number,
        default: 100,   // 没有这个值的话默认值是多少
    }
}
```



# mixin混入

>抽取实例之间相同的数据或方法然后共用

``minxin.js``

```js
export const mixin1 = {
    // 里面可以写所有的,例如:data,methods等等,如果数据和实例数据重复以实例的为准,如果是挂载方法则全都用,mixin的先执行
    methods: {
        showName() {
            alert(this.name)
        }
    }
}
export const mixin2 = {
    data() {
        return {
            x: 100,
            y: 100
        }
    }
}
```

``xxx.vue 局部引用``

```vue
<template>
    <div>
        <h1 @click="showName">{{ name }}</h1>
        <h1>{{ x }}</h1>
        <h1>{{ y }}</h1>
    </div>
</template>

<script>
// 局部引用
import {mixin1, mixin2} from "../mixin"

export default {
    name: 'HelloWorld',
    data() {
        return {
            name: "张三"
        }
    },
    // 使用
    mixins: [mixin1, mixin2]
}
```

``main.js 全局引用``

```js
// 全局引用
import {mixin1, mixin2} from "./mixin"
// 全局使用
Vue.mixin(mixin1)
Vue.mixin(mixin2)
```



# 插件

``plugins.js``

```js
export default {
    // 参数就是vue,方法名必须是install,可以在这个方法里面注册过滤器,自定义指令,混入等等
    install(vue) {
        // 过滤器
        vue.filter("slice", function (value) {
            return value.slice(0, 4)
        })

        // 自定义指令
        vue.directive("get", {
            // 指令与元素绑定后
            bind(element, bindalue) {
                element.value = bindalue.value
            },
            // 指令所在元素被插入页面后
            inserted(element) {
                element.focus()
            },
            // 模版重新解析时调用
            update(element, bindalue) {
                element.value = bindalue.value
            }
        })

        // 混入
        vue.mixin({
            data() {
                return {
                    x: 100,
                    y: 100
                }
            }
        })
    }
}
```

``main.js``

```js
import plugins from "./plugins"

// 使用插件
Vue.use(plugins)
```

>可以在使用插件时传递自己想要的参数,例如:
>
>Vue.user(plugins,1,2,3)
>
>在plugins.js的install(vue,a,b,c)接受传递的参数,其中a就是1,b是2,c是3



# 自定义事件

>``子组件给父组件传递数据`` 
>
>
>
>第一种方法是立即绑定事件,第二种方法可以做到延时绑定事件,也就是在做完某些事后再绑定事件
>
>
>
>组件上绑定的事件都会默认为是自定义事件包括@click,变为原生dom事件就需要使用``@click.native=""`` ,使用native修饰
>
>所有的自定义事件都可以绑定修饰符,例如``.once``只触发一次等等修饰符



## 第一种方法

>在要传递消息的组件上绑定一个自定义事件,自定义事件的回调方法声明在接受数据的组件上
>
>绑定使用 ``v-on:自定义时间名称="回调方法" 或 @自定义事件名称="回调方法"`` 
>
>传递消息触发时间使用 ``this.$emit(事件名称,传递的数据)`` 

``App.vue``

```vue
<template>
    <div id="app">
        <HelloWorld @getNameData="getName"></HelloWorld>
    </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
    name: 'App',
    components: {
        HelloWorld
    },
    methods: {
        getName(name) {
            console.log("App收到data: ", name)
        }
    },
}
</script>
```

``HelloWorld.vue``

```vue
<template>
    <div>
        <button @click="sendName">点击向父组件传递name</button>
    </div>
</template>

<script>
export default {
    name: 'HelloWorld',
    data() {
        return {
            name: "张三",
            age: "18"
        }
    },
    methods: {
        sendName() {
            this.$emit("getNameData", this.name)
        }
    }
}
</script>
```



## 第二种方法

>使用``ref="xxx"`` 然后在挂载钩子上给需要传递数据的组件绑定事件``this.$refs.hello.$on("getAgeData", this.getAge)``  
>
>传递数据组件依旧使用``this.$emit(事件名称,传递的数据)`` 触发事件

``App.vue``

>1. this.$refs.hello.$on("getAgeData", this.getAge)
>
>2. 使用箭头函数
>
>   1. ```vue
>      this.$refs.hello.$on("getAgeData", (age)=>{ console.log(age) })
>      ```
>
>   2. 一定不能使用function(){}这种形式,箭头函数的this是当前组件,function的this是调用组件

```vue
<template>
    <div id="app">
        <HelloWorld ref="hello"></HelloWorld>
    </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
    name: 'App',
    components: {
        HelloWorld
    },
    methods: {
        getAge(age) {
            console.log("App收到data: ", age)
        }
    },
    mounted() {
        this.$refs.hello.$on("getAgeData", this.getAge)
    }
}
</script>
```

``HelloWorld.vue``

```vue
<template>
    <div>
        <button @click="sendAge">点击向父组件传递age</button>
    </div>
</template>

<script>
export default {
    name: 'HelloWorld',
    data() {
        return {
            name: "张三",
            age: "18"
        }
    },
    methods: {
        sendAge() {
            this.$emit("getAgeData", this.age)
        }
    }
}
</script>


```

## 解除绑定事件

>在那个实例上绑定的就在那个实例上解除绑定

```js
// 解绑一个
this.$off("getAgeData")

// 解绑多个用数组
this.$off(["getAgeData","getNameData"])

// 不传参数解绑全部
this.$off()
```



# 全局事件总线

``main.js``

```js
new Vue({
    render: h => h(App),
    beforeCreate() {
        // 安装全局事件总线
        Vue.prototype.$bus = this
    }
}).$mount('#app')

```

>使用: ``this.$bus.on("事件名称","回调方法")`` 
>
>触发: ``this.$emit("事件名称","参数")`` 
>
>谁绑定的就一定要在绑定实例销毁时``beforeDestroy``解绑事件: ``this.$off("事件名称")`` 一定不能解绑全部事件因为这个``bus`` 还有别的实例在用



# 消息订阅与发布

>安装
>
>``npm i pubsub-js``
>
>订阅: ``this.pubId = pubsub.subscribe("订阅的名字", (定于的名字,接收的数据) => {...})``返回订阅id, ``第一个参数不是传递的数据而是订阅名称``
>
>发布: `` pubsub.publish("订阅的名字", 传递的数据)`` 
>
>取消订阅: `` pubsub.unsubscribe(订阅id)``  



## 订阅消息

>用来接受消息

```vue
<script>
import pubsub from "pubsub-js"

export default {
    name: 'HelloWorld',
    data() {
        return {}
    },
    mounted() {
      	// 返回一个订阅id, 取消订阅需要使用这个id所以把这个id方法this身上
        this.pubId = pubsub.subscribe("test", (data) => {
            console.log("HelloWorld: ", data)
        })
    },
    beforeDestroy() {
      	// 取消订阅消息用id来取消订阅
        pubsub.unsubscribe(this.pubId)
    }
}
```



## 发布消息

>用来发送数据

```vue
<script>
import pubsub from "pubsub-js";

export default {
    name: "Stu",
    data() {
        return {
            name: "张三",
        }
    },
    methods: {
        sendName() {
          	// 发布数据
            pubsub.publish("test", this.name)
        }
    }
}
</script>
```



# 动画

```vue
<!-- transition: 只能用于一个元素的   name: 这个动画的名称,  appear 页面加载就执行一次进入动画   -->
<transition name="hello" appear>
    <h1 class="tar" v-show="show">transition</h1>
</transition>
<hr/>
<transition-group name="hello" appear>
    <h1 class="tar" v-show="!show" key="1">transition-group</h1>
    <h1 class="tar" v-show="show" key="2">transition-group</h1>
</transition-group>

<style>
h1 {
    background-color: aqua;
}

/*进入的起点, 离开的终点*/
.hello-enter, .hello-leave-to {
    transform: translateX(-100%);
}

/*进入的活动, 离开的活动*/
.hello-enter-active, .hello-leave-active {
    transition: 0.5s linear;
}

/*进入的终点, 离开的起点*/
.hello-enter-to, .hello-leave {
    transform: translateX(0);
}
</style>

```



## 第三方库

```shell
npm install animate.css
```

引用

```vue
import 'animate.css';
```

使用

```vue
<transition-group
        appear
        name="animate__animated animate__bounce"
        enter-active-class="animate__heartBeat"
        leave-active-class="animate__backOutUp"
>
    <h1 class="tar" v-show="show" key="2">transition-group</h1>
</transition-group>
```



# 请求

## 解决跨域

``vue.config.js``

```js
module.exports = {
    // 方法一
    // devServer: {
    //     proxy: 'http://localhost:9888'
    // }
    // 方法二
    devServer: {
        proxy: {
          	// 前缀,所有是这个前缀的请求才会走代理,例如: http://127.0.0.1/demo/stu
            "/demo": {
              	// 要请求的后端服务器地址
                target: "http://127.0.0.1:9888",
              	// 因为后端的请求中没有 /demo 这个前缀所以需要去掉这个前缀把 /demo/stu 变为 /stu
                pathRewrite: {"^/demo": ""},
              	// websocket
              	ws: true,
              	// 是否请求的端口和服务器端口一样,true就是一样,false就是代理服务器的端口号
                changeOrigin: true
            },
          	// 可以配置多个服务
          	"": {
            },...
        }
    }
}
```



## Get

```js
// 这里的/demo/stu 会被过滤为 /stu
axios.get("http://localhost:8081/demo/stu").then(
  Response => {
      console.log(Response.data)
  },
  Error => {
      console.log(Error.message)
  }
)
```



## Post

```js
axios.post("http://localhost:8081/demo/car").then(
  Response => {
      console.log(Response.data)
  },
  Error => {
      console.log(Error.message)
  }
)
```



# 插槽

## 默认插槽

``app.vue``

```vue
<HelloWorld>
  <h1>这是默认插槽</h1>
</HelloWorld>
```

``HelloWorld.vue``

```vue
<div>
  <slot>没有内容时显示</slot>
</div>
```

## 具名插槽

``app.vue``

>``v-slot:demo1`` 这个标签必须放到``template`` 标签中才可以使用

```vue
<HelloWorld2>
  <template v-slot:demo1>
      <h1>这个内容放到名字叫demo1的插槽中</h1>
      <h1>这个内容放到名字叫demo1的插槽中</h1>
  </template>
  <h1 slot="demo2">这个内容放到名字叫demo2的插槽中</h1>
</HelloWorld2>
```

``HelloWorld2.vue``

```vue
<div>
  <slot name="demo1">demo1</slot>
  <slot name="demo2">demo2</slot>
</div>
```



## 作用域插槽

>必须使用``template`` 标签包裹
>
>``v-slot:demo="demoData"``  ``v-slot`` 关键字 ``:demo`` 作用到``名字叫demo的插槽中`` `` ="demoData"`` 接受插槽的数据到`` demoData`` 的对象上
>
>``v-slot:demo="{testData}"`` 解析``demoData.testData``的数据到``testData`` 

``app.vue``

```vue
<HelloWorld3>
  <template v-slot:demo="demoData">
      <h1>{{ demoData.testData }}</h1>
  </template>
</HelloWorld3>

// 或使用:
<template v-slot:demo="{testData}">
  <h1>{{ testData }}</h1>
</template>
```

``HelloWorld3.vue``

```vue
<div>
  <slot name="demo" :testData="slotData"></slot>
</div>
data() {
  return {
      slotData: "内容来自使用 <slot></slot> 的组件中"
  }
},
```



# Vuex

>数据共享

## 环境搭建

```shell
npm i vuex@3
```



``store/index.js``

```js
import Vue from "vue";
import Vuex from "vuex"

Vue.use(Vuex)

const actions = {}

const mutations = {}

const state = {}

const getters = {}

export default new Vuex.Store({
    actions, mutations, state, getters
})
```

``main.js``

```js
import Vue from 'vue'
import App from './App.vue'

// 引入创建的js文件
import Store from "@/store";

Vue.config.productionTip = false

new Vue({
    render: h => h(App),
  	// 使用
    store: Store,
}).$mount('#app')
```



## 单文件使用

>组件调用``this.$store.dispatch("actions里函数名字", 传递的参数)`` store/index.js里的action接收到然后调用``context.commuit("mutations里函数名字",参数)`` mutations调用state里面的数据``state.data`` 做出具体的修改
>
>``state相当于组件里面的data, getters相当于计算属性`` 

```js
// 基础访问:
{{ $store.state.data }}		{{ $store.getters.data }}
// 简单写法
import {mapState, mapGetters, mapActions, mapMutations} from "vuex";

computed: {
  // 直接使用数组形式相当于这个方法名就是数据名
  ...mapState(["num"]),
  ...mapGetters(["bigNum"])
  // 对象写法
  ...mapState({方法名:"数据名"}),
},
methods: {
  // 使用这种办法的话参数需要再使用的时候传递,也可以使用对象方式
  ...mapActions(["actions里面的方法名字"]),
  ...mapMutations(["mutations里面的方法名字"])
}
{{ 方法名字也就是数据名字 }}

// 使用简写的方法
@click="方法名(参数)"
```

## 多文件使用

``Options1.js``

```js
export default {
    namespace: true,
    actions: {},
    mutations: {},
    state: {}
}
```



``Options2.js``

```js
export default {
    namespace: true,
    actions: {},
    mutations: {},
    state: {}
}
```



``store/index.js``

```js
import Optuions1 from "./Options1"
import Optuions2 from "./Options2"
export default new Vuex.Store({
    modules: {
        Optuions1, Optuions2
    }
})
```

使用

```js
// 在原有的基础上指定namespace
...mapState(Options1,["num"]),

// 非简写形式
this.$store.Options1.state.data
// 必须用 / 隔开namespace和方法名
this.$store.dispatch("namespace/方法名", 参数)
this.$store.commit("namespace/方法名", 参数)
```



# 路由

下载

```npm
npm i vue-router@3
```

## 注意点

>1. 普通页面放到components文件夹下,路由的页面放到pages文件夹下
>
>2. 每个路由页面都有自己专属的``$route`` 但是所有的路由页面共享一个``$router`` 
>3.  "隐藏"的路由组件是被``销毁`` 了,需要的时候会再被``挂载`` 

## 使用

``router/index.js``

```js
import VueRouter from "vue-router";
import About from "@/pages/About.vue";
import Home from "@/pages/Home.vue";
import News from "@/pages/News.vue";
import Message from "@/pages/Message.vue";
import Detatil from "@/pages/Detatil.vue";

export default new VueRouter({
    routes: [
        {
            path: "/about",
            component: About
        }, {
            path: "/home",
            component: Home,
            children: [
                {
                    path: "news",
                    component: News
                }, {
                    path: "message",
                    component: Message,
                    children: [
                        {
                          	// 多级路由可以配置name属性方便后续跳转写法
                            name:"xiangqing",
                            path: "detatil",
                            component: Detatil
                        }
                    ]
                }
            ]
        }
    ]
})
```

``main.js``

```js
import Vue from 'vue'
import App from './App.vue'
// 引入路由
import VueRouter from "vue-router";
// 引入自己写的路由配置
import router from "./router"

Vue.config.productionTip = false
// 使用路由
Vue.use(VueRouter)
new Vue({
    render: h => h(App),
  	// 配置自己写的路由配置
    router
}).$mount('#app')

```

``xxx.vue``

```vue
// 相当于a标签, to就是要跳转的页面,active-class跳转到的页面会有active这个属性,用于高亮显示
<router-link active-class="active" to="/home">Home</router-link>
// 在需要显示的位置写上,跳转的页面就是显示到该位置
<router-view></router-view>

// 需要传递参数的写法,建议这样写不要写成?拼接参数, 一级路由直接写 to=""更简单
<router-link :to="{
  // path:'/home/message/detatil',
  // 使用name更简单
  name:'xiangqing',
  query:{
      id:m.id,
      title:m.msgData
  }
}">
  {{ m.msgData }}
</router-link>

// 在路由跳转的页面使用``this.$route.query`` 获取参数
{{ this.$route.query.id }} -- {{ this.$route.query.title }}
```



## 传递query参数

>传递: ``query:{ id:m.id,  title:m.msgData}`` 
>
>接收: ``{{this.#route.query.data}}`` 



## 传递params参数

一. 在router/index.js里面声明

```js
{
  name: "xiangqing",
  // 声明
  path: "detatil/:x/:y",
  component: Detatil
}
```

二. 传递参数

>使用params传递参数必须使用``name``,不能使用``path`` 
>
>注意在路由配置里面的占位符名字要和传递的参数key相同
>
>``path: "detatil/:x/:y"   x:'qwe'+m.id, y:'asd'+m.msgData`` 

```js
<router-link :to="{
  name:'xiangqing',
  params:{
      x:'qwe'+m.id,
      y:'asd'+m.msgData
  }
}">
  {{ m.msgData }}
</router-link>
```

三. 接收参数

```js
{{ this.$route.params.x }} -- {{ this.$route.params.y }}
```



## 接收参数简写

``router/index.js``

```js
// params的简写形式
{
  name: "xiangqing",
  path: "detatil/:x/:y",
  component: Detatil,
  // 把传递的params的数据以props的方式传递
  props: true
}
// 接收
props: ["x", "y"]
{{ x }}--{{ y }}

// query的简写形式
{
  name: "xiangqing",
  path: "detatil/:x/:y",
  component: Detatil,
  // 参数就是 this.$route, 使用结构赋值的方式写为{query}更简单
  props({query}) {
      return {
          x: query.x,
          y: query.y
      }
  }
}
// 接收不变
props: ["x", "y"]
{{ x }}--{{ y }}
```



## replace模式

>不写默认为push,push的可以被浏览器记录给记录下来,可以适用浏览器的左右小箭头
>
>replace是替换模式,点击后会替换掉上一个记录,也就是无法使用左右的小箭头了

```js
<router-link replace class="list-group-item" active-class="active" to="/about"></router-link>
```



## 编程式路由导航

```js
pushShow(m) {
  // 对象里面的写法和 to 一样
  this.$router.push({
      name: 'xiangqing',
      query: {
          x: m.id,
          y: m.msgData
      },
  })
},
replaceShow(m) {
  this.$router.push({
      name: 'xiangqing',
      query: {
          x: m.id,
          y: m.msgData
      },
  })
},
// 后退
back() {
  this.$router.back()
},
// 前进
forward() {
  this.$router.forward()
},
go() {
  // 正数前进n步,负数后退n步
  this.$router.go(3)
}
```



## 缓存路由组件

>使用``keep-alive`` 标签包裹即可,``include`` :设置只有哪些组件缓存,value为组件名称.不设置include则在这个位置显示的组件都将被缓存,如果缓存多个:``:include="["A","B"]"`` 

```vue
<keep-alive include="News">
  <router-view/>
</keep-alive>
```



## 路由周期钩子

```js
activated() {
  console.log("这个路由组件展示的时候调用")
},
deactivated() {
  console.log("这个路由组件被切换走的时候调用")
}
```



## 全局前置/后置路由守卫

>``beforeEach``:路由组件跳转前调用
>
>``afterEach``:路由组件跳转后调用

```js
{
  path: "news",
  component: News,
  // 添加meta数据方便前置和后置函数使用
  meta: {title: "新闻", isAu: true},
}
// to: 去哪 form: 哪来 next是否跳转, 前置用来校验权限,后置用来修改title
router.beforeEach((to, from, next) => {
    console.log("前置:", to, from, next)
    if (to.meta.isAu) {
        alert("需要校验权限")
    } else {
        next()
    }
})
router.afterEach((to, from) => {
    console.log("前置:", to, from)
    document.title = to.meta.title || "loveyu"
})
```

## 独享路由守卫

>只有前置没有后置

```js
{
  path: "/about",
  component: About,
  meta: {title: "关于"},
  beforeEnter: (to, from, next) => {
    // 写法和之前的一样
      console.log("独享守卫", to, from, next)
      next()
  }
}
```



## 组件内路由守卫

>``beforeRouteEnter``:根据路由规则进入组件前调用
>
>``beforeRouteLeave``:根据路由规则离开组件前调用
>
>直接引用这个组件是不会调用这两个函数的

```js
export default {
    name: "Detatil",
    props: ["x", "y"],
    beforeRouteEnter(to, from, next) {
        console.log("beforeRouteEnter", to, from, next)
        next()
    },
    beforeRouteLeave(to, from, next) {
        console.log("beforeRouteLeave", to, from, next)
        next()
    }
}
```



## 路由模式

>默认为``hash``模式也就是路径上带``#``号的,``history``不带``#``号,但是``#``需要服务器配置使用
>
>hash比history兼容性好

```js
const router = new VueRouter({
    mode: "history",
```



# elementUI

>按需引入的配置文件,官网的是老版本不对

``babel.config.js``

```js
module.exports = {
    presets: [
        '@vue/cli-plugin-babel/preset',
        ["@babel/preset-env", {"modules": false}],
    ],
    plugins: [
        [
            "component",
            {
                "libraryName": "element-ui",
                "styleLibraryName": "theme-chalk"
            }
        ]
    ]
}

```





# 其他

>查看版本

```nom
npm view less-load versions
```



>覆盖原对象内容
>
>info有的dataObj也有呢就以dataObj的为主覆盖掉info的,dataObj没有info还是原值

```js
this.info = {...this.info, ...dataObj}
```



>vue-router必须是npm下载才会有代码提示

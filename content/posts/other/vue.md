---
title: "Vue"
date: 2020-11-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- 其他
tags: 
- vue
---



# 基础

>```js
>const x = new Vue({
>el: '#root',    //指定容器
>data: {     //设置数据
>   name: 'world',
>   age: 18,
>   url: "www.baidu.com",
>   user: {
>       username: "zhangsan",
>       password: "qwe"
>   }
>}
>})
>```

## 插值语法

>{{name}}

## 指令语法

>```
><a v-bind:href="url" v-bind:x="name">百度</a>
>```
>
>`v-bind:`可以简写为     `:`

## el和data写法

>```js
>const x = new Vue({
>el: '#root',    //指定容器
>data: {     //设置数据
>   name: 'world',
>   age: 18,
>   url: "www.baidu.com",
>   user: {
>       username: "zhangsan",
>       password: "qwe"
>   }
>}
>})
>```
>
>等同于：
>
>```js
>const vm = new Vue({
>  // data: function () {
>  //     return {
>  //         name: "zhangsan233"
>  //     }
>  // }
>data() {
>    return {
>        name: "zhangsan233"
>    }
>}
>})
>vm.$mount("#root")
>```
>
>data(){return}是data:function(){return}的简写

## 单双向绑定

>单向：`v-bind`
>
>双向：`v-model`
>
>单向是vm向m传递；双向是vm向m传递，m可以向vm传递
>
>`v-model:value="name" `简写：`v-model="name"`

## #数据代理

>```js
>var num = 19
>let user = {
>username: "qwe"
>}
>Object.defineProperty(user,"age",{
>value: 12,
>enumerable: true,   //是否可以枚举（遍历）默认false
>writable: true, //是被可以被改写，默认false
>configurable: true,  //是否可以被删除，默认false
>//读取age属性就会调用这个函数
>get:function () {
>   return num
>},
>//修改age调用,value就是修改的值, set(){} * set:function(){}
>set(value) {
>   console.log(value)
>   num = value
>}
>})
>```

## 事件绑定

>```js
><button v-on:click="o1">a</button>
><button @click="o2(22)">b</button>
>
>new Vue({
>el: '#root',
>methods:{
>o1(){
> alert("a")
>},
>o2(num) {
> alert(num)
>}
>}
>})
>```
>
>`v-on:click` 简写为 `@click`
>
>var a = true
>
>click后边可以下一写简单的语句：@click=" a ? b : c "

## 事件修饰符

>```js
><!--    事件修饰符可以连着用
>@click.stop.once
>-->
><!--  prevent：阻止默认事件，超链接跳转  -->
><a href="www.baidu.com" @click.prevent="c1">跳转</a>
><!--  stop：阻止冒泡  -->
><div class="d1" @click="c1">div1
>   <div class="d2" @click.stop="c1">div2</div>
></div>
><!-- once：只触发一次   -->
><button type="button" @click.once="c1">button</button>
><!-- capture：使用事件的捕获，正常情况是点d2，触发d1，冒泡d2，d2相应事件，d1相应事件
>         使用事件捕获就是d1会先触发，在冒泡到d2触发-->
><div class="d1" @click.capture="c1">div1
>   <div class="d2" @click="c1">div2</div>
></div>
><!--  self：只有event.target是当前操作的元素才会触发事件,也可以阻止冒泡发生  -->
><div class="d1" @click.self="c1">div1
>   <div class="d2" @click="c1">div2</div>
></div>
><!--    passive：异步，先相应触发结果同时对触发事件做处理-->
>```

## 键位事件

>```
><!--@keyup：键盘事件
>keyup：按键需要按下松开才可以
>keydown：按键只需按下即可
>-->
><!--
>vue提供的事件有：
>enter，delete，esc，上下左右，空格，换行
>vue未提供的用按键的名字绑定，如果这个按键是两个单词，两个首字母小写中间-隔开
>特殊的按键：
>ctrl，shift，alt，meta这几个是系统按键，想要这几个键位触发事件需要在按下这个键的同时按下其他键然后释放其他键才可以
>keyup需要这样，keydown不需要
>-->
><!--可以使用组合键
>@keyup.q.w
>-->
>```
>
>```js
>methods:{
>en(){
>  // console.log(e.keyCode)     //获取键盘按键对应数字
>  // console.log(e.key)    //获取键盘按键对应键位名字
>   alert("。。。")
>}
>}
>```

## 计算属性

>```html
><input v-model:value="a">
><input v-model:value="b">
><input v-model:value="c">
>```
>
>```js
>const vm = new Vue({
>el: '#root',
>data: {
>   a: "a",
>   b: "b"
>},
>computed: {
>   c:{
>       get(){
>           return this.a+"-"+this.b
>       },
>       set(v){
>           const tr = v.split("-")
>           this.a = tr[0]
>           this.b = tr[1]
>       }
>   }
>}
>})
>```
>
>计算属性在读取第一次后会被缓存，下次使用的时候就不会调用而是拿缓存
>
>当计算属性被修改就会调用set方法
>
>当计算属性被读取的时候调用get方法，当依赖的数据被修改也会调用
>
>*当计算属性只有get方法的时候可以简写为*：
>
>```js
>c() {
>return this.a + "-" + this.b
>}
>```
>
>这个fun函数就相当于get了

## 监视属性

>```js
>//也可以监视计算属性
>const vm = new Vue({
>   el: '#root',
>   data: {
>       name: "a"
>   },
>   // watch:{
>   //     name: {
>   //         immediate: true,
>   //         handler(newName,oldName){
>   //             console.log("newName:",newName,",oldName:",oldName)
>   //         }
>   //     }
>   // }
>})
>
>//第二种写法：
>vm.$watch('name',{
>   immediate: true,
>   handler(newName,oldName){
>       console.log("newName:",newName,",oldName:",oldName)
>   }
>})
>```
>
>`immediate`：程序启动就调用一次handler
>
>`handler`：当被监视的属性发成改变的时候会被调用，参数为新的属性值和旧的属性值
>
>```js
>//监听多级结构中的某一个属性
>'a.b':{
>handler(newA,oldB){
>   console.log("newA:",newA,",oldB:",oldB)
>}
>},
>//监听多级结构的全部属性
>a: {
>deep: true,
>handler(newA,oldA){
>   console.log("newA/B:",newA,",oldA/B:",oldA)
>}
>}
>```
>
>`deep`：深度监视；默认是false，fasle是不能监听的
>
>简写：简写相当于只有一个handler函数，也就不能开启深度监视了
>
>```js
>name(n,o){
>console.log(n,o)
>}
>
>vm.$watch("name",function (n,o) {
>	  console.log(n,o)
>})
>```

## 样式绑定

>```js
><!--        适用于：样式类名不确定需要动态绑定-->
><div class="a" :class="b">b</div>
><!--        适用于：样式个数不确定，名字也不确定-->
><div class="a" :class="cd">cd</div>
><!--        适用于：绑定个数确定，名字也确定，单要动态决定用那个-->
><div class="a" :class="bcd">bcd</div>
><!--        控制style样式用key-value格式-->
><div class="a" :style="d">d</div>
>```
>
>```js
>new Vue({
>el: '#root',
>data: {
>   b: "b",
>   cd: ["c","d"],
>   bcd:{
>       b: true,
>       c: true,
>       d: true
>   },
>   d: {
>       background: "red"
>   }
>}
>})
>```

## 条件渲染

>```js
><!--    if和elseIf和else这三个不能分开-->
><h1 v-if="a *= 1">1</h1>
><h1 v-else-if="a *= 2">2</h1>
><h1 v-else>3</h1>
><!--    show的条件为false是只是调用display：none-->
><h1 v-show="a *= 4">v-show</h1>
><!--    template里面的是一组，加载时不会加载template这个标签-->
><template v-if="a*=5">
>   <h1>a</h1>
>   <h1>b</h1>
>   <h1>c</h1>
>   <h1>d</h1>
></template>
>```

## 遍历

>```js
><h1>遍历数组</h1>
><ul>
>   <li v-for="(p,index) in persons" :key="p.id">
>       {{index}}---{{p.age}}---{{p.username}}
>   </li>
></ul>
><h1>遍历kv</h1>
><ul>
>   <li v-for="(v,k) in user" :key="k">
>       {{k}}---{{v}}
>   </li>
></ul>
><h1>循环</h1>
><ul>
>   <li v-for="(number,index) in 10">
>       {{number}}---{{index}}
>   </li>
></ul>
>```
>
>```js
>new Vue({
>el: '#root',
>data: {
>   persons:[
>       {id:"1",username:"张1",age:11},
>       {id:"2",username:"张2",age:12},
>       {id:"3",username:"张3",age:13},
>       {id:"4",username:"张4",age:14},
>   ],
>   user:{
>       id: 19,
>       username: "qwe",
>       password: "asd"
>   }
>}
>})
>```

## 搜索

>```js
><div id="root">
><input type="text" v-model="sear">
><button @click="sortType = 1">升序</button>
><button @click="sortType = 2">降序</button>
><button @click="sortType = 0">原序</button>
><ul>
>   <li v-for="u in searUsers" :key="u.id">
>       {{u.id}}----{{u.name}}---{{u.age}}
>   </li>
></ul>
></div>
>```
>
>```js
>//watch实现
>//     new Vue({
>//         el: '#root',
>//         data: {
>//             users:[
>//                 {id:1,"name":"abc","age":10},
>//                 {id:2,"name":"bcd","age":11},
>//                 {id:3,"name":"cde","age":12},
>//                 {id:4,"name":"def","age":13},
>//             ],
>//             sear: "",
>//             searUsers: "",
>//         },
>//         watch: {
>//             sear:{
>//                 immediate: true,
>//                 handler(n,o){
>//                     this.searUsers = this.users.filter((u)=>{
>//                         return u.name.indexOf(n) !* -1
>//                     })
>//                 }
>//             }
>//         }
>//     })
>
>
>//监视实现
>new Vue({
>el: '#root',
>data: {
>   users: [
>       {id: 1, "name": "abc", "age": 10},
>       {id: 2, "name": "bcd", "age": 4},
>       {id: 3, "name": "cde", "age": 1},
>       {id: 4, "name": "def", "age": 13},
>   ],
>   sear: "",
>   sortType: 0
>},
>computed: {
>   searUsers() {
>       const arr = this.users.filter((u) => {
>           return u.name.indexOf(this.sear) !* -1
>       })
>       if (this.sortType) {
>           arr.sort((u1,u2)=>{
>               return this.sortType *= 1 ? u1.age - u2.age : u2.age - u1.age
>           })
>       }
>       return arr
>   }
>}
>})
>```

## 表单

>```html
><div id="root">
><!--
>number:转化为整形
>trim：去掉前后空格
>lazy：失去焦点再收集
>-->
><form @click.prevent="">
>   账号：<input type="number" v-model.number="info.account"><br/><br/>
>   密码：<input type="password" v-model.trim="info.password"><br/><br/>
>   性别：男：<input type="radio" value="nan" name="s" v-model="info.sex">
>        女：<input type="radio" value="nv" name="s" v-model="info.sex"><br/><br/>
>   爱好：a<input type="checkbox" value="a" v-model="info.aihao">
>   b<input type="checkbox" value="b"  v-model="info.aihao">
>   c<input type="checkbox" value="c"  v-model="info.aihao"><br/><br/>
>   地址：<select v-model="info.addr">
>   <option value="bejing">北京</option>
>   <option value="shanghai">上海</option>
>   <option value="guangzhou">广州</option>
>       </select><br/><br/>
>   other：<textarea v-model.lazy="info.other"></textarea><br/><br/>
>   协议:<input type="checkbox" v-model="info.xieyi"><br/><br/>
>   <button>提交</button>
></form>
></div>
>```
>
>```js
>new Vue({
>el: '#root',
>data: {
>   info: {
>       account: "",
>       password: "",
>       sex: "",
>       aihao: [],
>       addr: "",
>       other: "",
>       xieyi: ""
>   }
>}
>})
>```

## 管道过滤

>```html
><div id="root">
><h1>{{age | add}}</h1>
><h1>{{age | add | add2}}</h1>
><h1>{{age | add | add2 | add3}}</h1>
></div>
>```
>
>```js
>//全局
>Vue.filter("add3",function (value){
>return value * 10
>})
>new Vue({
>el: '#root',
>data: {
>   age: 10
>},
>//局部
>filters: {
>   add(value){
>       return value+10
>   },
>   add2(value){
>       return value + 100
>   }
>}
>})
>```

## 内置指令

>```html
><div id="root">
><!--
>v-text：整体替换调标签内的内容
>v-html：可以编译内容中的语法
>v-pre：vue加载是会调过这个标签
>v-one：编译一次后就变为静态的
>-->
><h1>msg:{{msg}}</h1>
><h1 v-text="msg"></h1>
><h1 v-html="msg"></h1>
><h1 v-pre v-html="msg">aaa</h1>
><h1 v-one v-html="msg"></h1>
></div>
>```
>
>```js
>new Vue({
>el: '#root',
>data: {
>   msg: "<h1>abc</h1>"
>}
>})
>```

## 自定义指定

>```html
><div id="root">
><h1>{{msg}}</h1>
><h1 v-add="msg"></h1>
></div>
>```
>
>```js
>new Vue({
>el: '#root',
>data: {
>   msg: 10
>},
>directives: {
>   //简写
>   add(element,binding){
>       element.innerHTML = binding.value + 10
>   },
>   //完全体
>   del:{
>       bind(element,binding){
>           console.log("指令与元素绑定时调用")
>       },
>       inserted(element,binding){
>           console.log("指令所在元素插入页面时调用")
>       },
>       update(element,binding){
>           console.log("指令所在模版重新解析时调用")
>       }
>   }
>}
>})
>```



# 组件

## 非单文件组件

>在一个文件里面写多个组件
>
>```js
>const school = Vue.extend({
>template:  `<div><h1>{{name}}</h1><h1>{{address}}</h1></div>`,
>data(){
>   return{
>       name: "哈哈",
>       address: "北京"
>   }
>}
>})
>const user = Vue.extend({
>template:  `<div><h1>{{name}}</h1><h1>{{age}}</h1></div>`,
>data(){
>   return{
>       name: "张三",
>       age: 10
>   }
>}
>})
>const gUser = Vue.extend({
>template:  `<div><h1>{{name}}</h1><h1>{{age}}</h1></div>`,
>data(){
>   return{
>       name: "张三",
>       age: 10
>   }
>}
>})
>Vue.component("guser",gUser)
>new Vue({
>el: '#root',
>components: {
>   school,
>   user
>}
>})
>new Vue({
>   el: '#root2'
>})
>```
>
>`Vue.component`:全局注册
>
>`components: {school,user}`:局部注册
>
>```html
><div id="root">
><school></school>
><hr/>
><user></user>
><user></user>
><hr/>
><guser></guser>
></div>
><hr/>
><div id="root2">
><guser></guser>
></div>
>```
>
>使用：直接当作标签使用



# 插件：

定义plugins.js

```js
export default {
    install(vue) {
        vue.mixin({
            data(){
                return{
                    pluginName: "plugins.js"
                }
            }
        })
        //可以添加全局过滤器，自定义指定等全局的设置
    }
}
```

main.js使用

```js
//引入插件

import plugins from "./plugins/plugins"

//可以加参数，然后再接收即可 user(plugins,1,2,"aaa","bb")
Vue.use(plugins)
```

# 自定义事件

>父组件和子组件通信使用

## 用法：

### 父组件绑定事件

#### 两种方法

1. ```vue
   <HelloWorld @getNameData="回调方法" />
   
   methods: {
     回调方法(value) {
     	console.log(value);
     },
   }
   ```

   1. 使用这个办法只需在mehods中添加一个回调方法即可，多个参数逗号隔开

      

2. ```vue
   <HelloWorld ref="helloWorld" />
   
   mounted() {
   	this.$refs.helloWorld.$on("getAgeData", 回调方法);
   },
   ```

   1. 使用2方法需要在钩子函数`mouted`中绑定上这个事件，多个参数逗号隔开

      

### 子组件调用事件

1. ```vue
   this.$emit("事件名称", 数据);
   ```

### 子组件解除绑定事件

1. ```
   this.$off("getNameData");
   多个参数用[“a”,"b"]数组方式
   不传递参数就是全部解除绑定
   ```

## 案例：

### 父组件

>```vue
><template>
><div id="app">
><!-- 第一种办法 -->
><HelloWorld @getNameData="getNameData" />
><!-- 第二种办法 -->
><HelloWorld ref="helloWorld" />
><!-- 如果要给组件绑定本来就有的事件需要加上 native ，组件上编定的事件默认都认为是自定义的-->
><HelloWorld @click.native="a" />
></div>
></template>
>
><script>
>import HelloWorld from "./components/HelloWorld.vue";
>
>export default {
>name: "App",
>components: {
>HelloWorld,
>},
>methods: {
>getNameData(value) {
> console.log(value);
>},
>getAgeData(value) {
> console.log(value);
>},
>},
>mounted() {
>this.$refs.helloWorld.$on("getAgeData", this.getAgeData);
>},
>};
></script>
>
><style>
></style>
>
>```
>
>

### 子组件：

>```vue
><template>
><div class="hello">
><h1>{{ name }}</h1>
><h1>{{ age }}</h1>
><button @click="sendNameData">点击给app发送name数据</button>
><button @click="sendAgeData">点击给app发送age数据</button>
>
><button @click="gg">解绑自定义事件</button>
></div>
></template>
>
><script>
>export default {
>name: "HelloWorld",
>data() {
>return {
> name: "张三",
> age: 10,
>};
>},
>methods: {
>sendNameData() {
> this.$emit("getNameData", this.name);
>},
>sendAgeData() {
> this.$emit("getAgeData", this.age);
>},
>gg() {
> //解绑多个就用off(['a','b'])，解绑全部就用off()不加参数就是接绑全部的自定义事件
> this.$off("getNameData");
>},
>},
>};
></script>
>
><!-- Add "scoped" attribute to limit CSS to this component only -->
><style>
></style>
>
>```
>
>



# 全局事件总线

>全局事件总线就是在自定义事件的基础上进行修改，让vm当作一个 工具人 供其他组件相互传递数据

用法和自定义事件差不多只需在app.vue中添加`bus`总线即可

在app.vue中添加bus	`Vue.prototype.$bus = this`

```vue
export default {
  name: "App",
  components: {...},
  beforeCreate() {
    Vue.prototype.$bus = this; // 添加全局总线
  },
};
```

使用时把自定义事件中的换成bus即可，如：

```
this.$bus.$on()
this.$bus.$emit()
this.$bus.$off()
```



# 订阅发布

需要第三方库：npm i subpub-js 下载

使用：

接收消息的为订阅；发送消息为发布

## 发布消息：

> import pubsub from "pubsub-js";
>
> 
>
> pubsub.publish("管道名字", 数据);

## 订阅消息：

```vue
import pubsub from "pubsub-js";



this.subId = pubsub.subscribe("管道名字", (msgName, data) => {

​      subid用来后续的关闭，msgName就是管道的名字，data为传送数据

});



beforeDestroy() {

//取消订阅

​    pubsub.unsubscribe(this.subId);

},
```



# 动画

导入：animate.css

```vue
<template>
  <div class="hello">
<!--
transition：只能包裹一个标签
transition-group：可以包裹多个标签但是，里面的标签的key必须唯一
-->
    <transition-group
        name="animate__animated animate__bounce"
        appear
        enter-active-class="animate__swing"
        leave-active-class="animate__backOutUp"
    >
      <h1 v-show="show" key="1">Hello World !</h1>
      <h1 v-show="!show" key="2">Hello World !</h1>
    </transition-group>
    <button @click="show=!show">显示/隐藏</button>
  </div>
</template>
<!--
进来的起点 * 离开的终点
进来的终点 * 离开的起点
v=transition标签的name，没写name就是v，写了就是name的value-xxx-xxx
元素进入的样式：
  v-enter:进来的起点
  v-enter-active:进来的过程
  v-enter-to:进来的终点
元素离开的样式：
  v-leave:离开的起点
  v-leave-active:离开的过程
  v-leave-to:离开的终点

-->
```

# 插槽

```vue
app:
<template>
  <div id="app">
    <HelloWorld>
      <h1>默认插槽</h1>
      <template v-slot:name1>
        <h1>具名插槽</h1>
      </template>
      <template scope="data">
        <h1>作用域插槽，收到data：{{data.sendMsg}}</h1>
      </template>
    </HelloWorld>
  </div>
</template>
helloWorld:
<template>
  <div class="hello">
    <slot>当这个插槽没有使用就会显示这些默认值</slot>
    <slot name="name1">有名字的插槽</slot>
    <slot :sendMsg="msg">作用域插槽，可以向使用组件传递data</slot>
  </div>
</template>
```

# Vuex

共享数据

引入：cnpm i vuex

创建store文件夹创建index.js写入：

```js
import vuex from "vuex"
import Vue from "vue";

Vue.use(vuex)

//响应组件动作
const actions = {
    add(context, value) {
        context.commit("ADD", value)
    },
    jian(c, v) {
        c.commit("JIAN", v)
    },
    demo(c,v) {
        //这里可以dis在调用其他的actions方法
        c.dispatch("add",v)
    }
}
//加工数据，这里的数据就是state里面的
const mutations = {
    ADD(state, value) {
        state.num += value
    },
    JIAN(s, v) {
        s.num -= v
    }
}
const getters = {
    bigNum(state) {
        return state.num * 10
    }
}
//保存数据
const state = {
    num: 0
}

//k v 一样就不用写=了
export default new vuex.Store({
    actions,
    mutations,
    state,
    getters
})
```

app组件中引入store并使用

```vue
import store from "./store"

export default {
  name: 'App',
  store,
  components: {
    HelloWorld,Demo
  }
}
```

# 路由：

1. 在main.js中添加路由并使用

```js
import vueRouter from "vue-router"

Vue.use(vueRouter)
```

2. 创建router/router.js文件，用来设置路由

```js
import vueRouter from "vue-router"
import About from "@/pages/About";
import Home from "@/pages/Home";
import Message from "@/pages/Message";
import News from "@/pages/News"
import Tt from "@/pages/Tt";

//meta 路由配置中可以用来存储数据
const router = new vueRouter({
    mode: "history",  //默认是hash，hash带#号，使用history不带#号但需要后端适配一下
    routes:[
        {
            name: "about",
            path: "/about",
            component: About,
            meta: {title:"关于"},
            beforeEnter(to,from,next){
                console.log("独享守卫，只有前置没有后置",to,from,next)
                next()
            }
        },
        {
            name: "home",
            path: "/home",
            component: Home,
            meta: {title:"主页"},
            children:[
                {
                    name: "message",
                    path:"message",
                    component: Message,
                    meta: {title:"消息"},
                    children: [
                        {
                            name: "tt",
                            path: "tt",
                            component: Tt,
                            meta: {title:"消息Other",isAuth:true}
                        }
                    ]
                },
                {
                    name: "news",
                    path: "news",
                    component: News,
                    meta: {title:"新闻"},
                    children: [
                        {
                            name: "newtt",
                            path: "newtt/:id/:title",
                            component: Tt,
                            meta: {title:"新闻Other",isAuth:true}

                            //把query和params参数进行封装，在组件中直接用props接收即可

                            // props($router){
                            //     return {
                            //         id: $router.params.id,
                            //         title: $router.params.title
                            //     }
                            // }
                        }
                    ]
                }
            ]
        }
    ]
})
//to:去哪 from：哪来 next：跳不跳转 beforeEach：每次路由跳转前
router.beforeEach((to,from,next)=>{
    console.log(to,from,next)
    if (to.meta.isAuth) {
        if (localStorage.getItem("key") *= "value") {
            next()
        } else {
            alert("no auth")
        }
    } else {
        next()
    }
})
//afterEach 每次路由跳转后
router.afterEach((to, from)=>{
    console.log(to,from)
    document.title = to.meta.title || "hzyy"
})


export default router
```

3. 再App.vue中使用

```js
import router from "./router"
import TopCom from "@/components/TopCom";

export default {
  name: 'App',
  components:{
    TopCom
  },
  router: router
}
```

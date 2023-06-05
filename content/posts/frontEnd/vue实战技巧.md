---
title: "Vue实战技巧"
date: 2023-04-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- other
tags: 
- vue
---

# params为空

>params为空url错误问题,在path最后加上问号即可解决

```js
path: "/search/:keyword?"
```

# params为空字符串

>当为空字符串url一样会有错误
>
>``this.keyword || undefined`` 解决

```js
this.$router.push({
  name: "search",
  params: {
      "keyword": this.keyword || undefined
  	}
	}
)
```

# 重复路由报错问题

>重写push和replace

```js
// 重写push和replace方法,解决重复点击路由报错问题
// call 和 apply的区别就是call传递参数是用逗号隔开,apply是用数组,相同点都可以改变this指向
let originPush = VueRouter.prototype.push;
// resolve:成功调用的方法,reject:失败调用的方法; replace重写方式和push一样
VueRouter.prototype.push = function (obj, resolve, reject) {
    if (resolve && reject) {
        originPush.call(this, obj, resolve, reject)
    } else {
        originPush.call(this, obj, () => {
        }, () => {
        })
    }
}
```



# axios二次封装和请求进度条

``api/requests.js``

```js
import axios from "axios";
import nprogress from "nprogress"
import "nprogress/nprogress.css"

const requests = axios.create({
    baseURL: "http://localhost:9888",
    timeout: 5000
})
requests.interceptors.request.use((config) => {
    nprogress.start()
    return config
})
requests.interceptors.response.use((res) => {
    nprogress.done()
    return {status: res.status, data: res.data}
}, (error) => {
    nprogress.done()
    console.log(error)
    return {status: error.response.status, err: error.response.data}
})
export default requests
```

``api/index.js``

```js
import requests from "@/api/requests";

export const test = () => {
    return requests({
        url: "/",
        method: "get"
    })
}
```

使用

```js
// 在组件中使用
import {test} from "@/api";
test().then(
  // res的信息就是上边封装后返回的信息
  res => {
      if (res["status"] !== 200) {
          console.log(res["err"])
      } else {
          console.log(res["data"])
      }
  }
)

// 在vuex中使用,必须使用async和await
actions: {
  async getBannerList({commit}) {
      let result = await getBaseCategoryList()
      if (result["status"] === 200) {
          commit("BANNERLIST", result["data"])
      }
  },
},
```

# 鼠标悬浮事件

```js
@mouseenter 鼠标悬浮
@mouseleave 鼠标离开
```



# 防抖和节流

>防抖: 前面的所有触发都取消,最后一次执行结束一定时间后再触发,也就是连续的触发只会执行一次.例如:``搜索提示``,不能输入框每次变化就会发送请求,使用防抖让输入框不发生变化后的一定时间后再触发函数
>
>节流: 在规定时间内不会重复触发,只有大于设定的时间间隔才会触发,把频繁触发变为少量触发,例如:``二三级联动展示,轮播图``,鼠标的快速悬浮通过和快速点击都会频繁触发回调,使用节流可以在指定时间内只触发一次,例如轮播图设置节流事件为1秒,那么在这1秒内无论用户点击多少次只会触发一次,下一次触发要在下一个1s

实现

``使用lodash.js,lodash.js在vue项目中默认就有不需要再下载``

```js
// 按需引入
import throttle from "lodash/throttle"
import debounce from "lodash/debounce"

// 防抖
let res1 = debounce(function () {
    console.log("处理逻辑")
}, 1000)
// 执行
res1()

// 节流
let res2 = throttle(function () {
    console.log("处理逻辑")
}, 50)
res2()

// 在vue中使用
methods: {
  	// 使用key:value的方式,index为mouse方法的参数
    mouse: throttle(function (index) {
        this.isBack = index
    }, 50),
}
```



# 事件委派

>给一二三级联动展示内的a标签添加路由跳转
>
>1. 使用router-link,会产生卡顿,因为当标签过多router-link创建的实例也多
>
>2. 给a标签添加@click,同理a标签过多添加的@click也会过多,性能不好
>
>3. 使用时间委派,只需要添加一个@click解决问题
>
>   1. 在一二三级的共同父标签上添加@click
>
>      1. ```html
>         <div @click="toSearch">
>           一二三级标签
>         </div>
>         ```
>
>      2. 
>
>   2. 在一二三级的a标签添加自定义属性,``自定义属性一定要加上data-前缀不然dataset无法解析出来``
>
>      1. ```html
>         <a
>           :data-categoryName="c1.categoryName"
>           :data-categoryId="c1.categoryId"
>         >
>         {{ c1.categoryName }}
>         </a>
>         ```
>
>   3. 通过event解析出来

```js
toSearch(event) {
  // 获取点击的标签
  let el = event.target
  // 解构出来自定义标签
  let {categoryname, categoryid} = el.dataset
  // categoryname存在说明是a标签
  if (categoryname) {
      let location = {
          name: "search",
          query: {
              "categoryname": categoryname,
              "categoryid": categoryid
          }
      }
      // 路由跳转
      this.$router.push(location)
  }
}
```



# swiper实现轮播图

```js
npm i swiper@5
npm install sass-loader@8.0.2 sass@1.26.5  --save-dev
// 如果包deep错误:
原因是在scss里面   /deep/  想要使用样式穿透，让子组件匹配上这个样式
在 less 中 将 /deep/ 替换成 ::v-deep 即可
```

>在``main.js``引入swiper的css样式,因为轮播图在多个组件都有用,如果只在一个组件用则在呢个组件中引入即可

``main.js``

```js
// 是css目录下的css样式不是直接引用swiper/下的css
import "swiper/css/swiper.min.css"
```

``.vue``

```vue
<div class="swiper-container" id="mySwiper">
  <div class="swiper-wrapper">
      <div class="swiper-slide" v-for="image in images" :key="image.Id">
          <img :src="image.Src"/>
      </div>
  </div>
  <!-- 如果需要分页器 -->
  <div class="swiper-pagination"></div>
  <!-- 如果需要导航按钮 -->
  <div class="swiper-button-prev"></div>
  <div class="swiper-button-next"></div>
</div>
<script>
import {mapActions, mapState} from "vuex";
import Swiper from "swiper";

export default {
    name: "index",
    computed: {
        ...mapState("home", ["images"])
    },
    methods: {
        ...mapActions("home", ["getImages"])
    },
    mounted() {
        // 加载轮播图的图片
        this.getImages()
    },
    watch: {
        images: {
            handler() {
                this.$nextTick(() => {
                    new Swiper('.swiper-container', {
                        //direction: 'vertical', // 垂直切换选项
                        loop: true, // 循环模式选项
                        // 如果需要分页器
                        pagination: {
                            el: '.swiper-pagination',
                            // 点击小球也可以跳转
                            clickable: true
                        },
                        // 如果需要前进后退按钮
                        navigation: {
                            nextEl: '.swiper-button-next',
                            prevEl: '.swiper-button-prev',
                        },
                    })
                })
            }
        }
    }
}
</script>
```


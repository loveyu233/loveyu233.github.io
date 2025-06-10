---
title: "js常用操作"
date: 2020-12-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- js
- frontEnd
tags: 
- js
- 前端
---



```js
var let const

声明变量

if else

条件语句

switch case

条件语句

alert confirm prompt

弹框

function

函数

for

循环

while

循环

break

终止循环

continue

跳过循环

onLoad onUnload onFocus onBlur onChange onSubmit onClick onMouseOver onMouseOut

事件太多了随便列举几个

try catch

异常捕获

throw

抛异常

String Number Boolean Null Object Array Symbol BigInt undefined

基本类型

Promise then catch reject resolve all any race allSettled

DOM

window location screen history Navigator

BOM

Node.nodeName   //返回节点名称

Node.nodeType   //返回节点类型的常数值

Node.nodeValue  //返回Text或Comment节点的文本值

Node.textContent  //返回当前节点和它的所有后代节点的文本内容，可读写

Node.baseURI    //返回当前网页的绝对路径

Node.ownerDocument  //返回当前节点所在的顶层文档对象，即document

Node.nextSibling  //返回紧跟在当前节点后面的第一个兄弟节点

Node.previousSibling  //返回当前节点前面的、距离最近的一个兄弟节点

Node.parentNode   //返回当前节点的父节点

Node.parentElement  //返回当前节点的父Element节点

Node.childNodes   //返回当前节点的所有子节点

Node.firstChild  //返回当前节点的第一个子节点

Node.lastChild   //返回当前节点的最后一个子节点

Node.isconnecoted   //返回一个布尔值，表示当前节点是否在文档之中

Node.appendChild(node)   //向节点添加最后一个子节点

Node.hasChildNodes()   //返回布尔值，表示当前节点是否有子节点

Node.cloneNode(true);  // 默认为false(克隆节点), true(克隆节点及其属性，以及后代)

Node.insertBefore(newNode,oldNode)  // 在指定子节点之前插入新的子节点

Node.removeChild(node)   //删除节点，在要删除节点的父节点上操作

Node.replaceChild(newChild,oldChild)  //替换节点

Node.contains(node)  //返回一个布尔值，表示参数节点是否为当前节点的后代节点。

Node.compareDocumentPosition(node)   //返回一个7个比特位的二进制值，表示参数节点和当前节点的关系

Node.isEqualNode(node)  //返回布尔值，用于检查两个节点是否相等。所谓相等的节点，指的是两个节点的类型相同、属性相同、子节点相同。

Node.isSameNode(node)  //返回一个布尔值，表示两个节点是否为同一个节点。

Node.normalize()   //用于清理当前节点内部的所有Text节点。它会去除空的文本节点，并且将毗邻的文本节点合并成一个。

Node.getRootNode()  //返回当前节点所在文档的根节点document，与ownerDocument属性的作用相同。

Node.length   //返回 NodeList 实例包含的节点数量

Node.forEach(fn，this)   //用于遍历 NodeList 的所有成员

Node.item(index) // 接受一个整数值作为参数，表示成员的位置，返回该位置上的成员

Node.keys()  //返回键名的遍历器

Node.values()   //返回键值的遍历器

Node.entries()  //返回的遍历器同时包含键名和键值的信息

//parentNode接口

Node.children  //返回指定节点的所有Element子节点

Node.firstElementChild  //返回当前节点的第一个Element子节点

Node.lastElementChild   //返回当前节点的最后一个Element子节点

Node.childElementCount  //返回当前节点所有Element子节点的数目

Node.append()  //为当前节点追加一个或多个子节点，位置是最后一个元素子节点的后面。

Node.prepend()   //为当前节点追加一个或多个子节点，位置是第一个元素子节点的前面。

//ChildNode接口

Node.remove()  //用于从父节点移除当前节点

Node.before()  //用于在当前节点的前面，插入一个或多个同级节点。两者拥有相同的父节点。

Node.after()   //用于在当前节点的后面，插入一个或多个同级节点，两者拥有相同的父节点。

Node.replaceWith()  //使用参数节点，替换当前节点



document

//快捷方式属性

document.defaultView //返回document对象所属的window对象

document.doctype   //返回文档类型节点

document.documentElement  //返回当前文档的根节点

document.body   //返回当前文档的<body>节点

document.head   //返回当前文档的<head>节点

document.activeElement  //返回当前文档中获得焦点的那个元素

document.fullscreenElement  //返回当前以全屏状态展示的 DOM 元素



//节点集合属性

document.links  //返回当前文档的所有a元素

document.forms  //返回页面中所有表单元素

document.images  //返回页面中所有图片元素

document.embeds  //返回网页中所有嵌入对象

document.scripts  //返回当前文档的所有脚本

document.styleSheets  //返回当前网页的所有样式表



//文档静态信息属性

document.documentURI  //表示当前文档的网址

document.URL  //返回当前文档的网址

document.domain  //返回当前文档的域名

document.lastModified  //返回当前文档最后修改的时间

document.location  //返回location对象，提供当前文档的URL信息

document.referrer  //返回当前文档的访问来源

document.title    //返回当前文档的标题

document.compatMode  //返回浏览器处理文档的模式

document.characterSet  //返回渲染当前文档的字符集，比如UTF-8、ISO-8859-1



//文档状态属性

document.readyState  //返回当前文档的状态

document.hidden  //返回一个布尔值，表示当前页面是否可见

document.visibilityState  //返回文档的可见状态



//其他属性

document.cookie   //用来操作Cookie

document.designMode  //控制当前文档是否可编辑，可读写

document.currentScript  //返回当前脚本所在的那个 DOM 节点，即<script>元素的 DOM 节点

document.implementation  //返回一个DOMImplementation对象



//元素特性相关属性

Element.id  //返回指定元素的id属性，可读写

Element.tagName  //返回指定元素的大写标签名

Element.dir //用于读写当前元素的文字方向

Element.accessKey  //用于读写分配给当前元素的快捷键

Element.draggble //返回一个布尔值，表示当前元素是否可拖动

Element.lang //返回当前元素的语言设置。该属性可读写

Elemnt.tabIndex //返回一个整数，表示当前元素在 Tab 键遍历时的顺序。该属性可读写。

Element.title  //用来读写当前元素的 HTML 属性title

Element.attributes  //返回当前元素节点的所有属性节点

Element.className  //返回当前元素的class属性，可读写

Element.classList  //返回当前元素节点的所有class集合

Element.innerHTML   //返回该元素包含的HTML代码，可读写

Element.outerHTML  //返回指定元素节点的所有HTML代码，包括它自身和包含的的所有子元素，可读写

Element.dataset   //返回元素节点中所有的data-*属性



//元素状态的相关属性

Element.hidden //返回一个布尔值，表示当前元素的hidden属性，用来控制当前元素是否可见。该属性可读写

Element.contentEditable //返回一个字符串，表示是否设置了contenteditable属性

Element.iscontentEditable //返回一布尔值，表示是否设置了contenteditable属性，只读



//尺寸属性

Element.clientHeight   //返回元素节点可见部分的高度

Element.clientWidth   //返回元素节点可见部分的宽度

Element.clientLeft   //返回元素节点左边框的宽度

Element.clientTop   //返回元素节点顶部边框的宽度

Element.scrollHeight  //返回元素节点的总高度

Element.scrollWidth  //返回元素节点的总宽度

Element.scrollLeft   //返回元素节点的水平滚动条向右滚动的像素数值,通过设置这个属性可以改变元素的滚动位置

Element.scrollTop   //返回元素节点的垂直滚动向下滚动的像素数值

Element.offsetParent  //返回最靠近当前元素的、并且 CSS 的position属性不等于static的上层元素

Element.offsetHeight   //返回元素的垂直高度(包含border,padding)

Element.offsetWidth    //返回元素的水平宽度(包含border,padding)

Element.offsetLeft    //返回当前元素左上角相对于Element.offsetParent节点的垂直偏移

Element.offsetTop   //返回水平位移

Element.style  //返回元素节点的行内样式



//节点相关属性

Element.children   //包括当前元素节点的所有子元素

Element.childElementCount   //返回当前元素节点包含的子HTML元素节点的个数

Element.firstElementChild  //返回当前节点的第一个Element子节点

Element.lastElementChild   //返回当前节点的最后一个Element子节点

Element.nextElementSibling  //返回当前元素节点的下一个兄弟HTML元素节点

Element.previousElementSibling  //返回当前元素节点的前一个兄弟HTML节点



//属性方法

Element.getAttribute()：读取指定属性

Element.getAttributeNames()：返回当前元素的所有属性名

Element.setAttribute()：设置指定属性

Element.hasAttribute()：返回一个布尔值，表示当前元素节点是否有指定的属性

Element.hasAttributes()：当前元素是否有属性

Element.removeAttribute()：移除指定属性



//查找方法

Element.querySelector()   //接受 CSS 选择器作为参数，返回父元素的第一个匹配的子元素

Element.querySelectorAll() //接受 CSS 选择器作为参数，返回一个NodeList实例，包含所有匹配的子元素

Element.getElementsByTagName()  //返回一个HTMLCollection实例，成员是当前节点的所有匹配指定标签名的子元素节点

Element.getElementsByClassName()  //返回一个HTMLCollection实例，成员是当前元素节点的所有具有指定 class 的子元素节点

Element.closest() //接受一个 CSS 选择器作为参数，返回匹配该选择器的、最接近当前节点的一个祖先节点（包括当前节点本身）



//事件方法

Element.addEventListener()：添加事件的回调函数

Element.removeEventListener()：移除事件监听函数

Element.dispatchEvent()：触发事件



//其他

Element.matches() //返回一个布尔值，表示当前元素是否匹配给定的 CSS 选择器

Element.scrollIntoView()   //滚动当前元素，进入浏览器的可见区域

Element.getBoundingClientRect()  //返回一个对象，包含top,left,right,bottom,width,height

Element.getClientRects()   //返回当前元素在页面上形参的所有矩形



//解析HTML字符串，然后将生成的节点插入DOM树的指定位置

Element.insertAdjacentHTML(where, htmlString);

Element.insertAdjacentHTML('beforeBegin', htmlString); // 在该元素前插入

Element.insertAdjacentHTML('afterBegin', htmlString); // 在该元素第一个子元素前插入

Element.insertAdjacentHTML('beforeEnd', htmlString); // 在该元素最后一个子元素后面插入

Element.insertAdjacentHTML('afterEnd', htmlString); // 在该元素后插入



Element.remove()  //用于将当前元素节点从DOM中移除

Element.focus()   //用于将当前页面的焦点，转移到指定元素上

Element.blur()  //用于将焦点从当前元素移除
```


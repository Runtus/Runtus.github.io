---
title: DOM扩展
date: 2021-12-26 10:22:54
tags: ["FE-FrontEnd"]
---

> 虽然原生DOM API已经能做许多事情了，但是仍然不断有标准或专有的扩展出现，以支持更多的功能，由于各个浏览器对DOM扩展的支持是专有的，为了统一这些专有的DOM API，W3C开始着手将这些**专有扩展转变为标准规范**。

## Selectors API

* Selectors APIs 是浏览器原生支持的CSS查询API，由于是原生支持，解析和遍历DOM树可以通过底层编译语言实现，相比其他Javascript库（比如jQuery），性能有数量级的提升。
* Selectors API Level 1 主要是两个API：querySelector() 和 querySelectorAll()



### querySelector()

* 该方法接受CSS选择符参数，返回匹配到的**第一个后代元素**，如果没有匹配元素则返回null。

```js
// 匹配 class 为 list 的第一个DOM元素
document.querySelector('.list')
// 匹配 id 为 myDiv 的第一个DOM元素
document.querySelector('#myDiv')

// 匹配 class=header DOM元素子元素中 id为someDiv的元素
const header = querySelector('.header');
header.querySelector('#someDiv')
```

* 如果直接在document上使用querySelector方法，**会从文档元素开始搜索（前两个例子）**；在Element上使用querySelector方法，**则只会在该元素的后代中查询（第三个例子）**。



### querySelectorAll()

* 该方法和querySelector类似，接受CSS选择符参数，只不过它会返回**匹配的所有节点**。即返回的是一个NodeList的**静态实例**。

* ⚠️注意：**无论是querySelector还是querySelectorAll返回的都是DOM元素的静态快照，即和之前提到的getElementById获取的动态实例不同，更改静态快照是不会影响DOM元素在页面上的渲染**。



### matches()

* matches方法接受一个css选择符参数，用于判断是否存在**对应css选择符的DOM元素**，如果存在返回true，否则返回false。

```js
if(document.body.matches("body.header")){
  // true
}
```



## CSS类扩展

* HTML5增加了一些特性以方便使用CSS类。



### getElementsByClassName()

* 该方法接受一个参数，包含**一个或多个类名**的字符串，返回类名中包含对应类的元素的NodeList。

```js
// 返回 class 包含 username 和 current 的类
document.getElementsByClassName('username current')
```



### classList属性

* classList属性提供了快速操作DOM元素类名的方法，如果是之前，对一个含有多个class的DOM元素进行类名操作会很麻烦，基本操作都是把其类字符串解析成数组，然后一个一个进行匹配操作，最后再合并为一个新的类字符串重新赋值回去。

```js
// 要删除"user"类
let targetClass = "user"; 
// 把类名拆成数组
let classNames = div.className.split(/\s+/); 
// 找到要删除类名的索引
let idx = classNames.indexOf(targetClass); 
// 如果有则删除
if (idx > -1) { 
 classNames.splice(i,1); 
} 
// 重新设置类名
div.className = classNames.join(" ");
```

* 现在通过classList可以快速对类名进行操作。
  1. add(value) -> 类名列表添加值为value的类
  2. contains(value) -> 返回布尔值，表示给定value是否存在
  3. remove(value) -> 从类名列表中删除指定字符串
  4. toggle(value) -> 如果类名中存在指定的value，则进行**删除**；如果不存在，则添加。

* **这个扩展方法在需要通过class来控制样式变换时非常有效**。

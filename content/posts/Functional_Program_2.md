---
title: "Functional_Program"
date: 2023-03-21T21:34:36+08:00
tags: ["FP"]
---

> 读 Professor Frisby's Mostly Adequate Guide to Functional Programming 记录

[Functional Programming原文链接](https://mostly-adequate.gitbook.io/mostly-adequate-guide/)

> 本章主要简单介绍函数式编程中的一等公民，以及函数式编程的一些注意事项和好处。

## 第二章: First Class Functions

* 在函数式编程中，函数是作为**一等公民**存在的，应该把函数和其他变量等同，即**函数也是变量，变量的类型也可以是函数，函数的参数也可以是函数**。

```js
const hi = name => `Hi, ${name}`
const greeting = hi
hi("小明");
```

* 常见的回调函数，其实就是把函数当作参数传递。

```js
const callback = (...args) => {
   // balabala
}

ajax("/xianbei114514/1919810", callback)
```



### 不要做无所谓的函数返回

* 在写代码时，有时会做无意义的操作，在一个函数中，将另一个函数返回。**其实它们的功能都没有变化，只是换了个变量名而已**。

```js
// 不好的操作
const getServerStuff = callback => ajaxCall(json => callback(json));
```

* 对于上述代码，getServerStuff本质就是想传递一个回调函数，并在回调函数中获取ajax请求得到的值，但完全没必要写那么多，它等同于下面代码实现。

```js
// 好的操作
const getServerStuff = ajaxCall;
```

* 此时getServerStuff的功能也是传递一个回调函数并获取到ajax的值，**这和上面有区别吗？没有区别。**



### 减少代码冗余，增强代码可读性以及可维护性

* 继续拿上述的ajaxCall例子来说，假如我们传递的回调函数如下所示。

```js
ajaxCall((json) => callback(json))
```

* 这只是一个调用，如果说有多处的ajaxCall都要进行如此的回调函数的传递，**那么一旦我们的业务发生改变时，就会去修改每一处代码，并且每次都要写一遍回调函数的形式，显得非常冗余**。
* 所以考虑将函数作为变量来传递。

```js
const cb = (json) => callback(json)

ajaxCall(cb)
```

* 此时一旦遇到上述情况，**直接去修改cb的内容即可，并且代码量也会减少**。



### 禁止this的出现

* 在函数式编程中，要保证函数是够**“纯”（1，5！）**的，即不能让函数内的任何行为影响到函数外，而`this`指针便是一个例子，this指向的是执行函数的作用域，一旦使用了this，便会导致函数和外界进行交互，从而**不纯**。
* 但在实际生产过程中，有时的业务情况又不得不这样操作，所以不能完全循规蹈矩，而是要懂随机应变，少量的交互并不会影响整体的代码质量。再说一句，若要调用含有`this`指针的函数，最好先使用bind函数改变`this`指针指向，这样不仅能让代码更加易读，还能减少一些未来执行时可能遇见的错误。

```js
// 借用网上的例子
// scary
fs.readFile('freaky_friday.txt', Db.save);

// less so
fs.readFile('freaky_friday.txt', Db.save.bind(Db));
```






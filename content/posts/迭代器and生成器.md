---
title: 迭代器and生成器
date: 2021-11-21 12:05:17
tags: ["javascript"]
---



## 迭代器 -> Symbol.iterator

* 当需要对某个迭代对象进行迭代处理时，由于**迭代之前需要事先知道如何使用数据结构**，以及**遍历顺序并不是数据结构固有的**，所以想寻求某种机制去统一迭代过程，对每一种可迭代类型，**都用同一种迭代方法**，从而增加开发体验。（**即无需事先知道如何迭代去实现迭代操作**）
* 于是基于以上原因，诞生了**迭代器**概念，意在统一化所有迭代对象的处理方式。
* 接受可迭代对象的原生语言特性包括如下：（即迭代对象都可以用下面的任意一种方式去处理）

1. for-of
2. 数组解构
3. 扩展操作符号（即`...`)
4. `Array.from`
5. 创建集合
6. 创建映射
7. `Promise.all()`接受由Promise组成的可迭代对象。
8. `Promise.rice()`接受由期约组成的可迭代对象。
9. `yield*`操作符，在生成器中使用。

* 基于以上特性，可以得知，迭代器也可以在宏观意义上理解为**类数组**。

### 判断与使用

* 如何判断一个对象是否为迭代类型是看它的`Symbol.iterator`属性是不是迭代器的工厂函数，如果是，则调用该函数后，会生成一个**迭代器对象**。

```js
const a = 1;
const b = {};
const c = [5,3,4,6]

console.log(a[Symbol.iterator]);
console.log(b[Symbol.iterator])
const c_iter = c[Symbol.iterator]() // -> 生成一个临时的迭代器对象，对迭代器对象可以通过统一方法进行迭代处理
// undefined
// undefined
// Object [Array Iterator] {} -> 迭代器对象
```

* 想对迭代器整体进行一个迭代处理，那么不需要显示去获取迭代器对象，直接调用上述的一系列方法去处理即可。
* 但如果是想对这个迭代过程进行**更细节的处理，即控制每一个元素**，那么需要获得迭代器对象，并调用`next()`方法去获取每一次迭代的值。

```js
const c = [5,3,4,6]
const c_iter = c[Symbol.iterator]()
console.log(c_iter.next()); // -> { value: 5, done: false }
```

* 可以从上述代码发现，调用next返回的值是一个对象，value为当前迭代元素的值，done则标记**本次获取值后是否迭代完成**。

* 除了`next`函数，迭代器还包括一个可选的`return`函数，它会在**迭代中止时执行**，而迭代中止可能的情况如下：
  1. for-of的break，continue，return，throw提前退出。
  2. 解构操作并未消费所有值。
* 所以如果需要hook到迭代中止的情景，那可以自定义`return`方法。

```js
class Counter { 
     constructor(limit) { 
    	 this.limit = limit; 
     } 
     [Symbol.iterator]() { 
       let count = 1, 
       limit = this.limit; 
       return { 
       next() { 
         if (count <= limit) { 
         return { done: false, value: count++ }; 
         } else { 
         return { done: true }; 
         } 
       }, 
      return() { 
        // 在这里面操作
        console.log('Exiting early'); 
        return { done: true }; 
    	} 
   	}; 
   } 
}
```

* 接上，如果某次中止后，迭代器并没关闭的化，则还可以接续从上次离开的地方继续迭代，比如数组的迭代器就是不可关闭的。

```js
let a = [1, 2, 3, 4, 5]; 
let iter = a[Symbol.iterator](); 
for (let i of iter) { 
 console.log(i); 
 if (i > 2) { 
 	break 
 } 
} 
// 1 
// 2 
// 3 
for (let i of iter) { 
 console.log(i); 
} 
// 4 
// 5
```



## 生成器

* 生成器是一个非常灵活的结构，拥有在一个函数块内暂停和恢复代码执行的能力。它的形式是一个函数，函数名称前面加一个`*`号就表示它是一个生成器。

```js
function *generartorFn() {}
// 注： 箭头函数不支持生成器声明
```

* 需要注意的是，**直接调用生成器函数生成的是一个生成器函数，而不是直接执行函数内部的内容**。即生成器对象一开始处于的是暂停执行的状态。与迭代器类似，生成器也实现Iterator接口，即它也是**可迭代的对象**，也具有next方法，**next方法就是控制生成器执行的方法**。

```js
function* generator(){
  yield 1;
  yield 2;
  yield 3;
  return 4;
}

const genObj = generator();
console.log(genObj.next())
console.log(genObj.next())
console.log(genObj.next())
console.log(genObj.next())

// { value: 1, done: false }
// { value: 2, done: false }
// { value: 3, done: false }
// { value: 4, done: true }

// or 
for (let o of genObj){ // genObj是可迭代对象，所以可以使用for-of循环
  console.log(o)
}

// 1
// 2
// 3
```

* 可以看到上述代码出现了`yield`关键字，它就是生成器函数中的暂停节点（中断执行），调用一次next方法后，代码会运行到下一个`yield`之处。
* 同时yield还提供了输入输出功能，比如上述代码中，**yield后的值就是本次调用next方法的返回值，若向本次的next方法中传入值，`yield`将作为临时变量存储对应的值**。

```js
function* generator(){
  console.log(yield)
  console.log(yield)
  console.log(yield)
}
              
const genObj = generator();
genObj.next(1)
genObj.next(2)
genObj.next(3)
genObj.next(4)
// 2
// 3
// 4
```

* 可以看到，1没有打印出来，是因为第一次调用next只是启动函数，并没有实际运行到console.log处。

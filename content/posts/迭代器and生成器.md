---
title: 迭代器and生成器
date: 2023-08-23T10:12:13+08:00
tags: ["javascript"]
---

> 很多的数据结构都具备迭代的性质，但是不同的数据结构的迭代方法有所不同，往往需要知道具体的数据结构来选择对应的迭代方法，例如数组的迭代可以通过索引来进行迭代。
>
> 为了统一迭代接口，使得我们**可以不了解具体的数据结构的前提下**也能进行迭代，所以有了迭代器这么一个概念，而生成器则是基于迭代器的一种能够控制函数流程的方法，生成器基于迭代器的原理运行，反过来生成器也能够快速构建迭代器。



## 迭代器

* 当需要对某个迭代对象进行迭代处理时，由于**迭代之前需要事先知道如何使用数据结构**，以及**遍历顺序并不是数据结构固有的**，所以想寻求某种机制去统一迭代过程，对每一种可迭代类型，**都用同一种迭代方法**，从而增加开发体验。（**即无需事先知道如何迭代去实现迭代操作**）
* 于是基于以上原因，诞生了**迭代器**概念，意在统一化所有迭代对象的处理方式。

#### 可迭代协议

* 一个对象是**可迭代对象**，则需要暴露一个属性作为**默认迭代器**，并且该属性的`key`为`Symbol.iterator`，它的`value`是一个**工厂函数**，用于返回**一个新迭代器**。
* `js`提供了一系列可以对**可迭代对象**进行操作的原生结构，如下图所示。

1. for-of
2. 数组解构
3. 扩展操作符号（即`...`)
4. `Array.from`
5. 创建集合
6. 创建映射
7. `Promise.all()`接受由Promise组成的可迭代对象。
8. `Promise.rice()`接受由期约组成的可迭代对象。
9. `yield*`操作符，在生成器中使用。

* 上述谈到的原生结构在对**可迭代对象**进行操作时，会默认调用**工厂函数**生成一个**迭代器**，然后对迭代器进行操作。

#### 迭代器协议

##### 1. `next`和`IteratorResult`

* 可迭代协议描述了一个**对象具有可迭代性质**的要求和前提，而**迭代器协议**则是用于规范**迭代器具有的性质和方法**。
* 迭代器对象具有一个方法`next`，通过迭代器API`next()`能够在**可迭代对象**中遍历数据，**每次调用`next`都能获取到一个`IteratorResult`**对象，其中包含迭代器返回的**下一个值**，如下所示。

```javascript
const array = [1, 2, 4];

// 迭代器对象
const iter = array[Symbol.iterator]();

console.log(iter.next())
console.log(iter.next())
console.log(iter.next())
console.log(iter.next())
console.log(iter.next())

// output
// { value: 1, done: false }
// { value: 2, done: false }
// { value: 4, done: false }
// { value: undefined, done: true }
// { value: undefined, done: true }
```

* 如上的输出结果所示，`IteratorResult`包含两个属性`value`和`done`，`value`表示本次迭代获取的值，而`done`则表示**迭代是否结束**，这从另一个角度也说明**迭代器只能通过`next`方法**来获取迭代器的当前位置。

##### 2. 迭代器的一次性和相互独立性

* 同时需要注意的是，**每个迭代器对象是一次性的，并且每个迭代器对象之间没有联系**，一次性体现为调用`next()方法是单向不可逆的，即遍历过的元素是不能再次遍历的。而每个通过工厂函数生成的迭代器是彼此独立的，**在遍历时不会互相干扰**。

```javascript
const array = [1, 2, 4];

// 迭代器对象
const iter1 = array[Symbol.iterator]();
const iter2 = array[Symbol.iterator]();

console.log(iter1.next())
console.log(iter2.next())
console.log(iter1.next())
console.log(iter2.next())

// output iter1和iter2遍历不会互相干扰，是彼此独立的
// { value: 1, done: false }
// { value: 1, done: false }
// { value: 2, done: false }
// { value: 2, done: false }
```



##### 3. 迭代器对象是引用

* 迭代器是通过直接引用**原可迭代对象**来获取其中的值，这意味着**在迭代器迭代过陈宝国周哥你，当原迭代对象更改内部的值时，迭代器迭代的值也会改变**，如下所示。

```javascript
const array = [1, 2, 4];

// 迭代器对象
const iter = array[Symbol.iterator]();

console.log(iter.next())
array[1] = 5
console.log(iter.next())
console.log(iter.next())
array.push(6)
console.log(iter.next())
array.push(7)
console.log(iter.next())

// output
// { value: 1, done: false }
// { value: 5, done: false }
// { value: 4, done: false }
// { value: 6, done: false }
// { value: 7, done: false }
```

* 上述结果可以看到，无论更改了`array`内部的值，还是`array`新增的值，**迭代器都能获取到迭代对象最新的内部值**。
* 在上述的例子中，都是通过`next()`方法来手动控制迭代器的位置，而在实际开发中对**可迭代对象**处理的常见方式是通过**上述的js提供的原生结构来对可迭代对象进行处理**，通过原生结构来处理可迭代对象时，本质就是`不停调用next方法`来获取每一个迭代值，只到迭代结束，例如下所示。

```javascript
const array = [1, 2, 4];
for (let o of array) {
    console.log(o)
}

// 等效于以下代码
const iter = array[Symbol.iterator]();
for (let o of iter) {
    console.log(o)
}
/*
输出
1
2
4
1
2
4
*/
```

* 通过上述代码也可以知道，迭代器本身也是可迭代的，并且具有**自引用性**。

```javascript
const iter = array[Symbol.iterator]();
iter === iter[Symbol.iterator]();
// true
```



#### 自定义迭代器

* 除了上述的默认迭代器，也可以自定义迭代器，通过覆盖`[Symbol.iterator]`的工厂函数来进行迭代器自定义。

```javascript
class Counter {
  constructor(limit) {
    this.limit = limit;
  }
  // 自定义迭代器
  [Symbol.iterator]() {
    let count = 1,
      limit = this.limit;
    return {
      next() {
        if (count <= limit) {
          return { done: false, value: count++ };
        } else {
          return { done: true, value: undefined };
        }
      },
    };
  }
}
let counter = new Counter(3);
for (let i of counter) {
  console.log(i);
}
// 1
// 2
// 3
for (let i of counter) {
  console.log(i);
}
// 1
// 2
// 3

```

* 上述的`Counter`类实现了自定义迭代器方法后，它便是一个**可迭代对象**，能够使用`for ... of ...`等一系列处理迭代的原生对象。



#### 提前终止迭代器

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

* 接上，如果某次中止后，迭代器并没有关闭，则还可以接续从上次离开的地方继续迭代，比如数组的迭代器就是不可关闭的。

```js
let a = [1, 2, 3, 4, 5]; 
let iter = a[Symbol.iterator](); 
for (let i of iter) { 
 console.log(i); 
 // 在i = 3时会终止迭代器，并且自动调用return方法。
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

* 生成器是一个非常灵活的结构，拥有在一个函数块内暂停和恢复代码执行的能力。它的形式是一个函数，函数名称前面加一个`*`号就表示它是一个生成器函数。

```js
function *generartorFn() {}
// 注： 箭头函数不支持生成器声明
```

* 需要注意的是，**直接调用生成器函数生成的是一个生成器对象，并且不会直接开始执行函数内部的代码**。生成器对象本身就具有迭代器的性质，它实现Iterator接口，是**可迭代的对象**，也具有next方法，**next方法就是控制生成器执行的方法**。

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

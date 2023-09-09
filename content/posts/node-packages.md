---
title: "Node Packages"
date: 2023-09-09T10:34:09+08:00
tags: ["node"]
---

> 之前一直被nodejs的模块执行规则所困扰，这种困扰来自于目前`Commonjs`和`ESModule`在Node端能够并存的情况，什么时候能够执行`ESModule`，什么时候不能执行；在引用第三方包`node_modules`，具体的包是根据什么来区分是`ESModule`还是`Commonjs`。
>
> 通过阅读了Nodejs的原文档，这些问题也随之迎刃而解，所以特此在本篇做一做记录。
>
> 注：以下内容都是基于**Nodejs V20.5.0** 进行编写。



## Nodejs中的模块系统

* 在`Node`环境下，如今共存着两种模块规范：`ESModule`和`Commonjs`，前者是ES6中提出模块化规范，所以为了统一整个Javascript的模块化规范，Nodejs也逐渐的由之前主流的`Commonjs`规范转变为`ESModule`，只不过为了兼容`Nodejs`老版本的代码，所以现在这两种模块在Node中是并存的关系，但是前者(`ESModule`)是趋势。
* 关于`Commonjs`和`ESMoudle`的模块技术细节，例如语法区别，加载规则等在本篇不会提到，接下来将会它们在`nodejs`中的设置以及引用规则。



### 模块的执行与加载

* Node在执行代码前会先去判断代码是`ES`模块还是`commonjs`模块，Node会从三个角度依次判断代码的模块归属。

#### ES模块

* 对于ES模块，当出现以下情况时，Nodejs会将文件视为ES模块。
  1. 扩展名为`.mjs`的文件。
  2. 项目所归属的`package.json`的`type`字段的值为`module`，此时文件扩展名为`.js`的文件都视为`ES`模块。
  3. 字符串作为参数传入 `--eval`，或通过 `STDIN` 管道传输到 `node`，带有标志 `--input-type=module`。
* `.mjs`扩展名的文件被视为`ES`模块的优先级是最高的，无论它身处何方，只要`Node`发现其是`.mjs`的文件，就会将其视作`ES`模块来执行。



#### commonjs模块

* 对于commonjs模块，当出现以下情况时，Nodejs会将文件视为`commonjs`模块。
  1. 扩展名为`.cjs`的文件。
  2. 项目所属的`package.json`的type字段的值为`commonjs`时，此时文件扩展名为`.js`的文件都视为`commonjs`模块。
  3. 字符串作为参数传入 `--eval` 或 `--print`，或通过 `STDIN` 管道传输到 `node`，带有标志 `--input-type=commonjs`。
* 和`ES`模块类似，如果文件扩展名为`.cjs`，那么Nodejs都会将其视作`commonjs`模块。
* **如果不设置type字段，那么该项目将会默认为`commonjs`语法的项目**。



#### 模块加载器

* Nodejs有两种加载器用于分别解析说明符和加载`ES`模块和`commonjs`模块。

* **`commonjs`模块加载器:**

  1. 🌟 处理`require()`调用，并且是同步加载模块的，
  2. 🌟 支持**以文件夹作为模块**。
  3. 🌟 不能用于加载`ECMAScript模块（ESModule）。
  4. 🌟 解析路径时，如果未找到**完全匹配项**，那么将尝试添加扩展名**`.js`,`.json`,`.node`**，然后最后再尝试解析**文件夹作为模块**。
  5. 将`.json`文件视为JSON文本文件，可以直接使用`require`加载。

  * 特别注意上述的四个带🌟的规则，在平时日常开发中应该是能够经常遇见的。**从第4点也能知道为什么使用`require`引用包时，可以不带文件的扩展名，因为node在解析路径时会自动添加**。

* **`ECMAScript`模块加载器**:

  1. 处理`import()`和`import`表达式，并且是异步加载模块的。
  2. 

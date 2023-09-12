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

  1. 🌟 处理`import()`和`import`表达式，并且是异步加载模块的。
  2. 🌟 只接收`Javascript`文本文件的`.js`,`.mjs`和`.cjs`扩展名。
  3. 🌟 可以加载JSON模块，但**需要导入断言**。
  4. 🌟 **不支持文件夹作为模块，必须完全指定目录索引。(例如`./dir/index.js`)**。
  5. 🌟 可以加载`Javascript Commonjs`模块，导入的`cjs`模块会通过`cjs-module-lexer`来尝试识别命名的导出，同时导入的`CommonJS`模块会将其URL转化为绝对路径，然后通过`CommonJS`模块加载器加载。

  * 需要注意的是第五点：`ESModule`可以加载`CommonJS`模块的文件，这和`Commonjs`语法是不同的。

* 以`.mjs`结尾的文件总是加载为`ESModule`，不会管最近的`package.json`。

* 以`.cjs`结尾的文件总是加载为`commonjs`，不会管最近的`package.json`。



### 包入口点

* `package.json`中定义了该`node`项目的包信息，那么如果该`package`想被另一个`package`所引用，正如我们开发时引用第三方包那样`import xxx from "xxx"`，那么需要在`package`中定义包的入口点，及入口文件。

#### main字段

* main字段可以定义包的入口点，**它适用于`ES`模块和`CommonJS`模块入口点，并且支持所有Nodejs版本。**
* 但是它的缺点也显而易见，**只能定义一个入口点，及包的主要入口点**。
* 例如我们`package`的文件结构如下所示。

```shell
├── package.json
├── src
│   ├── index.js
│   └── pack.js
├── test.json
└── yarn.lock
```

* 设置`main: "src/index.js"`，那么当外部在引用该`package`时，将会直接引用`src/index.js`。

#### exports字段

* `exports`是`main`的现代替代方案，它不仅仅有着和`main`一样的功能，它还具有以下新特性:
  1. 可以定义**多个入口**，**不同环境的入口解析支持。**
  2. 🌟 **可以防止除`exports`字段指定的入口点外的其他入口点被引用。（安全）**
  3. 支持条件导出（分别指定`ES`环境和`commonjs`环境所引用的文件）
* 需要注意的是，`exports`字段并不支持所有的`Node`版本，只支持`Nodejs 10`以上版本的`package`，当有`exports`和`main`共存的`package`中，`exports`的优先级会大于`main`（`exports`支持的前提下）。
* 除此之外，上述的`exports`的第二个新特性非常值得关注，这可能是一个突破性的变化，消费者只能使用`exports`定义的文件入口，下面将展示一个例子。
* 我选择使用[`got`](https://www.npmjs.com/package/got)库来作为第二个特性的展示demo，`got`库的文件结构如下所示。

```shell
# got filetree
.
├── dist
│   └── source
│       ├── as-promise
│       │   ├── index.d.ts
│       │   ├── index.js
│       │   ├── types.d.ts
│       │   └── types.js
│       ├── core
│       │   ├── calculate-retry-delay.d.ts
│       │   ├── ....(还有很多文件)
│       │   └── utils
│       │       ├── get-body-size.d.ts
│       │       ├── ....(还有很多文件)
│       ├── create.js
│       ├── index.d.ts
│       ├── index.js
│       ├── types.d.ts
│       └── types.js
├── license
├── package.json
└── readme.md
```

* `package.json`的`exports`字段如下所示。

```json
{
  // 一个语法糖写法
  "exports": "./dist/source/index.js",
  // 等同于下面写法
  "exports": {
  	".": "./dist/source/index.js"
	}
}
```

* 这意味着外部引用`got`包时，会直接引用包中的`/dist/source/index.js`文件。
* 如果此时在外界引用`got`中的其他包，`node`运行时会出现报错，如下所示。

```js
// 外部包
import got from 'got';
import create from 'got/dist/source/create.js'
```

* 运行报错如下所示。

```shell
Error [ERR_PACKAGE_PATH_NOT_EXPORTED]: Package subpath './dist/source/create.js' is not defined by "exports" in /User/xxxxx ....
```

* 可以看到，`node`会报`[ERR_PACKAGE_PATH_NOT_EXPORTED]`的错误，这正对应了之前所说的`exports`的第二个新特性，不允许引用除了`exports`指定文件的其他文件。
* 此时给`got`库中`exports`字段增加一个`create.js`的引用，那么我们在外界也能直接引用`create.js`。

```json
{
  "exports": {
  	".": "./dist/source/index.js",
    "./create.js": "./dist/source/create.js"
	}
}
```

* 然后在外部进行引用。

```js
import got from 'got';
import create from 'got/create.js'
```

* 此时执行文件，便能成功执行。

#### exports中为不同模块设置不同入口

* 从刚才的demo可以看到，`exports`能定义多个入口，自然便可以利用这一点，为不同的模块语法定义不同的入口文件，这样能够大大提高包的兼容性。
* 如果包想要为`require() - commonjs`和`import() - ESmodule`提供不同的模块导出，则可以写成如下的形式。

```json
{
  "exports": {
    "import": "./dist/index.mjs",
    "require": "./dist/index.cjs"
  }
}
```

* 如此设置后，外部使用不同语法引用该包时，会根据语法的不同来引用不同的文件。



> 以上便是在平时开发中接触的关于`node-package`比较多的知识点了，这一块儿知识很重要，它不仅解决了我之前关于`ESModule`和`Commonjs`在`node`中的执行理解混淆，还进一步解决了我在打包时对于`packagejson`配置的疑惑，关于更多的`node-package`知识点请移步[node官方文档](https://nodejs.cn/api/packages.html#nodejs-packagejson-%E5%AD%97%E6%AE%B5%E5%AE%9A%E4%B9%89)。

---
title: "Web components"
date: 2023-08-03T10:12:13+08:00
---

## Web-Components

* Web-Components是一项**标准，规范**，目前它包含了三项主要技术：
  1. `Custom Elements`自定义元素：（标签）它是一组JavaScript API，能够自定义Element以及其行为。
  2. `Shadow DOM`影子DOM。
  3. `HTML templates`HTML模板：通过`<template>`和`<slot>`元素编写不在呈现页面中显示的标记模板。

* 通过这三个特性的共同作用能够**创建封装功能的定制元素**，在说明Web-Component的用法之前，先简单说明上述三项特性。



### Custom Elements

* 自定义元素是Web Components中的一个重要特性，它能够让开发者将HTML页面（或者页面中的某个功能）封装为`custom elements`，从而达到复用的目的。目前支持`custom elements`的浏览器有**FireFox，Chrome，Opera**。
* Custom Elements的管理是通过`CustomElementRegistry`接口进行操作的，其用于处理Web文档中的`custom elements`，同时它还提供**注册自定义元素和查询已注册元素**的方法，它的实例通过`window.customElements`属性来获得。

* `CustomElementRegistry`接口有四个方法：
  1. [`CustomElementRegistry.define()`](https://developer.mozilla.org/zh-CN/docs/Web/API/CustomElementRegistry/define)：定义一个新的自定义元素。
  2. [`CustomElementRegistry.get()`](https://developer.mozilla.org/zh-CN/docs/Web/API/CustomElementRegistry/get)：返回指定自定义元素的构造函数，如果未自定义元素，则返回`undefined`。
  3. [`CustomElementRegistry.upgrade()`](https://developer.mozilla.org/zh-CN/docs/Web/API/CustomElementRegistry/upgrade): 更新一个自定义元素。
  4. [`CustomElementRegistry.whenDefined()`](https://developer.mozilla.org/zh-CN/docs/Web/API/CustomElementRegistry/whenDefined)：执行并返回一个已经定义的自定义元素的`promise`，即如果定义了这么一个元素，那么返回对应的`promise`。



#### `CustomElementRegistry.define()`

* 该方法是四个方法中最重要的方法，用于创建自定义元素，它接受三个参数：
  * 自定义元素的名称，且其必须**符合元素名称的`DOMString`标准字符串**。
  * 用于定义元素行为的**类**。
  * 一个包含 `extends` 属性的配置对象，是可选参数。它指定了所创建的元素继承自哪个内置元素，可以继承任何内置元素。

```javascript
// 自定义Div
class SelfDiv extends HTMLElement {
    constructor(){
        // super方法的调用是必须得
        super()

        //  元素的相关业务代码
    }
}

// 使用customElements实例(CustomElementRegistry接口)来完成注册功能
customElements.define("word-count", SelfDiv, { extends: "div" });
```

* 上述代码便是**自定义元素注册**的简单演示。



### shadow DOM

* shadown DOM也叫影子DOM，它最主要的功能是做封装，将**元素的标记结构，样式以及行为**隐藏起来，与外界隔离，这样能够保证封装的代码既不会被外界影响，同时也能保证内部的代码不会影响到外部的元素，这样便实现了Web Component的**解耦合**。
* `Shadow DOM`接口可以将一个**隐藏的，独立的**DOM附加到一个元素上，目前为止`FireFox,Chrome,Opera,Safari`默认支持`Shadow DOM`，`Chromium内核的Edge`也支持。
* **Shadow DOM**允许将**隐藏的DOM树**附加到常规的DOM树中 => **以shadow root节点为起始根节点（Shadow Root的创建后续会说明）**，在这根节点的下方可以添加任何DOM元素，**和普通的DOM元素没有任何区别。**
* 下面有个示意图可以帮助理解。

![shadow-dom](/images/shadow-dom.png)

* 上图的一些概念下面做一些解释：
  * `Shadow host`：常规的DOM节点，`Shadow DOM`将会挂载到此处。
  * `Shadow Tree`：`Shadow DOM`内部的DOM树。
  * `Shadow root`: `Shadow tree` 的根节点。
* 注：`Shadow DOM`的操作方式和普通DOM操作方式没有任何区别，包括**添加元素，设置属性等等**，只不过`Shadow DOM`内部的任何改变都影响不了外部的DOM元素。
* `Shadow DOM`的创建和挂载通过方法`ElementShadow()`来实现。

#### `Element.attachShadow()`

* `Element.attachShadow()`将`shadow root`附加到指定的元素上，它接受一个[`ShadowRootInit`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/attachShadow)对象作为参数，其中包含一个`mode: 'open' | 'closed'`属性。
* `Element.attachShadow()`返回一个`ShadowRoot`对象，通过`ShadowRoot`能够操作`Shadow DOM`中的元素

```js
// open表示可以通过页面内的javascript方法来获取Shadow DOM 
let shadow = elementRef.attachShadow({ mode: "open" });、
// close表示主动无法从外部通过javascript方法来获取shadow DOM
let shadow = elementRef.attachShadow({ mode: "closed" });

// 通过 Element.shadowRoot属性来获取
let myShadowDom = myCustomElem.shadowRoot;
// 如果mode = 'open'，则返回对应的Shadow DOM
// 如果mode = 'close'，则返回null
```

* 到这里，可以发现`Shadow DOM`能够隔离环境，而`Custom Element`能够封装基本的**HTML元素和相关业务代码**形成具有实际语义的自定义标签，那么便可以将二者结合，结合二者的特性，从而构建**具有单独环境的自定义标签**，这和我们预想的`component`已经非常接近了。**同时，这也是目前Shadow DOM最实用的用法**，代码如下所示。

```js
class SelfDiv extends HTMLElement {
    constructor(){
        super()

        // 将ShadowRoot挂载到自定义标签上。
        let shadow = this.attachShadow({ mode: "open" });

        // 基于ShadowDOM进行处理，就像处理常规的DOM一样
        shadow.appendChild(tpl.content.cloneNode(true))
    }
}
```

* 上述的例子很简单，在MDN上还有更详细的例子[Using Shadow DOM](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_components/Using_shadow_DOM)



### HTML templates and slots

* 虽然`Shadow DOM` + `Custom Elements`已经无限接近于理想中的`Components`，但它目前还有一些问题：`Custom Elements`和`Shadow DOM`都是通过Javascript API来进行控制的，它们并没有直接去**创建和定义HTML DOM的内容**，从上述的描述中都可以看到，它们只是去**获取已有的DOM元素或者是通过Javascript来定义简单的DOM元素。**
* 所以还需要一个方法去定义具有**Component**性质的DOM元素集合（DOM子树），`HTML templates`便可以用于解决`Web Components`中的DOM元素定义。
* 在`<template> </template>`中定义的元素**不会渲染到DOM树**中，但是可以通过`JavaScript`来获取它的引用，通过`Javascript API`的方式来手动控制`template`的渲染。


```html
// index.html
<template id="my-paragraph">
  <p>My paragraph</p>
</template>
```

```js
// script.js
let template = document.getElementById("my-paragraph");
let templateContent = template.content;
document.body.appendChild(templateContent);
```

#### Web Component中使用 template

* 将`template`和`web component`做结合能够发挥出很好的效果，大致的步骤如下所示
  1. 首先定义一个Web组件，并在js中获取到它的引用。
  2. 将组件作为`Shadow DOM`的内容。
  3. 将`ShadowRoot`挂载到自定义标签上。

```html
<template id="my-paragraph">
  <style>
    p {
      color: white;
      background-color: #666;
      padding: 5px;
    }
  </style>
  <p>My paragraph</p>
</template>

<!-- 直接引用注册的Custom element即可 -->
<my-paragraph></my-paragraph>
```

```js
// ShadowRoot 挂载到自定义标签上（注册）
customElements.define(
  "my-paragraph",
  class extends HTMLElement {
    constructor() {
      super();
      // 获取template的引用
      let template = document.getElementById("my-paragraph");
      let templateContent = template.content;

      const shadowRoot = this.attachShadow({ mode: "open" });
      // 将template挂载到Shadow DOM下
      shadowRoot.appendChild(templateContent.cloneNode(true));
    }
  },
);
```



#### slots

* `slots`顾名思义为插槽，它能够为`template`标签包含的内容**预留位置**，就像把`template`中的内容插入到对应的位置一样，所以叫作`slots`。
* `slots`有属性`name`标识，并且允许在`template`标记占位符，这样使用`template`时，其中的内容便能准确无误地插入到对应地位置上。

```html
<template id="tpl">
        <style>
            article {
                width: 20%;
                margin: 20px auto;
                border: solid 1px gray;
                padding: 8px;
            }

            header {
                background: lightblue;
                color: #fff;
                font-size: 24px;
                border: solid 1px lightblue;
            }

            .test {
                color: red;
            }
        </style>
        <article>
            <header>
                <!-- 博客标题插槽 -->
                <slot name='title'>博客标题</slot>
            </header>
            <section>
                <!-- 博客内容插槽 -->
                <slot name='cont'>博客内容博客内容博客内容博客内容博客内容博客内容...</slot>
            </section>
        </article>
    </template>
```

```js
    <my-blog>
        <span slot='title'>第一篇博文</span>
        <p slot='cont'>这是第一篇博文内容这是第一篇博文内容这是第一篇博文内容这是第一篇博文内容.</p>
    </my-blog>

    <my-blog>
        <span slot='title'>第二篇博文</span>
        <p slot='cont'>这是第二篇博文内容...</p>
    </my-blog>
    <my-blog>
        <p>sdfe</p>
    </my-blog>
```

* 最后效果如下图所示。

![webcomponent-slot](/images/webcomponent-slot.png)

* 可以看到，通过`slot`指定插槽的名称，其中的内容能够准确无误地渲染到`template`定义的位置上，如果`template`中没有对应的内容，那么插槽将会渲染`template`定义的默认值（如上述的**非具名slot内容**）。



> 以上便是Web Component相关的基础知识和基础用法，但是它的强大远不于此，Web Component还有一系列的生命周期Hooks，这些内容将会在后续的博客中讨论。

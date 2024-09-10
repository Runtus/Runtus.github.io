---
title: "Rnode Ts Packages"
date: 2024-03-24T18:31:23+08:00
draft: true
---

> 我常常会有开发Node端的npm包的需求，为了方便每次的新Package开发，特此搭建一个模版，用于每次的Node端npm-package的开发，并且会同步开发一个cli工具用于快速生成对应的开发模版（template）
>
> 本篇文章主要是用于记录本人的本次搭建模版和cli的一次实践过程。

## cli原理

* 在我们平时搭建前端环境中无处不在使用cli工具，例如使用`create-vite`cli工具来构建vite项目，使用`create-react-app`来构建react项目等等。

* 在搭建模版之前，需要明白nodejs中cli工具是如何运作的。

  



## Template需要的基础组件

* 在搭建模版之前需要明确咱们的开发模版中需要哪些基础组件库，然后根据需求一个一个集成，需要的基础组件库如下所示。

1. Typescript
2. Rollup（打包工具）
3. Eslint + Prettier（代码格式化工具）
4. Jest（代码测试框架）

* 于是在下面的搭建的过程中将会按照上述的组件库的顺序依次集成。

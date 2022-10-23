---
title: vuepress开发遇到的一些问题
date: 2021-11-13 16:31:09
tags: ["vue"]
---

1. vuepress是SSR渲染，即vue挂载之前是在服务端进行的，所以尽量不要在before Mounted之前的hooks中调用浏览器API，否则打包时会报错。
2. 关于css中的 mix-blender滤镜模式，和 z-index关联比较多，具体体现在我在使用darkmodejs时，如何避免图片被mix-blender渲染，虽然官方给的方法是加入isolation：isolate属性（另启层叠上下文），但是并没有什么用，感觉是哪个地方出问题了，关于层叠上下文还有上述提到的属性需要重新学习下。
   * 另外，层叠上下文z-index和position关联很大，这个也要去做深究，我如果只是给image加z-index，则无法避免被滤镜覆盖的事实，应该是需要把他们纳入统一个层叠上下文才行，所以需要position：relative（注意，position默认是static）。这一块儿的知识也要重点去温习。



```js
const testFn = async () => {
  const a = 2;
  for(let i = 0; i < a; i++){
    // xxxxxx
  }

  return new Promise((res) => {
    res();
  })
}
```

```graph LR
A-.->B


```
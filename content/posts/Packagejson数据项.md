## npm发包相关的packagejson配置说明

> 将自己的项目上传至npm上时，需要正确设置packagejson中的相关字段，否则会导致上传的项目无法正确在其他地方引用，甚至无法上传至npm官网。

### 1. name

* name是整个项目的名称，也是在npm仓库中的唯一索引，当别人下载自己上传的npm包时，其关键字索引便是项目名称。

```shell
yarn add axios # axios 便是项目名称
```



### 2. version

* version是版本号，它是一个字符串，每次发版时都要更新该字段。
  * 版本号的编写遵循语义化版本 2.0.0 规范，即**「主版本号. 次版本号. 修订号」**



### 3. main

* 指定加载的入口文件，node端和web端都可以使用，当外部使用**require**来引入npm包时，main指定的文件将以```module.export```的形式对外暴露。
* 所以一般情况下，main文件都会指向commonjs规范的文件，如下所示。
* mjs文件代表commonjs规范的js文件。

```json
{
  "main": "dist/index.cjs"
}
```

### 4. module

* 与main不同的是，module指定的是ESM规范的文件入口，web端和node端都可以使用，如果npm包导出的是ESM规范的包，则需要该字段来指定入口文件。
* mjs代表是ESM规范的js文件。

````json
{
  "module": "dist/index.mjs"
}
````

### 5. type

* type用于指定该node项目遵循ESM规范还是Commonjs规范，默认为commonjs项目，若要设置项目为ESM规范，则需要赋值module。

```json
{
  "type": "module"
}
```

* 同时当项目是ESM规范时，需要指定入口文件，即设置module字段。



### 6. types

* 指定声明文件位置，即d.ts文件。



### 7. files

* files字段接收一个数组，用来指明在发布npm包时，哪些文件需要上传至npm服务器上。或者说，用来描述当把 npm 包作为依赖包安装时需要说明的文件列表。
* 需要说明的是，如果有文件不想上传到npm服务器上，可以用.npmignore来说明哪些文件不需要上传（和.gitignore类似）
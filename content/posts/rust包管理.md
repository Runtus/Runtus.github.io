---
title: "Rust包管理"
date: 2023-07-05T17:33:19+08:00
tag: ["rust"]
---

> 本文参考了Rust语言圣经中有关Rust包的介绍，攥写本章的目的是为了更好的掌握rust中包和模块的关系以及它们的代码组织方式，方便于未来的项目开发。

## Rust中代码组织相关概念

- 在Rust中，代码的组织大致可以分为四个层次：
  1. 项目（Package）
  2. 工作空间（Workspace）
  3. 包（Crate）
  4. 模块（Module）

### Package 项目

- Package其实就是通过命令```cargo new```创建的项目文件，其**显著特征**便是包含了**cargo.toml**文件，该文件标注了该Package的一些基本信息：例如名称，版本号，依赖等等。
- 一个Package由**一个或多个包（Crate）**组成，但是它最多只能包含一个**库类型的包（即名为lib.rs的文件）。**
- Package还可以分为**二进制Package和库Package**。

#### 二进制Package

* 直接使用命令```cargo new package-name```的Package-项目即为二进制项目，虽然在**cargo.toml**中没有显示指出Package的入口文件，但Cargo的惯例是：**src/main.rs**即为二进制**包**的根文件，即入口文件，所有的代码的执行都是从**src/main.rs**中的```fn main()```中开始执行的。
* 输入```cargo run```可以直接编译运行。

#### 库Package

* 库Package在创建时需要增加```--lib```命令行参数选项，即```cargo new package-lib-name --lib```在，这样获得的Package是一个库Package，**它只能作为一个第三方库被其他项目引用，而不能单独编译运行。**
* 与 `src/main.rs` 一样，Cargo 知道，如果一个 `Package` 包含有 `src/lib.rs`，意味它包含有一个库类型的同名包 `my-lib`，该包的根文件是 `src/lib.rs`。



#### Package文件结构

* 需要注意的是，`main.rs`和`lib.rs`不是互斥关系，二者是可以共存的。当二者共存时，那就意味着它包含两个包：库包和二进制包，这两个包名也都是 `package-name` —— 都与 `Package` 同名。
* 下面是一个Package的文件结构。

```css
.
├── Cargo.toml
├── Cargo.lock
├── src
│   ├── main.rs
│   ├── lib.rs
│   └── bin
│       └── main1.rs
│       └── main2.rs
├── tests
│   └── some_integration_tests.rs
├── benches
│   └── simple_bench.rs
└── examples
    └── simple_example.rs
```

* 默认的二进制包：`src/main.rs`，编译后的包名称和Package名称相同。
* 其余的二进制包: `src/bin/main1.rs`和`src/bin/main2.rs`，编译后是一个自身文件名相同的二进制包。
* 唯一的库包：`src/lib.rs`
* 测试文件: `test/some_integration_tests.rs`
* 基准性能测试benchmarks: `benches/simple_bench.rs`





### 模块Module

* 一个**Package**内有一个或者多个**包（Crack）**，而一个**包（Crack）**里又可以有一个或者多个**模块（Module）**。
* 模块Module是用关键字```mod```实现的，并且模块之中可以直接嵌套模块，示例代码如下所示。

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

* 模块```front_of_house```中嵌套了子模块```hosting```和```serving```。

#### 模块的引用

* 调用模块内的函数的方式都是通过**路径应用**的方式来调用的，为何要称其为**路径引用**的方式，是因为模块的组织结构本身就是**一棵树**，称之为**模块树**，而**模块树**的组织方式和文件的组织方式相同，那么就可以把文件引用的方式来类比模块中函数的引用方式，所以称其为**路径引用**的方式。
* 路径应用方式有两种：**绝对路径引用**和**相对路径引用**。

##### 绝对路径引用

* 绝对路径应用以根包开头，即```crate```。例如下所示：

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();
}
```

* 需要注意的是：根包的名称是`crate`，并且每层的调用符号为`::`。

##### 相对路径

* 相对路径的起点便是以**当前文件所对应的模块位置为起点**进行调用，代码如下所示。

```rust


mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}

```

* 由于调用函数和模块定义在同一个文件中，所以可以直接引用模块名进行调用，**如果模块声明定义和函数调用不在一个文件下，需要在该文件中进行声明（类似于Javascript的import语法）**，具体的例子会在后续的例子中介绍。

#### 模块中的可见性

* 如果直接将上述路径引用的代码进行编译，会发现编译错误。具体原因便是**Rust出于安全考虑，默认情况下所有的类型都是私有化的，包括模块本身，函数，方法，结构体等等**。所以说在上述的例子中，除了能引用顶层模块`front_of_house`（因为它们的作用域在同一层中），其内层的模块以及方法都是无法调用的。
* ⚠️这里需要说明一点的是：**在Rust中子模块可以完全访问父模块的内容，但是父模块无法访问子模块的任何信息，子模块访问父模块信息通过`super`关键字进行调用**。
* 所以为了让模块以及其内部的方法能被外界调用，需要关键字`pub`进行声明，更改后的代码如下所示。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}

```

* 这样便能使得编译顺利通过。

#### super和self关键字

* `super`关键字在上述可见性一章已经谈到，用于**子模块引用父模块**的关键字。
* 关键字`self`则类似于`this`关键词，用于引用**自身模块**的项，例如刚才的**相对路径例子中的调用**，`front_of_house::hosting::add_to_waitlist();`，其本质是`self::hosting::add_to_waitlist()`。



### 模块与文件分离

* 为了代码合理组织，在大多数情况下最好把多个模块拆到多个文件中，正如刚才所言，多个模块组合在一起其实就形成了一个**模块树**，而模块树是类似于文件组织的结构，所以便可以**参考模块树将模块分离到各个文件中。**
* 现在将**模块[Module]章节**代码中的模块树画出，如下所示。

```console
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

* 先创建一个文件`src/front_of_house.rs`，并把`front_of_house`模块的定义放进去，如下代码所示。

```rust
// src/front_of_house.rs
pub mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}

        pub fn seat_at_table() {}
    }

    pub mod serving {
        pub fn take_order() {}

        pub fn serve_order() {}

        pub fn take_payment() {}
    }
}
```

* 然后在`src/lib.rs`下进行引用。

```rust
// src/lib.rs
mod front_of_house; // 声明

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

* 注意第一行代码```mod front_of_house```，这就是之前提到的模块声明，它的意思是**告诉Rust从另一个和模块`front_of_house`同名的文件中加载该模块的内容，类似于`import`的功能**。
* 此时front_of_house依然是个嵌套模块，所以可以考虑继续**将嵌套模块拆分**。

#### 拆分子模块到文件中

* 创建一个同名的文件夹，然后在文件夹中创建与子模块同名的文件，如`src/front_of_house/hosting.rs`。
* 然后将`hosting`模块的函数定义放到该文件下，如下所示。

```rust
// src/front_of_house/hosting.rs
pub fn add_to_waitlist() {}
pub fn seat_at_table() {}
```

* 其余子模块的创建方式与之类似，这里不再赘述。
* 但是此时如果直接编译还是会出现错误，因为需要**显示指定暴露了哪些子模块**，在如今Rust版本下，有两种方法可以选择：
  1. 在父模块目录里（本节中指的就是`front_of_house`文件夹）创建`mod.rs`，在里面指定要暴露的子模块。(此方法是Rust1.30版本之前的唯一暴露子模块的方法)
  2. 与父模块目录同级的目录里（本节中指的就是`src`文件夹）创建`front_of_house.rs`，在里面指定要暴露的子模块。
* 无论是哪种方式，它们其中要填写的内容都是需要暴露的子模块的名称，如下所示。

```rust
pub mod hosting; // 暴露hosting子模块
pub mod serving; // 暴露serving子模块
```


---
title: "Rust Trait"
date: 2023-07-17T20:09:43+08:00
tags: ['rust']
---

> Rust中的特征Trait类似于其他语言中的接口，它定义了**一组可以被共享的行为**，只要实现了特征，就能使用这组行为。

## 特征 Trait

#### 特征的定义

* 通过`trait`关键字对特征进行定义。

```rust
pub trait Student {
  fn GoClass(&self);
  fn LeaveClass(&self);
  fn getClassRoom(&self) -> String;
}
```

* 上述声明了**身为学生应该有的几个特征行为：**即上课，下课和获取教室房间号，那么其他具有学生性质的结构体（或者说为类）需要遵循该特征。
* **需要注意：特征只是坐函数签名，并不是真正的实现函数，函数的实现在绑定了该特征的类里实现，下面一节会提到。**



#### 类型实现特征

* 使用`for`关键字来为类实现特征。

```rust
pub struct Bob {
  pub year: String,
  pub sex: String
}

impl Student for Bob {
 // 实现特征
  fn GoClass(&self){
    // ....
  }
  
  fn LeaveClass(&self) {
    // ...
  }
  
  fn getClassRoom(&self) {
    // ...
  }
}
```

* 在Bob类型中实现了**Student特征声明的三个函数**，这是必须的，**除非在特征中有默认的实现**。
* 如果**特征中有默认的函数实现，那么绑定的类型可以不用再次实现函数，若实现了对应的函数，那么会覆盖默认的特征函数实现**。如下所示。

```rust
pub trait Student {
  fn GoClass(&self){
    println!("this is student!");
  }
  fn LeaveClass(&self);
  fn getClassRoom(&self) -> String;
}

impl Student for Bob {
  fn GoClass(&self){
    println!("this is Bob!");
  }
  
  fn LeaveClass(&self) {
    // ...
  }
  
  fn getClassRoom(&self) {
    // ...
  }
}
```

* 调用Bob.GoClass时，会打印`this is Bob!`而不是`this is student!`，因为Bob类型中已经覆盖了`GoClass`之前的函数定义。



#### 使用特征来作为函数参数

* 用特征来作为函数参数是非常重要的一点，**相对类型的参数限制，特征的参数限制则显得更加细粒度**，只要让传递的参数满足对应的特征即可，而不需要把所有类型都列举出来。

```rust
pub fn enterClassRoom(people: &impl Student) {
  // TODO
}
```

* 上述代码中展示了使用如何使用特征作为函数的参数，`impl Student`便是实现语法，它本身是语法糖，正式的书写方式如下所示。

```rust
pub fn enterClassRoom<T: Student>(people: &T) {
  
}
```

* 其实不难理解，把特征作为函数参数，其实就是用**特征来约束范型**，这也叫做**特征约束**。



#### 多重特征约束

* 上述展示的代码是单个特征约束，除了单特征约束，还可以实现**多特征约束**，例如上述例子中，除了使参数满足`Student`约束外，还可以使其实现`Monitor`约束，如下所示。

```rust
// 语法糖
pub fn enterAdminRoom(people: &(impl Student + Monitor)){
  
}

// 特征约束式
pub fn enterAdminRoom<T: Student + Monitor>(prople: &T){
  
}
```

##### where关键字

* 当特征比较多时，用上述的表达方式会显得代码冗杂不易读，此时便可以使用`where`关键字来实现形式上的改进，如下所示。

```rust
pub fn someFunction<T, U>(t: &T, u: &U) -> i32
	where T: Display + Clone,
				U: Clone + Debug
{
  
}
```

* 上述代码使得参数`t`必须满足`Display`和`Clone`特征，参数U必须满足`Clone`和`Debug`特征。



#### 使用特征限制返回值

* 特征除了能够限制参数外，还可以限制返回值，其关键字用法也是`impl Trait`，和**特征传参的语法糖形式相同。**

```rust
fn returns_summarizable() -> impl Summary {
    Weibo {
        username: String::from("sunface"),
        content: String::from(
            "m1 max太厉害了，电脑再也不会卡",
        )
    }
}
```

* 实现了`Summary`特征的类型才能返回，这样做的好处是：如果返回的真实类型非常复杂时，我们需要把所有类型都要进行声明，这样也会增加代码复杂度，那么便可以使用`impl Trait`的方式要求返回值，**只要让待返回数据的类型满足指定的特征就能返回。**
* 例如使用`impl Iterator`来告诉调用者，该函数将会返回一个迭代器。
* 但是这种形式的返回有个限制：**只能返回一个具体的类型**。假如函数内部可能会返回两个及以上的类型，那么编译器会报错，如下所示。

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        Post {
            title: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Weibo {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
        }
    }
}
```

* 上述代码中，可能返回`Post`类型，也可能返回`Weibo`类型，虽然它们都有`Summary`特征，但是编译依然会报错，如果要实现返回不同类型，则需要**特征对象**来实现，关于特征对象将会在下一章中介绍。

#### derive派生特征

* 在开发过程中，经常会遇到`#[derive(xxxx)]`类似的标注，其被称为**特征派生语法**，被 `derive` 标记的对象会自动实现对应的**默认特征代码，继承相应的功能。**

```rust
#[derive(Debug)]
struct Point{
    x: i32,
    y: i32
}
fn main() {
    let p = Point{x:3,y:3};
    println!("{:?}",p);
}
```

* 上述代码中，为`Point`类型派生了`Debug`特征，使得其类型具有了**格式化输出的功能**，即能够直接通过`println!`来打印类型。

---
title: go-learning-1
date: 2022-01-28 09:13:38
tags: ["golang"]
---

> Golang学习记录

## 关于swtich

* Golang中的switch的每个case自带break关键字，即不用手动去设置break关键字。

```go
import (
	"fmt"
)

func main(){
  variables := 12
  switch variables {
    case 24:
    fmt.Println('24')
    case 12:
    fmt.Println('12')
 	 	case 36:
    fmt.Println('36')
    default:
    fmt.Println('114514')
  }
}

// 12
```

* 如果在其他语言中这样编写switch代码块，36和default代码块中的输出也会执行。

### fallthrough关键字

* 当然，golang也提供了能够**无视掉默认break的关键字**，即fallthrough，在指定的case最后一行加上fallthrough，则对应的case代码块将会忽视掉默认的break操作。

```go
// 接上
func main(){
  variables := 12
  switch variables {
    case 24:
    fmt.Println('24')
    case 12:
    fmt.Println('12')
    fallthrough
 	 	case 36:
    fmt.Println('36')
    default:
    fmt.Println('114514')
  }
}

// 12
// 36
```



## 关于数组

* 在golang中的数组和c一样，一旦定义了大小就不可更改，且声明方式有多种，如下:

```go
// var variable_name [SIZE] variable_type
func main(){
  var balance = [3]float32{100, 2.21, 4.12} // {}中的数字个数不能大于[]中定义的数组大小
  var c = [5]int{1,2,3}
  d := [...] int{1,2,3,4} // 不显示设置数组大小，而是根据{}中定义的数字个数来确定
  e := [...] int{4: 100}// [0, 0, 0, 0, 100] -> 这w里的4: 100 表示数组中第五号元素为100，其余位置补0
  f := [...] int{1: 12, 4: 12, 9: 12} // [0 12 0 0 12 0 0 0 0 12] -> 也可以指定多个下标元素
}

```

* 和js不一样的是，如果需要获得数组的长度，需要调用len方法，并把对应数组当作参数传入。

```go
len(array) // array的长度
```

### 值类型

* 在golang中，数组并不是引用类型的变量，**而是值类型的变量**，即给新变量赋值数组时，新变量接收到的不是指针，而是数组本身的副本copy，新数组和旧数组内部元素的更改不会互相影响。

```go
package main

import "fmt"

func main() {  
    a := [...]string{"USA", "China", "India", "Germany", "France"}
    b := a // a copy of a is assigned to b
    b[0] = "Singapore"
    fmt.Println("a is ", a) 
    fmt.Println("b is ", b) 
}
// a is  [USA China India Germany France]
// b is  [Singapore China India Germany France]
// 可以看到a数组内部的元素并没有同步更新
```



## Slice 切片

* Slice是对Go语言中数组的抽象。由于数组不可改变，特定场景中这样的集合变量就不太实用，于是slice便出现了，它也可以理解为“动态数组”可以追加元素。
* 需要注意的是，**Slice只是对现有数组的引用，它本身没有任何数据，Slice上的改变都会影响到底层数组内值的更改。**
* 从概念上来说，Slice就像一个结构体，它包含三个元素:

1. 指针，指向数组中slice指定的开始位置（本质是对数组的引用）
2. 长度，slice的长度。
3. 最大长度，slice的开始位置到**数组的最后位置的长度**



### Slice定义语法

```go
// 声明语法
var identifier []type
```

* 切片不需要说明长度，也可以使用make函数来进行切片创建。

```go
var slice []type = make([]type, len)
slice := make([]type, len)
// make创建的范式如下
make([]T, length, capacity)
```

* 如果要进行初始化，可以如下

```go
s := []int {1, 2, 3} // 注意, []中没有三个点，如果有三个点，表明是数组的初始化。
s := array[startIndex: endIndex] // 通过数组的切片来获得，此时s表示数组array的指针。

```



### 修改切片

* 由于silce只是底层数组的引用，所以slice上的修改会**同步反映到底层数组中**。

```go
func main() {  
    darr := [...]int{57, 89, 90, 82, 100, 78, 67, 69, 59}
    dslice := darr[2:5]
    fmt.Println("array before",darr)
    for i := range dslice {
        dslice[i]++
    }
    fmt.Println("array after",darr) 
}

// array before [57 89 90 82 100 78 67 69 59]  
// array after [57 89 91 83 101 78 67 69 59]
// 可以看到，第三个元素到第五个元素都自增了一次
```

### len() 和 cap()

* len方法和array的用法一样，用来获得切片此时的长度，而cap方法则是获取切片的最大长度。

```go
func main() {
	a := []string{"USA", "China", "India", "Germany", "France"}
	d := a[1:2]
	fmt.Println(len(d), cap(d))
}

// 1, 4 
// cap(d) = 4 -> 从切片开始的位置（第二个元素）到数组最后一个元素位置有4个
```

### append() 和 copy()

* append是在切片后增加元素，copy则是获取切片的拷贝（注意，这里的拷贝是深拷贝，即底层的数组也会跟着一起拷贝，两个切片之间不会建立联系）
* append增加元素也会影响到底层数组，**但如果增加元素后，切片已经超过最大长度，则golang会重新在底层创建一个适合于新切片的最大长度的数组，并将对应的slice指向新的数组上**，此时便不会影响到旧的底层数组，示例代码如下

```go
package main

import "fmt"

func main() {
	a := []string{"USA", "China", "India", "Germany", "France"}
	b := a[1:3]
	b = append(b, "Jap")
	d := a[1:5]
	d = append(d, "smart")
	fmt.Println("切片a: ", a)
	fmt.Println("切片b: ", b)
	fmt.Println("切片d: ", d)
}
/*
切片a:  [USA China India Jap France]
切片b:  [China India Jap]
切片d:  [China India Jap France smart]
可以看到，切片b的增加影响到了切片a，但是切片d的增加却对a没有任何影响。
*/
```





> 待解决问题:  interface变量是啥

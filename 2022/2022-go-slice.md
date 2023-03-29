

## 什么是切片(slice)

slice和数组（array）很类似，可以用下标的方式进行访问，如果越界，就会产生panic，但是它比数组更加的灵活，可以自动的进行扩容



## 切片本质

切片是由指针，长度，容量组成，切片并不是数组或者数组指针，它是通过内部指针和相关属性引用数组片段，来实现变长的方案

`指针`：指向底层数组

`长度`：表示切片可用元素的个数，也就是会用下标对slice进行访问时候，下标不能超过的长度 

`容量`:   底层数组的元素个数，容量>=长度，在底层数组不进行扩容的情况下，容量也是slice可以扩张的最大限度

## 切片特点

一个slice 是一个轻量级的数据结构，提供了访问数组子序列元素的功能

底层引用了一个数组对象，指针指向第一个slice元素对象的底层数组元素的地址 

`注意`：底层数组是可以被多个 slice 同时指向的，因此对一个 slice 的元素进行操作是有可能影响到其他 slice ,

**两个slice不能用==比较**

## 切片和数组的区别

slice切片底层是数组，slice是对数组的封装，它描述了一个数组的片段，俩者都可以用下标来访问元素

数组是固定长度的，长度定义好后，不能更改 

切片非常灵活，它可以动态扩容，切片的类型和长度无关



## 切片的创建

- 直接声明： var slice []int 

- 字面量： slice1 := []int{1,2,3}

- make:     slice1 :=make([]int,3,5)

- new: 	  slice1 := *new([]int)

- 切片或者数组截取： slice1 := array1[1:4] or slice1 := slice2[1:3]

 

###  直接声明

第一种直接声明创建的slice 是nil slice ，它的长度和容量都为0，和nil 比较的结果为true

```go
package  main

import "fmt"

func main() {
	var s1 []int
	fmt.Println(s1 == nil)
}
//result
true

```

**空切片**

```go
silce := make( []int , 0 )
slice := []int{ }
```

`注意`：空切片和 nil 切片的区别在于，空切片指向的地址不是nil，指向的是一个内存地址，但是它没有分配任何内存空间，即底层元素包含0个元素

### 字面量

```go
s2 := []int{1,2,3,5:10}
s3 := []int{1, 2, 3, 4, 5, 6}
s4 := []int{}  //创建空切片
s5 := []string{99: 100}   //初始化第100个元素

fmt.Println(s2,len(s2),cap(s2))
//
[1 2 3 0 0 10] 6 6 //s2
//唯一值得注意的是上面的代码例子中使用了索引号,直接赋值 ,这样其他未注明的元素则默认 0 值
```

### make

`make`函数需要传入三个参数：切片类型，长度，容量。当然，容量可以不传，默认和长度相等

如果使用字面量的方式创建切片，大部分的工作就都会在编译期间完成，但是当我们使用 `make` 关键字

创建切片时，很多工作都需要运行时的参与；调用方必须在 `make` 函数中传入一个切片的大小以及可选的容量

```go
package main

import "fmt"

func main() {
	slice := make([]int, 5, 10) // 长度为5，容量为10
	slice[2] = 2 // 索引为2的元素赋值为2
	fmt.Println(slice)
}
```

数组切片和切片的切片

```go
var array = [10]int{1, 2, 3, 4, 5, 6} //定义一个数组
var s6  = array[1:4] //[2,3,4] 左闭右开
var s7  = array[4:] //[5,6,0,0,0,0] 
var s8 = array[2:4:6] //data[low, high, max] low表示索引开始处闭区间，high表示len开区间，max表示容量开区间 结果分析 [3,4] -> len=2,cap=4 

slice := []int{1, 2, 3, 4, 5, 6} //定义一个切片
s10 := slice[:4]  //beginIndex如果为空则表示从0开始
s11 := slice[4:]  //endIndex如果为空则表示到数组最后一个元素
var 12 = slice[2:4:6] //data[low, high, max] low表示索引开始处闭区间，high表示len开区间，max表示容量开区间 
```



## append追加元素

append会返回新的slice，append返回值必须使用否则编译器会报错

```go
slice := append(slice, elem1, elem2)    //可以传入多个元素
slice := append(slice, slice_other...)  //可以传入一个切片 切片后面要加三个点 ...
```

## 复制切片

```go
package main

import "fmt"

func main() {
	slice1 := []string{"a","n"}
	out := slice1[:]
	out1 := slice1
	fmt.Printf("out=%v,p=%p\n",out,&out)
	fmt.Printf("out1=%v,p=%p",out1,&out1)
}
//output
out=[a n],p=0xc00000c0a0
out1=[a n],p=0xc00000c0c0

```

## copy切片

```go
由于 Value 是值拷贝的，并非引用传递，所以直接改 Value 是达不到更改原切片值的目的的
func main() {
	array := []int{10, 20, 30, 40}
	slice := make([]int, 6)
	n := copy(slice, array)
	fmt.Println(n,slice)
}
```



## for- range

for循环会对slice元素值一次拷贝到item。更改item中的值不会改变原slice的元素值

```go
slice := []int{1,2,3}
for _, item := range slice {
    item++
}
fmt.Println(slice)
//output: [1,2,3]
```

## 函数传参

函数传slice是引用传参，修改被调函数的值，调用函数的slice也会改变。

```go
func main()  {
    slice := []int{1,2,3}
    test(slice)
    fmt.Println(slice)
}
func test(a []int) {
    a[1] = 100
}
//output [1,100,3]
```

## 切片坑和困惑

- 切片做函数参数是传引用
- append扩容问题，append 函数会创建一个新的底层数组,拷贝已存在的值和将要被附加的新值

```go
package main
import "fmt"
func s(s []string) {    //切片是引用传参
	s[0] = "bds:234"
}
func main() {
	s1 :=[]string{"123"}
	s(s1)
	fmt.Println(s1)
} 
//output
[0 1 1]
[bds:234]
/*
切片做函数参数的时候，是使用传引用（也就是传地址）
相当于是指针指向的内存地址这个引用，由于指向的是同一块内存地址，
所以在函数内部通过s[0] = "bds:234" 修改切片，最后修改成功
*/
```



```go
package main
import "fmt"
func myAppend(s []int) []int {
   y := s[:1]
   for _,v := range s  {
      y = append(y,v)
   }
   return y
}
func main() {
   s := []int{1,2,3,4,5}
   newS := myAppend(s)
   fmt.Println(s)
   fmt.Println(newS)

}
//output: 
[1 1 1 1 1]
[1 1 1 1 1 1]

```



```go
package main

import "fmt"

func Add2Slice(s []int, t int) []int{
	s[0]++
	s1 := append(s, t)
	//fmt.Println("s1",s1)
	s[0]++
	return s1
}
func main() {
	a := []int{0, 1, 2, 3}
	c := Add2Slice(a, 4)
	fmt.Println(c)
	fmt.Println("a", a)
	b := Add2Slice(a, 5)
	fmt.Println(b)

	d := []int{1}
	nd := append(d, 3)
	fmt.Printf("d=%v,P = %p\n", d, &d)
	fmt.Printf("nd=%v,P = %p ", nd, &nd)

}
//output
[1 1 2 3 4]
a [2 1 2 3]
[3 1 2 3 5]
d=[1],P = 0xc0000a6080
nd=[1 3],P = 0xc0000a60a0 

```



### 群里热心大佬分享一个考题

![17631606914834_.pic_hd](https://gitee.com/budongshu/blogimg/raw/master/image/17631606914834_.pic_hd.jpg)

```go
我的思路：
s2 = s1 此时 是相同的内存地址 相当于复制拷贝一份

append 操作了 s2 按照go的扩容规则，内存地址改变，指针指向随之发生改变 
进入函数s  = append(s,0) 操作s1 时候， s1被扩容，地址发生改变，所以后面s[i]++操作的是扩容后新地址切片 所以s1 还是 12 

slice 形参是传引用 相当于指针变量进行复制一份，但是指针指向的内存地址是相同的 ，所以后面操作s[0]++ 相当于通过修改了内存地址里面的变量值，所以会s[i]++生效

进入函数s  = append(s,0) 操作s2的时候，根据扩容规则，容量满足，地址没有发生改变，所有后面操作的是原地址切片，值s[i]++ s2 变成234

```




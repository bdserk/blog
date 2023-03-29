## 1 函数定义

函数是组织好的，可重复使用的，用于执行指定任务的代码块，本文介绍go语言函数的相关内容

Go 语言支持： 函数 ，匿名函数，闭包

Go 语言中定义函数使用func 关键字 

```go
func 函数名(函数参数）函数返回值 {
	函数体
}
```

- 函数名： 由 字母 数字 下划线组成 但是函数名的第一个字母不能是数字，在同一个包内，函数名称，不能重名
- 函数参数：参数由参数变量和参数变量的类型组成，多个参数可以使用逗号分隔
- 函数返回值：返回值由返回值变量和其变量类型组成，也可以只写返回值，多个返回值必须用()包裹，并且用逗号，分隔
- 函数体： 实现指功能的代码块

## 2 函数使用案例

```go
package main
import (
	"fmt"
)
func sumFunc(x int ,y int)   int {
	sum := x + y
	return sum
}
func subFunc(x,y int ) int {      //当数据类型是相同的情况下， 可以最后只写一个
	sub := x - y
	return sub
}


func main() {
	sum1 := sumFunc(1, 3)
	fmt.Println(sum1)

	sub1 := subFunc(3,1)
	fmt.Println(sub1)
}
```

## 3 函数可变参数

通常可变参数要写在最后一个参数,Go中使用在参数后面加... 来标识

```go 
func sumFunc1(x ...int) {         //可变参数： 可以用三个点 ... 来接收多个值
	fmt.Printf("%v,%T\n",x,x)
}
func sumFunc2(x ...int) int {     //可变参数： 可以用三个点 ... 来接收多个值
	sum := 0
	for _,v := range x {
		sum += v
	}
	return  sum
}
func calc1(x int,y ...int )  int  {   //传递可变长参数，可变长参数放在关键词后面
	sum := 0
	for _,v := range y {
		sum += v
	}
	return sum

}
	sumFunc1(1,3,35,6)
	sum2 := sumFunc2(1,100,100,99)
	fmt.Println(sum2)
  sumCalc1 := calc1(3,2,100,100,88)
	fmt.Println(sumCalc1)

```

## 4 函数多个返回值

```GO
package main
import "fmt"

func calc(x, y int)  (int ,int ) {
	sum := x + y
	sub := x - y
	return sum,sub
}

func calc1(x ,y int ) (sum int,sub int) {
	sum = x + y
	sub = x - y
	return sum ,sub

}
func main() {
	sumRes,subRes := calc(3,1)
	fmt.Println(sumRes)
	fmt.Println(subRes)
	sum1,sub1 := calc1(10,3)
	fmt.Println(sum1)
	fmt.Println(sub1)
 
}
```

## 5 函数的参数和返回值是可选的

函数的参数和返回值都是可选的。下面我们实现一个参数和返回值都没有的函数

```go
func say() {
	fmt.Println("test")
}
```

## 6 把排序封装成方法

>  实现整形的升序和降序顺序排列,string类型按照字符的首字母进行排序,首字母相同，继续往后一次对比

### 6.1 int类型升序

```go
package main

import "fmt"

func sortIntAsc(slice []int) []int {
	for i :=0;i< len(slice);i++ {
		for j := i +1 ;j<len(slice) ;j++ {
			if slice[i] > slice[j] {
				tmp := slice[i]
				slice[i] = slice[j]
				slice[j] = tmp
			}
		}
	}
	return slice
}
func main() {
	slice1 := []int{12,23,34,1,25,100,444,2}
	sortSlice := sortIntAsc(slice1)
	fmt.Println(sortSlice)
}
```

### 6.2 int类型降序

```GO
func sortIntDes(slice []int) []int {
	for i :=0;i< len(slice);i++ {
		for j := i +1 ;j<len(slice) ;j++ {
			if slice[i] < slice[j] {
				tmp := slice[i]
				slice[i] = slice[j]
				slice[j] = tmp
			}
		}
	}
	return slice
}
```

### 6.3 map对象 按照key进行排序

```go
package main

import (
	"fmt"
	"sort"
)

func mapSort(m map[string]string) string {
	var sliceKey   []string

	for k,_ := range m {
		sliceKey = append(sliceKey, k)

	}
	sort.Strings(sliceKey)
	var str string
	for _,v := range sliceKey {
		str += fmt.Sprintf("%v=>%v |" ,v,m[v])
	}
	return str

}
func main() {
	var map1 = map[string]string {
		"username": "bds",
		"age": "19",
		"worker": "linux",
		"sex": "男",
		"aaa": "aaa",
	}
	str1 := mapSort(map1)
	fmt.Println(str1)
}

//age=>19 |sex=>男 |username=>bds |worker=>linux |

```



## 6 函数变量作用域

全局变量: 全局变量是定义在函数外部的变量，他在程序整个运行周期内都有效

局部变量： 局部变量是定义在函数内部的变量， 函数定义的变量无法在函数外使用

```go
package main

import "fmt"

var a  = "全局变量"
func run() {
	var b = "局部变量"
	fmt.Println("run方法a: ",a)
	fmt.Println("run方法b: ",b)
}
func main() {
	run()
	fmt.Println("main方法a: ",a)
	// fmt.Println("main方法b: ",b)     //undefine：b 报错

	//i 是局部变量，只能在for方法内中使用
	for i := 0;i < 10 ;i++ {
		fmt.Println(i)
	}
	//本作用域中相当于全局变量
	var flag = true
	if flag {
		fmt.Println("true")
	}
	// 局部作用域 ，局部变量
	if flag := true ;flag {
		fmt.Println("true")
	}
	fmt.Println(flag) //undefine: flag
}
```

## 7 自定义函数类型

我们可以使用type关键字来定一个函数类型，具体格式如下： 

```go
type calc func(int,init) int 
```

上面语句定义了一个calc 类型，他是一个函数类型， 这种函数接收俩个int类型的参数并且返回一个int类型的返回值

简单来说，凡是满足这个条件的函数都是calc类型的函数，如果是不满足条件强行赋值是会报错的

```go
package main

import "fmt"
//自定义方法类型
type calc func(int,int) int
//自定义整形类型
type myInt int

func add(x,y int) int {
	return x + y
}
func sub(x,y int) int {
	return x - y
}
func test()  {
	fmt.Println("test")
}

func main() {
	var c calc
	c = add
	fmt.Printf("c的类型: %T\n",c )  //c的类型: main.calc   这里是我们自定义类型
	f := sub
	fmt.Printf("f的类型：%T\n",f)   //f的类型：func(int, int) int  这里是类型推导
	var a = 10
	var b myInt = 10
	fmt.Printf("%v,%T\n",a,a)   //10,int
	fmt.Printf("%v,%T\n",b,b)   //10,main.myInt
	//因为b重新定义了myInt，myInt虽然是始于int定义的，但是还是属于俩个类型，不能相加
	//fmt.Println(a+b)  //invalid operation: a + b (mismatched types int and myInt)
 	fmt.Println(a+int(b))         //20

}
```

## 8 函数作为另一个函数的参数

8.1 函数作为参数

```go
package main

import (
	"fmt"

)

type calcType  func(int,int) int


func add(x,y int) int {
	return x + y
}
func sub(x,y int ) int {
	return x - y
}

func do(a,b int,cb calcType) int {
  return cb(a,b)
}

func main() {
	var c = do
	doValue :=	c(1,3,add)
	fmt.Println(doValue)
  //用匿名函数直接作为参数进行传参
	cValue := c(1,3,func(x,y int) int {
		return x * y
	})  
}
```

8.2 函数作为返回值

```go
package main

import "fmt"

type calcType func(int,int) int
func add(x,y int) int {
	return x + y
}
func sub(x,y int) int {
	return x - y
}
func do(cmd string) calcType {
	switch cmd {
	case "+":
		return add
	case "-":
		return sub

	case "*":
		return func(x,y int ) int {
			return x * y
		}
	default:
		return nil
	}

}
func main() {
	var c = do("*")
	fmt.Println(c(2,3))

}
```



## 9 匿名函数和闭包

函数当然还可以作为返回值，但是在Go语言中函数内部不能在像之前那样定义函数了，只能是定义匿名函数，匿名函数就没有函数名字的函数，匿名函数的定义格式如下： 

```go
func(参数)(返回值) {
  函数体
}
```

匿名函数因为没有函数名字，所以没办法像普通函数那样被调用，所以匿名函数需要保存到某个变量或者作为立即执行函数

```go
package main

import "fmt"

func main() {
	//通过变量保存匿名函数
	var add = func(x,y int) int {
		return x + y
	}
	fmt.Println(add(10,20))
	//自执行函数，匿名函数定义后面加（）直接执行
	func(x,y int)  {
		fmt.Println(x * y)
	}(10,20)
	
}
```

闭包

闭包可以理解成 定义在一个函数内部的函数，在本质上，闭包是将函数内部和函数外部连接起来的桥梁，或说是函数和其引用环境的组合体，首先我们先看一个例子

- 闭包就是指有权访问另外一个函数作用域中的变量的函数
- 创建闭包的常见方式就是在一个函数内部创建另外一个函数，通过另外一个函数访问这个函数的局部变量

注意： 由于闭包里作用域返回的局部变量资源不会被立刻销毁回收，所以可能会占用更多的内存，过度使用闭包会导致性能下降。

全局变量 

- 常驻内存
- 污染全局

局部变量的特点

- 非偿住内存
- 不污染全局

闭包：

- 可以让一个变量常驻内存
- 可以让一个变量不污染全局



```go
package main

import "fmt"

func addr(y int) (s int)  {
	var i = 10  //常驻内存，不污染全局
	var fn1  = func(y int) int {
		i += y
		return i
	}(y)
	s = fn1
	return s

}
func main() {
	var fn = addr(3)
	fmt.Println(fn)
	fmt.Println(fn)
	fmt.Println(fn)

}
/*
13
13
13
*/
```


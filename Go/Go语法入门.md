# Go入门

## 安装

直接下载go 安装

![image-20210416225001989](assets/image-20210416225001989.png)

## 安装Goland

直接安装Goland  IDE

## 创建项目

创建项目被设置GOPROXY

![image-20210416225103570](assets/image-20210416225103570.png)



## 创建GO文件

**每一个项目必须有一个main.go 文件 以及main方法 pageage 必须为main**

![image-20210416225203293](assets/image-20210416225203293.png)



## 声明变量

### 显示声明

```go
var a string = ""
```

### 隐式声明

```go
b:="" // 常用
```

![image-20210416230229148](assets/image-20210416230229148.png)

## 注释

```go
// 单行注释
/*范围注释*/
```

## 包

**一个文件夹下不能出现多个包，比如一个文件夹下有了main包，那么不能在创建其他名字的包，必须在其他文件夹下创建**

但是一个包中可以有多个文件

![image-20210416230541190](assets/image-20210416230541190.png)

### 包别名

如果引入的包名过长，可以在import中重新起一个别用，使用时通过 **别名.变量名** 的方式调用

![image-20210416230659947](assets/image-20210416230659947.png)

### 直接引入

在导入的包前使用 .  就直接引入当前的文件中

![image-20210416230824316](assets/image-20210416230824316.png)

## 外部引用

如果包中的变量名和方法名开头是小写，说明是私有的，不能被其他包引用。

**如果需要其他包引用，那么必须为大写**



## 基本类型

### string

```go
var name string = "hening"
```

### 整形

Int:所有的整数( 分为 int8、int16、int32、int64 取值范围不同)

uint：正整数(如果给一个负数，会报错) 同样分为uint8、uint16、uint32、uint64 取值范围不同

```go
var num1 int = 123
var num2 uint = 123
```

### 浮点数

float64

Float32

```go
var num2 float64 = 3.14
```

### 布尔

```go
var bool1 bool = true
```



![image-20210417001011506](assets/image-20210417001011506.png)

### 获取参数类型

```go
fmt.Printf("%T",a)
```



### 参数类型转换

使用strconv 包进行转换，可以返回多个值并接受（java是单值返回），可以接受错误err

```go
// string - int
int1,_:=strconv.Atoi(name)
// string - int64
strconv.ParseInt()
// string - bool
strconv.ParseBool()
```



## 跳转语句

break 跳出循环

continue：跳出本次循环

goto  调往另一个结构体

```go
package main
import "fmt"
func main(){
	a:=0
	A:
		for a<10 {
			a++
			fmt.Println(a)
			if a == 5 {
				break A // 跳出结构体A
				goto B  // 转到B结构体
			}
		}
	B:
		fmt.Println("rush to B")
}
```

![image-20210417183146214](assets/image-20210417183146214.png)



## 数组

格式：a:= [5]int{1,2,3,4,5} //直接赋值

​			var c = new([5]int)  // 不赋值

```go
package main

import "fmt"

func main(){
	// [元素个数]元素类型{元素值}
	a:= [5]int{1,2,3,4,5}
	b:=[...]int{6,7,8,9,10,11,12,13}
	var c = new([5]int)
	c[2] = 2
	fmt.Println(a,b,c)
}
```

### 循环数组

两种循环方式

使用for循环

使用range 获取每个元素的下标和值 

Len() ：获取数组长度

cap(): 获取数组容量

```go
package main

import "fmt"

func main(){
	zoom := [...]string{"猴子","大象","猩猩"}
	for i:=0;i<len(zoom);i++ {
		fmt.Println(zoom[i])
	}

	for i,v := range zoom {
		fmt.Println(i,v)
	}
}
```



### 二维数组

```go
// 二维数组
	er := [3][3]int {
		{1,2,3},
		{4,5,6},
		{7,8,9},
	}
	fmt.Println(er)
```

## 切片

切片是数组的一部分，切片内的值被改变，那么数组中的值也会被改变

使用： 对数组进行切片，获取到的切片打印出来跟数组一样

```go
package main

import "fmt"

func main() {
	a := [3]int{1, 2, 3}
	c := a[:]
	fmt.Println(a, c)
	// 获取切片，前闭后开原则
	d := a[0:2]
	fmt.Println(d)
  // 从1 开始取到最后
	e := a[1:]
	fmt.Println(e)s
}
```

![image-20210417185955182](assets/image-20210417185955182.png)

### 切片扩容

append(切片，元素值) 方法

**使用append 后，切片与原数组没有关系，不会修改原数组的值**

```go
package main

import "fmt"

func main() {
   a := [3]int{1, 2, 3}
   c := a[:]
   fmt.Println(a, c)
   c = append(c, 4)
   fmt.Println(a,c)
   // 使用append 后，切片与原数组没有关系，不会修改原数组的值
   c[0] = 0
   fmt.Println(a,c)
}
```

### 切片长度、容量

len() : 获取长度

cap() : 获取容量

**当切片的长度等于容量后，再次append后，容量会扩容为原来的两倍**

![image-20210417190726168](assets/image-20210417190726168.png)

### 切片复制 copy 

当后一个切片的值，覆盖到前一个切片中，可以指定

```go
package main
import "fmt"
func main() {
	a := [3]int{1, 2, 3}
	c := a[:]
	d := a[2:]
	fmt.Println(a,c,d)
	c = append(c, 4)
  // 可以指定从c的那个位置开始覆盖 c[1:2],默认从首位开始覆盖 
	copy(c,d)
	fmt.Println(a,c,d)
}
```

![image-20210417191056122](assets/image-20210417191056122.png)

### 创建切片的方式

1、 从数组中获取切片

2、直接声明一个切片

3、make方法

```go
package main
import "fmt"
func main() {
	// 创建切片
	var a []int
	a = append(a, 1)
	fmt.Println(a)
	// 初始化切片，指定长度和容量
	var b = make([]int,3,10)
	fmt.Println(b)
	fmt.Println(len(b),cap(b))
}
```

![image-20210417191640586](assets/image-20210417191640586.png)

## map

### 创建map

三种方式

var m map[string]string

m1 := map[string]string{}

m2 := make(map[string]string)

```go
package main

import "fmt"

func main() {
	// 创建map
	var m map[string]string
	m = map[string]string{}
	m["name"] = "hening"
	m["sex"] = "man"
	fmt.Println(m)

	m1 := map[string]string{}
	m1["name"] = "duhan"
	m1["sex"] = "woman"
	fmt.Println(m1)

	m2 := make(map[string]string)
	m2["name"] = "hening2"
	m2["sex"] = "man"
	fmt.Println(m2)
}
```

### 存储不同类型value

m := map[string]interface{}{}  : **interface{} 表示value接受不同类型**

```go
package main

import "fmt"

func main() {
	// 不同类型value
	m := map[string]interface{}{}
	m["hening"] = "my name is hening"
	m["work"] = true
	m["age"] = 22
	fmt.Println(m)
}
```

### map删除

```go
delete(m,"work")
fmt.Println(m,len(m))
```

### map循环

使用range 循环，获取k，v

```go
// 不同类型value
	m := map[string]interface{}{}
	m["hening"] = "my name is hening"
	m["work"] = true
	m["age"] = 22
	for k,v := range m {
		fmt.Println(k,v)
	}
```

![image-20210417215045980](assets/image-20210417215045980.png)

## func 函数方法

func声明方法

func 方法名(参数名1 参数类型1，参数名2 参数类型2)(出参1 出参类型1，....) {

​	// 处理代码逻辑

}

```go
package main

import "fmt"

func main() {
	// func 函数
	r1,r2 := test(22,"年龄")
	fmt.Println("main",r1)
	fmt.Println("main",r2)
}

func test(data1 int,data2 string)(res1 int,res2 string){
	fmt.Println(data1)
	fmt.Println(data2)
	return data1,data2
}
```

### 匿名函数

实际上相当于在main函数外新建一个函数，**然后将这个函数赋值给了b，然后通过b来调用

```go
b := func(data string) {
		fmt.Println(data)
	}
	b("我是一个匿名函数")
```

### 不定项函数

指入参、出参数量补丁的参数

```go
func main() {
	more(9527,"1","2","3","4")
}

func more(num int,data ...string)  {
	fmt.Println(num)
	fmt.Println(data)
	for k,v := range data {
		fmt.Println(k,v)
	}
}
```

![image-20210417220555209](assets/image-20210417220555209.png)

### 参数不定项参数

如果希望将数组传递为不定项参数，那么传入数组时，在后面加三个点

**arr...** 

```go
func main() {
	more(9527,"1","2","3","4")
	arr := []string{"5","6","7","8"}
  // 传入数组
	more(9528,arr...)
}

func more(num int,data ...string)  {
	fmt.Println(num)
	fmt.Println(data)
	for k,v := range data {
		fmt.Println(k,v)
	}
}
```

### 自执行函数

在函数内定义的函数，不需要调用，再声明的时候就被调用

```go
func main() {
	(func() {
		fmt.Println("我是自执行函数")
	})()
}
```

### 闭包函数

指函数返回的类型还是一个函数，同样可以指定入参和出参

**注意函数的出参要和返回值得函数一致，不一致会报错**

```go
func main() {
	more()(24)
}
// more 函数出参也是一个函数
func more()func(num int)  {
	return func(num int) {
		fmt.Println("我是一个闭包函数",num)
	}
}
```

![image-20210417221422502](assets/image-20210417221422502.png)

## 延迟执行

defer 关键字

```go
func main() {
	// 最先声明defer的最后执行
	defer more()(24)
	defer more()(25)
	fmt.Println(1)
	fmt.Println(2)
	fmt.Println(3)
}

func more()func(num int)  {
	return func(num int) {
		fmt.Println("我是一个闭包函数",num)
	}
}
```

![image-20210417221820066](assets/image-20210417221820066.png)

## 指针和地址

&：使用 &符号取出变量的内存地址

使用 * 取出指针变量的值

```go
func main() {
	// 指针的使用
	var a int
	a = 123
	var b *int
	b = &a
	fmt.Println(a,b)
}
```

![image-20210418174902477](assets/image-20210418174902477.png)

因为使用指针，指向的是内存地址，那么修改一个指针的值，和他指向同一个内存地址的其他变量的值也会发生改变

```go
func main() {
	// 指针的使用
	var a int
	a = 123
	var b *int
	b = &a
	// 改变指针指向内存地址的值
	*b = 321
	fmt.Println(a,*b)
}
```

 ### 数组指针&指针数组

数组指针

```go
func main() {
   // 数组指针
   var arr [5]string
   arr = [5]string{"1","2","3","4","5"}
   fmt.Println(arr)
   // arrP一个数组的指针
   var arrP  *[5]string
   arrP = &arr
   fmt.Println(arr,arrP)
}
```

指针数组，指数组中存的是指针

```go
func main() {
	// 指针数组，数组中存了五个string的指针
	var arr [5]*string
	str1 := "1"
	str2 := "2"
	str3 := "3"
	str4 := "4"
	str5 := "5"
  // 使用& 表示指针
	arr = [5]*string{&str1,&str2,&str3,&str4,&str5}
	fmt.Println(arr)
	// 改变指针指向内存的值，同时str3也改变
	*arr[2] = "123"
	fmt.Println(str3)
}
```

![image-20210418180046102](assets/image-20210418180046102.png)

### func函数指针传参

如果在func函数内，传入一个指针，在func内进行修改了值，那么函数外的变量也会被修改



### map指针

Key 和 value 都可以用指针标识

```go
func main() {
	// map 指针
	var m map[*string]string
	m = map[*string]string{}
	str := "hening"
	m[&str] = str
	fmt.Println(m)
	fmt.Println(m[&str])
}
func main() {
	// map 指针
	var m map[string]*string
	m = map[string]*string{}
	str := "hening"
	m["name"] = &str
	fmt.Println(m)
	fmt.Println(*m["name"])
}
```

![image-20210418180645265](assets/image-20210418180645265.png)

![image-20210418180802987](assets/image-20210418180802987.png)

## 结构体Struct

包含多种数据结构的特殊数据结构，**可以类比为java中的对象**

声明一个结构体

**type 名称 struct {**

**}**

注意，如果是直接使用new(Type) 初始化的一个结构体，那么得到的是一个指针

```go
package main

import "fmt"

type Hening struct {
	Name string
	Age int
	Sex bool
	Hobit []string
}

func main() {
	// 声明结构体
	var hening Hening
	hening.Name = "hening"
	hening.Age = 24
	hening.Sex =true
	hening.Hobit = []string{"睡觉","游戏"}
	fmt.Println(hening)

	// 隐式声明
	duhan := Hening{
		Name:  "duhan",
		Age:   23,
		Sex:   false,
		Hobit: []string{"睡觉"},
	}
	fmt.Println(duhan)
	// new 出来的拿到的是一个指针，指向内存地址
	var hening2 = new(Hening)
	hening2.Name = "hening2"
	fmt.Println(hening2)
	hening3 := hening2
	hening3.Name = "hening3"
	// 指向同一个内存地址的变量也会被修改
	fmt.Println(hening2)
}
```

### 结构体传参出参

将定义的结构体变量，作为参数传递到func函数中即可，

```go
package main
import "fmt"
type Hening struct {
	Name string
	Age int
	Sex bool
	Hobit []string
}

func main() {
	// 声明结构体
	var hening Hening
	hening.Name = "hening"
	hening.Age = 24
	hening.Sex =true
	hening.Hobit = []string{"睡觉","游戏"}
	du :=testhening(hening)
	fmt.Println(du)
}
// 将Hening这个结构体，作为入参和出参
func testhening(he Hening)(du Hening)  {
	fmt.Println(he)
	du.Name = "duhan"
	return du
}
```

### 结构体的指针

结构体的指针取出后可以进行使用，并修改

```go
package main
import "fmt"
type Hening struct {
	Name string
	Age int
	Sex bool
	Hobit []string
}
func main() {
	// 声明结构体
	var hening Hening
	hening.Name = "hening"
	hening.Age = 24
	hening.Sex =true
	hening.Hobit = []string{"睡觉","游戏"}

	var heningP *Hening
	// 结构体的指针，取出后可以直接修改,并且修改的是内存地址里的值
	heningP = &hening
	heningP.Name= "duhan"
	fmt.Println(hening,heningP)
}
```

**注意点**

**如果想通过*取出指针对应的变量进行修改，需要加上括号,不然IEA会任务你想修改的是 结构体里面Name这个指针（由于Name不是指针，所以会报错）**

```go
(*heningP).Name = "hening2"
	fmt.Println(hening,*heningP)
```

### 给结构体声明方法

声明一个func 然后如下 指定是那一个结构体，然后加上方法名以及入参出参

func (he Hening)Song(name string)(res string)  {
	res = he.Name + "唱了一首"+name+",观众觉得唱的不错"
	fmt.Println("from song",res)
	return res
}

```go
package main

import "fmt"

type Hening struct {
	Name string
	Age int
	Sex bool
	Hobit []string
}
// 声明结构体的方法
func (he Hening)Song(name string)(res string)  {
	res = he.Name + "唱了一首"+name+",观众觉得唱的不错"
	fmt.Println("from song",res)
	return res
}

func main() {
	// 声明结构体
	var hening Hening
	hening.Name = "hening"
	hening.Age = 24
	hening.Sex =true
	hening.Hobit = []string{"睡觉","游戏"}
	res := hening.Song("惊雷")
	fmt.Println(res)
}
```

### 嵌套结构体

可以新声明一个结构体，然后在另一个结构体中使用该结构体作为变量，然后在生成实例可以，调用该结构体的变量以及方法等

```go
package main

import "fmt"

type Hening struct {
	Name string
	Age int
	Sex bool
	Hobit []string
  // 嵌套Home 结构体
	myHome Home
}
// 声明结构体的方法
func (he Hening)Song(name string)(res string)  {
	res = he.Name + "唱了一首"+name+",观众觉得唱的不错"
	fmt.Println("from song",res)
	return res
}

type Home struct {
	address string
}

func (home *Home)OPen()()  {
	fmt.Println("Open",home.address)
}

func main() {
	// 声明结构体
	var hening Hening
	hening.Name = "hening"
	hening.Age = 24
	hening.Sex =true
	hening.Hobit = []string{"睡觉","游戏"}
	res := hening.Song("惊雷")
	hening.myHome.address = "四川成都"
	hening.myHome.OPen()
	fmt.Println(res)
}
```

## interface 接口的使用

type 接口名称 interface {} 定义一个接口，里面定义接口的方法

**定义结构体，然后定义结构体的方法，这个方法实现接口中的方法，那么就表示这个结构体属于这个接口的实现（类比java中的接口和实现类）一类统一行为的抽象**

```go
package main

import "fmt"

// 定义接口
type Person interface {
	Eat()
	Run()
}
// 定义结构体
type Hening struct {
	Name string
	Age int
}
type Duhan struct {
	Name string
	Age int
}
// 给结构体挂载接口的方法
func (he Hening)Eat()  {
	fmt.Println(he.Name,"Eat ")
}
func (he Hening)Run()  {
	fmt.Println(he.Name,"Run ")
}
func (du Duhan)Eat()  {
	fmt.Println(du.Name,"Eat ")
}
func (du Duhan)Run()  {
	fmt.Println(du.Name,"Run ")
}
func main() {
	var p Person
	hening := Hening{
		Name: "hening",
		Age:  24,
	}
	p = hening
	p.Eat()
	p.Run()
}
```

## 并发神器 goroutine channel

time.Sleep() 线程休眠

time.second 时间单位 秒

go + 方法名 ： 表示这个方法已协程的方式执行，不与Main在同一线程

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	go Run()
	time.Sleep(1 * time.Second)
}

func Run()  {
	fmt.Println("Run method")
}
```

### 协程管理器

实际代码中不可能手动进行睡眠，所有使用协程管理器 **sync.WaitGroup**

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
  // add1 表示有一个协程执行，需要等到减为0
	wg.Add(1)
	// & 去除wg的指针
	go Run(&wg)
	wg.Wait()
}
// 注意使用指针传参，这样才能操作到真正的wg
func Run(wg *sync.WaitGroup)  {
	fmt.Println("Run method")
  // Done 减1
	wg.Done()
}
```

### channel

```go
// 无缓冲channel
c1 := make(chan int)

// 有缓冲区channel
c2 := make(chan int ,5)
```

向有缓冲的channel中存取

c1 <- 1 表示存入channel c1

<- c1 表示从c1中取

```go
func main() {
	// 无缓冲channel
	c1 := make(chan int,5)

	// 有缓冲区channel
	//c2 := make(chan int ,5)
	c1 <- 1
	fmt.Println(<-c1)
}
```

使用协程向channel中存数据，然后再取数据（**和视频中观察不一样，不知道是不是版本升级导致**）

并不是将channel缓冲装满才开始取，而是**在这个过程中就可能打印了出来**

```go
func main() {
   // 缓冲channel
   c1 := make(chan int,10)
   // 协程 自执行函数
   go func(){
      for i:= 0;i<10;i++{
         c1 <- i
      }
   }()
   for i := 0; i < 10; i++ {
      fmt.Println(<-c1)
   }
}
```



**可读可写channel**

顾名思义就是只能读只能写的channel，不然会报错。

注意可读 可写channel声明的差别 **<- 位置不同**

```go
func main() {
	// 可读可写channel
	c1 := make(chan int,10)
	var readc <- chan int = c1
	var write chan <- int = c1
	write<-1
	fmt.Println(<-readc)
}
```

**关闭channel**

close(chaneel)：如果不需要向channel中存数据了，那么就可以close，关闭后还是可以取数据的

```go
c1 := make(chan int)
close(c1)
```

**如果想使用for range语法获取channel中的全部值，那么必须先close掉channel才能循环取**



### select case语句

```go
func main() {
	c1 := make(chan int,1)
	c2 := make(chan int,1)
	c3 := make(chan int,1)
	c1 <- 1
	// 如果case后的语句可以执行，那么就走这个分支 有点类型switch？
  // 如果多个条件满足或者都满足，那么就随机执行（不像switch了）
	select {
	case <-c1:
		fmt.Println("c1")
	case <-c2:
		fmt.Println("c2")
	case <-c3:
		fmt.Println("c3")
	default:
		fmt.Println("all false")
	}
}
```

### go协程与channel结合

通过协程异步给channel中写值，然后再从channel中读取

```go
package main

import (
	"fmt"
)

func main() {
	c := make(chan int,1)
	var readc <-chan int = c
	var writec chan <- int = c
	go setChan(writec)
	getChan(readc)
}
func setChan(writce chan <- int)  {
	for i := 0; i < 10; i++ {
		writce <- i
	}
}
func getChan(readc <-chan int)  {
	for i := 0; i < 10; i++ {
		fmt.Println("从getChan函数取值",<-readc)
	}
}
```


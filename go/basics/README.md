# Advanved Go
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/go_init.png" width="900" height="500">
</div>

不用关心Go语言中函数栈和堆的问题, 编译器和运行时会帮我们搞定; 同样不要假设变量在内存中的位置是固定不变的, 指针随时可能变化, 特别是在你不期望它变化的时候

Go线程之所以快的原因:
1. 线程的栈大小是动态而不是固定的
2. 调度器可以在n个系统级线程上调度m个goroutine

# GO Basics

## Go Package
Let’s take a look at an example. If Go was installed under ```/usr/local/go``` and your GOPATH was set to ```/home/myproject:/home/mylibraries```, the compiler would look for the net/http package in the following order:
```
/usr/local/go/src/pkg/net/http
/home/myproject/src/net/http
/home/mylibraries/src/net/http
```

Go可以通过url import: ```import "github.com/spf13/viper"```

### Named imports
```
import (
    "fmt"
    myfmt "mylib/fmt"
)

func main() {
    fmt.Println("Standard Library")
    myfmt.Println("mylib/fmt")
}
```
类似于python里的from as, 用于解决重名问题
## 变量的赋值

```go
var a int = 8
// a:=8
```
这两种方式是一样的，注意:=左边一定是还没被声明的变量，如果声明的时候不指定值，那么默认为0

## String
字符串在Go语言内存模型中用一个2字长的数据结构表示。它包含一个指向字符串存储数据的指针和一个长度数据。因为string类型是不可变的，对于多字符串共享同一个存储数据是安全的。切分操作str[i:j]会得到一个新的2字长结构，一个可能不同的但仍指向同一个字节序列(即上文说的存储数据)的指针和长度数据。这意味着字符串切分可以在不涉及内存分配或复制操作。这使得字符串切分的效率等同于传递下标。

## 函数

```go
func function_name( [parameter list] ) [return_types] {
   函数体
}

func swap(x, y string) (string, string) {
   return y, x
}

go默认使用值传递
```

## 数组
```go
var variable_name [SIZE] variable_type
var balance [10] float32
var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
var balance = [...]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
array := [4][2]int{1: {0: 20}, 3: {1: 41}}
func foo(array *[1e6]int) {

}
```

## Slice
```go
var identifier []type   //注意这里是没有大小的，有大小的是数组

make([]T, length, capacity)
var slice1 []type = make([]type, len)
slice1 := make([]type, len)

s :=[] int {1,2,3 } 
```

```go
slice := []int{10, 20, 30, 40, 50}
newSlice := slice[1:3]
```
对于上面代码:
For **slice[i:j]** with an underlying array of capacity k

Length:   j - i

Capacity: k - i

所以newSlice只包含{20,30}

For **slice[i:j:k]**  or  [2:3:4]

Length: j-i or 3-2=1 

Capacity:k-i or 4-2=2

If you attempt to set a capacity that’s larger than the available capacity, you’ll get a runtime error.

**合并两个slice:**
```
fmt.Printf("%v\n", append(s1, s2...))
```

**slice传递**
```go
func foo(slice []int) []int {
...
    return slice
}
```
slice本身就是类似于指正, 底部指向数组, 因此slice就不用像数组一样传递

### Slice的扩容
Slice每次扩容会重新调整指针的地址

* 如果新的大小是当前大小2倍以上, 则大小增长为新大小
* 否则循环以下操作: 如果当前大小小于1024, 按每次2倍增长, 否则每次按当前大小1/4增长, 直到增长的大小超过或等于新大小

## new和make的区别
new(T) 为一个 T 类型新值分配空间并将此空间初始化为 T 的零值，返回的是新值的地址，也就是 T 类型的指针 *T，该指针指向 T 的新分配的零值。

The make built-in function allocates and initializes an object（分配空间 + 初始化） of type slice, map or chan **only**. Like new , the first arguement is a type, not a value. Unlike new, make’s return type is the same as the type of its argument, not a pointer to it. 
实际上返回的是引用

## Map
```go
/* 声明变量，默认 map 是 nil */
var map_variable map[key_data_type]value_data_type

/* 使用 make 函数 */
map_variable := make(map[key_data_type]value_data_type)
dict := map[string]string{"Red": "#da1337", "Orange": "#e95a22"}
dict := map[int][]string{}


var colors map[string]string //注意这种方式创建的map是没有初始化的, 不能赋值
colors := map[string]string{}   //通过这两种方法初始化, 所以很麻烦, 还不如直接:=呢
colors := make(map[string]string)

//countryCapitalMap := map[string]string{"France": "Paris", "Italy": "Rome", "Japan": "Tokyo", "India": "New delhi"}
var countryCapitalMap map[string]string /*创建集合 */
countryCapitalMap = make(map[string]string)

/* map插入key - value对,各个国家对应的首都 */
countryCapitalMap [ "France" ] = "巴黎"

/*使用键输出地图值 */ 
for country := range countryCapitalMap {
    fmt.Println(country, "首都是", countryCapitalMap [country])
}
delete(countryCapitalMap, "France")

/*查看元素在集合中是否存在 */
capital, ok := countryCapitalMap [ "American" ] /*如果确定是真实的,则存在,否则不存在, 不存在的话value是"", 但还是别用*/
/*fmt.Println(capital) */
/*fmt.Println(ok) */
if (ok) {
    fmt.Println("American 的首都是", capital)
} else {
    fmt.Println("American 的首都不存在")
}

if value,ok:=mp[target-num];ok{
        return []int{value,i}
}

//删除
delete(colors, "Coral")
```

**map是传递地址的**, 因此可以修改元素

map基本和java的map是相同的, 不过扩容是拷贝旧bucket的内容到新bucket上, 因此在拷贝的过程中会是无法读取的. Concurrent read (read only) is ok. Concurrent write and/or read is not ok.

# 结构体
```go
type struct_variable_type struct {
   member definition;
   member definition;
   ...
   member definition;
}

type Books struct {
   title string
   author string
   subject string
   book_id int
}

book := Books{
    title: "",
    author: "",
    subject: "",
    book_id: 1,
}
func main() {
    // 创建一个新的结构体
    fmt.Println(Books{"Go 语言", "www.runoob.com", "Go 语言教程", 6495407})

    // 也可以使用 key => value 格式
    fmt.Println(Books{title: "Go 语言", author: "www.runoob.com", subject: "Go 语言教程", book_id: 6495407})

    // 忽略的字段为 0 或 空
   fmt.Println(Books{title: "Go 语言", author: "www.runoob.com"})
}
```

```go
type user struct{
	name string
	email string
}
func (u user) notify(){
	fmt.Printf("%s + %s\n",u.name,u.email)
}

func (u *user) change(email string){
		user.email=email
}
```

# Method Interface

**方法传递很奇怪:**
Values                    Methods Receivers

T                         (t T)

*T                        (t T) and (t *T)

也就是说,参数是指针, 那么只能传指正, 参数是普通, 则都可以

## Type embedding
```go
type user struct{
    name string
    email string
}

type admin struct{
    user
    level string
}
```

### Exporting and unexporting identifiers
When an identifier starts with a lowercase letter, the identifier is unexported or unknown to code outside the package. When an identifier starts with an uppercase let- ter, it’s exported or known to code outside the package. 

# Goroutines
Go语言支持goroutine, 每个goroutine需要能够运行, 所以它们都有自己的栈. 假如每个goroutine分配固定栈大小并且不能增长, 太小则会导致溢出, 太大又会浪费空间, 无法存在许多的goroutinue.

为了解决这个问题, goroutine可以初始时只给栈分配很小的空间, 然后随着使用过程中的需要自动增长. 这就是为什么Go可以开千千万万个goroutine而不会耗尽内存.

### 基本原理
每次执行函数调用Go的runtime都会进行检每次执行函数调用时Go的runtime都会进行检测，若当前栈的大小不够用，则会触发“中断”，从当前函数进入到Go的运行时库，Go的运行时库会保存此时的函数上下文环境，然后分配一个新的足够大的栈空间，将旧栈的内容拷贝到新栈中，并做一些设置，使得当函数恢复运行时，函数会在新分配的栈中继续执行，仿佛整个过程都没发生过一样，这个函数会觉得自己使用的是一块大小“无限”的栈空间。

## channels
```go
// Unbuffered channel of integers. An unbuffered channel is a channel with no capacity to hold any value before it’s received. These types of channels require both a sending and receiving goroutine to be ready at the same instant before any send or receive operation can complete. 
unbuffered := make(chan int)
// Buffered channel of strings.
buffered := make(chan string, 10)

buffered <- "Gopher"       //send

value := <-buffered        //recieve
```


# Interface
The interface in go doesn't need to implement all methods like Java, so it's convenient to add methods in golang.

# Defer
越后面的defer表达式越先被调用

## defer使用时的坑
```go
func f() (result int) {
    defer func(){
        result++
    }()
    return 0
}

func f() (r int){
    t:=5
    defer func(){
        t = t+5
    }()
    return t
}

func f() (r int) {
    defer func (r int){
        r=r+5
    }(r)
    return 1
}
```

defer是在return之前执行的, return语句并不是一条原子指令.

函数返回的过程是这样的: 先给返回值赋值, 然后调用defer表达式, 最后才是返回到调用函数中. 如下
```
func () (返回值 int)
返回值 = xxx
调用defer函数
空的return
```

因此, 改写上面的

```go
func f() (result int){
    result = 0
    func() {
        result++
    }()
    return
}
//1

func f() (r int){
    t:=5
    r=t
    func(){
        t=t+5
    }
    return
}
//5

func f() (r int){
    r=1
    func (r int){
        r=r+5
    }(r)
    return
}
1
```

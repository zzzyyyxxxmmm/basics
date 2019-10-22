# GO Basics
之前只是快速的略读了go的语法，最近在学分布式系统时觉得代码读起来很困难，因此还是再次系统性的学习一下吧

学习一个语言，第一个代码一定是hello world，那先写个hello world吧:

```go
package main
import "fmt"
func main() {
	fmt.Println("hello world")
}
```

命令行运行：go run hello.go 
编译并运行：go build hello.go   ->   ./hello 

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

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func main() {

	runtime.GOMAXPROCS(2)

	var wg sync.WaitGroup
	wg.Add(2)

	fmt.Println("Start Goroutines")

	go func() {
		defer wg.Done()
		for count := 0; count < 3; count++ {
			for char := 'a'; char < 'a'+26; char++ {
				fmt.Printf("%c ",char)
            }
            
		}
	}()


	go func() {
		defer wg.Done()
		for count := 0; count < 3; count++ {
			for char := 'A'; char < 'A'+26; char++ {
				fmt.Printf("%c ",char)
			}
		}
	}()

	fmt.Println("Waiting To Finish")
	wg.Wait();

	fmt.Println("\nTerminating Program")
}
```

```go
mutex.Lock()
{

}
mutex.Unlock()
```

## channels
```go
// Unbuffered channel of integers. An unbuffered channel is a channel with no capacity to hold any value before it’s received. These types of channels require both a sending and receiving goroutine to be ready at the same instant before any send or receive operation can complete. 
unbuffered := make(chan int)
// Buffered channel of strings.
buffered := make(chan string, 10)

buffered <- "Gopher"       //send

value := <-buffered        //recieve
```
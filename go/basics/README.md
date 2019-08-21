#GO Basics
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
```

## 结构体
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
func main() {
    // 创建一个新的结构体
    fmt.Println(Books{"Go 语言", "www.runoob.com", "Go 语言教程", 6495407})

    // 也可以使用 key => value 格式
    fmt.Println(Books{title: "Go 语言", author: "www.runoob.com", subject: "Go 语言教程", book_id: 6495407})

    // 忽略的字段为 0 或 空
   fmt.Println(Books{title: "Go 语言", author: "www.runoob.com"})
}
```

结构体通过指针方式传递，则可以修改值

## Slice
```go
var identifier []type   //注意这里是没有大小的，有大小的是数组

make([]T, length, capacity)
var slice1 []type = make([]type, len)
slice1 := make([]type, len)

s :=[] int {1,2,3 } 
```

## Map
```go
/* 声明变量，默认 map 是 nil */
var map_variable map[key_data_type]value_data_type

/* 使用 make 函数 */
map_variable := make(map[key_data_type]value_data_type)


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
    capital, ok := countryCapitalMap [ "American" ] /*如果确定是真实的,则存在,否则不存在 */
    /*fmt.Println(capital) */
    /*fmt.Println(ok) */
    if (ok) {
        fmt.Println("American 的首都是", capital)
    } else {
        fmt.Println("American 的首都不存在")
    }
```
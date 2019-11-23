### 1

Let's first take loot at the following code snippet.
```go
package main

import "fmt"
type A struct {
	num int
}

func Change(o interface{}) {
	o = nil
}

func main() {
	a := &A{num: 1}
	Change(a)
	fmt.Println(a)
}
```

Will the value of a will be changed after we call Change method? Of course not because go passes parameters by value not reference.

If we want change it, we need to find the real value it points to.

### 2
But we can not do it like o.num=2, because we don't know the type of o.

Let first review the structure of interface

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/go_interface.png" width="700" height="200">
</div>

It contains two elements: type and value, we can get them by reflection.

```go
func Change(o interface{}) {
	fmt.Println(reflect.TypeOf(o))
	fmt.Println(reflect.ValueOf(o))
	o = nil
}
```

output:
```
*main.A
&{1}
```

Although go passes parameters by value, but here o is a pointer points to a, so if we can change the value of where o point to, which means we can change the value of a.

### 3
Ok, let's do it.

```go
func Change(o interface{}) {
	value:=reflect.ValueOf(o).Elem()
	value.Set(reflect.ValueOf(A{num:2}))
}
```

Output:
&{2}

I will not dig into the reflection here, please read chapter reflection in a reference book.

### 4
Now, we can change the value of o and a. But as you can see in the function. We have to know struct A in advance. We want to implement a general function in order to apply it on arbitrary type.

If you are familir with json.unmarshal method in go, I'm sure you can easily understand the following code.

```go
func Change(o interface{}) {
	value:=reflect.ValueOf(o).Elem()
	for i:=0; i<value.NumField();i++{
		v:=value.Field(i)
		switch v.Kind() {
		case reflect.Int:
			v.SetInt(2)
		}
	}
}
```

### 5
Let's make it be more generic
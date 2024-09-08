---
title: Go 指针接收者
categories: Go
tags:
  - Go
date: 2024-09-02 14:09:31
---

Go 语言中可以对方法声明**接收者**，接收者类型可以是结构体也可以是接口。这意味着被声明的方法只能被该结构体或者是接口的实例调用。

语法如下：

```go
func (t Type) methodMame(params) returns {
    // method body
}
```

特别地，在此处可以声明指针接收者，语法如下：

```go
func (t *Type) methodMame(params) returns {
    // method body
}
```

在 [A tour of Go](https://go.dev/tour/methods/4) 中介绍到

> You can declare methods with pointer receivers.
> // 你可以为方法绑定一个指针接收者
>
> This means the receiver type has the literal syntax *T for some type T. (Also, T cannot itself be a pointer such as *int.)
> // 这意味着接收者类型是 T 的指针，其中 T 不能是像 \*int 这样的指针类型。

他的作用也很简单，就是可以修改接收者指向的值。如果没有使用指针接收者，那么修改的只是接收者的一个副本，并不影响原来的值。

从字面语义上也很直观，我们通常理解的指针就是指向变量的地址，对指针的操作会影响原来的变量值。

## 实例

举例来说，声明一个结构体`Vertex`，为`Vertex`绑定方法`Abs`、`Scale`和`Shrink`，其中`Abs`、`Shrink`方法使用值接收者，`Scale`方法使用指针接收者。

```go
import (
        "fmt"
        "math"
)

type Vertex struct {
        X, Y float64
}

func (v Vertex) Abs() float64 {
        return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) {
        v.X = v.X * f
        v.Y = v.Y * f
}

func (v Vertex) Shrink(f float64) {
        if f == 0 {
                return
        }
        v.X = v.X / 10
        v.Y = v.Y / 10
}

func main() {
        v := Vertex{3, 4}
        fmt.Println(v.Abs())
        v.Scale(10)
        fmt.Println(v.Abs())
        v.Shrink(10)
        fmt.Println(v.Abs())
}
```

输出的结果为：

```
5
50
50
```

可以看到，`Abs`和`Shrink`方法使用值接收者，所以`Abs`和`Shrink`方法内部对`v`的修改不会影响原来的`v`。

而`Scale`方法使用指针接收者，所以`Scale`方法内部对`v`的修改会影响原来的`v`。

如果将`Shrink`方法的接收者类型改为指针接收者：

```go
func (v *Vertex) Shrink(f float64) {
        if f == 0 {
        	return
        }
        v.X = v.X / 10
        v.Y = v.Y / 10
}
```

结果为：

```
5
50
5
```

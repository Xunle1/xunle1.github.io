---
title: Go 中的初始化
date: 2024-08-07 19:22:31
categories: Go
tags:
  - Go
---

Go 语言会在声明变量类型的时候进行初始化，如果没有指定值，那将会使用零值。各种类型的零值：

- int: 0
- float: 0.0
- bool: false
- string: ""
- 切片: []
- 指针、函数、接口: nil
- 结构体: 由属性的零值组成

需要注意的是，Go 语言中声明（declaration）和定义（definition）都会为变量分配空间。意思是不在需要像 Java 那样必须调用 new 来构造对象。

举例如下：

```go
package main

import "fmt"

type StructA struct {
        A int
        B *int
}

type Inter interface {
        Hello(string) error
}

type StructB struct {
        Int       int
        String    string
        Struct    StructA
        Point     *StructA
        Array     []int
        Fun       func(int, int) int
        Interface Inter
}

func main() {
        var declaration StructB
        fmt.Printf("Declaration StructB: \n%+v\n", declaration)
        var definition = StructB{Int: 3, String: "foo"}
        fmt.Printf("Definition StructB: \n%+v\n", definition)
}
```

输出为：

```
Declaration StructB:
{Int:0 String: Struct:{A:0 B:<nil>} Point:<nil> Array:[] Fun:<nil> Interface:<nil>}
Definition StructB:
{Int:3 String:foo Struct:{A:0 B:<nil>} Point:<nil> Array:[] Fun:<nil> Interface:<nil>}
```

可以发现声明的变量也可以直接使用，并且属性都被赋予了零值。

# Reference

[Effective Go](https://go.dev/doc/effective_go#data)


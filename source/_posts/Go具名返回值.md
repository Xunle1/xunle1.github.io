---
title: Go 具名返回值
date: 2024-09-02 14:43:00
categories: Go
tags:
  - Go
---

Go 语言可以对函数的返回值命名，返回值会被自动被初始化为零值而不需要你的处理，同时也可以使用“裸”返回语句。以下是 [A Tour of Go](https://go.dev/tour/basics/7) 中的例子：

```go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 3 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(10))
}
```

## 什么时候使用

[Effective Go](https://go.dev/doc/effective_go#named-results) 中主要说明了**具名返回值**可以提高函数的可读性。特别是在返回值类型相同的情况下，比如一个返回经纬度的函数：

```go
func location(name City) (lat, long float64)
```

我们可以从返回值列表中知道经度和纬度分别对应第几个参数。

## 注意事项

1. [A Tour of Go](https://go.dev/tour/basics/7) 中也提到，应该在**短方法**中使用“裸返回”，否则会降低代码的可读性。

2. 具名返回值也会因为我们处理不当而造成错误。比如：

```go
func todoWithTimeout(c *context.Context) (err error) {
  // do something

  if c.Err() != nil {
    return
  }

  // do something and return
}
```

在检查了上下文是否超时后，直接返回了，导致`err`还是`nil`，造成了错误。

正确做法应该是：

```go
func todoWithTimeout(c *context.Context) (err error) {
  // do something

  if err = c.Err(); err != nil {
    return
  }

  // do something and return
}
```


---
title: Go init 函数顺序（转）
date: 2024-09-02 17:31:28
categories: Go
tags:
  - Go
---

转自[一文读懂 Golang init 函数执行顺序](https://cloud.tencent.com/developer/article/2138066)

Go 语言中，`init` 函数用于在程序启动时执行初始化操作。每个包都可以包含一个 `init` 函数，Go 会按顺序执行所有包的 `init` 函数。

以下为 `init` 函数的执行顺序一图流：

![init 函数执行顺序](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/init-sequence.png)

程序中如果在不同包的不同源文件有多个 init 函数时，其执行顺序可概述为：

- 同一个源文件的 init 函数执行顺序与其定义顺序一致，从上到下；
- 同一个包中不同文件的 init 函数的执行顺序按照文件名的字典序；
- 对于不同的包，如果不相互依赖的话，按照 main 包中 import 的顺序调用其包中的 init 函数；
- 如果包存在依赖，不同包的 init 函数按照包导入的依赖关系决定执行顺序。 调用顺序为最后被依赖的最先被初始化，如导入顺序 main > a > b > c，则初始化顺序为 c > b > a > main，依次执行对应的 init 方法；
- 如果包存在包级变量，则先于包的 init 函数完成初始化。


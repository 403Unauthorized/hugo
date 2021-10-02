---
title: "Golang 基础知识"
date: 2021-10-02T10:49:14+09:00
tags: ["Golang"]
categories: ["Programming"]
draft: false
author: "Torres"
toc:
  enable: true
  auto: true
share:
  enable: true
comment:
  enable: true
featuredImagePreview: "/hugo/images/posts/golang-fundamentals/golang-fundamentals.png"
featuredImage: "/hugo/images/posts/golang-fundamentals/golang-fundamentals.png"
---

## Hello Golang

Golang 是由 Google 开发并开源出来的一种编程语言，Kubernetes 就是由 Go 主要开发的，由此可见 Go 在云原生开发中的重要地位。在这里我们主要介绍 Golang 的基础知识，例如各种语法，数据结构，以及一些 Tips。

## 声明和赋值

Go 中声明变量和给变量赋值的方法和 JavaScript 比较相似。如下：

```go
// var 是声明变量的关键字，如果只声明变量而不给它赋值
// 则该变量会被赋值为该类型的零值（zero value）
var num int

// := 是声明并赋值的操作符
// 其工作流程为：
// 1. 声明一个名为 num 的变量
// 2. 将 10 赋值给变量 num
num := 10
```

> 但是要注意的是，每个类型的零值不相同，比如说 `int` 的零值为 0，浮点类型的零值为 0.0，而 `string` 的零值为空字符串：""... 如 `struct` 或者指针类型的变量，零值为 `nil`。

## 基础数据类型

Go 的基础数据类型和其他的编程语言相似，在这里我们介绍一下 Go 中的整型、字符串、浮点型、布尔型等：

### 整型

**有符号整型**：

| 类型  | 大小       | 范围           |
| :---- | :--------- | :------------- |
| int8  | 8 bits     | -128 to 127    |
| int16 | 16 bits    | -215 to 215 -1 |
| int32 | 32 bits    | -231 to 231 -1 |
| int64 | 64 bits    | -263 to 263 -1 |
| int   | 取决于平台 | 取决于平台     |

`int` 的大小取决于平台，在32位系统中它是32位的，而在64位系统中，它是64位。

**无符号整型**：

| 类型   | 大小       | 范围        |
| :----- | :--------- | :---------- |
| uint8  | 8 bits     | 0 to 255    |
| uint16 | 16 bits    | 0 to 216 -1 |
| uint32 | 32 bits    | 0 to 232 -1 |
| uint64 | 64 bits    | 0 to 264 -1 |
| uint   | 取决于平台 | 取决于平台  |

`uint` 和 `int` 的大小一样取决于所在的平台。

> 提示：当你使用 integer 类型的值时，除非你有更好的原因去使用无符号整型或者带位数的整型类型，否则请使用 `int` .

Golang 中还有两个额外的类型可以表示整型：

| 类型 | 表示为 |
| :--- | :----- |
| byte | uint8  |
| rune | int32  |

在 Go 中，`rune` 和 `byte`是用来区分字符和整型类型的。Go 中没有 `char` 类型，它用 `rune` 和 `byte` 来表示字符（但是它们本质上都是整数类型）：

```go
var firstLetter = 'A' // Type inferred as `rune` (Default type for character values)
var lastLetter byte = 'Z'
```

而 `byte` 可以转换成 ASCII 码中对应的数字，如 `firstLetter` 可以转换成 65。`rune` 变量能转换为 Unicode 中对应的编码。

```go
package main
import "fmt"

func main() {
	var myByte byte = 'a'
	var myRune rune = '♥'

	fmt.Printf("%c = %d and %c = %U\n", myByte, myByte, myRune, myRune)
}
```

```go
# Output
a = 97 and ♥ = U+2665
```

可以看到 a 被转成了 97，Unicode 值 '♥' 被转换成了对应的 Unicode 码：`U+2665`。*U* 代表 Unicode，后面的数字则是16进制的整数。

### 浮点型

浮点类型的数字包括两种：

- **float32**：在内存中占用 32 bits，以单精度浮点格式存储值。
- **float64**：在内存中占用 64 bits，以双精度浮点格式存储值。

## 集合类型

### 数组

在 Go 中，数组是**固定长度**的数据类型，包含相同类型元素的**连续内存块**。定义数组的代码如下：

```go
// 声明一个长度为5的数组，默认初始化每个元素为0（参考上面说到的零值）
var arr [5]int

// 声明并初始化一个长度为5的数组
arr := [5]int{1,2,3,4,5}

// 声明一个int数组，Go会基于初始化元素的个数来决定其长度
arr := [...]int{1,2,3}

// 声明一个长度为5的int数组，但是只初始化指定位置的元素
arr := [5]int{1: 20, 3: 40}

```

> **注意**：
>
> 1. 数组只要被声明之后，就无法再更改其长度和类型了。
> 2. 在 functions 之间传递数组是非常昂贵的，如果是在需要传递数组，可以考虑传递它的指针。
>    1. 声明一个一百万个元素的 int 数组，在64位操作系统上会占用 8MB 内存。
>    2. 而在 function 之间传递时，每调用一次 function，都会在栈中分配一个 8MB 的空间给这个数组。
>    3. 而传递这个数组的指针则会更高效，指针对象只会占用8个字节空间。

### Slice



### Map


---
title: "Golang Fundamentals"
date: 2021-10-02T10:49:20+09:00
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
featuredImagePreview: "/images/posts/golang-fundamentals/golang-fundamentals.png"
featuredImage: "/images/posts/golang-fundamentals/golang-fundamentals.png"
---

## Hello Golang

Golang is developed and published by Google, Kubernetes - which is know as a tool for container orchestration, is mainly developed by Go, so Go is very important in Cloud Native. In this article, we are gonna introduce all kinds of grammars and data structure in Golang, also we will share some tips for programming.

## Declaring and Assignment

Declaring a variable in Go is similar to JavaScript, you can see following example:

```go
// var 是声明变量的关键字，如果只声明变量而不给它赋值
// 则该变量会被赋值为该类型的零值（zero value）
var num int

// := is operator of declaring and assignment
// the workflow is:
// 1. Declaring a variable named num
// 2. Assign 10 to variable num
num := 10
```

> But you need to pay attention to that zero value of each type is different, such as zero value of `int` is 0, floating point value's zero value is 0.0, string is "" (empty string). Variables like `struct` or pointer type, their zero value is `nil`.

## Primitive Type in Go

### Integers

**Signed Integers**:

| Type  | Size               | Range              |
| :---- | :----------------- | :----------------- |
| int8  | 8 bits             | -128 to 127        |
| int16 | 16 bits            | -215 to 215 -1     |
| int32 | 32 bits            | -231 to 231 -1     |
| int64 | 64 bits            | -263 to 263 -1     |
| int   | Platform dependent | Platform dependent |

The size of the generic `int` type is platform dependent. It is 32 bits wide on a 32-bit system and 64-bits wide on a 64-bit system.

**Unsigned Integers**:

| Type   | Size               | Range              |
| :----- | :----------------- | :----------------- |
| uint8  | 8 bits             | 0 to 255           |
| uint16 | 16 bits            | 0 to 216 -1        |
| uint32 | 32 bits            | 0 to 232 -1        |
| uint64 | 64 bits            | 0 to 264 -1        |
| uint   | Platform dependent | Platform dependent |

The size of `uint` type is platform dependent. It is 32 bits wide on a 32-bit system and 64-bits wide on a 64-bit system.

> Tips: When you are working with integer values, you should always use the `int` data type unless you have a good reason to use the sized or unsigned integer types.

There are two additional type that are alias for `uint8` and `int32` data types:

| Type | Alias For |
| :--- | :-------- |
| byte | uint8     |
| rune | int32     |

In Go, `rune` and `byte` data types are used to distinguish characters and integers.

Golang doesn't have `char` type, it uses `byte` and `rune` to represent character value. But they are essentially integers.

```go
var firstLetter = 'a'
var lastLetter byte = 'z'
```

`byte` can be converted to ASCII code, for example `firstLetter` can represents 97. `rune` value can be converted to code in **Unicode** as following shows:

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

Variable `a` is converted to 97 and `rune` variable with a unicode value '♥' is converted to corresponding unicode codepoint `U+2665`, where `U+` means unicode and the numbers are hexadecimal, which is essentially an integer.

### Float

Go has two floating point type:

- **float32**: Occupy 32 bits in memory and store values in single precision floating point format.
- **float64**: Occupy 64 bits in memory and store values in double precision floating point format.

## Collections

### Arrays

In Go, array is a **fixed-length** data type which contains **contiguous memory block** of elements of same type. Let's see how to declare an array:

```go
// Declare an integer array with length 5. Each element in it will be assigned 0 by default. 
// Because 0 is zero value of int
var arr [5]int

// Decalre an integer array with 5 elements and initialized.
arr := [5]int{1,2,3,4,5}

// Go will identify its length based on the initialized elements.
arr := [...]int{1,2,3}

// Declare an integer array and initialize specific elements.
arr := [5]int{1: 20, 3: 40}

```

> **Attention**：
>
> 1. Once array is declared, we can't change its length and type。
> 2. It's expensive to pass array between functions，if you really need to pass array to a function, consider pass its pointer。
>    1. Declare an array with one million elements will allocate 8 megabytes memory on 64-bit system。
>    2. When pass array between functions, every call to the function, 8 megabytes has to be allocated on the stack, then all of the 8 megabytes array will be copied into that allocation。
>    3. But you can pass a pointer to the array and only need to copy eight bytes instead of eight megabytes on the stack。

### Slices



### Maps


---
title: Convert Functions to Interfaces and vis versa in Go
description: How to convert a function to an interface in Go and how to convert an interface to a function
slug: go-convert-functions-to-interfaces
date: 2024-08-31 00:00:00+0000
categories:
    - Go
---

In this article, we will show how to use a function when you code expects an interface, and vis versa.


## From Interface to Function

This is the easy direction, to use a single-method interface where a function is expected, simply pass the interface's method:

Let's say that you have an object implementing this interface:

```go
type Interface interface {
    MethodName() string
}
```

And you want to use `funcUserCode` that requires a function object:

```go
type FuncType func() string
func funcUserCode(f FuncType) {
    println(f())
}
```
Then, you can directly pass the method as argument:

```go
func convert(myObject Interface) {
    var function FuncType = myObject.MethodName
    funcUserCode(myObject.MethodName)
}
```
Or you can wrap the method call in an anonymous function:

```go
func convert(myObject Interface) {
    function := func() string { return i.MethodName() }
    funcUserCode(function)
}
```

## From Function to Interface

On the other hand, if you need to convert a function to a single-method interface, then you can define a method on the function type directly.

For instance, let's say that you have the following function:

```go
func funcName() string {
    return "https://takia.dev"
}
```

And you need an object implementing this interface in order to user the `userCode` function.

```go
type Interface interface {
    MethodName() string
}

func userCode(i Interface) {
    println(i.MethodName())
}
```

The following code creates a user-defined function type and defines a method `MethodName` on that type:

```go
type FuncType func() string

func (f FuncType) MethodName() string {
    return f()
}
```

Which let's you easily convert between the function and the interface as follows:

```go
var i Interface = FuncType(funcName)
userCode(i)
```
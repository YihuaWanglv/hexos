---
title: Golang Tour学习记录
date: 2018-12-13 17:38:04
tags: [go, golang, Go, Tour]
categories: Go
---


## Go Tour: Golang tour学习中的一些记录，帮组理解和记忆

- 设置GOPATH，可以改变go工作空间

- 工作空间中需要一个src目录，需要一个bin目录

- 导入package时，使用pkg的时候，只需要导入路径中的最后一段即可引用该包

- go build or go install,注意目录

- package中大写的定义是对外开放的，小写的则外面不可用

- 类型：string boolean int(int int8 int32 int 64) uint(...) complex(complex64 complex 128)

- if else, for, switch, 没有while，while使用for代替

- = 是赋值, := 声明并赋值

- structs, array, slice, map, pointer

- Function, Method

func 可以作为参数，有func闭包

Method则是定义于struct上的方法，如果是定义于struct值上的方法，不会改变struct值本身；如果是定义于struct指针上的方法，可以改变struct的值。

给struct定方法有2种方式，一种是把struct作为方法的receiver，另一种是把struct作为方法的参数。

两者有一个重要的差别:

functions with a pointer argument must take a pointer:
```
var v Vertex
ScaleFunc(v, 5)  // Compile error!
ScaleFunc(&v, 5) // OK
```

while methods with pointer receivers take either a value or a pointer as the receiver when they are called:
```
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
```

当一个函数有一个指针参数，那么传入的参数必须是指针；
而当一个函数有一个指针的receiver，那么调用这个方法时，用指针或者值都是可以的。



- interface

type只要实现了interface的方法，即为interface的实现，无需显式的定义。

在底层interface values可以被看做是含value和type的一个元祖：
```
(value, type)
```

an interface value that holds a nil concrete value is itself non-nil.
就是没有空指针问题


- Errors

function call的时候，就看err这个返回值，如果是nil，意味着成功，非空意味着执行报错了。

- Reader

Reader可以嵌套

- Image

- Goroutines


- Channels

Channels are a typed conduit through which you can send and receive values with the channel operator, <-.
```
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and
           // assign value to v.
(The data flows in the direction of the arrow.)
```
Like maps and slices, channels must be created before use:
```
ch := make(chan int)
```

- Range and Close

channel能Range，也能close，只有sender能close channel，receiver不能。
一般不需要close channel，除非你需要显式的通知receiver channel已关闭。

- Select

它会监听case中的channel，当channel发生变时，再触发一次选择

- sync.Mutex

互斥锁

```
// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
  c.mux.Lock()
  // Lock so only one goroutine at a time can access the map c.v.
  c.v[key]++
  c.mux.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
  c.mux.Lock()
  // Lock so only one goroutine at a time can access the map c.v.
  defer c.mux.Unlock()
  return c.v[key]
}
```

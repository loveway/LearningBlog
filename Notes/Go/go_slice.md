
# Go 中 slice 的那些事 

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/go.png'>
</p>

## 一、定义
&emsp;&emsp;我们都知道在 Go 语言中，数组的长度是不可变的，那么为了更加灵活的处理数据，Go 提供了一种功能强悍的类型切片（slice），slice 可以理解为 “动态数组”。但是 slice 并不是真正意义上的动态数组，而是一个引用类型。slice 总是指向一个底层 array，slice 的声明也可以像 array 一样，只是不需要长度。slice 的声明和数组类似，如下
```go
var iSlice []int
```
这里的声明和数组一样，只是少了长度，注意两者的比较
```go
//声明一个保存 int 的 slice
var iSlice []int

//声明一个长度为 10 的 int 数组
var iArray [10]int

```
还有一种声明的方法是使用 make() 函数，如下

```go
slice1 := make([]int, 5, 10)
```
用 make() 函数创建的时候有三个参数，`make(type, len[, cap])` ，依次是类型、长度、容量。

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/slice.png'>
</p>

如图所示，上图表示创建了 slice1 ，长度是 5，默认的值都是 0，容量是 10，这样声明就开辟了一块容量是 10 的连续的一块内存。当然如果我们不指定容量也是可以的，如下
```go
slice2 := make([]int, 5)
```
这样就会根据实际情况动态分配内存，而不是最开始指定一块固定大小的内存。需要注意的是我们一般使用 `make()` 函数来创建 slice，因为我们可以指定 slice 的容量，这样在最开始创建的时候就分配好空间，避免数据多次改变导致多次重新改变 cap 分配空间带来不必要的开销。
## 二、slice 的特性
&emsp;&emsp;关于 slice 的一些基本特性，[《Go Web 编程》](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.2.md) 这本书里已经讲的很详细，有对基本知识不清楚的童鞋可以去补习一下，这里就不一一叙述了。我么来看一个例子，
```go
package main

import (
	"fmt"
)

func main() {
	aSlice := []int{1, 2, 3, 4, 5}
	fmt.Printf("aSlice length = %d, cap = %d, self = %v\n", len(aSlice), cap(aSlice), aSlice)
	aSlice = append(aSlice, 6)
	fmt.Printf("aSlice length= %d, cap = %d, self = %v", len(aSlice), cap(aSlice), aSlice)
}
```
这个时候我们运行，控制台打印

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/slice10.png'>
</p>

我们会看到 aSlice 进行 append 操作以后，它的容量增加了一倍，cap 并没有变成我们想象中的 6 ，而是变成了 10

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/slice2.png'>
</p>

如果我们最开始 slice 的容量是 10，长度是 5 ，那么再加一个元素是不会改变切片的容量的。也就是说，当我们往 slice中增加元素超过原来的容量时，slice 会自增容量，当现有长度 < 1024 时 cap 增长是翻倍的，当超过 1024，cap 的增长是 1.25 倍增长。我们来看一下 [slice.go](https://github.com/golang/go/blob/master/src/runtime/slice.go) 的源码会发现有这样一个函数，里面说明了 cap 的增长规则
```go
func growslice(et *_type, old slice, cap int) slice {
/**
    ....省略....
**/
	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.len < 1024 {
			newcap = doublecap
		} else {
			// Check 0 < newcap to detect overflow
			// and prevent an infinite loop.
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
/**
    ....省略....
**/
}
```

从上面的源码，在对 slice 进行 append 等操作时，可能会造成 slice 的自动扩容。其扩容时的大小增长规则是：

* 如果新的 slice 大小是当前大小2倍以上，则大小增长为新大小
* 否则循环以下操作：如果当前slice大小小于1024，按每次 2 倍增长，否则每次按当前大小 1/4 增长，直到增长的大小超过或等于新大小。
* append 的实现只是简单的在内存中将旧 slice 复制给新 slice

来看一个例子，
```go
package main

import "fmt"

func main() {

	aSlice := make([]int, 3, 5)
	bSlice := append(aSlice, 1, 2)
	fmt.Printf("a %v , cap = %d, len = %d\n", aSlice, cap(aSlice), len(aSlice))
	fmt.Printf("b %v , cap = %d, len = %d\n", bSlice, cap(bSlice), len(bSlice))
	aSlice[0] = 6
	fmt.Printf("a %v , cap = %d, len = %d\n", aSlice, cap(aSlice), len(aSlice))
	fmt.Printf("b %v , cap = %d, len = %d", bSlice, cap(bSlice), len(bSlice))

}

```
我们会看到控制台输出

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/slice3.png'>
</p>

变化过程如下图所示

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/slice4.png'>
</p>

上面说明，在 slice 的 cap 范围内增加元素， slice 只会发生 len 的变化不会发生 cap 的变化，同样也说明 slice 实际上是指向一个底层的数组，当多个 slice 指向同一个底层数组的时候，其中一个改变，其余的也会跟着改变，这里需要注意一下。我们同样从 [slice.go](https://github.com/golang/go/blob/master/src/runtime/slice.go) 的源码中 slice 的定义可以看出，
```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

这里关于底层的东西就不多叙述，有兴趣的可以看看 [一缕殇流化隐半边冰霜](http://www.jianshu.com/u/12201cdd5d7a) 冰霜的 [深入解析 Go 中 Slice 底层实现](http://www.jianshu.com/p/030aba2bff41) 这篇文章，对 slice 的底层实现的讲解。接下来我们把上面的代码改变一下

```go
package main

import "fmt"

func main() {

	aSlice := make([]int, 3, 5)
	bSlice := append(aSlice, 1, 2, 3, 4)
	fmt.Printf("a %v , cap = %d, len = %d\n", aSlice, cap(aSlice), len(aSlice))
	fmt.Printf("b %v , cap = %d, len = %d\n", bSlice, cap(bSlice), len(bSlice))
	aSlice[0] = 6
	fmt.Printf("a %v , cap = %d, len = %d\n", aSlice, cap(aSlice), len(aSlice))
	fmt.Printf("b %v , cap = %d, len = %d", bSlice, cap(bSlice), len(bSlice))

}
```
我们可以看到下面的输出

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/slice5.png'>
</p>

上面代码可以用下图说明

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/slice6.png'>
</p>

也就是说，当 append 的数据超过原来的容量以后，就会重新分配一块新的内存，并把原来的数据 copy 过来，并且保留原来的空间，供原来的 slice（aSlice） 使用这样 aSlice 和 bSlice 就各自指向不同的地址，当 aSlice 改变时，bSlice 不会改变。
关于 cap 还有一点需要注意，我们来用一个例子说明
```go
package main

import "fmt"

func main() {

	Array_a := [10]byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
	Slice_a := Array_a[2:5]
	Slice_b := Slice_a[6:7]
	fmt.Printf("Slice_a %v , cap = %d, len = %d\n", string(Slice_a), cap(Slice_a), len(Slice_a))
	fmt.Printf("Slice_b %v , cap = %d, len = %d\n", string(Slice_b), cap(Slice_b), len(Slice_b))
}
```
控制台打印

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/slice7.png'>
</p>

这里我们会发现 Slice_b 对 Slice_a 进行重新切片后，并没有报错，而是还有输出，这是因为 Slice_a 的 cap 是 8 ，并不是我们想象的 3，slice 指向的是一块连续的内存，所以 Slice_a 的容量其实是一直到 Array_a 的最后的。所以这里 Array_b 对 Array_a 进行切片后会得到值，[《Go Web 编程》](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.2.md) 上这张图形象的解释了对数组的切片结果，这里是需要注意的一个点。

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/slice8.png'>
</p>

## 三、关于 copy
&emsp;&emsp;我们来看下面代码
```go
package main

import "fmt"

func main() {

	aSlice := []int{1, 2, 3}
	bSlice := []int{4, 5, 6, 7, 8, 9}
	copy(bSlice, aSlice)
	fmt.Println(aSlice, bSlice)//[1 2 3] [1 2 3 7 8 9]
       //如果是 copy( aSlice, bSlice) 则结果是 [4 5 6] 
}
```
也就是说 `copy()` 函数有两个参数，一个是 to 一个是 from，就是将第二个 copy 到第一个上面，如果第一个长度小于第二个，那么就会 copy 与第一个等长度的值，如 copy( aSlice, bSlice) 的结果是 [4 5 6] ，反之则是短的覆盖长的前几位。当然我们也可以指定复制长度
```go
package main

import "fmt"

func main() {

	aSlice := []int{1, 2, 3}
	bSlice := []int{4, 5, 6, 7, 8, 9}
	copy(bSlice[2:5], aSlice)
	fmt.Println(aSlice, bSlice)//[1 2 3] [4 5 1 2 3 9]
}
```
关于 slice 的 copy 的规则逻辑我们也可以在源码中看出
```go
func slicecopy(to, fm slice, width uintptr) int {
	if fm.len == 0 || to.len == 0 {
		return 0
	}

	n := fm.len
	if to.len < n {
		n = to.len
	}

	if width == 0 {
		return n
	}

	if raceenabled {
		callerpc := getcallerpc()
		pc := funcPC(slicecopy)
		racewriterangepc(to.array, uintptr(n*int(width)), callerpc, pc)
		racereadrangepc(fm.array, uintptr(n*int(width)), callerpc, pc)
	}
	if msanenabled {
		msanwrite(to.array, uintptr(n*int(width)))
		msanread(fm.array, uintptr(n*int(width)))
	}

	size := uintptr(n) * width
	if size == 1 { // common case worth about 2x to do here
		// TODO: is this still worth it with new memmove impl?
		*(*byte)(to.array) = *(*byte)(fm.array) // known to be a byte pointer
	} else {
		memmove(to.array, fm.array, size)
	}
	return n
}
```
我们看源码接着往下看会发现这样一个方法
```go
func slicestringcopy(to []byte, fm string) int {
	if len(fm) == 0 || len(to) == 0 {
		return 0
	}

	n := len(fm)
	if len(to) < n {
		n = len(to)
	}

	if raceenabled {
		callerpc := getcallerpc()
		pc := funcPC(slicestringcopy)
		racewriterangepc(unsafe.Pointer(&to[0]), uintptr(n), callerpc, pc)
	}
	if msanenabled {
		msanwrite(unsafe.Pointer(&to[0]), uintptr(n))
	}

	memmove(unsafe.Pointer(&to[0]), stringStructOf(&fm).str, uintptr(n))
	return n
}
```
我们会发现这个函数的两个参数分别是 []byte 和 string ，这里其实是 Go 实现了一个将 string 复制到 []byte 上的方法，这个方法有什么用，我们来看个例子
```go
package main

import "fmt"

func main() {

	s := "hello"
	c := []byte(s) // 将字符串 s 转换为 []byte 类型
	c[0] = 'c'
	s2 := string(c) // 再转换回 string 类型
	fmt.Printf("%s\n", s2)
	fmt.Printf("s-%x, c-%x, s2-%x", &s, &c, &s2)
}
```
控制台输出

<p align='center'>
<img src='https://github.com/loveway/LearningBlog/blob/master/Notes/Go/image/slice9.png'>
</p>

在 Go 中字符串是不可以改变的，我们可以用上面的方法来改变字符串，这里可以看到是实现了 string 和 []byte 的互相转换，达到了修改 string 的目的。我们去看看 [string.go](https://github.com/golang/go/blob/master/src/runtime/string.go) 的源码会发现，有下面的方法
```go
func stringtoslicebyte(buf *tmpBuf, s string) []byte {
	var b []byte
	if buf != nil && len(s) <= len(buf) {
		*buf = tmpBuf{}
		b = buf[:len(s)]
	} else {
		b = rawbyteslice(len(s))
	}
	copy(b, s)
	return b
}

```
可以看到上面有个 copy(b, s) ，这里就是将 string 复制到 []byte 上，在 slice.go 已经实现过了的。从源码中我们也可以看出每次 b 都是重新分配的，然后将 s 复制 给 b，从我们上面程控制台输出也可以看到每次地址都有变化，所以说 string 和 []byte 的相互转换是有内存开销的，不过对于现在的机器来说，这点开销也不算什么。

最后，这是我学习 Go 的 slice 的一些理解与总结，由于能力有限，如果有理解不到位的地方，可以随时留言与我交流。

>Reference:
>
>1、**[build-web-application-with-golang](https://github.com/astaxie/build-web-application-with-golang)**
>
>2、**[go](https://github.com/golang/go)**



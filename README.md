# go-escape-analysis
escape analysis in golang



## Stack:
Stack is a block of memory allotted to each function. Memory is calculated on compile time.

## Heap:
Heap is a block of memory allotted demanded by your software at runtime.
Before goes into deep some terms must know

## Escape Analysis:

Gc compiler does global escape analysis across function and package boundaries. It checks a memory that it really needs to be allocated at a heap or it could be managed within a stack itself.

## Function Inlining:
Only short and simple functions are inlined. Function inlining is nothing but the whole code replaces the function call.

## Command For Escape Analysis:
```
go build -gcflags="-m"

```


## 1. Stack analysis 

````
package main

import (
	"fmt"
)

func init(){}

func main() {
	fmt.Println("main function")
	smallAllocation()
}
type smallStruct struct {
	a, b int64
	c, d float64
 }
 
 //go:noinline
 func smallAllocation() *smallStruct {
	return &smallStruct{}
 }

````

``
 $ go build -gcflags="-m"
 ``
 ```
 ./main.go:7:6: can inline init.0
./main.go:10:13: inlining call to fmt.Println
./main.go:20:9: &smallStruct literal escapes to heap
./main.go:10:14: "main function" escapes to heap
./main.go:10:13: []interface {} literal does not escape
<autogenerated>:1: .this does not escape
 ```

The meaning of escapes to the heap is variables needs to be shared across the function stack frames

## Example 2 

```
package main

import "fmt"
func main() {
   test()
}

func test() {
   fmt.Println("Called heapAnalysis", heapAnalysis())
   x := 45
   fmt.Println("Called x", x)
   fmt.Println("Called allocation ", new(1,2))
}
//go:noinline
func heapAnalysis() *int {
   data := 55 
   return &data
}

type alloc struct {
   a int32
   b int32
}

func new(a,b int32) *alloc {
   return &alloc{
      a : a,
      b : b,
   }
}

```
``
go build -gcflags="-m"
``
```
./main.go:25:6: can inline new
./main.go:9:15: inlining call to fmt.Println
./main.go:11:15: inlining call to fmt.Println
./main.go:12:41: inlining call to new
./main.go:12:15: inlining call to fmt.Println
./main.go:4:6: can inline main
./main.go:16:4: moved to heap: data
./main.go:9:16: "Called heapAnalysis" escapes to heap
./main.go:9:15: []interface {} literal does not escape
./main.go:11:16: "Called x" escapes to heap
./main.go:11:16: x escapes to heap
./main.go:11:15: []interface {} literal does not escape
./main.go:12:16: "Called allocation " escapes to heap
./main.go:12:41: &alloc literal escapes to heap
./main.go:12:15: []interface {} literal does not escape
./main.go:26:11: &alloc literal escapes to heap
<autogenerated>:1: .this does not escape
```


reference - https://mayurwadekar2.medium.com/escape-analysis-in-golang-ee40a1c064c1

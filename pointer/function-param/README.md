# 如何在函数内部修改外部结构体指针变量

## 前言
众所周知，Go的函数传参均为值传递（拷贝一份值传入函数体）。这就意味着，在函数内部对接受的参数进行任何的修改，都不会影响到原来的值。

```go
func IntFunc(x int){
	x++
	fmt.Println(x) // output: 2
}

func main(){
	x := 1
	MyFunc(x)
	fmt.Println(x) // output: 1
}
```

当然，一些引用类型（Map/Slice）除外。
```go
func SliceFunc(x []int) {
	x[0] = 2
	fmt.Println(x) // output: [2]
}

func main() {
	x := []int{1}
	MyFunc(x)
	fmt.Println(x) // output: [2]
}
```


## 坑在哪里
通常我们更多的会使用结构体，并且传入结构体指针。这样函数内部修改当前参数之后，外部结构体的值依然会发生变化。

### 常见情况
```go
type MyStruct struct{
	Age int
}

func StructFunc(s *MyStruct) {
	s.Age = 18
}

func main(){
	s := MyStruct{Age: 1}
	fmt.Println(s) // output: {1}
	StructFunc(&s)
	fmt.Println(s) // output: {18}
}
```
上述场景在开发过程中非常常见，只要基础过关，基本都没有问题。


### 但是
我相信你一定会碰到，当你需要修改`外部结构体指针`的场景。

先来一个业务场景中不太碰到，但比较容易理解的例子。
```go
func StructFunc2(s *MyStruct) {
	s = &MyStruct{Age: 18}
}

func main(){
	s := MyStruct{Age: 1}
	fmt.Println(s) // output: {1}
	StructFunc2(&s)
	fmt.Println(s) // output: {1}
}
```

上述代码就直接修改了`s`的值，但外部的结构体指针并不会发生任何改变。

刚开始我很困惑，为什么同样是对结构体指针的修改，前者修改属性会生效，后者直接修改指针不会生效呢？

事实上由于Go结构体的特性，在`s.Age`这个语句上很容易产生误解。**我们往往会认为只要修改变量`s`的相关值，就能修改外部结构体**。但结果是，你会发现，**若你要修改`s`本身，是不会生效的**。


后来我做了一些测试，有了大概的思索。实际上`s.Age = 18`这行代码的执行，应该被理解为`(*s).Age = 18`。通过结构体指针寻找到外部结构体，再修改其属性值。

因此若将上述代码修改为，就可以更改了。
```go
func StructFunc2(s *MyStruct) {
	*s = MyStruct{Age: 18}
}
```

而在常见业务中，我们往往外部声明了一个指针变量（多数情况可能为空）。

```go
func someInitial(s *MyStruct){
	*s = MyStruct{Age: 1} // panic
}

func main(){
	var p *MyStruct
	someInitial(p) 
	fmt.Println(p)
}
```
很显然，上述的代码会出现异常`panic: runtime error: invalid memory address or nil pointer dereference`。

那么问题来了，这类情况我们要怎么修改外部结构指针的值呢？答案是二级指针。

```go
func someInitial2(s **MyStruct) {
	*s = &MyStruct{Age: 1}
}

func main() {
	var p *MyStruct
	fmt.Println(p) // output: <nil>
	someInitial2(&p)
	fmt.Println(p) // output: &{1}
}
```

## 结论

所以我们平时在开发过程中，对于需要修改外部变量的函数传参应当注意些什么？

- 在可能条件下，尽量return修改后的变量。这几乎可以解决所有作用域引起的问题，虽然不良的代码习惯很可能造成大量内存逃逸，但耐不住开BUG率低呀。


- 对于值类型（除了Map/Slice之外，绝大部分Go的数据结构都为值类型）的传参，若要在函数内部修改该变量，则应当传入当前变量的指针。
  >例如：若要修改结构体，则应当传入结构体指针；若要修改结构体指针，则应当传入结构体指针的指针。

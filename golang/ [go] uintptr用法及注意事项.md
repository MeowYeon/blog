#  [go] uintptr用法及注意事项
## 起因
golang是强类型语言[TODO]，处于安全的考虑，仅对内置类型提供了强转操作，对某些操作产生了限制，比如指针运算。  
基于此golang提供了unsafe.Pointer类型与uintptr类型，简单介绍下这两种类型：  
unsafe.Pointer可以理解为C中的 void* 类型，是一种通用的指针类型。你可以通过该类型与所有指针类型进行强转。  
uintptr类型，我们知道指针本质就是保存指向变量地址的变量，简单说就是地址。不同架构的计算机所支持的地址空间不同，所以uintptr类型是依赖实现的。  
通过unsafe.Pointer与uintptr类型，就可以实现指针运算。

## uintptr
uintptr类型主要用作两个目的：
1. 获取指针保存的地址，一般用于打印
2. 配合unsafe.Pointer实现指针运算

## unsafe.Pointer
golang为unsafe.Pointer类型定义了四种特殊的操作：
1. 任意类型的指针转换到unsafe.Pointer类型
2. unsafe.Pointer类型转换到任意类型的指针
3. unsafe.Pointer类型转换到uintptr类型
4. uintptr类型转换到unsafe.Pointer类型

unsafe.Pointer主要应用有六种模式：  
1. 任意两种指针类型强转  
```go
type T1 int32
type T2 int64
t1 T1
t2 T2 = *(*T2)(unsafe.Pointer(&t1))
```
2. 转换为uintptr类型以获取指针中的地址  
注意：由于GC机制，unsafe.Pointer一旦转换为uintptr之后，便无法安全转换回unsafe.Pointer.
golang GC机制[TODO]主要依赖于分析对象树上的对象的引用，发现不被引用的对象后将其回收。而uintptr本身并不引用对象，所以当unsafe.Pointer转换为uintptr之后，unsafe.Pointer作用域失效，对象引用递减。这个时候对象是有可能被GC回收的，所以再次将uintptr转回unsafe.Pointer是不安全的。  
3. 借助uintptr实现指针运算  
最典型的用法，获取struct中某field地址：  
```go
type People struct {
    Age int
    Name string
}

var yeon People = People{Age:27,Name:"yeon"}
name := (*string)(unsafe.Pointer(uintptr(unsafe.Pointer(&yeon)) + unsafe.Offsetof(yeon.Name)))
```
注意：golang禁止越界访问，所以尝试越界访问为定义地址的指针会报错  
4. 由于syscall.Syscall调用参数是uintptr类型，所以为了保证调用过程中对象不会被回收，应该将unsafe.Pointer一并传给调用参数:  
```go
syscall.Syscall(SYS_READ, uintptr(fd), uintptr(unsafe.Pointer(p)), uintptr(n))
```
解释：因为函数调用过程中，编译器会将unsafe.Pointer一同压入调用参数栈，因此对象引用会一直存在，直到函数调用完成
5. 关于反射  
当反射返回uintptr类型时要尽快将其转为unsafe.Pointer类型，防止对象被回收，一般写成同一条表达式
6. 反射特例 reflect.SliceHeader 与 reflect.StringHeader
这两种类型的Data字段可以直接传入uintptr类型变量

# 参考链接
https://golangbyexample.com/understanding-uintptr-golang/  
https://golang.org/pkg/unsafe/#Pointer  推荐阅读

# timeline
> 2021.06.10 yeon.guo ShangHai YangPu

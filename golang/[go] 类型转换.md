# [go] 类型转换
golang的类型转换区分接口类型和具体类型，各自有不同的转换方式。

## 接口类型
接口类型是golang提供的特殊类型，是一种抽象类型。接口类型的类型转换通过断言方式来实现。
1. 类型断言
```go
// 该语法下，如果断言失败会直接panic
func f(x interface{}) {
    i := x.(int)
}
// 该语法下，如果断言失败可以通过bool类型的返回值确认
func f(x interface{}) {
    i, ok := x.(int)
}
```
2. 类型switch
```go
func f(x interface{}) {
    switch x.(type) {
        case nil:
        case int:
        case string:
        default:
    }
}
```
**注意**：断言不仅可以用来进行*接口类型*到*具体类型*的转换，还可以执行*接口类型*到*接口类型*的转换。

## 具体类型
golang中具体类型之间的转换通过强制类型转换实现。
1. 基础类型，应用强转语法
```go
int8 i = 1;
int16 j=int16(i);
```
2. 自定义类型，使用unsafe包实现，参见：[go] uintptr用法及注意事项
```go
type A struct {
    i int
}
type B struct {
    j int
}

a := A{i:1}
b := *(*B)unsafe.Pointer(&a)
```

# 参考链接
类型断言 https://golang.org/ref/spec#Type_assertions   
类型switch https://golang.org/ref/spec#Type_switches  
https://learnku.com/articles/42797

# timeline
> 2021.06.28 yeon.guo ShangHai YangPu

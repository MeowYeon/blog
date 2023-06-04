# [go] map介绍  
map是golang提供的哈希表实现，属于引用类型，即传值过程发生浅拷贝。  

## 构建map

map有两种构建方式make or initial：
```go
m := make(map[string]int)

m := map[string]int{}
```  
**注意**：仅声明变量不会为map分配存储空间，如下：
```go
var m map[string]int // m == nil
```
尝试读取未初始化的map返回零值，尝试向未初始化的map写入会panic  

## 基础操作

read，golang提供了两种读取方式
```go
m := map[string]int {
    "one": 1,
    "two": 2,
}

i := m["one"]
i, ok := m["one"]
```
因为尝试读取未写入的map键会返回值类型的零值，所以当写入的值中包含零值时，带bool值的读取方式可以用来判断元素是否存在  

write
```go
m := make(map[string]int)

m["one"] = 1
```

delete
```go
m := map[string]int {
    "one": 1,
    "two": 2,
}

delete(m, "one")
```

iterate
```go
m := map[string]int {
    "one": 1,
    "two": 2,
}

for key, value := range m {
    fmt.Println(key, value)
}
```

## 并发操作

map 不是并发安全的，当你需要在多协程中使用并发访问map，需要自己维护读写锁 sync.RWMutex 或者使用 sync.Map.  
相对来说，如果用户在自定义类型中使用map，通过自行维护读写锁比map本身支持锁性能会更好，这也是map不支持并发的考量之一。  

sync.Map[TODO] 针对两种使用场景做了优化：  
1. write once and only grow  
2. 当不同协程访问map的不同子集  

当使用以上两种场景时，使用sync.Map比自行维护map的读写锁会有更好的性能。  

## 使用案例

blog中记录了几种map使用案例，仅做记录：  
因为map的遍历操作不保证顺序，当需要确定顺序的map遍历集，可以通过对key排序并保存结果来实现
```go
import "sort"

var m map[int]string
var keys []int
for k := range m {
    keys = append(keys, k)
}
sort.Ints(keys)
for _, k := range keys {
    fmt.Println("Key:", k, "Value:", m[k])
}
```

当嵌套层数较多时，考虑自己的数据模型是否可以通过使用struct类型的key实现：  
下面案例中，当需要增加统计维度时，前者对代码侵入性较高
```go
hits := make(map[string]map[string]int)
func add(m map[string]map[string]int, path, country string) {
    mm, ok := m[path]
    if !ok {
        mm = make(map[string]int)
        m[path] = mm
    }
    mm[country]++
}
add(hits, "/doc/", "au")

type Key struct {
    Path, Country string
}
hits := make(map[Key]int)
hits[Key{"/", "vn"}]++
```

依赖map读取返回value零值减少代码：  
这里注意，同样作为引用类型的list不需要额外的make操作也可以直接写入
```go
type Person struct {
    Name  string
    Likes []string
}
var people []*Person

likes := make(map[string][]*Person)
for _, p := range people {
    for _, l := range p.Likes {
        likes[l] = append(likes[l], p)
    }
}
```

# 参考链接

https://blog.golang.org/maps  参考本文而成  
https://golang.org/doc/faq#atomic_maps  

# timeline

> 2021.06.26 yeon.guo ShangHai YangPu

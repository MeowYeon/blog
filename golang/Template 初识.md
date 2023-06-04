# Template 初识
## Template 初识
golang标准库包含两个template包：text/template，html/template，其中html/template相关调用可以对解析过程出现的特殊字符进行安全转义，保证不被注入攻击

### 值
template使用dot(.) 接收当前扫描到的变量，同时每个template的顶级作用域中包含一个const变量$代表当前模板接收到的变量。  
普通变量以$开头，初次赋值使用:=赋值  
模板变量都有其作用域，即声明位置到{{end}}
```go
func main() {
	t1 := template.New("test1")
	tmpl, _ := t1.Parse(
`
{{- define "T1"}}ONE {{println .}}{{end}}
{{- define "T2"}}{{template "T1" $}}{{end}}
{{- template "T2" . -}}
`)
	_ = tmpl.Execute(os.Stdout, "hello world")
}
```

### 流程控制
template包含三种流程控制结构：if，range，with
```go
{{if pipeline}} T1 {{end}}
{{if pipeline}} T1 {{else}} T0 {{end}}
{{if pipeline}} T1 {{else if pipeline}} T0 {{end}}
{{if pipeline}} T1 {{else}}{{if pipeline}} T0 {{end}}{{end}}

{{range pipeline}} T1 {{end}}
{{range pipeline}} T1 {{else}} T0 {{end}}
// range支持遍历时赋值
{{range $value := .}}
{{range $key,$value := .}}

{{with pipeline}} T1 {{end}}
{{with pipeline}} T1 {{else}} T0 {{end}}
```

### 函数
template包支持函数，方便在pipeline中定义操作，template预定义以下函数
```go
and
    返回第一个为空的参数或最后一个参数。可以有任意多个参数。
    and x y等价于if x then y else x

not
    布尔取反。只能一个参数。

or
    返回第一个不为空的参数或最后一个参数。可以有任意多个参数。
    "or x y"等价于"if x then x else y"。

print
printf
println
    分别等价于fmt包中的Sprint、Sprintf、Sprintln

len
    返回参数的length。

index
    对可索引对象进行索引取值。第一个参数是索引对象，后面的参数是索引位。
    "index x 1 2 3"代表的是x[1][2][3]。
    可索引对象包括map、slice、array。

call
    显式调用函数。第一个参数必须是函数类型，且不是template中的函数，而是外部函数。
    例如一个struct中的某个字段是func类型的。
    "call .X.Y 1 2"表示调用dot.X.Y(1, 2)，Y必须是func类型，函数参数是1和2。
    函数必须只能有一个或2个返回值，如果有第二个返回值，则必须为error类型。
```
基于比较的内置函数
```go
eq arg1 arg2：
    arg1 == arg2时为true
ne arg1 arg2：
    arg1 != arg2时为true
lt arg1 arg2：
    arg1 < arg2时为true
le arg1 arg2：
    arg1 <= arg2时为true
gt arg1 arg2：
    arg1 > arg2时为true
ge arg1 arg2：
    arg1 >= arg2时为true
```
同时，也可以自己指定自定义函数
```go
funcMap := template.FuncMap{
	"strupper": func(str string) string { return strings.ToUpper(str) },
}
t1 := template.New("test")
t1 = t1.Funcs(funcMap)
```

### 模块化
模板支持{{define "name"}} ... {{end}}用来定义模板  
也可以使用{{block "name" pipeline}} ... {{end}}在使用时指定模板默认值，即如果模板不存在，则使用block定义的模板
```go
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
        <title>Go Web Programming</title>
    </head>
    <body>
        {{ block "content" . }}
            <h1 style="color: blue;">Hello World!</h1>
        {{ end }}
    </body>
</html>
```

### 注释/空白
可以直接在模板中使用注释
``` {{/* comment */}} ```
扫描模板文件时，执行到没有输出的行时会生成空行，可以使用``` {{- ```和``` -}} ```去掉pipeline左边或者右边的空白(包含换行)

### 转义
开始提到html/template包可以自动转义模板解析出现的特殊字符，保证可以被安全解释，如果不想转义，可以使用以下方法：
```go
type CSS
type HTML
type JS
type URL

func process(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("tmpl.html")
	t.Execute(w, template.HTML(r.FormValue("comment")))
}
```

### 解析文件
Parse仅支持解析string，template包提供了以下接口解析模板文件：
```go
func (t *Template) ParseFiles(filenames ...string) (*Template, error)
// ParseGlob提供一个pattern选项，用来匹配模板名称
func (t *Template) ParseGlob(pattern string) (*Template, error)
```

### 参考链接
[用法详解](https://www.cnblogs.com/f-ck-need-u/p/10053124.html)
[深入剖析](https://www.cnblogs.com/f-ck-need-u/p/10035768.html)

## 时间线
> 2020.06.06 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

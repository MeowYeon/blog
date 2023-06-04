# go mod 初识
go mod为go官方推出的包管理工具，这里简单整理其使用

TODO https://go.dev/blog/using-go-modules

## 包管理工具
go mod之前依赖GOPATH, GOROOT, vendor进行编译
go mod之后可以跳出GOPATH依赖

以往包管理工具多依赖vendor进行设计，go mod依赖包版本信息，默认使用最新的tag版本，如果没有tag版本则使用最新commit生成版本信息

## 使用go mod
需要打开GO111MODULE: on, off, auto(当前目录不在GOPATH中并且有go.mod文件，才会使用go mod)
go mod拉取的包默认保存在GOPATH/pkg/mod目录下

使用go mod init生成go.mod文件
使用go build编译当前工程，会在go.mod文件中生成依赖信息

go mod可以生成版本相关的依赖信息，这样当其他人fetch你的工程编译时，可以保证使用相同的依赖进行编译，防止出现兼容问题
go mod默认使用当前目录的go.mod，如果没有发现，则依次向父目录寻找

### 实践
当需要指定版本时，可以使用`go mod -require=module@version`指定版本，执行后会更新go.mod文件中依赖信息

由于MVS存在，当工程需要同时依赖两个major版本module时，可以使用类似module/v2的模块名称，go build时会自动拉取指定major版本最新版本进行编译，同时不会破坏历史依赖信息

可以使用go mod -vendor生成vendor以兼容历史

## 参考链接
[初窥Go module](https://tonybai.com/2018/07/15/hello-go-module/)

## 时间线
> 2020.06.03 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

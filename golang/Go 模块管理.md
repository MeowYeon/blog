# Go 模块管理
经历了vendor，dep，go mod 工具已成为标准的模块管理工具

### 模块加载
开启gomod之后，执行go get, go build, go run 命令都会自动加载包到$GOPATH/pkg/mod目录下
通过go mod加载的包都会带版本标签

### golang.org/x/
无论go get还是go mod 比较典型的加载失败的包：golang.org/x/
因为墙的缘故，golang.org/x/一般是无法直接加载到的
#### github.com替换
最初的解决方案是从github.com/golang/下载同名包，然后进行替换
#### GOPROXY
gomod发布后一起发布了Module proxy protocal协议，用来为gomod加载的包搭建代理
通过本地GOPROXY变量可以指定该代理地址，默认地址国内访问不到，七牛云搭建的非营利go模块代理：https://goproxy.cn
#### GOSUMDB
与GOPROXY一起的还有GOSUMDB变量支持，该变量用来加载包之后对包做校验，防止加载的包被篡改

### 私有库加载
私有库因为不对外开发，所以走GOPROXY是无法加载的
#### GOPRIVATE
为此，golang提供了GOPRIVATE变量支持，为该变量指定私有库地址
配置GOPRIVATE之后，会从该地址加载私有库，并且跳过GOSUM校验
GOPRIVATE变量会对需要加载的包进行前缀匹配
```
加载私有库报错：
fatal: could not read Username for 'https://xxx': terminal prompts disabled
因为git默认使用https拉取源端库，而私有库一般配置ssh方式访问，所以需要修改git拉取方式未ssh
使用如下命令：
git config --global --add url."git@xxx:".insteadOf "https://xxx"
执行后，可以在~/.gitconfig文件中查看修改
```

## 参考链接
[go mod](post/go-mod.md)  
[通过github.com加载golang.org/x](https://blog.csdn.net/u011768994/article/details/78477143)  
[GOPROXY](https://segmentfault.com/a/1190000020293616)  
[GOPRIVATE](https://segmentfault.com/a/1190000021127791)  
[git 使用ssh方式加载](https://blog.csdn.net/jackgo73/article/details/90604180)  

## 时间线
> 2020.12.10 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

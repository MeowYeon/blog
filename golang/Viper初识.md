# Viper初识
## Viper初识
Viper是Golang配置解决方案，可以Cobra(Golang命令行解决方案)配合使用。
直观感觉Viper里面是一个map[string]interface{}结构存储左右的配置项，使用mapstructure将map转struct。

### Set/Get
Viper支持Set直接设置配置项，同时支持配置项别名
```go
viper.Set()
viper.RegisterAlias()
```
Viper支持不同类型的Get，以方便配置读取
```go
Get(key string) : interface{}
GetBool(key string) : bool
GetFloat64(key string) : float64
GetInt(key string) : int
GetIntSlice(key string) : []int
GetString(key string) : string
GetStringMap(key string) : map[string]interface{}
GetStringMapString(key string) : map[string]string
GetStringSlice(key string) : []string
GetTime(key string) : time.Time
GetDuration(key string) : time.Duration

IsSet(key string) : bool
AllSettings() : map[string]interface{}
```

### 命令行参数
通过cobra/pflag包，viper配置可以绑定命令行参数使用
```go
BindPFlag(key string, flag *pflag.Flag) error
BindPFlags(flags *pflag.FlagSet) error
```

### 环境变量
Viper支持获取配置项时直接读取ENV变量，不详述，请参考链接
```go
AutomaticEnv()
BindEnv(string...) : error
SetEnvPrefix(string)
SetEnvKeyReplacer(string...) *strings.Replacer
AllowEmptyEnv(bool)
```

### Read/Write
Viper支持读取本地配置文件，**这应当是最容易想到的使用方式**，同时也支持直接从io.Reader中读取配置
```go
SetConfigName(in string)
SetConfigType(in string)
AddConfigPath(in string)

ReadInConfig() error
ReadConfig(in io.Reader) error //从io.Reader中读取配置
MergeInConfig() error //合并当前viper中配置与磁盘中配置文件
```
值得注意的功能，Viper支持动态加载磁盘配置文件
```go
WatchConfig()
WatchRemoteConfig() error

OnConfigChange(run func(in fsnotify.Event)) //指定配置文件变化时回调函数
```

Viper支持将viper中配置写回磁盘，Safe接口均不会覆盖原文件，As接口均写入到参数提供的配置文件中
```go
WriteConfig() error
SafeWriteConfig() error
WriteConfigAs(filename string) error
SafeWriteConfigAs(filename string) error
```

### Remote Key/Value
Viper支持从remote加载配置，支持etcd，Consul，firestore，不详述，参考链接
```go
AddRemoteProvider(provider, endpoint, path string) error
ReadRemoteConfig() error
WatchRemoteConfig() error
```

### 默认值
```go
SetDefault(key string, value interface{})
```

### 优先级顺序
降序，符合直观感受
```go
显示调用Set设置值
命令行参数（flag）
环境变量
配置文件
key/value存储
默认值
```

### 其他功能
Viper支持获取子配置项
```go
Sub(key string) *Viper 
```
Viper也支持类似json的序列化功能，默认使用mapstructure进行反序列化，可以使用mapstructure为需要特殊指定的key指定tag
当遇到结构体嵌入时，可以在标签中指定squash自动解析到对应子结构，详见参考链接

## 参考链接
[Go语言配置管理神器——Viper中文教程](https://www.liwenzhou.com/posts/Go/viper_tutorial/)

## 时间线
> 2020.06.05 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

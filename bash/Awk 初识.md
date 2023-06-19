# Awk 初识
## 调用方式
awk 提供两种调用方式
```bash
# 命令行调用，bash脚本调用类同
awk '{print}' local.file
cat local.file | awk '{print}'

# awk脚本调用
awk -f print.awk local.file
cat local.file | awk -f print.awk
```

## 模式匹配
awk下模式匹配方式分为5种
```bash
# BEGIN, END
BEGIN { print "----start-----" }
END { print "----stop----" }
# 正则匹配
/regex/ { print }
# 正则range区间，从第一个正则开始到第二个正则匹配的区间
/regex-start/,/regex-stop/ { print }
# 表达式匹配
i > j { print }

# 复杂组合，使用 &&, ||, !组合表达式模式
i > j && j < k { print }
```

## 输出
awk 提供两种方式输出：
```bash
# 将var1, var2拼接到一起输出
print var1 var2...
# 将var1, var2用空格分隔输出
print var1, var2...

# 类似C语言格式化输出，默认不换行
printf "format" var1, var2...
```

## 变量
### 内置变量
awk默认包含以下内置变量
```bash
RS, ORS: 记录分隔符，输出时记录分隔符
FS, OFS: 字段分隔符，输出时字段分隔符
FILENAME: 处理的文件名
NF: 当前记录字段个数
NR: 当前记录行号，因为awk流式处理每次读取一条记录然后进行处理，所以NR只代表走到当前位置处理的记录个数
ARGC, ARGV: 参数个数以及参数数组，暂未用
```
### 自定义变量
awk中自定义变量不需要提前声明
若想从命令行传入值，可以通过-v 指定
```bash
awk -v key="value" -v key2="value2" '{print key, key2, $0}'
```
** PS. awk 支持数组变量，其行为像map，下标支持字符串 **

## 语句
awk 语法与C语言非常类似
```bash
# 分支
if (expression) {
    ...
} else {
    ...
}

switch (expression) {
    case value:
    ...;
    ...;
    case v1 or v2:
    ...;
    default:
    ...; 
}

# 循环
for (var in arr) {
    print var
}
for (i=0; i<n; i++) {
    ...;
}
while (expression) {
    ...;
    continue;
}
do {
    ...;
    break;
} while(expression)

PS. awk 支持break, continue语句
```

## 函数
### 内置函数
```bash
math: 
srand(): 设置随机种子
rand(): 获取随机数

string:
length(): 获取字符串长度，可用于数组
split(str, arr, dlm): 使用dlm分隔str，并将结果存储于arr中
sub(reg, new, str): 对str使用new替换由reg匹配首个位置
gsub(reg, new, str): 对str使用new替换由reg匹配**全部**位置
```
### 自定义函数
```bash
# 定义函数需要指定参数名称
function func_name(parm1, parm2...) {
    ...;
    return ret;
}
```

## awk脚本中调用bash命令
使用system调用，参数为需要执行的命令的字符串
```bash
awk '{system(date)}'
awk '{key="value"; system("echo "key)}'
```

## 参考链接
[linux 文本处理三剑客](https://www.cnblogs.com/along21/p/10366886.html)  
[awk 入门指南](https://awk.readthedocs.io/en/latest/chapter-one.html)  
[gawk 入门](https://www.ibm.com/developerworks/cn/education/aix/au-gawk/index.html)  
[awk 内置函数](https://www.runoob.com/w3cnote/awk-built-in-functions.html)

## 时间线
> 2020.10.21 yeon.guo ShangHai YangPu  
> 2023.05.28 yeon.guo ShangHai MinHang 迁移  

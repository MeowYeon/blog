# Pushd与Popd
## 比cd更方便的目录切换
目录切换时，常常需要在多个目录之间跳转
如果只有两个目录来回跳，使用 `cd -` 就可以了
如果有更多目录，就需要 `pushd` `popd` 操作

### 基本使用
```bash
# pushd 将当前目录压入栈，并切换到指定位置
/data/]$ pushd /yeon/
/yeon/ /data/
/yeon/]$ dirs
/yeon/ /data/

# popd 将当前目录出栈，目录切换到新的栈顶元素
/yeon/]$ popd
/data/
/data/]$ 

# pushd +n 支持跳转到栈中指定位置目录
# 栈中元素下标从0开始
# 栈中的元素会类似环形队列旋转
/dir1/]$ dirs
dir1 dir2 dir3 dir4
/dir3/]$ pushd +2
dir3 dir4 dir1 dir2

# popd +n 支持删除栈中元素
/dir1/]$ dirs
dir1 dir2 dir3 dir4
/dir1/]$ popd +2
dir1 dir2 dir4
/dir1/]$ 
```

### 其他用法
```
# pushd 不切换当前目录，将指定目录入栈
/dir1/]$ dirs
/dir1/ /dir2/ /dir3/
/dir1/]$ pushd -n /dir4/
/dir1/ /dir4/ /dir2/ /dir3
/dir1/]$ 

# dirs 清空栈
/dir1/]$ dirs
/dir1/ /dir2/ /dir3/
/dir1/]$ dirs -c
/dir1/]$ dirs
/dir1/
```

## 参考链接
[pushd popd使用](https://blog.csdn.net/muzilanlan/article/details/45564163)

## 时间线
> 2020.12.18 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

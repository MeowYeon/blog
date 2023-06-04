# 移除大文件头N行
# 移除大文件头N行

## sed -i '1,Nd' filename

## tail -n +N filename

使用tail比较快，原理应该还是拷贝文件只不过速度相对快，sed中应该是有内容的分析所以会比较慢。
有兴趣后续可以深入了解。

## 时间线
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

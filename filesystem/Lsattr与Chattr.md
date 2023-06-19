# Lsattr与Chattr
## 问题起源
对本地.ssh/authorized_keys 进行写操作时，报错：
**Operation not permitted**
第一反应检查权限，发现是赋予了写权限的
最后定位是文件的第二扩展文件属性赋予了禁止写的权限位

## 命令操作
lsattr 列出设备第二扩展文件属性
lsattr filename
i: 禁止写入，改名，删除，链接
a: 只允许追加写

chattr 修改设备第二扩展文件属性
使用方式与chmod类似

## 参考链接
[lsattr chattr命令详解](http://www.ha97.com/5172.html)  

## 时间线
> 2020.12.18 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

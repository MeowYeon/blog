# golang IDE支持vim
## [vim8 安装][1]

> 1. vim8 安装 python 支持，只要指定 python2 或者 python3 其中一个，同时指定有问题
> 2. 安装完成后，vim --version 可以查看编译选项
>    如果没有-Wl,-export-dynamic 选项，那么 vim 中执行 python 命令会报错 undefined symbol
>    原因是 vim 没有保留某些符号表，导致引用 python 动态库发生找不到符号，详见[undefined symbol][2]

## [vim-go 插件][3]

### [vim golang 插件大全][4]

### [deoplete 自动补全插件][5]

### [deoplete for golang][6]

[1]: https://github.com/ycm-core/YouCompleteMe/wiki/Building-Vim-from-source
[2]: https://blog.csdn.net/jq0123/article/details/1340839
[3]: https://github.com/fatih/vim-go/wiki/Tutorial#vim-go-tutorial
[4]: https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins
[5]: https://github.com/Shougo/deoplete.nvim
[6]: https://github.com/deoplete-plugins/deoplete-go

## 时间线
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

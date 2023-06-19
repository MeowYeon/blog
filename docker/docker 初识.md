# docker 初识
## docker 命令
```bash
# 拉取docker镜像
docker pull domain:port/user/mirror:tag
# 查看体积
docker system df
# 列出镜像
docker image ls [-f | -a | -q]
# 删除镜像
docker image rm <name>
# 批量删除镜像
docker image rm $(docker image ls -q)

# 制作镜像
# 1. docker commit (不推荐)
# 2. docker build
# 默认在context目录下寻找docker file，也可以使用-f指定
# context上下文，可以是本地路径，URL地址，tgz包文件
docker build -t <repo:tag> <context>
# 3. docker import
docker import [tgz | URL | -] <repo:tag>
# docker save | docker load (不推荐)
# eg. 镜像迁移
docker save <镜像名> | bzip2 | pv | ssh <用户名>@<主机名> 'cat | docker load'
```

## 时间线
> 2020.10.23 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

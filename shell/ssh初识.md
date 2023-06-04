# ssh初识
## ssh初识
ssh linux下远程登录工具

## ssh流程
ssh登录流程分为两步：
- 建立通道
- 登录验证

### 建立通道
建立TCP连接
服务端返回协议版本，客户端校验
服务端返回host public key，客户端校验
```
初次登录的机器，会提示：
The authenticity of host 'xxxx' can't be established.
RSA key fingerprint is SHA256:ssss.
Are you sure you want to continue connecting (yes/no)?
用户确认服务器指纹无误，输入yes继续登录
初次登录后，服务器公钥会被保存在ssh/.known_hosts文件中

这步用来防止中间人劫持，初次登录靠用户确认
```
```
如果服务器返回的公钥与本地ssh/.known_hosts中保存的记录不同，客户端会报错：
Host key verification failed
```

### 登录验证
#### 密码登录
客户端输入密码，在通道中加密传输到服务端验证
#### ssh免密登录
客户端发送密钥ID到服务端
服务端根据该ID在本地ssh/authorized_keys 查找对应的客户端公钥
服务端生成随机数R，使用客户端公钥加密后返回客户端
客户端使用私钥解密，并计算md5(R)，将结果返回服务端
服务端同样计算md5(R)，并校验客户端传过来的值

总结：服务端验证本地的客户端公钥与客户端的私钥是否是一对

## ssh命令
### ssh
远程登录命令，不解释
### ssh-keyscan
获取源端服务器公钥，用于建立通道阶段，服务器公钥验证
```
ssh-keyscan host
ssh-keyscan -f host.list
```
### ssh-copy-id
将本机公钥拷贝到远程服务器中
```
ssh-copy-id -i pubkey.file user@ip
```
### ssh 隧道
本地转发
```
# -L 指定本地转发地址
# 监听本地端口local_port的请求
# 将请求通过user@remote_ip机器转发到remote_ip:remote_port上
# -N 建立ssh连接后不执行任务命令
# -f 建立ssh连接后转入后端执行
ssh -N -f -L local_port:remote_ip:remote_port user@remote_ip
```
远程转发
```
# -R 指定远程转发地址
# 使远程机器user@remote_ip监听端口remote_port的请求
# 将请求通过user@remote_ip机器转发到local_ip:local_port上
ssh -N -f -R remote_port:local_ip:local_port user@remote_ip
```
命令讲解
```
# 监听本机8080端口
# 将请求通过root@10.0.0.2 机器将请求转发到10.0.0.3:9090
# 用来跳过10.0.0.1 与 10.0.0.2 之间的防火墙
[10.0.0.1]# ssh -L 8080:10.0.0.3:9090 root@10.0.0.2

# 监听本机8080端口
# 将请求通过root@10.0.0.2 机器将请求转发到localhost:9090
# 用来访问 10.0.0.2 上并不提供外网服务的服务
[10.0.0.1]# ssh -L 8080:localhost:9090 root@10.0.0.2
```

## 参考链接
[ssh 流程](https://blog.cuiyongjian.com/safe/ssh-create/)  
[ssh 端口转发](https://www.zsythink.net/archives/2450)  
[ssh-keyscan](https://liam.page/2018/01/24/ssh-keyscan/)  
[ssh-copy-id](https://wangchujiang.com/linux-command/c/ssh-copy-id.html)  
[Host key verification failed](https://www.thegeekdiary.com/how-to-fix-the-error-host-key-verification-failed/)  

## 时间线
> 2020.12.09 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

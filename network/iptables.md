# iptables
iptables 用来操作netfilter。通过iptables配置防火墙规则

## default policy
iptables 策略分三种：ACCEPT通, DROP堵
设置默认策略：iptables -P chain policy 

## chain 
iptables包含五条chain：INPUT, FORWARD, OUTPUT, PREROUTING, POSTROUTING
INPUT和OUTPUT比较常用，分别用于控制进来和出去的包，FORWARD 用来控制中转的包

## filter
过滤包参数常用五个：-s -d --sport --dport -p
-s 指定源ip，可以使用掩码
-d 指定目的端ip
--sport 指定源端口  --sport 10:20 开发10-20端口
--dport 指定目的端口
-p 指定协议 tcp/udp

-i input网卡
-o output网卡

-m match匹配
-m state 
-m string

## table
iptables 控制四张表
raw 高级功能，待后续
filter 包过滤
nat 地址转换
mangle 包修改

## action
动作支持：
ACCEPT
DROP
REJECT
REDIRECT
SNAT
DNAT
MASQUERADE IP伪装
LOG日志

## 参考链接：
[iptables命令说明](https://wangchujiang.com/linux-command/c/iptables.html)

## 时间线
> 2020.12.07 yeon.guo ShangHai YangPu
> 2023.05.28 yeon.guo ShangHai MinHang 迁移

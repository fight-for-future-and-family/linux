## ipset介绍
###### ipset是iptables的扩展,它允许你创建 匹配整个地址集合的规则。而不像普通的iptables链只能单IP匹配, ip集合存储在带索引的数据结构中,这种结构即时集合比较大也可以进行高效的查找，除了一些常用的情况,比如阻止一些危险主机访问本机，从而减少系统资源占用或网络拥塞,IPsets也具备一些新防火墙设计方法,并简化了配置

#### 创建一个ipset
```
ipset create xxx hash:net （也可以是hash:ip ，这指的是单个ip,xxx是ipset名称）
```
---
## 存储类型
> **前面例子中的 vader 这个集合是以 hash 方式存储 IP 地址，也就是以 IP 地址为 hash 的键。除了 IP 地址，还可以是网络段，端口号（支持指定 TCP/UDP 协议），mac 地址，网络接口名称，或者上述各种类型的组合。比如指定 hash:ip,port就是 IP 地址和端口号共同作为 hash 的键。查看 ipset 的帮助文档可以看到它支持的所有类型。**

##### hash:net
```
ipset create r2d2 hash:net
ipset add r2d2 1.2.3.0/24
ipset add r2d2 1.2.3.0/30 nomatch
ipset add r2d2 6.7.8.9
ipset test r2d2 1.2.3.2
```
##### hash:ip,port
```
ipset create c-3po hash:ip,port
ipset add c-3po 3.4.5.6,80
ipset add c-3po 5.6.7.8,udp:53
ipset add c-3po 1.2.3.4,80-86
```
---
## 自动过期，解封
>ipset 支持 timeout 参数，这就意味着，如果一个集合是作为黑名单使用，通过 timeout 参数，就可以到期自动从黑名单里删除内容

```
下面第一条命令创建了名为 obiwan 的集合，后面多加了 timeout 参数，值为 300，往集合里添加条目的默认 timeout 时间就是 300。
第三条命令在向集合添加 IP 时指定了一个不同于默认值的 timeout 值 60，那么这一条就会在 60 秒后自动删除。
ipset create obiwan hash:ip timeout 300
ipset add obiwan 1.2.3.4
ipset add obiwan 6.6.6.6 timeout 60

如果要重新为某个条目指定 timeout 参数，要使用 -exit 这一选项。
ipset -exist add obiwan 1.2.3.4 timeout 100
这样 1.2.3.4 这一条数据的 timeout 值就变成了 100，如果这里设置 300，那么它的 timeout，也就是存活时间又重新变成 300。
```
#### 想要默认条目不会过期（自动删除），又需要添加某些条目时加上 timeout 参数，可以在创建集合时指定 timeout 为 0。
```
#ipset create aa hash:ip,port timeout 0
#ipset add aa 1.1.1.1,udp:110  timeout 30
#ipset list aa
```
--- 
 
--- 
```
ipset del yoda x.x.x.x    # 从 yoda 集合中删除内容
ipset list yoda           # 查看 yoda 集合内容
ipset list                # 查看所有集合的内容
ipset flush yoda          # 清空 yoda 集合
ipset flush               # 清空所有集合
ipset destroy yoda        # 销毁 yoda 集合
ipset destroy             # 销毁所有集合
ipset save yoda           # 输出 yoda 集合内容到标准输出
ipset save                # 输出所有集合内容到标准输出
ipset restore             # 根据输入内容恢复集合内容
```
#### ipset默认可以存储65536个元素，使用maxelem指定数量
```
ipset create blacklist hash:net maxelem 1000000    #黑名单
ipset create whitelist hash:net maxelem 1000000    #白名单
```
#### 查看已创建的ipset
```
ipset list
```
#### 加入一个名单ip
```
ipset add blacklist 10.60.10.xx
```
#### 去除名单ip
```
ipset del blacklist 10.60.10.xx
```
#### 创建防火墙规则，与此同时，allset这个IP集里的ip都无法访问80端口（如：CC攻击可用）
```
iptables -I INPUT -m set --match-set setname src -p tcp --destination-port 80 -j DROP
```
### 将ipset规则保存到文件
```
ipset save blacklist -f blacklist.txt
ipset save whitelist -f whitelist.txt
```
#### 删除ipset
```
ipset destroy blacklist
ipset destroy whitelist
```
#### 导入ipset规则
```
ipset restore -f blacklist.txt
ipset restore -f whitelist.txt
```
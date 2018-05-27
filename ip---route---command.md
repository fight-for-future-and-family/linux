### 实例
```
ip link show                     # 显示网络接口信息
ip link set eth0 upi             # 开启网卡
ip link set eth0 down            # 关闭网卡
ip link set eth0 promisc on      # 开启网卡的混合模式
ip link set eth0 promisc offi    # 关闭网卡的混个模式
ip link set eth0 txqueuelen 1200 # 设置网卡队列长度
ip link set eth0 mtu 1400        # 设置网卡最大传输单元
ip addr show     # 显示网卡IP信息
ip addr add 192.168.0.1/24 dev eth0 # 设置eth0网卡IP地址192.168.0.1
ip addr del 192.168.0.1/24 dev eth0 # 删除eth0网卡IP地址

ip route show # 显示系统路由
ip route add default via 192.168.1.254   # 设置系统默认路由
ip route list                 # 查看路由信息
ip route add 192.168.4.0/24  via  192.168.0.254 dev eth0 # 设置192.168.4.0网段的网关为192.168.0.254,数据走eth0接口
ip route add default via  192.168.0.254  dev eth0        # 设置默认网关为192.168.0.254
ip route del 192.168.4.0/24   # 删除192.168.4.0网段的网关
ip route del default          # 删除默认路由
ip route delete 192.168.1.0/24 dev eth0 # 删除路由
```
### route 命令使用举例
##### 添加到主机的路由 
```
# route add -host 192.168.1.2 dev eth0 
# route add -host 10.20.30.148 gw 10.20.30.40     #添加到10.20.30.148的网管
```
##### 屏蔽一条路由
```
# route add -net 224.0.0.0 netmask 240.0.0.0 reject
```
##### 添加到网络的路由 
```
# route add -net 10.20.30.40 netmask 255.255.255.248 eth0   #添加10.20.30.40的网络
# route add -net 10.20.30.48 netmask 255.255.255.248 gw 10.20.30.41 #添加10.20.30.48的网络
# route add -net 192.168.1.0/24 eth1
```
##### 添加默认路由 
```
# route add default gw 192.168.1.1
```
##### 删除路由 
```
# route del -host 192.168.1.2 dev eth0:0
# route del -host 10.20.30.148 gw 10.20.30.40
# route del -net 10.20.30.40 netmask 255.255.255.248 eth0
# route del -net 10.20.30.48 netmask 255.255.255.248 gw 10.20.30.41
# route del -net 192.168.1.0/24 eth1
# route del default gw 192.168.1.1
```
### 设置包转发
##### 在 CentOS 中默认的内核配置已经包含了路由功能，要配置和调整内核参数可以使用 sysctl 命令。例如：要开启 Linux 内核的数据包转发功能可以使用如下的命令。
```
# sysctl -w net.ipv4.ip_forward=1
```
##### 这样设置之后，当前系统就能实现包转发，但下次启动计算机时将失效。为了使在下次启动计算机时仍然有效，需要将下面的行写入配置文件/etc/sysctl.conf
```
# vi /etc/sysctl.conf
net.ipv4.ip_forward = 1
```
##### 用户还可以使用如下的命令查看当前系统是否支持包转发。
```
# sysctl net.ipv4.ip_forward
```

---
### linux 多网卡同网段IP配置方法
##### 笨方法是写路由表，但是会导致一个通一个不通，两个网卡不能同时使用，只要像windows那样加上跳点就行了，命令如下，加到 /etc/rc.local里 实现的是以源地址来进行路由，而不是传统的以目的地址路由。。。 如果同一个网段，一个是接前段的负载，一个是接数据库。如果是一个网段的话，他会有冲突的。
```
ifconfig  eth0 IP地址1 netmask 子网掩码 up

ifconfig  eth1 IP地址2 netmask 子网掩码 up

route del default

ip route add default via 网关

ip route add default via 网关 dev eth0 src IP地址1 table 100

ip route add default via 网关 dev eth1 src IP地址2 table 200

ip rule add from IP地址1 table 100

ip rule add from IP地址2 table 200
```

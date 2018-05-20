## Linux 网桥配置命令：brctl
Linux网关模式下将有线LAN和无线LAN共享网段实现局域网内互联:
思路其实很简单：就是将虚拟出一个bridge口，将对应的有线LAN和无线LAN都绑定在这个虚拟bridge口上，并给这个bridge口分配一个地址，其他子网微机配置网关为bridge口的地址便可以了。当然，因为是设备是网关模式，路由和nat也是必须的了。如果设备本身便是网桥模式，那么路由和nat便可以省掉了。
```shell
    brctl addbr bridge
    brctl addif bridge eth0
    brctl addif bridge ath0
    ifconfig eth0  0.0.0.0
    ifconfig bridge 10.0.0.1 netmask 255.255.255.0 up
    添加iptables -t nat -A POSTROUTING -o eth1 -j SNAT --to 192.168.2.173
    将有线和无线都设置为10.0.0.*网段，即可通过网上邻居进行访问
```
 
>当然了，要是Linux可以工作在网桥模式，必须安装网桥工具bridge-utils，运行命令：
yum install bridge-utils
或者下载bridge-utils-1.4.tar.gz进行安装，步骤如下：
编译安装bridge-utils

- （1）进入到/usr/src 目录下，下载bridge-utils-1.4.tar.gz ：
```shell
#cd /usr/src
#wget http://launchpad.net/bridgeutils/
main/1.4/+download/bridge-utils-
1.4.tar.gz
```
- （2）解压缩：
```shell
#tar zxvf bridge-utils-1.4.tar.gz
进入bridge-utils-1.4目录：
#cd bridge-utils-1.4
```
- （3）编译安装：
```shell
#autoconf
生成configure文件：
#./configure
#make
#make install
编译安装完成。最后将命令brctl复制到/sbin下：
#cp/usr/local/sbin/brctl/sbin
```
 
=========================================================================
 
**下面是参考的一片文章：**
 
有五台主机。其中一台主机装有linux ，安装了网桥模块，而且有四块物理网卡，分别连接同一网段的其他主机。我们希望其成为一
个网桥，为其他四台主机(IP分别为192.168.1.2 ，192.168.1.3，192.168.1.4，192.168.1.5) 之间转发数据包。同时，为了方便管
理，希望网桥能够有一个IP（192.168.1.1），那样管理员就可以在192.168.1.0/24网段内的主机上telnet到网桥，对其进行配置，
实现远程管理。

- 前一节中提到，网桥在同一个逻辑网段转发数据包。针对上面的拓扑，这个逻辑网段就是192.168.1.0/24网段。我们为这个逻辑网段一个名称，br0。首先需要配置这样一个逻辑网段。
\# brctl addbr br0                    (建立一个逻辑网段，名称为br0)
实际上，我们可以把逻辑网段192.168.1.0/24看作使一个VLAN ，而br0则是这个VLAN的名称。
 
- 建立一个逻辑网段之后，我们还需要为这个网段分配特定的端口。在Linux中，一个端口实际上就是一个物理网卡。而每个物理网卡
的名称则分别为eth0，eth1，eth2，eth3。我们需要把每个网卡一一和br0这个网段联系起来，作为br0中的一个端口。
\# brctl addif br0 eth0               (让eth0成为br0的一个端口)
\# brctl addif br0 eth1               (让eth1成为br0的一个端口)
\# brctl addif br0 eth0               (让eth2成为br0的一个端口)
\# brctl addif br0 eth3               (让eth3成为br0的一个端口)
 
- 网桥的每个物理网卡作为一个端口，运行于混杂模式，而且是在链路层工作，所以就不需要IP了。
\# ifconfig eth0 0.0.0.0
\# ifconfig eth1 0.0.0.0
\# ifconfig eth2 0.0.0.0
\# ifconfig eth3 0.0.0.0
 
- 然后给br0的虚拟网卡配置IP：192.168.1.1。那样就能远程管理网桥。
\# ifconfig br0 192.168.1.1
 给br0配置了IP之后，网桥就能够工作了。192.168.1.0/24网段内的主机都可以telnet到网桥上对其进行配置。
 
以上配置的是一个逻辑网段，实际上Linux网桥也能配置成多个逻辑网段(相当于交换机中划分多个VLAN)。
 
 
 
### [另外一篇有助理解的文章](http://www.2cto.com/os/201202/118320.html)    
  
``` shell  
[root@xenserver ~]# brctl --help
Usage: brctl [commands]
commands:
        addbr           <bridge>                add bridge
        delbr           <bridge>                delete bridge
        addif           <bridge> <device>       add interface to bridge
        delif           <bridge> <device>       delete interface from bridge
        setageing       <bridge> <time>         set ageing time
        setbridgeprio   <bridge> <prio>         set bridge priority
        setfd           <bridge> <time>         set bridge forward delay
        sethello        <bridge> <time>         set hello time
        setmaxage       <bridge> <time>         set max message age
        setpathcost     <bridge> <port> <cost>  set path cost
        setportprio     <bridge> <port> <prio>  set port priority
        show                                    show a list of bridges
        showmacs        <bridge>                show a list of mac addrs
        showstp         <bridge>                show bridge stp info
        stp             <bridge> {on|off}       turn stp on/off
        addbr bridge的名称  #添加bridge；
        delbr bridge的名称              #删除bridge；
        addif bridge的名称device的名称#添加接口到bridge；
        delif bridge的名称device的名称#从bridge中删除接口
        setageing bridge的名称时间     #设置老化时间，即生存周期
        setbridgeprio bridge的名称 优先级#设置bridge的优先级
        setfd bridge的名称时间         #设置bridge转发延迟时间
        sethello bridge的名称时间      #设置hello时间
        setmaxage bridge的名称时间     #设置消息的最大生命周期
        setpathcost bridge的名称 端口 权重#设置路径的权值
        setportprio bridge的名称 端口 优先级#设置端口的优先级
        show     #显示bridge列表
        showmacs bridge的名称  #显示MAC地址
        showstp  bridge的名称           #显示bridge的stp信息
        stp bridge的名称{on|off}       #开/关stp
```  

设置linux让网桥运行    配置网桥  
-  1 我们需要让linux知道网桥，首先告诉它，我们想要一个虚拟的以太网桥接口：（这将在主机bridge上执行，不清楚的看看测试场景）
root@bridge:~> brctl addbr br0

-  2 其次，我们不需要STP(生成树协议)等。因为我们只有一个路由器，是绝对不可能形成一个环的。我们可以关闭这个功能。（这样也可以减少网络环境的数据包污染）：
root@bridge:~> brctl stp br0 off

-  3 经过这些准备工作后，我们终于可以做一些立竿见影的事了。我们添加两个（或更多）以太网物理接口，意思是：我们将他们附加到刚生成的逻辑（虚拟）网桥接口br0上。
root@bridge:~> brctl addif br0 eth0
root@bridge:~> brctl addif br0 eth1

-  4 现在，原来我们的两个以太网物理接口变成了网桥上的两个逻辑端口。那两个物理接口过去存在，未来也不会消失。要不信的话，去看看好了。.现在他们成了逻辑网桥设备的一部分了，所以不再需要IP地址。下面我们将这些IP地址释放掉  
```shell
root@bridge:~> ifconfig eth0 down
root@bridge:~> ifconfig eth1 down
root@bridge:~> ifconfig eth0 0.0.0.0 up
root@bridge:~> ifconfig eth1 0.0.0.0 up
```

好了！我们现在有了一个任何IP地址都没有的box w/o了。好了，这下如果你想通过TP配置你的防火墙或路由器的话，你就只能通过本地的控制端口了。你不会告诉我你的机器上连串行端口都没有吧？
注：上面红色部分其实是可选的，在试验中，我发现，就算不把原有的网卡地址释放掉，网桥也能工作！但是，为了更规范，或者说
为了避免有什幺莫名其妙的问题，最好还是按要求做，执行这四步吧！
 
- 5 最后，启用网桥
root@bridge:~> ifconfig br0 up
可选： 我们给这个新的桥接口分配一个IP地址
root@bridge:~> ifconfig br0 10.0.3.129
或者把最后这两步合成一步：
root@bridge:~> ifconfig br0 10.0.3.129 up
就是多一个up!
 
这下我们做完了 。
 

**关闭网桥命令**
 
     brctl delif ena eth1;
     brctl delif ena eth0;
     ifconfig ena down;
     brctl delbr ena;
 
 
==================================================
 
什么是网桥
网桥是一种在链路层实现中继，对帧进行转发的技术，根据MAC分区块，可隔离碰撞，将网络的多个网段在数据链路层连接起来的网络设备。
Linux 网桥配置命令：brctl
在Linux中配置网络一般使用 brctl 命令，使用此命令首先要安装：bridge-utils软件包。
```shell
[inbi@debian~]#apt-get install bridge-utils
[inbi@debian~]#modprobe bridge
[inbi@debian~]#echo "1">/proc/sys/net/ipv4/ip_forward
安装bridge-utils软件包，并加载bridge模块和开启内核转发。

[inbi@debian~]#brctl
\#直接输入brctl命令将显示帮助信息！
```   

```shell
增加网桥
[inbi@debian~]#brctl addbr br0
\#增加一个网桥
[inbi@debian~]#ifconfig eth0 0.0.0.0 promisc
[inbi@debian~]#ifconfig eth1 0.0.0.0 promisc
[inbi@debian~]#brctl addif br0 eth0 eth1
\#将两块已有的网卡添加到网桥，此时这两个网卡工作于混杂模式，所以不需要IP了，因为网桥是工作在链路层的。
[inbi@debian~]#brctl show
\#查看已有网桥
你也可以为 br0 设置一个IP，已访问这台机器。
[inbi@debian~]#ifconfig br0 10.10.1.1 netmask 255.255.0.0 up
删除网桥
[inbi@debian~]#brctl delif br0 eth0 eth1
\#增加网桥中的接口
[inbi@debian~]#brctl delbr br0
\#删除网桥
关闭生成树

[inbi@debian~]#brctl stp br0 off
\#关闭生成树协议，减少数据包污染，因为我这里只有一个路由器哦！
配置桥开机激活
 
[inbi@debian~]#echo "modprobe bridge">>/etc/rc.local
\#开机加载 bridge 模块，或者echo "bridge">>/etc/modules
[inbi@debian~]#cp /etc/network/interfaces /etc/network/interfaces.default
\#备份下，方便以后使用啊！
[inbi@debian~]#vim /etc/network/interfaces
auto lo eth0 eth1 br0
iface lo inet loopback
iface br0 inet static
    address 10.10.10.1
    netmask 255.255.0.0
    gateway 10.10.10.254
    pre-up ip link set eth0 promisc on
    pre-up ip link set eth1 promisc on
    pre-up echo "1">/proc/sys/net/ipv4/ip_forward
    bridge_ports eth0 eth1
\#配置eth0 eth1 br0开机启动，eth0，eth1未设置IP信息，在启动br0网卡时，开启了eth0，eth1的混杂模式，并桥接了它们。
```
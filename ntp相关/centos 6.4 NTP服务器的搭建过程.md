##    [设置NTP Server前的准备](https://blog.csdn.net/iloli/article/details/6431757)

###### 其实这个标题应该改为设置"NTP Relay Server"前的准备更加合适. 因为不论我们的计算机配置多好运行时间久了都会产生误差,所以不足以给互联网上的其他服务器做NTP Server. 真正能够精确地测算时间的还是原子钟. 但由于原子钟十分的昂贵,只有少部分组织拥有, 他们连接到计算机之后就成了一台真正的NTP Server. 而我们所要做的就是连接到这些服务器上同步我们系统的时间,然后把我们自己的服务器做成NTP Relay Server再给互联网或者是局域网内的用户提供同步服务

#### 1  架设一个NTP Relay Server其实非常简单,我们先把需要的RPM包装上
-  是否已经安装了NTP包可以用这条命令来确定：  

```shell
[root@NTPser ~]# rpm -qa | grep ntp
ntp-4.2.2p1-9.el5_4.1
chkfontpath-1.10.1-1.1

出现以上代码则表示已安装NTP包，否则用下面方法安装：
代码:
# rpm -ivh ntp-4.2.2p1-5.el5.rpm
```   

#### 2 接下来我们就要找到在互联网上给我们提供同步服务的NTP Server      

-  http://www.pool.ntp.org 是NTP的官方网站,在这上面我们可以找到离我们城市最近的NTP Server. NTP建议我们为了保障时间的准确性,最少找两个个NTP Server
那么比如在英国的话就可以选择下面两个服务器   
0.uk.pool.ntp.org
1.uk.pool.ntp.org
它的一般格式都是number.country.pool.ntp.org  

-  第二步要做的就是在打开NTP服务器之前先和这些服务器做一个同步,使得我们机器的时间尽量接近标准时间. 
这里我们可以用ntpdate命令手动更新时间   
       
``` shell
# ntpdate 0.uk.pool.ntp.org
 6 Jul 01:21:49 ntpdate[4528]: step time server 213.222.193.35 offset -38908.575181 sec
# ntpdate 0.pool.ntp.org
 6 Jul 01:21:56 ntpdate[4530]: adjust time server 213.222.193.35 offset -0.000065 sec
```   

-  假如你的时间差的很离谱的话第一次会看到调整的幅度比较大,所以保险起见可以运行两次. 那么为什么在打开NTP服务之前先要手动运行同步呢?
  1. 因为根据NTP的设置,如果你的系统时间比正确时间要快的话那么NTP是不会帮你调整的,所以要么你把时间设置回去,要么先做一个手动同步
  2. 当你的时间设置和NTP服务器的时间相差很大的时候,NTP会花上较长一段时间进行调整.所以手动同步可以减少这段时间

#### 3 配置和运行NTP Server
** 现在我们就来创建NTP的配置文件了, 它就是/etc/ntp.conf. 我们只需要加入上面的NTP Server和一个driftfile就可以了 **
```shell
# vi /etc/ntp.conf

server 210.72.145.44     #这是中国国家授时中心的IP
server 0.uk.pool.ntp.org
server 1.uk.pool.ntp.org
                                     
fudge 127.127.1.0 stratum 0  stratum 
这行是时间服务器的层次。设为0则为顶级，如果要向别的NTP服务器更新时间，请不要把它设为0
driftfile /var/lib/ntp/ntp.drift

非常的简单. 接下来我们就启动NTP Server,并且设置其在开机后自动运行
代码:
# /etc/init.d/ntpd start
# chkconfig --level 35 ntpd on
```
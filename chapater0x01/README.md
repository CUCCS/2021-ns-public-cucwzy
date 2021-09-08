 # 实验一：基于VirtualBox的网络攻防基础环境搭建
 ## 实验目的
 - 掌握 VirtualBox 虚拟机的安装与使用
 - 掌握 VirtualBox 的虚拟网络类型和按需配置
 - 掌握 VirtualBox 的虚拟硬盘多重加载

 ## 实验环境
 - VirtualBox 虚拟机
 - 攻击者主机（Attacker）：Kali Rolling 2020.3
 - 网关（Gateway, GW）：Debian Buster
 - 靶机（Victim）： xp-sp3 / Kali

 ## 实验要求
 - [ √ ] 靶机可以直接访问攻击者主机
 - [ √ ] 攻击者主机无法直接访问靶机
 - [ √ ] 网关可以直接访问攻击者主机和靶机
 - [ √ ] 靶机的所有对外上下行流量必须经过网关
 - [ √ ] 所有节点均可以访问互联网

## 实验过程
- 虚拟硬盘配置成多重加载

我们选择好需要进行多重加载配置的虚拟机之后，先点击左上角的管理，之后再点击虚拟介质管理，选择好相应的虚拟硬盘之后将其类型从普通改为多重加载，之后再点击应用释放盘片，多重加载就配置好了。

![](img/1.png)

![](img/2.png)

- 配置虚拟机
```
配置两台kail:攻击者kail-attacker, 靶机kail-victim-1

配置两台xp:靶机xp-victim-1, 靶机xp-victim-2

配置两台Debian:网关Debian-gateway, 靶机Debian-victim-2
```

![](img/3.png)

## 网络配置

- 1.构建一个如下的网络拓扑：

![](img/4.png)

- 2.配置网关Debian-gateway

我们需要配置4块网卡：网络地址转换NAT，仅主机Host-Only网络，内部网络intnet1,内部网络intnet2

![](img/5.png)

打开虚拟机Debian-gateway，依次执行以下代码：

```
#用户切换
su

#修改配置文件
vi /etc/network/interfaces

#重启
/sbin/ifup enp0s9
/sbin/ifup enp0s10
sudo systemctl restart networking

#安装dnsmasq
apt-get update  
apt-get install dnsmasq 

#修改/etc/dnsmasq.d/gw-enp09.conf
interface=enp0s9
dhcp-range=172.16.111.10,172.16.111.150,240h


#修改/etc/dnsmasq.d/gw-enp10.conf
interface=enp0s10
dhcp-range=172.16.222.10,172.16.222.150,240h

#备份dnsmasq.conf文件
cp dnsmasq.conf dnsmasq.conf.bak

#修改dnsmasq.conf文件
#log-dhcp--->log-dhcp
#log-queries--->log-queries
#在log-queries下面加一条命令
log-facility=/var/log/dnsmasq.log

#重启dnsmasq
/etc/init.d/dnsmasq restart
```

之后执行ip a有结果如下：

![](img/6.png)

- 3.攻击者kail-attacker配置

网卡使用NAT网络

![](img/7.png)

![](img/8.png)

- 4.配置四个靶机

(1)xp-victim-1的网卡设置为内部网络intnet1

![](img/9.png)

ip地址(172.16.111.139)为手动指定，另外关闭防火墙（为了网关可以访问xp靶机）

![](img/10.png)

![](img/33.png)

(2)xp-victim-2的网卡设置为内部网络intnet2

![](img/11.png)

ip地址(169.254.95.35)为手动指定，另外关闭防火墙（为了网关可以访问xp靶机）

![](img/12.png)

(3)kali-victim-1的网卡设置为内部网络intnet1

![](img/13.png)

![](img/14.png)

(4)Debian-victim-2的网卡设置为内部网络intnet2

![](img/15.png)

![](img/16.png)

归纳各个系统的ip地址：
```
Debian-getway:10.0.2.15/24,192.168.56.113/24,172.16.111.1/24,172.16.222.1/24

kali-victim-1:172.16.111.111/24

xp-victim-1:172.16.111.139

xp-victim-2:172.16.222.121

Debian-victim-2:172.16.222.102/24

kali-attacker:10.0.2.15/24
```

# 网络连通性测试

1.靶机可以直接访问攻击者主机

(1)kali-victim-1访问kail-attacker

![](img/18.png)

(2)Debain-victim-2访问kali-attacker

![](img/19.png)

(3)xp-victim-1访问kali-attacker

![](img/20.png)

(4)xp-victim-2访问kali-attacker

![](img/21.png)

2.攻击者主机无法直接访问靶机

![](img/17.png)

3.网关可以直接访问攻击者主机和靶机

![](img/22.png)

![](img/23.png)

![](img/24.png)

4.靶机的所有对外上下行流量必须经过网关

在网关上安装tcpdump与tmux，并对对应网卡进行监控。在各个节点上访问互联网，观察捕获到了上下行的包，说明靶机的所有对外上下行流量必须经过网关。

```
apt insatll tcpdump
apt install tmux
/usr/sbin/tcpdump -i enp0s8 # etc
```

![](img/31.png)

![](img/32.png)

之后将捕获到的数据包拷贝到主机，并使用Wireshark进行查看与分析。

5.所有节点均可以访问互联网

kali-victim-1访问互联网：

![](img/25.png)

kali-attacker访问互联网：

![](img/26.png)

xp-victim-1访问互联网：

![](img/27.png)

xp-victim-2访问互联网：

![](img/28.png)

Debian-gateway访问互联网：

![](img/29.png)

Debian-victim-2访问互联网：

![](img/30.png)

## 参考资料

- [基于 VirtualBox 的网络攻防基础环境搭建](https://c4pr1c3.github.io/cuc-ns/chap0x01/exp.html)
- [Virtualbox多重加载](https://blog.csdn.net/Jeanphorn/article/details/45056251)
- [使用adduser命令在Debian Linux中创建用户](https://blog.csdn.net/Aria_Miazzy/article/details/84790364)
- [2020-ns-public-Crrrln](https://github.com/CUCCS/2020-ns-public-Crrrln/blob/chap0x01/chap0x01/实验报告.md)

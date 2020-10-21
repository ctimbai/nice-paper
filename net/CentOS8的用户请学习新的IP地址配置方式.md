> 21 世纪最宝贵的是时间和精力，人们对于碎片化阅读越来越挑剔，既不想花太多时间，又想学到知识，而摆在眼前的长篇大论一来废话太多，二来耗时耗力，看后还是两眼发懵，所以我们力求在五分钟之内讲懂一个知识点，不贪多、不废话，关注我们，让网络不再难懂。



上文讲了 Ubuntu，本文讲讲 CentOS 8 及以上版本配置 IP 的方法。



### Centos/Redhat(8.x) 配置 IP 方法

> 说明：CentOS 8 是新发布的系统（发布时间：2019.9），IP 配置方式和以前版本不一样。使用 NetworkManager工具配置。
>
> 而以前的版本时通过修改配置文件来配置，并由network.service 提供服务。
>
> CentOS 8 已废弃 network.service，默认只能通过NetworkManager.service 提供的 nmcli 命令修改网络配置
>
> 当然如果希望 8 版本以后支持修改配置文件的方式，需要安装 network.service
> `yum install network-scripts`



配置静态 IP：

```sh
# 创建 eth0 网卡配置信息，包含：指定永久静态IP、网关、并ifup启动
nmcli connection add type ethernet con-name eth0 ifname eth0 ipv4.addresses 192.168.1.5/24 ipv4.gateway 192.168.1.1 ipv4.method manual
```



配置动态 IP：

```sh
# 创建 eth0 网卡配置信息，指定动态获取IP，并ifup启动
nmcli connection add type ethernet con-name eth0 ifname ens33 ipv4.method auto
```



修改IP（非交互式）:

```sh
nmcli connection modify eth0 ipv4.addresses 192.168.1.6/24
nmcli connection up eth0 #相当于ifup eth0
```



修改 IP（交互式）：

```sh
# nmcli connection edit eth0
nmcli> goto ipv4.addresses
nmcli ipv4.addresses> change
Edit 'addresses' value: 192.168.1.7/24
Do you also want to set 'ipv4.method' to 'manual'? [yes]: yes
nmcli ipv4.addresses> back
nmcli ipv4> save
nmcli ipv4> activate
nmcli ipv4> quit
```



查看 ip（类似于`ifconfig`、`ip addr`）:

```sh
# nmcli
```



这里也说下之前版本的配置方法，让大家有个对比。

### Centos/Redhat (5.x, 6.x, 7.x) 配置 IP 方法

修改对应网卡的 IP 地址的配置文件

```sh
[root@centos]# vi /etc/sysconfig/network-scripts/ifcfg-eth0
```



修改以下内容：

```sh
DEVICE=eth0 #描述网卡对应的设备别名，例如ifcfg-eth0的文件中它为eth0
BOOTPROTO=static #设置网卡获得ip地址的方式，可能的选项为static，dhcp或bootp，分别对应静态指定的 ip地址，通过dhcp协议获得的ip地址，通过bootp协议获得的ip地址
BROADCAST=192.168.0.255 #对应的子网广播地址
HWADDR=00:07:E9:05:E8:B4 #对应的网卡物理地址
IPADDR=12.168.1.2 #如果设置网卡获得 ip地址的方式为静态指定，此字段就指定了网卡对应的ip地址
NETMASK=255.255.255.0 #网卡对应的网络掩码
NETWORK=192.168.1.0 #网卡对应的网络地址
GATEWAY=192.168.1.1 #网关地址
DNS1=114.114.114.114 #DNS地址
DNS2=8.8.8.8 
IPV6INIT=no # 禁用ipv6启动
IPV6_AUTOCONF=no # 禁用ipv6自动配置
ONBOOT=yes #系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备
```

其中，包括 IP、网关、DNS 等



网关地址可以通过上面的方式配置，也可以修改下面网关的配置文件：

```sh
[root@centos]# vi /etc/sysconfig/network
```



修改以下内容:  

```sh
NETWORKING=yes #(表示系统是否使用网络，一般设置为yes。如果设为no，则不能使用网络，而且很多系统服务程序将无法启动)
HOSTNAME=centos #(设置本机的主机名，这里设置的主机名要和 /etc/hosts 中设置的主机名对应)
GATEWAY=192.168.1.1 #(设置本机连接的网关的IP地址。例如，网关为10.0.0.2)
```



同样，DNS 可以用上面的方式配置，也可以修改 DNS 配置文件:

```sh
[root@centos]# vi /etc/resolv.conf
```

修改以下内容:

```
nameserver 8.8.8.8 #google域名服务器
nameserver 8.8.4.4 #google域名服务器
```



最后，记得重新启动网络配置生效：

```sh
# /etc/init.d/network restart
```



*注意：文章说的都是永久生效方式，临时生效就是用`ifconfig`或`ip addr`命令配置即可。*



OK，今天的文章不用五分钟，相信大家已经 get 了两个新技能。嘿嘿。




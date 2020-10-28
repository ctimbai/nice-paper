很多文章都会告诉你，一个 Network Namespace 有自己独立的网络环境，包括：网卡设备（Loopback Device）、端口、socket、路由表、iptables 规则、网络协议栈等。

但究竟是不是这样，我们就从代码上来一探究竟。

以 Linux 4.0 为例，Network Namespace 代码位于 `linux-4.0/include/net/net_namespace.h` 和 `../net_namespace.c` 中。

由 `struct net` 结构定义，我们来看看该结构，就知道 Network Namespace 都包含些什么。

如下，该结构比较大，我们删除一些不必要的定义：

```c
struct net {
    /*...*/
	struct list_head	list;  // netns 列表
	struct user_namespace   *user_ns;	// netns权限管理
	struct idr		netns_ids;  // netns list cur id

	struct sock 		*rtnl; /* rtnetlink socket */
	struct sock		*genl_sock;

	int			ifindex; // interface
	
    struct proc_dir_entry 	*proc_net; // /proc/net
	struct proc_dir_entry 	*proc_net_stat; // /proc/net/stat

#ifdef CONFIG_SYSCTL
	struct ctl_table_set	sysctls; // sysctl /proc/sys/net
#endif
	struct list_head	rules_ops; // fib rules
	struct net_device       *loopback_dev; // lo 设备
    /*...*/
    struct netns_core	core; // sysctl
	struct netns_mib	mib;
	struct netns_packet	packet;
	struct netns_unix	unx; // unix stack
	struct netns_ipv4	ipv4; // ipv4 stack
#if IS_ENABLED(CONFIG_IPV6)
	struct netns_ipv6	ipv6; // ipv6 stack
#endif
#if defined(CONFIG_IP_SCTP) || defined(CONFIG_IP_SCTP_MODULE)
	struct netns_sctp	sctp; // sctp
#endif
#ifdef CONFIG_NETFILTER
	struct netns_nf		nf; // netfilter
	struct netns_xt		xt;
#if defined(CONFIG_NF_TABLES) || defined(CONFIG_NF_TABLES_MODULE)
	struct netns_nftables	nft; // nftables
#endif
	struct net_generic __rcu	*gen;
#if IS_ENABLED(CONFIG_IP_VS)
	struct netns_ipvs	*ipvs; // ipvs
#endif
    /*...*/
};
```

可以看到，一个 Network Namespace 基本包括了一个网络协议栈该有的元素。

大体如开篇说的，但如果要详细说，一个 Network Namespace 隔离了如下网络环境：

- 权限管理，使用 `struct user_namespace`
- netlink/genetlink socket
- fib rules，即路由表规则
- Loopback 设备
- unix socket
- Unix/IPv4/IPv6 协议栈，socket 通信方式
- netfilter/nftables，即防火墙规则
- ipvs，即负载均衡规则
- `/proc/net` 目录
- `/sys/class/net` 目录
- `/proc/sys/net` 目录
- ...



https://time.geekbang.org/column/article/64948

http://www.hyuuhit.com/2019/03/23/netns/

https://www.junmajinlong.com/virtual/namespace/network_namespace/
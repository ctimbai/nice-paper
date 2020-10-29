> 21 世纪最宝贵的是时间和精力，人们对于碎片化阅读越来越挑剔，既不想花太多时间，又想开卷有益，而摆在眼前的长篇大论一来废话太多，二来耗时耗力，看后还是两眼发懵，所以我们力求在五分钟之内讲懂一个知识点，不贪多、不废话，关注我们，让网络不再难懂。

其实 network namespace 的配置命令 `ip netns` 就实现了持久化的功能。所以，本文其实就讲这个命令的实现。（下文统一用 netns 来表述 network namespace）

所谓持久化，是指 namespace 中的所有进程即使退出了，namespace 也依然有效。正常情况下，当 namespace 中所有进程都退出了，namespace 就会随之销毁。持久化这个特性，也不仅限于 netns ，是所有 namespace 都有的。

以 netns  为例来说，它是怎么实现的呢？

其实很简单，持久化最简单的思路就是使用文件来保存 namespace 的信息，只要文件的 fd 不删除，那么 namespace 的信息就不会丢失。所以，只要建立 namespace 和一个文件的联系就可以了。

这种联系相信你也能想到就是 mount，我们可以将 namespace 的信息 mount bind 到 root namespace 中的一个文件就能实现。

事实上 `ip netns` 就是通过 mount namespace 来实现持久化的，mount namespace  做的事就是一个 mount bind 操作。

namespace 隔离的是进程资源，所以它是进程的一部分，每个进程都会对它的资源建立相应的文件来管理，可以在 `/proc/[pid]` 目录下看见这些文件，对于 netns ，相关文件在 `/proc/[pid]/ns/net` 下。比如我们查看当前进程的 netns：

```sh
# ll -i /proc/$$/ns/net
30888 lrwxrwxrwx. 1 root root 0 Oct 29 01:01 /proc/3849/ns/net -> net:[4026531956]
```

可以看到，该文件是个软链接，指向了 一个 netns，每个 netns 会用一个 ID 来标识（实质是文件的 inode 号），如果两个进程指向的 netns 的 ID 相同，则说明它们在同一个 netns 下。

也就是，只用将 `/proc/[pid]ns/net` 文件 mount bind 到 root namespace 中任何一个文件，就能建立 netns 和文件之间的联系，只要该文件存在，该 netns 也会永远存在，这就是 netns 实现持久化的思路。

接下来，我们就来实操一下。

这里用到 `unshare` 命令，它的作用是让当前进程脱离当前的 namespace，然后加入到一个新建的 namespace 中，其中，`unshare -n` 或者 `unshare --net` 就是创建 netns。下面就用 `unshare` 来实操。

### 非持久化的 netns

```sh
$ sudo unshare -n /bin/bash
$ echo $$ # 进入到了新 netns
16370
$ sudo readlink /proc/$$/ns/net # 查看 netns
net:[4026532268]

# 打开另一个终端
$ sudo readlink /proc/16370/ns/net
net:[4026532268]

# 原来的终端退出 netns
$ exit # 退出回到 root ns
exit
$ echo $$
3849

# 查看到该 netns 已经不存在
$ ls -i /proc/16370/ns/net
ls: cannot access /proc/16370/ns/net: No such file or directory
```

### 持久化的 netns

```sh
$ sudo unshare -n /bin/bash # 同样创建一个 netns
$ echo $$ # 进入到了新 netns
16461
$ sudo touch ~/netns1 # 随便创建一个 mount 的文件
$ sudo mount --bind /proc/$$/ns/net ~/netns1 # mount
$ sudo readlink /proc/$$/ns/net # 查看 netns 
net:[4026532268]

# 打开另一个终端
$ sudo readlink /proc/16461/ns/net
net:[4026532268]
$ ls -i ~/netns1
4026532268 /root/netns1 # 可以看到该文件的 inode=netns ID

# 原来的终端退出 netns
$ exit
exit
$ echo $$
3849

# 查看到该 netns 还在
$ ls -i ~/netns1
4026532268 /root/netns1

# 再根据 inode 重新进入 netns，使用 nsenter 命令
$ sudo nsenter --net=/root/netns1 /bin/bash
$ echo $$
16571
$ sudo readlink /proc/$$/ns/net
net:[4026532268]

# 删除 netns
$ exit
$ sudo umount ~/netns1
$ rm -rf ~/netns1
```

可以看到，mount bind 之后，即使 netns 退出之后，后面还可以通过该文件重新进入到 netns 里。

这里进入的命令是 `nsenter` ，需要指定 mount 文件 `--net=/root/netns1`。看到这里相信你也想到上面的 mount 操作其实可以用 `unshare --net=xxx` 这条命令，一步就可以完成，`--net=xxx` 指定要 mount bind 的文件即可。

以上就是 `ip netns` 持久化的实现思路，日常我们使用 netns，一条命令 `ip netns add xx` 就可以替代上面的操作，并且它使用的文件位置位于 `/var/run/netns/xx`，每当创建一个 netns，都会在该目录下看到一个 ns 文件。`ip netns del xx` 该文件就会随着 netns 一起删除。

https://www.junmajinlong.com/virtual/namespace/network_namespace/
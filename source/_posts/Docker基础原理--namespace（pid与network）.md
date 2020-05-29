---
title: Docker基础原理--namespace（pid与network）
urlname: st1f9c
date: 2019-12-18 21:10:11 +0800
tags: [docker,namespace]
categories: []
---

## 前言

在计算机系统中为了标识不同的作用域，而引入 namespace（命名空间）的概念。namespace 作用在内核级别的隔离，可以有效阻止不同进程间的通信。在日常的 Linux 系统中，通常服务在启动后的进程，彼此之间不会特意进行隔离，例如 Web 服务器中，Nginx、MySQL 和 rides，三个服务之间亦可访问任意主机中的文件。如果任一服务不慎被攻破，会被入侵其他的服务。因而 namespace 的引入，可以有效防止不同服务或进程间的相互干扰。
Docker 使用 Linux 的 namespace 技术，对实现的容器，做了多种层次的隔离。Docker 所使用的七种 namespace 如下：`CLONE_NEWCGROUP`、`CLONE_NEWIPC`、`CLONE_NEWNET`、`CLONE_NEWNS`、`CLONE_NEWPID`、`CLONE_NEWUSER`  和  `CLONE_NEWUTS`。通过在进程中添加如上属性，便能保证与宿主机的对应资源进行有效隔离。

## 进程

在分时操作系统中，进程作为程序的基本执行单元，也是资源（CPU、内存）分配的基本单位。当服务、程序启动后，操作系统为其开辟一个进程，并为它分配资源，除等待 CPU 资源，其他资源以及就绪，此时进入到**就绪态。**当时间片分割的进程，在高优先级调度队列被调度后，进程开始运行。
进程的三种状态：就绪态、运行态和阻断状态。
Linux 的进程，可以通过`ps -elf`命令进行查看，下图为 Centos 系统的截图：

```bash
ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 9月24 ?       00:01:32 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root         2     0  0 9月24 ?       00:00:00 [kthreadd]
root         3     2  0 9月24 ?       00:00:00 [rcu_gp]
root         4     2  0 9月24 ?       00:00:00 [rcu_par_gp]
```

当前机器中，运行着很多进程，但都以一个 1 号进程开始，即 pid=1 的进程 init。此进程作为所以后续进程的父进程。例如 pid=39 的进程，其父进程（PPID=1）。

- pid=0 号进程 idle，其前身是系统创建的第一个进程，也是唯一一个没有通过 fork 或者 kernel_thread 产生的进程。完成加载系统后，演变为进程调度、交换
- pid=1 号进程 init，负责执行内核的初始化工作和系统配置。
- pid=2 的 kthreadd 进程，始终运行在内核空间, 负责所有内核线程的调度和管理。

## Docker 容器内进程 PID

以 RabbitMQ 容器为例，通过如下命令进入 rabbitmq 容器后，使用 ps -ef 命令，再次查看容器内的进程，可以与上面的系统进程进行比较。

```bash
[root@li1559-144 ~]# docker exec -it 28b6862b34a9 bash
root@28b6862b34a9:/# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
rabbitmq     1     0  0 16:07 ?        00:00:00 /bin/sh /opt/rabbitmq/sbin/rabbitmq-server
rabbitmq   138     1  0 16:07 ?        00:00:00 /usr/local/lib/erlang/erts-10.4.4/bin/epmd -daemon
```

当前容器内，PID=1 的进程为 rabbitmq-server，与 Linux 的宿主机的 PID=1 的 init 进程不同。此时，容器内的进程以及与宿主机完全隔离。
宿主机与 rabbitmq 容器之间的进程对应关系，如下所示（摘自@Draveness）：
![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1576771885943-5e1c23b8-82e6-43ee-bb68-1f9ef836e98d.png#align=left&display=inline&height=260&name=image.png&originHeight=520&originWidth=1200&size=83888&status=done&style=none&width=600)
因此容器内的进程，对宿主机的进程一无所知。实现了内核级的隔离。

Docker 在使用上述隔离技术，进行资源的隔离。当我们每次运行 `docker run`  或者 `docker start`  时，都会调用以下代码块（moby:[https://github.com/moby](https://github.com/moby)），创建资源隔离的容器：

```go
func (daemon *Daemon) createSpec(c *container.Container) (retSpec *specs.Spec, err error) {
	var (
		opts []coci.SpecOpts
		s    = oci.DefaultSpec()
	)
	opts = append(opts,
		WithCommonOptions(daemon, c),
		WithCgroups(daemon, c),	//添加Cgroups，进行资源限制
		WithResources(c),
		WithSysctls(c),
		WithDevices(daemon, c),
		WithUser(c),
		WithRlimits(daemon, c),
		WithNamespaces(daemon, c),	//添加namespace，进行资源隔离
		WithCapabilities(c),
		WithSeccomp(daemon, c),
		WithMounts(daemon, c),		//添加挂载点
		WithLibnetwork(daemon, c),
		WithApparmor(c),
		WithSelinux(c),
		WithOOMScore(&c.HostConfig.OomScoreAdj),
	)
```

在 `WithNamespaces(daemon, c)`  函数中，会设置 `user` `network` `ipc` `pid`  `uts`  以及 `cgroup` ，需要主要的是 moby 项目中，`CLONE_NEWNS`即文件系统被单独在 `WithMounts(daemon, c)`  函数中设置。

```go
func WithNamespaces(daemon *Daemon, c *container.Container) coci.SpecOpts {
	return func(ctx context.Context, _ coci.Client, _ *containers.Container, s *coci.Spec) error {
		userNS := false
		// user
        // network
        // ipc
        //pid
        if c.HostConfig.PidMode.IsContainer() {
			ns := specs.LinuxNamespace{Type: "pid"}
			pc, err := daemon.getPidContainer(c)
			if err != nil {
				return err
			}
			ns.Path = fmt.Sprintf("/proc/%d/ns/pid", pc.State.GetPID())
			setNamespace(s, ns)
			if userNS {
				// to share a PID namespace, they must also share a user namespace
				nsUser := specs.LinuxNamespace{Type: "user"}
				nsUser.Path = fmt.Sprintf("/proc/%d/ns/user", pc.State.GetPID())
				setNamespace(s, nsUser)
			}
		} else if c.HostConfig.PidMode.IsHost() {
			oci.RemoveNamespace(s, specs.LinuxNamespaceType("pid"))
		} else {
			ns := specs.LinuxNamespace{Type: "pid"}
			setNamespace(s, ns)
		}
        // uts
        //cgroup
        return nil
	}
}
```

## Docker 容器内网络 Network

Linux 的 namespace 主要隔离的对象有：网络设备、IPv4 和 IPv6 协议栈、路由表、防火墙、/proc/net 目录、/sys/class/net 目录、套接字（socket）等。Linux 中，物理网络设备默认放置在 root namespace 中。
Docker 中为实现网络的独立，又能与宿主机进行通信，为此创建 veth pair。将其中的一段放置在容器内通常，通常命名为 eth0，另外一段挂载为物理网络设备上所在的 namespace 中。实际中，docker 在 docker daemon 启动后会创建 docker0 网桥，docker 容器的网络对另一端通常绑定在 docker0 网桥。
现行的 docker 项目中，自 docker>1.9 之后，已经将网络单独剥离为 `libnetwork`  库，自此网络模式也被抽象为同样接口。docker 现有的网络模式有五种： `bridge驱动` 、 `host驱动` 、 `overlay驱动` 、 `remote驱动` 、 `null驱动` 。
接下来，我们手动创建 `veth pair` ，用于模仿 docker 网桥模式。docker0（lxcbr0）网桥作为二层交换网桥，之所以在上面配置 IP，可以认为内部有一个可以用于配置 IP 信息的网卡接口。Docker 在桥接模式下，docker0 网桥用于连接容器上的默认网关而存在，并且可以保证在必要时候与宿主机进行网络隔离。

```bash
## 首先，我们先增加一个网桥lxcbr0，模仿docker0
brctl addbr lxcbr0
brctl stp lxcbr0 off
ifconfig lxcbr0 192.168.10.1/24 up #为网桥设置IP地址

## 接下来，我们要创建一个network namespace - ns1

# 增加一个namesapce 命令为 ns1 （使用ip netns add命令）
ip netns add ns1

# 激活namespace中的loopback，即127.0.0.1（使用ip netns exec ns1来操作ns1中的命令）
ip netns exec ns1   ip link set dev lo up

## 然后，我们需要增加一对虚拟网卡

# 增加一个pair虚拟网卡，注意其中的veth类型，其中一个网卡要按进容器中
ip link add veth-ns1 type veth peer name lxcbr0.1

# 把 veth-ns1 按到namespace ns1中，这样容器中就会有一个新的网卡了
ip link set veth-ns1 netns ns1

# 把容器里的 veth-ns1改名为 eth0 （容器外会冲突，容器内就不会了）
ip netns exec ns1  ip link set dev veth-ns1 name eth0

# 为容器中的网卡分配一个IP地址，并激活它
ip netns exec ns1 ifconfig eth0 192.168.10.11/24 up


# 上面我们把veth-ns1这个网卡按到了容器中，然后我们要把lxcbr0.1添加上网桥上
brctl addif lxcbr0 lxcbr0.1

# 为容器增加一个路由规则，让容器可以访问外面的网络
ip netns exec ns1     ip route add default via 192.168.10.1

# 在/etc/netns下创建network namespce名称为ns1的目录，
# 然后为这个namespace设置resolv.conf，这样，容器内就可以访问域名了
mkdir -p /etc/netns/ns1
echo "nameserver 8.8.8.8" > /etc/netns/ns1/resolv.conf

## 引自：https://coolshell.cn/articles/17029.html
```

详细的 Linux 的 `ip`  命令，可以参见：[https://wangchujiang.com/linux-command/c/ip.html](https://wangchujiang.com/linux-command/c/ip.html)

# 附录 A. LXC 替代方案：Docker 和 OpenVZ

LXC 设计并且非常适合运行完整系统容器；这意味着 LXC 实例包含整个操作系统分发的文件系统，非常类似于虚拟机。尽管 LXC 可以运行单个进程，或者用自定义脚本替换 init 系统，但有其他更适合仅执行单个自包含程序的容器替代方案。在本附录中，我们将查看两种与 LXC 并存的容器实现替代方案：Docker 和 OpenVZ。

# 使用 OpenVZ 构建容器

OpenVZ 是最古老的操作系统级虚拟化技术之一，始于 2005 年。它类似于 LXC，专注于运行整个操作系统，而不是像 Docker 那样只运行单个程序。作为容器化技术，它与主机操作系统内核共享，没有 hypervisor 层。OpenVZ 使用的是经过补丁处理的 Red Hat 内核版本，与 Vanilla 内核分开维护。

让我们探索一些 OpenVZ 的特性，并看看它们与 LXC 的比较：

对于这个示例部署，我们将使用 Debian Wheezy：

```
root@ovz:~# lsb_release -rd
Description:      Debian GNU/Linux 7.8 (wheezy)
Release:    7.8
root@ovz:~#

```

首先添加 OpenVZ 仓库和密钥，然后更新软件包索引：

```
root@ovz:~# cat << EOF > /etc/apt/sources.list.d/openvz-rhel6.list
deb http://download.openvz.org/debian wheezy main
EOF
root@ovz:~#
root@ovz:~# wget ftp://ftp.openvz.org/debian/archive.key
root@ovz:~# apt-key add archive.key
root@ovz:~# apt-get update

```

接下来，安装 OpenVZ 内核：

```
root@ovz:~# apt-get install linux-image-openvz-amd64

```

如果使用 GRUB，使用 OpenVZ 内核更新启动菜单；在此示例中，内核被添加为菜单项 2：

```
root@ovz:~# cat /boot/grub/grub.cfg | grep menuentry
menuentry 'Debian GNU/Linux, with Linux 3.2.0-4-amd64' --class debian --class gnu-linux --class gnu --class os {
menuentry 'Debian GNU/Linux, with Linux 3.2.0-4-amd64 (recovery mode)' --class debian --class gnu-linux --class gnu --class os {
menuentry 'Debian GNU/Linux, with Linux 2.6.32-openvz-042stab120.11-amd64' --class debian --class gnu-linux --class gnu --class os {
menuentry 'Debian GNU/Linux, with Linux 2.6.32-openvz-042stab120.11-amd64 (recovery mode)' --class debian --class gnu-linux --class gnu --class os {
root@ovz:~#
root@ovz:~# vim /etc/default/grub
...
GRUB_DEFAULT=2
... 
root@ovz:~# update-grub
Generating grub.cfg ...
Found linux image: /boot/vmlinuz-3.2.0-4-amd64
Found initrd image: /boot/initrd.img-3.2.0-4-amd64
Found linux image: /boot/vmlinuz-2.6.32-openvz-042stab120.11-amd64
Found initrd image: /boot/initrd.img-2.6.32-openvz-042stab120.11-amd64
done
root@ovz:~#

```

我们需要在内核中启用路由并禁用代理 ARP：

```
root@ovz:~# cat /etc/sysctl.d/ovz.conf
net.ipv4.ip_forward = 1
net.ipv6.conf.default.forwarding = 1
net.ipv6.conf.all.forwarding = 1
net.ipv4.conf.default.proxy_arp = 0
net.ipv4.conf.all.rp_filter = 1
kernel.sysrq = 1
net.ipv4.conf.default.send_redirects = 1
net.ipv4.conf.all.send_redirects = 0
root@ovz2:~#
root@ovz:~# sysctl -p /etc/sysctl.d/ovz.conf
...
root@ovz:~#

```

现在是重新启动服务器的时候了，然后检查 OpenVZ 是否已加载：

```
root@ovz:~# reboot
root@ovz:~# uname -a
Linux ovz 2.6.32-openvz-042stab120.11-amd64 #1 SMP Wed Nov 16 12:07:16 MSK 2016 x86_64 GNU/Linux
root@ovz:~#

```

接下来，安装用户空间工具：

```
root@ovz:~# apt-get install vzctl vzquota ploop vzstats

```

OpenVZ 使用与 LXC 类似的模板。这些模板是归档的根文件系统，可以使用 `debootstrap` 等工具构建。让我们在 OpenVZ 默认期望它们的目录中下载一个 Ubuntu 模板：

```
root@ovz:~# cd /var/lib/vz/template/
root@ovz:/var/lib/vz/template# wget http://download.openvz.org/template/precreated/ubuntu-16.04-x86_64.tar.gz
root@ovz:/var/lib/vz/template#

```

在放置模板归档文件的位置后，让我们创建一个容器：

```
root@ovz:/var/lib/vz/template# vzctl create 1 --ostemplate ubuntu-16.04-x86_64 --layout simfs
Creating container private area (ubuntu-16.04-x86_64)
Performing postcreate actions
CT configuration saved to /etc/vz/conf/1.conf
Container private area was created
root@ovz:/var/lib/vz/template# 

```

我们指定 `simfs` 作为底层容器存储的类型，它将在主机操作系统上创建根文件系统，类似于 LXC 和默认目录类型。OpenVZ 还提供了其他选项，例如 Ploop，它会创建包含容器文件系统的镜像文件。

接下来，创建一个 Linux 桥接器：

```
root@ovz:/var/lib/vz/template# apt-get install bridge-utils
root@ovz:/var/lib/vz/template# brctl addbr br0

```

为了允许 OpenVZ 将其容器连接到主机桥接器，创建以下配置文件：

```
root@ovz:/var/lib/vz/template# cat /etc/vz/vznet.conf
#!/bin/bash
EXTERNAL_SCRIPT="/usr/sbin/vznetaddbr"
root@ovz:/var/lib/vz/template#

```

该文件指定了一个外部脚本，该脚本将把容器的虚拟接口添加到我们之前创建的桥接器中。

通过指定容器内外接口的名称和应连接到的桥接器，配置我们的容器网络接口：

```
root@ovz:/var/lib/vz/template# vzctl set 1 --save --netif_add eth0,,veth1.eth0,,br0
CT configuration saved to /etc/vz/conf/1.conf
root@ovz:/var/lib/vz/template#

```

通过执行以下命令列出主机上可用的容器：

```
root@ovz:/var/lib/vz/template# vzlist -a
CTID      NPROC STATUS    IP_ADDR      HOSTNAME
 1          - stopped      -               -
root@ovz:/var/lib/vz/template# cd

```

要启动我们的容器，请运行以下命令：

```
root@ovz:~# vzctl start 1
Starting container...
Container is mounted
Setting CPU units: 1000
Configure veth devices: veth1.eth0
Adding interface veth1.eth0 to bridge br0 on CT0 for CT1
Container start in progress...
root@ovz:~#

```

然后，要附加或进入容器，请执行以下命令：

```
root@ovz:~# vzctl enter 1
entered into CT 1
root@localhost:/# exit
logout
exited from CT 1
root@ovz:~#

```

可以像 LXC 一样，在不重新启动容器的情况下实时操作可用的容器资源。让我们将内存设置为 1 GB：

```
root@ovz:~# vzctl set 1 --ram 1G --save
UB limits were set successfully
CT configuration saved to /etc/vz/conf/1.conf
root@ovz:~#

```

每个 OpenVZ 容器都有一个配置文件，当传递 `--save` 选项给 `vzctl` 工具时，它会被更新。要查看它，请运行以下命令：

```
root@ovz:~# cat /etc/vz/conf/1.conf | grep -vi "#" | sed '/^$/d'
PHYSPAGES="0:262144"
SWAPPAGES="0:512M"
DISKSPACE="2G:2.2G"
DISKINODES="131072:144179"
QUOTATIME="0"
CPUUNITS="1000"
NETFILTER="stateless"
VE_ROOT="/var/lib/vz/root/$VEID"
VE_PRIVATE="/var/lib/vz/private/$VEID"
VE_LAYOUT="simfs"
OSTEMPLATE="ubuntu-16.04-x86_64"
ORIGIN_SAMPLE="vswap-256m"
NETIF="ifname=eth0,bridge=br0,mac=00:18:51:A1:C6:35,host_ifname=veth1.eth0,host_mac=00:18:51:BF:1D:AC"
root@ovz:~#

```

容器运行时，确保主机上的虚拟接口已添加到桥接中。请注意，桥接接口本身处于 `DOWN` 状态：

```
root@ovz:~# brctl show
bridge name bridge id          STP enabled    interfaces
br0         8000.001851bf1dac  no             veth1.eth0
root@ovz:~# ip a s
...
4: br0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN
link/ether 00:18:51:bf:1d:ac brd ff:ff:ff:ff:ff:ff
6: veth1.eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN
 link/ether 00:18:51:bf:1d:ac brd ff:ff:ff:ff:ff:ff
 inet6 fe80::218:51ff:febf:1dac/64 scope link
 valid_lft forever preferred_lft forever
root@ovz:~#

```

我们可以在容器内执行命令，无需附加到它。让我们为容器的接口配置一个 IP 地址：

```
root@ovz:~# vzctl exec 1 "ifconfig eth0 192.168.0.5"
root@ovz:~#

```

启动主机上的桥接接口并配置一个 IP 地址，以便我们可以从主机访问容器：

```
root@ovz:~# ifconfig br0 up
root@ovz:~# ifconfig br0 192.168.0.1

```

让我们测试连通性：

```
root@ovz:~# ping -c3 192.168.0.5
PING 192.168.0.5 (192.168.0.5) 56(84) bytes of data.
64 bytes from 192.168.0.5: icmp_req=1 ttl=64 time=0.037 ms
64 bytes from 192.168.0.5: icmp_req=2 ttl=64 time=0.036 ms
64 bytes from 192.168.0.5: icmp_req=3 ttl=64 time=0.036 ms
--- 192.168.0.5 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.036/0.036/0.037/0.005 ms
root@ovz:~#

```

让我们进入容器并确保可用内存确实为 1 GB，正如我们之前设置的那样：

```
root@ovz:~# vzctl enter 1
entered into CT 1
root@localhost:/# free -g
total    used    free   shared  buff/cache   available
Mem:     1        0       0         0           0           0
Swap:    0        0       0
root@localhost:/# exit
logout
exited from CT 1
root@ovz:~#

```

注意 OpenVZ 容器如何使用 `init` 启动所有其他进程，就像虚拟机一样：

```
root@ovz:~# ps axfww
...
3303 ?        Ss     0:00 init -z
3365 ?        Ss     0:00  \_ /lib/systemd/systemd-journald
3367 ?        Ss     0:00  \_ /lib/systemd/systemd-udevd
3453 ?        Ss     0:00  \_ /sbin/rpcbind -f -w
3454 ?        Ssl    0:00  \_ /usr/sbin/rsyslogd -n
3457 ?        Ss     0:00  \_ /usr/sbin/cron -f
3526 ?        Ss     0:00  \_ /usr/sbin/xinetd -pidfile /run/xinetd.pid -stayalive -inetd_compat -inetd_ipv6
3536 ?        Ss     0:00  \_ /usr/sbin/saslauthd -a pam -c -m /var/run/saslauthd -n 2
3540 ?        S      0:00  |   \_ /usr/sbin/saslauthd -a pam -c -m /var/run/saslauthd -n 2
3542 ?        Ss     0:00  \_ /usr/sbin/apache2 -k start
3546 ?        Sl     0:00  |   \_ /usr/sbin/apache2 -k start
3688 ?        Ss     0:00  \_ /usr/lib/postfix/sbin/master
3689 ?        S      0:00  |   \_ pickup -l -t unix -u -c
3690 ?        S      0:00  |   \_ qmgr -l -t unix -u
3695 ?        Ss     0:00  \_ /usr/sbin/sshd -D
3705 tty1     Ss+    0:00  \_ /sbin/agetty --noclear --keep-baud console 115200 38400 9600 vt220
3706 tty2     Ss+    0:00  \_ /sbin/agetty --noclear tty2 linux
root@ovz:~#

```

我们现在知道，所有容器实现都使用 cgroups 来控制系统资源，OpenVZ 也不例外。让我们看看 cgroup 层次结构是如何挂载的：

```
root@ovz:~# mount | grep cgroup
beancounter on /proc/vz/beancounter type cgroup (rw,relatime,blkio,name=beancounter)
container on /proc/vz/container type cgroup (rw,relatime,freezer,devices,name=container)
fairsched on /proc/vz/fairsched type cgroup (rw,relatime,cpuacct,cpu,cpuset,name=fairsched)
tmpfs on /var/lib/vz/root/1/sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,size=131072k,nr_inodes=32768,mode=755)
cgroup on /var/lib/vz/root/1/sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /var/lib/vz/root/1/sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /var/lib/vz/root/1/sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio,name=beancounter)
root@ovz:~#

```

我们之前创建的容器的 ID 为 `1`，如前面的示例所示。我们可以通过运行以下命令获取容器内所有进程的 PID：

```
root@ovz:~# cat /proc/vz/fairsched/1/cgroup.procs
3303
3304
3305
3365
3367
3453
3454
3457
3526
3536
3540
3542
3546
3688
3689
3690
3695
3705
3706
root@ovz:~#

```

我们还可以获取分配给容器的 CPU 数量：

```
root@ovz:~# cat /proc/vz/fairsched/1/cpu.nr_cpus
0
root@ovz:~#

```

让我们为 ID 为 `1` 的容器分配两个核心：

```
root@ovz:~# vzctl set 1 --save --cpus 2
UB limits were set successfully
Setting CPUs: 2
CT configuration saved to /etc/vz/conf/1.conf
root@ovz:~#

```

然后确保更改在同一文件中可见：

```
root@ovz:~# cat /proc/vz/fairsched/1/cpu.nr_cpus
2
root@ovz:~#

```

容器的配置文件也应反映该更改：

```
root@ovz:~# cat /etc/vz/conf/1.conf | grep -i CPUS
CPUS="2"
root@ovz:~#

```

使用 `ps` 命令，或者通过读取前述系统文件，我们可以获取容器内 `init` 进程的 PID，在本示例中为 `3303`。知道了 PID 后，我们可以通过运行以下命令获取容器的 ID：

```
root@ovz:~# cat /proc/3303/status | grep envID
envID:      1
root@ovz:~#

```

由于容器的根文件系统存在于主机上，迁移 OpenVZ 实例与 LXC 类似——我们首先停止容器，然后归档根文件系统，将其复制到新服务器，并提取它。我们还需要容器的配置文件。让我们来看一个将 OpenVZ 容器迁移到新主机的示例：

```
root@ovz:~# vzctl stop 1
Stopping container ...
Container was stopped
Container is unmounted
root@ovz:~# 
root@ovz:~# tar -zcvf /tmp/ovz_container_1.tar.gz -C /var/lib/vz/private 1
root@ovz:~# scp  /tmp/ovz_container_1.tar.gz 10.3.20.31:/tmp/
root@ovz:~# scp /etc/vz/conf/1.conf 10.3.20.31:/etc/vz/conf/
root@ovz:~#

```

在第二台服务器上，我们提取根文件系统：

```
root@ovz2:~# tar zxfv /tmp/ovz_container_1.tar.gz --numeric-owner -C /var/lib/vz/private
root@ovz2:~#

```

配置文件和文件系统准备好后，我们可以通过运行以下命令来列出容器：

```
root@ovz2:~# vzlist -a
stat(/var/lib/vz/root/1): No such file or directory
CTID      NPROC STATUS    IP_ADDR         HOSTNAME
1          - stopped       -               -
root@ovz2:~# 

```

最后，要启动 OpenVZ 实例并确保它在新主机上运行，请执行以下命令：

```
root@ovz2:~# vzctl start 1
Starting container...
stat(/var/lib/vz/root/1): No such file or directory
stat(/var/lib/vz/root/1): No such file or directory
stat(/var/lib/vz/root/1): No such file or directory
Initializing quota ...
Container is mounted
Setting CPU units: 1000
Setting CPUs: 2
Configure veth devices: veth1.eth0
Container start in progress...
root@ovz2:~# vzlist -a
CTID      NPROC STATUS    IP_ADDR         HOSTNAME
1         47 running       -               -
root@ovz2:~#

```

OpenVZ 没有集中式控制守护进程，这使得与 `upstart` 或 `systemd` 等初始化系统的集成更加容易。需要注意的是，OpenVZ 是 Virtuozzo 公司提供的 Virtuozzo 虚拟化解决方案的基础，其最新版本将是一个完整操作系统的 ISO 镜像，而不是运行带有单独工具集的修补内核。

### 注意

要了解最新的 OpenVZ 和 Virtuozzo 版本，请访问 [`openvz.org`](https://openvz.org)。

# 使用 Docker 构建容器

Docker 项目于 2013 年发布，并迅速获得了广泛的关注，超越了 OpenVZ 和 LXC。现在，大型生产部署都在运行 Docker，配合各种编排框架，如 Apache Mesos 和 Kubernetes，提供 Docker 集成。

与 LXC 和 OpenVZ 不同，Docker 更适合在最小化容器设置中运行单个应用程序。它使用 Docker Engine 守护进程，该进程控制 `containerd` 进程来管理容器的生命周期，从而使它更难与其他初始化系统（如 `systemd`）集成。

Docker 提供了一个方便的 API，供各种工具使用，并使得从预构建的镜像中快速配置容器变得容易，这些镜像可以托管在远程的公共或私有仓库/注册表中。

我们可以在同一主机上运行 LXC 和 Docker 容器而不会遇到任何问题，因为它们有清晰的隔离。在接下来的部分，我们将探索 Docker 的大部分功能，通过检查 Docker 容器的生命周期并了解它与 LXC 的比较。

让我们首先更新服务器并安装仓库和密钥：

```
root@docker:~# apt-get update && apt-get upgrade && reboot
... 
root@docker:~# apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
Executing: /tmp/tmp.Au9fc0rNu3/gpg.1.sh --keyserver
hkp://ha.pool.sks-keyservers.net:80
--recv-keys
58118E89F3A912897C070ADBF76221572C52609D
gpg: requesting key 2C52609D from hkp server ha.pool.sks-keyservers.net
gpg: key 2C52609D: public key "Docker Release Tool (releasedocker) <docker@docker.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1) 
root@docker:~# echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list
deb https://apt.dockerproject.org/repo ubuntu-xenial main
root@docker:~# apt-get update

```

列出当前可用的软件包版本并安装最新的候选版本：

```
root@docker:~# apt-cache policy docker-engine
docker-engine:
 Installed: (none)
 Candidate: 1.12.4-0~ubuntu-xenial
 Version table:
 1.12.4-0~ubuntu-xenial 500
 500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
 1.12.3-0~xenial 500
 500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
 1.12.2-0~xenial 500
 500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
 1.12.1-0~xenial 500
 500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
 1.12.0-0~xenial 500
 500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
 1.11.2-0~xenial 500
 500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
 1.11.1-0~xenial 500
 500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
 1.11.0-0~xenial 500
 500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages 
root@docker:~# apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual
root@docker:~# apt-get install docker-engine

```

启动 Docker 服务并确保它们正在运行：

```
root@docker:~# service docker start
root@docker:~# pgrep -lfa docker
24585 /usr/bin/dockerd -H fd://
24594 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --shim docker-containerd-shim --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --runtime docker-runc
root@docker:~#

```

Docker 守护进程运行后，让我们列出所有可用的容器，目前我们没有任何容器：

```
root@docker:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
root@docker:~#

```

让我们通过执行以下命令，从上游公共注册表中查找一个 Ubuntu 镜像：

```
root@docker:~# docker search ubuntu
NAME      DESCRIPTION        STARS     OFFICIAL  
AUTOMATED
ubuntu  Ubuntu is a Debian-based Linux operating s...   5200      [OK]
ubuntu-upstart  Upstart is an event-based replacement for ...   69        [OK]
...
root@docker:~#

```

我们为容器选择官方的 Ubuntu 镜像；要创建它，运行以下命令：

```
root@docker:~# docker create --tty --interactive ubuntu bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
af49a5ceb2a5: Pull complete
8f9757b472e7: Pull complete
e931b117db38: Pull complete
47b5e16c0811: Pull complete
9332eaf1a55b: Pull complete
Digest: sha256:3b64c309deae7ab0f7dbdd42b6b326261ccd6261da5d88396439353162703fb5
Status: Downloaded newer image for ubuntu:latest
ec66fcfb5960c0779d07243f2c1e4d4ac10b855e940d416514057a9b28d78d09
root@docker:~#

```

我们现在应该在系统中有一个缓存的 Ubuntu 镜像：

```
root@docker:~# docker images
REPOSITORY   TAG     IMAGE ID       CREATED      SIZE
ubuntu       latest  4ca3a192ff2a  2 weeks ago   128.2 MB
root@docker:~#

```

让我们再次列出主机上可用的容器：

```
root@docker:~# docker ps --all
CONTAINER ID    IMAGE      COMMAND     CREATED           STATUS         PORTS         NAMES
ec66fcfb5960    ubuntu     "bash"     29 seconds ago      Created                 docker_container_1
root@docker:~#

```

启动 Ubuntu Docker 容器同样简单：

```
root@docker:~# docker start docker_container_1
docker_container_1
root@docker:~# docker ps
CONTAINER ID    IMAGE   COMMAND     CREATED             STATUS          PORTS     NAMES
ec66fcfb5960    ubuntu  "bash"   About a minute ago   Up 17 seconds         docker_container_1
root@docker:~#

```

通过检查进程列表，注意到我们现在有一个单一的 bash 进程作为 `dockerd` 和 `docker-containerd` 进程的子进程运行：

```
root@docker:~# ps axfww
...
24585 ?        Ssl    0:07 /usr/bin/dockerd -H fd://
24594 ?        Ssl    0:00  \_ docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --shim docker-containerd-shim --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --runtime docker-runc
26942 ?        Sl     0:00      \_ docker-containerd-shim ec66fcfb5960c0779d07243f2c1e4d4ac10b855e940d416514057a9b28d78d09 /var/run/docker/libcontainerd/ec66fcfb5960c0779d07243f2c1e4d4ac10b855e940d416514057a9b28d78d09 docker-runc
26979 pts/1    Ss+    0:00          \_ bash
root@docker:~#

```

通过附加到容器，我们可以看到它正在运行一个单独的 bash 进程，而不是像 LXC 或 OpenVZ 那样使用完整的初始化系统：

```
root@docker:~# docker attach docker_container_1
root@ec66fcfb5960:/# ps axfw
PID TTY    STAT   TIME   COMMAND
1   ?        Ss     0:00   bash
10   ?       R+     0:00   ps axfw 
root@ec66fcfb5960:/# exit
exit
root@docker:~#

```

注意，在退出容器后，它会被终止：

```
root@docker:~# docker attach docker_container_1
You cannot attach to a stopped container, start it first
root@docker:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
ec66fcfb5960        ubuntu              "bash"              
3 minutes ago       Exited (0) 26 seconds ago                   docker_container_1
root@docker:~#

```

让我们重新启动它；我们可以像使用 OpenVZ 或 libvirt LXC 一样，指定其名称或 ID：

```
root@docker:~# docker start ec66fcfb5960
ec66fcfb5960 
root@docker:~# docker ps
CONTAINER ID        IMAGE               COMMAND          CREATED             STATUS              PORTS               
NAMES
ec66fcfb5960        ubuntu              "bash"              
3 minutes ago       Up 2 seconds                            docker_container_1
root@docker:~#

```

要更新容器的内存，执行以下命令：

```
root@docker:~# docker update --memory 1G docker_container_1
docker_container_1
root@docker:~# 

```

检查容器的内存设置，确保内存已成功更新：

```
root@docker:~# docker inspect docker_container_1 | grep -i memory
"Memory": 1073741824,
"KernelMemory": 0,
"MemoryReservation": 0,
"MemorySwap": 0,
"MemorySwappiness": -1,
root@docker:~#

```

与 LXC 和 OpenVZ 一样，相应的 cgroup 层次结构已被更新。我们应该能够在容器 ID 的 cgroup 文件中看到相同的内存量：

```
root@docker:~# cat
/sys/fs/cgroup/memory/docker/ec66fcfb5960c0779d07243f2c1e4d4ac10b855e940d416514057a9b28d78d09/memory.limit_in_bytes
1073741824
root@docker:~#

```

让我们更新 CPU 配额：

```
root@docker:~# docker update --cpu-shares 512 docker_container_1
docker_container_1
root@docker:~#

```

然后，检查 cgroup 文件中的设置，将容器 ID 替换为正在你主机上运行的 ID：

```
root@docker:~# cat /sys/fs/cgroup/cpu/docker/ec66fcfb5960c0779d07243f2c1e4d4ac10b855e940d416514057a9b28d78d09/cpu.shares
512
root@docker:~#

```

Docker 提供了很少的监控容器状态和资源利用率的方法，非常类似于 LXC：

```
root@docker:~# docker top docker_container_1
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                27812               27774               0                   17:41               pts/1               00:00:00            bash 
root@docker:~# docker stats docker_container_1
CONTAINER           CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
docker_container_1   0.00%               1.809 MiB / 1 GiB   0.18%               648 B / 648 B       0 B / 0 B           1
^C
root@docker:~#

```

我们也可以在不附加到容器的情况下，在容器的命名空间内运行命令：

```
root@docker:~# docker exec docker_container_1 ps ax
PID TTY    STAT   TIME COMMAND
1 ?        Ss+    0:00 bash
11 ?       Rs     0:00 ps ax
root@docker:~#

```

使用以下命令将文件从主机文件系统复制到容器中；我们在 LXC 和 OpenVZ 中看过类似的例子：

```
root@docker:~# touch /tmp/test_file
root@docker:~# docker cp /tmp/test_file docker_container_1:/tmp/
root@docker:~# docker exec docker_container_1 ls -la /tmp
total 8
drwxrwxrwt  2 root root 4096 Dec 14 19:39 .
drwxr-xr-x 36 root root 4096 Dec 14 19:39 ..
-rw-r--r--  1 root root    0 Dec 14 19:38 test_file
root@docker:~#

```

使用 Docker 在主机之间移动容器更加简单；无需手动归档根文件系统：

```
root@docker:~# docker export docker_container_1 > docker_container_1.tar
root@docker:~# docker import docker_container_1.tar
sha256:c86a93369be687f9ead4758c908fe61b344d5c84b1b70403ede6384603532aa9 
root@docker:~# docker images
REPOSITORY        TAG           IMAGE ID           CREATED             SIZE
<none>           <none>        c86a93369be6     6 seconds ago       110.7 MB
ubuntu           latest        4ca3a192ff2a      2 weeks ago         128.2 MB
root@docker:~#

```

要删除本地镜像，请运行以下命令：

```
root@docker:~# docker rmi c86a93369be6
Deleted:sha256:c86a93369be687f9ead4758c908fe61b344d5c84b1b70403ede6384603532aa9
Deleted:sha256:280a817cfb372d2d2dd78b7715336c89d2ac28fd63f7e9a0af22289660214d32
root@docker:~#

```

Docker API 暴露了一种定义软件网络的方法，类似于我们在 libvirt LXC 中看到的内容。让我们安装 Linux 桥接并查看 Docker 主机上的内容：

```
root@docker:~# apt-get install bridge-utils
root@docker:~# brctl show
bridge name    bridge id         STP   enabled interfaces
docker0       8000.0242d6dd444c  no      vethf5b871d
root@docker:~#

```

请注意由 Docker 服务创建的`docker0`桥接。让我们列出自动创建的网络：

```
root@docker:~# docker network ls
NETWORK ID          NAME           DRIVER          SCOPE
a243008cd6cd        bridge         bridge          local
24d4b310a2e1        host           host            local
1e8e35222e39        none           null            local
root@docker:~# 

```

要检查`bridge`网络，请运行以下命令：

```
root@docker:~# docker network inspect bridge
[
{
 "Name": "bridge",
 "Id":"a243008cd6cd01375ef389de58bc11e1e57c1f3
         e4965a53ea48866c0dcbd3665",
 "Scope": "local",
 "Driver": "bridge",
 "EnableIPv6": false,
 "IPAM": {
 "Driver": "default",
 "Options": null,
 "Config": [
 {
 "Subnet": "172.17.0.0/16"
 }
 ]
 },
 "Internal": false,
 "Containers": {
"ec66fcfb5960c0779d07243f2c1e4d4ac10b8
               55e940d416514057a9b28d78d09": {
 "Name": "docker_container_1",
 "EndpointID":"27c07e8f24562ea333cdeb5c
                    11015d13941c746b02b1fc18a766990b772b5935",
 "MacAddress": "02:42:ac:11:00:02",
 "IPv4Address": "172.17.0.2/16",
 "IPv6Address": ""
 }
 },
 "Options": {
 "com.docker.network.bridge.default_bridge": "true",
 "com.docker.network.bridge.enable_icc": "true",
 "com.docker.network.bridge.enable_ip_masquerade": 
                "true",
 "com.docker.network.bridge.host_binding_ipv4":  
                "0.0.0.0",
 "com.docker.network.bridge.name": "docker0",
 "com.docker.network.driver.mtu": "1500"
 },
 "Labels": {}
 }
]
root@docker:~#

```

最后，要停止并删除 Docker 容器，请执行以下命令：

```
root@docker:~# docker stop docker_container_1
docker_container_1
root@docker:~#  docker rm docker_container_1

```

# 运行非特权 LXC 容器

简要讨论一下 LXC 的安全性。从 LXC 1.0 版本开始，引入了对非特权容器的支持，允许非特权用户运行容器。以 root 身份运行 LXC 容器的主要安全问题是，容器内的 UID 0 与主机上的 UID 0 相同；因此，突破容器将使你在服务器上获得 root 权限。

在第一章，*Linux 容器简介*中，我们详细讨论了用户命名空间以及它如何使得用户命名空间内的进程拥有与默认命名空间不同的用户和组 ID。在 LXC 的上下文中，这使得进程可以在容器内以 root 身份运行，同时在主机上具有非特权 ID。为了利用这一点，我们可以为每个容器创建一个映射，使用一组定义的 UID 和 GID 在主机和 LXC 容器之间进行映射。

让我们看一个设置并运行 LXC 容器作为非特权用户的示例。

首先，更新您的 Ubuntu 系统并安装 LXC：

```
root@ubuntu:~# apt-get update&& apt-get upgrade
root@ubuntu:~# reboot
root@ubuntu:~# apt-get install lxc

```

接下来，创建一个新用户并为其分配密码：

```
root@ubuntu:~# useradd -s /bin/bash -c 'LXC user' -m lxc_user
root@ubuntu:~# passwd lxc_user
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
root@ubuntu:~#

```

记下我们创建的新用户在系统中的 UID 和 GID 范围：

```
root@ubuntu:~# cat /etc/sub{gid,uid} | grep lxc_user
lxc_user:231072:65536
lxc_user:231072:65536
root@ubuntu:~#

```

请记下创建的 Linux 桥接的名称：

```
root@ubuntu:~# brctl show
bridge name bridge id         STP enabled interfaces
lxcbr0            8000.000000000000 no
root@ubuntu:~#

```

指定为用户可以添加到桥接的虚拟接口数量，在此示例中为`50`：

```
root@ubuntu:~# cat /etc/lxc/lxc-usernet
# USERNAME TYPE BRIDGE COUNT
lxc_user veth lxcbr0 50
root@ubuntu:~#

```

接下来，作为`lxc_user`，创建目录结构和配置文件，如下所示：

```
root@ubuntu:~# su - lxc_user
lxc_user@ubuntu:~$ pwd
/home/lxc_user
lxc_user@ubuntu:~$ mkdir -p .config/lxc
lxc_user@ubuntu:~$ cp /etc/lxc/default.conf .config/lxc/
lxc_user@ubuntu:~$ cat .config/lxc/default.conf
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:xx:xx:xx
lxc.id_map = u 0 231072 65536
lxc.id_map = g 0 231072 65536
lxc_user@ubuntu:~$

```

前面的`id_map`选项将映射`lxc_user`的虚拟 ID 范围。

我们现在可以像往常一样创建容器：

```
lxc_user@ubuntu:~$ lxc-create --name user_lxc --template download
...
Distribution: ubuntu
Release: xenial
Architecture: amd64
Downloading the image index
Downloading the rootfs
Downloading the metadata
The image cache is now ready
Unpacking the rootfs
...
lxc_user@ubuntu:~$

```

容器处于停止状态；要启动它，请运行以下命令：

```
lxc_user@ubuntu:~$ lxc-ls -f
NAME     STATE   AUTOSTART GROUPS IPV4 IPV6
user_lxc STOPPED 0         -      -    -
lxc_user@ubuntu:~$ lxc-start --name user_lxc
lxc_user@ubuntu:~$ lxc-ls -f
NAME     STATE   AUTOSTART GROUPS IPV4 IPV6
user_lxc RUNNING 0         -      -    -
lxc_user@ubuntu:~$

```

请注意，容器中的进程由`lxc_user`而不是 root 拥有：

```
lxc_user@ubuntu:~$ ps axfwwu
lxc_user  4291  0.0  0.0  24448  1584 ?        Ss   19:13   0:00 /usr/lib/x86_64-linux-gnu/lxc/lxc-monitord /home/lxc_user/.local/share/lxc 5
lxc_user  4293  0.0  0.0  32892  3148 ?        Ss   19:13   0:00 [lxc monitor] /home/lxc_user/.local/share/lxc user_lxc
231072    4304  0.5  0.0  37316  5356 ?        Ss   19:13   0:00  \_ /sbin/init
231072    4446  0.1  0.0  35276  4056 ?        Ss   19:13   0:00      \_ /lib/systemd/systemd-journald
231176    4500  0.0  0.0 182664  3084 ?        Ssl  19:13   0:00      \_ /usr/sbin/rsyslogd -n
231072    4502  0.0  0.0  28980  3044 ?        Ss   19:13   0:00      \_ /usr/sbin/cron -f
231072    4521  0.0  0.0   4508  1764 ?        S    19:13   0:00      \_ /bin/sh /etc/init.d/ondemand background
231072    4531  0.0  0.0   7288   816 ?        S    19:13   0:00      |   \_ sleep 60
231072    4568  0.0  0.0  15996   856 ?        Ss   19:13   0:00      \_ /sbin/dhclient -1 -v -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases -I -df /var/lib/dhcp/dhclient6.eth0.leases eth0
231072    4591  0.0  0.0  15756  2228 pts/1    Ss+  19:13   0:00      \_ /sbin/agetty --noclear --keep-baud pts/1 115200 38400 9600 vt220
231072    4592  0.0  0.0  15756  2232 pts/2    Ss+  19:13   0:00      \_ /sbin/agetty --noclear --keep-baud pts/2 115200 38400 9600 vt220
231072    4593  0.0  0.0  15756  2224 pts/0    Ss+  19:13   0:00      \_ /sbin/agetty --noclear --keep-baud pts/0 115200 38400 9600 vt220
231072    4594  0.0  0.0  15756  2228 pts/2    Ss+  19:13   0:00      \_ /sbin/agetty --noclear --keep-baud console 115200 38400 9600 vt220
231072    4595  0.0  0.0  15756  2232 ?        Ss+  19:13   0:00      \_ /sbin/agetty --noclear --keep-baud pts/3 115200 38400 9600 vt220
lxc_user@ubuntu:~$

```

当以非 root 用户运行容器时，根文件系统和配置文件的位置是不同的：

```
lxc_user@ubuntu:~$ ls -la .local/share/lxc/user_lxc/
total 16
drwxrwx---  3   231072 lxc_user 4096 Dec 15 19:13 . 
drwxr-xr-x  3 lxc_user lxc_user 4096 Dec 15 19:13 ..
-rw-rw-r--  1 lxc_user lxc_user  845 Dec 15 19:13 config
drwxr-xr-x 21   231072   231072 4096 Dec 15 03:59 rootfs
-rw-rw-r--  1 lxc_user lxc_user    0 Dec 15 19:13 user_lxc.log
lxc_user@ubuntu:~$ cat .local/share/lxc/user_lxc/config | grep -vi "#" | sed '/^$/d'
lxc.include = /usr/share/lxc/config/ubuntu.common.conf
lxc.include = /usr/share/lxc/config/ubuntu.userns.conf
lxc.arch = x86_64
lxc.id_map = u 0 231072 65536
lxc.id_map = g 0 231072 65536
lxc.rootfs = /home/lxc_user/.local/share/lxc/user_lxc/rootfs
lxc.rootfs.backend = dir
lxc.utsname = user_lxc
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:a7:f2:97
lxc_user@ubuntu:~$

```

# 总结

在本章中，我们看到了如何使用 OpenVZ 和 Docker 等替代技术部署容器的示例。

OpenVZ 是最古老的容器解决方案之一，直到写作时，它正在重新命名为 Virtuozzo。LXC 和 OpenVZ 之间的主要区别在于 OpenVZ 运行的自定义 Linux 内核。它基于 Red Hat 内核，并且很快将以单一安装 ISO 的形式发布，而不像我们在之前的示例中使用的那样，依赖于打包的内核和用户空间工具。

Docker 是容器化领域的事实标准，也是采纳的领导者。作为较新的容器化技术之一，它的易用性和可用的 API 使其成为大规模运行微服务的理想解决方案。Docker 不需要修补内核即可工作，并且集中式注册表用于存储容器镜像，使其在许多场景中成为一个绝佳的选择。

我们通过展示如何运行非特权的 LXC 容器的示例结束了这一章。这个功能相对较新，它在容器安全性方面迈出了正确的一步。

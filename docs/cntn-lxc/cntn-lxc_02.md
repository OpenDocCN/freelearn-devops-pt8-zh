# 第二章：在 Linux 系统上安装和运行 LXC

LXC 利用内核命名空间和 cgroups 创建我们称之为容器的进程隔离，正如我们在前一章中看到的那样。因此，LXC 不是 Linux 内核中的一个独立软件组件，而是一组用户空间工具、`liblxc`库和各种语言绑定。

本章将涵盖以下主题：

+   使用发行版包在 Ubuntu 和 CentOS 上安装 LXC

+   从源代码编译和安装 LXC

+   使用提供的模板和配置文件构建和启动容器

+   使用`debootstrap`和`yum`等工具手动构建根文件系统和配置文件

# 安装 LXC

在写本书时，LXC 有两个长期支持版本：1.0 和 2.0。它们提供的用户空间工具在命令行标志和弃用功能上有一些小差异，我会在使用时指出这些差异。

## 使用 apt 在 Ubuntu 上安装 LXC

让我们首先在 Ubuntu 14.04.5（Trusty Tahr）上安装 LXC 1.0：

1.  安装主要的 LXC 包、工具和依赖项：

    ```
    root@ubuntu:~# lsb_release -dc
    Description:       Ubuntu 14.04.5 LTS
    Codename:          trusty 
    root@ubuntu:~# apt-get -y install -y lxc bridge-utils 
          debootstrap libcap-dev 
          cgroup-bin libpam-systemdbridge-utils
    root@ubuntu:~#

    ```

1.  Trusty Tahr 当前提供的包版本是 1.0.8：

    ```
    root@ubuntu:~# dpkg --list | grep lxc | awk '{print $2,$3}'
    liblxc1 1.0.8-0ubuntu0.3
    lxc 1.0.8-0ubuntu0.3
    lxc-templates 1.0.8-0ubuntu0.3
    python3-lxc 1.0.8-0ubuntu0.3
    root@ubuntu:~# 

    ```

要安装 LXC 2.0，我们需要使用 Backports 仓库：

1.  在`apt`源文件中添加以下两行：

    ```
    root@ubuntu:~# vim /etc/apt/sources.list
    deb http://archive.ubuntu.com/ubuntu trusty-backports main 
          restricted universe multiverse
    deb-src http://archive.ubuntu.com/ubuntu trusty-backports 
          main restricted universe multiverse

    ```

1.  从源站点重新同步软件包索引文件：

    ```
    root@ubuntu:~# apt-get update

    ```

1.  安装主要的 LXC 包、工具和依赖项：

    ```
    root@ubuntu:~# apt-get -y install -y 
          lxc=2.0.3-0ubuntu1~ubuntu14.04.1 
          lxc1=2.0.3-0ubuntu1~ubuntu14.04.1 
          liblxc1=2.0.3-0ubuntu1~ubuntu14.04.1 python3-
          lxc=2.0.3-0ubuntu1~ubuntu14.04.1 cgroup-
          lite=1.11~ubuntu14.04.2 
          lxc-templates=2.0.3-0ubuntu1~ubuntu14.04.1bridge-utils
    root@ubuntu:~#

    ```

1.  确保软件包版本在 2.x 分支上，本例中为 2.0.3：

    ```
    root@ubuntu:~# dpkg --list | grep lxc | awk '{print $2,$3}'
    liblxc1 2.0.3-0ubuntu1~ubuntu14.04.1
    lxc2.0.3-0ubuntu1~ubuntu14.04.1
    lxc-common    2.0.3-0ubuntu1~ubuntu14.04.1
    lxc-templates 2.0.3-0ubuntu1~ubuntu14.04.1
    lxc1          2.0.3-0ubuntu1~ubuntu14.04.1
    lxcfs         2.0.2-0ubuntu1~ubuntu14.04.1
    python3-lxc   2.0.3-0ubuntu1~ubuntu14.04.1
    root@ubuntu:~#

    ```

## 从源代码在 Ubuntu 上安装 LXC

要使用最新版的 LXC，您可以从上游的 GitHub 仓库下载源代码并进行编译：

1.  首先，安装`git`并克隆仓库：

    ```
    root@ubuntu:~# apt-get install git
    root@ubuntu:~# cd /usr/src
    root@ubuntu:/usr/src# git clone https://github.com/lxc/lxc.git
    Cloning into 'lxc'...
    remote: Counting objects: 29252, done.
    remote: Compressing objects: 100% (156/156), done.
    remote: Total 29252 (delta 101), reused 0 (delta 0), 
          pack-reused 29096
    Receiving objects: 100% (29252/29252), 11.96 MiB | 12.62 
          MiB/s, done.
    Resolving deltas: 100% (21389/21389), done.
    root@ubuntu:/usr/src#

    ```

1.  接下来，让我们安装构建工具和各种依赖项：

    ```
    root@ubuntu:/usr/src# apt-get install -y dev-utils 
          build-essential aclocal automake pkg-config git bridge-utils 
          libcap-dev libcgmanager-dev cgmanager
    root@ubuntu:/usr/src#

    ```

1.  现在，生成`configure`脚本，该脚本将尝试为编译过程中使用的不同系统相关变量猜测正确的值：

    ```
    root@ubuntu:/usr/src# cd lxc
    root@ubuntu:/usr/src/lxc#./autogen.sh

    ```

1.  `configure`脚本提供了可以启用或禁用的选项，具体取决于您希望编译哪些功能。要了解可用的选项，并查看每个选项的简短描述，请运行以下命令：

    ```
    root@ubuntu:/usr/src/lxc# ./configure -help

    ```

1.  现在是运行`configure`的时间了。在这个例子中，我将启用 Linux 能力和`cgmanager`，它将管理每个容器的 cgroup：

    ```
    root@ubuntu:/usr/src/lxc# ./configure --enable-capabilities 
          --enable-cgmanager
    ...
    ----------------------------
    Environment:
     - compiler: gcc
     - distribution: ubuntu
     - init script type(s): upstart,systemd
     - rpath: no
     - GnuTLS: no
     - Bash integration: yes
    Security features:
     - Apparmor: no
     - Linux capabilities: yes
     - seccomp: no
     - SELinux: no
     - cgmanager: yes
    Bindings:
     - lua: no
     - python3: no
    Documentation:
     - examples: yes
     - API documentation: no
     - user documentation: no
    Debugging:
     - tests: no
     - mutex debugging: no
    Paths:
    Logs in configpath: no
    root@ubuntu:/usr/src/lxc#

    ```

    从前面的简短输出中，我们可以看到编译后将会有哪些选项可用。请注意，我们现在没有启用任何安全功能，比如`Apparmor`。

1.  接下来，使用`make`进行编译：

    ```
    root@ubuntu:/usr/src/lxc# make

    ```

1.  最后，安装二进制文件、库和模板：

    ```
    root@ubuntu:/usr/src/lxc# make install

    ```

    截至本文写作时，LXC 的二进制文件将其库文件搜索路径与安装路径不同。要修复此问题，只需将其复制到正确的位置：

    ```
    root@ubuntu:/usr/src/lxc# cp /usr/local/lib/liblxc.so* 
          /usr/lib/x86_64-linux-gnu/

    ```

1.  要检查已编译并安装的版本，请执行以下代码：

    ```
    root@ubuntu:/usr/src/lxc# lxc-create --version
    2.0.0
    root@ubuntu:/usr/src/lxc# 

    ```

## 使用 yum 在 CentOS 上安装 LXC

CentOS 7 当前在其上游仓库中提供 LXC 1.0.8 版本。以下说明应该适用于 RHEL 7 和 CentOS 7：

1.  安装主包和发行版模板：

    ```
    root@centos:~# cat /etc/redhat-release
    CentOS Linux release 7.2.1511 (Core)
    root@centos:~# yum install -y lxc lxc-templates
    root@centos:~#

    ```

1.  检查已安装的包版本：

    ```
    root@centos:~# rpm -qa | grep lxc
    lua-lxc-1.0.8-1.el7.x86_64
    lxc-templates-1.0.8-1.el7.x86_64
    lxc-libs-1.0.8-1.el7.x86_64
    lxc-1.0.8-1.el7.x86_64
    root@centos:~#

    ```

## 从源代码在 CentOS 上安装 LXC

要安装最新版本的 LXC，我们需要从 GitHub 下载并编译它，类似于我们在 Ubuntu 上所做的：

1.  安装构建工具、`git` 和各种依赖项：

    ```
    root@centos:# cd /usr/src
    root@centos:/usr/src# yum install -y libcap-devel libcgroup 
          bridge-utils git
    root@centos:/usr/src# yum groupinstall "Development tools"
    root@centos:/usr/src#

    ```

1.  接下来，克隆仓库：

    ```
    root@centos:/usr/src# git clone https://github.com/lxc/lxc.git
    root@centos:/usr/src# cd lxc/
    root@centos:/usr/src/lxc#

    ```

1.  生成配置文件：

    ```
    root@centos:/usr/src/lxc# ./autogen.sh
    root@centos:/usr/src#

    ```

1.  准备软件进行编译：

    ```
    root@centos:/usr/src/lxc# ./configure
    ...
    ----------------------------
    Environment:
     - compiler: gcc
     - distribution: centos
     - init script type(s): sysvinit
     - rpath: no
     - GnuTLS: no
     - Bash integration: yes
    Security features:
     - Apparmor: no
     - Linux capabilities: yes
     - seccomp: no
     - SELinux: no
     - cgmanager: no
    Bindings:
     - lua: no
     - python3: no
    Documentation:
     - examples: yes
     - API documentation: yes
     - user documentation: no
    Debugging:
     - tests: no
     - mutex debugging: no
    Paths:
    Logs in configpath: no
    root@centos:/usr/src/lxc#

    ```

1.  编译并安装二进制文件、库文件和发行版模板：

    ```
     root@centos:/usr/src/lxc# make && make install

    ```

1.  将库文件复制到二进制文件预期的位置：

    ```
    root@centos:/usr/src/lxc# cp /usr/local/lib/liblxc.so* 
          /usr/lib64/

    ```

1.  最后，要检查已编译和安装的版本，请执行以下代码：

    ```
    root@centos:/usr/src/lxc# lxc-create --version
    2.0.0
    root@centos:/usr/src/lxc#

    ```

    CentOS 7 使用 `systemd` 作为其初始化系统。要启动 LXC 服务，请运行以下命令：

    ```
    root@centos:/usr/src/lxc# systemctl start lxc.service
    root@centos:/usr/src/lxc# systemctl status lxc.service
     * lxc.service - LXC Container Initialization and Autoboot 
          Code
    Loaded: loaded (/usr/lib/systemd/system/lxc.service; 
          disabled; vendor preset: disabled)
    Active: active (exited) since Tue 2016-08-30 20:03:58 
          UTC; 6min ago
    Process: 10645 ExecStart=/usr/libexec/lxc/lxc-autostart-
          helper start (code=exited, status=0/SUCCESS)
    Process: 10638 ExecStartPre=/usr/libexec/lxc/lxc-devsetup 
          (code=exited, status=0/SUCCESS)
    Main PID: 10645 (code=exited, status=0/SUCCESS)
    CGroup: /system.slice/lxc.service
    Aug 30 20:03:28 centos systemd[1]: Starting LXC Container 
          Initialization and Autoboot Code...
    Aug 30 20:03:28 centos lxc-devsetup[10638]: Creating 
          /dev/.lxc
    Aug 30 20:03:28 centos lxc-devsetup[10638]: /dev is devtmpfs
    Aug 30 20:03:28 centos lxc-devsetup[10638]: Creating 
          /dev/.lxc/user
    Aug 30 20:03:58 centos lxc-autostart-helper[10645]: Starting 
          LXC autoboot containers:  [  OK  ]
    Aug 30 20:03:58 nova systemd[1]: Started LXC Container 
          Initialization and Autoboot Code.
    root@centos:/usr/src/lxc#

    ```

    为了确保 LXC 在安装过程中配置正确，请运行以下命令：

    ```
    root@centos:/usr/src/lxc# lxc-checkconfig
    Kernel configuration found at /boot/config-
          3.10.0-327.28.2.el7.x86_64
    --- Namespaces ---
    Namespaces: enabled
    Utsname namespace: enabled
    Ipc namespace: enabled
    Pid namespace: enabled
    User namespace: enabled
    Network namespace: enabled
    Multiple /dev/pts instances: enabled
    --- Control groups ---
    Cgroup: enabled
    Cgroup clone_children flag: enabled
    Cgroup device: enabled
    Cgroup sched: enabled
    Cgroup cpu account: enabled
    Cgroup memory controller: enabled
    Cgroup cpuset: enabled
    --- Misc ---
    Veth pair device: enabled
    Macvlan: enabled
    Vlan: enabled
    Bridges: enabled
    Advanced netfilter: enabled
    CONFIG_NF_NAT_IPV4: enabled
    CONFIG_NF_NAT_IPV6: enabled
    CONFIG_IP_NF_TARGET_MASQUERADE: enabled
    CONFIG_IP6_NF_TARGET_MASQUERADE: enabled
    CONFIG_NETFILTER_XT_TARGET_CHECKSUM: enabled
    --- Checkpoint/Restore ---
    checkpoint restore: enabled
    CONFIG_FHANDLE: enabled
    CONFIG_EVENTFD: enabled
    CONFIG_EPOLL: enabled
    CONFIG_UNIX_DIAG: enabled
    CONFIG_INET_DIAG: enabled
    CONFIG_PACKET_DIAG: enabled
    CONFIG_NETLINK_DIAG: enabled
    File capabilities: enabled
     Note : Before booting a new kernel, you can check its 
     configuration:
    usage : CONFIG=/path/to/config /usr/bin/lxc-checkconfig
    root@centos:/usr/src/lxc#

    ```

## LXC 目录安装布局

以下表格显示了在包和源代码安装后创建的 LXC 目录布局。目录的设置因发行版和安装方法而异：

| **Ubuntu 包** | **CentOS 包** | **源代码安装** | **描述** |
| --- | --- | --- | --- |
| `/usr/share/lxc` | `/usr/share/` `lxc` | `/usr/local/share/` `lxc` | LXC 基础目录 |
| `/usr/share/lxc/` `config` | `/usr/share/lxc/` `config` | `/usr/local/share/lxc/` `config` | 基于发行版的 LXC 配置文件集合 |
| `/usr/share/lxc/` `templates` | `/usr/share/lxc/` `templates` | `/usr/local/share/lxc/` `template`s | 容器模板脚本集合 |
| `/usr/bin` | `/usr/bin` | `/usr/local/bin` | 大多数 LXC 二进制文件的位置 |
| `/usr/lib/x86_64-linux-gnu` | `/usr/lib64` | `/usr/local/lib` | `liblxc` 库的位置 |
| `/etc/lxc` | `/etc/lxc` | `/usr/local/etc/` `lxc` | 默认 LXC 配置文件的位置 |
| `/var/lib/` `lxc/` | `/var/lib/` `lxc/` | `/usr/local/var/` `lib/lxc/` | 创建的容器的根文件系统和配置位置 |
| `/var/log/lxc` | `/var/log/lxc` | `/usr/local/var/` `log/lxc` | LXC 日志文件 |

在构建、启动和终止 LXC 容器时，我们将探索大多数目录。

### 提示

在从源代码构建 LXC 时，您可以通过传递参数给配置脚本（例如 `configure --prefix`）来更改默认的安装路径。

# 构建和操作 LXC 容器

与手动创建命名空间并通过 cgroups 应用资源限制相比，使用提供的用户空间工具来管理容器生命周期非常方便。本质上，这正是 LXC 工具所做的：创建和操作我们在第一章，*Linux 容器简介* 中看到的命名空间和 cgroups。LXC 工具实现了在第四章，*LXC 与 Python 的代码集成* 中将要看到的 `liblxc` API 中定义的功能。

LXC 附带了多种模板，用于为不同的 Linux 发行版构建根文件系统。我们可以使用这些模板来创建各种类型的容器。例如，在 CentOS 主机上运行 Debian 容器。我们也可以选择使用`debootstrap`和`yum`等工具来构建自己的根文件系统，稍后我们将探讨这一点。

## 构建我们的第一个容器

我们可以使用模板创建我们的第一个容器。`lxc-download`文件像`templates`目录中的其他模板一样，是一个用 bash 编写的脚本：

```
root@ubuntu:~# ls -la /usr/share/lxc/templates/
drwxr-xr-x 2 root root  4096 Aug 29 20:03 .
drwxr-xr-x 6 root root  4096 Aug 29 19:58 ..
-rwxr-xr-x 1 root root 10557 Nov 18  2015 lxc-alpine
-rwxr-xr-x 1 root root 13534 Nov 18  2015 lxc-altlinux
-rwxr-xr-x 1 root root 10556 Nov 18  2015 lxc-archlinux
-rwxr-xr-x 1 root root  9878 Nov 18  2015 lxc-busybox
-rwxr-xr-x 1 root root 29149 Nov 18  2015 lxc-centos
-rwxr-xr-x 1 root root 10486 Nov 18  2015 lxc-cirros
-rwxr-xr-x 1 root root 17354 Nov 18  2015 lxc-debian
-rwxr-xr-x 1 root root 17757 Nov 18  2015 lxc-download
-rwxr-xr-x 1 root root 49319 Nov 18  2015 lxc-fedora
-rwxr-xr-x 1 root root 28253 Nov 18  2015 lxc-gentoo
-rwxr-xr-x 1 root root 13962 Nov 18  2015 lxc-openmandriva
-rwxr-xr-x 1 root root 14046 Nov 18  2015 lxc-opensuse
-rwxr-xr-x 1 root root 35540 Nov 18  2015 lxc-oracle
-rwxr-xr-x 1 root root 11868 Nov 18  2015 lxc-plamo
-rwxr-xr-x 1 root root  6851 Nov 18  2015 lxc-sshd
-rwxr-xr-x 1 root root 23494 Nov 18  2015 lxc-ubuntu
-rwxr-xr-x 1 root root 11349 Nov 18  2015 lxc-ubuntu-cloud
root@ubuntu:~#

```

如果你仔细检查这些脚本，你会注意到它们大多数是创建`chroot`环境，在这些环境中会安装包和各种配置文件，以便为选定的发行版创建根文件系统。

让我们首先使用`lxc-download`模板来构建一个容器，这会要求提供发行版、版本和架构，然后使用适当的模板为我们创建文件系统和配置：

```
root@ubuntu:~# lxc-create -t download -n c1
Setting up the GPG keyring
Downloading the image index
---
DIST       RELEASE    ARCH       VARIANT    BUILD
---
centos    6          amd64      default    20160831_02:16
centos    6          i386       default    20160831_02:16
centos    7          amd64      default    20160831_02:16
debian    jessie     amd64      default    20160830_22:42
debian    jessie     arm64      default    20160824_22:42
debian    jessie     armel      default    20160830_22:42
... 
ubuntu    trusty     amd64      default    20160831_03:49
ubuntu    trusty     arm64      default    20160831_07:50
ubuntu    yakkety    s390x      default    20160831_03:49
---
Distribution: ubuntu
Release: trusty
Architecture: amd64
Unpacking the rootfs
---
You just created an Ubuntu container (release=trusty, arch=amd64, variant=default)
To enable sshd, run: apt-get install openssh-server
For security reason, container images ship without user accounts
and without a root password.
Use lxc-attach or chroot directly into the rootfs to set a root password
or 
create user accounts.
root@ubuntu:~#

```

让我们列出所有容器：

```
root@ubuntu:~# lxc-ls -f
NAME                  STATE    IPV4  IPV6  AUTOSTART
----------------------------------------------------
c1                    STOPPED  -     -     NO
root@nova-perf:~#

```

### 提示

根据 LXC 的版本，一些命令选项可能会有所不同。如果遇到错误，请阅读每个工具的手册页面。

我们的容器目前没有运行，让我们在后台启动它并将日志级别提高到`DEBUG`：

```
root@ubuntu:~# lxc-start -n c1 -d -l DEBUG

```

### 提示

在某些发行版中，LXC 在构建第一个容器时不会创建宿主桥接，这会导致错误。如果发生这种情况，可以通过运行`brctl addbr virbr0`命令来创建它。

执行以下命令列出所有容器：

```
root@ubuntu:~# lxc-ls -f
NAME                  STATE    IPV4        IPV6  AUTOSTART
----------------------------------------------------------
c1                    RUNNING  10.0.3.190  -     NO
root@ubuntu:~# 

```

要获取有关容器的更多信息，请运行以下命令：

```
root@ubuntu:~# lxc-info -n c1
Name:           c1
State:          RUNNING
PID:            29364
IP:             10.0.3.190
CPU use:        1.46 seconds
BlkIO use:      112.00 KiB
Memory use:     6.34 MiB
KMem use:       0 bytes
Link:           vethVRD8T2
 TX bytes:      4.28 KiB
 RX bytes:      4.43 KiB
 Total bytes:   8.70 KiB
root@ubuntu:~#

```

新的容器现在已连接到宿主机的桥接`lxcbr0`：

```
root@ubuntu:~# brctl show
bridge name        bridge id              STP enabled        interfaces
lxcbr0         8000.fea50feb48ac          no             vethVRD8T2 
root@ubuntu:~# ip a s lxcbr0
4: lxcbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
link/ether fe:a5:0f:eb:48:ac brd ff:ff:ff:ff:ff:ff
inet 10.0.3.1/24 brd 10.0.3.255 scope global lxcbr0
valid_lft forever preferred_lft forever
inet6 fe80::465:64ff:fe49:5fb5/64 scope link
valid_lft forever preferred_lft forever 
root@ubuntu:~# ip a s vethVRD8T2
8: vethVRD8T2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master lxcbr0 state UP group default qlen 1000
link/ether fe:a5:0f:eb:48:ac brd ff:ff:ff:ff:ff:ff
inet6 fe80::fca5:fff:feeb:48ac/64 scope link
valid_lft forever preferred_lft forever
root@ubuntu:~#

```

使用下载模板并未指定任何网络设置时，容器会从`dnsmasq`服务器获取 IP 地址，这个服务器在私有网络`10.0.3.0/24`上运行。宿主机通过`iptables`中的 NAT 规则允许容器连接到其他网络和互联网：

```
root@ubuntu:~# iptables -L -n -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  10.0.3.0/24         !10.0.3.0/24
root@ubuntu:~#

```

连接到桥接的其他容器可以相互访问并访问宿主机，只要它们都连接到同一个桥接且未标记为不同的 VLAN ID。

让我们看看容器启动后进程树的样子：

```
root@ubuntu:~# ps axfww
...
1552 ?        S      0:00 dnsmasq -u lxc-dnsmasq --strict-order --bind-interfaces --pid-file=/run/lxc/dnsmasq.pid --conf-file= --listen-address 10.0.3.1 --dhcp-range 10.0.3.2,10.0.3.254 --dhcp-lease-max=253 --dhcp-no-override --except-interface=lo --interface=lxcbr0 --dhcp-leasefile=/var/lib/misc/dnsmasq.lxcbr0.leases --dhcp-authoritative
29356 ?        Ss     0:00 lxc-start -n c1 -d -l DEBUG
29364 ?        Ss     0:00  \_ /sbin/init
29588 ?        S      0:00      \_ upstart-udev-bridge --daemon
29597 ?        Ss     0:00      \_ /lib/systemd/systemd-udevd --daemon
29667 ?        Ssl    0:00      \_ rsyslogd
29688 ?        S      0:00      \_ upstart-file-bridge --daemon
29690 ?        S      0:00      \_ upstart-socket-bridge --daemon
29705 ?        Ss     0:00      \_ dhclient -1 -v -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases eth0
29775 pts/6    Ss+    0:00      \_ /sbin/getty -8 38400 tty4
29777 pts/1    Ss+    0:00      \_ /sbin/getty -8 38400 tty2
29778 pts/5    Ss+    0:00      \_ /sbin/getty -8 38400 tty3
29787 ?        Ss     0:00      \_ cron
29827 pts/7    Ss+    0:00      \_ /sbin/getty -8 38400 console
29829 pts/0    Ss+    0:00      \_ /sbin/getty -8 38400 tty1
root@ubuntu:~#

```

注意新生成的`init`子进程，它是从`lxc-start`命令克隆出来的。它在实际容器中的 PID 是`1`。

接下来，让我们运行`attach`命令与容器交互，列出所有进程和网络接口，并检查连接性：

```
root@ubuntu:~# lxc-attach -n c1
root@c1:~# ps axfw
PID TTY      STAT   TIME COMMAND
1 ?        Ss     0:00 /sbin/init
176 ?        S      0:00 upstart-udev-bridge --daemon
185 ?        Ss     0:00 /lib/systemd/systemd-udevd --daemon
255 ?        Ssl    0:00 rsyslogd
276 ?        S      0:00 upstart-file-bridge --daemon
278 ?        S      0:00 upstart-socket-bridge --daemon
293 ?        Ss     0:00 dhclient -1 -v -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases eth0
363 lxc/tty4 Ss+    0:00 /sbin/getty -8 38400 tty4
365 lxc/tty2 Ss+    0:00 /sbin/getty -8 38400 tty2
366 lxc/tty3 Ss+    0:00 /sbin/getty -8 38400 tty3
375 ?        Ss     0:00 cron
415 lxc/console Ss+   0:00 /sbin/getty -8 38400 console
417 lxc/tty1 Ss+    0:00 /sbin/getty -8 38400 tty1
458 ?        S      0:00 /bin/bash
468 ?        R+     0:00 ps ax 
root@c1:~# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid_lft forever preferred_lft forever
inet6 ::1/128 scope host
valid_lft forever preferred_lft forever
7: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
link/ether 00:16:3e:b2:34:8a brd ff:ff:ff:ff:ff:ff
inet 10.0.3.190/24 brd 10.0.3.255 scope global eth0
valid_lft forever preferred_lft forever
inet6 fe80::216:3eff:feb2:348a/64 scope link
valid_lft forever preferred_lft forever 
root@c1:~# ping -c 3 google.com
PING google.com (216.58.192.238) 56(84) bytes of data.
64 bytes from ord30s26-in-f14.1e100.net (216.58.192.238): icmp_seq=1 ttl=52 time=1.77 ms
64 bytes from ord30s26-in-f14.1e100.net (216.58.192.238): icmp_seq=2 ttl=52 time=1.58 ms
64 bytes from ord30s26-in-f14.1e100.net (216.58.192.238): icmp_seq=3 ttl=52 time=1.75 ms
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.584/1.705/1.779/0.092 ms 
root@c1:~# exit
exit
root@ubuntu:~#

```

### 提示

在某些发行版中，如 CentOS，或者如果是从源代码安装的，`dnsmasq`服务器默认未配置和启动。你可以手动安装并配置它，或者按我在本章后面展示的那样，给容器配置 IP 地址和默认网关。

注意，一旦我们连接到容器，终端的主机名就发生了变化。这是 LXC 如何使用 UTS 命名空间的一个例子，正如我们在第一章，*Linux 容器简介*中看到的。

让我们检查一下构建 `c1` 容器后创建的目录：

```
root@ubuntu:~# ls -la /var/lib/lxc/c1/
total 16
drwxrwx---  3 root root 4096 Aug 31 20:40 .
drwx------  3 root root 4096 Aug 31 21:01 ..
-rw-r--r--  1 root root  516 Aug 31 20:40 config
drwxr-xr-x 21 root root 4096 Aug 31 21:00 rootfs
root@ubuntu:~#

```

`rootfs` 目录看起来像一个普通的 Linux 文件系统。您可以通过直接修改那里文件或使用 `chroot` 来操作容器。

为了演示这一点，让我们通过 *chroot* 到其 `rootfs` 来更改 `c1` 容器的根密码，而不是通过连接到容器来实现：

```
root@ubuntu:~# cd /var/lib/lxc/c1/
root@ubuntu:/var/lib/lxc/c1# chroot rootfs
root@ubuntu:/# ls -al
total 84
drwxr-xr-x 21 root root 4096 Aug 31 21:00 .
drwxr-xr-x 21 root root 4096 Aug 31 21:00 ..
drwxr-xr-x  2 root root 4096 Aug 29 07:33 bin
drwxr-xr-x  2 root root 4096 Apr 10  2014 boot
drwxr-xr-x  4 root root 4096 Aug 31 21:00 dev
drwxr-xr-x 68 root root 4096 Aug 31 22:12 etc
drwxr-xr-x  3 root root 4096 Aug 29 07:33 home
drwxr-xr-x 12 root root 4096 Aug 29 07:33 lib
drwxr-xr-x  2 root root 4096 Aug 29 07:32 lib64
drwxr-xr-x  2 root root 4096 Aug 29 07:31 media
drwxr-xr-x  2 root root 4096 Apr 10  2014 mnt
drwxr-xr-x  2 root root 4096 Aug 29 07:31 opt
drwxr-xr-x  2 root root 4096 Apr 10  2014 proc
drwx------  2 root root 4096 Aug 31 22:12 root
drwxr-xr-x  8 root root 4096 Aug 31 20:54 run
drwxr-xr-x  2 root root 4096 Aug 29 07:33 sbin
drwxr-xr-x  2 root root 4096 Aug 29 07:31 srv
drwxr-xr-x  2 root root 4096 Mar 13  2014 sys
drwxrwxrwt  2 root root 4096 Aug 31 22:12 tmp
drwxr-xr-x 10 root root 4096 Aug 29 07:31 usr
drwxr-xr-x 11 root root 4096 Aug 29 07:31 var
root@ubuntu:/# passwd
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully 
root@ubuntu:/# exit
exit
root@ubuntu:/var/lib/lxc/c1#

```

注意，当我们使用 `chroot` 并在退出受限环境后，控制台上的路径发生了变化。

为了测试根密码，我们首先连接到容器，然后使用 `ssh` 连接并安装 **安全外壳** (**SSH**) 服务器：

```
root@ubuntu:~# lxc-attach -n c1
root@c1:~# apt-get update&&apt-get install -y openssh-server
root@c1:~# sed -i 's/without-password/yes/g' /etc/ssh/sshd_config
root@c1:~#service ssh restart
root@c1:/# exit
exit 
root@ubuntu:/var/lib/lxc/c1# ssh 10.0.3.190
root@10.0.3.190's password:
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-91-generic x86_64)
* Documentation:  https://help.ubuntu.com/
Last login: Wed Aug 31 22:25:39 2016 from 10.0.3.1
root@c1:~# exit
logout
Connection to 10.0.3.190 closed.
root@ubuntu:/var/lib/lxc/c1#

```

我们成功地通过 `ssh` 连接到容器，并使用之前手动设置的根密码。

## 在 Ubuntu 上使用 debootstrap 创建自定义容器

使用提供的发行版模板脚本和配置文件是最迅速的 LXC 配置方法。然而，要完全控制根文件系统的布局——应该包含哪些软件包、块设备和网络设置——则需要更为手动的方法。

为此，我们可以使用 `debootstrap` 工具来创建容器的 `rootfs`，然后手动创建描述容器属性的配置文件。这在 Ubuntu 和 RHEL/CentOS 发行版上有效，并且提供了一种在 RHEL/CentOS 和 Debian/Ubuntu 系统上运行 Debian 和 Ubuntu 容器的方式。

首先安装`debootstrap`，如果尚未安装。在 Ubuntu 上，运行以下命令：

```
root@ubuntu:~# apt-get install -y debootstrap

```

类似地，在 CentOS 上，运行以下命令：

```
root@centos:~# yum install -y debootstrap

```

为了创建稳定的 Debian 发行版文件系统，我们可以提供以下参数：

```
root@ubuntu:~# debootstrap --arch=amd64 --include="openssh-server vim" stable ~/container http://httpredir.debian.org/debian/
W: Cannot check Release signature; keyring file not available /usr/share/keyrings/debian-archive-keyring.gpg
I: Retrieving Release
I: Retrieving Packages
I: Validating Packages
...
I: Configuring libc-bin...
I: Configuring systemd...
I: Base system installed successfully.
root@ubuntu:~#

```

我们指定了架构、要安装的软件包、操作系统的发行版以及 `rootfs` 将创建的位置，在本例中是 `~/container`。

接下来，我们需要为容器准备一个 `config` 文件。指定各种 LXC 属性有许多可选项。让我们从一个相对简单的配置开始：

```
root@ubuntu:~# vim ~/config
lxc.devttydir = lxc
lxc.pts = 1024
lxc.tty = 4
lxc.pivotdir = lxc_putold
lxc.cgroup.devices.deny = a
lxc.cgroup.devices.allow = c *:* m
lxc.cgroup.devices.allow = b *:* m
lxc.cgroup.devices.allow = c 1:3 rwm
lxc.cgroup.devices.allow = c 1:5 rwm
lxc.cgroup.devices.allow = c 1:7 rwm
lxc.cgroup.devices.allow = c 5:0 rwm
lxc.cgroup.devices.allow = c 5:1 rwm
lxc.cgroup.devices.allow = c 5:2 rwm
lxc.cgroup.devices.allow = c 1:8 rwm
lxc.cgroup.devices.allow = c 1:9 rwm
lxc.cgroup.devices.allow = c 136:* rwm
lxc.cgroup.devices.allow = c 10:229 rwm
lxc.cgroup.devices.allow = c 254:0 rm
lxc.cgroup.devices.allow = c 10:200 rwm
lxc.mount.auto = cgroup:mixed proc:mixed sys:mixed
lxc.mount.entry = /sys/fs/fuse/connections sys/fs/fuse/connections none bind,optional 0 0
lxc.mount.entry = /sys/kernel/debug sys/kernel/debug none bind,optional 0 0
lxc.mount.entry = /sys/kernel/security sys/kernel/security none bind,optional 0 0
lxc.mount.entry = /sys/fs/pstore sys/fs/pstore none bind,optional 0 0
lxc.mount.entry = mqueue dev/mqueue mqueue rw,relatime,create=dir,optional 0 0
# Container specific configuration
lxc.arch = x86_64
lxc.rootfs = /root/container
lxc.rootfs.backend = dir
lxc.utsname = manual_container
# Network configuration
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:e4:68:91
lxc.network.ipv4 = 10.0.3.151/24 10.0.3.255
lxc.network.ipv4.gateway = 10.0.3.1
# Limit the container memory to 512MB
lxc.cgroup.memory.limit_in_bytes = 536870912

```

### 注意

上述配置是在 LXC 2.0 上测试的，可能与 1.0 版本分支上的版本不兼容。

以下表格简要总结了我们正在使用的最重要的选项：

| **Option** | **Description** |
| --- | --- |
| `lxc.devttydir` | 控制台设备在 `/dev/` 中的位置 |
| `lxc.tty` | 为容器提供的 TTY 数量 |
| `lxc.cgroup.devices.allow` | 容器中允许的设备列表 |
| `lxc.mount` | 要挂载的设备 |
| `lxc.arch` | 容器的架构 |
| `lxc.rootfs` | 根文件系统的位置 |
| `lxc.rootfs.backend` | 后端存储的类型 |
| `lxc.utsname` | 容器的主机名 |
| `lxc.network.type` | 网络虚拟化的类型 |
| `lxc.network.link` | 容器将连接的主机桥接器的名称 |
| `lxc.network.flags` | 启动网络接口 |
| `lxc.network.hwaddr` | 网络接口的 MAC 地址 |
| `lxc.network.ipv4` | 如果不使用 DHCP，网络接口的 IP 地址 |
| `lxc.network.ipv4.gateway` | 容器内的默认网关 |
| `lxc.cgroup` | 设置 cgroup 参数，如`memory`、`cpu`和`blkio` |

### 提示

欲了解所有可用的配置选项，请参阅`lxc.container.conf`手册页。

配置完成后，让我们创建容器：

```
root@ubuntu:~# lxc-create --name manual_container -t none -B dir --dir ~/container -f ~/config

```

注意，这次我们没有指定模板，而是提供了之前用`debootstrap`创建的`rootfs`。

让我们通过将日志设置为`DEBUG`并将其重定向到一个文件来启动容器，以防我们需要排查错误：

```
root@ubuntu:~# lxc-start -n manual_container -l DEBUG -o container.log
root@ubuntu:~# lxc-ls -f
NAME             STATE   AUTOSTART GROUPS IPV4       IPV6
manual_container RUNNING 0         -      10.0.3.151 - 
root@ubuntu:~# lxc-info -n manual_container
Name:           manual_container
State:          RUNNING
PID:            4283
IP:             10.0.3.151
CPU use:        0.21 seconds
BlkIO use:      0 bytes
Memory use:     11.75 MiB
KMem use:       0 bytes
Link:           vethU29DXE
TX bytes:      690 bytes
RX bytes:      840 bytes
Total bytes:   1.49 KiB
root@ubuntu:~#

```

为了测试，我们可以像往常一样运行`attach`：

```
root@ubuntu:~# lxc-attach -n manual_container
[root@manual_container ~]# exit
exit
root@ubuntu:~#

```

## 使用 yum 在 CentOS 上制作自定义容器

在 CentOS 7 上，我们可以使用`debootstrap`工具创建基于 Debian 和 Ubuntu 的根文件系统，并像前一节中描述的那样构建容器。

然而，要构建 RHEL、Fedora 或 CentOS 容器，我们需要使用像`rpm`、`yumdownloader`和`yum`这样的工具。让我们看一个稍微复杂一点的例子，构建一个新的 CentOS `rootfs`，容器将使用它：

1.  首先，创建一个将包含根文件系统的目录：

    ```
    root@centos:~# mkdir container

    ```

1.  在创建`container`目录后，我们需要创建并初始化`rpm`软件包数据库：

    ```
    root@centos:~# rpm --root /root/container -initdb
    root@centos:~# ls -la container/var/lib/rpm/
    total 108
    drwxr-xr-x. 2 root root  4096 Sep  1 19:13 .
    drwxr-xr-x. 3 root root    16 Sep  1 19:13 ..
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Basenames
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Conflictname
    -rw-r--r--. 1 root root     0 Sep  1 19:13 .dbenv.lock
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Dirnames
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Group
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Installtid
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Name
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Obsoletename
    -rw-r--r--. 1 root root 12288 Sep  1 19:13 Packages
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Providename
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Requirename
    -rw-r--r--. 1 root root     0 Sep  1 19:13 .rpm.lock
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Sha1header
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Sigmd5
    -rw-r--r--. 1 root root  8192 Sep  1 19:13 Triggername
    root@centos:~#

    ```

1.  数据库初始化完成后，我们可以下载 CentOS 发行版的发布文件。如果你更倾向于构建一个 Fedora 容器，可以在命令行中将`centos-release`替换为`fedora-release`。发布文件包含`yum`仓库和`yum`及`rpm`使用的其他重要文件：

    ```
    root@centos:~# yumdownloader --destdir=/tmp centos-release
    Loaded plugins: fastestmirror
    Loading mirror speeds from cached hostfile
    * base: mirrors.evowise.com
     * extras: mirror.netdepot.com
     * updates: mirror.symnds.com
    centos-release-7-2.1511.el7.centos.2.10.x86_64.rpm   
          | 23 kB  00:00:00
    root@centos:~# ls -la /tmp/centos-release
          -7-2.1511.el7.centos.2.10.x86_64.rpm
    -rw-r--r--. 1 root root 23516 Dec  9  2015 /tmp/centos-release-
          7-2.1511.el7.centos.2.10.x86_64.rpm
    root@centos:~# 

    ```

1.  接下来，安装`container`中`root`目录下的`rpm`软件包的发布文件：

    ```
    root@centos:~# rpm --root /root/container -ivh --nodeps 
          /tmp/centos-release-7-2.1511.el7.centos.2.10.x86_64.rpm
    warning: /tmp/centos-release-7-2.1511.el7.centos.2.10.x86_64.rpm: 
          Header V3 
          RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
    Preparing...    
          ################################# [100%]
    Updating / installing...
     1:centos-release-7-
          2.1511.el7.cento#################################[100%]
     root@centos:~# ls -la container/
    total 8
    drwxr-xr-x. 5 root root   36 Sep  1 19:19 .
    dr-xr-x---. 4 root root 4096 Sep  1 19:13 ..
    drwxr-xr-x. 6 root root 4096 Sep  1 19:19 etc
    drwxr-xr-x. 4 root root   28 Sep  1 19:19 usr
    drwxr-xr-x. 3 root root   16 Sep  1 19:13 var
    root@centos:~#

    ```

    从前面的输出可以看出，`container`文件系统开始成形。它目前包含了使用包管理器和安装其余系统文件所需的所有文件。

1.  现在，到了在`rootfs`中安装最小化的 CentOS 发行版的时刻；这与`debootstrap`在 Debian 和 Ubuntu 中所做的类似：

    ```
    root@centos:~# yum --installroot=/root/container -y group install 
          "Minimal install"
    Loaded plugins: fastestmirror
    There is no installed groups file.
    Maybe run: yum groups mark convert (see man yum)
    Determining fastest mirrors
     * base: centos.aol.com
     * extras: mirror.lug.udel.edu
     * updates: mirror.symnds.com
    Resolving Dependencies
    --> Running transaction check
    ---> Package NetworkManager.x86_64 1:1.0.6-30.el7_2 will be installed
    --> Processing Dependency: ppp = 2.4.5 for package: 1:NetworkManager-
          1.0.6-30.el7_2.x86_64
    --> Processing Dependency: NetworkManager-libnm(x86-64) = 
          1:1.0.6-30.el7_2 for 
          package: 1:NetworkManager-1.0.6-30.el7_2.x86_64
    --> Processing Dependency: wpa_supplicant >= 1:1.1 for 
          package: 1:NetworkManager-
          1.0.6-30.el7_2.x86_64
    --> Processing Dependency: libnl3 >= 3.2.21-7 for 
          package: 1:NetworkManager-
          1.0.6-30.el7_2.x86_64
    --> Processing Dependency: glib2 >= 2.32.0 for 
          package: 1:NetworkManager-
          1.0.6-30.el7_2.x86_64
    ...
    Complete!
    root@centos:~# 

    ```

1.  现在，`~/container`目录包含了完整的 CentOS 7 发行版的根文件系统：

    ```
    root@centos:~# ls -la container/
    total 32
    dr-xr-xr-x. 17 root root 4096 Sep  1 19:20 .
    dr-xr-x---.  4 root root 4096 Sep  1 19:13 ..
    lrwxrwxrwx.  1 root root    7 Sep  1 19:20 bin -> usr/bin
    dr-xr-xr-x.  3 root root   43 Sep  1 19:21 boot
    drwxr-xr-x.  2 root root   17 Sep  1 19:21 dev
    drwxr-xr-x. 74 root root 8192 Sep  1 19:21 etc
    drwxr-xr-x.  2 root root    6 Aug 12  2015 home
    lrwxrwxrwx.  1 root root    7 Sep  1 19:20 lib -> usr/lib
    lrwxrwxrwx.  1 root root    9 Sep  1 19:20 lib64 -> usr/lib64
    drwxr-xr-x.  2 root root    6 Aug 12  2015 media
    drwxr-xr-x.  2 root root    6 Aug 12  2015 mnt
    drwxr-xr-x.  2 root root    6 Aug 12  2015 opt
    dr-xr-xr-x.  2 root root    6 Aug 12  2015 proc
    dr-xr-x---.  2 root root   86 Sep  1 19:21 root
    drwxr-xr-x. 16 root root 4096 Sep  1 19:21 run
    lrwxrwxrwx.  1 root root    8 Sep  1 19:20 sbin -> usr/sbin
    drwxr-xr-x.  2 root root    6 Aug 12  2015 srv
    dr-xr-xr-x.  2 root root    6 Aug 12  2015 sys
    drwxrwxrwt.  7 root root   88 Sep  1 19:21 tmp
    drwxr-xr-x. 13 root root 4096 Sep  1 19:20 usr
    drwxr-xr-x. 19 root root 4096 Sep  1 19:21 var
    root@centos:~#

    ```

    如果你希望在容器构建之前安装更多的包或进行更改，可以通过`chroot`进入`~/containers`并像往常一样进行操作。

1.  接下来，如果桥接器尚未存在，创建它，并编写容器配置文件：

    ```
    root@centos:~# brct addbr lxcbr0
    root@centos:~# brct show
    bridge name    bridge id          STP enabled       interfaces
    lxcbr0         8000.000000000000          no 
    root@centos:~# vim config
    lxc.devttydir = lxc
    lxc.pts = 1024
    lxc.tty = 4
    lxc.pivotdir = lxc_putold
    lxc.cgroup.devices.deny = a
    lxc.cgroup.devices.allow = c *:* m
    lxc.cgroup.devices.allow = b *:* m
    lxc.cgroup.devices.allow = c 1:3 rwm
    lxc.cgroup.devices.allow = c 1:5 rwm
    lxc.cgroup.devices.allow = c 1:7 rwm
    lxc.cgroup.devices.allow = c 5:0 rwm
    lxc.cgroup.devices.allow = c 5:1 rwm
    lxc.cgroup.devices.allow = c 5:2 rwm
    lxc.cgroup.devices.allow = c 1:8 rwm
    lxc.cgroup.devices.allow = c 1:9 rwm
    lxc.cgroup.devices.allow = c 136:* rwm
    lxc.cgroup.devices.allow = c 10:229 rwm
    lxc.cgroup.devices.allow = c 254:0 rm
    lxc.cgroup.devices.allow = c 10:200 rwm
    lxc.mount.auto = cgroup:mixed proc:mixed sys:mixed
    lxc.mount.entry = /sys/fs/fuse/connections sys/fs/fuse/connections 
          none bind,optional 0 0
    lxc.mount.entry = /sys/kernel/debug sys/kernel/debug none 
          bind,optional 0 0
    lxc.mount.entry = /sys/kernel/security sys/kernel/security none 
          bind,optional 0 0
    lxc.mount.entry = /sys/fs/pstore sys/fs/pstore none bind,optional 0 0
    lxc.mount.entry = mqueue dev/mqueue mqueue 
          rw,relatime,create=dir,optional 0 0
    # Container specific configuration
    lxc.arch = x86_64
    lxc.rootfs = /root/container
    lxc.rootfs.backend = dir
    lxc.utsname = c1
    # Network configuration
    lxc.network.type = veth
    lxc.network.link = lxcbr0
    lxc.network.flags = up
    lxc.network.hwaddr = 00:16:3e:e4:68:92
    lxc.network.ipv4 = 10.0.3.152/24 10.0.3.255
    lxc.network.ipv4.gateway = 10.0.3.1

    ```

1.  配置好`rootfs`和`config`后，让我们创建并启动容器：

    ```
    root@centos:~# lxc-create --name c1 -t none -B dir --dir 
          ~/container -f ~/config
    root@centos:~# lxc-ls -f
    NAME STATE   AUTOSTART GROUPS IPV4       IPV6
    c1   RUNNING 0         -      10.0.3.151 -
    root@centos:~#

    ```

1.  最后，我们可以像往常一样运行`attach`：

    ```
    root@centos:~# lxc-attach -n c1
    [root@c1 ~]# exit
    exit
    root@centos:~# 

    ```

# 总结

LXC 提供了一套工具，使得构建、启动和操作容器变得非常简单和方便。使用包含的模板和配置文件可以进一步简化这一过程。在本章中，我们展示了如何在 Ubuntu 和 CentOS 发行版上安装、配置和启动 LXC 的实际示例。你学习了如何创建容器根文件系统，以及如何编写简单的配置文件。

在下一章，我们将了解如何在 LXC 中配置系统资源，并通过使用`libvirt`工具包和库，探索与 LXC 协同工作的替代方法。

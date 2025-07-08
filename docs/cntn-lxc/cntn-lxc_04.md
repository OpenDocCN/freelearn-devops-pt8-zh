# 第四章：LXC 与 Python 的代码集成

本章将介绍 LXC 和 libvirt API 提供的 Python 绑定。我们将探索哪些容器功能是可能的，哪些是不可能的，使用 Ubuntu 上的上游 `lxc-dev` 和 `python-libvirt` 包以及 CentOS 上的 `lxc-devel` 和 `libvirt-python` 包。

为了充分利用本章内容，需要具备一定的 Python 知识。如果你是开发者，本章可能是最重要的一章。

本章将按以下顺序涵盖以下主题：

+   使用 `lxc` Python 绑定构建和管理容器

+   使用 libvirt Python 绑定创建和编排容器

+   使用 LXC 作为 Vagrant 开发和测试的后端

+   开发一个简单的前端 RESTful API 来与 LXC 交互，使用 Bottle 微框架和 `lxc` 库

# LXC Python 绑定

LXC 提供稳定的 C API 和 Python 绑定，支持 Python 2.x 和 3.x 版本。让我们通过编写一个代码来探索一些功能，这些功能覆盖了我们在前几章中看到的用户空间工具所提供的大多数功能，使用 Python 2.7.6 版本。

## 在 Ubuntu 和 CentOS 上安装 LXC Python 绑定并准备开发环境

让我们从安装所有必需的包开始，这些包将使我们能够编写功能完整的 Python 代码，包括 LXC API 库和带有 `ipython` 和 `virtualenv` 的 Python 开发环境。

1.  要准备一个 Ubuntu 主机，请运行以下命令：

    ```
    root@ubuntu:~# apt-get update && apt-get upgrade && reboot
    root@ubuntu:~# apt-get install python-pip python-dev ipython
    root@ubuntu:~# apt-get install lxc-dev=2.0.3-0ubuntu1~ubuntu14.04.1 
    liblxc1=2.0.3-0ubuntu1~ubuntu14.04.1 cgroup-lite=1.11~ubuntu14.04.2 

    root@ubuntu:~# apt-get install 
    lxc-templates=2.0.3-0ubuntu1~ubuntu14.04.1
    lxc1=2.0.3-0ubuntu1~ubuntu14.04.1 
    python3-lxc=2.0.3-0ubuntu1~ubuntu14.04.1

    ```

    上述命令将确保我们运行的是最新的 Ubuntu 包，并包含诸如 `pip` 这样的工具，用于安装和管理 Python 包，以及用于 Python 交互式编程的 `ipython` 工具。在 CentOS 上，安装以下软件包以提供相同的功能：

    ```
    [root@centos ~]# yum update && reboot
    [root@centos ~]# yum install python-devel python-pip lxc 
    lxc-devel lxc-templates libcgroup-devel ipython

    ```

1.  在此阶段，让我们为容器创建一个 Linux 桥接，以便后续连接：

    ```
    [root@centos ~]# brctl addbr virbr0

    ```

    对于本章中的示例，我们将使用一个单独的 Python 虚拟环境，以保持项目的依赖项要求独立。我们可以通过使用 `virtualenv` 包来实现。

1.  让我们首先通过 `pip` 安装它：

    ```
    root@ubuntu:~# pip install virtualenv

    ```

1.  接下来，让我们为项目创建一个工作目录并激活虚拟环境：

    ```
    root@ubuntu:~# mkdirlxc_python
    root@ubuntu:~# virtualenv lxc_python
    New python executable in /root/lxc_python/bin/python
    Installing setuptools, pip, wheel...done.
    root@ubuntu:~# 
    root@ubuntu:~# source lxc_python/bin/activate
    (lxc_python) root@ubuntu:~# cd lxc_python
    (lxc_python) root@ubuntu:~/lxc_python#

    ```

1.  激活虚拟环境后，让我们安装 Python LXC API 绑定包，并使用 `pip` 列出开发环境中的内容：

    ```
    (lxc_python) root@ubuntu:~/lxc_python# pip install lxc-python2
    (lxc_python) root@ubuntu:~/lxc_python# pip freeze
    lxc-python2==0.1
    (lxc_python) root@ubuntu:~/lxc_python#

    ```

至此，我们已经具备了所有必需的包、库和工具，可以使用 Python 创建和使用 LXC 容器。让我们写点 Python 代码，享受乐趣吧！

## 用 Python 构建我们的第一个容器

让我们启动 `ipython` 工具并导入之前安装的 LXC 库：

```
(lxc_python) root@ubuntu:~/lxc_python# ipython
In [1]: import lxc

```

接下来，我们需要使用 `Container` 类并指定一个名称来创建 `container` 对象：

```
In [2]: container = def build():Container("python_container")
In [3]: type(container)
Out[3]: lxc.Container

```

现在我们已经有了 `container` 对象，可以使用 `create` 方法来构建我们的第一个容器。

`container.create()` 方法的定义以及每个参数的解释如下：

**定义**：`container.create(self, template=None, flags=0, args=())`，为容器创建一个新的 `rootfs`。以下是各个参数的说明：

+   `template`：此参数必须是一个有效的模板名称，才能被传递。

+   `flags`：这是可选的。它是一个整数，表示要传递的可选创建标志。

+   `args`：这是可选的。它是一个元组，包含要传递给模板的参数。也可以提供为字典形式。

创建一个 Ubuntu 容器就像运行以下代码一样简单：

```
In [4]: container.create("ubuntu")

Checking cache download in /var/cache/lxc/trusty/rootfs-amd64 ... 
Copy /var/cache/lxc/trusty/rootfs-amd64 to /var/lib/lxc/python_container/rootfs ...
Copying rootfs to /var/lib/lxc/python_container/rootfs ...
Generating locales...
en_US.UTF-8... up-to-date
Generation complete.
Creating SSH2 RSA key; this may take some time ...
Creating SSH2 DSA key; this may take some time ...
Creating SSH2 ECDSA key; this may take some time ...
Creating SSH2 ED25519 key; this may take some time ...
update-rc.d: warning: default stop runlevel arguments (0 1 6) do not match ssh Default-Stop values (none)
invoke-rc.d: policy-rc.d denied execution of start.
Current default time zone: 'Etc/UTC'
Local time is now: Tue Sep 20 16:30:31 UTC 2016.
Universal Time is now: Tue Sep 20 16:30:31 UTC 2016.
##
# The default user is 'ubuntu' with password 'ubuntu'!
# Use the 'sudo' command to run tasks as root in the container.
##
Out[4]: True

```

输出 `True` 表示操作成功地定义了容器。

## 使用 Python 收集容器信息

现在我们已经构建了第一个 LXC 容器，让我们来检查它的一些属性。

首先，检查一下容器的名称：

```
In [5]: container.name
Out[5]: u'python_container'

```

还可以检查其状态：

```
In [6]: container.state
Out[6]: u'STOPPED'

```

列出当前主机操作系统上所有的容器：

```
In [7]: lxc.list_containers()
Out[7]: (u'python_container')

```

上面的 `containers()` 方法返回一个包含容器名称的元组。在此例中，就是我们刚刚构建的唯一一个容器。

默认情况下，当我们使用用户空间工具如 `lxc-create` 构建 LXC 容器时，根文件系统和配置文件位于 `/var/lib/lxc/containername/`。让我们通过调用 `get_config_path()` 和 `get_config_item()` 方法来查看我们构建的容器的根文件系统位置：

```
In [8]: container.get_config_path()
Out[8]: u'/var/lib/lxc'
In [9]: container.get_config_item('lxc.rootfs')
Out[9]: u'/var/lib/lxc/python_container/rootfs'

```

从 `get_config_path()` 方法的输出中，我们可以观察到默认的 LXC 配置位置与使用 `lxc-create` 命令构建容器时的配置位置相同。

在前面的代码示例中，我们还将 `lxc.rootfs` 配置选项传递给 `get_config_item()` 方法，以获取根文件系统的位置，这也与默认设置一致，假如使用命令行工具的话。

我们可以传递不同的配置参数给 `get_config_item()` 方法，以获取容器的当前设置。让我们查询一下 `memory.limit_in_bytes` 选项：

```
In [10]: 
container.get_config_item('lxc.cgroup.memory.limit_in_bytes')
Out[10]: u''

```

### 注意

要列出我们创建的 `container` 对象上所有可用的方法和变量，在 `ipython` 中，输入 `container` 然后按一次 ***Tab*** 键。要获取关于方法、函数或变量的更多信息，可以输入其名称后加上问号，例如 `container.get_ips?`。

你可以进一步尝试，打开容器的配置文件，如前面输出所示，并将其作为参数传递给 `get_config_item()` 方法。

要获取容器的 IP 配置信息，我们可以调用 `get_ips()` 方法，无需任何参数，如下所示：

```
In [11]: container.get_ips()
Out[11]: ()

```

由于容器没有运行，并且没有应用内存限制，因此输出分别是空字符串和空元组。与停止的容器一起工作不是特别有趣；让我们探索一下如何在 Python 中操作一个正在运行的容器。

## 启动容器、应用更改并使用 Python 列出配置选项

让我们通过打印 `container` 对象上运行布尔值的值来检查容器是否正在运行：

```
In [12]: container.running
Out[12]: False

```

要启动容器，我们可以使用 `start()` 方法。该方法的文档字符串如下所示：

```
start(useinit = False, daemonize=True, close_fds=False, cmd = (,)) ->boolean

```

启动容器并在成功时返回 `True`。当设置 `useinit` 时，LXC 会使用 `lxc-init` 启动容器。容器可以通过 `daemonize=False` 在前台启动。所有 `fds` 也可以通过传递 `close_fds=True` 来关闭。看起来很简单。让我们通过守护进程方式启动容器，并且不使用 `lxc-init` 管理器，而是使用 Python 解释器：

```
In [13]: container.start(useinit = False, daemonize = True)
Out[13]: True

```

和之前一样，`True` 的输出表示操作成功执行。让我们使用 `wait()` 方法来等待容器达到 `RUNNING` 状态，或者在 5 秒内超时：

```
In [14]: container.wait("RUNNING", 5)
Out[14]: True

```

输出表明容器现在正在运行。让我们通过打印运行状态和状态变量的值来再次确认：

```
In [15]: container.running
Out[15]: True
In [16]: container.state
Out[16]: u'RUNNING'

```

在一个独立的终端中，让我们使用 LXC 用户空间工具来检查我们用 Python 库构建的容器：

```
root@ubuntu:~# lxc-ls -f
NAME             STATE   AUTOSTART GROUPS IPV4      IPV6
python_container RUNNING 0         -      10.0.3.29 -
root@ubuntu:~#

```

`lxc-ls` 命令的输出确认了 `container.state` 变量的返回值。

让我们在 Python shell 中获取容器的 PID：

```
In [17]: container.init_pid
Out[17]: 4688L

```

此时的 PID 是 `4688`；我们可以通过执行以下命令来确认它是否与当前在主机系统上运行的进程匹配：

```
root@ubuntu:~# psaxfw
...
4683 ?Ss     0:00 /usr/bin/python /usr/bin/ipython
4688 ?Ss     0:00  \_ /sbin/init
5405 ?        S      0:00      \_ upstart-socket-bridge --daemon
6224 ?        S      0:00      \_ upstart-udev-bridge --daemon
6235 ?Ss     0:00      \_ /lib/systemd/systemd-udevd --daemon
6278 ?        S      0:00      \_ upstart-file-bridge --daemon
6280 ?Ssl    0:00      \_ rsyslogd
6375 ?Ss     0:00      \_ dhclient -1 -v -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases eth0
6447 pts/3    Ss+    0:00      \_ /sbin/getty -8 38400 tty4
6450 pts/1    Ss+    0:00      \_ /sbin/getty -8 38400 tty2
6451 pts/2    Ss+    0:00      \_ /sbin/getty -8 38400 tty3
6462 ?Ss     0:00      \_ /usr/sbin/sshd -D
6468 ?Ss     0:00      \_ cron
6498 pts/4    Ss+    0:00      \_ /sbin/getty -8 38400 console
6501 pts/0    Ss+    0:00      \_ /sbin/getty -8 38400 tty1
root@ubuntu:~#

```

这里没有什么惊讶的。注意，启动容器 `init` 系统的主进程是 `python` 而不是 `lxc-init`，因为我们在 `start()` 方法中传递的就是 Python。

现在我们的容器已经启动，我们可以从中获取更多信息。让我们从获取其 IP 地址开始：

```
In [18]: container.get_ips()
Out[18]: (u'10.0.3.29',)

```

结果是一个元组，包含容器所有接口的 IP 地址，在本例中只有一个 IP 地址。

我们可以像在第三章，*使用本地和 Libvirt 工具的命令行操作*中看到的那样，通过调用 `attach_wait()` 方法，使用 `lxc-attach` 命令以编程方式附加到容器并运行命令，代码如下：

```
In [19]: container.attach_wait(lxc.attach_run_command, ["ifconfig", "eth0"])
eth0      Link encap:EthernetHWaddr 00:16:3e:ea:1c:38
inet addr:10.0.3.29  Bcast:10.0.3.255  Mask:255.255.255.0
inet6addr: fe80::216:3eff:feea:1c38/64 Scope:Link
 UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
 RX packets:53 errors:0 dropped:0 overruns:0 frame:0
 TX packets:52 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
 RX bytes:5072 (5.0 KB)  TX bytes:4892 (4.8 KB)
Out[19]: 0L

```

`attach_wait()` 方法将一个函数作为参数，在前面的示例中是内置的 `lxc.attach_run_command`，但它也可以是你编写的任何其他 Python 函数。我们还指定了一个包含我们要执行的命令及其参数的列表。

我们还可以指定命令应在哪个命名空间上下文中运行。例如，要列出容器挂载命名空间中所有的文件（该命名空间由 `CLONE_NEWNS` 标志指定），我们可以传递 `namespaces` 参数：

```
In [20]: container.attach_wait(lxc.attach_run_command, ["ls", "-la"], namespaces=(lxc.CLONE_NEWNS))
total 68
drwxr-xr-x  21 root root 4096 Sep 20 18:51 .
drwxr-xr-x  21 root root 4096 Sep 20 18:51 ..
drwxr-xr-x   2 root root 4096 Sep 14 15:20 bin
drwxr-xr-x   2 root root 4096 Apr 10  2014 boot
drwxr-xr-x   7 root root 1140 Sep 20 18:51 dev
drwxr-xr-x  65 root root 4096 Sep 20 18:51 etc
drwxr-xr-x   3 root root 4096 Sep 20 16:30 home
drwxr-xr-x  12 root root 4096 Sep 14 15:19 lib
drwxr-xr-x   2 root root 4096 Sep 14 15:19 lib64
drwxr-xr-x   2 root root 4096 Sep 14 15:18 media
drwxr-xr-x   2 root root 4096 Apr 10  2014 mnt
drwxr-xr-x   2 root root 4096 Sep 14 15:18 opt
dr-xr-xr-x 143 root root    0 Sep 20 18:51 proc
drwx------   2 root root 4096 Sep 14 15:18 root
drwxr-xr-x  10 root root  420 Sep 20 18:51 run
drwxr-xr-x   2 root root 4096 Sep 14 15:20 sbin
drwxr-xr-x   2 root root 4096 Sep 14 15:18 srv
dr-xr-xr-x  13 root root    0 Sep 20 18:51 sys
drwxrwxrwt   2 root root 4096 Sep 20 19:17 tmp
drwxr-xr-x  10 root root 4096 Sep 14 15:18 usr
drwxr-xr-x  11 root root 4096 Sep 14 15:18 var
Out[20]: 0L

```

我们可以通过指定多个 `namespaces` 标志来运行命令。在下一个示例中，我们通过明确指定挂载和进程命名空间（分别使用 `CLONE_NEWNS` 和 `CLONE_NEWPID` 标志），列出容器中的所有进程：

```
In [21]: container.attach_wait(lxc.attach_run_command, ["ps", "axfw"], namespaces=(lxc.CLONE_NEWNS + lxc.CLONE_NEWPID))
PID TTY      STAT   TIME COMMAND
1751 pts/0    R+     0:00psaxfw
1 ?Ss     0:00 /sbin/init
670 ?        S      0:00 upstart-socket-bridge --daemon
1487 ?        S      0:00 upstart-udev-bridge --daemon
1498 ?Ss     0:00 /lib/systemd/systemd-udevd --daemon
1541 ?        S      0:00 upstart-file-bridge --daemon
1543 ?Ssl    0:00 rsyslogd
1582 ?Ss     0:00 dhclient -1 -v -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases eth0
 1654 lxc/tty4 Ss+    0:00 /sbin/getty -8 38400 tty4
 1657 lxc/tty2 Ss+    0:00 /sbin/getty -8 38400 tty2
 1658 lxc/tty3 Ss+    0:00 /sbin/getty -8 38400 tty3
1669 ?Ss     0:00 /usr/sbin/sshd -D
1675 ?Ss     0:00 cron
 1705 lxc/console Ss+   0:00 /sbin/getty -8 38400 console
 1708 lxc/tty1 Ss+    0:00 /sbin/getty -8 38400 tty1
Out[21]: 0L

```

使用 `set_config_item()` 和 `get_config_item()` 方法，我们可以对运行中的容器应用配置更改并查询它们。为了演示这一点，让我们为容器指定一个内存限制，然后获取新设置的值：

```
In [22]: container.set_config_item("lxc.cgroup.memory.limit_in_bytes", "536870912")
Out[22]: True
In [23]: container.get_config_item('lxc.cgroup.memory.limit_in_bytes')
Out[23]: [u'536870912']

```

上述更改在容器重启时不会持久化；为了使更改永久生效，我们可以使用 `append_config_item()` 和 `save_config()` 方法将它们写入配置文件：

```
In [24]: container.append_config_item("lxc.cgroup.memory.limit_in_bytes", "536870912")
Out[24]: True
In [25]: container.save_config()
Out[25]: True

```

为了验证这一点，`lxc.cgroup.memory.limit_in_bytes` 参数已经保存在配置文件中；让我们检查它：

```
root@ubuntu:~# cat /var/lib/lxc/python_container/config
lxc.include = /usr/share/lxc/config/ubuntu.common.conf
# Container specific configuration
lxc.rootfs = /var/lib/lxc/python_container/rootfs
lxc.rootfs.backend = dir
lxc.utsname = python_container
lxc.arch = amd64
# Network configuration
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:ea:1c:38
lxc.cgroup.memory.limit_in_bytes = 536870912
root@ubuntu:~#

```

### 注意

配置文件中的最后一行是我们用两个 Python 调用追加的那一行。

除了 `set_config_item()` 方法外，Python 绑定还提供了 `set_cgroup_item()` 和 `get_cgroup_item()` 方法，用于专门操作 cgroup 参数。让我们使用这两个方法设置并获取相同的 `memory.limit_in_bytes` 选项：

```
In [26]: container.get_cgroup_item("memory.limit_in_bytes")
Out[26]: u'536870912'
In [27]: container.set_cgroup_item("memory.limit_in_bytes", "268435456")
Out[27]: True
In [28]: container.get_cgroup_item("memory.limit_in_bytes")
Out[28]: u'268435456'

```

## 使用 Python 更改容器状态

在第三章，*使用原生和 Libvirt 工具进行命令行操作*中，我们看到了如何使用 `lxc-freeze` 和 `lxc-unfreeze` 命令冻结和解冻 LXC 容器，以保存其状态。我们可以使用 `freeze()` 和 `unfreeze()` 方法做到这一点。要冻结容器，执行以下命令：

```
In [29]: container.freeze()
Out[29]: u'FROZEN'

```

按如下方式检查状态：

```
In [30]: container.state
Out[30]: u'FROZEN'

```

我们也可以检查 cgroup 文件，以确认更改已发生：

```
root@ubuntu:~# cat /sys/fs/cgroup/freezer/lxc/python_container/freezer.state
FROZEN
root@ubuntu:~#

```

要解冻容器并检查新状态，请调用 `unfreeze()` 方法：

```
In [31]: container.unfreeze()
Out[31]: True
In [32]: container.state
Out[32]: u'RUNNING'
root@ubuntu:~# cat /sys/fs/cgroup/freezer/lxc/python_container/freezer.state
THAWED
root@ubuntu:~#

```

## 使用 Python 停止容器

Python 绑定提供了一个方便的方式来停止容器，使用 `stop()` 方法。让我们停止容器并检查其状态：

```
In [33]: container.stop()
Out[33]: True
In [34]: container.state
Out[34]: u'STOPPED'

```

最后，列出主机上的所有容器：

```
root@ubuntu:~# lxc-ls -f
NAME             STATE   AUTOSTART GROUPS IPV4 IPV6
python_container STOPPED 0         -      -    -
root@ubuntu:~#

```

## 使用 Python 克隆容器

容器处于 `STOPPED` 状态时，让我们运行 `clone()` 方法并创建一个副本：

```
In [35]: cloned_container = container.clone("cloned_container")
In [35]: True

```

使用 `list_containers()` 方法列出 `lxc` 对象上的可用容器，我们得到一个元组：

```
In [36]: lxc.list_containers()
Out[36]: (u'cloned_container', u'python_container')

```

要在主机操作系统上确认，请执行以下操作：

```
root@ubuntu:~# lxc-ls -f
NAME             STATE   AUTOSTART GROUPS IPV4 IPV6
cloned_container STOPPED 0         -      -    -
python_container STOPPED 0         -      -    -
root@ubuntu:~#

```

要找到克隆容器的根文件系统位置，我们可以在新的 `container` 对象上调用 `get_config_item()` 方法：

```
In [37]: cloned_container.get_config_item('lxc.rootfs')
Out[37]: u'/var/lib/lxc/cloned_container/rootfs'

```

现在在默认的容器路径下存在两个目录：

```
root@ubuntu:~# ls -la /var/lib/lxc
total 20
drwx------  5 root root 4096 Sep 20 19:51 .
drwxr-xr-x 47 root root 4096 Sep 16 13:40 ..
drwxrwx---  3 root root 4096 Sep 20 19:51 cloned_container
drwxrwx---  3 root root 4096 Sep 20 16:30 python_container
root@ubuntu:~#

```

最后，让我们启动克隆的容器并确保它正在运行：

```
In [38]: cloned_container.start()
Out[38]: True
In [39]: cloned_container.state
Out[39]: u'RUNNING'

```

## 使用 Python 销毁容器并清理虚拟环境

在我们能够在 Python 中移除或销毁容器之前，像命令行工具一样，我们需要先停止它们：

```
In [40]: cloned_container.stop()
Out[40]: True
In [41]: container.stop()
Out[41]: True

```

在 `container` 对象上调用 `destroy()` 方法，以删除根文件系统并释放它所使用的所有资源：

```
In [42]: cloned_container.destroy()
Out[42]: True
In [43]: container.destroy()
Out[43]: True

```

通过 `list_containers()` 方法列出容器，此时返回一个空元组：

```
In [44]: lxc.list_containers()
Out[44]: ()

```

最后，让我们停用之前创建的 Python 虚拟环境——注意，文件仍然会保留在磁盘上：

```
(lxc_python) root@ubuntu:~/lxc_python# deactivate
root@ubuntu:~/lxc_python# cd ..
root@ubuntu:~#

```

# Libvirt Python 绑定

在 第三章，*使用原生和 Libvirt 工具的命令行操作*，我们探讨了通过使用 libvirt 用户空间工具处理 LXC 的替代方法。Libvirt 提供了 Python 绑定，我们可以用它来编写应用程序，主要的好处是与其他虚拟化技术的一致性。使用一个通用的库来编写 KVM、XEN 和 LXC 的 Python 应用程序非常方便。

在本节中，我们将探讨 libvirt 库提供的一些 Python 方法，用于创建和控制 LXC 容器。

## 安装 libvirt Python 开发包

让我们首先安装所需的包并启动服务。

在 Ubuntu 上，运行以下命令：

```
root@ubuntu:~# apt-get install python-libvirt debootstrap
root@ubuntu:~# service libvirt-bin start

```

在 CentOS 上，库和服务的名称不同：

```
[root@centos ~]# yum install libvirt libvirt-python debootstrap
[root@centos ~]# service libvirtd start

```

由于 libvirt 不提供模板来使用，我们需要创建自己的根文件系统：

```
root@ubuntu:~# debootstrap --arch=amd64 --include="openssh-server vim" stable ~/container http://httpredir.debian.org/debian/ 
...
root@ubuntu:~#

```

激活 Python 虚拟环境并启动解释器：

```
root@ubuntu:~# source lxc_python/bin/activate
(lxc_python) root@ubuntu:~# ipython
In [1]:

```

## 使用 libvirt Python 构建 LXC 容器

现在是导入库并调用 `open()` 方法以创建与 LXC 驱动程序的连接的时候了。我们传递给 `open()` 方法的参数应该是熟悉的——我们在 第三章，*使用原生和 Libvirt 工具的命令行操作*，中使用过它，在导出 `LIBVIRT_DEFAULT_URI` 环境变量时，告诉 libvirt LXC 将是默认的虚拟化驱动程序：

```
In [1]: import libvirt
In [2]: lxc_conn = libvirt.open('lxc:///')

```

在指定默认虚拟化驱动程序和 URI 后，我们可以使用接下来的两个方法来返回我们设置的驱动程序的名称和路径：

```
In [3]: lxc_conn.getType()
Out[3]: 'LXC'
In [4]: lxc_conn.getURI()
Out[4]: 'lxc:///'

```

### 注

要获取我们之前创建的 `lxc_conn` 对象上的可用方法和变量列表，请输入 `lxc_conn`，然后按 ***Tab*** 键。要获取有关方法、函数或变量的更多信息，请键入其名称后加上问号，例如，`lxc_conn.getURI?`。

我们可以使用 `getInfo()` 方法提取主机节点的硬件信息：

```
In [5]: lxc_conn.getInfo()
Out[5]: ['x86_64', 1996L, 2, 3000, 1, 2, 1, 1]

```

结果是一个包含以下值的列表：

| **成员** | **描述** |
| --- | --- |
| `list[0]` | 指示 CPU 型号的字符串 |
| `list[1]` | 以兆字节为单位的内存大小 |
| `list[2]` | 活跃 CPU 的数量 |
| `list[3]` | 预期的 CPU 频率（单位：MHz） |
| `list[4]` | NUMA 节点的数量，`1` 表示统一内存访问 |
| `list[5]` | 每个节点的 CPU 插槽数 |
| `list[6]` | 每个插槽的核心数 |
| `list[7]` | 每个核心的线程数 |

要构建容器，我们需要先在 XML 文件中定义它。让我们使用以下示例并将其分配给 `domain_xml` 字符串变量：

```
In [6]: domain_xml = '''
<domain type='lxc'>
 <name>libvirt_python</name>
 <memory unit='KiB'>524288</memory>
 <currentMemory unit='KiB'>524288</currentMemory>
 <vcpu placement='static'>1</vcpu>
 <os>
 <type arch='x86_64'>exe</type>
 <init>/sbin/init</init>
 </os>
 <clock offset='utc'/>
 <on_poweroff>destroy</on_poweroff>
 <on_reboot>restart</on_reboot>
 <on_crash>destroy</on_crash>
 <devices>
 <emulator>/usr/lib/libvirt/libvirt_lxc</emulator>
 <filesystem type='mount' accessmode='passthrough'>
 <sourcedir='/root/container/'/>
 <targetdir='/'/>
 </filesystem>
 <interface type='bridge'>
 <mac address='00:17:3e:9f:33:f7'/>
 <source bridge='lxcbr0'/>
 <link state='up'/>
 </interface>
 <console type='pty' />
 </devices>
</domain>
'''

```

使用之前分配给变量的 XML 配置，我们可以使用 `defineXML()` 方法来定义容器。此方法将 XML 定义作为参数并定义容器，但不会启动容器：

```
In [7]: container = lxc_conn.defineXML(domain_xml)

```

让我们验证容器是否已成功在主机上定义：

```
root@ubuntu:~# virsh --connect lxc:/// list --all
Id    Name                           State
----------------------------------------------------
-     libvirt_python                 shut off
root@ubuntu:~#

```

我们可以使用`listDefinedDomains()`方法列出所有已定义但未运行的域，它返回一个列表：

```
In [8]: lxc_conn.listDefinedDomains()
Out[8]: ['libvirt_python']

```

## 使用 libvirt Python 启动容器并运行基本操作

要启动之前定义的容器，我们需要调用`create()`方法：

```
In [9]: container.create()
Out[9]: 07

```

为了验证容器是否在主机上运行，在调用`create()`方法后，我们将执行以下操作：

```
root@ubuntu:~# virsh --connect lxc:/// list --all
Id    Name                           State
----------------------------------------------------
23749 libvirt_python                 running
root@ubuntu:~#

```

有许多方法可以获取容器的信息。我们可以通过在`container`对象上调用`XMLDesc()`方法来获取 XML 定义：

```
In [10]: container.XMLDesc()
Out[10]: "<domain type='lxc' id='25535'>\n  <name>libvirt_python</name>\n  <uuid>6a46bd23-f0df-461b-85e7-19fd36be90df</uuid>\n  <memory unit='KiB'>524288</memory>\n  <currentMemory unit='KiB'>524288</currentMemory>\n  <vcpu placement='static'>1</vcpu>\n  <resource>\n    <partition>/machine</partition>\n  </resource>\n  <os>\n    <type arch='x86_64'>exe</type>\n    <init>/sbin/init</init>\n  </os>\n  <clock offset='utc'/>\n  <on_poweroff>destroy</on_poweroff>\n  <on_reboot>restart</on_reboot>\n  <on_crash>destroy</on_crash>\n  <devices>\n    <emulator>/usr/lib/libvirt/libvirt_lxc</emulator>\n    <filesystem type='mount' accessmode='passthrough'>\n      <source dir='/root/container/'/>\n      <target dir='/'/>\n    </filesystem>\n    <interface type='bridge'>\n      <mac address='00:17:3e:9f:33:f7'/>\n      <source bridge='lxcbr0'/>\n      <target dev='vnet0'/>\n      <link state='up'/>\n    </interface>\n    <console type='pty' tty='/dev/pts/1'>\n      <source path='/dev/pts/1'/>\n      <target type='lxc' port='0'/>\n      <alias name='console0'/>\n    </console>\n  </devices>\n  <seclabel type='none'/>\n</domain>\n"

```

让我们通过调用`isAlive()`函数来验证容器是否正在运行，该函数返回一个布尔值：

```
In [11]: lxc_conn.isAlive()
Out[11]: 1

```

我们可以获取容器 ID，该 ID 应与通过运行前述`virsh`命令返回的 ID 匹配：

```
In [12]: lxc_conn.listDomainsID()
Out[12]: [23749]

```

毫无惊讶，ID 是相同的。

下一个代码片段遍历已定义容器的列表，并通过调用`listAllDomains()`方法返回域对象列表，从中打印出它们的名称：

```
In [13]: domains = lxc_conn.listAllDomains(0)
In [13]: for domain in domains:
....:     print('  '+domain.name())
....:
libvirt_python

```

API 提供了两种查找容器的方法，通过名称和 ID，并将其赋值给对象变量：

```
In [14]: container = lxc_conn.lookupByName("libvirt_python")
In [15]: container = lxc_conn.lookupByID(23749)

```

这在我们想要操作已存在的容器时非常有用。现在可以像平常一样使用`container`对象，通过调用其方法。

## 使用 libvirt Python 收集容器信息

让我们收集容器的内存信息。`maxMemory()`方法返回容器配置的最大内存：

```
In [16]: container.maxMemory()
Out[16]: 524288L

```

收集内存统计信息是通过`memoryStats()`方法完成的，该方法返回一个字典对象：

```
In [17]: container.memoryStats()
Out[17]: {'actual': 524288L, 'rss': 1388L, 'swap_in': 1388L}

```

当我们在 XML 文件中定义容器时，我们指定了域的操作系统类型为`exe`，这意味着容器将执行指定的二进制文件。要在运行中的容器上获取该信息，可以调用`OSType()`方法：

```
In [18]: container.OSType()
Out[18]: 'exe'

```

最后，为了获取更多关于容器的信息，我们可以调用`info()`函数：

```
In [19]: container.info()
Out[19]: [1, 524288L, 1352L, 1, 8080449759L]

```

结果是一个包含以下值的列表：

| **成员** | **描述** |
| --- | --- |
| `list[0]` | 表示容器状态的字符串 |
| `list[1]` | 容器的最大内存 |
| `list[2]` | 当前内存利用率 |
| `list[3]` | CPU 数量 |
| `list[4]` | CPU 时间 |

容器启动后，让我们看看接下来如何停止它并清理环境。

## 使用 libvirt Python 停止和删除 LXC 容器

在销毁容器之前，让我们验证一下它的状态和名称。

```
In [20]: container.isActive()
Out[20]: 1
In [21]: container.name()
Out[21]: 'libvirt_python'

```

要停止容器，调用`container`对象上的`destroy()`方法：

```
In [22]: container.destroy()
Out[22]: 0

```

在删除容器之前，让我们验证容器在主机上没有运行：

```
root@ubuntu:~# virsh --connect lxc:/// list --all
Id    Name                           State
----------------------------------------------------
-     libvirt_python                 shut off
root@ubuntu:~#

```

要删除容器，我们调用`undefine()`方法：

```
In [23]: container.undefine()
Out[23]: 0
root@ubuntu:~# virsh --connect lxc:/// list --all
Id    Name                           State
----------------------------------------------------
root@ubuntu:~#

```

### 注意

需要注意的是，并非所有的方法、函数和变量都可以在 libvirt LXC 驱动程序中使用，即使它们在导入 libvirt 库后可以在 ipython 解释器中列出。这是由于 libvirt 对多个虚拟化管理程序（如 KVM 和 XEN）的支持。在探索 API 调用时请记住这一点。

# Vagrant 和 LXC

Vagrant 是一个优秀的开源项目，通过使用插件提供了构建隔离开发环境的方式，支持各种虚拟化技术，如 KVM 和 LXC。

在本节中，我们将简要介绍如何使用 LXC 进行隔离，设置 Vagrant 开发环境。

让我们从在 Ubuntu 上下载并安装 Vagrant 开始：

```
root@ubuntu:~# cd /usr/src/
root@ubuntu:/usr/src# wget https://releases.hashicorp.com/vagrant/1.8.5/vagrant_1.8.5_x86_64.deb
--2016-09-26 21:11:56-- https://releases.hashicorp.com/vagrant/1.8.5/vagrant_1.8.5_x86_64.deb
Resolving releases.hashicorp.com (releases.hashicorp.com)... 151.101.44.69
Connecting to releases.hashicorp.com (releases.hashicorp.com)|151.101.44.69|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 76325224 (73M) [application/x-debian-package]
Saving to: 'vagrant_1.8.5_x86_64.deb'
100%[============================================================
================================>] 76,325,224   104MB/s   in 0.7s
2016-09-26 21:11:57 (104 MB/s) - 'vagrant_1.8.5_x86_64.deb' saved [76325224/76325224]
root@ubuntu:/usr/src#

```

安装这个软件包非常简单：

```
root@ubuntu:/usr/src# dpkg --install vagrant_1.8.5_x86_64.deb
Selecting previously unselected package vagrant.
(Reading database ... 60326 files and directories currently installed.)
Preparing to unpack vagrant_1.8.5_x86_64.deb ...
Unpacking vagrant (1:1.8.5) ...
Setting up vagrant (1:1.8.5) ...
root@ubuntu:/usr/src#

```

在 CentOS 上，步骤如下：

1.  执行以下命令以下载并安装 Vagrant：

    ```
     [root@centos ~]# cd /usr/src/
     [root@centossrc]# wget   
          https://releases.hashicorp.com/vagrant/1.8.5/
          vagrant_1.8.5_x86_64.rpm
    --2016-09-26 21:16:06-- 
          https://releases.hashicorp.com/vagrant/1.8.5/
          vagrant_1.8.5_x86_64.rpm
     Resolving releases.hashicorp.com (releases.hashicorp.com)...     
          151.101.44.69
     Connecting to releases.hashicorp.com       
          (releases.hashicorp.com)|151.101.44.69|:443... connected.
     HTTP request sent, awaiting response... 200 OK
     Length: 75955433 (72M) []
     Saving to: 'vagrant_1.8.5_x86_64.rpm'
     100%   
          [========================================================
          =====================================>] 
          75,955,433  104MB/s   in 0.7s
     2016-09-26 21:16:07 (104 MB/s) - 'vagrant_1.8.5_x86_64.rpm' saved 
          [75955433/75955433]
     [root@centossrc]# rpm --install vagrant_1.8.5_x86_64.rpm

    ```

1.  接下来，如果 LXC 尚未附加到新的桥接设备，创建一个新的桥接设备：

    ```
     [root@centossrc]# brctladdbr lxcbr0

    ```

1.  现在是时候安装 LXC 插件了：

    ```
     root@ubuntu:/usr/src# vagrant plugin install vagrant-lxc
     Installing the 'vagrant-lxc' plugin. This can take a few minutes...
     Installed the plugin 'vagrant-lxc (1.2.1)'!
     root@ubuntu:/usr/src#

    ```

1.  如果一切顺利，请列出已安装的 Vagrant 插件：

    ```
     root@ubuntu:/usr/src# vagrant plugin list
     vagrant-lxc (1.2.1)
     vagrant-share (1.1.5, system)
     root@ubuntu:/usr/src#

    ```

1.  安装了 LXC 插件后，创建一个新的项目目录：

    ```
     root@ubuntu:/usr/src# mkdirmy_project
     root@ubuntu:/usr/src# cd my_project/

    ```

1.  接下来，通过指定我们将使用的盒子类型或虚拟机镜像，初始化一个新的 Vagrant 环境。在以下示例中，我们将使用来自 `fgremh` 仓库的 Ubuntu Precise LXC 镜像：

    ```
    root@ubuntu:/usr/src/my_project# vagrant init fgrehm/precise64-lxc
    A `Vagrantfile` has been placed in this directory. You are now
    ready to `vagrant up` your first virtual environment! Please read
    the comments in the Vagrantfile as well as documentation on
    `vagrantup.com` for more information on using Vagrant.
    root@ubuntu:/usr/src/my_project#

    ```

    如输出所示，项目目录中已创建了一个新的 `Vagrantfile`：

    ```
    root@ubuntu:/usr/src/my_project# ls -alh
    total 12K
    drwxr-xr-x 2 root root 4.0K Sep 26 21:45 .
    drwxr-xr-x 5 root root 4.0K Sep 26 21:18 ..
    -rw-r--r-- 1 root root 3.0K Sep 26 21:45 Vagrantfile
    root@ubuntu:/usr/src/my_project#

    ```

    ### 注意

    要查看 Vagrant 盒子列表，请访问：[`atlas.hashicorp.com/boxes/search`](https://atlas.hashicorp.com/boxes/search)。

    让我们看看 `Vagrantfile`：

    ```
     root@ubuntu:/usr/src/my_project# cat Vagrantfile | grep -v "#" 
          | sed '/^$/d'
    Vagrant.configure("2") do |config|
    config.vm.box = "fgrehm/precise64-lxc"
    end
    root@ubuntu:/usr/src/my_project#

    ```

1.  配置非常简洁，仅指定 Vagrant 虚拟机将使用的镜像。我们通过明确指定提供程序来启动容器：

    ```
    root@ubuntu:/usr/src/my_project# vagrant up --provider=lxc
    Bringing machine 'default' up with 'lxc' provider...
    ==>default: Importing base box 'fgrehm/precise64-lxc'...
    ==>default: Checking if box 'fgrehm/precise64-lxc' is up to date...
    ==>default: Setting up mount entries for shared folders...
    default: /vagrant => /usr/src/my_project
    ==>default: Starting container...
    ==>default: Waiting for machine to boot. This may take a few 
          minutes...
    default: SSH address: 10.0.3.181:22
    default: SSH username: vagrant
    default: SSH auth method: private key
    default:
    default: Vagrant insecure key detected. Vagrant will automatically 
          replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH 
          key...
    ==>default: Machine booted and ready!
    root@ubuntu:/usr/src/my_project#

    ```

1.  要验证是否有正在运行的 LXC 容器，请在命令行中执行以下命令：

    ```
    root@ubuntu:/usr/src/my_project# lxc-ls -f
    NAME                      STATE   AUTOSTART GROUPS IPV4      IPV6
    my_project_default_1474926399170_41712 RUNNING 0         -      
          10.0.3.80 -
    root@ubuntu:/usr/src/my_project#

    ```

1.  让我们检查 Vagrant 虚拟机的状态：

    ```
    root@ubuntu:/usr/src/my_project# vagrant status
    Current machine states:
    default                   running (lxc)
    ...
    root@ubuntu:/usr/src/my_project#

    ```

1.  要连接到 LXC 容器，请运行以下命令：

    ```
    root@ubuntu:/usr/src/my_project# vagrant ssh
    Welcome to Ubuntu 12.04.5 LTS (GNU/Linux 3.13.0-91-generic x86_64)
     * Documentation:  https://help.ubuntu.com/
    vagrant@vagrant-base-precise-amd64:~$ ps ax
     PID TTY      STAT   TIME COMMAND
    1 ?Ss     0:00 /sbin/init
    194 ?        S      0:00 upstart-socket-bridge --daemon
    2630 ?        S      0:00 upstart-udev-bridge --daemon
    2653 ?Ss     0:00 /sbin/udevd --daemon
    2672 ?Sl     0:00 rsyslogd -c5
    2674 ?Ss     0:00 rpcbind -w
    2697 ?Ss     0:00 rpc.statd -L
    2830 ?Ss     0:00 dhclient3 -e IF_METRIC=100 -pf 
          /var/run/dhclient.eth0.pid -lf 
          /var/lib/dhcp/dhclient.eth0.leases -1 eth0
    2852 ?Ss     0:00 /usr/sbin/sshd -D
    2888 ?Ss     0:00 cron
    3259 ?Ss     0:00 /sbin/getty -8 38400 tty4
    3260 ?Ss     0:00 /sbin/getty -8 38400 tty2
    3261 ?Ss     0:00 /sbin/getty -8 38400 tty3
    3262 ?Ss     0:00 /sbin/getty -8 38400 console
    3263 ?Ss     0:00 /sbin/getty -8 38400 tty1
    3266 ?Ss     0:00 sshd: vagrant [priv]
    3278 ?        S      0:00 sshd: vagrant@pts/9
     3279 pts/9    Ss     0:00 -bash
     3305 pts/9    R+     0:00ps ax
    vagrant@vagrant-base-precise-amd64:~$ exit
    logout
    Connection to 10.0.3.181 closed.
    root@ubuntu:/usr/src/my_project#

    ```

## 配置 Vagrant LXC

`Vagrantfile` 有很好的文档说明，以下是如何通过指定可用于 LXC 容器的内存量来定制 Vagrant 虚拟机的简要示例：

1.  停止正在运行的 Vagrant 虚拟机：

    ```
    root@ubuntu:/usr/src/my_project# vagrant halt
    ==>default: Attempting graceful shutdown of VM...
    root@ubuntu:/usr/src/my_project#

    ```

1.  编辑 `Vagrantfile` 并设置 `cgroup.memory.limit_in_bytes` cgroup 限制。新的配置应该如下所示：

    ```
    root@ubuntu:/usr/src/my_project# vim Vagrantfile
    Vagrant.configure("2") do |config|
    config.vm.box = "fgrehm/trusty64-lxc"
    config.vm.provider :lxc do |lxc|
    lxc.customize 'cgroup.memory.limit_in_bytes', '1024M'
    end
    end
    root@ubuntu:/usr/src/my_project#

    ```

1.  保存文件并重新启动 Vagrant 虚拟机：

    ```
    root@ubuntu:/usr/src/my_project# vagrant up --provider=lxc
    Bringing machine 'default' up with 'lxc' provider...
    ==>default: Setting up mount entries for shared folders...
    default: /vagrant => /usr/src/my_project
    ==>default: Starting container...
    ==>default: Waiting for machine to boot. This may take a few 
          minutes...
    default: SSH address: 10.0.3.181:22
    default: SSH username: vagrant
    default: SSH auth method: private key
    ==>default: Machine booted and ready!
    ==>default: Machine already provisioned. Run `vagrant provision` or 
          use the `--provision`
    ==>default: flag to force provisioning. Provisioners marked to run 
          always will still run.
    root@ubuntu:/usr/src/my_project#

    ```

1.  验证是否应用了 cgroup 限制：

    ```
    root@ubuntu:/usr/src/my_project# cat /sys/fs/cgroup/memory/lxc/ 
          my_project_default_1474926399170_41712/memory.limit_in_bytes
    1073741824
    root@ubuntu:/usr/src/my_project#

    ```

1.  最后，让我们通过删除所有 Vagrant 虚拟机残留文件来清理：

    ```
    root@ubuntu:/usr/src/my_project# vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
    ==>default: Forcing shutdown of container...
    ==>default: Destroying VM and associated drives...
    root@ubuntu:/usr/src/my_project# lxc-ls -f
    NAME                    STATE   AUTOSTART GROUPS IPV4 IPV6
    root@ubuntu:/usr/src/my_project#

    ```

# 将所有内容结合起来 – 使用 Python 构建一个简单的 LXC RESTful API

通过之前实验 Python 的 LXC 绑定所掌握的所有知识，我们可以编写一个简单的 RESTful API，来构建、管理和销毁 LXC 容器。

### 注意

为了保持代码尽可能简单，我们将跳过所有错误和异常处理以及程序中的任何输入验证。

用于构建 API 的最简单 Python Web 框架之一是 Bottle。我们先安装它：

```
root@ubuntu:~# source lxc_python/bin/activate
(lxc_python) root@ubuntu:~# pip install bottle
Collecting bottle
 Downloading bottle-0.12.9.tar.gz (69kB)
 100% |████████████████████████████████| 71kB 7.1MB/s
Building wheels for collected packages: bottle
 Running setup.py bdist_wheel for bottle ... done
 Stored in directory: 
/root/.cache/pip/wheels/6e/87/89/f7ddd6721f4a208d44f2dac02f281b2403a314dd735d2b0e61
Successfully built bottle
Installing collected packages: bottle
Successfully installed bottle-0.12.9
(lxc_python) root@ubuntu:~#

```

在继续之前，请确保安装了 `bottle` 和 `lxc-python2` 库：

```
(lxc_python) root@ubuntu:~# pip freeze
bottle==0.12.9
lxc-python2==0.1
(lxc_python) root@ubuntu:~#

```

打开新创建的 `lxc_api.py` 文件并编写以下代码：

```
import lxc
from bottle import run, request, get, post

@get('/list')
def list():
     container_list = lxc.list_containers()

     return "List of containers: {0}\n".format(container_list)
run(host='localhost', port=8080, debug=True)

```

`run`类提供了`run()`调用，用于启动内置服务器。在我们的示例中，服务器将监听本地主机上的`8080`端口。`get()`装饰器将下方函数中的代码链接到一个 URL 路径。在前面的代码中，`/list`路径与`list()`函数绑定。当然，您已经熟悉了`list_containers()`方法。

要测试这个简单的 API 前端，保存文件并执行程序：

```
(lxc_python) root@ubuntu:~# python lxc_api.py
Bottle v0.12.9 server starting up (using WSGIRefServer())...
Listening on http://localhost:8080/
Hit Ctrl+C to quit. 

```

### 提示

如果你遇到`socket.error: [Errno 98] Address already in use`错误，说明另一个进程已经绑定了`8080`端口。要解决此问题，只需在`run()`方法中更改 Python 应用程序监听的端口。

在一个独立的终端窗口中，执行以下操作：

```
root@ubuntu:~# curl localhost:8080/list
List of containers: ()
root@ubuntu:~#

```

就是这么简单；我们创建了一个 API 调用来列出 LXC 容器！

## 用于构建和配置 LXC 容器的 API 调用

让我们稍微扩展一下功能，添加构建容器的能力。编辑文件并添加以下函数：

```
@post('/build')
def build():
     name = request.headers.get('X-LXC-Name')
     template = str(request.headers.get('X-LXC-Template'))

     container = lxc.Container(name)
     container.create(template)
     return "Building container {0} using the {1} template\n".format(
    name, template)

```

在这种情况下，我们将使用`@post`装饰器和`request`类提供的`headers.get()`方法来获取包含容器和模板名称的自定义头信息。

### 注意

有关 Bottle 框架的完整 API 参考，请参阅[`bottlepy.org/docs/dev/api.html`](http://bottlepy.org/docs/dev/api.html)。

保存更新的文件并重启程序。让我们在第二个终端中测试新的调用：

```
root@ubuntu:~# curl -XPOST --header "X-LXC-Name: api_container" --header "X-LXC-  
Template: ubuntu" localhost:8080/build 
Building container api_container using the ubuntu template
root@ubuntu:~#

```

我们使用了`--header`标志通过`curl`将容器和模板名称作为头信息传递，使用`POST`动词。如果你检查正在运行的应用程序的终端，你可以看到容器构建的日志，以及 HTTP 路由和错误代码：

```
Bottle v0.12.9 server starting up (using WSGIRefServer())...
Listening on http://localhost:8080/
Hit Ctrl-C to quit.
Copying rootfs to /var/lib/lxc/api_container/rootfs ...
Generating locales...
en_US.UTF-8... up-to-date
Generation complete.
Creating SSH2 RSA key; this may take some time ...
Creating SSH2 DSA key; this may take some time ...
Creating SSH2 ECDSA key; this may take some time ...
Creating SSH2 ED25519 key; this may take some time ...
update-rc.d: warning: default stop runlevel arguments (0 1 6) do not match ssh  
Default-Stop values (none)
invoke-rc.d: policy-rc.d denied execution of start.
...
127.0.0.1 - - [06/Oct/2016 21:36:10] "POST /build HTTP/1.1" 200 59

```

让我们使用之前定义的`/list`路由来列出所有容器：

```
root@ubuntu:~# curl localhost:8080/list
List of containers: (u'api_container',)
root@ubuntu:~#

```

太好了！现在我们可以构建和列出容器了。让我们创建一个新路由来启动 LXC。将以下函数添加到`lxc_api.py`文件中：

```
@post('/container/<name>/start')
def container_start(name):
     container = lxc.Container(name)
     container.start(useinit = False, daemonize = True)

     return "Starting container {0}\n".format(name)

```

我们再次使用了`POST`装饰器和动态路由。动态路由由一个名称组成，在我们的示例中为`<name>`，它将保存我们通过`curl`命令传递给路由的字符串值。绑定到`container_start(name)`路由的方法也接受一个同名的变量。保存更改，重启应用程序，并执行以下操作：

```
root@ubuntu:~# curl -XPOST localhost:8080/container/api_container/start
Starting container api_container
root@ubuntu:~#

```

我们在 URL 中传递了`api_container`，并且我们定义的路由能够匹配它，并将其作为变量传递给`container_start`函数。

我们的简单 API 尚未提供获取容器状态的路由，所以让我们确保它确实在运行：

```
root@ubuntu:~# lxc-ls -f
NAME          STATE   AUTOSTART GROUPS IPV4       IPV6
api_container RUNNING 0         -      10.0.3.198 -
root@ubuntu:~#

```

让我们向 API 添加一个`state`调用：

```
@get('/container/<name>/state')
def container_status(name):
     container = lxc.Container(name)
     state = container.state

     return "The state of container {0} is {1}\n".format(name, state)

```

这次我们使用了`@get`装饰器，并在`container`对象上调用了`state()`方法。让我们测试一下新路由：

```
root@ubuntu:~# curl localhost:8080/container/api_container/state
The state of container api_container is RUNNING
root@ubuntu:~#

```

现在我们有了一个正在运行的容器，让我们添加列出其 IP 地址的功能：

```
@get('/container/<name>/ips')
def container_status(name):
     container = lxc.Container(name)
     ip_list = container.get_ips()
     return "Container {0} has the following IP's {1}\n".format(
     name, ip_list)

```

这里没有什么新的需要注意的内容，接下来让我们看看返回的结果：

```
root@ubuntu:~# curl localhost:8080/container/api_container/ips
Container api_container has the following IP's (u'10.0.3.198',)
root@ubuntu:~#

```

我们之前已经在本章中看到如何冻结和解冻容器；现在让我们把这个功能添加到我们的 API 中：

```
@post('/container/<name>/freeze')
def container_start(name):
     container = lxc.Container(name)
     container.freeze()
     return "Freezing container {0}\n".format(name)

@post('/container/<name>/unfreeze')
def container_start(name):
     container = lxc.Container(name)
     container.unfreeze()
     return "Unfreezing container {0}\n".format(name)

```

这里使用 `POST` 方法更为合适，因为我们正在修改容器的状态。现在让我们冻结容器并检查其状态：

```
root@ubuntu:~# curl -XPOST localhost:8080/container/api_container/freeze
Freezing container api_container
root@ubuntu:~# curl localhost:8080/container/api_container/state
The state of container api_container is FROZEN
root@ubuntu:~# lxc-ls -f
NAME          STATE  AUTOSTART GROUPS IPV4       IPV6
api_container FROZEN 0         -      10.0.3.198 -
root@ubuntu:~#

```

最后，让我们用我们刚才创建的新的 API 调用解冻它：

```
root@ubuntu:~# curl -XPOST localhost:8080/container/api_container/unfreeze
Unfreezing container api_container
root@ubuntu:~# curl localhost:8080/container/api_container/state
The state of container api_container is RUNNING
root@ubuntu:~# lxc-ls -f
NAME          STATE   AUTOSTART GROUPS IPV4       IPV6
api_container RUNNING 0         -      10.0.3.198 -
root@ubuntu:~#

```

接下来，作为结论，我们将编写两个新函数来停止和删除容器：

```
@post('/container/<name>/stop')
def container_start(name):
     container = lxc.Container(name)
     container.stop()

     return "Stopping container {0}\n".format(name)

@post('/container/<name>/destroy')
def container_start(name):
     container = lxc.Container(name)
     container.destroy() 

     return "Destroying container {0}\n".format(name)

```

到目前为止，我们使用的所有方法，都已经在本章中进行过测试；如有需要，请随时参考它们的描述。

## 使用 API 调用进行清理

现在是清理的时候了，通过调用 `stop` API 路由：

```
root@ubuntu:~# curl -XPOST localhost:8080/container/api_container/stop
Stopping container api_container 
root@ubuntu:~# curl localhost:8080/container/api_container/state
The state of container api_container is STOPPED
root@ubuntu:~#

```

在执行完之前的所有 API 调用后，控制台中的运行应用程序应显示类似如下内容：

```
127.0.0.1 - - [06/Oct/2016 22:00:08] "GET /container/api_container/state HTTP/1.1" 200 48
127.0.0.1 - - [06/Oct/2016 22:02:45] "GET /container/api_container/ips HTTP/1.1" 200 64
127.0.0.1 - - [06/Oct/2016 22:08:14] "POST /container/api_container/freeze HTTP/1.1" 200 33
127.0.0.1 - - [06/Oct/2016 22:08:19] "GET /container/api_container/state HTTP/1.1" 200 47
127.0.0.1 - - [06/Oct/2016 22:08:30] "POST /container/api_container/unfreeze HTTP/1.1" 200 35
127.0.0.1 - - [06/Oct/2016 22:08:32] "GET /container/api_container/state HTTP/1.1" 200 48
127.0.0.1 - - [06/Oct/2016 22:18:39] "POST /container/api_container/stop HTTP/1.1" 200 33
127.0.0.1 - - [06/Oct/2016 22:18:41] "GET /container/api_container/state HTTP/1.1" 200 48

```

最后，让我们销毁容器：

```
root@ubuntu:~# curl -XPOST localhost:8080/container/api_container/destroy
Destroying container api_container
root@ubuntu:~# lxc-ls -f
root@ubuntu:~#

```

我们可以轻松地添加之前实验过的所有 LXC Python 方法，只需遵循相同的模式 —— 只需记住捕获所有异常并验证输入。

这是整个程序：

```
import lxc
from bottle import run, request, get, post
@post('/build')
def build():
 name = request.headers.get('X-LXC-Name')
 memory = request.headers.get('X-LXC-Memory')
 template = str(request.headers.get('X-LXC-Template'))

 container = lxc.Container(name)
 container.create(template)

 return "Building container {0} using the {1} template\n".format(
    name, template)

@post('/container/<name>/start')
def container_start(name):
 container = lxc.Container(name)
 container.start(useinit=False, daemonize=True)

 return "Starting container {0}\n".format(name)

@post('/container/<name>/stop')
def container_start(name):
 container = lxc.Container(name)
 container.stop()

 return "Stopping container {0}\n".format(name)

@post('/container/<name>/destroy')
def container_start(name):
 container = lxc.Container(name)
 container.destroy()

 return "Destroying container {0}\n".format(name)

@post('/container/<name>/freeze')
def container_start(name):
 container = lxc.Container(name)
 container.freeze()

 return "Freezing container {0}\n".format(name)

@post('/container/<name>/unfreeze')
def container_start(name):
 container = lxc.Container(name)
 container.unfreeze()

 return "Unfreezing container {0}\n".format(name)

@get('/container/<name>/state')
def container_status(name):
 container = lxc.Container(name)
 state = container.state

return "The state of container {0} is {1}\n".format(name, state)

@get('/container/<name>/ips')
def container_status(name):
 container = lxc.Container(name)
 ip_list = container.get_ips()

return "Container {0} has the following IP's {1}\n".format(
    name, ip_list)

 @get('/list')
def list():
 container_list = lxc.list_containers()

 return "List of containers: {0}\n".format(container_list)

run(host='localhost', port=8080, debug=True)

```

# 总结

LXC 和 libvirt API 提供的 Python 绑定是编程创建和管理 LXC 容器的一个好方法。

在本章中，我们通过编写简单的代码片段，探索了两组 Python 绑定，这些代码实现了用户空间工具提供的大部分功能。事实上，了解这些 API 的最佳方法是查看命令行工具的源代码，尽管它们是用 C 语言实现的。

我们简要介绍了如何使用 Vagrant 配置 LXC，以便在隔离的环境中测试代码。我们在本章的结尾实现了一个简单的 RESTful API，该 API 使用我们之前探索的一些方法来配置、管理和销毁 LXC。在第五章，*在 LXC 中使用 Linux 桥接和 Open vSwitch 进行网络配置*，我们将探讨 LXC 的网络方面，使用 Linux 桥接、NAT 中的 Open vSwitch 和直接路由模式，并查看如何互联容器与宿主操作系统的示例。

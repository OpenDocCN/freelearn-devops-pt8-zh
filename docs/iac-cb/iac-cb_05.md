# 第五章：使用 Cloud-Init 配置最后一公里

本章将涵盖以下内容：

+   在 AWS、Digital Ocean 或 OpenStack 上使用 cloud-init

+   使用 cloud-init 处理文件

+   使用 cloud-init 配置服务器的时区

+   使用 cloud-init 管理用户、密钥和凭证

+   使用 cloud-init 管理仓库和软件包

+   在启动过程中使用 cloud-init 运行命令

+   使用 cloud-init 配置 CoreOS

+   使用 cloud-init 从头到尾部署 Chef 客户端

+   使用 cloud-init 部署远程 Docker 服务器

# 引言

Cloud-init 是一种云实例初始化系统，几乎所有 Linux 发行版都支持它。所有最近的发行版（Ubuntu、Arch、CentOS/Red Hat、Fedora 等）以及 CoreOS 系统中的一个变种都支持它。

使用 cloud-init 时，在启动云实例（无论是新实例还是已有实例）时，会执行多个操作：安装软件包、复制文件或 SSH 密钥、部署 Chef、定义仓库或重启（完成后）。

cloud-init 的作用范围确实是针对初始化阶段的；它不是一个配置管理工具，也不是用于后续更新配置的工具，像 Ansible 或 Chef 那样会反复运行。它仅用于确保实例已正确配置以进行下一步操作，并确保在启动时按顺序执行一组命令。换句话说，Terraform（在第二章，*使用 Terraform 配置 IaaS*，第三章，*深入 Terraform*，以及第四章，*用 Terraform 自动化完整基础设施* 中讨论的工具）非常适合定义底层基础设施的所有方面，但 cloud-init 是一个简单而强大的解决方案，用于处理首次启动以及后续启动，之后让像 Chef 或 Ansible 这样的完整配置管理工具发挥作用，持续管理实例的生命周期。

**Cloud-init** 被定义为一个简单的 YAML 文件（cloud-config），该文件通过云实例的 **user-data** 字段传递。接下来我们将看到它是如何工作的。

本章将展示 cloud-init 的最常见使用案例，例如复制文件、创建用户、管理 SSH 密钥、添加仓库和安装软件包、运行任意命令、引导 Chef 客户端或使用它管理 CoreOS 和 Docker。

# 在 AWS、Digital Ocean 或 OpenStack 上使用 cloud-init

由于 cloud-init 是为云实例提供初始化服务的系统，我们需要找到一种方法，将 cloud-config YAML 文件传递给引导过程。在所有支持 cloud-init 的 IaaS 提供商中，都有一个字段可以粘贴我们的文件。我们将回顾 cloud-init 在三个重要的 IaaS 提供商（AWS、Digital Ocean 和 OpenStack）上的工作原理。

## 准备工作

要完成此食谱，你需要在 Amazon Web Services、Digital Ocean 或某个 OpenStack 部署上有一个账户，或者如果你想尝试所有平台的话，可以在它们所有平台上都有账户！

## 如何操作…

为了说明 cloud-init 的使用，我们将在 Ubuntu 16.04 和 CentOS 7.2 上创建最简单的 cloud-config 文件，安装 `htop`、`tcpdump`、`docker` 或 `nmap` 等包，这些包通常不会在大多数 Linux 发行版中默认安装。这是一个非常简单的 cloud-config 文件示例：

```
#cloud-config
# Install packages on first boot
packages:
  - tcpdump
  - docker
  - nmap
```

### 在 Amazon Web Services 上使用 cloud-init

使用 AWS 控制台时，在启动实例时，点击 **高级详细信息**，我们就能粘贴我们的示例（和简单的）cloud-config YAML 文件，或者甚至可以直接上传它：

![在 Amazon Web Services 上使用 cloud-init](img/B05671_05_01.jpg)

在这种情况下，我们刚启动的 Ubuntu 16.04 实例已经安装了 `htop` 和 `tcpdump` 系统工具，以及该 Linux 发行版支持的 Docker 版本：

```
ubuntu@ip-172-31-40-77:~$ which htop
/usr/bin/htop
ubuntu@ip-172-31-40-77:~$ which tcpdump
/usr/sbin/tcpdump
ubuntu@ip-172-31-40-77:~$ docker --version
Docker version 1.11.2, build b9f10c9

```

### 注意

我们可以通过关闭实例手动更新某个实例的 `cloud-config.yml` 文件，然后在 **操作** 菜单下，进入 **实例设置** | **查看/更改用户数据**。重新启动 EC2 实例后，更新的配置将生效。

### 在 Digital Ocean 上使用 cloud-init

在 Digital Ocean 上的情况类似。在创建新的 **droplet** 时，确保在 **选择附加选项** 部分勾选 **用户数据** 复选框，并粘贴 cloud-config 文件内容：

![在 Digital Ocean 上使用 cloud-init](img/B05671_05_02.jpg)

在几秒钟的启动时间和软件包安装后，我们定制的 Ubuntu 发行版就可以使用了：

```
root@ubuntu-512mb-nyc3-01:~# which htop
/usr/bin/htop
root@ubuntu-512mb-nyc3-01:~# which tcpdump
/usr/sbin/tcpdump
root@ubuntu-512mb-nyc3-01:~# docker --version
Docker version 1.11.2, build b9f10c9

```

### 在 OpenStack 上使用 cloud-init

在 OpenStack 上创建实例时，使用 Horizon 控制面板，点击 **Post-Creation** 标签，并将 cloud-config YAML 内容粘贴到文本框中。或者，也可以上传文件：

![在 OpenStack 上使用 cloud-init](img/B05671_05_03.jpg)

验证所请求的软件包是否已安装，这次是在 CentOS 7.2 系统上：

```
[centos@cloud-init-demo ~]$ which nmap
/usr/bin/nmap
[centos@cloud-init-demo ~]$ docker --version
Docker version 1.10.3, build cb079f6-unsupported
[centos@cloud-init-demo ~]$ which tcpdump
/usr/sbin/tcpdump

```

### 将 cloud-init 和 Terraform 结合用于任何 IaaS

在之前的 Terraform 章节中，我们实际上已经使用过几次 `cloud-init` 文件。

在 Amazon Web Services 上，使用 `aws_instance` 资源启动 EC2 虚拟机时，我们通过 `user_data` 参数传递 cloud-config 文件内容，在这种情况下，使用 `file()` 插值：

```
resource "aws_instance" "vm" {
  ami           = "ami-643d4217"
  instance_type = "t2.micro"
  key_name      = "manual cloud init"
  user_data     = "${file("cloud-config.yml")}"
}
```

Digital Ocean 虚拟机的等效参数也是 `user_data`：

```
resource "digitalocean_droplet" "vm" {
  image              = "ubuntu-14-04-x64"
  name               = "ubuntu"
  region             = "ams3"
  size               = "512mb"
  ssh_keys           = ["keys/admin_key"]
  user_data          = "${file("cloud-config.yml")}"
}
```

# 使用 cloud-init 处理文件

我们每个人都会面临的一个早期需求是，在实例启动的初期就有一个文件、许可证或脚本。Cloud-init 提供了多种方法来将这些文件发送到新的实例中。我们将看看如何使用纯文本和 base64 数据编码来发送文件。

## 准备工作

要完成此食谱，你需要：

+   访问启用 cloud-config 的基础设施

## 如何操作…

我们将编写的第一个文件是**MOTD**（即**今日消息**），其根目录具有读写权限，其他用户只有只读权限。该文件的内容将直接从 cloud-config 文件中声明：

```
#cloud-config
write_files:
  - path: /etc/motd
    content: |
      This server is configured using cloud-init.
      Welcome.
    owner: root:root
    permissions: '0644'
```

当该机器启动时，将会有`/etc/motd`文件，并在登录时显示该字符串：

```
$ ssh ubuntu@server_ip
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-36-generic x86_64)
[...]
This server is configured using cloud-init.
Welcome.
[...]
ubuntu@ip-172-31-44-177:~$

```

另一种包括文件内容的方法是将其编码为 base64。假设我们想要创建一个名为`/etc/server-id`的文件，内容为`abc-123`，并且权限为`0600`。首先获取该文件的 base64 版本：

```
$ base64 server-id
YWJjLTEyMwo=

```

这是我们将集成到 cloud-config 文件`content`字段中的输出：

```
 - path: /etc/server-id
 content: YWJjLTEyMwo=
 encoding: b64
 permissions: '0600'

```

让我们验证远程内容是否符合预期：

```
$ ls -al /etc/server-id
-rw------- 1 root root 8 Sep 20 10:15 /etc/server-id
$ sudo cat /etc/server-id
abc-123

```

它工作了！我们的文件仅对所有者可读写，内容是`abc-123`。

另一种方法是使用`gzip`压缩文件，甚至将压缩后的 gzip 文件进行 base64 编码。

# 使用 cloud-init 配置服务器的时区

在新的实例上，一个非常常见的配置步骤是设置时区。这次我们将明确设置服务器的 EDT（纽约）时区（即使服务器运行在欧洲或其他地方）。有时候，尽早正确设置日期和时间非常重要（例如用于注册时间、延迟和其他依赖日期和时间的问题）。

### 注意

在大多数设置中，我个人更喜欢确保所有系统都设置为 GMT，无论它们位于地球的哪个地方，是否处于 GMT 时区。这样，当出现故障时，更容易调试、比较日志或行为，而不必浪费时间进行时区的计算。

## 准备就绪

要按此配方进行操作，你需要：

+   访问启用 cloud-config 的基础设施

## 如何操作…

要自动将服务器的时区设置为`America/New_York`，使用`timezone`指令：

```
#cloud-config
timezone: "America/New_York"
```

就是这样！我们的服务器现在已从一开始就配置为使用正确的时区：

```
$ date
Sun Sep 25 10:48:32 EDT 2016

```

事实上，这只是将`/etc/timezone`文件设置为正确的值：

```
$ cat /etc/timezone
America/New_York

```

# 使用 cloud-init 管理用户、密钥和凭证

我们很可能不会打算使用默认的 root 账户，甚至不会使用我们发行版中的默认用户账户（那些 ubuntu 或 centos 用户）。更有可能的是，我们在过程的早期就需要一个 Unix 账户，甚至在适当的配置管理工具介入之前。

假设我们的 IT 安全政策要求我们为 IT 安全团队创建一个名为`emergency`的用户账户，该账户位于一个名为`infosec`的组中，拥有无密码的`sudo`权限，并且只有简单的`/bin/sh` shell。此账户将自动填充一个授权的公钥。政策还要求删除默认的`ubuntu`账户。

## 准备就绪

要按此配方进行操作，你需要：

+   访问启用 cloud-config 的基础设施

## 如何操作…

要创建一个组，我们使用一个简单名为`groups`的指令，接受一个组列表。任何组都可以包含一个用户子列表，将这些用户添加到该组：

```
#cloud-config
groups:
  - infosec: [emergency]
```

要创建一个用户，我们使用名为`users`的指令，它接受一个用户列表。这个用户列表有一组键，例如用户所在的`groups`，`sudo`权限，默认使用的`shell`，或者用于授权的 SSH 公钥。以下是我们为用户`emergency`设置的示例：

```
users:
 - name: emergency
 groups: sudo
 shell: /bin/sh
 sudo: ['ALL=(ALL) NOPASSWD:ALL']
 ssh-authorized-keys:
 - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC+fAfzjw5+mUZ7nGokB0tzO9fOLKrjHGVlabpRUxvsIN/dRRmiBA9NDh5YRZ/ThAhn+RvPKGTBrXmuv3qWd/iWc3nie0fc2zDX1/Dc8EAIF9ybXfSxT2DXOWWLOvNdUVOZNifmsmCQ1z0p9hg3bo65c0ZEBpXHIk+l75uFWAIYZ/4jnXyFWz1ptmQR7gnAk2KBK19sj1Ii0pNjGyVbl5bNitWb3ulaviIT3FCswZoOsYvcLpOwQrMA3k12kEAb30CYpesGcq6WDHAZSpWkFvc3Cd/AET4/SjtyYpQVEhUn84v106WbNeDyJpUX6cz2WG2UaEqZc0VqZVhI63jG7wUR emergency@host

```

登录为`emergency`用户，使用私钥后，让我们验证 cloud-init 是否完成了任务：

```
$ whoami
emergency
$ groups emergency
emergency : emergency sudo
$ echo $SHELL
/bin/sh
$ sudo whoami
root

```

### 注意

我们从未明确要求删除默认的`ubuntu`用户账户：一旦创建初始用户，这一操作会自动完成。

然而，如果我们希望保留 Linux 发行版的默认用户，只需将以下`default`用户添加到`users`指令中：

```
users:
 - default

```

# 使用 cloud-init 管理仓库和软件包

除非我们需要一个非常特定版本的 Linux 发行版，否则很可能我们会希望尽快得到一个完全更新的系统（考虑到安全补丁和其他 bug 修复）。同样，我们通常期望新系统中可以使用一组工具。然而，事情可能会发生变化，默认工具可能会被移除——最好小心为上。如果我们的引导脚本需要`wget`、`curl`和`nmap`，我们应该确保在适当的配置管理工具（如 Chef 或 Puppet）开始工作之前，这些工具已经存在。我们还可能希望在应用关键的初始软件包（如内核）后重启服务器，或者添加自定义的包仓库。

## 准备工作

要执行这个步骤，你需要：

+   访问启用了 cloud-config 的基础设施

## 如何操作……

在引导完成后立即升级所有软件包，只需将`package_upgrade`指令设置为`true`：

```
#cloud-config
package_upgrade: true
```

另一个有用的指令是，如果包管理器要求重新启动系统（通常是内核更新时的常见情况），就会执行重启。通常最好尽早重启，以确保使用最安全的内核，但根据你自己的环境小心操作（你可能不希望在进行其他操作时重启，比如运行 Chef 或类似的管理软件）：

```
apt_reboot_if_required: true
```

为确保安装了所需的软件包，请使用`packages`指令：

```
packages:
  - htop
  - nmap
  - curl
  - wget
```

我们还可以使用`apt_sources`添加自定义的 APT 仓库：

```
apt_sources:
  -  source: "ppa:nginx/stable"
```

让我们启动一个新的实例，并验证它已完全更新，以便无法应用任何更新：

```
$ sudo apt-get dist-upgrade
Reading package lists... Done
Building dependency tree
Reading state information... Done
Calculating upgrade... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.

```

验证我们所需的工具是否可用：

```
$ which nmap
/usr/bin/nmap
$ which htop
/usr/bin/htop
$ which curl
/usr/bin/curl
$ which wget
/usr/bin/wget

```

好消息！现在我们可以确保从一开始就始终拥有一个完全更新的系统，并且安装了所需的工具集，甚至是我们自己的工具。

# 在启动过程中使用 cloud-init 运行命令

在引导新服务器或实例时，第一次引导通常与实例生命周期中的其他引导有很大不同，我们通常希望在引导过程中非常早或非常晚执行一些命令。例如，假设我们的云实例启动时附带了一个块存储。我们可能想要格式化这个存储空间并确保它挂载到主机上，但虽然我们始终希望磁盘能够挂载，我们可能不希望它在每次引导时都被格式化！`bootcmd`指令用于处理引导过程中非常早期需要执行的命令，而`runcmd`指令则在引导过程中较晚执行（且仅执行一次）。

### 注意

`bootcmd`将在每次实例引导时执行。

## 准备工作

要执行这个步骤，你需要：

+   访问启用 cloud-config 的基础设施

## 如何操作…

我们将在引导过程中启动三个命令。第一个是一个包含动态内容的简单文件（由 cloud-init 提供的`$INSTANCE_ID`变量），它将在每次引导时无论如何都会被重写。第二个命令是将日期打印到日志中（这样我们可以知道引导过程何时开始）。最后一个命令是格式化附加到`/dev/xvdb`上的块设备为`ext4`格式。为了演示，我们还将在主机上将新设备挂载到`/srv/www`下。

要在每次机器启动时尽可能早地执行某个命令，只需将其添加到`bootcmd`指令的命令列表中：

```
#cloud-config
bootcmd:
  - echo bootcmd started at $(date)
  - echo $INSTANCE_ID > /etc/instance_id
```

如果我们删除或修改这个文件，下次重启时它将被覆盖。

另一方面，如果我们只想在`bootcmd`指令中运行一次命令，可以使用辅助脚本`cloud-init-per`。你可以选择每次`boot`时或每个`instance`时执行一次命令。在我们的例子中，我们想要格式化`/dev/xvdb`设备（因此，除非我们希望每次重启时都格式化磁盘，否则我们可能只希望在此实例上执行一次。让我们将`instance`参数添加到`cloud-init-per`辅助脚本中）：

```
#cloud-config
bootcmd:
  - cloud-init-per instance mkfs-xvdb mkfs -t ext4 /dev/xvdb
```

最后，让我们使用`mounts`指令将现在格式化的`/dev/xvdb`挂载到`/srv/www`文件夹：

```
mounts:
  - [ /dev/xvdb, /srv/www ]
```

启动后，让我们验证块设备是否已挂载：

```
# df -h /srv/www/
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvdb       4.8G   10M  4.6G   1% /srv/www
```

我们还可以测试我们创建的文件是否存在：

```
# cat /etc/instance_id
i-03005dd324599df11
```

尝试删除这个文件并重启服务器：文件将再次出现。

现在，让我们来看一下`runcmd`指令有何不同。我们将向`bootcmd`指令中的日期输出添加一个非常相似的命令：

```
runcmd:
 - 'echo runcmd started at $(date)'
```

启动一个新实例，并观察时间戳的变化：

```
$ grep "started at" /var/log/cloud-init-output.log
bootcmd started at Fri Sep 23 07:02:35 UTC 2016
+ echo runcmd started at Fri Sep 23 07:02:47 UTC 2016
runcmd started at Fri Sep 23 07:02:47 UTC 2016

```

`runcmd`指令比`bootcmd`指令晚 12 秒启动。

现在重新启动实例，并观察到`runcmd`没有再次执行：

```
$ grep "started at" /var/log/cloud-init-output.log
bootcmd started at Fri Sep 23 07:04:31 UTC 2016

```

现在我们知道在每种情况下该使用哪个指令了。

# 使用 cloud-init 配置 CoreOS

CoreOS 支持其自有版本的 cloud-init，并增强了对 CoreOS 环境的支持，同时不包含任何与其环境不兼容的部分，这样我们就可以启动一个完全配置好的系统和集群。

我们将看看 CoreOS 的特殊性，参考之前的提示，学习如何管理用户、文件、授权的 SSH 密钥以及其他标准的 cloud-init 指令。在这部分结束时，您将学会如何配置 etcd 键值存储、fleet 集群管理器、flannel 覆盖网络、控制更新机制，并确保 systemd 单元尽早启动。

### 注意

CoreOS 提供了一个非常有用的云配置文件验证工具，位于 [`coreos.com/validate/`](https://coreos.com/validate/)。当我们不确定某个指令是否被发行版支持时，这个工具非常有用。

## 准备工作

要完成本配方，您将需要：

+   访问启用云配置的基础设施

## 如何操作……

我们将介绍可以为 CoreOS 配置的最重要选项。这包括 etcd 分布式键值存储、fleet 调度器、fleet 网络、更新策略以及一些 systemd 单元配置。

### 使用 cloud-init 配置 etcd

etcd 键值存储在 CoreOS 中用于在同一集群的成员之间共享多个配置数据。首先，我们需要一个发现令牌，可以从[`discovery.etcd.io/new`](https://discovery.etcd.io/new)获得。

```
$ curl -w "\n" 'https://discovery.etcd.io/new'
https://discovery.etcd.io/638d980c4edf94d6ddff8d6e862bc7d9
```

### 注意

我们可以通过在 URL [`discovery.etcd.io/new?size=3`](https://discovery.etcd.io/new?size=3) 中添加 `size=` 参数来指定 CoreOS 集群的最小所需大小。

现在我们有了有效的发现令牌，让我们将其添加到 `cloud-config.yml` 文件中的 `etcd2` 指令下：

```
#cloud-config
coreos:
  etcd2:
    discovery: "https://discovery.etcd.io/638d980c4edf94d6ddff8d6e862bc7d9"
```

下一步是配置 etcd：

+   etcd 应该如何监听对等流量？（`listen-peer-urls`）。我们希望使用本地接口，并使用默认端口（TCP/`2380`）。

+   etcd 应该如何监听客户端流量？（`listen-client-urls`）。我们希望使用所有可用的接口，并使用默认端口（TCP/`2379`）。

+   etcd 应该如何初步向集群的其他节点通告？（`initial-advertise-peer-urls`）。我们希望使用本地接口，并使用相同的对等流量端口（TCP/`2380`）。

+   etcd 应该如何向集群的其他节点通告客户端 URL？（`advertise-client-urls`）。我们希望使用本地接口，并使用相同的客户端流量端口（TCP/`2379`）。

为了使配置更具动态性，我们可以使用兼容大多数 IaaS 提供商的变量——`$private_ipv4` 和 `$public_ipv4`。

这就是我们包含所有 etcd 配置的 `cloud-config.yml` 文件：

```
#cloud-config
coreos:
  etcd2:
    discovery: "https://discovery.etcd.io/b8724b9a1456573f4d527452cba8ebdb"
    advertise-client-urls: "http://$private_ipv4:2379"
    listen-client-urls: "http://0.0.0.0:2379"
    initial-advertise-peer-urls: "http://$private_ipv4:2380"
    listen-peer-urls: "http://$private_ipv4:2380"
```

这将生成在 `/run/systemd/system/etcd2.service.d/20-cloudinit.conf` 中找到的 `systemd` 单元文件中的正确变量。

```
$ cat /run/systemd/system/etcd2.service.d/20-cloudinit.conf
[Service]
Environment="ETCD_ADVERTISE_CLIENT_URLS=http://172.31.15.59:2379"
Environment="ETCD_DISCOVERY=https://discovery.etcd.io/b8724b9a1456573f4d527452cba8ebdb"
Environment="ETCD_INITIAL_ADVERTISE_PEER_URLS=http://172.31.15.59:2380"
Environment="ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379"
Environment="ETCD_LISTEN_PEER_URLS=http://172.31.15.59:2380"

```

当我们的集群准备好后，我们可以通过指定端口以客户端身份请求信息：

```
$ etcdctl cluster-health
member 7466dcc2053a98a4 is healthy: got healthy result from http://172.31.15.59:2379
member 8f9bd8a78e0cca38 is healthy: got healthy result from http://172.31.8.96:2379
member e0f77aacba6888fc is healthy: got healthy result from http://172.31.1.27:2379
cluster is healthy

```

我们还可以浏览 etcd 键值存储，以确认是否能够访问它：

```
$ etcdctl ls
/coreos.com

```

### 使用 cloud-init 配置 fleet

Fleet 是基于 systemd 的分布式初始化管理器，我们使用它在我们的 CoreOS 集群上安排服务。

最重要的配置参数如下：

+   `public_ip`：这指定要用于与其他主机通信的接口。我们希望主机的公共 IP，以便我们可以直接从我们的工作站与 fleet 互动。

+   `metadata`: 这是与我们需求相关的任何关键值，因此我们可以相应地安排单元。我们想要存储提供者（`aws`），地区（`eu-west-1`）以及集群的名称（`mycluster`）。这完全是任意的；请根据您自己的需求调整键和值。

这是在`cloud-config.yml`文件中的样子：

```
coreos:
  fleet:
    public-ip: "$public_ipv4"
    metadata: "region=eu-west-1,provider=aws,cluster=mycluster"
```

这将在`/run/systemd/system/fleet.service.d/20-cloudinit.conf`中的 systemd 单元中生成正确的变量：

```
$ cat /run/systemd/system/fleet.service.d/20-cloudinit.conf
[Service]
Environment="FLEET_METADATA=region=eu-west-1,provider=aws,cluster=mycluster"
Environment="FLEET_PUBLIC_IP=52.209.159.4"
```

使用 fleet 超出了本书的范围，但我们至少可以从实例验证与 fleet 集群管理器的连接是否正常：

```
$ fleetctl list-machines
MACHINE         IP              METADATA
441bf02a...     52.31.10.18     cluster=mycluster,provider=aws,region=eu-west-1
b95a5262...     52.209.159.4    cluster=mycluster,provider=aws,region=eu-west-1
d9fa1d18...     52.31.109.156   cluster=mycluster,provider=aws,region=eu-west-1
```

现在我们可以提交并启动我们工作中的 fleet 集群上的服务！

### 使用 cloud-init 配置更新策略

CoreOS 可以以各种方式处理更新，包括在新的 CoreOS 版本可用后立即重新启动，使用 etcd 进行理想时间的调度，使集群永不中断，两者混合（默认），甚至永不重新启动。我们还可以明确指定要使用的 CoreOS 通道（稳定版、测试版或 Alpha 版）。我们希望使用`etcd-lock`策略确保集群永不中断，并确保使用稳定的发布版：

```
coreos:
  update:
    reboot-strategy: "etcd-lock"
    group: "stable"
```

此部分生成`/etc/coreos/update.conf`文件：

```
$ cat /etc/coreos/update.conf
GROUP=stable
REBOOT_STRATEGY=etcd-lock
```

我们可以强制执行更新检查以验证其是否正常工作（从具有可用更新的系统中获取的示例）：

```
$ sudo update_engine_client -update
[0924/131749:INFO:update_engine_client.cc(243)] Initiating update check and install.
[0924/131750:INFO:update_engine_client.cc(248)] Waiting for update to complete.
CURRENT_OP=UPDATE_STATUS_UPDATE_AVAILABLE
[...]
```

### 使用 cloud-init 配置 locksmith

现在我们确信更新系统已正确触发，我们面临一个新问题：当更新可用时，我们的集群节点可以随时重新启动。在高负载环境中可能不太理想。因此，我们可以配置**locksmith**，仅允许在特定时间范围内重新启动，例如“每周五到周六的夜间，从凌晨 4 点到 6 点”。我们不限于一天，所以我们也可以允许在凌晨 4 点时的任何一天重新启动：

```
coreos:
  locksmith:
    window-start: Sat 04:00
    window-length: 2h 
```

这将在`/run/systemd/system/locksmithd.service.d/20-cloudinit.conf`中生成以下内容：

```
$ cat /run/systemd/system/locksmithd.service.d/20-cloudinit.conf
[Service]
Environment="REBOOT_WINDOW_START=04:00"
Environment="REBOOT_WINDOW_LENGTH=2h"
```

随时可以使用`locksmithctl`命令检查重启槽的可用性：

```
$ locksmithctl status
Available: 1
Max: 1
```

如果另一台机器当前正在重新启动，其 ID 将显示出来，以便我们知道是谁在重新启动。

### 使用 cloud-init 配置 systemd 单元

我们可以轻松地从 cloud-init 管理单元，因此系统的关键部分在需要时立即启动。例如，我们知道我们希望 etcd2 和 fleet 服务在每次启动时启动：

```
  coreos: 
units:
    - name: etcd2.service
      command: start
    - name: fleet.service
      command: start 
```

### 使用 cloud-init 配置 flannel

Flannel 用于在集群中的所有主机之间创建一个覆盖网络，这样容器就可以通过网络相互通信，无论它们在哪个节点上运行。为了在启动 Flannel 之前进行配置，我们可以向 cloud-config 文件中添加更多配置内容。我们知道希望我们的 Flannel 网络在 10.1.0.0/16 网络上运行，因此我们可以创建一个 drop-in systemd 配置文件，其中包含将在`flanneld`服务之前执行的内容。在这种情况下，设置 Flannel 网络的操作是通过将键/值组合写入 etcd 的`/coreos.com/network/config`下完成的：

```
coreos:
  units:
    - name: flanneld.service
      drop-ins:
        - name: 50-network-config.conf
          content: |
            [Service]
            ExecStartPre=/usr/bin/etcdctl set /coreos.com/network/config '{ "Network": "10.1.0.0/16" }'
```

这将仅创建文件`/etc/systemd/system/flanneld.service.d/50-network-config.conf`：

```
$ cat /etc/systemd/system/flanneld.service.d/50-network-config.conf
[Service]
ExecStartPre=/usr/bin/etcdctl set /coreos.com/network/config '{ "Network": "10.1.0.0/16" }'
```

验证我们是否在正确的 IP 网络范围内有一个正确的`flannel0`接口：

```
$ ifconfig flannel0
flannel0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 8973
        inet 10.1.19.0  netmask 255.255.0.0  destination 10.1.19.0
[...]
```

启动一个容器，以验证它是否也在 10.1.0.0/16 网络中运行：

```
$ docker run -it --rm alpine ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 02:42:0A:01:13:02
          inet addr:10.1.19.2  Bcast:0.0.0.0  Mask:255.255.255.0
[...]
```

一切都运行得非常顺利！

### 注意

请注意，启动接口可能需要一些时间，这取决于主机的网络连接速度，因为 flannel 是从一个容器中运行的，首先需要下载该容器（截至目前为 51 MB）。

我们现在了解了使用 cloud-init 自动引导 CoreOS 集群的最有用配置选项。

# 从头到尾使用 cloud-init 部署 Chef 客户端

我们可以通过 cloud-init 使用官方的**omnibus**安装程序来部署 Chef。这个安装程序包含了部署 Chef 及其所有依赖项所需的所有内容。接着，我们将配置 Chef 客户端，以便与 Chef 服务器组织进行安全的身份验证，并最终应用初始的 cookbook。

### 注意

警告：当前与 Ubuntu 16.04 LTS 和 CentOS 7 一起发布的 cloud-init 版本在安装 Chef 时存在问题。此步骤使用的是 Ubuntu 14.04 LTS，等待该问题修复。

## 准备工作

要按照此步骤进行操作，您将需要以下内容：

+   访问启用了 cloud-config 的基础设施

+   一个可用的 Chef 服务器和组织设置

## 如何操作…

与 Chef 相关的所有内容都在名为`chef`的指令下进行配置。

### 使用 cloud-init 部署 Chef omnibus 安装程序

由于我们希望使用官方的 omnibus 构建（其他选择包括通过 Ruby gem 安装 Chef——这种方法已经不推荐使用，并且过于依赖本地安装的 Ruby 版本，或者通过已记录的包进行安装），让我们将安装类型定义为`omnibus`，并确保即使出于某种原因，Chef 客户端已经安装在系统上，也能安装该版本。最后，我们显式定义安装程序的完整 URL，以确保我们安装的是正确的版本（可能将其指向您自己服务器上的本地版本）。

```
#cloud-config
chef:
  install_type: "omnibus"
  force_install: true
  omnibus_url: "https://www.getchef.com/chef/install.sh"
```

这将在 cloud-init 日志中输出类似以下内容：

```
Getting information for chef stable  for ubuntu...
downloading https://omnitruck-direct.chef.io/stable/chef/metadata?v=&p=ubuntu&pv=14.04&m=x86_64
  to file /tmp/install.sh.1294/metadata.txt
[...]
version 12.14.89
[...]
Installing chef
[...]
Unpacking chef (12.14.89-1) ...
Setting up chef (12.14.89-1) ...
Thank you for installing Chef!
```

此时，您将会在`/opt/chef`目录下拥有一个有效的 Chef 安装，尽管尚未进行配置。

### 使用 cloud-init 配置 Chef 与 Chef 服务器组织

chef 客户端需要以下三项信息才能正确地在现有的 Chef Server 组织中进行身份验证：Chef 服务器的 URL（[`api.chef.io/organizations/iacbook`](https://api.chef.io/organizations/iacbook)）、允许您向该组织添加节点的私钥，以及与此密钥关联的名称（默认为组织名称，如 `iacbook`）。这些信息在 cloud-config 文件中表示如下：

```
#cloud-config
chef:
  server_url: "https://api.chef.io/organizations/iacbook"
  validation_name: "iacbook"
  validation_cert: |
    -----BEGIN RSA PRIVATE KEY-----
    MIIEowIBAAKCAQEAuR[...]
    -----END RSA PRIVATE KEY-----
```

有了这些信息，初次运行 chef-client 时将能够在 Chef 组织中进行身份验证并添加节点。在 cloud-init 日志中，此步骤出现在以下时刻：

```
[...]
Starting Chef Client, version 12.14.89
Creating a new client identity for i-0913e870fb28af4bd using the validator key.
[...]
```

### 使用 cloud-init 在引导时应用 Chef 食谱

我们当然希望至少应用一个初步的食谱来配置实例。在这种情况下，我们将简单地应用随启动套件一起提供的初始食谱，但我们可以根据需要添加任意数量的角色和食谱。有关获取此食谱的更多信息，请参阅本书的专门章节：

```
#cloud-config
chef:
  run_list:
  - "recipe[starter]"
```

在日志中，我们将看到它以这种方式应用：

```
[...]
Loading cookbooks [starter@1.0.0]
Storing updated cookbooks/starter/attributes/default.rb in the cache.
Storing updated cookbooks/starter/recipes/default.rb in the cache.
Storing updated cookbooks/starter/templates/default/sample.erb in the cache.
Storing updated cookbooks/starter/files/default/sample.txt in the cache.
Storing updated cookbooks/starter/metadata.rb in the cache.
Processing log[Welcome to Chef, Sam Doe!] action write (starter::default line 4)
Welcome to Chef, Sam Doe!
Chef Run complete in 2.625856409 seconds
```

我们的实例现在已经自动注册并配置完成，只需要在 cloud-config 文件中写几行代码即可。

# 使用 cloud-init 部署远程 Docker 服务器

拥有一个远程 Docker 服务器而非工作站默认的本地配置可能非常有用，原因包括带宽问题、测试生产环境、客户演示或远程团队协作。能够向远程服务器发送常规 Docker 命令有很多优势。为了速度和舒适性，我们将部署一个基本的 CoreOS 系统，添加一个用户（Jane）及其公钥。Docker 将被修改为通过类似 socket 的 systemd 服务监听网络，我们还将把服务器时区配置为纽约。

## 准备工作

要执行这个食谱，您需要：

+   访问启用 cloud-config 的基础设施

## 如何实现...

我们先从简单地将这个服务器命名为 `"docker"` 开始：

```
#cloud-config
hostname: "docker"
```

在最终系统中，这将把主机名设置为正确的值：

```
$ hostname
docker
$ cat /etc/hostname
docker
```

现在，让我们创建 `Jane` 用户，这样她就可以登录实例并远程帮助我们。她需要加入 `docker` 组，以便能够操作容器，并且她已经提供了她的 SSH 公钥。这是如何在 cloud-config 文件中表示的：

```
#cloud-config
users:
  - name: "jane"
    gecos: "Jane Docker"
    groups:
      - "docker"
    ssh-authorized-keys:
      - "ssh-rsa AAAAB[...] jane"
```

在最终系统中，Jane 可以使用她的私钥登录，并且由于她是 docker 组的成员，可以与 docker 守护进程进行交互：

```
jane@docker ~ $ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

SSH 公钥最终会出现在以下文件中：

```
jane@docker ~ $ cat .ssh/authorized_keys.d/coreos-cloudinit
ssh-rsa AAAAB [..] jane
```

### 使用 cloud-init 设置 CoreOS 的时区

CoreOS 使用一个基于 NTP（网络时间协议）系统，通过 `timedatectl` 命令进行控制。我们在 CoreOS 上找不到通常的 `/etc/timezone` 文件，因此我们在本书中之前看到的 cloud-init 的默认 `timezone` 指令将不起作用。要将时区设置为纽约，我们可以这样设置：

```
$ /usr/bin/timedatectl set-timezone America/New_York
```

很简单！所以我们通过 cloud-config 文件中的 systemd 单元来启动该命令，以确保时区已经设置。对 systemd 的深入了解超出了本书的范围，但为了做到这一点，我们需要为该单元添加两个选项：一个告诉 systemd 即使命令退出了也不要认为该单元崩溃了（`RemainAfterExit=yes`），另一个告诉该单元类型不是执行长期运行的进程，而是一个应该在继续之前退出的短期进程（`Type=oneshot`）。

这是 `cloud-config.yml` 文件中的单元：

```
coreos:
  units:
  - name: settimezone.service
    command: start
    content: |
      [Unit]
      Description=Setting the timezone

      [Service]
      ExecStart=/usr/bin/timedatectl set-timezone America/New_York
      RemainAfterExit=yes
      Type=oneshot
```

### 启用 Docker TCP 套接字以进行网络访问

我们的最终目标是能够从工作站远程使用 Docker 引擎。默认的 Docker 配置是监听 Unix 套接字（`/var/run/docker.sock`），而我们希望它监听 TCP 套接字上的 2375 端口（默认的非加密端口，强烈建议配置 TLS 加密；按照惯例，这将使用 TCP/2376）。为了配置这一点，我们将使用 systemd 的一项功能——套接字激活。简而言之，这会创建一个 systemd 服务，监听 2375 端口，并同时启动常规的 `docker.service` 单元和套接字描述。这样，这个特定的 Docker 引擎将会响应 TCP 套接字上的请求，而不是 Unix 套接字（同时可以激活更多 TCP 套接字，或保持默认的 `docker.service` 清洁）。以下是配置示例：

```
coreos:
  units:
  - name: docker-tcp.socket
    command: start
    enable: true
    content: |
      [Unit]
      Description=Docker Socket for the API

      [Socket]
      ListenStream=2375
      BindIPv6Only=both
      Service=docker.service

      [Install]
      WantedBy=sockets.target
```

让我们启动一个远程服务器，使用完整的配置并稍作演示（在这个例子中，Docker 远程主机是 `52.211.117.98`，我们将启动一个带有 HTTP 端口转发的 `nginx` 容器）。有关使用的命令行选项的更多信息，请参考本书的 Docker 部分：

```
user@workstation $ docker -H 52.211.117.98 run -it --rm -p 80:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
6a5a5368e0c2: Pull complete
4aceccff346f: Pull complete
c8967f302193: Pull complete
Digest: sha256:1ebfe348d131e9657872de9881fe736612b2e8e1630e0508c354acb0350a4566
Status: Downloaded newer image for nginx:latest  
1.2.3.4 - - [25/Sep/2016:16:06:30 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36" "-"
```

在远程 Docker 主机的 HTTP 端口上进行一些请求，它将会响应。现在我们拥有了一个完全按需的 CoreOS 主机，能够通过巧妙的 systemd 配置功能远程控制 Docker 引擎！

## 还有更多...

当连接到不同的远程 Docker 引擎时，我们迟早会遇到连接到一个服务器，而这个服务器的版本与我们客户端的版本不同。在这种情况下，我们会收到以下错误：

```
Error response from daemon: client is newer than server (client API version: 1.24, server API version: 1.22)
```

一个简单的解决方法是重写 `DOCKER_API_VERSION` 环境变量，并将其设置为与服务器相同的值（在这个例子中为 1.22）：

```
$ DOCKER_API_VERSION=1.22 docker -H 52.211.117.98 ps
```

### 注意

Docker 1.13 大大改善了这种情况，通过在 CLI 中直接管理客户端和服务器之间的版本/功能协商。

## 另见

+   有关 systemd 套接字激活的更多信息，请参考 [`0pointer.de/blog/projects/socket-activation.html`](http://0pointer.de/blog/projects/socket-activation.html)。

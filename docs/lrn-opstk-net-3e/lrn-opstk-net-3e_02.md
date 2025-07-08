# 第二章：安装 OpenStack

手动安装、配置和维护 OpenStack 云可能是一项艰巨的任务。许多供应商提供基于 OpenStack 的可下载云软件，使用 Chef、Puppet、Ansible 等工具提供部署和管理策略。

本章将逐步指导您在 Ubuntu 16.04 LTS 操作系统上安装以下 OpenStack 组件：

+   OpenStack 身份（Keystone）

+   OpenStack 镜像服务（Glance）

+   OpenStack 计算（Nova）

+   OpenStack 仪表板（Horizon）

本章中记录的安装过程基于*OpenStack 安装指南*，该指南可以在 [`docs.openstack.org/`](http://docs.openstack.org/) 找到。如果您希望在其他操作系统上安装 OpenStack，可以参考该网站上的指南。

如果您更愿意下载并安装基于 OpenStack 的第三方云发行版，可以尝试以下 URL：

+   OpenStack-Ansible: [`docs.openstack.org/openstack-ansible/`](https://docs.openstack.org/openstack-ansible/)

+   Red Hat RDO: [`openstack.redhat.com/`](http://openstack.redhat.com/)

+   Kolla: [`docs.openstack.org/kolla-ansible/latest/`](https://docs.openstack.org/kolla-ansible/latest/)

一旦安装完成，本书中使用的大部分概念和示例仍然适用于上述发行版，但可能需要额外的努力才能实现。

# 系统要求

OpenStack 组件旨在运行于标准硬件上，从桌面计算机到企业级服务器均可支持。为了获得最佳性能，`计算`节点的处理器需要支持虚拟化技术，如英特尔的 VT-x 或 AMD 的 AMD-V。

本书假设 OpenStack 将安装在满足以下最低要求的服务器上：

| **服务器** | **硬件要求** |
| --- | --- |
| 所有 | 处理器：64 位 x86CPU 数量：2-4 内存：4+ GB RAM 磁盘空间：32+ GB 网络：3x 1-Gbps 网络接口卡 |

虽然不符合这些最低要求的机器也能根据本书中提供的文档进行安装，但这些最低要求是为了确保顺利的体验。在创建多个虚拟机实例用于演示时，强烈建议增加额外的内存和存储。像 VMware ESXi、VMware Fusion 或 VirtualBox 等虚拟化产品可以替代物理硬件使用，但需要对环境和 OpenStack 进行额外的配置，这超出了本书的范围。如果选择虚拟化环境，建议启用硬件虚拟化扩展，以提升性能。

# 操作系统要求

OpenStack 可以安装在以下 Linux 发行版上：CentOS、Red Hat Enterprise Linux、openSUSE、SUSE Linux Enterprise Server 和 Ubuntu。本书假设在安装 OpenStack 之前，所有主机已经安装了 Ubuntu 16.04 LTS 服务器操作系统。你可以在以下网址找到 Ubuntu Server：[`www.ubuntu.com/download/server`](http://www.ubuntu.com/download/server)。

为了支持本书中讨论的所有网络功能，推荐使用以下最小内核版本：`4.13.0-45-generic`。

# 初始网络配置

为了理解每台主机应该如何初步配置网络，请参考以下图示：

![](img/1effdc54-7fb1-4547-ab01-8c02063634fb.png)

图 2.1

在*图 2.1*中，三个接口分别连接到每台主机。`eth0`接口将作为 OpenStack 服务和 API 访问的管理接口，而`eth1`将用于主机之间的覆盖网络流量。在`controller`节点，`eth2`将用于通过 Neutron 路由器向实例传递外部网络流量。如果使用 VLAN 租户网络代替覆盖网络，则`eth2`将配置在`compute`节点上，以支持这些网络。

至少，管理接口应该配置一个 IP 地址，并具有访问互联网的外部连接。需要互联网访问才能从 Ubuntu 软件包仓库下载 OpenStack 软件包。建议通过 SSH 从受信任的网络访问服务器的管理地址。

# 示例网络

在本书中，将会有配置和使用各种 OpenStack 服务的示例。下表列出了用于这些服务的 VLAN 和相关网络：

| **VLAN 名称** | **VLAN ID** | **网络** |
| --- | --- | --- |
| MGMT_NET | 10 | 10.10.0.0/24 |
| OVERLAY_NET | 20 | 10.20.0.0/24 |
| GATEWAY_NET | 30 | 10.30.0.0/24 |
| PROJECT_NET40 | 40 | 待定 |
| PROJECT_NET41 | 41 | 待定 |
| PROJECT_NET42 | 42 | 待定 |
| PROJECT_NET43 | 43 | 待定 |

以下表格提供了每个主机接口推荐的 IP 地址和 VLAN ID，供您在跟随示例时参考：

| **主机名** | **接口** | **IP 地址** | **交换端口** | **VLAN ID** |
| --- | --- | --- | --- | --- |
| controller01 | eth0 | 10.10.0.100 | 访问端口 | VLAN 10（未标记） |
| controller01 | eth1 | 10.20.0.100 | 访问端口 | VLAN 20（未标记） |
| controller01 | eth2 | 无 | Trunk 端口 | VLAN 30, 40-44（标记） |
| **主机名** | **接口** | **IP 地址** | **交换端口** | **VLAN ID** |
| compute01 | eth0 | 10.10.0.101 | 访问端口 | VLAN 10（未标记） |
| compute01 | eth1 | 10.20.0.101 | 访问端口 | VLAN 20（未标记） |
| compute01 | eth2 | 无 | Trunk 端口 | VLAN 30, 40-44（标记） |
| **主机名** | **接口** | **IP 地址** | **交换端口** | **VLAN ID** |
| compute02 | eth0 | 10.10.0.102 | 访问端口 | VLAN 10（未标记） |
| compute02 | eth1 | 10.20.0.102 | 访问端口 | VLAN 20（未标记） |
| compute02 | eth2 | 无 | 中继端口 | VLAN 30, 40-44（已标记） |
| **主机名** | **接口** | **IP 地址** | **交换端口** | **VLAN ID** |
| compute03 | eth0 | 10.10.0.103 | 访问端口 | VLAN 10（未标记） |
| compute03 | eth1 | 10.20.0.103 | 访问端口 | VLAN 20（未标记） |
| compute03 | eth2 | 无 | 中继端口 | VLAN 30, 40-44（已标记） |
| **主机名** | **接口** | **IP 地址** | **交换端口** | **VLAN ID** |
| snat01 | eth0 | 10.10.0.104 | 访问端口 | VLAN 10（未标记） |
| snat01 | eth1 | 10.20.0.104 | 访问端口 | VLAN 20（未标记） |
| snat01 | eth2 | 无 | 中继端口 | VLAN 30, 40-44（已标记） |

为了避免因接口配置错误或其他问题导致连接丢失，强烈建议使用带外管理访问服务器或其他类似机制。

# 接口配置

Ubuntu 16.04 LTS 使用位于`/etc/network/interfaces.d/`目录或`/etc/network/interfaces`文件中的配置文件来描述如何配置网络接口。

使用文本编辑器，在每个主机上更新网络接口文件，方法如下。

对于`controller01`：

```
auto eth0
iface eth0 inet static
    address 10.10.0.100
    netmask 255.255.255.0
    gateway 10.10.0.1
    dns-nameserver 8.8.8.8
auto eth1
iface eth1 inet static
    address 10.20.0.100
    netmask 255.255.255.0
auto eth2
iface eth2 inet manual 
```

对于`compute01`：

```
auto eth0
iface eth0 inet static
    address 10.10.0.101
    netmask 255.255.255.0
    gateway 10.10.0.1
    dns-nameserver 8.8.8.8
auto eth1
iface eth1 inet static
   address 10.20.0.101
   netmask 255.255.255.0
auto eth2
iface eth2 inet manual
```

对于`compute02`：

```
auto eth0
iface eth0 inet static
    address 10.10.0.102
    netmask 255.255.255.0
    gateway 10.10.0.1
 dns-nameserver 8.8.8.8
auto eth1
iface eth1 inet static
    address 10.20.0.102
    netmask 255.255.255.0
auto eth2
iface eth2 inet manual
```

对于`compute03`：

```
auto eth0
iface eth0 inet static
   address 10.10.0.103
   netmask 255.255.255.0
   gateway 10.10.0.1
   dns-nameserver 8.8.8.8
auto eth1
iface eth1 inet static
   address 10.20.0.103
   netmask 255.255.255.0
auto eth2
iface eth2 inet manual
```

对于`snat01`：

```
auto eth0
iface eth0 inet static
   address 10.10.0.104
   netmask 255.255.255.0
   gateway 10.10.0.1
   dns-nameserver 8.8.8.8
auto eth1
iface eth1 inet static
   address 10.20.0.104
   netmask 255.255.255.0
auto eth2
iface eth2 inet manual
```

`eth2`接口将在后续章节中描述的网络桥接中使用。要激活更改，请使用`ifdown`和`ifup`命令在每个节点上重启接口：

```
    # ifdown --all; ifup --all
```

有关配置网络接口的更多信息，请参考 Ubuntu 手册页，网址如下：[`manpages.ubuntu.com/manpages/xenial/man5/interfaces.5.html`](http://manpages.ubuntu.com/manpages/xenial/man5/interfaces.5.html)

# 初步步骤

在我们可以安装 OpenStack 之前，必须做一些准备工作，以确保每个系统能够成功安装。

# 权限

OpenStack 服务可以作为 root 用户或具有`sudo`权限的用户安装。后者可能需要将用户添加到每个主机的`sudoers`文件中。有关配置`sudoers`的提示，请访问以下网址：[`help.ubuntu.com/community/RootSudo.`](https://help.ubuntu.com/community/RootSudo)

对于此安装，除非另有说明，否则所有命令都应以 root 用户身份运行。

# 配置 OpenStack 仓库

当安装版本比操作系统自带版本更新的 OpenStack 时，Ubuntu 使用 Ubuntu Cloud Archive（UCA）。要启用 Cloud Archive 仓库，请更新`apt`缓存，并在所有主机上下载并安装`software-properties-common`包：

```
# apt update; apt install software-properties-common
```

安装完成后，应该将 OpenStack Pike 仓库添加为所有主机的 apt 源：

```
# add-apt-repository cloud-archive:pike
```

# 系统升级

在安装 OpenStack 之前，必须将每个节点上的内核和其他系统包升级到 Ubuntu 16.04 LTS 版本中通过 Cloud Archive 提供的最新版本。请在每个节点上执行以下`apt`命令：

```
    # apt update
    # apt dist-upgrade
```

# 设置主机名

在安装 OpenStack 之前，确保环境中的每个节点都已配置正确的主机名。使用`hostnamectl`命令在每个主机上相应设置主机名：

| **主机** | **命令** |
| --- | --- |
| `controller01` | `hostnamectl set-hostname controller01` |
| `compute01` | `hostnamectl set-hostname compute01` |
| `compute02` | `hostnamectl set-hostname compute02` |
| `compute03` | `hostnamectl set-hostname compute03` |
| `snat01` | `hostnamectl set-hostname snat01` |

为简化主机之间的通信，建议使用 DNS 或本地名称解析器来解析主机名。使用文本编辑器，更新每个节点上的`/etc/hosts`文件，以包含所有节点的管理 IP 地址和主机名：

```
10.10.0.100 controller01.learningneutron.com controller01
10.10.0.101 compute01.learningneutron.com compute01
10.10.0.102 compute02.learningneutron.com compute02
10.10.0.103 compute03.learningneutron.com compute03
10.10.0.104 snat01.learningneutron.com snat01 
```

在每个节点上，使用`hostname -f`命令验证完全限定的主机名是否正确反映：

```
root@controller01:~# hostname -f
controller01.learningneutron.com 
```

# 安装和配置网络时间协议

如网络时间协议 (**NTP**) 等时间同步程序是必需的，因为 OpenStack 服务依赖于主机之间的一致且同步的时间。

对于 Nova 计算，时间同步有助于避免在 `compute` 节点上调度虚拟机启动时出现问题。其他服务在时间未同步时也会遇到类似问题。

要安装 NTP 实现 `chrony`，请在环境中的所有节点上执行以下命令：

```
 # apt install chrony
```

在`controller`节点上，将以下行添加到`/etc/chrony/chrony.conf`文件中，以允许环境中的其他主机与控制器同步时间：

```
allow 10.10.0.0/24 
```

在其他节点上，注释掉`/etc/chrony/chrony.conf`文件中的任何`pool`行，并添加以下行，将它们的时间同步指向控制器：

```
# pool 2.debian.pool.ntp.org offline iburst
server controller01 iburst 
```

在每个主机上，重启 `chrony` 服务：

```
# systemctl restart chrony
```

# 重启系统

在继续安装之前，重启每个主机：

```
# reboot
```

# 安装 OpenStack

本节的步骤记录了在一个控制节点和三个`compute`节点上安装 OpenStack 服务（包括 Keystone、Glance、Nova 和 Horizon）的过程。Neutron，OpenStack 网络服务，将在下一章进行安装。

使用以下命令在`controller`节点上安装 OpenStack 命令行客户端 `python-openstackclient`，为配置各种服务做好准备：

```
 # apt install python-openstackclient
```

# 安装和配置 MySQL 数据库服务器

在`controller`节点上，使用`apt`安装 MySQL 数据库服务和相关的 Python 包：

```
 # apt install mariadb-server python-pymysql
```

如果提示，请将密码设置为`openstack`。

本书中使用了不安全的密码，以简化配置和概念演示，且不推荐用于生产环境。访问[`www.strongpasswordgenerator.org`](http://www.strongpasswordgenerator.org)为您的环境生成强密码。

安装完成后，创建并编辑 ` /etc/mysql/mariadb.conf.d/99-openstack.cnf` 配置文件。添加 `[mysqld]` 块和 `bind-address` 定义。这样可以允许其他主机连接 MySQL。`bind-address` 的值应为 `controller` 节点的管理 IP 地址：

```
[mysqld]
bind-address = 10.10.0.100 
```

除了添加 `bind-address` 定义外，还需要在 `[mysqld]` 部分添加以下选项：

```
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

保存并关闭文件，然后重新启动 `mysql` 服务：

```
    # systemctl restart mysql
```

MySQL 安全安装工具用于创建默认的 MySQL 数据库并为 MySQL root 用户设置密码。以下命令将开始 MySQL 的安装和配置过程：

```
    # mysql_secure_installation
```

在 MySQL 安装过程中，你将被提示输入密码并修改各种设置。默认的 root 密码可能尚未设置。当系统提示时，请设置 root 密码为 `openstack`。建议使用更安全的密码，以适应你的环境。

对其余问题回答 `[Y]es` 以退出配置过程。此时，MySQL 服务器已成功安装在 `controller` 节点上。

# 安装和配置消息服务器

**高级消息队列协议**（**AMQP**）是支撑大多数基于 OpenStack 的云计算平台的消息技术。像 Nova、Cinder 和 Neutron 等组件通过消息总线在内部以及彼此之间进行通信。以下是安装 RabbitMQ（AMQP 中介）的说明。

在 `controller` 节点上，安装消息服务器：

```
 # apt install rabbitmq-server
```

向 RabbitMQ 中添加名为 `openstack` 的用户，并设置密码为 `rabbit`，如以下命令所示：

```
 # rabbitmqctl add_user openstack rabbit 
```

设置 RabbitMQ 权限，以允许 `openstack` 用户进行配置、读取和写入操作：

```
    # rabbitmqctl set_permissions openstack ".*" ".*" ".*"  
```

到此为止，RabbitMQ 的安装和配置已经完成。

# 安装和配置 memcached

`memcached` 服务用于将常见的数据和对象缓存到 RAM 中，从而减少它们从磁盘读取的次数，多个 OpenStack 服务都会使用它。

在 `controller` 节点上，安装 `memcached`：

```
    # apt install memcached python-memcache
```

编辑 `/etc/memcached.conf` 文件，并将默认的监听地址替换为 `controller` 节点的 IP 地址：

+   来源：`127.0.0.1`

+   目标：`10.10.0.100`

重启 `memcached` 服务：

```
    # systemctl restart memcached
```

# 安装和配置身份服务

Keystone 是 OpenStack 的身份服务，用于在 OpenStack 云中对用户和服务进行认证和授权。Keystone 应该只安装在 `controller` 节点上，后续部分将进行介绍。

# 配置数据库

使用 `mysql` 客户端，创建 Keystone 数据库和相关用户：

```
    # mysql
```

在 `MariaDB [(none)] >` 提示符下输入以下 SQL 语句：

```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'keystone';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'keystone';
quit;
```

# 安装 Keystone

从 OpenStack 的 Kilo 版本开始，Keystone 项目使用 Apache HTTP 服务器和 `mod_wsgi` 来处理对身份 API 的请求，监听端口 `5000` 和 `35357`。

在`controller`节点上运行以下命令来安装 Keystone 软件包：

```
    # apt install keystone apache2 libapache2-mod-wsgi
```

更新`/etc/keystone/keystone.conf`文件中的`[database]`部分，将 Keystone 配置为使用 MySQL 作为数据库。在此安装中，用户名和密码为`keystone`。你需要用以下单行值覆盖现有的连接字符串：

```
[database]
...
connection = mysql+pymysql://keystone:keystone@controller01/keystone 
```

# 配置令牌和驱动程序

Keystone 支持可自定义的令牌提供程序，这些提供程序可以在配置文件的`[token]`部分中定义。Keystone 提供 UUID、PKI 和 Fernet 令牌提供程序。在此安装中，将使用 Fernet 令牌提供程序。相应地更新`/etc/keystone/keystone.conf`文件中的`[token]`部分：

```
[token]
...
provider = fernet 
```

使用`keystone-manage`工具填充 Keystone 数据库：

```
    # su -s /bin/sh -c "keystone-manage db_sync" keystone
```

使用以下命令初始化 Fernet 密钥存储库：

```
 # keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone # keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

# 引导身份验证服务

使用`keystone-manage`命令，使用身份验证端点引导服务目录。在此环境中，`admin`用户的密码为`openstack`：

```
 # keystone-manage bootstrap --bootstrap-password openstack      --bootstrap-admin-url http://controller01:35357/v3/ 
      --bootstrap-internal-url http://controller01:5000/v3/
 --bootstrap-public-url http://controller01:5000/v3/      --bootstrap-region-id RegionOne
```

# 配置 Apache HTTP 服务器

使用`sed`命令，将`ServerName`选项添加到 Apache 配置文件中，引用`controller`节点的短名称：

```
 # sed -i '1s/^/ServerName controller01\n&/' /etc/apache2/apache2.conf
```

重启 Apache web 服务以使更改生效：

```
 # systemctl restart apache2
```

# 设置环境变量

为了避免每次运行 OpenStack 命令时都提供凭据，可以创建一个包含环境变量的文件，以便随时加载。以下命令将创建一个名为`adminrc`的文件，其中包含`admin`用户的环境变量：

```
 # cat >> ~/adminrc <<EOF export OS_PROJECT_DOMAIN_NAME=default export OS_USER_DOMAIN_NAME=default export OS_PROJECT_NAME=admin export OS_USERNAME=admin export OS_PASSWORD=openstack export OS_AUTH_URL=http://controller01:35357/v3 export OS_IDENTITY_API_VERSION=3 EOF
```

以下命令将创建一个名为`demorc`的文件，其中包含`demo`用户的环境变量：

```
 # cat >> ~/demorc <<EOF export OS_PROJECT_DOMAIN_NAME=default export OS_USER_DOMAIN_NAME=default export OS_PROJECT_NAME=demo export OS_USERNAME=demo export OS_PASSWORD=demo export OS_AUTH_URL=http://controller01:35357/v3 export OS_IDENTITY_API_VERSION=3 EOF
```

使用`source`命令从文件加载环境变量：

```
    # source ~/adminrc
```

`demo`用户尚不存在，但将在接下来的步骤中创建。

# 在 Keystone 中定义服务和 API 端点

每个安装的 OpenStack 服务都应该注册到身份验证服务，以便跟踪其在网络上的位置。注册服务时涉及两个命令：

+   `openstack service create`：描述正在创建的服务

+   `openstack endpoint create`：将 API 端点与服务关联

在本章之前的引导过程中，已经创建了 OpenStack 身份验证服务和端点。你可以使用`openstack endpoint list`和`openstack service list`命令来验证对象是否已创建，如下所示：

![](img/d7daca5f-6761-4c07-ab18-3ad03b9ce9b4.png)

OpenStack 云中的资源 ID 是唯一的，并且在不同的环境之间会有所不同，因此如果你的 ID 与这里显示的不一致，请不用担心：

![](img/8b9c711f-033a-458b-a1e2-6635b1493603.png)

# 在 Keystone 中定义用户、项目和角色

一旦 Keystone 的安装完成，就需要设置将被各种 OpenStack 服务使用的域、用户、项目、角色和端点。

在此安装过程中，将使用`default`域。

在 Keystone 中，项目（或租户）代表一组逻辑上的用户，资源分配给这些用户。项目和租户在各种 OpenStack 服务中可互换使用，但项目是优选的术语。资源分配给项目，而不是直接分配给用户。在 Keystone 引导过程中创建了`admin`项目、用户和角色。为常规用户创建一个`demo`项目，为其他 OpenStack 服务创建一个`service`项目：

```
    # openstack project create --description "Service Project" service
 # openstack project create --description "Demo Project" demo
```

接下来，创建一个名为`demo`的常规用户。为`demo`用户指定一个安全密码：

```
 # openstack user create demo --password=demo
```

创建`user`角色：

```
 # openstack role create user
```

最后，将`user`角色添加到`demo`用户的`demo`项目中：

```
 # openstack role add --project demo --user demo user
```

# 安装和配置镜像服务

Glance 是 OpenStack 的镜像服务，负责存储实例的镜像和快照，并在创建实例时向`compute`节点提供镜像。

# 配置数据库

使用`mysql`客户端创建 Glance 数据库和关联的用户：

```
 # mysql
```

在`MariaDB [(none)] >`提示符下输入以下 SQL 语句：

```
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'glance';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'glance';
quit; 
```

# 定义 Glance 用户、服务和端点

使用`openstack`客户端创建 Glance 用户：

```
 # openstack user create glance --domain default --password=glance
```

将`admin`角色添加到`service`项目中的`glance`用户：

```
 # openstack role add --project service --user glance admin
```

接下来，创建`glance`服务实体：

```
 # openstack service create --name glance \ --description "OpenStack Image" image
```

最后，创建 Glance 端点：

```
 # openstack endpoint create --region RegionOne \ image public http://controller01:9292 # openstack endpoint create --region RegionOne \ image internal http://controller01:9292 # openstack endpoint create --region RegionOne \ image admin http://controller01:9292
```

# 安装和配置 Glance 组件

要安装 Glance，请在`controller`节点上运行以下命令：

```
 # apt install glance
```

更新位于`/etc/glance/glance-api.conf`的`glance-api`配置文件中的数据库连接字符串，以使用 MySQL Glance 数据库：

```
[database]
...
connection = mysql+pymysql://glance:glance@controller01/glance
```

对位于`/etc/glance/glance-registry.conf`的`glance-registry`配置文件重复该过程：

```
[database]
...
connection = mysql+pymysql://glance:glance@controller01/glance 
```

# 配置身份验证设置

在服务运行之前，必须更新`glance-api`和`glance-registry`服务配置文件中的适当身份验证设置。

更新位于`/etc/glance/glance-api.conf`的`glance-api`配置文件中的`[keystone_authtoken]`设置：

```
[keystone_authtoken]
...
auth_uri = http://controller01:5000
auth_url = http://controller01:35357
memcached_servers = controller01:11211
auth_type = password
user_domain_name = default
project_domain_name = default
project_name = service
username = glance
password = glance 
```

对位于`/etc/glance/glance-registry.conf`的`glance-registry`配置文件重复该过程：

```
[keystone_authtoken]
...
auth_uri = http://controller01:5000
auth_url = http://controller01:35357
memcached_servers = controller01:11211
auth_type = password
user_domain_name = default
project_domain_name = default
project_name = service
username = glance
password = glance
```

# 配置附加设置

更新位于`/etc/glance/glance-api.conf`的`glance-api`配置文件，添加以下设置：

```
[paste_deploy]
...
flavor = keystone

[glance_store]
...
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images  
```

接下来，更新位于`/etc/glance/glance-registry.conf`的`glance-registry`配置文件，添加以下设置：

```
[paste_deploy]
...
flavor = keystone 
```

当两个文件都更新后，使用`glance-manage`工具填充 Glance 数据库：

```
# su -s /bin/sh -c "glance-manage db_sync" glance
```

最后，使用以下命令重启`Glance`服务：

```
# systemctl restart glance-registry glance-api
```

# 验证 Glance 镜像服务的安装情况

运行`adminrc`脚本以设置或更新环境变量：

```
# source ~/adminrc
```

为了验证 Glance 是否正确安装和配置，从互联网下载一个测试镜像，并验证其是否可以上传到镜像服务器：

```
 # mkdir /tmp/images # wget -P /tmp/images http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
```

使用以下命令将镜像上传到 Glance：

```
    # openstack image create "cirros-0.4.0" --file /tmp/images/cirros-0.4.0-x86_64-disk.img --disk-format qcow2
    --container-format bare --public
```

使用`openstack image list`命令验证镜像是否存在于 Glance 中，如下所示：

![](img/d9808b2c-5e0b-48c3-877c-260199e6a987.png)

# 安装额外的镜像

CirrOS 镜像功能有限，仅推荐用于测试网络连接和基础的计算服务。多个供应商提供适用于 OpenStack 的云镜像，包括以下内容：

| **Canonical - Ubuntu** | [`cloud-images.ubuntu.com/`](http://cloud-images.ubuntu.com/) |
| --- | --- |
| **Red Hat - CentOS** | [`cloud.centos.org/centos/`](http://cloud.centos.org/centos/) |

要安装 Ubuntu 16.04 LTS 镜像，将文件下载到`/tmp/images`并上传到 Glance：

```
 # wget -P /tmp/images http://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img
```

使用`openstack image create`命令上传新镜像：

```
    # openstack image create "ubuntu-xenial-16.04" --file /tmp/images/xenial-server-cloudimg-amd64-disk1.img --disk-format qcow2 --container-format bare --public
```

再次查看镜像列表，显示新的 Ubuntu 镜像已准备好使用：

![](img/eea1c42a-47a3-429c-9910-4327fd092934.png)

# 安装和配置计算服务

OpenStack 计算是一个服务集合，允许云操作员和用户启动虚拟机实例。大多数服务在`controller`节点上运行，除`nova-compute`服务外，后者运行在`compute`节点上，负责在这些节点上启动虚拟机实例。

# 配置数据库

在`controller`节点上使用`mysql`客户端，创建 Nova 数据库及相关用户：

```
    # mysql
```

在`MariaDB [(none)] >`提示符下输入以下 SQL 语句：

```
CREATE DATABASE nova;
CREATE DATABASE nova_api;
CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'nova';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'nova';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'nova';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'nova';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'nova';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'nova';
quit;
```

# 定义 Nova 用户、服务和端点

如下所示，源`adminrc`凭证：

```
    # source ~/adminrc
```

使用`openstack`客户端，创建`nova`和`placement`用户：

```
    # openstack user create nova --domain default --password=nova # openstack user create placement 
      --domain default --password=placement
```

将`admin`角色添加到`nova`和`placement`用户的`service`项目中：

```
    # openstack role add --project service --user nova admin # openstack role add --project service --user placement admin
```

接下来，创建`compute`和`placement`服务实体：

```
    # openstack service create --name nova --description "OpenStack Compute" compute 
    # openstack service create --name placement
 --description "Placement API" placement
```

最后，创建`compute`和`placement`端点：

```
    # openstack endpoint create --region RegionOne 
      compute public http://controller01:8774/v2.1
 # openstack endpoint create --region RegionOne      compute internal http://controller01:8774/v2.1
 # openstack endpoint create --region RegionOne      compute admin http://controller01:8774/v2.1
 # openstack endpoint create --region RegionOne      placement public http://controller01:8778
 # openstack endpoint create --region RegionOne      placement internal http://controller01:8778
 # openstack endpoint create --region RegionOne      placement admin http://controller01:8778
```

# 安装和配置控制节点组件

在`controller`节点上执行以下命令，以安装控制节点使用的各种`Compute`服务：

```
    # apt install nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler nova-placement-api
```

使用以下数据库连接字符串，更新位于`/etc/nova/nova.conf`的 Nova 配置文件中的`[database]`和`[api_database]`部分：

```
[database]
...
connection = mysql+pymysql://nova:nova@controller01/nova 
[api_database]
...
connection = mysql+pymysql://nova:nova@controller01/nova_api 
```

更新 Nova 配置文件中的`[DEFAULT]`部分，以配置 Nova 使用 RabbitMQ 消息代理：

```
[DEFAULT]
...
transport_url = rabbit://openstack:rabbit@controller01 
```

VNC 代理是一个 OpenStack 组件，允许用户通过 VNC 客户端访问他们的实例。VNC 代表**虚拟网络计算**，是一种图形化桌面共享系统，使用远程帧缓冲协议通过网络控制另一台计算机。控制节点必须能够与`compute`节点通信，以便通过 Horizon 仪表板或其他 VNC 客户端正常工作 VNC 服务。

更新 Nova 配置文件中的`[vnc]`部分，以为`controller`节点配置适当的 VNC 设置：

```
[vnc]
...
enabled = true
vncserver_listen = 10.10.0.100
vncserver_proxyclient_address = 10.10.0.100 
```

# 配置认证设置

使用以下与 Keystone 相关的属性，更新位于`/etc/nova/nova.conf`的 Nova 配置文件：

```
[api]
...
auth_strategy= keystone

[keystone_authtoken]
...
auth_uri = http://controller01:5000
auth_url = http://controller01:35357
memcached_servers = controller01:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = nova 
```

# 其他控制器任务

更新位于`/etc/nova/nova.conf`的 Nova 配置文件，指定 Glance API 的位置：

```
[glance]
...
api_servers = http://controller01:9292 
```

更新 Nova 配置文件，设置 Nova 服务的锁文件路径并指定`controller`节点的 IP 地址：

```
[oslo_concurrency]
...
lock_path = /var/lib/nova/tmp 
[DEFAULT]
...
my_ip = 10.10.0.100 
```

配置`/etc/nova/nova.conf`中的`[placement]`部分。确保注释掉任何现有的`os_region_name`配置：

```
[placement]
...
os_region_name = RegionOne
auth_url = http://controller01:35357/v3
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = placement
password = placement
```

使用`nova-manage`工具填充`nova-api`数据库：

```
    # su -s /bin/sh -c "nova-manage api_db sync" nova
```

使用`nova-manage`工具注册`cell0`数据库：

```
    # su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
```

创建`cell1`单元：

```
    # su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova 
```

使用`nova-manage`工具填充`nova`数据库：

```
    # su -s /bin/sh -c "nova-manage db sync" nova
```

最后，重启基于控制器的计算服务，使更改生效：

```
    # systemctl restart nova-api nova-consoleauth nova-scheduler nova-conductor nova-novncproxy
```

# 安装并配置计算节点组件

一旦在`controller`节点上配置了`Compute`服务，至少还需要配置一个其他主机作为`compute`节点。`compute`节点接收来自`controller`节点的请求，托管虚拟机实例。

在`compute`节点上，安装`nova-compute`软件包及相关软件包。这些软件包为`compute`节点提供虚拟化支持服务：

```
    # apt install nova-compute
```

在`compute`节点上，使用以下与 Keystone 相关的设置更新位于`/etc/nova/nova.conf`的 Nova 配置文件：

```
[api]
...
auth_strategy= keystone

[keystone_authtoken]
...
auth_uri = http://controller01:5000
auth_url = http://controller01:35357
memcached_servers = controller01:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = nova 
```

接下来，更新 Nova 配置文件的`[DEFAULT]`部分，配置 Nova 使用 RabbitMQ 消息代理：

```
[DEFAULT]
...
transport_url = rabbit://openstack:rabbit@controller01 
```

然后，更新 Nova 配置文件，以通过`controller`节点的代理提供远程控制台访问实例。远程控制台可以通过 Horizon 仪表盘访问。配置为`my_ip`的 IP 应该是每个`compute`节点的管理 IP。

`Compute01`：

```
[DEFAULT]
...
my_ip = 10.10.0.101

[vnc]
...
vncserver_proxyclient_address = 10.10.0.101
enabled = True
vncserver_listen = 0.0.0.0
novncproxy_base_url = http://controller01:6080/vnc_auto.html 
```

`Compute02`：

```
[DEFAULT]
...
my_ip = 10.10.0.102

[vnc]
...
vncserver_proxyclient_address = 10.10.0.102
enabled = True
vncserver_listen = 0.0.0.0
novncproxy_base_url = http://controller01:6080/vnc_auto.html 
```

`Compute03`：

```
[DEFAULT]
...
my_ip = 10.10.0.103

[vnc]
...
vncserver_proxyclient_address = 10.10.0.103
enabled = True
vncserver_listen = 0.0.0.0
novncproxy_base_url = http://controller01:6080/vnc_auto.html 
```

# 其他计算任务

更新位于`/etc/nova/nova.conf`的 Nova 配置文件，指定 Glance API 的位置：

```
[glance]
...
api_servers = http://controller01:9292 
```

更新 Nova 配置文件，设置`Nova`服务的锁文件路径：

```
[oslo_concurrency]
...
lock_path = /var/lib/nova/tmp 
```

配置`/etc/nova/nova.conf`中的`[placement]`部分。确保注释掉任何现有的`os_region_name`配置：

```
[placement]
...
os_region_name = RegionOne
auth_url = http://controller01:35357/v3
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = placement
password = placement 
```

在所有`compute`节点上重启`nova-compute`服务：

```
    # systemctl restart nova-compute
```

# 将计算节点添加到单元数据库

当计算服务首次在`compute`节点上启动时，该节点通过 Nova Placement API 进行注册。要验证`compute`节点是否已注册，可以使用如下的`openstack compute service list`命令：

![](img/92a4b63f-82d2-4095-a36c-382dbab74ddc.png)

# 安装 OpenStack 仪表盘

OpenStack 仪表盘，也称为 Horizon，提供了一个基于 Web 的用户界面，用于访问 OpenStack 服务，包括计算、网络、存储和身份等。

为了将 Horizon 添加到环境中，在`controller`节点上安装以下软件包：

```
    # apt install openstack-dashboard
```

# 更新主机和 API 版本配置

编辑`/etc/openstack-dashboard/local_settings.py`文件，并将`OPENSTACK_HOST`的值从默认值更改为以下内容：

```
OPENSTACK_HOST = "controller01" 
```

使用以下字典设置 API 版本：

```
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
```

# 配置 Keystone 设置

编辑`/etc/openstack-dashboard/local_settings.py`文件，并将以下与 Keystone 相关的配置选项替换为这些值：

```
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST 
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True 
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default" 
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user" 
```

# 修改网络配置

股票 Horizon 配置启用了某些尚未实现的网络功能。因此，您可能会在仪表板中遇到错误，这些错误可能会影响可用性。现在，请通过设置以下配置禁用仪表板中的所有网络功能：

```
OPENSTACK_NEUTRON_NETWORK = {
    'enable_router': False,
    'enable_quotas': False,
    'enable_ipv6': False,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_lb': False,
    'enable_firewall': False,
    'enable_vpn': False,
'enable_fip_topology_check': False,
} 
```

# 卸载默认的 Ubuntu 主题（可选）

默认情况下，Ubuntu 上的 OpenStack Dashboard 安装包含由 Canonical 定制的主题。要移除该主题，请更新`/etc/openstack-dashboard/local_settings.py`文件，并将`DEFAULT_THEME`值从`ubuntu`替换为`default`：

```
DEFAULT_THEME = 'default' 
```

本书中的示例假设已安装默认主题。

# 重新加载 Apache

完成上述更改后，使用以下命令重新加载 Apache Web 服务器配置：

```
    # systemctl reload apache2
```

# 测试与仪表板的连接

在可以访问`controller01`节点管理网络的机器上，打开以下 URL：[`controller01/horizon/.`](http://controller01/horizon/)

我的工作站能够访问 API 网络，并且客户端工作站上的`/etc/hosts`文件已更新，以包含本章前面配置的相同主机名到 IP 的映射。

以下截图演示了成功连接到仪表板。用户名和密码是在本章前面的*定义用户、项目和 Keystone 中的角色*部分中创建的。在此安装中，域名为`Default`，用户名为`admin`，密码为`openstack`。

# 熟悉仪表板

成功登录后，仪表板默认显示身份面板。项目相关任务可以在项目面板中完成，而具有`admin`角色的用户将在管理员面板中找到可以完成管理任务的选项：

![](img/4004c6df-de06-40d9-94c1-bd5b43c59395.png)

通过进入管理员 / 系统面板中的系统信息页面，我们可以看到与本章前面安装的服务相对应的服务列表：

![](img/a6c19155-c30e-4f8c-92db-2fd750a53567.png)

OpenStack Dashboard 允许用户在其拥有权限的任何项目中进行更改。位于 OpenStack 标志右侧的菜单是一个下拉菜单，允许用户选择他们想要操作的项目：

![](img/9832a490-2327-4046-847d-d0e0b528b299.png)

只有用户被分配的项目才会列出。如果用户具有`admin`角色，可以在管理员面板中进行其他云范围的更改。使用 Horizon 仪表板执行网络功能将在后续章节中介绍。

# 总结

到目前为止，OpenStack 身份、镜像、仪表盘和计算服务已成功部署到云的各个节点。此时，环境尚未准备好托管实例，因为 OpenStack 网络服务尚未安装或配置。如果在本章描述的服务安装过程中出现问题，请务必查看 `/var/log/nova/`、`/var/log/glance/`、`/var/log/apache2/` 和 `/var/log/keystone/` 中的日志消息，以帮助故障排除。有关安装过程的更多信息，请随时查看 OpenStack 网站上提供的文档，网址为：[`docs.openstack.org`](http://docs.openstack.org)。

在下一章，我们将开始安装 Neutron 网络服务，并探索关于 OpenStack 网络内部架构的更多信息。

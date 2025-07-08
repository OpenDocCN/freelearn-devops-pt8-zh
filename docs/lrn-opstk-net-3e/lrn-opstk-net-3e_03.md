# 第三章：安装 Neutron

OpenStack Networking，也称为 Neutron，为云用户提供网络基础设施即服务平台。在上一章中，我们安装了一些 OpenStack 的基础服务，包括身份验证、镜像和计算服务。在本章中，我将指导你在上一章安装的 OpenStack 环境上安装 Neutron 网络服务。

需要安装的组件包括以下内容：

+   Neutron API 服务器

+   模块化第 2 层（ML2）插件

+   DHCP 代理

+   元数据代理

到本章结束时，你将对各种 Neutron 插件和代理的功能及操作有基本的了解，并且能够在此基础上构建虚拟交换基础设施。

# Neutron 中的基本网络元素

Neutron 使用对大多数系统和网络管理员来说都很熟悉的元素来构建虚拟网络，包括网络、子网、端口、路由器、负载均衡器等。

使用 Neutron 核心 API 的 2.0 版本，用户可以构建一个由以下实体组成的网络基础：

+   **网络**：网络是一个隔离的第 2 层广播域。通常，网络保留给创建它们的项目，但如果进行相应配置，也可以在项目之间共享。网络是 Neutron API 的核心实体。子网和端口必须始终与网络关联。

+   **子网**：子网是一个 IPv4 或 IPv6 地址块，可以为虚拟机实例分配 IP 地址。每个子网必须有一个 CIDR，并且必须与一个网络相关联。多个子网可以与一个网络关联，并且可以是非连续的。可以为子网设置 DHCP 分配范围，以限制提供给实例的地址。

+   **端口**：Neutron 中的端口是虚拟交换机端口的逻辑表示。虚拟机接口与 Neutron 端口相映射，这些端口定义了分配给接口的 MAC 地址和 IP 地址。Neutron 端口定义存储在 Neutron 数据库中，然后由相应的插件代理使用，以构建和连接虚拟交换基础设施。

云运营商和用户都可以通过创建和配置网络和子网来配置网络拓扑，然后指示 Nova 等服务将虚拟设备连接到这些网络上的端口。用户可以创建多个网络、子网和端口，但受到云管理员设置的每个项目配额定义的限制。

# 通过插件扩展功能

OpenStack Networking 项目提供了由 OpenStack 社区开发和支持的参考插件和驱动程序，同时也支持第三方插件和驱动程序，这些插件和驱动程序扩展了网络功能，并实现了 Neutron API。可以创建使用各种软件和硬件技术的插件和驱动程序，以实现由操作员和用户构建的网络。

Neutron 架构中有两种主要的插件类型：

+   核心插件

+   服务插件

**核心插件**实现了核心的 Neutron API，负责将由网络、端口和子网描述的逻辑网络转换为可以由主机上运行的 L2 代理和 IP 地址管理系统实施的内容。

**服务插件**提供额外的网络服务，如路由、负载均衡、防火墙等。

本书将讨论以下核心插件：

+   模块化第 2 层插件

以下服务插件将在后续章节中介绍：

+   路由器

+   负载均衡器

+   主干

Neutron API 为用户提供一致的体验，尽管选择了不同的网络插件。有关与 Neutron API 交互的更多信息，请访问以下网址：[`developer.openstack.org/api-ref/network/v2/index.html`](https://developer.openstack.org/api-ref/network/v2/index.html)。

# 模块化第 2 层插件

在 OpenStack 的 Havana 版本中引入 **模块化第 2 层（ML2）** 插件之前，Neutron 仅能使用单一的核心插件。这种设计导致了同质化的网络架构，无法扩展。操作员被迫对网络堆栈做出长期决策，这些决策在未来很难改变。另一方面，ML2 插件在设计上是可扩展的，支持异构的网络架构，可以同时利用多种技术。ML2 插件替换了其参考实现中的两个单体插件：Linux 桥接核心插件和 Open vSwitch 核心插件。

# 驱动程序

ML2 插件引入了 TypeDrivers 和 Mechanism 驱动程序的概念，以将网络类型的*实现*与这些类型的网络的*实现机制*分离。

# TypeDrivers

一个 ML2 **TypeDriver** 维护特定类型的网络状态，验证提供者网络属性，并使用提供者属性描述网络段。提供者属性包括网络接口标签、分段 ID 和网络类型。支持的网络类型包括`local`、`flat`、`vlan`、`gre`、`vxlan` 和 `geneve`。下表描述了这些网络类型之间的差异：

| **类型** | **描述** |
| --- | --- |
| 本地 | **本地网络**是指与其他网络和节点隔离的网络。连接到本地网络的实例可以与同一`compute`节点上的其他实例通信，但无法与其他主机上的实例通信。由于这种设计限制，本地网络仅推荐用于测试目的。 |
| 平面 | 在**平面网络**中，不进行 802.1q VLAN 标记或其他网络隔离。在许多环境中，平面网络对应于*接入*VLAN 或*本地*VLAN 上的干道。 |
| VLAN | **VLAN 网络**是利用 802.1q 标记来隔离网络流量的网络。处于同一 VLAN 中的实例被认为是同一网络的一部分，并且处于相同的第二层广播域内。VLAN 间路由或 VLAN 之间的路由只能通过物理或虚拟路由器来实现。 |
| GRE | **GRE 网络**使用**通用路由封装**隧道协议（IP 协议 47）来封装数据包并通过点对点网络在节点之间传输。在 GRE 头部的`KEY`字段用于隔离网络。 |
| VXLAN | **VXLAN 网络**使用一个独特的分段 ID，称为 VXLAN 网络标识符（VNI），来区分来自其他 VXLAN 网络的流量。来自一个实例到另一个实例的流量通过主机使用 VNI 进行封装，并通过现有的第三层网络使用 UDP 传输，在该网络中解封装并转发到目标实例。使用 VXLAN 封装数据包并通过现有网络传输，旨在解决 VLAN 和物理交换基础设施的局限性。 |
| GENEVE | **GENEVE 网络**类似于 VXLAN 网络，它使用一个独特的分段 ID，称为虚拟网络接口（**VNI**），来区分来自其他 GENEVE 网络的流量。数据包被封装在一个独特的头部中，并使用 UDP 作为传输机制。GENEVE 利用了多个覆盖技术的优势，如 VXLAN、NVGRE 和 STT，当前主要由 OVN 使用。 |

# 机制驱动程序

ML2 **机制驱动程序**负责获取由类型驱动程序建立的信息，并确保其正确实现。可以配置多个机制驱动程序同时运行，并可以通过三种类型的模型进行描述：

+   **基于代理的：** 包括 Linux bridge、Open vSwitch、SR-IOV 等

+   **基于控制器的：** 包括 Juniper Contrail、Tungsten Fabric、OVN、Cisco ACI、VMWare NSX 等

+   **机架顶端**：包括 Cisco Nexus、Arista、Mellanox 等

本书中将讨论的机制驱动程序包括以下内容：

+   Linux 桥接

+   Open vSwitch

+   L2 人口

Linux 桥接和 Open vSwitch ML2 Mechanism 驱动程序用于配置在承载实例和网络服务的节点内的虚拟交换技术。Linux 桥接驱动程序支持 `local`、`flat`、`vlan` 和 `vxlan` 网络类型，而 Open vSwitch 驱动程序支持这些网络类型以及 `gre` 网络类型。对于其他类型的驱动程序（如 `geneve`）的支持将根据实现的 Mechanism 驱动程序而有所不同。

L2 population 驱动程序用于限制在使用 VXLAN 网络时转发到覆盖网络中的广播流量数量。在正常情况下，未知的单播、多播和广播流量可能会从所有隧道泛洪到其他 `compute` 节点。这种行为可能对覆盖网络结构产生负面影响，特别是当云中主机数量增加时。

作为云中实例和其他网络资源的权威，Neutron 可以在所有主机上预填充转发数据库，以避免代价高昂的学习操作。ARP 代理是 L2 population 驱动程序的一个特性，允许 Neutron 以类似的方式在所有主机上预填充 ARP 表，以避免 ARP 流量在覆盖网络中广播。

# ML2 架构

下图展示了 Neutron API 服务如何与负责构建虚拟和物理网络的各种插件和代理进行交互：

![](img/74a84179-b5ae-470a-8ae5-e0b84965baf7.png)

图 3.1

上图展示了 Neutron API、Neutron 插件和驱动程序与 L2 和 L3 代理等服务之间的交互。有关 Neutron ML2 插件架构的更多信息，请参考以下网址：[`docs.openstack.org/neutron/pike/admin/config-ml2.html`](https://docs.openstack.org/neutron/pike/admin/config-ml2.html)

# 网络命名空间

OpenStack 设计时考虑到了多租户的需求，并为用户提供了创建和管理自己的计算和网络资源的能力。Neutron 支持每个租户拥有多个私有网络、路由器、防火墙、负载均衡器和其他网络资源，并能够通过使用网络命名空间将这些对象隔离开来。

**网络命名空间** 被定义为网络堆栈的逻辑副本，具有自己的路由、Firewall 规则和网络接口。在使用开源参考插件和驱动程序时，由用户创建的每个 DHCP 服务器、路由器和负载均衡器都会在网络命名空间中实现。通过使用网络命名空间，Neutron 能够为每个网络提供隔离的 DHCP 和路由服务，使用户能够在其他项目的用户甚至同一项目的其他网络中创建重叠的网络。

以下命名规则应适用于网络命名空间：

+   **DHCP 命名空间：**`qdhcp-<网络 UUID>`

+   **路由器命名空间：**`qrouter-<路由器 UUID>`

+   **负载均衡器命名空间：**`qlbaas-<负载均衡器 UUID>`

一个`qdhcp`命名空间包含一个 DHCP 服务，用于通过 DHCP 协议为实例提供 IP 地址。在参考实现中，`dnsmasq`是处理 DHCP 请求的进程。`qdhcp`命名空间有一个接口连接到虚拟交换机，并能够与相同网络中的实例和其他设备进行通信。对于启用了 DHCP 的相关子网，每个网络都会创建一个`qdhcp`命名空间。

一个`qrouter`命名空间代表一个虚拟路由器，负责将流量路由到连接的子网中的实例，或从实例路由出去。像`qdhcp`命名空间一样，`qrouter`命名空间根据配置连接到一个或多个虚拟交换机。在某些情况下，可能会使用多个命名空间来建立虚拟路由器基础设施。这些额外的命名空间，称为`fip`和`snat`，用于分布式虚拟路由器（DVR），将在本书后面讨论。

一个`qlbaas`命名空间代表一个虚拟负载均衡器，并包含一个服务，如 HAProxy，用于将流量负载均衡到实例。`qlbaas`命名空间连接到虚拟交换机，并可以与相同网络中的实例和其他设备进行通信。

有趣的事实：网络命名空间名称中的前导`q`代表 Quantum，这是 OpenStack Networking 服务的原始名称。

仅在运行 Neutron DHCP、L3 或 LBaaS 代理的节点上，才能看到上述类型的网络命名空间。这些服务通常仅在控制器或专用网络节点上配置。当配置分布式虚拟路由器时，您可能还会在`compute`节点上找到与路由器相关的命名空间。可以使用`ip netns list`命令列出可用的命名空间，并使用以下语法在命名空间内执行命令：

```
ip netns exec NAMESPACE_NAME <command> 
```

可以在命名空间中执行的命令包括`ip`、`route`、`iptables`等。这些命令的输出对应于在它们执行的命名空间中专用的数据。像`tcpdump`这样的工具也可以在网络命名空间中执行，以帮助排查虚拟网络基础设施的问题。

有关网络命名空间的更多信息，请参阅`ip netns`的手册页，网址为：[`man7.org/linux/man-pages/man8/ip-netns.8.html`](http://man7.org/linux/man-pages/man8/ip-netns.8.html)。

# 安装和配置 Neutron 服务

在此安装过程中，构成 OpenStack Networking 的各种服务将安装在`controller`节点上，而不是专用的网络节点。`compute`节点将运行 L2 代理，与`controller`节点进行接口，并为实例提供虚拟交换机连接。

请记住，这里推荐的配置设置以及在[docs.openstack.org](http://docs.openstack.org)上的在线设置，可能不适合生产环境。

# 创建 Neutron 数据库

在 `controller` 节点上使用 `mysql` 客户端创建 Neutron 数据库和相关用户：

```
    # mysql
```

在 `MariaDB [(none)] >` 提示符下输入以下 SQL 语句：

```
CREATE DATABASE neutron; 
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'neutron';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'neutron'; 
quit; 
```

# 在 Keystone 中配置 Neutron 用户、角色和端点

为了正常运行，Neutron 需要在 Keystone 中创建一个用户、角色和端点。当从 `controller` 节点执行时，以下命令将在 Keystone 中创建一个名为 `neutron` 的用户，将 `admin` 角色与 `neutron` 用户关联，并将 `neutron` 用户添加到 `service` 项目中：

```
 # source ~/adminrc # openstack user create --domain Default --password=neutron neutron # openstack role add --project service --user neutron admin
```

在 `controller` 节点上执行以下命令，在 Keystone 中创建一个描述 OpenStack 网络服务的服务：

```
    # openstack service create --name neutron --description "OpenStack Networking" network
```

要创建端点，请使用以下 `openstack endpoint create` 命令：

```
 # openstack endpoint create --region RegionOne network public http://controller01:9696 # openstack endpoint create --region RegionOne network internal http://controller01:9696 # openstack endpoint create --region RegionOne network admin http://controller01:9696
```

# 安装 Neutron 包

要在控制节点上安装 Neutron API 服务器、DHCP 和元数据代理以及 ML2 插件，请执行以下命令：

```
    # apt install neutron-server neutron-dhcp-agent 
neutron-metadata-agent neutron-plugin-ml2 python-neutronclient
```

Neutron 的 DHCP 和元数据代理可能并非所有机制驱动程序都需要，但在实现 `openvswitch` 和 `linuxbridge` 驱动程序时会使用它们。

在所有其他主机上，此时仅需要 ML2 插件：

```
 # apt install neutron-plugin-ml2
```

在所有节点上，更新 `/etc/neutron/neutron.conf` 中 Neutron 配置文件的 `[database]` 部分，使用基于前面值的正确 MySQL 数据库连接字符串，而不是默认值：

```
[database] 
... 
connection = mysql+pymysql://neutron:neutron@controller01/neutron 
```

# 配置 Neutron 使用 Keystone

Neutron 配置文件位于 `/etc/neutron/neutron.conf`，其中有数十个可以修改的设置，以满足 OpenStack 云管理员的需求。部分设置必须在此安装过程中从默认值更改。

要指定 Keystone 作为 Neutron 的身份验证方法，请在所有主机上更新 Neutron 配置文件的 `[DEFAULT]` 部分，使用以下设置：

```
[DEFAULT] 
... 
auth_strategy = keystone 
```

Neutron 还必须配置适当的 Keystone 身份验证设置。`neutron` 用户在 Keystone 中的用户名和密码在本章前面已经设置。在所有主机上更新 Neutron 配置文件的 `[keystone_authtoken]` 部分，使用以下设置：

```
[keystone_authtoken] 
... 
auth_uri = http://controller01:5000 
auth_url = http://controller01:35357 
memcached_servers = controller01:11211 
auth_type = password 
project_domain_name = default 
```

```
user_domain_name = default 
project_name = service 
username = neutron 
password = neutron 
```

# 配置 Neutron 使用消息服务

Neutron 与各种 OpenStack 服务通过 AMQP 消息总线进行通信。更新所有主机上 Neutron 配置文件的 `[DEFAULT]` 部分，以指定 RabbitMQ 作为消息代理：

```
[DEFAULT] 
... 
transport_url = rabbit://openstack:rabbit@controller01 
```

# 配置 Nova 使用 Neutron 网络

在 Neutron 作为 OpenStack 计算服务的网络管理器之前，必须在某些主机上设置 Nova 配置文件 `/etc/nova/nova.conf` 中的适当配置选项。

在控制节点和 `compute` 节点上，使用以下内容更新 `[neutron]` 部分：

```
[neutron]  
... 
url= http://controller01:9696
auth_url = http://controller01:35357
auth_type = password 
project_domain_name = default 
user_domain_name = default 
region_name = RegionOne 
project_name = service 
username = neutron 
password = neutron 
```

一旦确定了机制驱动程序，Nova 可能需要额外的配置。Linux bridge 和 Open vSwitch 机制驱动程序及其各自的代理和 Nova 配置更改将在后续章节中详细讨论。

# 配置 Neutron 以通知 Nova

Neutron 必须配置为通知 Nova 网络拓扑的变化。在`controller`节点上，更新位于`/etc/neutron/neutron.conf`的 Neutron 配置文件中的`[nova]`部分，设置如下：

```
[nova] 
... 
auth_url = http://controller01:35357 
auth_type = password 
project_domain_name = default 
user_domain_name = default 
region_name = RegionOne 
project_name = service 
username = nova 
password = nova 
```

# 配置 Neutron 服务

`neutron-server`服务将 Neutron API 暴露给用户，并将所有调用传递给配置的 Neutron 插件进行处理。默认情况下，Neutron 配置为监听所有配置的地址上的 API 调用，如 Neutron 配置文件中的默认`bind_hosts`选项所示：

```
bind_host = 0.0.0.0 
```

作为额外的安全措施，可以将 API 暴露在管理或 API 网络上。要更改默认值，请更新位于`/etc/neutron/neutron.conf`的 Neutron 配置文件中`[DEFAULT]`部分的`bind_host`值，并指定`controller`节点的管理地址。本书中解释的部署将保持默认值。

可能需要调整的其他配置选项包括：

+   `core_plugin`

+   `service_plugins`

+   `dhcp_lease_duration`

+   `dns_domain`

其中一些设置适用于所有节点，而另一些仅适用于`network`或`controller`节点。`core_plugin`配置选项指示 Neutron 使用指定的网络插件。从 Icehouse 版本开始，ML2 插件取代了 Linux bridge 和 Open vSwitch 单体插件。

在所有节点上，更新位于`/etc/neutron/neutron.conf`的 Neutron 配置文件中`[DEFAULT]`部分的`core_plugin`值，并指定 ML2 插件：

```
[DEFAULT] 
... 
core_plugin = ml2 
```

`service_plugins`配置选项用于定义 Neutron 加载的插件以实现额外功能。插件的示例包括`router`、`firewall`、`lbaas`、`vpnaas`和`metering`。此选项应仅在`controller`节点或运行`neutron-server`服务的其他节点上配置。服务插件将在后续章节中定义。

`dhcp_lease_duration`配置选项指定实例的 IP 地址租期持续时间。默认值为 86,400 秒，即 24 小时。如果设置的值过低，网络可能会因为短期租期和频繁的续租尝试而遭到流量淹没。实例本身的 DHCP 客户端负责续租，且这一操作的频率因操作系统而异。实例尝试在租期到期前很久就进行续租并不少见。然而，`dhcp_lease_duration`的设置并不会决定 IP 地址与实例关联的时间长度。一旦 Neutron 为端口分配了 IP 地址，直到端口或相关实例被删除，该 IP 地址将保持与端口关联。

`dns_domain`配置选项指定通过 DHCP 向实例提供的 DNS 搜索域，当它们获取租约时。默认值为`openstacklocal`。可以更改为适合您组织的任何值。对于本次安装，将值从`openstacklocal`更改为`learningneutron.com.` 在`controller`节点上，更新位于`/etc/neutron/neutron.conf`的 Neutron 配置文件中的`dns_domain`选项为`learningneutron.com`：

```
[DEFAULT] 
... 
dns_domain = learningneutron.com
```

当实例从 DHCP 服务器获取地址时，域名会附加到主机名，形成完全合格的域名。Neutron 默认不支持多个域名，而是依赖于名为 Designate 的项目来扩展对该功能的支持。有关 Designate 的更多信息，请访问以下 URL：[`docs.openstack.org/designate/latest/`](https://docs.openstack.org/designate/latest/)。

# 启动 neutron-server

在启动`neutron-server`服务之前，必须根据本章之前配置的选项更新 Neutron 数据库。在`controller`节点上使用`neutron-db-manage`命令相应地更新数据库：

```
    # su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

重启`controller`节点上的 Nova 计算服务：

```
    # systemctl restart nova-api nova-scheduler nova-conductor
```

重启`compute`节点上的 Nova 计算服务：

```
    # systemctl restart nova-compute
```

最后，重启`controller`节点上的`neutron-server`服务：

```
    # systemctl restart neutron-server
```

# 配置 Neutron DHCP 代理

Neutron 利用`dnsmasq`，一个免费的轻量级 DNS 转发器和 DHCP 服务器，为网络提供 DHCP 服务。`neutron-dhcp-agent`服务负责为每个使用 DHCP 的网络启动和配置`dnsmasq`以及元数据进程。

DHCP 驱动程序在`/etc/neutron/dhcp_agent.ini`配置文件中指定。可以配置 DHCP 代理使用其他驱动程序，但内置了对`dnsmasq`的支持，并且无需额外设置。默认的`dhcp_driver`值为`neutron.agent.linux.dhcp.Dnsmasq`，可以保持不变。

在`dhcp_agent.ini`配置文件中，还可以找到其他一些显著的配置选项：

+   `interface_driver`

+   `enable_isolated_metadata`

`interface_driver`配置选项应根据您环境中选择的二层代理进行适当配置：

+   **Linux 桥接**: `neutron.agent.linux.interface.BridgeInterfaceDriver`

+   **Open vSwitch**: `neutron.agent.linux.interface.OVSInterfaceDriver`

Linux 桥接和 Open vSwitch 驱动程序将在接下来的章节中进一步讨论。目前，默认值`<none>`已足够。

每个代理每次只能配置一个接口驱动程序。

`enable_isolated_metadata`配置选项在物理网络设备（如防火墙或路由器）作为实例的默认网关，但仍需要 Neutron 为这些实例提供元数据服务的情况下非常有用。当使用 L3 代理时，实例通过作为其默认网关的 Neutron 路由器访问元数据服务。孤立网络假定为 Neutron 路由器不作为网关的网络，但 Neutron 仍处理实例的 DHCP 请求的情况。当实例利用带有物理网关设备的扁平或 VLAN 网络时，通常出现这种情况。`enable_isolated_metadata`的默认值为`False`。当设置为`True`时，Neutron 可以通过 DHCP 在某些情况下为实例提供到元数据服务的静态路由。有关元数据的使用和此配置的更多信息，请参阅*第七章*，*将实例附加到网络*。在`controller`节点上，更新位于`/etc/neutron/dhcp_agent.ini`的 DHCP 代理配置文件中的`enable_isolated_metadata`选项为`True`：

```
[DEFAULT] 
... 
enable_isolated_metadata = True 
```

此处未提及的配置选项具有充分的默认值，除非您的环境需要，否则不应更改。

# 重新启动 Neutron DHCP 代理

使用以下命令在`controller`节点上重新启动`neutron-dhcp-agent`服务并检查其状态：

```
    # systemctl restart neutron-dhcp-agent # systemctl status neutron-dhcp-agent
```

输出应如下所示：

![](img/f7befff5-2ea0-46bb-9bb3-c772d28b3334.png)

代理应处于`active (running)`状态。使用`openstack network agent list`命令验证服务是否已注册：

![](img/a9aea419-11a2-4f30-a2d4-37228ee90a70.png)

在`Alive`列下的笑脸表示代理与`neutron-server`服务正常通信。

# 配置 Neutron 元数据代理

OpenStack Compute 提供了一个元数据服务，允许用户检索有关其实例的信息，这些信息可用于配置或管理运行中的实例。**元数据**包括主机名、固定和浮动 IP、公钥等信息。除元数据外，用户还可以访问**用户数据**，如脚本和其他引导配置，这些配置可以在引导过程中或实例激活后执行。OpenStack Networking 实现了一个代理，将实例的元数据请求转发到由 OpenStack Compute 提供的元数据服务。

实例通常在引导过程中通过 HTTP 访问元数据服务，地址为`http://169.254.169.254`。此机制由`cloud-init`提供，在大多数云准备的映像中可用，网址为：[`launchpad.net/cloud-init`](https://launchpad.net/cloud-init)。

下图概述了当`controller`节点托管网络服务时，从实例检索元数据的高级概述：

![](img/6851a731-a7a9-44ae-81c5-ac8cc919fed7.png)

在上面的示意图中，当实例向元数据服务发出请求时，会发生以下操作：

+   实例通过 HTTP 向 `169.254.269.254` 发送元数据请求

+   元数据请求根据实例中的路由，访问路由器或 DHCP 命名空间

+   命名空间中的元数据代理服务通过 Unix 套接字将请求发送给 Neutron 元数据代理服务

+   Neutron 元数据代理服务将请求转发给 Nova 元数据 API 服务

+   Nova 元数据 API 服务响应请求并将响应转发给 Neutron 元数据代理服务

+   Neutron 元数据代理服务将响应发送回命名空间中的元数据代理服务

+   元数据代理服务将 HTTP 响应转发给实例

+   实例接收元数据和/或用户数据，并继续启动过程

为确保元数据服务的正常运行，Neutron 和 Nova 必须配置为使用共享密钥进行通信。Neutron 使用此密钥对元数据请求的 `Instance-ID` 标头进行签名，以防止欺骗。在 `controller` 节点上，更新位于 `/etc/nova/nova.conf` 的 Nova 配置文件中的 `[neutron]` 部分，以添加以下元数据选项：

```
[neutron] 
... 
service_metadata_proxy = true 
metadata_proxy_shared_secret = MetadataSecret123 
```

接下来，更新位于 `/etc/neutron/metadata_agent.ini` 的元数据代理配置文件中的 `[DEFAULT]` 部分，添加 Neutron 身份验证详细信息和元数据代理共享密钥：

```
[DEFAULT] 
... 
nova_metadata_host = controller01 
metadata_proxy_shared_secret = MetadataSecret123 
```

此处未提及的配置选项具有足够的默认值，除非您的环境需要，否则不应更改。

# 重启 Neutron 元数据代理

使用以下命令重新启动 `controller` 节点上的 `neutron-metadata-agent` 和 `nova-api` 服务，并检查服务的状态：

```
 # systemctl restart nova-api neutron-metadata-agent # systemctl status neutron-metadata-agent
```

输出应类似于以下内容：

![](img/1c5649e2-d2d1-451c-bf78-f570601b3cde.png)

代理应处于 `active (running)` 状态。使用 `openstack network agent list` 命令验证服务是否已检查：

![](img/a2248299-e4a2-4223-8230-ebdbe09bde0e.png)

在 `Alive` 列下的笑脸符号表示代理正在与 `neutron-server` 服务正常通信。

如果服务未显示或在 `Alive` 列下显示 `XXX`，请检查位于 `/var/log/neutron` 的相关日志文件，以帮助故障排除。有关元数据使用的更多信息，请参阅 *第七章*、*将实例连接到网络* 以及后续章节。

# 与 OpenStack 网络接口

可以通过多种方式访问 OpenStack 网络 API，包括通过 Horizon 控制面板、`openstack` 和 `neutron` 客户端、Python SDK、HTTP 以及其他方法。接下来的几个部分将重点介绍与 OpenStack 网络接口的最常见方式。

# 使用 OpenStack 命令行接口

在 `openstack` 命令行客户端问世之前，每个项目负责维护自己的客户端。每个客户端通常使用自己独特的语法来管理对象，并且不同客户端之间缺乏一致性，给用户和运维人员带来了很多困难。`openstack` 客户端为命令和参数提供了一致的命名结构，以及一个一致的输出格式，支持如 csv、json 等可选的可解析格式。然而，并非所有 API 和服务都由 `openstack` 客户端支持，这可能意味着某些操作需要项目特定的客户端。

要调用 `openstack` 客户端，请在 Linux 命令行中输入 `openstack` 命令：

![](img/1bf6ed78-d603-4915-9ff0-7a17577e9ea6.png)

`openstack` shell 提供了可以用来创建、读取、更新和删除 OpenStack 云中网络配置的命令。通过在 `openstack` shell 中输入问号或 `help`，可以找到命令列表。此外，从 Linux 命令行运行 `openstack help` 也会提供每个命令功能的简要描述。

# 使用 Neutron 命令行界面

Neutron 提供了一个命令行客户端，用于与其 API 进行交互。Neutron 命令可以直接在 Linux 命令行中运行，或者可以通过输入 `neutron` 命令来启动 Neutron shell：

![](img/a551687e-ada2-47b4-905f-bb1d10a63d8a.png)

在调用 `openstack` 和 `neutron` 客户端之前，必须先加载凭证文件，否则会发生错误。

`neutron` shell 提供了可以用来创建、读取、更新和删除 OpenStack 云中网络配置的命令。通过在 Neutron shell 中输入问号或 `help`，可以找到命令列表。此外，从 Linux 命令行运行 `neutron help` 也会提供每个命令功能的简要描述。

`neutron` 客户端已被弃用，推荐使用 `openstack` 命令行客户端。然而，某些功能，包括与 LBaaS 相关的命令，在 `openstack` 客户端中尚不可用，必须使用 `neutron` 客户端进行管理。在未来版本的 OpenStack 中，`neutron` 客户端将不再可用。

客户端 `help` 列表中列出的许多命令将在本书的后续章节中介绍。

# 使用 OpenStack Python SDK

OpenStack Python SDK 可以在 PyPI 上找到，并可以通过以下命令进行安装：

```
    $ pip install openstacksdk
```

SDK 的文档可以通过以下网址访问：[`developer.openstack.org/sdks/python/openstacksdk/users/index.html`](https://developer.openstack.org/sdks/python/openstacksdk/users/index.html)。

# 使用 cURL 工具

OpenStack 网络 API 是基于 REST 的，可以通过 HTTP 直接进行操作。要使用 HTTP 调用 API，你需要一个令牌。加载 OpenStack 凭证文件，并使用下面显示的 `openstack token issue` 命令来获取令牌：

![](img/79cf5e5e-67d9-4d4a-a92f-e6f8d7bdab77.png)

`--fit-width` 参数在正常操作中不是必需的，但为了演示的方便，它有助于使令牌 ID 更易于管理。

要获取网络列表，命令应类似于以下内容：

```
    $ curl -v -X GET -H 'X-Auth-Token: <token id>' http://controller01:9696/v2.0/networks
```

输出将类似于以下内容：

![](img/31aba40b-482e-4d7a-bd35-62fa9adcdcbc.png)

在这个例子中，Neutron API 返回了一个 200 OK 的响应，格式为 json。目前没有网络，因此返回了一个空列表。Neutron 返回的 HTTP 状态码可以用来判断命令是否成功。

OpenStack 网络 API 文档可以通过以下网址查看：[`developer.openstack.org/api-ref/network/v2/`](https://developer.openstack.org/api-ref/network/v2/)。

# 摘要

OpenStack 网络提供了一个可扩展的插件架构，使得实现新的网络功能成为可能。Neutron 在其数据库中维护逻辑网络架构，而每个节点上的网络插件和代理负责相应地配置虚拟和物理网络设备。通过使用模块化第二层（ML2）插件，开发人员可以减少实现核心 Neutron API 功能的时间，更多地专注于开发增值功能。

现在，OpenStack 网络服务已在环境中的所有节点上安装完毕，配置机制驱动程序是创建实例之前唯一需要完成的工作。在接下来的两章中，您将学习如何配置 ML2 插件以及 Linux bridge 和 Open vSwitch 驱动程序及代理。我们还将探讨 Linux bridge 和 Open vSwitch 代理在功能和为实例提供连接方面的差异。

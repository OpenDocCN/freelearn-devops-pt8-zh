# 前言

OpenStack 是用于构建公共和私有云以及私有托管软件定义基础设施服务的开源软件。2017 年秋季，OpenStack 基金会发布了第 16 个版本的 OpenStack，称为 Pike。自从 2010 年由 NASA 和 Rackspace 作为开源项目推出以来，OpenStack 在全球开发者和运营商的努力下，功能和特性有了显著提升。正是他们的辛勤工作，造就了适合生产使用的云软件，支撑着全球各地各种规模的工作负载。

2012 年，OpenStack 的 Folsom 版本引入了一个独立的网络组件，当时被称为 Quantum。现在已更名为 Neutron，OpenStack 的网络组件为云运营商和用户提供了一个 API，用于创建和管理云中的网络资源。Neutron 的可扩展框架允许部署和管理第三方插件及附加的网络服务，如负载均衡器、防火墙和虚拟专用网络等。

作为自 2012 年以来管理数百个基于 OpenStack 的私有云的架构师和运营商，我见证了 OpenStack 在网络能力方面的诸多发展。在本书中，我总结了我认为目前最有价值且适用于生产环境的功能。在全书中，我们将探讨几种常见的网络和服务架构，并为部署和管理 OpenStack 网络奠定基础，这将帮助你作为 OpenStack 云运营商，发展和提升自己的技能。

# 本书适用人群

本书面向的是具有初级到中级经验的 OpenStack 云管理员或运营商，特别是那些希望通过使用 Neutron 网络服务来构建或增强云环境的读者。通过基于 [docs.openstack.org](http://docs.openstack.org) 上的上游文档进行 OpenStack 基础安装，读者应能够按照书中的示例获得对 OpenStack 网络各组件的功能性理解，使用开源参考架构进行部署。

# 本书涵盖的内容

*第一章**，* *OpenStack 网络介绍*，介绍了 OpenStack 网络以及支持的网络技术，并举例说明如何设计物理网络以支持 OpenStack 云。

*第二章**，安装 OpenStack*，提供了在 Ubuntu 16.04 LTS 操作系统上安装 Pike 版本 OpenStack 核心组件的指南，包括 Keystone、Glance、Nova 和 Horizon。

*第三章**, 安装 Neutron*，解释了如何安装 OpenStack 的 Neutron 网络组件。我们还将介绍 Neutron 的内部架构，包括使用代理和插件来协调网络连接。

*第四章**, 使用 Linux 桥接构建虚拟网络基础设施*，帮助你安装并配置 ML2 插件，以支持 Linux 桥接机制驱动程序和代理，并展示如何使用 Linux 桥接将实例连接到网络。

*第五章, 使用 Open vSwitch 构建虚拟交换基础设施*，帮助你安装并配置 ML2 插件，以支持 Open vSwitch 机制驱动程序和代理，并展示如何使用 Open vSwitch 将实例连接到网络。

*第六章**, 使用 Neutron 构建网络*，带你了解如何创建网络、子网、子网池和端口。

*第七章**, 将实例连接到网络*，展示了如何将实例连接到网络，并探讨了获取 DHCP 租约和元数据的过程。

*第八章**, 管理安全组*，讲解了如何使用 iptables 来确保计算节点的实例流量安全，并带你了解如何创建和管理安全组及相关规则。

*第九章**, 基于角色的访问控制*，解释了访问控制策略如何将某些网络资源的使用限制在特定项目组之间。

*第十章**, 使用 Neutron 创建独立路由器*，带你学习如何创建独立的虚拟路由器并将其连接到网络，为实例应用浮动 IP，并追踪流量通过路由器到达实例的过程。

*第十一章**, 使用 VRRP 实现路由器冗余*，探讨了虚拟路由冗余协议及其在提供高可用虚拟路由器中的应用。

*第十二章**, 分布式虚拟路由器*，带你了解如何创建和管理分布在计算节点上的虚拟路由器，以便实现更好的扩展性。

*第十三章**, 向实例分配负载均衡流量*，探讨了 Neutron 负载均衡器的基本组成部分，包括监听器、池、池成员和监视器，并带你学习如何创建和集成虚拟负载均衡器到网络中。

*第十四章**，高级网络主题*，探讨了其他高级网络功能，包括允许虚拟机实例为流量应用 802.1q VLAN 标签的 VLAN 感知虚拟机功能，提供动态路由到项目路由器的 BGP Speaker 功能，以及可以用于将关键网络组件（如 DHCP 和 L3 代理）分隔到不同区域的网络可用性区域功能。

# 为了最大限度地发挥本书的价值

本书假设读者具有中等水平的网络经验，包括 Linux 网络配置经验以及物理交换机和路由器配置经验。尽管本书会引导读者完成 OpenStack 的基本安装，但除了 Neutron 服务外，其他服务的讨论较少。因此，读者在配置 OpenStack 网络之前，应该对 OpenStack 及其基本配置有一定的了解。

本书需要以下操作系统：

+   Ubuntu 16.04 LTS

需要以下软件：

+   OpenStack Pike (2017.2)

安装 OpenStack 软件包并使用本书中的示例架构需要互联网连接。虽然可以使用 VirtualBox 或 VMware 等虚拟化软件模拟服务器和网络基础设施，但本书假设 OpenStack 安装在物理硬件上，并且已经搭建了物理网络基础设施。

如果本书中描述的 OpenStack 安装程序不再适用，请参考[docs.openstack.org](http://docs.openstack.org)上的安装指南，获取安装最新版本 OpenStack 的说明。

# 下载示例代码文件

你可以从你在[www.packtpub.com](http://www.packtpub.com)的账户下载此书的示例代码文件。如果你在其他地方购买了此书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便直接将文件通过邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  使用你的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在页面顶部的 SUPPORT 标签上。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称。

1.  选择你想要下载代码文件的书籍。

1.  从下拉菜单中选择你购买此书的来源。

1.  点击“代码下载”。

下载文件后，请确保使用以下最新版本的解压工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，链接为[`github.com/PacktPublishing/Learning-OpenStack-Networking-Third-Edition`](https://github.com/PacktPublishing/Learning-OpenStack-Networking-Third-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有来自我们丰富书籍和视频目录中的其他代码包，可以在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**查看。快来看看吧！

# 下载彩色图像

我们还提供了一份 PDF 文件，其中包含本书中使用的截图/图表的彩色图像。你可以在此下载：[`www.packtpub.com/sites/default/files/downloads/LearningOpenStackNetworkingThirdEdition.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearningOpenStackNetworkingThirdEdition.pdf)[.](https://www.packtpub.com/sites/default/files/downloads/LearningOpenStackNetworkingThirdEdition.pdf)

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：文本中的代码字、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名像这样显示：“`/etc/openstack-dashboard/local_settings.py` 文件中的`OPENSTACK_KEYSTONE_DEFAULT_ROLE`设置也必须修改，才能使用仪表盘。”

代码块的格式如下：

```
[DEFAULT]
...
my_ip = 10.254.254.101
vncserver_proxyclient_address = 10.254.254.101
vnc_enabled = True
vncserver_listen = 0.0.0.0
novncproxy_base_url = http://controller01:6080/vnc_auto.html
```

当我们希望引起你对代码块中特定部分的注意时，相关行或项目将用粗体显示：

```
nova boot --flavor <FLAVOR_ID> --image <IMAGE_ID> \
 --nic net-id=<NETWORK_ID>--security-group <SECURITY_GROUP_ID> \ INSTANCE_NAME
```

任何命令行输入或输出都按如下格式书写：

```
# systemctl restart nova-api
# systemctl restart neutron-server
```

**粗体**：新术语和重要单词用粗体显示。屏幕上看到的词语，例如菜单或对话框中的内容，像这样出现在文本中：“点击‘下一步’按钮会将你带到下一个屏幕。”

警告或重要提示像这样显示。

提示和技巧像这样显示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：我们始终欢迎读者的反馈。让我们知道你对这本书的看法——你喜欢或不喜欢什么。读者反馈对我们非常重要，因为它帮助我们开发出你能真正受益的书籍。

要发送一般反馈，只需通过电子邮件联系我们：feedback@packtpub.com，并在邮件主题中提及书名。

如果你在某个领域有专业知识，并且有兴趣撰写或为书籍做贡献，请参见我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

**勘误**：尽管我们已尽一切努力确保内容的准确性，但错误还是难以避免。如果您在我们的书籍中发现错误——可能是文本或代码错误——我们将非常感激您能向我们报告。通过这样做，您可以帮助其他读者避免困扰，并帮助我们改进后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书籍，点击勘误提交表单链接，并输入勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，勘误内容将上传到我们的网站或加入到该书籍的勘误清单中。

要查看以前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)并在搜索框中输入书名。相关信息将出现在勘误部分。

**盗版**：互联网中的版权材料盗版问题在所有媒体中都普遍存在。我们在 Packt 非常重视版权和许可证的保护。如果您在互联网上发现我们作品的非法复制品，无论形式如何，请立即提供相关的地址或网站名称，以便我们采取进一步行动。

请通过版权@packtpub.com 与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们的作者以及我们为您提供有价值内容的能力方面的帮助。

**如果您有兴趣成为作者**：如果您在某个领域具有专业知识，并且有兴趣撰写或参与撰写书籍，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。当您阅读并使用本书后，为什么不在您购买它的网站上留下评论呢？潜在的读者可以看到并使用您的公正意见来做出购买决策，Packt 可以了解您对我们产品的看法，我们的作者也能看到您对他们书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问[packtpub.com](https://www.packtpub.com/)。

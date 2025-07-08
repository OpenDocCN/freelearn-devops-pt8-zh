# 前言

欢迎阅读*掌握 OpenStack*，这是*Mastering*系列的第三版，将为您提供关于基于 OpenStack 部署和运行私有云基础设施的最新更新和前沿技术。由于 OpenStack 生态系统的复杂性以及不同版本之间的快速发展，组织在采用 OpenStack 时常常面临挑战。本书将通过对每个核心组件采取迭代方式，帮助您克服这些挑战，为生产就绪的私有云设计做好准备。在每一章中，本书从最常用的最佳实践中汲取灵感，用于部署和管理基础设施。这包括利用基础设施即代码、CI/CD 以及 DevSecOps 范式。由于本书将涉及新的 OpenStack 服务和高级概念，自动化将成为您成功部署它们的*守护天使*。本书同样适用于新的 OpenStack 用户，因为它会逐步介绍核心组件，并通过容器部署它们。虽然不要求具备深入的容器化技术知识，但建议有基本了解。最重要的是，本书专注于生产质量和受多行业启发的最流行部署。OpenStack 世界提供了大量的服务和部署选项。本书旨在帮助您理解如何从头开始构建和设计一个符合组织需求的私有云架构。在本版的最后，本书将揭示云市场中的一项新趋势：混合云。OpenStack 扮演着私有云的角色，并与 AWS 环境结合，构建混合云架构。在本书的结尾，将探索更多机会，展示如何在不同云服务提供商之间运行大规模环境，其中 OpenStack 作为私有云解决方案非常合适。

# 本书的读者

本书面向具备 OpenStack 生态系统基本理解以及云计算和虚拟化概念的云架构师和管理员。本书对拥有公共云工作经验的企业顾问和云开发人员也非常有用。建议读者熟悉 Linux 命令行并具备容器化和网络拓扑的基本知识，以便顺利阅读本书的大部分内容。

# 本书内容概述

*第一章* ，*重新审视 OpenStack——设计考虑*，总结了 OpenStack 主要架构服务的最新特性。本章介绍了 Antelope 及后续版本中每个核心组件的最新更新。初步的逻辑和物理设计模型将被草拟并作为后续章节的架构参考。

*第二章*，*正确启动 OpenStack 设置 – 正确的方法（DevSecOps）*，首先介绍了 DevOps 概念，并融合了安全方面的内容。本章探讨了 DevSecOps 如何帮助以最敏捷且安全的方式管理、部署和自动化大规模的 OpenStack 私有云基础设施。本章介绍了 Kolla Ansible 项目，它将用于基于容器部署不同的 OpenStack 云基础设施组件。

*第三章*，*OpenStack 控制平面 – 共享服务*，重点介绍了设计用于在云控制节点中运行的常见 OpenStack 服务。本章深入探讨了最新 OpenStack 更新中引入的控制平面内容，包括核心服务 API、消息传递和数据库服务。基于容器部署，本章将更深入地探讨 Kolla Ansible 仓库中与 OpenStack 控制平面相关的不同组件。

*第四章*，*OpenStack 计算 – 计算能力和规格*，更深入地回顾了 OpenStack 计算 Nova 服务。调度和 Placement 服务将被介绍，包括执行资源分配以在计算节点中运行实例的高级方法。本章通过涵盖可用区、主机聚合和单元等概念，探讨了大规模 OpenStack 部署的最新计算设计更新。大部分内容将专注于学习 OpenStack 中针对容器化技术的孵化项目。Zun 和 Magnum 将作为其他服务一样，通过基础设施即代码的方式进行部署。

*第五章*，*OpenStack 存储 – 块存储、对象存储和文件共享*，探索了最新 OpenStack 版本中的不同存储选项，从 Antelope 版本开始。本章揭示了与 Cinder 块存储服务的不同后端集成，如 NFS 和 Ceph。块存储调度的更多更新被详细突出显示。对于 Manila 和 Swift，将给出简洁的更新概述。所涉及的存储服务将通过使用 Kolla Ansible 仓库的部署模型进行集成。

*第六章*，*OpenStack 网络 – 连接性和管理服务选项*，更深入地解释了 OpenStack 中的 Neutron 网络服务。本章介绍了大规模部署的网络架构的新布局。Neutron 插件将与最新 OpenStack 版本更新一起进行探索，包括 Open vSwitch 和 OVN。路由将被讨论，并展示如何通过利用 Neutron 插件在后台实现。额外的服务，如新版本的负载均衡服务（代号 Octavia），将在本章中进行演示。

*第七章*，*运行高可用云 – 满足 SLA*，介绍了在每个 OpenStack 控制平面层中增加可用性和可扩展性的不同技术和设计模式。本章探讨了在 Neutron 中实现路由冗余的不同方式。它还超越了基础设施层，专门介绍了一个自动化的方法，通过一个名为 Masakari 的新兴 OpenStack 服务来管理实例的可用性。

*第八章*，*监控与日志记录 – 主动修复*，展示了一个集中式解决方案，用于在大型 OpenStack 云环境中运行复杂的监控系统。本章介绍了 Prometheus，作为所有 OpenStack 监控指标的单一监控界面。它还探讨了使用 Grafana 在一个系统中可视化和集中化指标的简单方法。对 OpenStack 中 Ceilometer 遥测服务的简要更新也有涉及。本章揭示了一种强大且自动化的方式，通过 OpenSearch 来消化和可视化大量 OpenStack 日志。

*第九章*，*基础设施基准测试 – 评估资源容量与优化*，介绍了针对 OpenStack 性能和资源优化的更高级别的操作任务。本章讨论了如何通过使用 Rally 工具进行基准测试来评估云基础设施的核心服务和限制。它还通过启用跟踪 OpenStack 不同调用的工具（名为 OSProfiler），将性能优化提升到了新层次。章节还重点介绍了 OpenStack 项目中新加入的一项资源优化工具，名为 Watcher。

*第十章*，*OpenStack 混合云 – 设计模式*，建议了实施、部署和集成 OpenStack 的新方法，超越了私有云模型。本章讨论了采用混合云模型的趋势，并介绍了针对多个用例的混合云设计模式。我们探讨了基于 OpenStack 的公共云和私有云之间的结合。

*第十一章*，*混合云超大规模用例 – 扩展 Kubernetes 工作负载*，以混合云的工作负载扩展为主题，结束本书的内容。章节深入讨论了如何在私有 OpenStack 云和 AWS 公共云环境之间扩展工作负载。通过引入双容器和微服务的概念来实现混合云模型，本章演示了如何在私有和公共云之间运行基于 Kubernetes 的工作负载，并介绍了一款名为 Juju 的流行 Canonical 工具，用于跨云管理和调度工作负载。

# 要充分利用本书

| **书中涉及的软硬件** | **操作系统/硬件要求** |
| --- | --- |

| OpenStack（Antelope 或更高版本）、AWS 账户、Python、YAML、JSON、Ansible、Kolla、Jenkins、Docker、Kubernetes、Ceph、Canonical Juju、KubeFed | Linux（Ubuntu 22.04）、Vagrant 2.4.1 或更高版本 | 测试环境（1 主机）

+   2 个 CPU 或更多

+   8 GB 内存或更多

+   50 GB 磁盘空间或更多

每个主机的最小开发/生产环境（至少 2 台机器）：

+   2 个 CPU

+   8 GB 内存

+   100 GB 磁盘空间或更多

|

**如果你使用的是本书的数字版，我们建议你自己键入代码或从本书的 GitHub 仓库访问代码（下节会提供链接）。这样可以帮助你避免因复制和粘贴代码而产生的潜在错误。**

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，地址为 [](https://github.com/PacktPublishing/Mastering-OpenStack) [](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还提供了其他代码包，来自我们丰富的书籍和视频目录，访问 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。快来看看！

重要提示

从本书的创作到出版，OpenStack 经历了几次重要的版本更新。因此，本书某些部分提到的代码可能需要进行调整才能正常工作。每一章的相关设置或故障排除步骤，以及更新后的代码文件，都将包含在上述 GitHub 仓库中。

# 使用的约定

本书中使用了许多文本约定。

**文本中的代码**：表示文本中的代码字、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号。以下是一个示例：“在 **/kolla-ansible/etc/kolla/** **globals.yml** 文件中，将 **enable_magnum** 设置为 **yes** 将启用 Magnum API 服务。”

代码块设置如下：

```
…
[magnum:children]
control
[magnum-api:children]
magnum
[magnum-conductor:children]
magnum
…
```

当我们希望特别指出代码块中的某个部分时，相关行或项目将以粗体显示：

```
"provider_summaries": {
        "ad54422c-b345-5v44-ba23-00cad76e376a": {
            "resources": {
                "DISK_GB": {
                    "used": 0,
                    "capacity": 2000
                }
            },
            "traits": ["SRG_SHARES_STORE"],
```

任何命令行输入或输出都以如下方式书写：

```
# openstack allocation candidate list --resource VCPU=4,
MEMORY_MB=2048,DISK_GB=500GB --required   HW_CPU_X86_SVM
```

**粗体**：表示新术语、重要单词或你在屏幕上看到的单词。例如，菜单或对话框中的单词会以 **粗体** 显示。以下是一个示例：“在 Jenkins 仪表盘上，通过指向 **管理 Jenkins** 并点击 **配置系统** 来配置已安装的 Anchore 插件。”

提示或重要说明

显示为这样。

# 联系我们

我们始终欢迎您的反馈。

**一般反馈**：如果你对本书的任何部分有疑问，请发送邮件至 customercare@packtpub.com，并在邮件主题中提及书名。

**勘误**：尽管我们已尽一切努力确保内容的准确性，但错误总是难免。如果你在本书中发现错误，我们将不胜感激。如果你发现了问题，请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 并填写表单。

**盗版**：如果你在互联网上遇到我们的作品的任何非法副本，请提供该位置地址或网站名称。请通过电子邮件 copyright@packt.com 联系我们并附上相关材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享你的想法

一旦你阅读了 *Mastering OpenStack*，我们很想听听你的想法！请 [点击这里直接进入亚马逊评论页面](https://packt.link/r/1835468918) 并分享你的反馈。

你的评价对我们和技术社区非常重要，它将帮助我们确保提供优质的内容。

# 下载本书的免费 PDF 版本

感谢你购买本书！

你喜欢随时随地阅读，但又不能随身携带纸质书籍吗？

你的电子书购买是否与你选择的设备不兼容？

不用担心，现在购买每本 Packt 图书时，你都可以免费获得该图书的无 DRM PDF 版本。

随时随地，在任何设备上阅读。直接从你最喜欢的技术书籍中搜索、复制并粘贴代码到你的应用程序中。

优惠不仅仅是这些，你还可以获得独家折扣、新闻通讯以及每天发送到你邮箱的精彩免费内容。

按照以下简单步骤获取这些福利：

1.  扫描二维码或访问以下链接

![img](img/B21716_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781835468913`](https://packt.link/free-ebook/9781835468913)

1.  提交你的购买证明

1.  就是这样！我们会将免费的 PDF 文件和其他福利直接发送到你的邮箱。

# 第一部分：OpenStack 生态系统架构设计

本书的第一部分将介绍 OpenStack 的最新更新，包括 Antelope 版本和后续发布的内容。这将重点介绍任何新增功能和为部署私有云环境所需的变更。该部分将回顾设计稳健架构并实现最稳定操作能力所需的所有核心组件，其中将包括控制平面和数据平面所需的所有服务，并提供不同的配置选项。书籍的第一部分是后续章节的基础，因为它将讨论通过基础设施即代码（Infrastructure as Code）和 DevSecOps 风格部署不同 OpenStack 服务的方法。

本部分包含以下章节：

+   *第一章*， *重新审视 OpenStack – 设计考虑事项*

+   *第二章*，*启动 OpenStack 设置 – 正确的方式 (DevSecOps)*

+   *第三章*，*OpenStack 控制平面 – 共享服务*

+   *第四章*，*OpenStack 计算 – 计算能力与规格*

+   *第五章*，*OpenStack 存储 – 块存储、对象存储和文件共享*

+   *第六章*，*OpenStack 网络 – 连接性与托管服务选项*

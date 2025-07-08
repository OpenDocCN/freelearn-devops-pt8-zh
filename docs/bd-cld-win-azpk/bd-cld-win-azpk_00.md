# 前言

微软 Azure 是增长最快的公共云，提供全面的服务列表，并且随着时间的推移，服务列表还在不断增加。

2013 年，微软发布了 Windows Azure Pack，这是一个集合了微软 Azure Pack 技术的产品，帮助组织和服务提供商构建云解决方案，并提供与世界上最大的公共云之一相一致的体验。从那时起，Windows Azure Pack 通过每季度增加更多服务和功能，持续提升。

Windows Azure Pack（简称 WAP）与微软的成熟技术（如 Windows Server 和 System Center）集成，提供包括 IaaS 和 PaaS 服务在内的云服务。所提供的服务不仅限于微软技术，还包括其他供应商和开源技术，例如 Linux 操作系统和 MySQL 数据库。虽然云计算已经成为现实，且不再是未来技术，但 Windows Azure Pack 提供了一个稳固的平台，具备可扩展的架构，用于构建私有云或服务提供商的云。

本书专注于帮助读者使用 Windows Azure Pack 构建云解决方案。内容涵盖设计、实施和管理的原则以及构建云的步骤。按照本书的指导，读者将能够在本地构建自己的云解决方案，提供广泛的服务和功能。

本书将介绍云平台和服务的以下方面：

+   学习概述、架构和功能

+   服务提供商和组织私有云的规划

+   部署和管理任务

+   租户或用户体验

# 本书涵盖的内容

第一章, *了解 Windows Azure Pack 及其架构*，将介绍微软的 Cloud OS 愿景及 Windows Azure Pack 在实现这一愿景中的作用。接着，介绍 Windows Azure Pack 的功能、服务提供，并总结 WAP 如何成为构建私有云或托管服务提供商云的绝佳平台。此外，本章还将探讨 WAP 基础云解决方案的架构、构建模块和部署模型。

第二章, *构建云基础设施*，将重点介绍构建云解决方案所需的基础层构建。本章涵盖了计算、虚拟化、网络和存储基础设施的规划与配置，另外还包括部署系统中心组件（包括 SCVMM 和 SPF）。

第三章，*安装和配置 Windows Azure Pack*，将包括 Windows Azure Pack 组件的安装和配置。它还涵盖了自定义 URL 和证书的门户配置。

第四章，*构建虚拟机云和 IaaS 服务*，将涵盖构建 IaaS 服务，包括虚拟机（Windows 和 Linux）、虚拟网络等。它包括为提供 IaaS 服务构建平台，并开发自服务自动化部署的服务。

第五章，*分配云服务 – 计划、附加组件、租户账户和订阅*，将涵盖通过利用 WAP 计划和附加组件将云服务分配给租户的机制。它还将讨论租户用户账户和订阅管理。

第六章，*体验云服务 – 租户视角*，将帮助我们作为租户体验之前构建的服务。它将涵盖作为租户的云服务提供和管理。

第七章，*提供 PaaS – WebSites 云和服务总线*，将涵盖 PaaS 服务（WebSites 和服务总线）的规划和部署。此章节还将讨论如何构建网站和服务总线云服务，并作为租户体验这些服务。

第八章，*提供数据库即服务*，将涵盖 DBaaS 服务（SQL 和 MySQL）的规划和部署。此章节还将讨论如何构建 SQL 和 MySQL 数据库云，并作为租户体验这些服务。

第九章，*自动化与身份验证 – 服务管理自动化与 ADFS*，将帮助我们了解 SMA 及其将自动化添加到云部署、服务提供和运营中的功能。接着，它将介绍如何利用 ADFS 进行 Windows Azure Pack 门户中的身份验证。

第十章，*通过合作伙伴的解决方案扩展 WAP 功能*，将介绍 Microsoft Azure Stack 和 WAP 更新模型。它还将介绍许多由合作伙伴开发的其他解决方案，以扩展 WAP 的功能和服务。

# 本书所需内容

虽然本书的所有内容不要求你运行任何基础设施或软件组件，但构建本书所涵盖的云解决方案时，以下软件和足够的硬件是必需的：

+   Windows Server 2012 R2

+   Microsoft SQL Server（2008 及以上版本）

+   System Center 2012 R2

+   Windows Azure Pack

+   MySQL Server

# 本书适用对象

本书面向的是希望亲身体验 Windows Azure Pack 的云和虚拟化专业人士。它将帮助虚拟化客户采用云架构，同时也帮助现有的云服务提供商理解 Azure Pack 的优势。同样，本书还将帮助其他平台的云专业人士，例如 VMware/OpenStack，了解和评估 Azure Pack。如果你在服务器虚拟化方面有经验，并希望学习如何构建本地云解决方案，那么这本书适合你。如果你已经了解或有使用其他本地云解决方案的经验，如 VMware 云产品、OpenStack 等，并期待了解 Windows Azure Pack 在云计算领域能提供什么，这本书也适合你。如果你是现有的虚拟化用户，并希望在 IT 操作中采用云技术，这本书适合你。对于提供 IaaS 和 PaaS 服务的云服务提供商同样适用。

# 约定

在本书中，你将看到许多文本样式，这些样式用来区分不同类型的信息。以下是一些样式的示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示：“还要验证系统数据库是否已创建，如`master`、`tempdb`”

代码块如下所示：

```
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

当我们希望引起你对某个代码块中特定部分的注意时，相关行或项会以粗体显示：

```
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出都如下所示：

```
# cp /usr/src/asterisk-addons/configs/cdr_mysql.conf.sample
 /etc/asterisk/cdr_mysql.conf

```

**新术语**和**重要词汇**以粗体显示。在屏幕上看到的词语，例如菜单或对话框中的内容，会以这种形式出现在文本中：“Web PI 提供了一个单击选项，名为**Windows Azure Pack: Portal and API Express**”

### 注意

警告或重要提示以框中的形式显示，如下所示。

### 提示

小贴士和技巧显示如下。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们来说非常重要，因为它帮助我们开发出您能够真正获得最大收益的书籍。

要向我们发送一般反馈，请通过电子邮件发送至`<feedback@packtpub.com>`，并在邮件的主题中提及书名。

如果您在某个领域拥有专长，并且有兴趣撰写或参与书籍的编写，请参阅我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，作为 Packt 书籍的自豪拥有者，我们为您提供了许多帮助，帮助您充分利用您的购买。

## 下载本书的彩色图片

我们还为您提供了一份 PDF 文件，其中包含了本书中使用的截图/图表的彩色图片。彩色图片将帮助您更好地理解输出结果的变化。您可以从以下链接下载该文件：

[`www.packtpub.com/sites/default/files/downloads/Building_Clouds_with_Windows_Azure_Pack_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/Building_Clouds_with_Windows_Azure_Pack_ColorImages.pdf)。

## 勘误

尽管我们已经尽力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现错误——可能是文本或代码的错误——我们将非常感激您能报告给我们。这样，您不仅能避免其他读者的困扰，还能帮助我们改进后续版本的书籍。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，勘误将上传到我们的网站，或添加到该书籍的勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需的信息将显示在**勘误**部分。

## 盗版

网络上侵犯版权的盗版问题在所有媒体中都在持续存在。在 Packt，我们非常重视版权和许可的保护。如果您在网络上发现我们作品的任何非法副本，请立即提供该副本的网址或网站名称，以便我们采取措施进行处理。

如果您发现涉嫌盗版的内容，请通过电子邮件联系我们，邮箱地址是`<copyright@packtpub.com>`，并附上盗版内容的链接。

我们感谢您在保护我们的作者和我们为您带来有价值内容方面的帮助。

## 问题

如果你在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。

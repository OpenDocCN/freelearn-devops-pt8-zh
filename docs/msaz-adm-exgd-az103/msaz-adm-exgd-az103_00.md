# 前言

Azure 是一个不断发展的平台。它提供了一个处于技术前沿的环境，适用于许多不同的行业需求。新的功能和特性快速推出，这使得保持更新变得困难。本书将从管理角度为你提供 Azure 当前所有功能和能力的完整概述，并且是准备 AZ-103 考试的完整指南。

本书将涵盖所有考试目标。它将从如何管理 Azure 订阅和资源开始，在这一部分中，你将学习如何管理 Azure 订阅和资源组，分析资源的利用和消耗，管理**基于角色的访问控制**（**RBAC**）。第二部分，你将学习如何通过创建和配置存储帐户来实现和管理存储，如何将数据导入和导出到 Azure，如何配置 Azure 文件并实施 Azure 备份。第三部分将涉及如何部署和管理**虚拟机**（**VMs**），在这里你将学习如何为 Windows 和 Linux 创建和配置虚拟机，如何管理 Azure 虚拟机和虚拟机备份。本书的第四部分将涵盖如何配置和管理虚拟网络，包括实施和管理虚拟网络，如何将本地网络与 Azure 虚拟网络集成，如何监控和排除虚拟网络的故障，如何创建和管理 Azure 安全组和 Azure DNS，以及如何实施 Azure 负载均衡器。本书的最后一部分将涵盖如何管理身份，在这一部分你将学习如何管理 Azure**活动目录**（**AD**），如何实施和管理混合身份，以及如何实施**多重身份验证**（**MFA**）。

每一章最后都会有一个*进一步阅读*部分，这是每一章中非常重要的一部分，因为它会提供额外的、有时是关键的信息，帮助你通过 AZ-103 考试。由于考试中的问题会随时间略有变化，而且这本书最终会过时，*进一步阅读*部分将是提供所有更新的地方。

# 本书适用对象

本书面向有经验的管理员，他们希望通过*考试 AZ-103：Microsoft Azure 管理员*并拓宽自己在 Azure 管理方面的知识。

# 本书内容

第一章，*管理 Azure 订阅和资源组*，涵盖如何配置 Azure 订阅和资源组，分配管理员权限，配置 Azure 订阅策略，实施并设置资源组标签，配置成本中心配额，配置资源锁，跨资源组移动资源，以及删除资源组。

第二章，*分析资源利用率和消耗*，涵盖了 Azure Monitor，包括如何创建和分析指标和警报，创建操作组，配置资源的诊断设置，使用 Azure Log Analytics，以及利用日志查询功能。

第三章，*管理基于角色的访问控制*，涵盖了 RBAC，通过分配角色配置对 Azure 资源的访问，配置 Azure 的管理访问，创建自定义角色，Azure 策略，以及实施和分配 Azure 策略。

第四章，*创建和配置存储账户*，涵盖了 Azure 存储账户，如何创建和配置存储账户，安装和使用 Azure Storage Explorer，配置存储账户的网络访问，生成和管理 SAS，以及实施 Azure 存储复制。

第五章，*导入和导出数据到 Azure*，涵盖了如何配置和使用 Azure Blob 存储，如何导入到 Azure 作业和从 Azure 导出，如何使用**Azure 内容分发网络**（**CDN**），如何配置 Azure CDN 端点，以及如何使用 Azure 数据箱。

第六章，*配置 Azure 文件和实施 Azure 备份*，涵盖了如何创建 Azure 文件共享和 Azure 文件共享同步服务，如何使用 Azure 备份，如何使用 Azure 站点恢复，如何执行备份和恢复操作，如何创建恢复服务保险库，以及创建和配置备份策略。

第七章，*为 Windows 和 Linux 创建和配置虚拟机*，涵盖了虚拟机，如何部署 Windows 和 Linux 虚拟机，配置高可用性，部署和配置规模集，以及修改和部署**Azure 资源管理器**（**ARM**）模板。

第八章，*管理 Azure 虚拟机及其备份*，涵盖了如何管理虚拟机大小，重新部署虚拟机，移动虚拟机，添加数据磁盘和网络接口，自动化配置管理，以及配置虚拟机备份和恢复。

第九章，*实施和管理虚拟网络*，涵盖了 Azure VNet、IP 地址，如何配置子网和虚拟网络，配置私有和公共 IP 地址，以及创建和配置 VNet 对等连接。

第十章，*将本地网络与 Azure 虚拟网络集成*，涵盖了 Azure **虚拟专用网络**（**VPN**）网关，创建和配置 Azure VPN 网关，创建和配置站点到站点 VPN，验证本地连接性，以及 VNet 到 VNet 功能。

第十一章，*监控和故障排除虚拟网络*，涵盖了网络监视器、网络资源监控、管理虚拟网络连接、监控和故障排除本地连接，以及管理外部网络。

第十二章，*Azure 安全组和 Azure DNS*，涵盖了**网络安全组**（**NSGs**），如何创建和配置 NSG，将 NSG 关联到子网或网络接口，创建和评估安全规则，使用 Azure DNS，以及如何配置私有和公共 DNS 区域。

第十三章，*实现 Azure 负载均衡器*，涵盖了 Azure 负载均衡器，配置内部负载均衡器，创建健康探测，创建负载均衡规则，配置公共负载均衡器。

第十四章，*管理 Azure Active Directory*，涵盖了 Azure AD，如何创建和管理用户及组，添加和管理来宾帐户，执行批量用户更新，配置自助密码重置，Azure AD 加入，如何管理设备设置，以及添加自定义域。

第十五章，*实现和管理混合身份*，涵盖了 Azure AD Connect、如何安装 Azure AD Connect、管理 Azure AD Connect 以及管理密码同步和密码回写。

第十六章，*实现多因素身份验证*，涵盖了 Azure MFA，配置用户帐户进行 MFA，配置验证方法，配置欺诈警报，配置旁路选项和配置受信任的 IP。

# 为了最大化利用本书

本书假设你已经熟悉管理使用存储、安全、网络和云计算能力的云服务。你应该深入理解每个服务在整个 IT 生命周期中的作用。你还应该有使用 PowerShell、命令行界面、Azure 门户、ARM 模板、操作系统、虚拟化、云基础设施、存储结构和网络的经验。

# 下载示例代码文件

你可以从[www.packt.com](http://www.packt.com)账户下载本书的示例代码文件。如果你从其他地方购买了本书，你可以访问[www.packt.com/support](http://www.packt.com/support)，注册后将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  登录或注册到[www.packt.com](http://www.packt.com)。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用以下最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Microsoft-Azure-Administrator-Exam-Guide-AZ-103`](https://github.com/PacktPublishing/Microsoft-Azure-Administrator-Exam-Guide-AZ-103)。如果代码有更新，将在现有的 GitHub 仓库中更新。

我们还提供了其他代码包，来自我们丰富的书籍和视频目录，地址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快去看看吧！

# 下载彩色图像

我们还提供了包含书中截图/图表彩色图像的 PDF 文件。你可以在此下载：[`www.packtpub.com/sites/default/files/downloads/9781838829025_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781838829025_ColorImages.pdf)。

# 使用的规范

本书中使用了若干文本规范。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。例如："打开`PacktNetworkWatcher`资源组并从列表中选择`VM1`"。

代码块如下所示：

```
 { 
"Name": "Packt Custom Role",
    "Id": null,
    "IsCustom": true,
    "Description": "Allows for read access to Azure Storage, Network and Compute resources and access to support"
}
```

当我们希望引起你对代码块中特定部分的注意时，相关行或项会使用粗体显示：

```
 { 
"Name": "Packt Custom Role",
    "Id": null,
    "IsCustom": true,
    "Description": "Allows for read access to Azure Storage, Network and Compute resources and access to support"
}
```

任何命令行输入或输出如下所示：

```
Connect-AzAccount
```

**粗体**：表示新术语、重要词汇或在屏幕上看到的词汇。例如，菜单或对话框中的词汇会像这样出现在文本中。示例：“点击顶部菜单中的 Assign。”

警告或重要提示以这种方式呈现。

提示和技巧以这种方式呈现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何部分有疑问，请在邮件主题中提及书名，并通过`customercare@packtpub.com`联系我们。

**勘误表**：尽管我们已尽最大努力确保内容的准确性，但错误仍然可能发生。如果你在本书中发现错误，我们将非常感激你向我们报告。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择你的书籍，点击勘误提交表单链接并填写相关详情。

**盗版**：如果你在互联网上发现任何我们作品的非法复制版本，我们将非常感激你提供其位置地址或网站名称。请通过`copyright@packt.com`与我们联系，并附上相关材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域具有专业知识，并且有兴趣写作或为书籍做贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评价。在阅读并使用完本书后，为什么不在你购买本书的网站上留下评价呢？潜在读者可以看到并参考你的公正意见来做出购买决策，我们在 Packt 也能了解你对我们产品的看法，而我们的作者也能看到你对他们书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问[packt.com](http://www.packt.com/)。

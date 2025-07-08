# 第十七章：模拟测试问题

# 第一章，管理 Azure 订阅和资源组访问控制

1.  你有一个名为 `Subscription1` 的 Azure 订阅，其中包含两个名为 `ResourceGroup1` 和 `ResourceGroup2` 的资源组。你需要确保所有全局管理员都可以管理这两个资源组中的所有资源。你通过 Azure Active Directory 属性面板启用 Azure 资源的访问管理。此解决方案是否满足你的目标？

    1.  是

    1.  否

1.  你正在创建一个 Windows Server **虚拟机** (**VM**)，计划将其作为未来部署的镜像。你需要确保其他管理员在你完成镜像之前无法对其进行任何更改。你应该怎么做？

    1.  编辑 **基于角色的访问控制** (**RBAC**) 权限，在虚拟机级别设置。

    1.  在资源组级别编辑 RBAC 权限。

    1.  在虚拟机级别设置删除锁。

    1.  在资源组级别设置只读锁。

1.  你确定组织内的业务单元将 Azure 资源分布在不同的 Azure 资源组中。你需要确保这些资源分配到正确的成本中心。你应该怎么做？

    1.  部署 Azure 策略

    1.  创建分类标签，并在资源级别分配它们

    1.  创建分类标签，并在资源组级别分配它们

    1.  使用查询获取正确的资源，并根据此结果创建报告

# 第二章，分析资源利用率和消耗情况

1.  你有一个包含 8 个虚拟机的 Azure 订阅。你需要配置监控，并希望在 **中央处理单元** (**CPU**) 或可用内存达到一定阈值时收到通知。通知需要通过电子邮件发送，并需要在公司问题追踪器中创建一个新问题。为满足这些要求，你需要创建的最小警报和操作组数量是多少？

    1.  八个警报和一个操作组

    1.  两个警报和两个操作组

    1.  一个警报和两个操作组

    1.  一个警报和一个操作组

1.  你有两个 Azure 资源组，分别名为 `ResourceGroup1` 和 `ResourceGroup2`。`ResourceGroup1` 资源组包含 20 个 Windows Server 虚拟机，所有虚拟机都连接到名为 `Workspace1` 的 Azure 日志分析工作区。你需要编写一个日志查询，收集所有具有以下属性的安全事件：所有安全级别除了 8，且事件 ID 为 4672。你应该如何编写查询？

    1.  `SecurityEvent | where Level == 8 | and EventID == 4672`

    1.  `SecurityEvent | where Level <> 8 | where EventID == 4672`

    1.  `SecurityEvent | where Level == 8 | summarize EventID == 4672`

    1.  `SecurityEvent | where Level <> 8 | and EventID == 4672`

1.  您的公司有一个应用程序，使用 Azure SQL 数据库存储信息。公司还部署了 System Center Service Manager。您需要在数据库 CPU 使用率达到 80%时配置一个警报。当此警报触发时，您希望管理员通过电子邮件和短信收到通知。您还需要在警报触发时自动在公司问题跟踪器中创建一个工单。您应该执行哪两个操作？

    1.  配置 System Center Service Manager 与 Azure 自动化集成

    1.  配置一个包含三项操作的操作组：一项用于电子邮件，一项用于短信，一项用于创建工单

    1.  配置 IT 服务管理连接器

    1.  配置两个操作组：一个用于电子邮件和短信，一个用于创建工单

# 第三章，基于角色的访问控制管理

1.  您需要将一些全局管理员权限委派给您办公室的一个新云工程师。您决定使用 JSON 文件创建一个自定义角色，并使用以下 PowerShell cmdlet 添加该自定义角色：`New-AzureRmRoleDefinition -InputFile "C:\ARM_templates/customrole.json"`。这样做是否正确？

    1.  是的

    1.  否

1.  您的公司有一个 Azure AD 租户和一个本地 AD，它们通过 Azure AD Connect 进行同步。您有一个名为`Packt_Main`的订阅。帮助台管理员是`Packt_HD`组的成员。您需要授予帮助台组通过 Azure 门户重置用户密码的权限，同时使用最少的权限。应该怎么做？

    1.  授予`Packt_HD`组 Azure 管理员中的密码管理员角色

    1.  将密码重置权限委派给`Packt_HD`组，在 Azure 目录用户和计算机中用户的**组织单位**（**OU**）上

    1.  将`Packt_HD`组添加到域管理员用户组中

    1.  将`Packt_HD`组授予`Packt_Main`订阅的所有者角色

1.  您想在 Azure 门户中创建一个资源组管理员组。您需要为他们分配哪个 RBAC 角色，以便管理 Azure 订阅中的所有资源组？

    1.  贡献者

    1.  读取器

    1.  所有者

    1.  监控读取器

# 第四章，创建和配置存储帐户

1.  您的公司正在开发一个.NET 应用程序，用于将信息存储在 Azure 存储帐户中。您需要确保信息以安全的方式存储。您要求开发人员在访问信息时使用**共享访问签名**（**SAS**）。您需要对存储帐户进行必要的配置，以遵循安全最佳实践。以下哪种说法是正确的？

    1.  您需要配置存储访问策略。

    1.  要撤销 SAS，您可以删除存储访问策略。

    1.  您应该将 SAS 开始时间设置为当前时间。

1.  您的公司有一个应用程序，它需要将来自 Blob 存储的数据从热存取层移动到归档存取层，以降低成本。您需要创建哪种类型的存储帐户？

    1.  通用 V2 存储帐户

    1.  通用 V1 存储帐户

    1.  Azure 文件存储

    1.  Azure Blob 存储

1.  你们公司想要部署一个存储账户。你需要确保在整个数据中心发生故障的情况下数据仍然可用。解决方案必须是最具成本效益的。你应该怎么做？

    1.  配置地理冗余存储。

    1.  配置本地冗余存储。

    1.  配置读取访问的地理冗余存储。

    1.  配置区域冗余存储。

# 第五章，导入和导出数据到 Azure。

1.  你们公司开发了一个使用动态和静态内容的 web 应用程序。该应用程序部署在多个区域以实现最佳性能。用户抱怨该 web 应用程序的性能，并报告加载图片需要很长时间。你决定配置一个 **内容分发网络**（**CDN**）。你应该执行哪两项操作？

    1.  在 CDN 上实现自定义缓存规则。

    1.  在网站上实现跨源共享。

    1.  在 CDN 上实现通用的 web 交付。

    1.  在 CDN 上实现动态站点加速。

1.  你们公司开发了一个使用动态和静态内容的 web 应用程序。该应用程序部署在多个区域以实现最佳性能。用户抱怨该 web 应用程序的性能，并报告加载图片需要很长时间。你决定配置一个 CDN。配置 CDN 的两种可能方式是什么？

    1.  配置一个单一的 Azure CDN Premium Verizon 端点，配置动态站点加速，并配置缓存规则。

    1.  配置一个单一的 Azure CDN Standard Akamai 端点，配置动态站点加速，并配置缓存规则。

    1.  配置一个单一的 Azure CDN Standard Verizon 端点，配置动态站点加速，并配置缓存规则。

    1.  配置一个单一的 Azure CDN Standard Microsoft 端点，配置动态站点加速，并配置缓存规则。

1.  你们公司有大量数据存储在本地数据库和文件服务器中（120 TB）。这些数据需要上传到 Azure。上传到 Azure 的最快方式是什么？

    1.  使用 Azure Data Box。

    1.  使用 Azure Storage Explorer。

    1.  从 Azure 门户手动上传。

    1.  创建 Azure 文件共享。

# 第六章，配置 Azure 文件和实施 Azure 备份。

1.  你们公司有八台本地文件服务器和一个 Azure 订阅，其中包含一个存储账户。你计划使用 Azure 文件共享同步以在混合配置中实现 Azure 文件共享。以下哪项陈述是正确的？

    1.  Azure 文件共享同步使用 IPSec 保护混合连接。

    1.  Azure 文件共享同步减少了本地文件服务器的存储占用。

    1.  Azure 文件共享同步为本地文件共享提供容错。

1.  你正在配置 Azure 文件同步，将本地文件共享与 Azure 文件存储同步。为了确保服务在你的服务器上成功运行，必须完成哪些两个操作？

    1.  为管理员和用户禁用 Internet Explorer 增强安全性。

    1.  仅为管理员禁用 Internet Explorer 增强安全性

    1.  确保服务器上部署了 PowerShell 版本 5.1 或更高版本

    1.  确保在服务器上安装了 Azure AD Connect

1.  你正在为 Azure 文件共享设置备份和恢复。要在 PowerShell 中创建备份策略，应该使用以下哪个 cmdlet？

    1.  `New-AzRecoveryServicesBackupProtectionPolicy`

    1.  `Get-AzRecoveryServicesBackupSchedulePolicyObject`

    1.  `Get-AzRecoveryServicesVault`

    1.  `Enable-AzRecoveryServicesBackupProtection`

# 第七章，创建和配置 Windows 和 Linux 的 VM

1.  你有一个名为 `PacktResourceGroup1` 的 Azure 资源组，其中包含一个名为 `PacktVM1` 的 Linux VM。你需要自动化部署 30 台额外的 Linux 机器。这些 VM 应该基于 `PacktVM1` VM 的配置。以下哪种解决方案能够实现目标？

    1.  在 VM 自动化脚本刀片中，点击部署

    1.  在模板刀片中，点击添加

    1.  在资源组的策略刀片中，点击分配策略

1.  你的公司有一台存储在资源组中的 VM。你需要在同一个资源组中部署更多的 VM。你计划使用 ARM 模板来部署它们。你需要使用 PowerShell 从原始 VM 创建模板。你应该使用哪个 cmdlet？

    1.  `使用 Export-AzResourceGroup`

    1.  `使用 Get-AzResourceGroupDeployment`

    1.  `使用 Get-AzResourceGroupDeploymentOperation`

    1.  `使用 Get-AzResourceGroupDeploymentTemplate`

1.  你在一个可用性集内部署了一台 Windows Server 2016 机器。你需要更改该 VM 的可用性集分配。你应该怎么做？

    1.  将 VM 迁移到另一个 Azure 区域

    1.  将 VM 分配到新的可用性集

    1.  从恢复点重新部署 VM

    1.  将 VM 移动到不同的可用性集

# 第八章，管理 Azure VM 和 VM 备份

1.  你的公司有两个不同的 Azure 订阅，分别命名为 `PacktSubscription1` 和 `PacktSubscription2`，它们各自分配了不同的 Azure Active Directory。你在 `PacktSubscription1` 订阅中的一个资源组 `PacktResourceGroup1` 内部署了一个 VM。你想将该 VM 移动到 `PacktSubscription2` 中的另一个资源组。当你尝试移动 VM 时，出现了错误。最可能导致此错误的原因是什么？

    1.  VM 已配置了托管磁盘

    1.  该 VM 是经典 VM

    1.  目标资源组位于不同的订阅中

    1.  订阅位于不同的 Azure AD 租户中

1.  你需要使用 PowerShell 移动一个 VM。你应该使用哪个 cmdlet？

    1.  `Set-AzVM`

    1.  `Get-AzVM`

    1.  `Update-AzVM`

    1.  `Redeploy-AzVM`

1.  你在一个资源组中部署了一台 VM，想要增加一个额外的数据磁盘以扩展存储。你想使用 PowerShell 添加这个磁盘。你应该使用哪个 cmdlet？

    1.  `Set-AzVMDataDisk`

    1.  `New-AzVMDataDisk`

    1.  `Add-AzVMDataDisk`

    1.  `New-AzDisk`

# 第九章，实施与管理虚拟网络

1.  你的公司部署了两个 **虚拟网络（VNets）**，`VNet1` 和 `VNet2`。你需要将这两个虚拟网络连接在一起。最具成本效益的解决方案是什么？

    1.  VNet 对 VNet

    1.  站点对站点连接

    1.  用户定义路由

    1.  VNet 对等连接

1.  一个名为 `PacktVM1` 的虚拟机已部署在名为 `PacktResourceGroup1` 的资源组中。该虚拟机连接到名为 `PacktVNet1` 的虚拟网络。你计划将 `PacktVM1` 虚拟机连接到一个额外的名为 `PacktVNet2` 的虚拟网络。你需要在 `PacktVM1` 虚拟机上创建一个额外的 **网络接口**，并将其连接到 `PacktVNet2` 虚拟网络。你应该使用哪两个 Azure **命令行界面（CLI）** 命令？

    1.  `az vm nic add`

    1.  `am vm nic create`

    1.  `az network update`

    1.  `az network nic create`

1.  你需要为一个名为 `PacktVM1` 的 Windows Server 虚拟机分配一个静态 IPv4 地址，该虚拟机运行在名为 `PacktVNet1` 的虚拟网络中。你应该怎么做？

    1.  修改与 `PacktVM1` 虚拟机关联的虚拟网络接口的 IP 配置

    1.  编辑 `PacktVNet1` 虚拟网络的地址范围

    1.  使用 WinRM 连接到 `PacktVM1` 虚拟机并运行 `Set-NetIPAddress` cmdlet

    1.  使用远程桌面协议连接到 `PacktVM1` 虚拟机，并编辑虚拟机的虚拟网络连接属性

# 第十章，集成本地网络与 Azure 虚拟网络

1.  你正在管理组织的网络。本地基础设施由多个子网组成。最近增加了一个新分支机构。新办公室的网络设备被分配到 `192.168.22.0/24` 子网。你需要配置 Azure VPN 网关，确保分支办公室中的所有网络设备也能从 Azure 网络访问。你应该使用哪个 PowerShell 命令？

    1.  `Add-AzureRmVirtualNetworkSubnetConfig`

    1.  `Set-AzureRmLocalNetworkGateway`

    1.  `Set-AzureRmNetworkInterface`

    1.  `Add-AzureRmNetworkInterfaceIpConfig`

1.  你在一个 Azure 虚拟机上运行一个应用程序。你的本地网络通过 Azure VPN 网关连接到 Azure 虚拟网络。由于安全要求，应用程序不能直接暴露在互联网。市场部的用户应能够在旅行时使用公司笔记本访问该应用程序。你应该配置哪种连接方式？

    1.  ExpressRoute

    1.  点对点连接

    1.  站点对站点连接

    1.  VNet 对 VNet

1.  你的组织在美国西部、西欧洲和澳大利亚东部地区部署了 Azure 资源。公司在这些地区有四个办公室。你需要通过私有通道提供所有本地网络与 Azure 中所有资源之间的连接。你为每个 Azure 区域配置了一个 VPN 网关，并为每个办公室配置了站点对站点 VPN，并连接到最近的 VPN 网关。然后你配置了虚拟网络对等连接。你需要确保用户拥有最低的流量延迟。这种解决方案是否能满足你的目标？

    1.  是的

    1.  否

# 第十一章，监控和故障排除虚拟网络

1.  你有一台在 Azure 上部署的 Windows Server，且使用了 ExpressRoute 连接。经过两个月的正常使用后，突然收到用户反馈，他们在尝试连接服务器时遇到网络问题。你需要使用什么工具来监控到服务器的网络流量？

    1.  网络性能监视器

    1.  应用洞察

    1.  Azure 监控

    1.  网络观察器

1.  你在多个 Azure 区域配置了多个虚拟网络。你的本地基础设施位于美国东部，并配置了四个子网。你在本地基础设施中遇到网络性能问题，决定使用网络性能监视器进行故障排除。你需要在所有本地服务器上安装 Log Analytics 代理吗？

    1.  是的

    1.  否

1.  你的组织在美国西部、西欧和澳大利亚东部地区部署了 Azure 资源。公司在这些地区有四个办公室。每个办公室通过站点到站点 VPN 连接到最近的 Azure 区域。来自每个区域的虚拟网络通过虚拟网络对等连接进行连接。你需要监控这些网络之间的流量。你配置了 Azure Network Watcher 的连接故障排除功能。这个解决方案能达到你的目标吗？

    1.  是的

    1.  否

# 第十二章，Azure 安全组和 Azure DNS

1.  你的公司计划发布一个新的 Web 应用。该应用将通过 Azure 中的 App Service 部署，并面向 `packtpub.com` 域的所有用户。你已经购买了 `packtpub.com` 域名。你配置了 `packtpub.com` 的 Azure **域名系统 (DNS)** 区域，并将其委托给 Azure DNS。你需要确保该 Web 应用能够通过 `packtpub.com` 域名访问。你决定使用 PowerShell 来实现这一目标。你应该使用哪个命令？

    1.  `` New-AzDnsRecordSet -Name "packtpub.com" -RecordType "AAAA" -ZoneName "packtpub.com" ``

        `` -ResourceGroupName "MyAzureResourceGroup" -Ttl 600 ``

        `-DnsRecords (New-AzDnsRecordConfig -IPv4Address "<your web app IP address>")`

        `` New-AzDnsRecordSet -ZoneName packtpub.com -ResourceGroupName PacktAzureResourceGroup ``

        `` -Name "applicationscs.azurewebsites.net" -RecordType "CNAME" -Ttl 600 ``

        `-DnsRecords (New-AzDnsRecordConfig -Value "packtpub.azurewebsites.net")`

    1.  `` New-AzDnsRecordSet -Name "@" -RecordType "A" -ZoneName "packtpub.com" `` 

        `` -ResourceGroupName "MyAzureResourceGroup" -Ttl 600 ``

        `-DnsRecords (New-AzDnsRecordConfig -IPv4Address "<your web app IP address>")`

        `` New-AzDnsRecordSet -ZoneName packtpub.com -ResourceGroupName PacktAzureResourceGroup `` 

        `` -Name "@" -RecordType "txt" -Ttl 600 ``

        `-DnsRecords (New-AzDnsRecordConfig -Value "packtpub.azurewebsites.net")`

    1.  `` New-AzDnsRecordSet -Name "applicationscs.azurewebsites.net" -RecordType "AAAA" -ZoneName "packtpub.com" ``

        `` -ResourceGroupName "MyAzureResourceGroup" -Ttl 600 ``

        `-DnsRecords (New-AzDnsRecordConfig -IPv4Address "<your web app IP address>")`

        `` New-AzDnsRecordSet -ZoneName packtpub.com -ResourceGroupName PacktAzureResourceGroup ``

        `` -Name "www.packtpub.com" -RecordType "AAAA" -Ttl 600 ``

        `-DnsRecords (New-AzDnsRecordConfig -Value "packtpub.azurewebsites.net")`

1.  你的公司计划发布一个新的 Web 应用，并希望它能够对`packtpub.com`域的所有用户开放。你决定在 Azure 中配置一个 DNS 区域，并检查该域名是否仍然可用。你配置 Azure DNS 以支持这个 Web 应用的第一步应该做什么？

    1.  在你的 Azure DNS 区域中创建一个授权开始记录，指向 Azure DNS 服务器

    1.  在 Azure 中配置一个正向 DNS 区域

    1.  在 Azure 中配置一个私有 DNS 区域

    1.  从第三方域名注册商处购买`packtpub.com`域名

1.  你设计了一个虚拟网络拓扑，具有以下特征：Web 子网：3 个 Web 前端虚拟机，应用子网：3 个应用服务器虚拟机，数据子网：3 个数据库服务器虚拟机。公司要求严格控制子网之间的网络流量，使用**网络安全组**（**NSG**）。你需要设计一个解决方案，尽量减少 NSG 规则的创建和维护。你应该怎么做？

    1.  启用每个 NSG 中的内建规则

    1.  将路由表绑定到每个子网

    1.  定义与每个应用层对齐的应用程序安全组

    1.  在每个 NSG 中启用虚拟网络 NSG 服务标签

# 第十三章，实施 Azure 负载均衡器

1.  你已经部署了一个 Azure 负载均衡器，使用基础层，并对一个名为`PacktSet1`的可用性集中的虚拟机进行负载均衡。现在，你需要对一个名为`PacktSet2`的可用性集中的虚拟机进行负载均衡。你应该怎么做？

    1.  用在标准层创建的新负载均衡器替换现有的负载均衡器，并将此负载均衡器用于两个可用性集

    1.  编辑现有的负载均衡器，添加一个额外的后端池

    1.  编辑现有的负载均衡器，添加一个新的前端 IP 配置，将流量负载均衡到新的可用性集

    1.  部署第二个负载均衡器，使用基础层，并将此负载均衡器配置为将流量负载均衡到新的可用性集

1.  你部署了一个 Azure 公共负载均衡器，将流量负载均衡到六台虚拟机。你希望通过公共负载均衡器使用**远程桌面协议**（**RDP**）远程访问 VM1。你应该怎么做？

    1.  设置一个前端 IP 配置，将公共 IP 地址映射到 VM1 的私有 IP 地址

    1.  配置一个入站网络地址转换规则，将**传输控制协议**（**TCP**）端口`3389`映射到 VM1

    1.  配置一个新的内部负载均衡器，并将其配置为允许 TCP 端口`3386`从互联网访问 VM1

    1.  配置一个负载均衡规则，使用 TCP 端口`3386`将流量转发到 VM1

1.  你部署了一个 Azure 内部负载均衡器，用于将流量负载均衡到内部公司门户。你希望确保用户仅查看门户的最新版本。你创建了一个名为 `NewVersion.html` 的文件，并希望配置负载均衡器，只将流量引导到包含这些文件的虚拟机。你应该怎么做？

    1.  创建一个新的健康探测，使用 HTTP 作为协议，并包含 `NewVersion.html` 文件的路径。

    1.  创建一个新的健康探测，使用 TCP 作为协议，并包含 `NewVersion.html` 文件的路径。

    1.  创建一个新的负载均衡规则，包含 `NewVersion.html` 文件的路径。只有在该文件存在时，规则才会负载均衡流量。

    1.  使用 `Set-AzureRmLoadBalancerProbeConfig` PowerShell cmdlet 创建一个新的健康探测，使用 HTTP 作为协议，并包含 `NewVersion.html` 文件的路径。

# 第十四章，管理 Azure Active Directory

1.  你被要求创建一组新的 Azure **Active Directory (AD)** 安全组，代表经理团队的完整层级结构。这包括被经理管理的人员。你需要使用最少的管理工作量来实现此请求。你应该怎么做？

    1.  使用直接报告规则创建新的组

    1.  为每个经理创建新的 Azure AD 组，并使用自定义脚本检测 `ManagerID` 属性变化，必要时修改组成员资格

    1.  使用规则集创建动态组和 Azure AD，包括 `ManagerID` 属性

    1.  创建多个 Azure AD 组，并将具有相同 `ManagerID` 属性值的成员添加到每个组中

1.  你需要为一位外部顾问授予访问权限，以访问你 Azure 订阅中的某些资源。你计划使用 PowerShell 添加该外部用户。你应该使用哪个 cmdlet？

    1.  `New-AzADUser`

    1.  `New-AzureADMSInvitation`

    1.  `Get-AzADUser`

    1.  `Get-AzureADMSInvitation`

1.  你需要为另一位管理员配置账号，该管理员负责管理你 Azure 订阅中的所有 **基础设施即服务** (**IaaS**) 部署。你为用户在 Azure AD 中创建了一个新账户。你需要配置该用户账户，以满足以下要求：对所有 Azure IaaS 部署具有读写访问权限，对 Azure AD 具有只读访问权限，且无法访问 Azure 订阅元数据。该解决方案还必须最小化你未来的访问权限维护工作。你应该怎么做？

    1.  在资源级别为用户账户分配所有者角色

    1.  为用户账户分配全局管理员目录角色

    1.  在订阅级别为用户账户分配虚拟机操作员角色

    1.  在资源组级别为用户账户分配贡献者角色

# 第十五章，实现和管理混合身份

1.  你被要求配置一个解决方案，使用户在登录 Office 365 应用时无需提供密码。你的公司还希望为某些用户配置基于云的两步验证。你应该怎么做？

    1.  启用密码哈希同步

    1.  启用直通认证

    1.  安装 Azure AD Connect

    1.  启用 Azure 多因素认证

1.  你使用 Azure AD Connect 将所有 AD 域用户和组与 Azure AD 同步。因此，所有用户都可以使用 **单点登录**（**SSO**）访问应用程序。你应该重新配置目录同步，排除不应访问应用程序的域服务帐户和用户帐户。你应该怎么做？

    1.  重新运行 Azure AD Connect

    1.  停止同步服务

    1.  手动删除域服务和用户帐户

    1.  在 Azure AD 中配置条件访问规则

1.  你的公司希望启用所有用户帐户使用 SSO 登录应用程序和 Office 365。公司拥有本地 AD，并使用智能卡认证。你需要部署什么解决方案以允许用户在不输入密码的情况下登录？

    1.  使用直通认证和 SSO 的 Azure AD Connect

    1.  使用密码哈希同步和 SSO 的 Azure AD Connect

    1.  使用密码哈希同步的 Azure AD Connect

    1.  活动目录联合服务

# 第十六章，实施多因素认证

1.  你在 Azure AD 租户中部署了 **多因素认证**（**MFA**）。你不希望用户在使用 MFA 时在浏览器中输入额外的密码或代码。你应该提供哪两种方法？

    1.  拨打电话进行验证

    1.  发送短信至手机

    1.  通过移动应用程序进行通知

    1.  使用硬件令牌的验证代码

1.  你的公司拥有一个 Azure AD 租户和一个通过 Azure AD Connect 同步的本地 AD。你的本地环境运行的是 Windows Server 2012 和 Windows Server 2016 混合服务器。你使用 Azure MFA 进行多因素认证。用户报告在使用公司设备时被要求使用 MFA。你需要关闭域加入设备的 MFA。你应该怎么做？

    1.  在 Azure AD Connect 上启用单点登录（SSO）

    1.  创建条件访问规则，允许用户在访问应用程序时使用 MFA 或域加入设备

    1.  在所有域加入设备上配置 Windows Hello for Business

    1.  将公司外部 IP 地址添加到 Azure MFA 受信任 IP 列表中

1.  你的公司拥有一个 Azure AD 租户和一个通过 Azure AD Connect 同步的本地 AD。安全部门注意到来自多个公共 IP 地址的登录次数异常。你应该采取什么措施来减少这些登录？

    1.  启用 Azure AD 智能锁定

    1.  将所有公共 IP 地址添加到条件访问中，并使用位置阻止来拒绝所有登录尝试

    1.  创建条件访问规则，要求所有标记为中等风险及以上的风险登录使用 MFA

    1.  开启 Azure MFA 欺诈警报

# 第十八章：模拟测试答案

# 第一章，管理 Azure 订阅和资源组

1.  **1**—是的，这符合您的目标。从 Azure Active Directory Properties blade 设置的访问管理确保分配给全局管理员角色的 Azure AD 用户对所有订阅资源拥有完全控制权。

1.  **4**—您需要在资源组级别设置只读锁定。这将确保管理员和所有其他用户无法对为您的 VM 创建的所有不同 Azure 资源进行更改，例如对虚拟网络、磁盘等的更改。

1.  **2**—您应该创建分类标签并将其分配到资源级别；因为 Azure 资源分布在不同的资源组上，您不能将其应用到资源组级别。

# 第二章，分析资源利用和消耗

1.  **4**—您应该为此创建一个警报和一个操作组。一个警报可以包含多个基于指标的条件，一个操作组可以包含多个通知或补救步骤。因此，您可以在一个警报中创建 CPU 和内存的指标。您可以使用一个操作组发送电子邮件并在企业问题跟踪器中创建问题。

1.  **2**—正确的查询应为`SecurityEvent | where Level <> 8 | where EventID == 4672`。

1.  **2**和**3**—您需要创建一个操作组，并配置**IT 服务管理连接器（ITSMC）**。此连接器将 System Center Service Manager 与 Azure 连接起来。

# 第三章，管理基于角色的访问控制

1.  **1**—是的，这是使用 PowerShell 创建自定义角色的正确方式。

1.  **1**—您应该授予`Packt_HD`组在 Azure AD 中的密码管理员角色。该角色授予重置非管理员密码的权限，这是必需的最低权限。

1.  **3**—您应该将所有者角色分配给资源组管理者组。

# 第四章，创建和配置存储账户

1.  **1**—正确，您需要配置存储访问策略。 **2**—正确，要撤销 SAS，您可以删除存储访问策略。 **3**—错误，当将定时器设置为现在时，托管存储账户的服务器时钟可能存在差异。这可能会导致短时间内的访问问题。

1.  **1**—您需要配置通用用途 V2 存储账户以在不同访问层之间移动数据。

1.  **4**—您应该配置带有**区域冗余存储（ZRS）**复制的存储账户。这将在同一区域的三个不同区域之间进行同步数据复制。

# 第五章，将数据导入和导出到 Azure

1.  **1**和**4**—您应该在 CDN 上实施自定义缓存和动态站点加速。动态站点加速在传递动态内容时提高性能。您可以为静态内容配置缓存规则。

1.  **2**和**3**—你可以配置 Azure CDN 标准 Akamai 和 Azure CDN 标准 Verizon 终结点，配置动态网站加速，并配置缓存规则。动态网站加速在交付动态内容时能提升性能。你可以为静态内容配置缓存规则。你不应创建 Azure CDN 标准 Microsoft 终结点，因为它不支持动态网站加速。你也不应创建 Azure CDN 高级 Verizon 终结点，因为缓存是通过规则引擎而非缓存规则进行配置的。

1.  **1**—你应该订购一个 Azure 数据盒。你可以将所有数据复制到该设备，并将其寄回微软。微软随后会直接从设备将这些数据上传到 Azure 数据中心。

# 第六章，配置 Azure 文件和实施 Azure 备份

1.  **2**和**3**—Azure 文件共享同步通过使用云分层减少本地文件服务器的存储占用。这会在本地文件共享上生成热图，并将不常访问的文件归档到 Azure。它还为本地文件共享提供容错能力。如果文件服务器脱机，你可以轻松地将其文件共享恢复到另一个文件服务器。

1.  **1**和**3**—你需要确保为管理员和用户禁用了 Internet Explorer 增强型安全性，并确保在服务器上部署了 PowerShell 5.1 或更高版本。

1.  **1**—当你在 PowerShell 中创建新策略时，应该使用`New-AzRecoveryServicesBackupProtectionPolicy`cmdlet。`Get-AzRecoveryServicesBackupSchedulePolicyObject` cmdlet 获取对基础策略项的引用。`Get-AzRecoveryServicesVault` cmdlet 获取对恢复服务保险库的引用，`Enable-AzRecoveryServicesBackupProtection` cmdlet 在创建后启用备份策略。

# 第七章，创建和配置 Windows 及 Linux 虚拟机

1.  **1**和**2**—你可以从虚拟机的自动化脚本面板部署虚拟机的 ARM 模板，也可以从 Azure 门户的模板面板部署该模板。

1.  **1**—你应该使用`Export-AzResourceGroup`cmdlet。它将指定的资源组捕获为模板并保存为 JSON 文件。

1.  **3**—你应该从恢复点重新部署虚拟机。虚拟机只能在初始部署时被分配到可用性集。

# 第八章，管理 Azure 虚拟机和虚拟机备份

1.  **4**—你无法移动虚拟机，因为订阅位于不同的 Azure AD 租户中。移动虚拟机的先决条件之一是源订阅和目标订阅必须位于同一个 Azure AD 租户中。

1.  **1**—你应该使用`Set-AzVM`cmdlet，后跟`-Redeploy`方法。

1.  **3**—`Add-AzVMDataDisk` cmdlet 将数据磁盘添加到虚拟机。你可以在创建虚拟机时添加数据磁盘，也可以将数据磁盘添加到现有的虚拟机中。

# 第九章，实施和管理虚拟网络

1.  **4**—VNet 对等连接是连接不同 VNets 的最具成本效益的解决方案。

1.  **1** 和 **4**—你应该使用 `az vm nic add` 来创建一个新的 NIC。然后你应该使用 `az network nic create` 将该 NIC 附加到 `PacktVM1`。

1.  **1**—你应该修改与 `PacktVM1` 关联的虚拟网络接口的 IP 配置。

# 第十章，集成本地网络与 Azure 虚拟网络

1.  **2**—你应该使用 `Set-AzureRmLocalNetworkGateway` cmdlet。你需要重新配置本地网络网关来完成此操作。

1.  **2**—你应该配置一个 Azure VPN 网关，以接受来自用户笔记本的点对站点 VPN 连接。

1.  **1**—是的——因为你为每个区域配置了 VPN 网关，这种解决方案达到了目标。这将为用户提供最低的流量延迟。

# 第十一章，监控和排除虚拟网络故障

1.  **1**—你应该使用 Network Performance Monitor 来监控网络流量。你也可以使用它来监控 ExpressRoute 连接上的网络流量。

1.  **2**—你不需要在所有本地服务器上安装 Log Analytics 代理。你只需要在每个网络子网上安装代理，因此你至少需要安装四个代理。

1.  **2**—不可以——网络监控器只能监控从 Azure 到本地网络的流量，而不能反向监控。你需要监控所有网络上的所有流量。

# 第十二章，Azure 安全组和 Azure DNS

1.  **2**—你应该使用以下命令来添加 DNS：

    `` New-AzDnsRecordSet -Name "@" -RecordType "A" -ZoneName "packtpub.com" `` 

    `` -ResourceGroupName "MyAzureResourceGroup" -Ttl 600 `` 

    `-DnsRecords (New-AzDnsRecordConfig -IPv4Address "<your web app IP address>")`

    `` New-AzDnsRecordSet -ZoneName packtpub.com -ResourceGroupName PacktAzureResourceGroup `` 

    `` -Name "@" -RecordType "txt" -Ttl 600 `` 

    `-DnsRecords (New-AzDnsRecordConfig -Value "packtpub.azurewebsites.net")`

1.  **4**—你应该先从第三方域名注册商处购买 `packtpub.com` 域名，再进行其他操作。

1.  **3**—你应该定义与每个应用程序层次结构对齐的 **应用程序安全组（ASGs）**。这将简化 Azure 中的网络管理，并使规则维护更加直接。

# 第十三章，实现 Azure 负载均衡器

1.  **1** 和 **4**—你应该使用 Basic 层部署第二个负载均衡器，并使用该负载均衡器将流量路由到新的可用性集，或者删除旧的负载均衡器并使用 Standard 层创建一个新的负载均衡器。只有 Standard 层允许将流量路由到不同的可用性集。

1.  **2**—你应该配置一个入站 **网络地址转换** (**NAT**) 规则，将 TCP 端口 `3389` 映射到 VM1。入站 NAT 规则用于将端口映射到 VM 的内部 IP 地址。

1.  **1**—你应该创建一个新的健康探针，使用 HTTP 作为协议，并包括指向 `NewVersion.html` 文件的路径。健康探针旨在测试端口或文件是否可以访问。

# 第十四章，管理 Azure Active Directory

1.  **1**—你应该使用直接报告规则创建新的组。这将创建一个动态组，包含所有具有相同 `ManagerID` 属性的成员。此规则还会相应地处理对组的更新。

1.  **2**—你应该使用 `New-AzureADMSInvitation` cmdlet 通过 PowerShell 将外部用户添加到你的 Azure AD 租户。

1.  **4**—你应该在资源组级别为用户账户分配 Contributor 角色。这将为用户提供在资源组级别的完整读/写访问权限，但不会授予用户在订阅或 Azure AD 级别的任何权限。

# 第十五章，实施和管理混合身份

1.  **2**—你应该启用通过身份验证。这将为用户启用单一登录（SSO），并使公司能够使用 Azure MFA 实现双因素认证。

1.  **1**—你应该重新运行 Azure AD Connect。这将执行 OU 筛选并刷新目录架构。

1.  **4**—你应该部署 ADFS。使用该解决方案，用户可以通过 SSO 登录并使用智能卡认证。智能卡认证不支持 Azure AD Connect。

# 第十六章，实施多因素认证

1.  **1** 和 **3**—两者，电话拨打和通过移动应用的通知都不需要用户在浏览器中输入代码。

1.  **2**—你应该创建一个条件访问规则，以允许用户在访问应用程序时使用 MFA 或者域加入的设备。当使用域加入的设备时，规则不会强制要求 MFA。

1.  **3**—你应该创建一个条件访问规则，要求对所有标记为中等风险及以上的风险登录进行 MFA 认证。Azure AD 可以根据一系列参数对所有登录尝试应用风险级别。你可以使用条件访问根据这些级别强制执行登录要求。

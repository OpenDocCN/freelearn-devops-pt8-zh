# 第十章：通过合作伙伴解决方案扩展 WAP 功能

在第一章中，您学习了 Windows Azure Pack 的扩展功能，这些功能为定制云服务提供了灵活性。在本章中，我们将讨论一些由微软合作伙伴和第三方供应商开发的 WAP 可用扩展。

除了合作伙伴的解决方案外，我们还将探讨 Microsoft Azure Stack —— 由微软推出的未来本地云解决方案及 Windows Azure Pack 更新。

本章将涵盖以下主题：

+   Microsoft Azure Stack

+   Windows Azure Pack 更新

+   Windows Azure Pack 合作伙伴生态系统

+   在 WAP 中提供 VMware —— vConnect by Cloud Assert

+   Konube Integrator — 与公共云连接

+   Apprenda — 企业级 PaaS 解决方案

+   BlueStripe 的 WAP 性能中心

+   Cloud Assert 的使用和计费

+   Cloud Cruiser for WAP

+   GridPro 的请求管理

+   Odin — WAP APS 包

+   Cisco ACI — 应用为中心的基础设施

+   5nine 云安全

+   WAP 团队访问控制

+   Nutanix 超融合基础设施为 WAP 云服务提供支持

+   NetApp 存储为微软云提供支持

# Microsoft Azure Stack

在 2015 年 4 月的 Ignite 大会上，微软宣布将发布一款名为 Microsoft Azure Stack 的本地云平台；Azure Stack 并不是 Azure Pack 的下一个版本。它基于不同的架构，并为微软公共云的 Azure 预览门户提供一致的体验。

在 Ignite 大会上，微软云和企业部门的副总裁 Brad Anderson 说：

> *"你们告诉我们，希望在数据中心中拥有完整的 Azure。Azure Stack …… 实际上是我们将整个 Azure 提供给你们，以便在数据中心运行。"*

除了 Azure 一致性外，Azure Stack 预计还将提供与 MS Azure 内建的混合能力，这些在 WAP 中是无法实现的。

Azure Stack 不包括 System Center 组件作为 IaaS 服务的核心强制要求。资源提供者将直接与包括 Windows 服务器和 Hyper-V 在内的结构组件进行通信。预计 Azure Stack 将提供更多类似 Azure 的服务，包括基础设施、虚拟机、存储（blob）、网络、应用程序（Azure Service Fabric）、Web 应用程序、服务总线、Azure 资源管理器等。Azure Stack 预计不会提供 Azure 所有的功能，但在与 Windows Azure Pack 比较时，列表将会非常长。

除了门户和 API，Azure Stack 可能在其架构中包含以下组件：

+   Azure 资源管理器

+   网络控制器（Windows Server 2016）

+   基于 Azure 的计算、网络和存储服务

+   基于 Windows 的计算、网络和存储结构

Azure 资源管理器是一种新的服务管理 API。它用于管理资源，如虚拟机、网络、网站等。租户可以创建资源组并添加资源以启用 **RBAC** (**基于角色的访问控制**)。与微软 Azure 公共云预览门户相比，ARM 具备 RBAC 功能。

Microsoft Azure Stack 将为管理员和租户提供一个统一的门户，并将利用以下任何身份验证服务：

+   Azure Active Directory

+   Windows Active Directory

+   **Active Directory 联邦服务** (**ADFS**)

在编写本书时，Azure Stack 尚未提供测试版本。预计将在 2016 年微软产品波段发布时推出。

必须注意，微软 Azure Stack 并不是 WAP 的替代品，也不意味着 WAP 的终结。WAP 将继续按其更新系统获得更新，关于这一点我们将在下一节讨论；要了解有关 Azure Stack 的更多信息，请访问 [`www.microsoft.com/en-in/server-cloud/products/azure-in-your-datacenter/ to learn more about Azure Stack`](http://www.microsoft.com/en-in/server-cloud/products/azure-in-your-datacenter/%20to%20learn%20more%20about%20Azure%20Stack)。

# Windows Azure Pack 更新

自首次发布以来，微软不断改进 Windows Azure Pack 解决方案，添加新功能并解决问题。WAP 技术计划每季度更新一次，通常与主要包括 SCVMM 和 SPF 的 System Center 产品更新同步。在编写本书时，最新的可用更新是更新汇总 8。

在更新汇总 4 之后，基于客户和合作伙伴反馈和建议，新增了改进和功能。如果你想为 Windows Azure Pack 提交一个想法或建议，访问 [`feedback.azure.com/forums/255259-azure-pack`](https://feedback.azure.com/forums/255259-azure-pack)。Windows Azure Pack 预计将在未来的更新中支持 Windows Server 和 System Center 的未来版本。

以下 KB 文章包含在 Azure Pack 更新中修复的问题和新增的增强功能：

+   **WAP 更新汇总 1**: [`support.microsoft.com/kb/2924386`](http://support.microsoft.com/kb/2924386)

+   **WAP 更新汇总 2**: [`support.microsoft.com/kb/2932946`](http://support.microsoft.com/kb/2932946)

+   **WAP 更新汇总 3**: [`support.microsoft.com/kb/2965416`](http://support.microsoft.com/kb/2965416)

+   **WAP 更新汇总 4**: [`support.microsoft.com/kb/2992027`](http://support.microsoft.com/kb/2992027)

+   **WAP 更新汇总 5**: [`support.microsoft.com/kb/3023209`](http://support.microsoft.com/kb/3023209)

+   **WAP 更新汇总 6**: [`support.microsoft.com/kb/3051166`](http://support.microsoft.com/kb/3051166)

+   **WAP 更新汇总 7**: [`support.microsoft.com/kb/3069121`](http://support.microsoft.com/kb/3069121)

+   **WAP 更新汇总 7.1**: [`support.microsoft.com/kb/3091399`](http://support.microsoft.com/kb/3091399)

+   **WAP 更新汇总 8**: [`support.microsoft.com/en-us/kb/3096392`](https://support.microsoft.com/en-us/kb/3096392)

# Windows Azure Pack 合作伙伴生态系统

WAP，凭借其自定义扩展能力，使合作伙伴和第三方供应商能够开发解决方案，为 WAP 部署增值。这些解决方案可以包括从改进现有部署到添加新特性的技术。

当前市场上有多种解决方案可用于扩展 WAP 功能；其中一些处于开发阶段。合作伙伴解决方案可分为以下几类：

+   **基础设施**：基础设施合作伙伴提供基础组件，如服务器、存储、网络等，用于运行 Windows Azure Pack 管理和租户的工作负载。示例包括 Nutanix 和 NetApp 为 WAP 提供的基础设施解决方案。

+   **资源提供者**：这些提供者将新服务添加到 WAP 云服务目录中。示例包括通过 Cloud Assert 的 vConnect 解决方案在 WAP 上提供的 VMware 虚拟机服务。

+   **集成**：这些解决方案与 WAP 部署集成，提供洞察信息，如性能或服务仪表板、费用分摊、请求管理等。示例包括 Cloud Cruiser 提供的计费和费用分摊解决方案。

+   **画廊项目**：画廊项目为租户提供了多种选择的目录；其中包括虚拟机云和网站目录。请参阅 第四章，*构建虚拟机云和 IaaS 服务*，以及 第七章，*提供 PaaS - 网站云和服务总线*，以了解更多关于画廊项目的信息。

我们将在接下来的章节中介绍一些由合作伙伴开发的解决方案。微软提供了一个开发工具包，用于开始开发 WAP 解决方案。开发工具包可以在 [`msdn.microsoft.com/library/dn448665.aspx`](https://msdn.microsoft.com/library/dn448665.aspx) 获得。

# 提供 VMware 与 WAP – vConnect by Cloud Assert

Windows Azure Pack 可用于通过利用 Cloud Assert 的 vConnect 在 VMware 的服务器虚拟化平台上提供虚拟机。在编写本书时，VMware 拥有服务器虚拟化市场的最高市场份额。vConnect 帮助组织和服务提供商利用其现有的 VMware 基础设施向内部用户和租户提供 IaaS 服务。

vConnect 允许租户通过 VM 模板在 VMware 上配置和运行虚拟机。配置定义在 VM 模板级别；参数由租户在自助服务门户中定义。

除了虚拟机，vConnect 还通过与 Veeam 备份解决方案的集成提供备份即服务功能。

vConnect 提供以下功能，同时通过 WAP 提供 VMware：

+   VMware vCenter 与 WAP 集成

+   为 IaaS 产品定制的 VM 模板

+   支持 VM 快照

+   使用 Veeam 进行备份

+   按需配置

+   用户配额和细粒度控制

vConnect 的架构组件包括以下内容：

+   Windows Azure Pack 配备 vConnect 管理扩展、租户扩展和 API。

+   vConnect 数据库（MS SQL）

+   托管 VMware vSphere PowerCLI 的机器

+   VMware vSphere 基础设施

+   VMware VM 模板用于 IaaS 目录

+   WAP 计划和订阅

了解更多关于 vConnect for WAP 的信息，请访问 [`www.cloudassert.com/Solutions/VConnect`](http://www.cloudassert.com/Solutions/VConnect)。

# Konube Integrator – 连接公有云

Konube Integrator 允许 Windows Azure Pack 用户和租户通过 WAP 门户连接到他们的公有云订阅，配置和管理他们的资源。这为租户提供了混合云体验。组织和服务提供商可以利用 Konube Integrator 为最终用户提供集成且一致的体验。

Konube Integrator 支持用户在其 WAP 门户中连接到公有云订阅的以下连接：

+   Microsoft Azure

+   亚马逊云服务

+   Rackspace

Rackspace 的连接通过 OpenStack 实现，使其能够连接到基于 OpenStack 的其他云提供商。

Konube 在其官方网站上写道，Konube Integrator 为各种规模的客户提供，解决混合云和多云挑战。如果你正在与多个云提供商合作，无论是自有云还是第三方云，现在你有一个简化工作流的解决方案。通过简单的界面和一个跨所有云资源的单一视图，你可以向最终用户提供增强的业务敏捷性。

访问 [`www.konube.com/`](http://www.konube.com/) 了解更多关于 Konube Integrator for Windows Azure Pack 的信息。

# Apprenda – 企业级 PaaS 解决方案

Apprenda 是领先的企业级 PaaS，推动下一代软件定义企业，颠覆行业并在软件中获胜。

Apprenda 与 Windows Azure Pack 集成，为现代 .NET 和 Java 应用程序提供完整的 PaaS 功能。Apprenda 服务可以提供给租户开发者，用于创建和管理包含 WCF 和 Windows 服务的 N-Tier、云启用的 .NET 应用程序，通过 Windows Azure Pack 租户门户进行管理。

Apprenda PaaS 服务与 WAP 集成提供以下服务，为服务提供商提供能力：

+   它将集成 WAP 和 Apprenda，将你的数据中心转变为 IT 专业人士和开发者的自服务平台

+   它将在多个私有云、公有云和托管云之间推动真正的混合云场景

+   它将加速网站和自定义应用程序的开发

+   我们可以使用云启用的现有应用程序，而无需重写它们

+   它通过 WAP 提供了一个统一的管理界面，用于管理 PaaS 工作负载

+   它使治理策略得以实施，允许应用程序在内部和外部云之间安全地迁移

通过访问[`apprenda.com/lp/apprenda-in-windows-azure-pack/`](https://apprenda.com/lp/apprenda-in-windows-azure-pack/) 和 [`apprenda.com/blog/video-how-apprenda-works-with-windows-azure-pack/`](https://apprenda.com/blog/video-how-apprenda-works-with-windows-azure-pack/) 了解更多关于 Apprenda 为 Windows Azure Pack 提供的解决方案。

# BlueStripe 为 WAP 提供的性能中心

BlueStripe 为 Windows Azure Pack 提供的性能中心使云管理员和租户可以使用 WAP 门户进行端到端的应用监控和管理。

性能中心包括以下功能：

+   实时应用性能仪表板和应用层级仪表板

+   动态应用和依赖关系图，可视化当前使用的虚拟基础设施

+   与 Windows Azure Pack 工作流的紧密集成，支持从分析到修复的无缝过渡

性能中心提供商服务提供了灵活性，可定义在 WAP 计划中为租户提供的功能级别。性能中心架构包括管理和租户门户扩展、BlueStripe 资源提供程序和 FactFinder 管理服务器。

根据订阅部署 FactFinder 管理服务器，性能中心可以利用它从公共云（如 MS Azure）、任何本地私有云或系统获取统计数据。性能中心可以由租户的应用所有者使用，以设置自定义监控和仪表板，包括响应时间等参数。在写本书时，微软已经收购了 BlueStripe。

要了解有关 BlueStripe 在 WAP 中使用的更多信息，请访问[`channel9.msdn.com/Series/Windows-Azure-Pack-Partner-Solutions/04`](https://channel9.msdn.com/Series/Windows-Azure-Pack-Partner-Solutions/04)。

# 使用情况和计费由 Cloud Assert 提供

Cloud Assert 提供了 Windows Azure Pack 的费用回收费解决方案，可用于管理租户和内部客户的使用情况和计费。

Cloud Assert 在其官方网站上为使用和计费解决方案写道：

使用和计费解决方案使 Windows Azure Pack 管理员能够为超过 100 个资源配置定价，维护多个定价配置文件，集成计费系统，并根据最终用户的实际使用情况自动生成发票。

它支持多个场景，包括以下内容：

+   **计划积分：** 这类似于 Azure。用户可以有一个总体积分，例如 X 金额，且可以自由使用任何资源，直至此 X 金额。超出使用量的部分将根据使用情况自动计费。

+   **捆绑优惠：** 这类似于流行的服务提供商优惠。用户提前购买一组资源，例如 X CPU、Y 数据库大小和 Z 网站。只有超出这些包括单元积分的使用量才会被计费。

+   **纯基于使用：** 计划或资源层面不会包含任何积分。所有使用量将根据计量的资源值进行计费。

若要了解 Cloud Assert 的使用和计费详情，请访问 [`www.cloudassert.com/Solutions/Usage`](http://www.cloudassert.com/Solutions/Usage)。

# Cloud Cruiser for WAP

Cloud Cruiser 为 Windows Azure Pack 提供财务管理解决方案，帮助服务提供商和组织管理财务事务。Cloud Cruiser 提供两种形式：

+   Cloud Cruiser Express

+   Cloud Cruiser Premium

Cloud Cruiser 在其官方网站上发布了关于各版本功能的描述。

Windows Azure Pack 的 Cloud Cruiser Express 提供以下功能：

+   直接从 WAP 门户查看您的云费用。

+   自动化您的 Windows Azure Pack 计算环境的费用分摊和云计费。

+   通过租户预算和自动警报通知，掌控您的云支出。

+   使用从 Windows Azure Pack 安装程序直接安装的简化版 Cloud Cruiser Express，10 分钟内即可启动并运行。

升级到 Windows Azure Pack 的 Cloud Cruiser Premium 提供以下功能：

+   扩展您的财务管理能力，以支持更大规模的部署，包括更多的虚拟机、数据库和网站实例。

+   支持异构计算环境，增加微软云产品或其他公共、私有或传统计算环境。

+   增加高级财务管理功能，如客户和服务分析、利润管理、需求预测等。

若要了解有关 Windows Azure Pack 的 Cloud Cruiser 的更多信息，请访问 [`www.cloudcruiser.com/partners/microsoft/`](http://www.cloudcruiser.com/partners/microsoft/)。

# 由 GridPro 提供的请求管理

由 GridPro 提供的请求管理允许 Windows Azure Pack 用户提交、跟踪和更新在 Azure Pack 内的资源请求和事件。请求管理与 System Center Service Manager 集成，以提供这些功能。

GridPro 列出了使用请求管理对于组织和服务提供商的以下好处：

+   通过 Windows Azure Pack 集成，在统一门户中提供请求管理和服务管理功能。

+   它使用户和租户能够提交、跟踪和更新事件和请求。

+   它允许决策者批准请求，并允许执行者在 Windows Azure Pack 门户中完成任务。

+   它从 Service Manager 2012 服务目录中向用户和租户发布新的请求服务。

+   收集的服务使用数据可以用来实现请求的货币化。

+   使主机服务商能够提供基于 Windows Azure Pack 的更高价值服务。

+   使用数据使得基于事件和请求的计费成为可能

+   基于声明的身份验证为用户访问提供灵活性

+   它可以通过为用户提供单一视图，轻松管理资源，使他们能够根据资源类型执行预定义的操作

+   它支持 Windows Azure Pack 支持的所有 12 种语言

要了解更多关于 GridPro 的请求管理，请访问 [`www.gridprosoftware.com/en/products/requestmanagement`](http://www.gridprosoftware.com/en/products/requestmanagement)。

# Odin – WAP APS 包

正如 Odin 在其官方网站上所述，Windows Azure Pack APS 包提供对 Windows Azure Pack 的完整支持，使服务提供商能够根据其基础设施提供 Windows Azure 服务。此 APS 包通过自动化计费、配置和客户管理，减少了提供 Windows Azure Pack 服务的市场推出时间。

当你部署带有服务自动化的 Windows Azure Pack APS 包时，你将获得以下完整的业务能力：

+   商店集成

+   自动化订购和配置

+   客户自服务控制面板

+   计费和服务计划管理

+   资源报告与跟踪

+   完全的转售商和白标功能

+   定制工作流的能力，以最高效地推动业务流程

要了解更多关于 Odin 针对 Windows Azure Pack 的解决方案，请访问 [`www.odin.com/products/automation/services-and-applications/windows-azure-package/`](http://www.odin.com/products/automation/services-and-applications/windows-azure-package/)。

# Cisco ACI – 应用中心基础设施

Cisco ACI 旨在根据应用程序需求和业务策略为应用程序提供数据中心网络。ACI 通过使应用中心策略能够托管应用程序基础设施（如虚拟机和网络）在网络上，从而以应用中心的方式促进这一过程。

Cisco 将其 ACI 应用策略框架与包括 Hyper、SCVMM 和 Azure Pack 在内的微软技术集成。虽然与 WAP 集成，但 ACI 为租户提供自服务体验，使他们能够根据网络需求选择策略。

ACI 通过为 Windows Azure Pack 开发 ACI 自定义资源提供程序和扩展来实现这一点。

启用此解决方案的 ACI 架构包含以下组件：

+   带有 ACI 资源提供程序和扩展的 Windows Azure Pack 解决方案（UR 5 或更高版本）

+   SCVMM 逻辑网络

+   Cisco APIC 代理程序用于 Hyper-V——安装在所有 Hyper-V 主机上

+   包含 ACI 服务的 WAP 计划和订阅

除了网络策略外，ACI 还通过 Citrix 和 NetScaler 的产品为租户提供第 4 层到第 7 层的访问能力。要学习并开始实现 Cisco ACI 与 Windows Azure Pack 的集成，请访问[`www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/`](http://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/)和[`www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/virtualization/b_ACI_Virtualization_Guide.pdf`](http://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/virtualization/b_ACI_Virtualization_Guide.pdf)。



5nine 云安全为 Windows Azure Pack 提供了安全即服务的功能，使服务提供商能够为客户提供安全服务。它允许租户管理防火墙配置，并保护运行 Windows 或 Linux 的虚拟机。

5nine 为 Windows Azure Pack 增加了两项主要功能，包括虚拟防火墙和**IDS**（**入侵检测系统**）。这些功能无需在虚拟机内部安装代理即可提供。无代理解决方案确保虚拟机性能不受影响。

5nine 与 Hyper-V 的虚拟交换机集成，以启用这些功能。请参考以下链接了解更多关于 5nine 云安全在 Windows Azure Pack 中的应用：

[`www.5nine.com/5nine-security-for-hyper-v-product.aspx#Azure`](http://www.5nine.com/5nine-security-for-hyper-v-product.aspx#Azure)

[`www.5nine.com/Docs/5nine_wap_extension_overview_en.pdf`](http://www.5nine.com/Docs/5nine_wap_extension_overview_en.pdf)

[`www.5nine.com/Docs/5nineCloudSecurityAzurePack_Guide.pdf`](http://www.5nine.com/Docs/5nineCloudSecurityAzurePack_Guide.pdf)

# WAP 的团队访问控制

由 Terawe 控制的团队访问是一个针对 Windows Azure Pack 的范围访问控制（类似于 RBAC）解决方案。团队访问控制使得管理员、团队经理和成员能够在受控且灵活的方式下使用资源，同时实现配额管理和访问控制。要了解更多关于 WAP 团队访问控制的信息，请访问[`www.terawe.com/tac4wapack`](http://www.terawe.com/tac4wapack)。

# Nutanix 超融合基础设施用于 WAP 云

作为超融合市场的领导者，Nutanix 已对 WAP 工作负载进行功能测试，并将其迁移至 Web Scale 虚拟计算平台。要访问在 Nutanix 平台上运行 Windows Azure Pack 云的参考架构，请访问[`go.nutanix.com/microsoft-private-cloud-windows-azure-pack.html`](http://go.nutanix.com/microsoft-private-cloud-windows-azure-pack.html)。

# NetApp 存储用于微软云

NetApp 存储解决方案与 Microsoft Cloud OS 集成，支持从私有云到公共云的扩展，加速基于云的数据中心基础设施、应用程序和服务的部署。要了解更多关于 NetApp 针对 Microsoft 云平台的解决方案，请访问 [`www.netapp.com/us/solutions/cloud/microsoft-cloud/index.aspx`](http://www.netapp.com/us/solutions/cloud/microsoft-cloud/index.aspx)。

# 总结

在本章中，我们讨论了 Microsoft Azure Stack，这是微软下一代的本地云构建平台。你了解了 Windows Azure Pack 更新系统。

我们介绍了 Windows Azure Pack 合作伙伴生态系统，并讨论了合作伙伴开发的解决方案，这些解决方案扩展了 Windows Azure Pack 的功能。

本章标志着我们使用 Windows Azure Pack 构建云的旅程的结束。在本书中，你学习了如何规划、实施、管理和体验基于 Microsoft 技术构建的云解决方案。如果你按照书中的步骤进行实验，那么现在，你应该已经准备好提供一个企业级的云解决方案，涵盖 IaaS、PaaS 和 SaaS 服务。

学习是一个在这个变化莫测的 IT 世界中永无止境的过程。

我推荐 [`social.technet.microsoft.com/wiki/contents/articles/20689.the-azure-pack-wiki-wapack.aspx`](http://social.technet.microsoft.com/wiki/contents/articles/20689.the-azure-pack-wiki-wapack.aspx) 来保持对 Windows Azure Pack 的最新了解。

我推荐 [`blogs.technet.com/b/privatecloud/`](http://blogs.technet.com/b/privatecloud/) 来了解其他 Microsoft 云技术的最新动态。

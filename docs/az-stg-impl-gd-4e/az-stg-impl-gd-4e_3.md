

商业环境变得越来越复杂。许多软件应用程序现在在不同的系统上运行，这些系统位于本地、外部、多个云端以及边缘设备上。对这些多样化环境的合理规划、实施和管理是帮助您的用户和组织充分利用它们的关键因素。本章重点讨论 Azure 混合云和多云解决方案的作用，特别是 Azure Arc 和 Azure Stack 解决方案家族。

到本章结束时，您将能够做到以下几点：

+   了解什么是混合云、多云和边缘计算

+   讨论成功的混合云和多云战略的关键因素

+   解释 Azure Arc 启用的服务器、数据服务和 Kubernetes 应用管理

+   展示 Azure Stack 产品组合并解释它如何使您的数据中心现代化

本章将介绍 Azure Arc 的连接要求以及在 Kubernetes 集群和托管 Kubernetes 服务上创建 Azure Arc 启用的数据服务的方法。您还将学习如何将 Azure Stack 解决方案家族纳入您的架构，并管理 Azure Stack Hub。在第一部分，我们将从一些术语入手。

## 第三章：什么是混合云、多云和边缘计算？

我们已经在*第一章*《介绍》中介绍了混合云、多云和边缘计算，但这里有一个快速回顾：

+   **混合云**方法涉及将云资源和非云资源（例如本地数据中心）结合使用。这可以提供更大的灵活性、更多的部署选项、可扩展性和操作一致性。

+   **多云**方法涉及利用来自多个服务提供商的云计算服务。这可以提供更高的灵活性和更好的风险缓解，因为您可以选择最适合组织需求的服务组合和地区提供商。

+   **边缘计算**方法利用云的计算能力，在接近数据创建和消费端点的设备上进行处理。这些端点可能包括生产线上的过程控制系统、建筑监控系统以及远程站点的传感器和执行器。

您选择使用这些计算环境中的哪一个取决于多种不同的因素。例如，包含本地系统的混合云方法可能是您的组织为满足监管和数据主权要求，同时增强韧性和业务连续性的最佳选择。

相反，如果您的组织希望减少响应延迟并确保现场的离线可用性，边缘计算通过其**虚拟机**（**VMs**）、容器和每个设备的数据服务可能至关重要。

需要牢记的是，混合云、多云和边缘计算也可能增加运营复杂性。例如，多云需要处理不同云环境和不同服务提供商，这对任何已经面临云技术人才短缺的组织来说，都是一个额外的挑战。在这种情况下，像 Azure 这样的解决方案可以提供帮助，因为它旨在简化多云以及混合云和边缘计算的管理。

现在我们已经讨论了术语，接下来让我们谈谈战略。

## 什么使得混合云和多云战略成功？

使您的团队能够使用适合其需求的技术，同时在整个组织的各个位置提供安全性和治理，是成功使用混合云和多云解决方案的关键目标。除了提升最终用户体验外，这样的环境还使应用程序开发和 IT 管理的统一操作成为可能。您的开发人员可以使用相同的工具、API 和部署方式在组织中构建应用程序。您的管理、安全性和治理可以在所有业务位置中实现一致性。跨位置一致的数据库和计算栈选择使您能够根据需要快速移动数据和工作负载。

总体而言，Azure 混合云和多云解决方案及产品让您能够优化：

+   **用户应用体验**：在您的整个云、本地和非本地资源中创建一致的应用体验。

+   **数据服务**：通过在您需要的任何地方运行 Azure 数据服务，实现数据迁移、管理和分析的无缝化。

+   **IT 管理**：通过集中控制，统一管理、治理和保护您的 IT 资源，涵盖所有组织域。

+   **安全性和威胁防护**：在您的数字资源中使用一个控制平面，提供安全性和高级威胁防护，保障所有工作负载。

+   **身份和用户访问**：使用统一的平台进行身份和访问管理，提供全球无缝的单点登录用户体验。

+   **网络**：通过使 Azure 成为您当前网络的扩展，安全地连接组织中分布的工作负载和位置。

Azure Arc 和 Azure Stack 是 Azure 混合云和多云解决方案及产品的两个组成部分。Azure Arc 提供简化的部署管理，并将 Azure 应用服务和数据扩展至多云、数据中心和边缘。Azure Stack 通过其三种产品以及可选择性地使用 Azure Arc，提供：

+   一种云原生集成系统，将 Azure 云服务带到本地，针对断开连接的场景。

+   一个**超融合基础设施**（**HCI**），通过使用现代的软件定义存储和网络结合 Hyper-V 进行计算，更新虚拟化主机，从而现代化数据中心。

+   用于运行边缘计算工作负载的云管理设备/边缘设备。

它们使您能够：

+   使用单一控制平面无缝管理、治理和保护跨多云、本地和边缘的服务器、Kubernetes 集群和应用程序。

+   在任何基础设施上保持 Azure 服务的最新状态，同时实现自动化、统一管理和安全性。

+   使用云原生技术现代化本地或边缘应用。

+   将虚拟化应用与本地 HCI 相结合，轻松为其提供 Azure 服务，以获得最佳的价格性能比。

+   将 Azure 计算、存储和 AI 扩展到 IoT 和其他边缘设备，并在边缘运行机器学习和高级分析，以获得实时洞察。

    #### 注意

    本章重点介绍 Azure Arc 和 Azure Stack 的规划、实施以及最佳实践。有关 Azure 安全性和相关主题的更多信息，请访问[`azure.microsoft.com/services/security-center/`](https://azure.microsoft.com/services/security-center/) 和 [`docs.microsoft.com/azure/?product=all`](https://docs.microsoft.com/azure/?product=all)。

    微软还提供了构建混合云和多云解决方案的架构示例，并提供灵活的迁移路径。以下是一个关于管理 Azure Arc 启用服务器配置的示例：[`docs.microsoft.com/azure/architecture/hybrid/azure-arc-hybrid-config`](https://docs.microsoft.com/azure/architecture/hybrid/azure-arc-hybrid-config)。

希望这些初步章节为您提供了一个良好的入门基础。在接下来的章节中，我们将深入探讨 Azure Arc。

## Azure Arc 概述

随着云计算、非云计算、本地解决方案和外部解决方案的扩展，组织必须管理日益复杂的 IT 环境。多个云和环境意味着多个不同的管理工具和学习曲线。作为额外的挑战，现有工具可能无法充分支持像 DevOps 和 ITOps 这样的云原生操作模型的新版本。

Azure Arc 提供了一种简化管理和治理这些环境的解决方案。使用 Azure Arc，您可以将 Azure 服务部署并扩展到任何地方，随时进行 Azure 管理。它是一个统一的平台（单一视图），用于一致地管理跨多云、本地和边缘资源，每个资源都通过 Azure Arc 映射到 **Azure 资源管理器** (**ARM**) 中。这使您可以像在 Azure 中运行一样，使用 Azure 服务和管理工具来管理虚拟机、Kubernetes 集群和数据库。

Azure Arc 的主要优势是：

+   在单一界面上实现对您的操作和合规性的可视化。使用 Azure 门户管理、治理和保护广泛的资源，包括 Windows、Linux、SQL Server 和 Kubernetes，涵盖数据中心、边缘和多云。

+   能够设计和架构大规模的混合应用，组件分布在公共云服务、私有云、数据中心和边缘位置，且不牺牲中心化的可见性和控制。可以集中编写和部署应用，确保在任何位置的任何 Kubernetes 分发版本上自信地进行部署。通过标准化部署、配置、安全性和可观察性加速开发。

+   将 Azure 应用服务和数据扩展到所有位置。利用 Azure 数据和应用服务实现云实践和自动化，持续部署可扩展的、最新的 Azure Arc 启用服务。

使用 Azure Arc，您可以：

+   继续并扩展由 IT 部门为您的业务（ITOps）管理的流程和服务，在所有这些环境中都可以应用。

+   部署 DevOps 来支持所有环境（包括云和非云）中的新云原生模式。

+   管理和治理 Windows 和 Linux 服务器，包括物理机和虚拟机，涵盖 Azure 内外。

+   管理和治理大规模的 Kubernetes 集群，并通过源代码管理实现应用的一致部署和配置。

+   在您组织内部和外部，按需扩展管理 Azure SQL 数据库和 PostgreSQL Hyperscale 服务（Azure 数据服务），并通过自动修补、升级和安全性进行支持。

具体来说，Azure Arc 的关键特性包括：

+   在各个环境中保持一致的服务器清单、管理、治理和安全性。

+   通过配置 Azure VM 扩展来监控、保障安全并更新您的服务器，以便使用 Azure 管理服务。

+   使用 DevOps 技术管理和扩展任何符合 CNFC 标准的 Kubernetes 分发版集群。

+   基于 GitOps 的管理，使用 **配置即代码**（**CaC**）跨多个集群直接从源代码管理工具部署应用和配置。

+   使用 Azure Policy 实现 Kubernetes 集群的自动化（零触发）合规性和配置。

+   在任何 Kubernetes 环境中部署 Azure SQL 托管实例和 Azure PostgreSQL Hyperscale（Azure 数据服务），并实现 Azure 级别的升级/更新、安全性、监控，同时利用 Azure SQL 高可用性功能和 PostgreSQL Hyperscale 离线场景支持。

+   无论资产位于何处，都能在 Azure Arc 启用资产中获得统一视图，无论是通过 Azure 门户、CLI、PowerShell 还是 REST API。

    #### 注意

    要在 Azure Stack HCI 上运行 Kubernetes 工作负载，客户可以在 Azure Stack HCI 上部署 Azure Kubernetes Service，这是为 Azure Stack HCI 特别设计的服务。

现在，我们已经对 Azure Arc 可以帮助你实现的目标有了一个概览，接下来我们将深入了解一些细节。我们将首先介绍 Azure Arc 启用的基础设施，重点讨论它如何促进大规模管理 Kubernetes。接下来的部分，我们将介绍 Azure Arc 启用的数据服务，这使得可以在本地、其他公共云和边缘运行 Azure 数据服务。

### 大规模的 Kubernetes 应用管理

容器的优势，包括可移植性、效率和可扩展性，使其在部署业务应用程序时变得流行。新应用程序可以作为微服务在 Kubernetes 集群中创建。遗留应用程序可以通过将其重写为容器来实现现代化。然而，随着这些可能性的出现，也带来了对大规模管理 Kubernetes 的需求。

Azure Arc 满足了这一需求，允许组织在多个地点的任何 Kubernetes 集群中快速部署应用程序，并在强大的管理政策下进行管理。此功能通过 **Azure Kubernetes 配置管理**（**AKCM**）启用，AKCM 是一项 Azure 服务，使用 GitOps 从 Azure 提供配置管理和应用部署。通过将应用政策与特定的 GitHub 仓库关联，可以确保持续合规的部署。通过这一功能，集群管理员可以在 Git 中声明其集群配置和应用程序。开发团队可以利用拉取请求和他们熟悉的工具（例如现有的 DevOps 流水线、Git、Kubernetes 清单和 Helm 图表等）轻松地将应用程序部署到 Azure Arc 启用的 Kubernetes 集群上，并在生产环境中进行更新。GitOps 代理监听变化，并在这些变化导致系统与源真相偏离时，便于自动回滚。

例如，一家拥有多个网点的零售企业可以将其店内应用程序迁移到 Kubernetes 集群中的容器。Azure Arc 使这些容器化应用程序能够在多个地点进行统一部署、配置和管理。新网点可以接收特定的应用程序集，并通过集中控制合规性和配置来管理。IT 团队还可以在所有网点中监控、保护并更改配置和应用程序，同时利用政策来保护网络连接并防止配置错误。

**Azure Kubernetes 服务**（**AKS**）可用于监控 Kubernetes 集群健康状况，进行维护、挂载存储卷，并为特定任务（例如，使用支持 GPU 的节点进行并行处理）激活节点。随后，Azure Arc 和 Azure 政策为零售商的 IT 团队提供了在 Azure 门户中查看所有网点集群的统一视图。你还可以在 Azure Stack HCI 上运行 Azure Arc 启用的数据服务。关于这一点将在本章后面讨论。

### Azure Arc 启用的服务器

通过启用 Azure Arc 的服务器，您可以将本机 Azure 虚拟机级别的管理应用到位于 Azure 外的 Windows 和 Linux 机器，例如在企业网络上或作为其他云服务提供商服务的一部分。这些非 Azure 资源可以连接到 Azure 并作为 Azure 资源进行管理。每个连接的混合机器将被包含在资源组中，并分配一个机器级资源 ID。它还可以使用标准的 Azure 功能进行管理，如 Azure 策略、RBAC 和标签。

例如，管理多个客户环境的服务提供商可以通过 Azure Arc 扩展本机 Azure 管理功能，例如 Azure Lighthouse，允许服务提供商登录到其租户，以便根据客户委托管理订阅和资源组。

Azure Arc 的控制平面功能提供：

+   管理组和标签用于组织您的资源。

+   使用 Azure 资源图进行索引和搜索。

+   使用 Azure **基于角色的访问控制**（**RBAC**）进行安全性和访问控制。

+   用于环境创建和自动化的模板和扩展。

+   更新管理。

### 支持的场景

当您将机器连接到启用 Azure Arc 的服务器时，以下好处可用：

+   **Azure 自动化变更跟踪和库存**：用于报告被监控服务器上的配置更改。此功能可用于 Microsoft 服务、Windows 注册表和文件、Linux 守护进程以及已安装的软件。

+   **Azure 自动化状态配置**：用于简化非 Azure Windows 或 Linux 机器的部署。您还可以使用自定义脚本扩展进行软件安装或部署后配置。

+   **更新管理**：用于管理 Windows 和 Linux 服务器操作系统的更新。有关更多信息，请查看 [`docs.microsoft.com/azure/automation/update-management/enable-from-automation-account`](https://docs.microsoft.com/azure/automation/update-management/enable-from-automation-account)。

+   **Azure 虚拟机监控**：此功能允许您监控连接机器上的来宾操作系统性能。您可以发现应用程序组件并监控它们的进程和依赖关系。

+   **Azure 策略来宾配置**：您可以像为 Azure 本机机器分配策略一样，为您的混合机器分配（而不仅仅是审核）Azure 策略来宾配置。

+   **Azure Defender**：用于检测和响应威胁，以及管理非 Azure 服务器的预防性安全性和合规性能力。

    #### 注意

    每个通过 Azure Arc 管理的混合机器都需要安装 Azure Connected Machine 代理程序。为了对机器上的操作系统和工作负载进行主动监视，您还应安装适用于 Windows 和 Linux 的 Log Analytics 代理程序，然后使用自动化运行簿、更新管理、Azure 安全中心或其他合适的服务管理设备。从混合设备收集和存储的日志数据可以通过其资源 ID 或其他属性在 Log Analytics 工作区中识别。

### 支持的区域

无论您是手动启用 Azure Arc 启用的服务器，还是通过运行模板脚本，选择地点自然是离您设备地理位置最近的 Azure 区域。您选择的区域可能还受数据存储位置要求的影响，因为数据存储在包含所选区域的 Azure 地理位置中。

如果您选择的 Azure 区域发生故障，您的连接机器不会受到影响。另一方面，基于 Azure 的管理操作可能无法完成。对于具有多个设备或位置的地理冗余服务，请将它们连接到不同的 Azure 区域。

从连接的机器收集和存储一些数据。以下内容存储在创建 Azure Arc 机器资源的区域中：

+   计算机 **完全合格域名**（**FQDN**）

+   计算机名称

+   Connected Machine 代理程序的版本

+   操作系统的名称和版本

每 5 分钟，连接的机器向服务发送一次心跳信号。如果这些信号停止，则服务将在 15 到 30 分钟内将设备在门户中的状态更改为已断开连接。当连接的机器代理发送新的心跳信号时，状态将恢复为已连接。

欲知更多信息，请参阅：

+   [`docs.microsoft.com/azure/azure-arc/servers/overview`](https://docs.microsoft.com/azure/azure-arc/servers/overview)

+   [`docs.microsoft.com/azure/azure-arc/servers/agent-overview`](https://docs.microsoft.com/azure/azure-arc/servers/agent-overview)

### Azure Arc 启用的数据服务

使用 Azure Arc 和 Kubernetes，指定的 Azure 数据服务可供您在 Azure 之外的选择基础设施上运行，例如，本地、边缘设备以及其他云中。Azure 创新、可扩展性和统一管理都可用于连接和断开连接的数据工作负载。首个 Azure Arc 启用的数据服务首先在公开预览中推出的是 SQL 托管实例和 PostgreSQL 超大规模。需要注意的是，您还可以在后面章节讨论的 Azure Stack HCI 上运行 Azure Arc 启用的数据服务。

### 永不陈旧（始终保持当前）

通过使用启用 Azure Arc 的数据服务处理本地工作负载，您可以系统地访问最新的 Azure 功能和能力。您还可以配置自动更新，从 Microsoft 容器注册表接收最新的补丁和升级，同时根据您的策略控制部署节奏，并最大化系统的正常运行时间。另一个优势是，您可以避免数据库的支持结束问题，因为启用 Azure Arc 的数据服务是一项订阅服务。

### 弹性

启用 Azure Arc 的数据服务为您的本地数据库提供类似云的弹性扩展能力，允许您满足波动性和突发性工作负载的要求，以及实时数据摄取和查询。扩展性没有限制，您可以在几秒钟内启动数据库实例，实现亚秒级的操作响应时间。数据库管理任务，如配置高可用性，已简化至仅需几次点击的程度。数据工作负载可以根据所需的容量动态扩展，而无需停止应用程序，并且可以进行横向扩展或纵向扩展，以增加读取副本或分片。

### 统一管理

您可以通过使用 Azure 门户、Azure 数据工作室或 Azure 数据 CLI 等工具，统一查看通过 Azure Arc 部署的数据资产。您可以使用来自 Kubernetes API 的日志和遥测数据检查基础设施的容量和健康状态。Azure Monitor 还可以提供跨所有数据资源的操作视图和洞察。

为了通过资源扩展管理，Azure Arc 自动化数据库管理任务。开箱即用的功能包括监控、快速配置、按需弹性扩展、修补、高可用性设置以及备份和恢复。借助 Azure Arc，您数字领域中的数据库还可以受益于 Azure 备份、Azure Monitor、Azure 策略、Azure RBAC 和高级数据安全功能。

### 在断开连接场景中的管理

您可以通过直接或非直接、持续的云连接访问 Azure 的好处。许多服务，包括监控、自动备份/恢复和自助服务配置，无论是否可以直接连接到 Azure，都可以在本地运行。本地的 Azure Arc 数据控制器为您的自托管环境提供全面的管理功能，支持配置、弹性扩展、备份、自动更新、监控和高可用性。直接连接到 Azure 还可以增加与其他 Azure 服务（如 Azure Monitor）的集成功能，并使您可以从任何地方使用 Azure 门户和 Azure 资源管理器 API 来管理启用 Azure Arc 的数据服务。

### Azure 数据服务的先决条件

你需要基于主要 Kubernetes 发行版的 Kubernetes 集群，用于在你选择的基础设施上编排 Azure 数据服务。你还需要在配置 Azure 数据服务并在你的环境中使用管理功能之前安装 Azure Arc 数据控制器。

### 你应该选择启用 Arc 的 SQL Server 还是启用 Arc 的托管实例？

Azure 已经提供了不同的部署和管理选项，用于托管 SQL Server 功能。通过 Azure Arc 对 SQL Server 的支持，扩展了更多的可能性。Azure Arc 的可能性可以如下进行比较：

+   **启用 Azure Arc 的 SQL Server（目前处于预览阶段）**：对于你自己的基础设施或其他公共云中的 SQL 服务器，启用 Azure Arc 的 SQL Server 让你可以将这些 SQL 服务器连接到 Azure，并利用相关的 Azure 服务。连接和注册到 Azure 不会对 SQL 服务器产生影响。也不需要进行数据迁移或停机。通过 Azure 门户作为你的中央管理仪表板，你可以管理所有的 SQL 服务器。SQL Server 按需评估服务让你定期验证 SQL Server 环境的健康状况，减轻风险并提升性能。

+   **启用 Azure Arc 的 SQL 托管实例（目前处于预览阶段）**：启用 Azure Arc 的 SQL 托管实例是一个 Azure SQL 数据服务。它可以在你的基础设施中的任何地方创建，并且与最新的 SQL Server 数据库引擎几乎 100% 兼容。通过启用 Azure Arc 的 SQL 托管实例，你可以将应用程序迁移到 Azure Arc 数据服务，同时最大程度地减少对应用程序和数据库的修改。

你可以在不离开现有基础设施的情况下，将现有的 SQL Server 应用程序迁移到 SQL Server 引擎的最新版本，并获得类似 PaaS 的集成管理能力的额外优势。这些能力帮助你满足合规性标准，如数据主权。为此，你将利用 Kubernetes 平台与 Azure 数据服务，可以在任何基础设施上进行部署。

目前，以下优势是可用的：

+   在一分钟内轻松创建、移除并进行弹性扩展或缩减托管实例。

+   该平台会自动安装升级、更新和补丁，以确保你运行的是最新版本的 SQL Server。

+   监控、高可用性、备份和恢复作为集成管理服务提供。

### 启用 Azure Arc 的 PostgreSQL Hyperscale 还是 Azure Database for PostgreSQL Hyperscale？

这两个实体之间的差异类似于启用 Arc 的 SQL Server 与启用 Arc 的 SQL 托管实例之间的差异。Azure PostgreSQL Hyperscale 是由 Microsoft 在其数据中心运营的 Azure 服务。而启用 Azure Arc 的 PostgreSQL Hyperscale 是 Azure Arc 启用的数据服务的一部分，并在你自己的基础设施上运行。然而，两个实体都基于由 Citus 扩展支持的 PostgreSQL 数据库的超大规模（hyperscale）形式。

### 连接模式

你的启用 Azure Arc 的数据服务环境可以通过不同的方式连接到 Azure，具体取决于如业务政策、政府法规以及可用的网络连接等因素。通过启用 Azure Arc 的数据服务，你可以选择以下连接模式：

+   直接连接（写作时不支持）

+   间接连接

+   永不连接（写作时不支持）

启用 Azure Arc 的数据服务的一些功能是否可用，将取决于你选择的连接模式。连接模式的选择决定了传输到 Azure 的数据量以及与 Arc 数据控制器的用户交互类型。我们来讨论直接连接与间接连接的区别。

### 直接连接

当启用 Azure Arc 的数据服务直接连接到 Azure 时：

+   用户可以通过 Azure 资源管理器 API、Azure CLI 和 Azure 门户操作 Azure Arc 数据服务。

+   **Azure Active Directory**（**Azure AD**）和 Azure RBAC 可用，因为在直接连接模式下有持续和直接的通信。

+   诸如 Azure Defender 安全服务、容器洞察和 Azure 备份到 Blob 存储等服务可以在直接连接模式下使用。

直接连接模式下的操作类似于使用 Azure 门户管理服务，例如，提供/撤销提供、配置和扩展。

可以使用直接连接：

+   使用公共云的组织，如 Azure、AWS 或 Google Cloud Platform。

+   边缘站点位置，例如零售店，通常可以连接互联网并允许使用。

+   企业数据中心，允许更广泛地连接其数据中心的数据区域和互联网。

### 间接连接

当启用 Azure Arc 的数据服务间接连接到 Azure 时：

+   在 Azure 门户中仅提供只读视图。你可以查看托管实例和 Postgres Hyperscale 实例的实例及其部署详情，但不能在 Azure 门户中操作它们。

+   所有操作必须通过本地启动，使用 Azure Data Studio、Azure Data CLI（`azdata`）或 Kubernetes 原生工具，如 `kubectl`。

+   Azure AD 和 Azure RBAC 不可用。

+   诸如 Azure Defender 安全服务、容器洞察和 Azure 备份到 Blob 存储等服务不可用。

截至撰写时，只有间接连接模式被支持（处于预览阶段）。

间接连接可用于：

+   本地数据中心，例如金融、医疗或政府部门，可能会禁止进出数据区域的连接。这是由于业务或合规政策，通常是为了避免外部攻击或数据泄露的风险。

+   边缘站点位置，例如油气或军事领域应用，通常无法连接到互联网。

+   边缘站点位置，例如只有间歇性连接的船只。

### 从未连接

在“从未连接”模式下，无法以任何方式向 Azure 发送或从 Azure 接收数据。此场景的使用案例可能是一个高度机密的政府设施。这种类型的空气隔离环境确保完全的数据隔离。

请注意，目前此模式尚不支持。

### 连接要求

您环境中的代理始终是您环境与 Azure 之间通信的发起者。这对于在 Azure 门户中由用户发起的操作也适用，这些操作会转化为排队任务。然后，您环境中的代理通过与 Azure 发起通信来检查任务队列。代理执行这些任务并向 Azure 报告任务状态（完成或失败）。

### 间接连接模式下可用的连接

当前预览阶段仅支持的模式是间接连接模式，该模式有三种可用的连接方式。它们分别是：

+   Azure Monitor API

+   Azure 资源管理器 API

+   **Microsoft 容器注册表**（**MCR**）

所有与 Azure 和 Microsoft 容器注册表的 HTTPS 连接都是加密的。它们使用 SSL/TLS 和官方签名且可验证的证书。

### Azure Monitor API 和 Azure 资源管理器 API

Azure Data Studio、Azure Data CLI（`azdata`）和 Azure CLI 连接到 Azure 资源管理器 API，用于向 Azure 发送数据和接收数据，以便启用某些功能。

当前，所有浏览器到 Grafana 和 Kibana 仪表盘的 HTTPS/443 连接，以及从 Azure Data CLI（`azdata`）到数据控制器 API 的连接，都使用 SSL 加密并且是自签名证书。计划推出一个功能，使您可以使用自己的证书来加密这些 SSL 连接。

### Microsoft 容器注册表

MCR 是用于 Azure Arc 启用的数据服务容器镜像的仓库。您可以从 MCR 拉取这些镜像并将其推送到私有容器注册表。然后，您可以配置数据控制器部署过程，从该私有容器注册表拉取容器镜像。

从 Azure Data Studio 和 Azure Data CLI（`azdata`）到 Kubernetes API 服务器的连接，使用您已配置的 Kubernetes 身份验证和加密。为了执行与 Azure Arc 启用的数据服务相关的许多操作，Azure Data Studio 和 Azure Data CLI 的用户必须通过身份验证连接到 Kubernetes API。

### Azure Arc 数据控制器创建

有几种不同的方法可以在 Kubernetes 集群和托管 Kubernetes 服务上创建启用 Azure Arc 的数据服务。

您可以在此找到当前支持的 Kubernetes 服务和发行版的列表：[`docs.microsoft.com/azure/azure-arc/data/create-data-controller`](https://docs.microsoft.com/azure/azure-arc/data/create-data-controller)。

此 URL 还将为您提供有关服务和发行版的要求。例如，支持的最低版本、虚拟机大小、内存和存储要求，以及连接性要求。此外，您还将找到在创建控制器过程中所需的信息。

### Azure Arc 在 GitHub 上

GitHub 仓库 [`github.com/microsoft/azure_arc`](https://github.com/microsoft/azure_arc) 包含了不同的资源，帮助您使用启用 Azure Arc 的服务器、启用 Azure Arc 的 SQL Server、启用 Azure Arc 的 Kubernetes 和启用 Azure Arc 的数据服务。

我们将查看不同的指南，这些指南可以帮助您采用 Azure Arc。

### 启用 Azure Arc 的服务器

这些部署场景为指导您将不同的 Windows 和 Linux 服务器部署接入 Azure Arc 提供了帮助：

+   **通用**：可以用于将现有的 Windows 或 Linux 服务器连接到 Azure Arc 的示例。

+   **Microsoft Azure**：指导如何将 Azure 虚拟机接入 Azure Arc 作为启用 Azure Arc 的服务器。

+   额外的指南提供了有关使用 Vagrant、**Amazon Web Services** (**AWS**)、**Google Cloud Platform** (**GCP**) 和 VMware 的信息。

+   **启用 Azure Arc 的服务器—第 2 天场景和用例**：在将服务器资源通过 Azure Arc 投射到 Azure 后，这些指南展示了如何使用原生 Azure 管理工具（如资源标签、Azure 政策和日志分析）管理这些服务器，将其作为原生 Azure 资源进行管理。

+   **启用 Azure Arc 的服务器—扩展部署场景**：指导如何在不同平台和现有环境中扩展 Azure Arc 对虚拟机的接入。

### 启用 Azure Arc 的 SQL Server

这些部署场景提供了将不同平台上的 Microsoft SQL Server 部署接入 Azure Arc 的指导。

+   **Microsoft Azure**：指导如何将 Azure 虚拟机（VM）与 SQL Server 安装一起接入 Azure Arc，作为启用 Azure Arc 的 SQL Server 和 Azure Arc 启用的服务器。本指南仅供演示和测试使用（不提供支持）。

+   **AWS**：提供了在 AWS 中进行端到端部署的指导，包括使用 SQL Server 的 Windows Server 和通过 Terraform 将其接入 Azure Arc。

+   **GCP**：提供了在 GCP 中进行端到端部署的指导，包括使用 SQL Server 的 Windows Server 和通过 Terraform 将其接入 Azure Arc。

+   **VMware**：提供了在 VMware vSphere 中进行端到端部署的指导，包括使用 SQL Server 的 Windows Server 和通过 Terraform 将其接入 Azure Arc。

### 启用 Azure Arc 的 Kubernetes

本节提供了 Azure Arc 启用 Kubernetes 的选项，用于快速创建准备好在 Azure Arc 中进行投影并通过 Azure 本地工具进行管理的 Kubernetes 集群：

+   **通用**：通过连接到 Arc 或现有 Kubernetes 集群的示例演示。

+   **AKS**：创建 AKS 集群以模拟本地运行的集群的示例演练。部署示例包括使用 Terraform 和 ARM 模板。

+   **Amazon Elastic Kubernetes Service**（**EKS**）：使用 Terraform 在 AWS 上部署 EKS 集群，并将此 EKS 集群与 Azure 连接，使用 Azure Arc 进行管理的示例。

+   **Google Kubernetes Engine**（**GKE**）：使用 Terraform 在 Google Cloud 上部署 GKE 集群，并将此 EKS 集群与 Azure 连接，使用 Azure Arc 进行管理的示例。

+   **Rancher k3s**：在 Azure 虚拟机或 VMware 上部署 Rancher k3s 轻量级 Kubernetes 的示例（例如，用于边缘计算、物联网或嵌入式 Kubernetes），并将集群通过 Azure Arc 进行接入。

+   **Azure Red Hat OpenShift**（**ARO**）**V4**：使用 Terraform 部署新的 ARO 集群，并将集群通过 Azure Arc 进行接入的示例。

+   **Kubernetes in Docker**（**kind**）：使用 kind 在本地机器上创建 Kubernetes 集群的示例（通过 Docker 容器 "节点" 作为本地 Kubernetes 集群运行），并将集群作为 Azure Arc 启用的 Kubernetes 集群进行接入。

+   **MicroK8s**：使用 MicroK8s 在本地机器上创建 Kubernetes 集群，并将集群作为 Azure Arc 启用的 Kubernetes 集群进行接入的示例演练。

+   **Azure Arc 启用 Kubernetes—第二天场景和使用案例**：在 Kubernetes 集群通过 Azure Arc 投影到 Azure 后，这些指南展示了如何使用原生 Azure 管理工具（如 Azure Monitor、Azure Policy 和 GitOps 配置）将这些集群作为原生 Azure 资源进行管理。例如，对于 AKS（也适用于 GKE），您可以部署 GitOps 配置，并在 AKS 上执行基本的和基于 Helm 的 GitOps 流程作为 Azure Arc 连接集群。您还可以使用 Azure Policy for Kubernetes 在 AKS 上应用 GitOps 配置，并将 Azure Monitor for containers 集成到作为 Azure Arc 连接集群的 AKS 中。

### Azure Arc 启用的数据服务（目前处于预览阶段）

本节包含了 Azure Arc 启用的数据服务的部署选项。目标是快速创建新的 Kubernetes 集群，并部署已准备好在 Azure Arc 中进行投影并使用 Azure 本地工具进行管理的 Azure Arc 启用的数据服务。如果您已经有了 Kubernetes 集群，您可以使用以下信息将 Azure Arc 启用的数据服务部署到现有 Kubernetes 集群：[`docs.microsoft.com/azure/azure-arc/data/create-data-controller`](https://docs.microsoft.com/azure/azure-arc/data/create-data-controller)：

+   **在 AKS 上的数据服务**：创建 AKS 集群并在该集群上部署 Azure Arc 数据服务的示例演练。例如，使用 Azure Arc 数据控制器 Vanilla、Azure SQL 托管实例以及 Azure PostgreSQL Hyperscale 部署在 AKS 上，配合 Azure ARM 模板：[`github.com/microsoft/azure_arc/tree/main/azure_arc_data_jumpstart/aks/arm_template`](https://github.com/microsoft/azure_arc/tree/main/azure_arc_data_jumpstart/aks/arm_template)。

+   **在 AWS Elastic Kubernetes 上的数据服务**：创建 EKS 集群并在该集群上部署 Azure Arc 数据服务的示例演练。

+   **在 GCP Google Kubernetes 上的数据服务**：创建 GKE 集群并在该集群上部署 Azure Arc 数据服务的示例演练。

+   `Kubeadm`：创建单节点 Kubernetes 集群并在该集群上部署 Azure Arc 数据服务的示例演练。

GitHub 仓库提供了极好的快速入门文档。它涵盖了各种场景，并且他们总是在寻找更多的贡献者，如果您有兴趣的话。

在我们概述 Azure Arc 时，已经覆盖了很多内容。希望这些部分为您开始概念验证项目提供了一个坚实的起点。现在我们必须将注意力转向 Azure Stack，这是 Azure 混合和多云战略中的另一个关键组成部分。

## 使用 Azure Stack 现代化您的数据中心

Azure Stack 解决方案家族可以将 Azure 服务和能力扩展到您的所有位置，如本地数据中心、远程办公室和边缘设备。您还可以在各个位置和环境边界内一致地创建和运行混合应用程序，以满足多样化工作负载的需求。

Azure Stack 是三个产品的家族：

+   **Azure Stack Hub**：一种将 Azure 云服务带到本地的云原生集成系统

+   **Azure Stack HCI**：一种通过使用现代软件定义存储和网络，并结合 Hyper-V 进行计算，来现代化数据中心的超融合基础设施

+   **Azure Stack Edge**：一种云管理的边缘设备，用于运行 AI/ML、物联网解决方案和边缘计算工作负载

这三种 Azure Stack 解决方案可以按如下方式定位：

1.  Azure Stack Hub：

    +   私有、自治云——支持完全断开连接的场景以及连接的混合云场景，同时保持与 Azure 的操作一致性

    +   应用现代化（云原生应用）

    +   数据主权、合规性和监管场景

    +   连接和断开连接或隔离的场景

1.  Azure Stack HCI：

    +   与 Azure 集成的混合和超融合解决方案

    +   现代化本地架构，消除 SAN 存储的复杂性

    +   可扩展的虚拟化、存储和网络

    +   高性能工作负载

    +   远程和分支办公室

1.  Azure Stack Edge：

    +   作为服务的 Azure 管理设备

    +   硬件加速计算、AI/ML 和物联网应用

    +   低延迟工作负载

    +   云存储网关功能

本章重点讨论 Azure Stack Hub 和 Azure Stack HCI。有关 Azure Stack Edge 的更多信息，请访问：[`azure.microsoft.com/products/azure-stack/edge/`](https://azure.microsoft.com/products/azure-stack/edge/)。

现在我们已经简要回顾了三种 Azure Stack 解决方案，接下来让我们详细讨论其中的一种变体，Azure Stack Hub。

### Azure Stack Hub 概述

Azure Stack Hub 将 Azure 服务带到本地，部署在您的数据中心，它是 Azure 的扩展。通过在其数字资产中使用相同的云平台，您的组织可以自信地让业务需求推动技术决策，而不是让技术限制影响业务决策。

### 为什么使用 Azure Stack Hub？

虽然原生 Azure 为开发人员提供了构建现代应用的全面服务，但延迟、间歇性连接、数据法规和合规性问题可能会给基于云的应用带来挑战。为了解决这些问题，Azure 和 Azure Stack Hub 为应用程序开辟了新的混合云场景，无论是面向客户还是内部使用：

+   **本地云应用**：使用 Azure 服务和基于容器的微服务及无服务器架构，您可以扩展现有应用或创建新应用，以利用云的优势并保持操作一致性。Azure Stack Hub 和本地云 Azure 之间的一致性允许使用一套 DevOps 流程，帮助加速应用现代化，并构建强大的关键任务应用。

+   **云应用合规性**：Azure Stack Hub 允许您将本地要求与云的好处相结合。您可以在 Azure 环境中创建和开发应用，然后在本地使用 Azure Stack Hub 部署，无需修改代码即可满足监管和合规性要求。示例包括费用报告、财务报告、外汇交易和全球审计等应用。

+   **间歇性或无连接**：您可以在只有间歇性甚至没有连接到 Azure 或互联网的情况下使用 Azure Stack Hub。例如远程生产现场、船只和军事应用等。数据可以在 Azure Stack Hub 安装中处理，然后在方便的时间汇总到 Azure，以进行额外的分析，避免延迟或持续连接的问题。

### Azure Stack Hub 的使用案例

在众多可能性中，我们列出了来自六大行业的精选示例：

1.  金融服务：

    +   使用 Azure 服务和容器支持的微服务架构，现代化关键任务应用。

    +   在本地运行云应用并保持数据、应用和身份安全，从而满足监管要求，同时简化操作。

    +   利用实时洞察，降低风险，避免延迟，并通过运行在本地 HCI 上的 AI 和其他应用来提升客户体验。

1.  政府：

    +   使用预测性维护管理车队，并通过结合机器学习的物联网解决方案提高建筑物的能效。

    +   通过更好的本地应用和数据库性能提升市民服务，包括对传统应用的支持。

    +   使用一致的工具集进行应用和数据管理，确保合规并增强治理。

1.  制造业：

    +   使用混合云能力，通过运行 Azure 服务而无需常驻互联网连接，提高生产力和效率。

    +   通过边缘计算中的人工智能应用提高员工安全，及时提醒潜在危险，并通过预测性维护避免机器故障。

    +   实时监控生产阶段的输出，以提高质量并减少缺陷和损坏。

1.  零售业：

    +   通过更智能地使用数据提升客户满意度，包括分析促销活动和各零售店本地的客户兴趣。

    +   通过监控库存和购买速度，优化产品的可用性，及时重新订购合适数量的产品。

    +   通过来自视频和应用数据的实时智能，减少由于缩水、盗窃、退货欺诈或其他库存问题造成的损失。

1.  能源：

    +   在与互联网未连接的活动和区域中使用必要的云服务，例如远程勘探站点和电网。

    +   减少并避免昂贵的设备故障，更快解决远程站点问题，并在影响员工安全之前发现并修复问题。

    +   在本地存储和处理数据，以便立即优化油井、炼油厂、电站、风电场等的输出和操作。

1.  健康：

    +   现代化传统系统，帮助保护患者免受系统故障的影响，同时在类似云环境和超融合环境中集成医疗设备和应用。

    +   通过将基础设施迁移到灵活的云配置中，改善临床环境和资源利用率，包括手术室表现和病房入住率。

    +   通过本地聚合、处理和医疗记录存储在可扩展、容器化的基础设施中，优化数据分析和健康记录管理。

### Azure Stack Hub 的应用、PaaS 和 IaaS 级别使用

Azure Stack Hub 在这三个传统的云资源层级中均有应用：

+   **应用层级**：Azure Stack Hub 可以通过现代的 DevOps 实践支持您的应用部署和运营。DevOps 团队可以通过使用基础设施即代码、**持续集成/持续部署**（**CI/CD**）、与 Azure 一致的虚拟机扩展和其他功能，最大化生产力和成果。

+   **平台即服务（PaaS）级别**：Azure Stack Hub 也是一个用于构建和运行需要本地 PaaS 服务（如 Event Hubs 和 Web 应用）的应用的平台。这些服务在 Azure Stack Hub 中的可用方式与在 Azure 中相同，提供统一的混合开发和运行环境。

+   **基础设施即服务（IaaS）层**：Azure Stack Hub 启用强隔离、自服务资源，具有详细的使用跟踪和多租户使用报告。这使其成为企业私有云和服务提供商的理想 IaaS 解决方案。Azure Stack Hub 本质上是一个 IaaS 平台，我们已经探讨了[`azure.microsoft.com/blog/azure-stack-iaas-part-one/`](https://azure.microsoft.com/blog/azure-stack-iaas-part-one/)系列文章。

### Azure Stack Hub 架构

Azure Stack Hub 集成系统被打包为 4 到 16 台服务器的组（称为**扩展单元**），在由受信任的硬件合作伙伴构建后交付到您的数据中心。集成系统与其硬件和软件提供了一个解决方案，提供了云端创新的速度并简化了 IT 管理。Azure Stack Hub 使用行业标准硬件，并启用了与 Azure 订阅相同的管理工具，因此您可以独立于任何与 Azure 的连接，使用一致的 DevOps 流程。Azure Stack Hub 集成系统由微软和硬件合作伙伴共同支持。

### Azure Stack Hub 身份提供者

Azure Stack Hub 使用以下两种身份提供者之一：Azure AD 或**Active Directory 联合身份验证服务**（**AD FS**）。许多互联网连接的混合配置使用 Azure AD。另一方面，断开连接的部署则需要使用 AD FS。Azure Stack Hub 资源提供者及其他应用在 Azure AD 或 AD FS 下的功能类似。请注意，Azure Stack Hub 具有其自身的 Active Directory 实例和 Active Directory 图形 API。

### 如何管理 Azure Stack Hub？

与微软通过 Azure 服务向租户提供服务的方式类似，您可以使用 Azure Stack Hub 操作为租户用户提供不同的服务和应用。这是因为 Azure 和 Azure Stack Hub 使用相同的操作模型。

Azure Stack Hub 引入了一个新的角色，称为**操作员**。这是一个管理员级别的功能，用于管理、监控和配置 Azure Stack Hub。它是 Azure Stack Hub 环境中的关键角色，要求具备广泛的技能和知识——这些都在 Microsoft Certified: Azure Stack Hub Operator Associate（[`docs.microsoft.com/learn/certifications/azure-stack-hub-operator?WT.mc_id=Azure_blog-wwl`](https://docs.microsoft.com/learn/certifications/azure-stack-hub-operator?WT.mc_id=Azure_blog-wwl)）认证和 AZ-600 考试：使用 Microsoft Azure Stack Hub 配置和操作混合云（[`docs.microsoft.com/learn/certifications/exams/az-600?WT.mc_id=Azure_blog-wwl`](https://docs.microsoft.com/learn/certifications/exams/az-600?WT.mc_id=Azure_blog-wwl)）中有所体现。

为了准备这次考试，我们已经发布了一系列资料（包括 Azure Stack Hub 基础核心 [`github.com/Azure-Samples/Azure-Stack-Hub-Foundation-Core/tree/master/ASF-Training`](https://github.com/Azure-Samples/Azure-Stack-Hub-Foundation-Core/tree/master/ASF-Training) 资料），这些资料在 TechCommunity 博客中列出：[`techcommunity.microsoft.com/t5/azure-stack-blog/azure-stack-hub-operator-certification-az-600/ba-p/2024434`](https://techcommunity.microsoft.com/t5/azure-stack-blog/azure-stack-hub-operator-certification-az-600/ba-p/2024434)。

Azure Stack Hub 可以通过三种选项进行管理：

+   管理员门户

+   用户门户

+   PowerShell

管理员门户可用于在 Azure Stack Hub 上执行管理操作，如集成系统的状态监控或健康维护、添加市场项目、增加容量以及添加新的资源提供程序以启用新的 PaaS 服务。Azure Stack Hub 管理门户快速入门（[`docs.microsoft.com/azure-stack/operator/azure-stack-manage-portals`](https://docs.microsoft.com/azure-stack/operator/azure-stack-manage-portals)）包含了更多关于使用管理员门户管理 Azure Stack Hub 的信息。

Azure Stack Hub 快速入门模板也可提供示例，用于部署资源，从简单的 VM 安装到更复杂的部署，如 Exchange 和 SharePoint。

作为 Azure Stack Hub 操作员，您可以为用户启用各种资源类型。例如，这可以包括 SQL 和 MySQL 服务器，以及 VM 自定义映像、应用服务、Azure 函数、事件中枢等。作为操作员，您还可以管理容量问题，为租户创建使用优惠和订阅，以及响应警报。

用户可以利用用户门户的自助服务功能来消费云资源，如 Web 应用程序、存储帐户和虚拟机。用户使用由操作员提供的服务，并且可以创建、监控和管理他们订阅的服务，如存储、Web 应用程序和虚拟机。用户可以选择使用用户门户或 PowerShell 管理其环境。

Azure Stack Hub 提供多租户环境。这使得各种服务提供商（包括云解决方案提供商、托管服务提供商和**独立软件供应商**（**ISV**））能够在 Azure Stack Hub 平台上构建和提供价值，并将其交付给多个客户，每个客户在其自己的 Azure Stack Hub 订阅中都是独立和安全的。

### 资源提供程序

资源提供程序是构成所有 Azure Stack Hub IaaS 和 PaaS 服务基础的服务。Azure Stack Hub 有三个**基础 IaaS 资源提供程序**：

+   **计算资源提供程序**：这允许您的 Azure Stack Hub 租户创建虚拟机。使用此提供程序，可以创建 VM 和 VM 扩展。

+   **网络资源提供者**：支持创建网络安全组、公共 IP、虚拟网络和软件负载均衡器。

+   **存储资源提供者**：支持创建 Azure Blob、表和队列存储服务。Azure 密钥保管库（用于创建和管理密钥）也通过此资源提供者提供支持。

您还可以与 Azure Stack Hub 一起部署并使用以下 **可选的 PaaS 资源提供者**：

+   **应用服务**：一项 PaaS 服务，允许客户为任何设备或平台创建 Web、API 和 Azure Functions 应用。您的内外部客户可以自动化他们的业务流程，并将您的应用与他们的本地应用集成。这些客户应用可以由 Azure Stack Hub 云操作员在共享或专用虚拟机上运行，且完全托管。

+   **事件中心**：Azure Stack Hub 上的事件中心让您能够实现混合云场景。支持基于流和事件的解决方案，适用于本地和 Azure 云处理。无论您的场景是混合（连接）还是断开连接，您的解决方案都可以支持大规模事件/流的处理。您的场景仅受集群大小的限制，您可以根据需要配置集群。（来源：[`docs.microsoft.com/azure/event-hubs/event-hubs-about`](https://docs.microsoft.com/azure/event-hubs/event-hubs-about)）

+   **物联网中心（预览）**：Azure Stack Hub 上的物联网中心让您能够创建混合物联网解决方案。物联网中心是一个托管服务，充当您物联网应用和其管理的设备之间双向通信的中央消息中心。您可以使用 Azure Stack Hub 上的物联网中心构建具有可靠且安全的通信的物联网解决方案，实现物联网设备与本地解决方案后端之间的通信。（来源：[`github.com/MicrosoftDocs/azure-docs/blob/master/articles/iot-hub/about-iot-hub.md`](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/iot-hub/about-iot-hub.md)）

+   **SQL 服务器**：此功能允许您通过提供连接器将 SQL 数据库作为服务提供给您的 Azure Stack Hub 租户。

+   **MySQL 服务器**：此功能使您可以通过提供连接器将 MySQL 数据库作为 Azure Stack Hub 服务提供。

### Azure Stack Hub 管理基础

有几个 Azure Stack Hub 管理的基本方面是您必须了解的，包括理解构建、管理工具的选择、Azure Stack Hub 操作员的典型职责，以及需要与用户沟通的内容，帮助他们提高生产力。

### 了解构建

以下是您应该了解的两个与构建相关的组件：

+   **集成系统**：更新包为 Azure Stack Hub 集成系统分发 Azure Stack Hub 的更新版本。您可以通过管理员门户中的 **更新** 磁贴导入这些包并应用它们。

+   **开发工具包**：**Azure Stack 开发工具包**（**ASDK**）作为一个沙箱环境供你评估 Azure Stack Hub，并在非生产环境中构建和测试应用程序。ASDK 部署页面提供了更多关于部署的信息：[`docs.microsoft.com/azure-stack/asdk/asdk-install`](https://docs.microsoft.com/azure-stack/asdk/asdk-install)。

Azure Stack Hub 的创新迅速，定期发布新的构建版本。如果你正在运行 ASDK 并希望迁移到最新的构建版本以及 Azure Stack Hub 的最新功能，你不能简单地应用更新包——你必须重新部署 ASDK。Azure 网站上的 ASDK 文档反映了最新的发布版本：[`docs.microsoft.com/azure-stack/asdk/`](https://docs.microsoft.com/azure-stack/asdk/)。

### 保持对可用服务的关注

Azure Stack Hub 支持不断发展的 Azure 服务子集。你需要随时了解可以为 Azure Stack Hub 用户提供的服务。

### 基础服务

默认情况下，Azure Stack Hub 在部署时包括以下基础服务：

+   计算

+   存储

+   网络

+   密钥保管库

这些基础服务使你能够以最小的配置为用户提供 IaaS 服务。

### 附加服务

以下是当前支持的附加 PaaS 服务：

+   应用服务

+   Azure Functions

+   SQL 和 MySQL RP

+   事件中心

+   IoT Hub（预览版）

+   Kubernetes（预览版）

在向用户提供这些附加服务之前，需要进一步配置。欲了解更多信息，请查阅这里的*教程*和*操作指南*部分：[`docs.microsoft.com/azure-stack/operator/`](https://docs.microsoft.com/azure-stack/operator/)。

### 你可以使用哪些管理工具？

管理员门户是学习基本管理概念的最简单方式。你也可以使用 PowerShell 进行 Azure Stack Hub 管理，尽管有一些准备步骤。通过访问*在 Azure Stack Hub 上开始使用 PowerShell*，你可以了解 PowerShell 如何应用于 Azure Stack Hub：[`docs.microsoft.com/azure-stack/user/azure-stack-powershell-overview`](https://docs.microsoft.com/azure-stack/user/azure-stack-powershell-overview)。

作为底层部署、管理和组织机制，Azure Stack Hub 使用 Azure 资源管理器。欲了解更多关于资源管理器的信息，以便你管理 Azure Stack Hub 并帮助支持用户，请访问*开始使用 Azure 资源管理器白皮书*：[`download.microsoft.com/download/E/A/4/EA4017B5-F2ED-449A-897E-BD92E42479CE/Getting_Started_With_Azure_Resource_Manager_white_paper_EN_US.pdf`](https://download.microsoft.com/download/E/A/4/EA4017B5-F2ED-449A-897E-BD92E42479CE/Getting_Started_With_Azure_Resource_Manager_white_paper_EN_US.pdf)。

### 典型的管理员/提供者职责

用户认为 Azure Stack Hub 操作员角色是为他们提供所需服务。因此，决定哪些服务将被提议，然后创建计划、优惠和配额来使这些服务可用。另请参阅 *Azure Stack Hub 服务提供概览*：[`docs.microsoft.com/azure-stack/operator/service-plan-offer-subscription-overview`](https://docs.microsoft.com/azure-stack/operator/service-plan-offer-subscription-overview)。

你还需要向 Azure Stack Hub Marketplace 添加项目。从 Azure 下载 Marketplace 项目到 Azure Stack Hub 是最简单的方法。

#### 注意事项

你应该使用用户门户，而不是管理员门户，来测试你的服务的用户可用性以及这些服务的计划和优惠。

除了提供服务之外，还有定期的操作员任务，以确保 Azure Stack Hub 持续正常运行。这些任务包括：

+   为 Azure AD 部署创建用户帐户（[`docs.microsoft.com/azure-stack/operator/azure-stack-add-new-user-aad`](https://docs.microsoft.com/azure-stack/operator/azure-stack-add-new-user-aad)）或为 AD FS 部署创建用户帐户（[`docs.microsoft.com/azure-stack/operator/azure-stack-add-users-adfs`](https://docs.microsoft.com/azure-stack/operator/azure-stack-add-users-adfs)）。

+   RBAC 角色分配（不限于管理员）（[`docs.microsoft.com/azure-stack/operator/azure-stack-manage-permissions`](https://docs.microsoft.com/azure-stack/operator/azure-stack-manage-permissions)）。

+   基础设施健康监控（[`docs.microsoft.com/azure-stack/operator/azure-stack-monitor-health`](https://docs.microsoft.com/azure-stack/operator/azure-stack-monitor-health)）。

+   网络管理（[`docs.microsoft.com/azure-stack/operator/azure-stack-viewing-public-ip-address-consumption`](https://docs.microsoft.com/azure-stack/operator/azure-stack-viewing-public-ip-address-consumption)）和存储资源管理（[`docs.microsoft.com/azure-stack/operator/azure-stack-manage-storage-accounts`](https://docs.microsoft.com/azure-stack/operator/azure-stack-manage-storage-accounts)）。

+   更换故障硬件，例如损坏的磁盘（[`docs.microsoft.com/azure-stack/operator/azure-stack-replace-disk`](https://docs.microsoft.com/azure-stack/operator/azure-stack-replace-disk)）。

### 与用户沟通

用户需要知道如何连接到 Azure Stack Hub 环境，如何订阅优惠，并且如何使用 Azure Stack Hub 服务。Azure Stack Hub 用户文档（[`docs.microsoft.com/azure-stack/user/`](https://docs.microsoft.com/azure-stack/user/)）是一个有用的信息来源。

### 了解如何使用 Azure Stack Hub 服务

在用户开始使用 Azure Stack Hub 中的服务并构建应用程序之前，有一些先决条件，如特定版本的 PowerShell 和 API。同时，Azure 服务和相应的 Azure Stack Hub 服务之间也存在一些功能差异。因此，用户应熟悉以下内容：

+   使用服务或为 Azure Stack Hub 构建应用的关键注意事项：[`docs.microsoft.com/azure-stack/user/azure-stack-considerations`](https://docs.microsoft.com/azure-stack/user/azure-stack-considerations )

+   Azure Stack Hub 中虚拟机的注意事项：[`docs.microsoft.com/azure-stack/user/azure-stack-vm-considerations`](https://docs.microsoft.com/azure-stack/user/azure-stack-vm-considerations )

+   Azure Stack Hub 存储的差异和注意事项：[`docs.microsoft.com/azure-stack/user/azure-stack-acs-differences`](https://docs.microsoft.com/azure-stack/user/azure-stack-acs-differences )

这三篇文章中的信息概述了 Azure 和 Azure Stack Hub 服务之间的差异。它是针对某一 Azure 服务的全球 Azure 文档的附加信息。

### 用户连接到 Azure Stack Hub 的方式

在 ASDK 的情况下，如果用户没有使用远程桌面，他们可以配置**虚拟专用网络**（**VPN**）连接以连接到 ASDK 环境。更多信息请参见：[`docs.microsoft.com/azure-stack/asdk/asdk-connect`](https://docs.microsoft.com/azure-stack/asdk/asdk-connect)。

用户还需要了解如何通过用户门户（[`docs.microsoft.com/azure-stack/user/azure-stack-use-portal`](https://docs.microsoft.com/azure-stack/user/azure-stack-use-portal)）或使用 PowerShell 进行连接。在集成系统环境中，用户门户的 URL 会有所不同，因此务必向用户提供正确的地址。

如果门户已发布到互联网，用户也可以使用 Cloud Shell 等工具来管理和创建 Azure Stack Hub 上的资源（请注意，Cloud Shell 服务本身在 Azure Stack Hub 上不可用——你需要处于连接环境中）。

在通过 PowerShell 使用服务之前，用户可能需要注册资源提供者，例如管理负载均衡器、网络接口和虚拟网络等资源的网络资源提供者。用户必须安装 PowerShell（[`docs.microsoft.com/azure-stack/operator/powershell-install-az-module`](https://docs.microsoft.com/azure-stack/operator/powershell-install-az-module)），下载附加模块（[`docs.microsoft.com/azure-stack/operator/azure-stack-powershell-download`](https://docs.microsoft.com/azure-stack/operator/azure-stack-powershell-download)），并配置 PowerShell（其中包括资源提供者注册）：[`docs.microsoft.com/azure-stack/user/azure-stack-powershell-configure-user`](https://docs.microsoft.com/azure-stack/user/azure-stack-powershell-configure-user)。

### 订阅优惠

用户在使用服务之前，还必须订阅运营商创建的优惠（[`docs.microsoft.com/azure-stack/operator/azure-stack-subscribe-plan-provision-vm`](https://docs.microsoft.com/azure-stack/operator/azure-stack-subscribe-plan-provision-vm)）。优惠是一个或多个计划的组合，使 Azure Stack Hub 运营商能够控制诸如试用优惠、容量规划等内容，甚至将 Azure Stack Hub 订阅的创建委托给其他用户。

### 获取支持

Azure Stack Hub 涵盖许多领域。在联系 Microsoft 支持之前，请查看此网页。它提供了常见问题和故障排除链接：[`docs.microsoft.com/azure-stack/operator/azure-stack-servicing-policy`](https://docs.microsoft.com/azure-stack/operator/azure-stack-servicing-policy)。

除了上述问题外，还有一些额外的支持考虑事项。

### 集成系统

集成系统受益于 Microsoft 与 Microsoft 的**原始设备制造商**（**OEM**）硬件合作伙伴之间协调的升级和解决过程。

如果出现云服务问题，Microsoft 支持可以提供帮助。要打开支持请求，请在管理员门户的右上角选择帮助和支持图标（问号）。然后选择帮助 + 支持，再选择支持部分下的“新建支持请求”。

如果部署、补丁、更新、硬件（包括可更换单元）或任何硬件品牌的软件（如在硬件生命周期主机上运行的软件）出现问题，您的 OEM 硬件供应商是您的首要联系人。

对于任何其他问题，您应联系 Microsoft 支持。

### Azure Stack 开发工具包

ASDK 是 Azure Stack Hub 的单节点部署，您可以免费下载并使用。所有 ASDK 组件都安装在运行在单一主机计算机上的虚拟机中，该计算机必须有足够的资源。ASDK 旨在提供一个环境，您可以在其中评估 Azure Stack Hub，并使用与 Azure 一致的 API 和工具开发现代应用程序，但这是一个非生产环境。

由于 ASDK 是免费的产品，因此不提供正式支持。您可以在定期监控的微软论坛中提问有关 ASDK 的支持问题（[`social.msdn.microsoft.com/Forums/azure/home?forum=azurestack`](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)）。要访问论坛，请在管理员门户的右上角选择帮助和支持图标（问号），然后选择**帮助 + 支持**，再选择**支持**部分下的**MSDN 论坛**。但是，微软支持不提供官方支持，因为 ASDK 是一个评估环境。

### Azure Stack Hub 更新管理

您可以通过应用完整和快速更新、热修复以及 OEM 提供的驱动程序和固件更新来帮助保持 Azure Stack Hub 的最新状态。然而，请记住，Azure Stack Hub 更新包仅适用于集成系统，不能应用于 ASDK，对于 ASDK，需要重新部署。有关重新部署的信息，请参见 [`docs.microsoft.com/azure-stack/asdk/asdk-redeploy`](https://docs.microsoft.com/azure-stack/asdk/asdk-redeploy) 中的*重新部署 ASDK*。

### 更新包类型

对于集成系统，有三种不同类型的更新包：

+   **Azure Stack Hub 软件更新**：这些更新直接从微软下载，可能包括 Azure Stack Hub 功能更新、Windows Server 安全更新和非安全相关的更新。每个更新包可以是**完整**或**快速**类型：

    +   完整更新包会更新规模单元中的物理主机操作系统。它们需要更长的维护窗口。

    +   快速更新包是有针对性的，不会更新底层的物理主机操作系统。

+   **Azure Stack Hub 热修复**：这些时间敏感的更新解决了特定问题。每个热修复都会发布相应的微软知识库文章，详细说明问题、原因和解决方法。热修复的下载和安装方式与完整更新包类似。您可以在此处阅读更多信息：[`docs.microsoft.com/azure-stack/operator/azure-stack-servicing-policy#hotfixes`](https://docs.microsoft.com/azure-stack/operator/azure-stack-servicing-policy#hotfixes)。

+   **OEM 硬件供应商提供的更新**：这些更新由每个硬件供应商管理。它们通常包含驱动程序和固件更新，每个供应商将这些更新托管在自己的网站上。

### 何时更新

上述更新以不同的频率发布：

+   **Azure Stack Hub 软件更新**：微软每年发布多次完整和快速软件更新包。

+   **Azure Stack Hub 热修复**：这些是时间敏感的，可以随时发布。当您从一个主要版本升级到另一个版本时，如果新版本中有任何热修复，它们会自动安装。

+   **OEM 硬件供应商提供的更新**：这些更新由 OEM 硬件供应商根据需要发布。

如果你希望继续获得支持，必须将 Azure Stack Hub 环境保持在受支持的 Azure Stack Hub 软件版本上。有关更多信息，请访问此链接阅读 Azure Stack Hub 服务政策：[`docs.microsoft.com/azure-stack/operator/azure-stack-servicing-policy`](https://docs.microsoft.com/azure-stack/operator/azure-stack-servicing-policy)。

### 检查可用的更新

更新通知依赖于诸如互联网连接和更新类型等因素：

+   对于 Microsoft 发布的软件更新和热修复，你将在**更新**面板中看到 Azure Stack Hub 实例的更新警报（仅限已连接互联网的实例）。如果未显示此面板，请重新启动基础架构管理控制器虚拟机。对于未连接的实例，你可以订阅 RSS 订阅源（[`azurestackhubdocs.azurewebsites.net/xml/hotfixes.rss`](https://azurestackhubdocs.azurewebsites.net/xml/hotfixes.rss)），以接收每次热修复发布的通知。

+   来自 OEM 硬件供应商的更新通知取决于你与制造商的沟通。有关更多信息，请访问 Azure Stack Hub OEM 更新页面：[`docs.microsoft.com/azure-stack/operator/azure-stack-update-oem`](https://docs.microsoft.com/azure-stack/operator/azure-stack-update-oem)。

### 从一个主要版本更新到下一个版本

从一个主要版本更新到下一个主要版本必须按照正确的顺序和步骤进行，不能跳过任何一个主要版本的更新。有关详细信息，请查看此链接：[`docs.microsoft.com/azure-stack/operator/azure-stack-updates#how-to-know-an-update-is-available`](https://docs.microsoft.com/azure-stack/operator/azure-stack-updates#how-to-know-an-update-is-available)。

### 主要版本中的热修复

在一个主要版本发布中，可能会有多个热修复。这些修复是累积的，最后一个热修复包包含该主要版本的所有前期热修复。有关更多信息，请参阅[`docs.microsoft.com/azure-stack/operator/azure-stack-servicing-policy#hotfixes`](https://docs.microsoft.com/azure-stack/operator/azure-stack-servicing-policy#hotfixes)。

### 更新资源提供者

Azure Stack Hub 通过更新资源提供者处理 Microsoft 软件更新的应用。该提供者验证更新是否已应用到所有 Service Fabric 应用及其运行时、所有物理主机、所有虚拟机及其相关服务。

在更新过程中，你可以在 Azure Stack Hub 中查看到一个高层次的状态概览，在其中你可以看到不同子系统的进展情况。

我们已经介绍了 Azure Stack Hub 的基础知识。接下来，我们将介绍 Azure Stack HCI，这是一种 HCI 集群解决方案。

## Azure Stack HCI 解决方案概述

Azure Stack HCI 旨在托管经过验证的合作伙伴硬件上的虚拟化 Linux 和 Windows 工作负载。它专为混合环境（本地和云）设计，Azure Stack HCI 与 Azure 混合服务相结合，提供诸如灾难恢复和虚拟机备份选项等功能，还可以实现基于云的监控。你还可以通过单一的管理界面查看所有部署，并通过 Windows 管理中心、系统中心和 PowerShell 等工具管理集群。此外，Azure Stack HCI 上还提供了 AKS。这是 AKS 编排器的本地实现——详细信息请见 [`docs.microsoft.com/azure-stack/aks-hci/overview`](https://docs.microsoft.com/azure-stack/aks-hci/overview)。

Azure Stack HCI 软件可以在这里下载：[`azure.microsoft.com/products/azure-stack/hci/hci-download/`](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)。它是一个带有内置混合云连接的本地解决方案，作为 Azure 服务计费到你的 Azure 订阅中。  

### Azure Stack HCI 的应用场景  

Azure Stack HCI 有许多应用场景，下面我们将讨论这些场景。  

### 数据中心整合与现代化  

使用 Azure Stack HCI 来刷新较旧的虚拟化主机并进行可能的整合，有几个潜在的好处。它可以增强可扩展性，简化管理和提高安全性。它还可以通过替换传统的 SAN 存储来减少占地面积和总体拥有成本。统一的工具和界面，以及单一的支持点，简化了系统管理和操作。  

### 分支机构的混合基础设施管理  

对于拥有分支机构的企业来说，管理混合基础设施可能会面临挑战。当多个地点必须在没有专职 IT 员工的情况下运行时，保持身份服务同步、进行数据备份和部署应用程序会变得更加复杂。在这种情况下，企业通常需要数周或数月的时间才能在多个办事处和基础设施中推出应用程序更新。因此，需要更好的解决方案，以便快速、轻松地在远程办事处部署应用和身份更改，同时实现对异常和违规行为的集中监控。  

以一家拥有 300 个全球办事处的国际银行为例，每次全球更新所有办事处都需要一年时间。更复杂的是，多个地点使得始终如一地修正配置和避免潜在的安全风险（如开放端口）变得困难。对拥有大量办事处的公司来说，向分支机构推出新的和更新的应用程序也是一大难题，特别是当远程站点因延迟或网络连接受限，需要在本地服务器上运行应用程序时。  

为帮助您克服这些挑战，Azure Stack HCI 提供基于行业标准 x86 服务器的软件定义计算、存储和网络。使用内置于 Windows 管理中心的 Azure Arc 集成，您可以轻松开始使用 Azure 门户云来管理您的 HCI。通过减少或消除对本地 IT 员工的需求，您可以满足分支办公室、零售店和现场位置日益增长的 IT 需求，随时部署基于容器的应用程序以及高度可用的虚拟机中的关键业务应用程序。然后，Azure 监控可以让您查看这些不同领域的系统健康状况。

### 远程和分支办公室

Azure Stack HCI 提供了一种经济实惠的方式来现代化远程和分支办公室，包括零售店和现场站点（目前可用的双服务器集群解决方案，每个位置的费用不到 $20,000）。嵌套的容错性意味着即使发生多重硬件故障，卷也可以保持在线并可访问。云见证技术使得可以使用 Azure 作为轻量级的集群仲裁者。这可以防止分脑现象发生，同时不涉及第三台主机的成本。在 Azure 门户中，您可以看到远程 Azure Stack HCI 部署的集中视图。

### 虚拟桌面

Azure Stack HCI 为您的本地虚拟桌面提供类似本地的性能。如果您需要支持用户的数据主权以及低延迟，这非常理想。

### Azure 集成的好处

Azure Stack HCI 让您通过混合基础设施利用云和本地资源的结合，提供原生云监控、安全性和备份。

首先，您的 Azure Stack HCI 集群必须与 Azure 注册。然后，您将能够利用 Azure 门户进行：

+   **监控**：您可以查看 Azure Stack HCI 集群的总体情况。在这里，您可以按资源组对集群进行分组并标记。即将发布的新功能也将支持从门户中创建和管理虚拟机。

+   **计费**：使用您的 Azure 订阅来支付 Azure Stack HCI（免费预览阶段后）。

+   **支持**：通过标准或专业直接 Azure 支持计划提供 Azure Stack HCI 支持。

这些额外的 Azure 混合服务也可供订阅：

+   Azure 站点恢复提供 **灾难恢复即服务** (**DRaaS**) 和高可用性。

+   Azure 监控，提供应用、基础设施和网络事件的集中视图，包括由 AI 驱动的高级分析。

+   云见证，利用 Azure 作为轻量级的集群仲裁者。

+   Azure 备份通过将数据存储到异地来保护数据，同时提供对勒索病毒的防护。

+   Azure 更新管理，用于评估和部署 Azure 托管和本地 Windows 虚拟机的更新。

+   Azure 网络适配器，使用点对站点 VPN 来连接您的 Azure 虚拟机与本地资源。

+   Azure 文件同步，用于文件服务器的云同步。

另请参见 *将 Windows Server 连接到 Azure 混合服务*：[`docs.microsoft.com/windows-server/manage/windows-admin-center/azure/index`](https://docs.microsoft.com/windows-server/manage/windows-admin-center/azure/index)。

### Azure Stack HCI 前提条件

最初，您将需要：

+   至少选择两个服务器，来自 Azure Stack HCI 目录和您选择的 Microsoft 硬件合作伙伴，组成集群。

+   Azure 订阅。

+   每个集群中的服务器每 30 天或更频繁需要进行 HTTPS 外发传输的互联网连接。

如果您的集群跨多个站点部署，您至少需要 1 Gb 的站点间连接（但最好使用 25 Gb 的 RDMA 连接），并且如果计划在两个站点同时进行同步复制并写入数据，平均往返延迟应为 5 毫秒。

如果您计划使用 **软件定义网络** (**SDN**)，还应规划为 Azure Stack HCI 操作系统创建网络控制器虚拟机（**VHD**）的过程（有关更多信息，请访问 *计划部署网络控制器* 页面：[`docs.microsoft.com/azure-stack/hci/concepts/network-controller`](https://docs.microsoft.com/azure-stack/hci/concepts/network-controller)）。

另请参见：

+   系统要求：[`docs.microsoft.com/azure-stack/hci/concepts/system-requirements`](https://docs.microsoft.com/azure-stack/hci/concepts/system-requirements)

+   Azure Stack HCI 上的 AKS 要求（适用于 Azure Stack HCI 上的 AKS）：[`docs.microsoft.com/azure-stack/aks-hci/overview#what-you-need-to-get-started`](https://docs.microsoft.com/azure-stack/aks-hci/overview#what-you-need-to-get-started)

### 硬件合作伙伴

通过从您选择的 Microsoft 合作伙伴处订购经过验证的 Azure Stack HCI 配置，您可以在没有过多设计或部署时间的情况下启动操作。实施和支持也可以通过 Microsoft 合作伙伴通过联合支持协议提供单一联系点支持。选项包括购买经过验证的节点，或者购买预安装了 Azure Stack HCI 操作系统的集成系统，配合驱动程序和固件更新合作伙伴扩展。

### 部署选项

+   **集成系统**：从硬件合作伙伴购买经过验证的服务器，并预安装 Azure Stack HCI。

+   **经过验证的节点**：从硬件合作伙伴购买经过验证的裸机服务器，然后注册 Azure Stack HCI 服务。进入 Azure 门户，您可以在其中下载 Azure Stack HCI 操作系统。

+   **重用硬件**：重新利用现有硬件。有关此路径的详细信息，请查看以下链接：[`docs.microsoft.com/azure-stack/hci/deploy/migrate-cluster-same-hardware`](https://docs.microsoft.com/azure-stack/hci/deploy/migrate-cluster-same-hardware)。

您可以在这里阅读更多关于 Azure Stack HCI 的内容：[`azure.microsoft.com/overview/azure-stack/hci`](https://azure.microsoft.com/overview/azure-stack/hci)。

您还可以查看 Azure Stack HCI 目录。它详细介绍了来自微软合作伙伴的 100 多个解决方案。您可以在此处找到在线目录：[`azure.microsoft.com/products/azure-stack/hci/catalog/`](https://azure.microsoft.com/products/azure-stack/hci/catalog/).

### 软件合作伙伴

微软合作伙伴正在开发软件，构建在 Azure Stack HCI 平台上，无需更换 IT 管理员已熟悉的工具。要查看突出显示的 ISV 及其应用程序列表，请访问此链接：[`docs.microsoft.com/azure-stack/hci/concepts/utility-applications`](https://docs.microsoft.com/azure-stack/hci/concepts/utility-applications).

### 许可、计费和定价

Azure Stack HCI 计费使用物理处理器核心数来计算每月订阅费，而不是永久许可证。连接到 Azure 时，上传并自动评估您使用的核心数，以计费。因此，更高密度的虚拟环境不会导致更高的成本，从而导致更具吸引力的定价。

### 管理工具

您拥有 Azure Stack HCI 的完全管理员集群权限，可以直接管理：

+   **故障转移群集**: [`docs.microsoft.com/windows-server/failover-clustering/failover-clustering-overview`](https://docs.microsoft.com/windows-server/failover-clustering/failover-clustering-overview )

+   **Windows Server 上的 Hyper-V**: [`docs.microsoft.com/windows-server/virtualization/hyper-v/hyper-v-on-windows-server`](https://docs.microsoft.com/windows-server/virtualization/hyper-v/hyper-v-on-windows-server )

+   **SDN**: [`docs.microsoft.com/windows-server/networking/sdn/`](https://docs.microsoft.com/windows-server/networking/sdn/)

+   **存储空间直通**: [`docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview`](https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview )

您还可以使用以下管理工具：

+   **PowerShell**: [`docs.microsoft.com/powershell/`](https://docs.microsoft.com/powershell/)

+   **服务器管理器**: [`docs.microsoft.com/windows-server/administration/server-manager/server-manager`](https://docs.microsoft.com/windows-server/administration/server-manager/server-manager )

+   **System Center**: [`www.microsoft.com/system-center`](https://www.microsoft.com/system-center)

+   **Windows Admin Center**: [`docs.microsoft.com/windows-server/manage/windows-admin-center/overview`](https://docs.microsoft.com/windows-server/manage/windows-admin-center/overview )

+   诸如 5Nine 管理器之类的非微软工具

## 开始使用 Azure Stack HCI 和 Windows Admin Center

这些部分适用于 Azure Stack HCI 版本 20H2。假设你已经在 Azure Stack HCI 安装中设置了集群，并且提供了连接到集群和监控集群及存储性能的说明。

### 安装 Windows Admin Center

Windows Admin Center 是一个本地部署的基于浏览器的应用程序，用于管理 Azure Stack HCI。Windows Admin Center 可以安装在**服务模式**的服务器上，但在本地管理 PC 上以**桌面模式**安装会更简单。在服务模式下，需要使用属于 Windows Admin Center 服务器上的 Gateway Administrators 组的帐户来执行需要 CredSSP 的任务，如创建集群和安装更新及扩展。有关更多详细信息，请查看 [`docs.microsoft.com/windows-server/manage/windows-admin-center/configure/user-access-control#gateway-access-role-definitions`](https://docs.microsoft.com/windows-server/manage/windows-admin-center/configure/user-access-control#gateway-access-role-definitions)。

### 添加并连接到 Azure Stack HCI 集群

安装 Windows Admin Center 后，你可以从 Windows Admin Center 仪表板的主概览页面添加一个集群进行管理。仪表板还会显示有关 CPU、内存和存储使用情况的信息，以及有关服务器、磁盘和卷的警报和健康信息。集群性能信息，如**输入/输出操作/秒**（**IOPS**）和延迟，可以按小时、天、周、月或年显示在仪表板底部。

### 监控单个组件

仪表板左侧的**工具**菜单允许你深入查看集群的任何组件，查看虚拟机、服务器、卷、驱动器和虚拟交换机的摘要和清单。在仪表板中，性能监视器工具可以让你实时查看、比较和添加 Windows、应用程序或设备的性能计数器。

### 查看集群仪表板

要查看集群健康和性能的仪表板信息，请在**所有连接**下选择集群名称。然后，在左侧的**工具**下选择**仪表板**。你将看到：

+   平均集群延迟（以毫秒为单位）

+   集群事件警报

+   一个列表：

    +   集群中可用的磁盘驱动器

    +   加入集群的服务器

    +   在集群上运行的虚拟机

    +   集群中可用的卷

+   总计集群：

    +   集群的 CPU 使用情况

    +   IOPS

    +   集群的内存使用情况

    +   集群的存储使用情况

有关更多信息，请访问此链接：[`docs.microsoft.com/azure-stack/hci/manage/cluster`](https://docs.microsoft.com/azure-stack/hci/manage/cluster)。

### 使用 Azure Monitor 进行监控和警报

你还可以使用 Azure Monitor 收集事件和性能计数器进行分析和报告，响应特定条件并接收电子邮件通知。通过点击**工具**菜单中的 Azure Monitor，你将直接从 Windows Admin Center 连接到 Azure。

### 收集诊断信息

**工具**菜单中的**诊断**选项卡可以让你收集集群的故障排除信息。如果你请求微软支持，这些信息也可能会被要求提供。

### 使用 Windows Admin Center 管理虚拟机

你可以使用 Windows Admin Center 在 Azure Stack HCI（Azure Stack HCI 版本 20H2 和 Windows Server 2019）上启动并管理你的虚拟机。功能包括：

+   创建新的虚拟机。

+   列出服务器或集群中的虚拟机。

+   查看虚拟机详细信息，包括来自给定虚拟机专用页面的详细信息和性能图表。

+   查看聚合虚拟机指标，查看集群中所有虚拟机的总体资源使用和性能。

+   更改虚拟机的内存、处理器、磁盘大小等设置（注意，在更改某些设置之前，您需要先停止虚拟机）。

附加功能包括：

+   将虚拟机迁移到另一台服务器或集群

+   将虚拟机加入域

+   克隆虚拟机

+   导入和导出虚拟机

+   查看虚拟机导出日志

+   使用 Azure Site Recovery 保护虚拟机（一个可选的增值服务）

请注意，你还可以通过其他方式连接到虚拟机，而不仅仅是通过 Windows Admin Center：

+   通过使用**远程桌面协议**（**RDP**）连接到 Hyper-V 主机

+   通过使用 Windows PowerShell

你还可以为集群进行 Storage Spaces Direct 更改，并更改一般集群设置。这些可以包括：

+   集群见证者

+   节点关机行为

+   设置和管理访问点

+   流量加密

+   虚拟机负载均衡

你还可以通过 Azure Monitor 监控你的 Azure HCI 集群。

## 比较 Azure Stack Hub 和 Azure Stack HCI

Azure Stack HCI 和 Azure Stack Hub 都可以在你的混合云和多云战略中发挥关键作用。以下是它们差异的简要总结：

+   **Azure Stack Hub**：这允许你在本地运行云应用程序，无论是断开连接时，还是为了满足合规性要求，利用一致的 Azure 服务。

+   **Azure Stack HCI**：这允许你在本地运行虚拟化应用程序，加强并替换老化的服务器基础设施，并连接到 Azure 以提供云服务。

Azure Stack Hub 提供创新的流程，尽管可能需要新的技能。它将 Azure 服务带入你的数据中心。另一方面，使用 Azure Stack HCI，你可以使用现有的技能和熟悉的流程，将你的数据中心连接到 Azure 服务。比较 Azure Stack Hub 和 Azure Stack HCI 无法做到的事情或可能的限制可能也很有帮助。

Azure Stack Hub 的限制：

+   Azure Stack Hub 必须至少有 4 个节点，并且必须有自己的网络交换机。

+   Azure Stack Hub 限制了 Hyper-V 的配置和功能集，以保持与 Azure 的一致性。

+   Azure Stack Hub 不会让你访问底层基础设施技术。

Azure Stack HCI 的限制：

+   Azure Stack HCI 本身不提供或强制实施多租户架构。

+   Azure Stack HCI 不提供本地 PaaS 功能。

+   Azure Stack HCI 不包含本地的 DevOps 工具集。

+   Azure Stack HCI 仅支持虚拟化工作负载——不能在裸金属上部署应用。

要了解更多内容，请访问 [`docs.microsoft.com/azure-stack/operator/compare-azure-azure-stack?view=azs-2008`](https://docs.microsoft.com/azure-stack/operator/compare-azure-azure-stack?view=azs-2008)。

## 摘要

随着商业环境的发展，软件应用可能在不同的系统上运行，包括本地、非本地、多个云平台以及网络边缘。混合云和多云解决方案可以帮助你成功地部署和管理应用，只要你在组织的各个位置创建一个统一、一致的环境。通过这种方式，你为开发者提供了一套统一的工具来构建应用。你可以轻松地在不同位置之间迁移应用和数据，以确保效率和合规性，并为用户提供最佳体验。

Azure Arc 和 Azure Stack 组合是 Azure 混合云和多云解决方案与产品的两大核心组件。Azure Arc 提供统一管理，允许你在任何地方部署 Azure 服务并扩展 Azure 管理。通过启用 Azure Arc 的服务器，你可以将本地的 Azure 虚拟机管理应用到 Azure 外的 Windows 和 Linux 机器上。借助 Azure Arc 和 Kubernetes，你可以在任何基础设施上运行指定的 Azure 数据服务。

相比之下，Azure Stack 使 Azure 服务能够跨云和非云位置提供服务，让你能够在不同的环境和位置上一致地创建和运行混合应用，以应对多样化的工作负载。Azure Stack Hub 是一个云原生的集成系统，可以让你在本地使用 Azure 云服务，而 Azure Stack HCI 提供现代化数据中心和虚拟化主机的 HCI。

无论单独使用，还是与其他微软和 Azure 解决方案结合使用，Azure Arc 和 Azure Stack 都可以帮助你应对挑战，充分利用在金融、政府、制造业、零售、能源、健康等领域的机会。

你可以立即尝试像 Azure Arc 这样的功能，使用 Azure Arc Jumpstart 在 AKS、AWS Elastic Kubernetes Service、GKE 或 Azure 虚拟机中进行操作。一些示例步骤如下：

1.  安装客户端工具。

1.  创建 Azure Arc 数据控制器。

1.  在 Azure Arc 上创建托管实例。

1.  在 Azure Arc 上创建 PostgreSQL Hyperscale 服务器组的 Azure 数据库。

有关 Azure 混合云和多云解决方案及产品的更多信息，以及如何成功地为你的企业或组织使用它们，请访问以下微软网页：

+   Azure 混合云和多云解决方案：[`azure.microsoft.com/solutions/hybrid-cloud-app/`](https://azure.microsoft.com/solutions/hybrid-cloud-app/)

+   Azure 混合云和多云架构：[`docs.microsoft.com/azure/architecture/browse/?azure_categories=hybrid`](https://docs.microsoft.com/azure/architecture/browse/?azure_categories=hybrid)

+   Azure 混合云和多云模式与解决方案文档: [`docs.microsoft.com/hybrid/app-solutions/?view=azs-1910`](https://docs.microsoft.com/hybrid/app-solutions/?view=azs-1910)

我们希望本章内容能在您的 Azure 混合云和多云之旅中提供帮助。接下来，我们将探讨如何规划和实施迁移到 Azure。

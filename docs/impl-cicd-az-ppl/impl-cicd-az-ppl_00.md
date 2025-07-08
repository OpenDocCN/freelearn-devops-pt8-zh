# 前言

**持续集成**（**CI**）是利用自动化工具，根据开发人员对源代码所做的更改，自动持续地编译、打包和测试，从而确保它们的稳定性并能够提供预期的功能。

**持续交付**（**CD**）通过 CI 生成的工件，将这些应用程序自动部署到最终用户，无需人工干预，确保这些应用程序始终保持最新版本，并且在大多数情况下，在到达最终用户之前会在多个环境中进行额外验证。

所有这些都可以通过利用 Azure Pipelines 来实现，Azure Pipelines 是支持**软件开发生命周期**（**SDLC**）各个方面的领先平台之一；然而，本书将重点关注 CI/CD 方面，以及如何利用可用的功能来实现 DevOps 框架下的各种任务自动化选项，甚至触及 DevSecOps 的相关内容。

本书为您提供了从基础概念开始学习的工具，并从这里构建更复杂的场景。这将引导您实现端到端的场景，使您的软件开发团队能够在整个应用程序交付过程中（从源代码到运行平台），使用自动化工具来完成每个步骤。

# 本书适合谁阅读

从初学者到最先进的用户，任何希望更好理解如何利用 Azure Pipelines 的人，都能从本书中受益。

该内容的主要受众有三类人物，分别如下：

+   **软件开发人员**：他们将学习如何在非常早期的阶段，自动构建和部署他们的软件产品，无论目标平台如何，从而加速软件开发生命周期（SDLC）。

+   **DevOps 工程师**：他们将学习如何让 Azure Pipelines 支持任何自动化需求，无论流程的哪个阶段，都能在每一步注入质量和检查。

+   **安全工程师**：他们将学习如何将他们的工具集成到 CI/CD 流程中，在构建和部署过程的初期强制执行安全性和质量。

# 本书内容概述

*第一章*，*了解 Azure Pipelines*，提供了 CI/CD、Azure DevOps、Azure Pipelines 及其组件的介绍。它解释了为什么 Azure 管道是某些场景的正确选择，介绍了 Azure DevOps 下的其他服务（如 Azure Repos），并指导您如何设置新项目、配置自托管代理、准备管道环境以及配置代理池和部署组。

*第二章*，*创建构建管道*，教您如何在 Azure DevOps 中创建和管理管道、阶段、作业、任务、触发器和工件，以及在将代码推送到 Azure Repos 后如何运行管道。

*第三章*，*设置变量、环境、审批和检查*，涵盖了在 Azure DevOps 中创建服务连接、变量组、机密文件和发布管道的内容。它还解释了如何为 Azure Repos 和 GitHub 连接设置服务帐户。此外，你将学习如何安全地存储机密密钥并使用环境、审批和检查来控制阶段进展。

*第四章*，*使用 YAML 扩展高级 Azure Pipelines*，帮助你理解如何使用 YAML 创建构建和发布管道。它详细讨论了如何使用 YAML 语法创建阶段、作业和任务来进行 Web 应用程序部署。

*第五章*，*使用部署任务实现构建管道*，探讨了如何为构建过程创建和重用构建任务。本章介绍了使用 YAML 语法的流行 Node.js、NPM、.NET、Docker 和 SQL Server 部署任务。

*第六章*，*集成测试、安全任务和其他工具*，帮助你理解 Azure Pipelines 如何与其他工具扩展。本章涵盖了流行的代码分析工具 SonarQube 和用于制品的 Jenkins。

*第七章*，*监控 Azure Pipelines*，教你如何监控 Azure Pipelines 及相关任务，如构建任务、部署任务和管道代理。你还将学习如何在管道中构建监控，以确定部署是否改善或恶化了系统的质量。

*第八章*，*使用基础设施即代码（IaC）配置基础设施*，考察了如何为**基础设施即代码**（**IaC**）过程创建和重用部署任务。本章涵盖了流行的 IaC 工具 Terraform、Azure Bicep 和 ARM 模板，均使用 YAML 语法。

*第九章*，*为 Azure 服务实现 CI/CD*，展示了如何为 Azure 服务部署创建 YAML 和管道。你将学习如何在**Azure 应用服务**、**Azure Kubernetes 服务**（**AKS**）、**Azure 容器应用**和**Azure 容器实例**（**ACI**）上设置和部署应用程序。

*第十章*，*为 AWS 实现 CI/CD*，探讨了如何创建 YAML 和管道来在不同的服务上部署容器化应用程序，如**AWS Lightsail**、**Elastic Kubernetes 服务**（**EKS**）和**Elastic 容器服务**（**ECS**）。

*第十一章*，*使用 Flutter 自动化 CI/CD 跨移动应用程序*，介绍了如何使用 YAML 创建流水线来自动化移动应用程序的构建和发布流程。您还将学习如何实现 YAML 流水线以在 Apple TestFlight 和 Google Play 控制台上部署 Flutter，设置端到端流程的环境。

*第十二章*，*在 Azure Pipelines 中导航常见陷阱和未来趋势*，教您如何避免常见错误，并探讨 Azure Pipelines 的潜在未来趋势。

# 要充分利用本书

您需要基本了解如何使用自动化构建和部署应用程序；但是，本书将指导您如何在 Azure Pipelines 中执行这些操作。每章都有具体的技术要求。

| **书中涉及的软件/硬件** | **操作系统要求** |
| --- | --- |
| Docker | Windows、Linux 或 macOS |
| Visual Studio Code | Windows、Linux 或 macOS |

**如果您正在使用本书的数字版本，建议您自己输入代码或从书籍的 GitHub 存储库中获取代码（下一节中提供链接）。这样做将有助于避免与复制粘贴代码相关的潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件，网址为 [`github.com/PacktPublishing/Implementing-CI-CD-Using-Azure-Pipelines`](https://github.com/PacktPublishing/Implementing-CI-CD-Using-Azure-Pipelines)。如果代码有更新，将在 GitHub 存储库中更新。

我们还有来自丰富书籍和视频目录的其他代码包，可以在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 查看！

# 使用的约定

本书使用了许多文本约定。

`文本中的代码`：指示文本中的代码字词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。例如：“在 Linux 上添加以下基本脚本 - `echo "Hello Second Task"`。”

代码块设置如下：

```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "",
  "apiProfile": "",
  "parameters": {  },
  "variables": {  },
  "functions": [  ],
  "resources": [  ],
  "outputs": {  }
}
```

命令行输入或输出如下所示：

```
$id=az ad sp list –display-name azure-pipelines –query "[].id" -o tsv
```

**粗体**：表示新术语、重要单词或屏幕上看到的词语。例如，菜单或对话框中的单词以**粗体**显示。例如：“首先点击主菜单下的**管道**中的**环境**选项。”

提示或重要注意事项

如此显示。

# 联系我们

我们非常欢迎读者的反馈。

**总体反馈**：如果您对本书的任何方面有疑问，请发送电子邮件至 customercare@packtpub.com，并在消息主题中提及书名。

**勘误**：尽管我们已尽全力确保内容的准确性，但难免会出现错误。如果您在本书中发现错误，我们将非常感谢您向我们报告。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 并填写表单。

**盗版**：如果您在互联网上发现我们的作品有任何非法复制版本，我们将感激您提供相关的网址或网站名称。请通过 copyright@packtpub.com 联系我们，并提供该材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，且有兴趣写书或为书籍贡献内容，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

阅读完 *使用 Azure Pipelines 实现 CI/CD* 后，我们希望听到您的反馈！请 [点击这里直接进入亚马逊评论页面](https://packt.link/r/1804612499) 分享您的想法。

您的评价对我们和技术社区非常重要，将帮助我们确保提供优质的内容。

# 下载本书的免费 PDF 版本

感谢购买本书！

您喜欢随时随地阅读，但又无法将纸质书带在身边吗？

您的电子书购买与所选设备不兼容吗？

不用担心，现在每本 Packt 出版的书籍，您都能免费获得该书的无 DRM PDF 版本。

任何地方、任何设备上都能阅读。您可以直接将最喜爱的技术书籍中的代码搜索、复制并粘贴到您的应用程序中。

福利不仅如此，您还可以获得独家折扣、新闻通讯以及每天通过邮箱收到的精彩免费内容

按照以下简单步骤即可享受福利：

1.  扫描二维码或访问以下链接：

![下载本书的免费 PDF 版本](img/B18875_QR_Free_PDF.jpg)

[`packt.link/free-ebook/978-1-80461-249-1`](https://packt.link/free-ebook/978-1-80461-249-1)

1.  提交您的购买证明

1.  就是这样！我们将直接通过电子邮件将您的免费 PDF 和其他福利发送给您

# 第一部分：开始使用 Azure Pipelines

本部分将带你了解 Azure Pipelines 的基础知识，帮助你理解其概念，并展示如何快速入门，实现自动化构建和部署任务。

本部分包含以下章节：

+   *第一章*，*理解 Azure Pipelines*

+   *第二章*，*创建构建管道*

+   *第三章*，*设置变量、环境、审批和检查*

+   *第四章*，*使用 YAML 扩展高级 Azure Pipelines*

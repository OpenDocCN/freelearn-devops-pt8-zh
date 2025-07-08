# 前言

*Mastering Identity and Access Management with Microsoft Azure* 是一本简明实用的动手指南，包含了项目场景/插图，专为真正的混合云和纯云挑战量身定制。开发人员、安全专家、IT 顾问和架构师是本书的读者群体。通过本书，您将获得一份完整的伴侣，帮助您通过所有相关的 Microsoft 技术解决身份和访问管理领域的关键主题，并提供实践相关的简洁明了的内容，帮助您将理论付诸实践。本书深入探讨了 Microsoft 365 安全与合规计划以及其他与身份和访问管理相关的 Azure 服务。

本书分为三部分。第一部分涵盖了至关重要的身份管理主题，例如身份同步的整体内容，包括在纯云和混合云环境中的监控和保护主题。第二部分提供了与不同认证方法相关的所有基本知识和深入信息，您可以使用这些方法安全地发布和公开您的应用程序，结合本地技术和 Azure AD 功能集。书的最后一部分完全聚焦于 Microsoft 信息保护技术。另一个亮点是您将获得超过 40 个实用操作手册，通过实际技巧支持学习过程。通过这个丰富的资源包，您还将了解到 Windows 10 和 Windows Server 2016/2019 的功能。

本版与第一版有何不同，为什么会有超过 85%新内容的第二版？

首先，感谢所有读者和我收到的宝贵反馈。我很高兴能倾听大家的声音！

自 2016 年撰写第一版书籍以来，许多功能已经被完全更新、添加、更改，甚至删除。Microsoft Azure 的世界正在快速变化，从单纯的基础设施向面向对象和服务的环境转变。因此，书中需要包含多种开发方面的内容。目前，一些功能正在将其授权完全转移到云端。

然而，目前没有一个整体的解决方案能够满足混合云环境中可持续身份和访问管理的所有不同方面。因此，必须为各个服务制定基础，以确保功能的更好转换。

我决定编写更新版的另一个重要原因是，我从读者和研讨会参与者那里听到他们需要更多技术指导，而非决策管理方面的信息。这促使我采取了一种方法，在书中提供了超过 40 个实用指南，帮助读者以一种实践和引导的方式测试所有相关信息。此外，我们的研讨会参与者和客户发现，很难找到合格且有效的实验室示例，以节省时间和精力。

许多读者和我们的参与者喜欢第一版书中三个场景的结构。然而，我经常收到请求，希望能以技术或主题流的方式提供理论和实践指导，以便更容易跟进，尤其是当你只对某些特定主题感兴趣，或者希望将本书作为一个活跃的参考。

在编写第一版书时，Azure 信息保护技术还没有现在这样完备的实现方式。由于这项技术现在已经成熟，并且是访问管理的核心组成部分，因此我认为，针对这一主题增加章节是绝对必要的。

Windows Server 2019 也可用于操作，因此我更新了本书，使其适配新版本的服务器，主要聚焦于混合云场景。

# 本书适合人群

本书面向网络安全专家、系统与安全工程师、开发人员以及 IT 顾问/架构师，旨在帮助他们利用 Microsoft Azure 技术规划、设计并实施身份和访问管理解决方案。

# 如何最大化利用本书

为了高效使用本书，你应该对安全解决方案、Active Directory、访问权限/权限控制以及身份验证方法有所了解。编程知识不是必需的，但如果你希望使用 PowerShell 或通过 API 自定义解决方案，这些知识会有所帮助。阅读第一版书籍并不是跟随本书章节的前提。

在本书中，我们将完全在 Azure 平台上进行操作。唯一的要求是有互联网连接和一台微软或苹果客户端电脑。在我们使用的多个试用版本期间，实验可以免费进行。我们强烈建议你在 Azure 上关闭虚拟机，以节省与实践指导相关的运行时。在第七章，《在 Azure AD 和 ADFS 上部署解决方案》一章中，我们将提供架构概览，并提供所有必要的信息，以帮助进行资源规划以及使用和在章节中提到的不同产品。我们还将指导你使用 Let's Encrypt 创建公共证书。存在一个小的费用要求。如果你想运行所有不同的实验，你需要注册三个公共 DNS 域名，并且有权访问相关的公共 DNS。请记住，这个实验是为了学习和测试功能，而不是表示一个生产环境。按照各章节中的指示安排正确的资源。所有的脚本和示例文件都包含在示例代码文件中，你可以在下载示例代码文件部分的网页上下载它们。

# 下载示例代码文件

你可以从你在[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果你是在其他地方购买的此书，你可以访问[www.packt.com/support](http://www.packt.com/support)并注册以便将文件直接发送到你的邮箱。

你可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，按照屏幕上的指示操作。

文件下载完成后，请确保使用最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Mastering-Identity-and-Access-Management-with-Microsoft-Azure.Second-Edition`](https://github.com/PacktPublishing/Mastering-Identity-and-Access-Management-with-Microsoft-Azure.Second-Edition)。如果代码有更新，将会在现有的 GitHub 库中更新。

我们还提供了来自我们丰富书籍和视频目录的其他代码包，地址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快来看看吧！

# 下载彩色图像

我们还提供了一份 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。你可以在此下载：`www.packtpub.com/sites/default/files/downloads/9781789132304_ColorImages.pdf`

# 使用的约定

本书中使用了一些文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。例如：“打开你的 Visual Studio，并打开名为`WebApp-RoleClaims-DotNet.sln`的解决方案文件。”

一段代码如下所示：

```
$ServicePrincipalName="RMSPowerShell"
Connect-AadrmService
$bposTenantID=(Get-AadrmConfiguration).BPOSId
Disconnect-AadrmService
Connect-MsolService
New-MsolServicePrincipal -DisplayName $ServicePrincipalName
```

当我们希望你注意到代码块中的特定部分时，相关行或项目会以粗体显示：

```
$cred = Get-Credential
Install-AIPScanner -SqlServerInstance YD1APP01 -ServiceUserCredentials $cred
```

所有命令行输入或输出如下所示：

```
$ImportFile = Import-csv "$dirpath\ADUsers.csv"
$TotalImports = $importFile.Count
```

**粗体**：表示新术语、重要词汇或屏幕上显示的词汇。例如，菜单或对话框中的词语会以这种方式出现在文本中。示例：“点击添加新规则，并选择入站。”

警告或重要说明如下所示。

提示和技巧如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请在邮件主题中提到书名，并通过`customercare@packtpub.com`联系我们。

**勘误表**：虽然我们已尽最大努力确保内容的准确性，但错误难免发生。如果你在本书中发现错误，感谢你向我们报告。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择你的书籍，点击“勘误表提交表单”链接，并输入相关信息。

**盗版**：如果你在互联网上发现任何我们作品的非法复制版本，感谢你提供相关的地址或网站名称。请通过`copyright@packt.com`联系我们，并附上相关材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专长，并且有兴趣撰写或参与撰写书籍，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 书评

请留下书评。阅读并使用本书后，为什么不在你购买书籍的站点留下书评呢？潜在读者可以看到并参考你的公正意见来做出购买决定，我们在 Packt 可以了解你对我们产品的看法，作者也能看到你对其书籍的反馈。谢谢！

要了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。

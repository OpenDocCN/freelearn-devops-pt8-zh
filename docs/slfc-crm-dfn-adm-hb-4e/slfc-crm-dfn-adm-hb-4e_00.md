# 前言

作为一款行业领先的**客户关系管理**（**CRM**）应用程序，Salesforce CRM 帮助大大小小的企业改善客户关系。它极大地提升了销售业绩，并为您的业务提供了一个强大的 CRM 系统。为了从 Salesforce CRM 系统中获得最佳性能，作为 Salesforce 管理员，您需要解决许多问题。这是唯一一本提供 Salesforce CRM 管理方面全面指南的书籍。

本书将为您提供管理这一强大 CRM 应用程序所需的所有信息。它是实施 Salesforce CRM 的权威指南。无论您是想增强核心功能，还是已经开始自定义您的 Salesforce CRM 系统并寻找有关高级功能的指导，本书将向您展示如何从这一创新产品中获得最大收益。

# 本书内容概述

第一章，*设置 Salesforce CRM 和组织公司资料*，展示了如何设置影响系统外观和感觉的组织范围设置，并为组织内的所有用户提供访问功能。

第二章，*管理用户和控制系统访问*，描述了如何管理和维护用户记录和密码策略，并描述了个人用户的权限如何受到配置文件和权限集的影响。

第三章，*配置对象和应用程序*，涵盖了通过使用对象和字段配置和调整系统的各种方法，以适应组织内部信息使用的方式，并提供了对自定义字段治理的介绍。

第四章，*确保数据访问和数据验证的安全性*，详细探讨了 Salesforce CRM 中的数据访问安全模型以及可以在组织级别、对象级别、字段级别和记录级别应用数据访问和安全性的多个层次。

第五章，*管理 Salesforce CRM 中的数据*，描述了通过使用数据验证规则和依赖字段来提高数据质量的功能，并概述了导入和导出 Salesforce CRM 数据的设施。

第六章，*通过报告和仪表盘生成数据分析*，讨论了 Salesforce 系统中可用的分析构建模块，并详细介绍了报告和仪表盘的创建和使用。

第七章，*在 Salesforce CRM 中实现业务流程*，探讨了自动化业务工作流和审批机制的功能，旨在自动化、提高质量并在组织内生成高价值的流程。

第八章，*介绍 Sales Cloud、Service Cloud 以及 Salesforce CRM 的协作功能*，描述了 Salesforce CRM 中的核心功能领域，使销售团队、营销团队和服务团队能够成功并进行协作。

第九章，*扩展和增强 Salesforce CRM*，展示了如何扩展和增强系统中的标准功能，并描述了如何通过使用第三方应用程序在内部和外部添加高级定制和附加功能。

第十章，*管理 Salesforce CRM 的移动功能*，讨论了如何在 Salesforce CRM 中使用已成为个人和职业生活中常见设备的移动设备，并描述了 Salesforce 提供的移动解决方案。

第十一章，*为认证管理员考试学习*，描述了 Salesforce 认证管理员考试，并介绍了准备考试的资源，如基于课堂的培训课程 ADM-201，还提供了有关考试问题类型和考试计划的建议。

# 本书所需的内容

本书的前提条件是配备互联网连接的计算机以及以下支持的浏览器之一：Google Chrome、Mozilla Firefox、Apple Safari 或 Microsoft Internet Explorer。你需要 Salesforce CRM 的 Enterprise、Unlimited、Performance 或 Developer 版本以及系统管理员权限。

# 本书适合人群

本书面向希望在 Salesforce CRM 的配置和系统管理领域发展和增强技能的管理员。无论您是初学者还是更有经验的管理员，本书旨在提高您对 Salesforce CRM 平台的知识和理解。阅读完本书后，您将能够在一个完全支持您业务需求的实际环境中配置和管理 Salesforce CRM 和 Salesforce 移动解决方案。

# 约定

本书中，您会看到多种文本样式，用于区分不同类型的信息。以下是这些样式的示例，以及它们的含义解释。

文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入以及 Twitter 用户名如下所示：

Salesforce 提供了一套标准的预构建组件，例如`<apex:actionFunction>`和`<apex:actionStatus>`。

代码块的设置如下：

```
 var address = 
 "{!SUBSTITUTE(JSENCODE(Account.BillingStreet),'\r\n','
 ')}, " + 
 "{!Account.BillingCity}, " + 
 "{!Account.BillingPostalCode}, " + 
 "{!Account.BillingCountry}";
```

新术语和重要单词以**粗体**显示。您在屏幕、菜单或对话框中看到的单词，通常会像这样出现在文本中：导航到**帐户**选项卡并选择一个现有帐户。

### 注意

警告或重要说明将以类似于此的框体出现。

### 提示

提示和技巧将像这样显示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对本书的看法——喜欢什么或不喜欢什么。读者反馈对我们非常重要，它帮助我们开发出能让您真正受益的书籍。

若要向我们发送一般反馈，只需通过电子邮件发送至 feedback@packtpub.com，并在邮件主题中提及书名。

如果您在某个主题上有专业知识，并且有兴趣写作或参与书籍的编写，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)上的作者指南。

# 客户支持

既然您已经是 Packt 书籍的骄傲拥有者，我们为您准备了许多资源，帮助您充分利用您的购买。

## 下载本书的彩色图像

我们还提供了一份 PDF 文件，其中包含本书中使用的截图/图表的彩色图像。彩色图像将帮助您更好地理解输出中的变化。您可以从以下链接下载该文件：[`www.packtpub.com/sites/default/files/downloads/SalesforceCRMTheDefinitiveAdminHandbookFourthEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/SalesforceCRMTheDefinitiveAdminHandbookFourthEdition_ColorImages.pdf)

## 勘误

虽然我们已尽力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误——可能是文本或代码中的错误——我们将非常感激您能向我们报告。通过这样做，您可以帮助其他读者避免困扰，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入勘误详情进行报告。一旦您的勘误得到验证，您的提交将被接受，勘误将上传到我们的网站，或添加到该书的勘误列表中。

若要查看先前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将出现在“勘误”部分下。

## 盗版

互联网版权盗版问题在所有媒体中持续存在。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即向我们提供其位置地址或网站名称，以便我们采取措施。

请通过 copyright@packtpub.com 联系我们，并附上可疑盗版资料的链接。

感谢您帮助我们保护作者权益，并支持我们为您提供有价值的内容。

## 问题

如果您在书籍的任何方面遇到问题，可以通过 questions@packtpub.com 联系我们，我们将尽力解决。

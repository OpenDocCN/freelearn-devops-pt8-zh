# 前言

无服务器世界及其为新一代开发者带来的巨大潜力，已经引起了广泛的关注。在 *Google Cloud 实战：无服务器计算* 一书中，我们聚焦于 Google Cloud 提供的三大无服务器平台：App Engine、Cloud Functions 和 Cloud Run。每一节将为您提供产品的高层次介绍，并概述每项技术的主要特点。书的结尾部分将通过两个案例研究，展示无服务器的强大功能。这些案例研究的目的是演示如何在 Google Cloud 上管理无服务器工作负载。

# 本书适合的人群

本书的受众是那些正在使用 Google Cloud 并希望了解如何将无服务器技术与项目集成的人员。书中内容涵盖了 Google Cloud 上支持无服务器工作负载的主要产品。此外，书中还提供了多个示例，旨在帮助开发者自信地在 Google Cloud 上构建无服务器应用。

# 本书内容

第一章，*介绍 App Engine*，提供了 App Engine 的高层次概览，App Engine 是 Google 在 2008 年发布的首个无服务器平台（没错，已经十多年了）。本章让你了解什么是 App Engine，接着再深入探讨支持组件的细节。

第二章，*使用 App Engine 开发*，让你全面了解 App Engine 的各种功能。在本章中，我们阐述了平台的关键要素，展示了即便是最基础的应用也能轻松实现规模化和弹性扩展，而无需额外的维护工作。

第三章，*介绍轻量级函数*，讨论了转向单一功能函数的价值。减少应用程序的体积有助于更容易地调试和应对工作负载。在本章中，我们还介绍了 Google Cloud Functions 的概念，它将这一理念应用于在 Google Cloud 上提供无服务器工作负载。

第四章，*开发云函数*，介绍了如何在 Google Cloud 上使用云函数。在本章中，我们重点讨论了如何构建、部署和维护无服务器应用的关键要素。为了补充本章内容，我们讨论了功能框架（Functions Framework），它支持本地开发模拟云环境，并介绍了新手网页开发者需要了解的 HTTP 协议。

第五章，*探索作为服务的函数*，通过一系列示例构建一个服务。在本章中，我们开始利用 Google APIs 和各种资源与云函数进行集成。通过本章内容，我们可以看到事件处理如何帮助简化代码的开发。

第六章，*Cloud Functions 实验室*，创建了一系列示例，展示 Cloud Functions 的实际用例。在这一章中，我们将演示如何构建响应事件的 Web 组件。我们还将探讨如何通过服务账户保护 Cloud Functions。

第七章，*介绍 Cloud Run*，将讨论转向一个新主题——Cloud Run。在这一章中，我们将介绍与微服务和容器工作相关的一些关键基础知识。此外，我们还将介绍 Knative、gVisor 及其与 Cloud Run 的关系。

第八章，*使用 Cloud Run 进行开发*，讨论了 Google Cloud 上 Cloud Run 的界面以及如何使用它构建复杂的系统，如 REST API。我们还首次探讨了如何通过自动化一些任务来提高开发人员的生产力，使用 Google Cloud 的开发工具集。

第九章，*为 Anthos 开发 Cloud Run*，介绍了 Cloud Run for Anthos 的概念，这意味着我们第一次提到了 Kubernetes。这一章对于理解未来平台以及 Cloud Run 如何为多个环境提供支持至关重要。

第十章，*Cloud Run 实验室*，讲解了如何使用 Cloud Run，以及如何在 Google Kubernetes Engine 上集成无服务器工作负载。此外，我们还探讨了如何与 Cloud Run 产品一起进行持续集成（CI）。

第十一章，*构建 PDF 转换服务*，介绍了 Cloud Run 在 Google Cloud 上的两个用例中的第一个。在这个示例中，我们学习如何集成一个开源应用并将其部署为容器。此外，我们利用 Google Cloud Storage 的事件通知来最小化额外代码的开发。

第十二章，*通过 REST API 使用第三方数据*，继续探索 Google Cloud 上的 Cloud Run。在第二个示例中，我们学习如何为概念验证应用构建多个服务。该应用演示了几种技术，并展示了如何集成服务，如 Cloud Pub/Sub、Cloud Build 和 Container Registry。在练习结束时，你将获得如何消费 JSON 并围绕数据管理和传播构建可扩展解决方案的经验。

# 为了最大限度地利用本书

如果你已经具备如何在 Google Cloud 中进行导航的基本理解，包括如何访问所提供产品的控制面板，以及如何打开 Cloud Shell，那将是最理想的。

本书的大多数活动需要一个 Google Cloud 项目，另外，你也可以使用像 Qwiklabs（[`qwiklabs.com`](https://qwiklabs.com)）这样的沙箱环境。使用沙箱可以确保你所做的任何更改不会影响你常规的 Google Cloud 项目。

大多数章节都包含示例代码，可以通过以下链接获取：

[`github.com/PacktPublishing/Hands-on-Serverless-Computing-with-Google-Cloud-Platform`](https://github.com/PacktPublishing/Hands-on-Serverless-Computing-with-Google-Cloud-Platform)

本仓库包含每个章节所需的基础组件以及一个解决方案子目录。

完成每章末尾的所有小测验，并在进入下一章之前解决任何错误的答案。你必须了解为什么某个选项是正确答案，而不仅仅是知道它是正确答案。

这本书分为四个部分。为了全面了解特定的产品，我建议阅读关于*App Engine*（第 1-2 章）、*Cloud Functions*（第 3-6 章）和*Cloud Run*（第 7-10 章）的章节。若要查看如何在 Google Cloud 上部署无服务器工作负载的实际示例，请参阅第十一章和第十二章中的示例。

# 下载示例代码文件

你可以通过你的账户在[www.packt.com](http://www.packt.com)下载本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[www.packtpub.com/support](https://www.packtpub.com/support)并注册，将文件直接发送到你的邮箱。

你可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

一旦文件下载完成，请确保使用以下最新版本解压或提取文件夹：

+   WinRAR/7-Zip（适用于 Windows）

+   Zipeg/iZip/UnRarX（适用于 Mac）

+   7-Zip/PeaZip（适用于 Linux）

本书的代码包也托管在 GitHub 上，地址是[`github.com/PacktPublishing/Hands-on-Serverless-Computing-with-Google-Cloud`](https://github.com/PacktPublishing/Hands-on-Serverless-Computing-with-Google-Cloud)。如果代码有更新，它将会在现有的 GitHub 仓库中更新。

我们还提供了其他代码包，来自我们丰富的书籍和视频目录，访问链接是**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快来看看吧！

# 下载彩色图像

我们还提供了一份包含本书中截图/图表彩色图像的 PDF 文件。你可以在这里下载：[`static.packt-cdn.com/downloads/9781838827991_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781838827991_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。示例：“在命令提示符中，键入 `hostname` 并按下 *Enter* 键。”

**粗体**：表示新术语、重要词汇或屏幕上显示的词语。例如，菜单或对话框中的词语在文本中会这样显示。示例：“从上下文菜单中选择属性。”

警告或重要提示将以这种方式显示。

小贴士和窍门将以这种方式显示。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果您对本书的任何部分有疑问，请在邮件主题中提及书名，并通过 `customercare@packtpub.com` 联系我们。

**勘误**：尽管我们已尽力确保内容的准确性，但错误难免。如果您在本书中发现错误，我们将非常感激您向我们报告。请访问 [www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)，选择您的书籍，点击“勘误提交表单”链接，并输入相关信息。

**盗版**：如果您在互联网上遇到我们作品的任何非法复制，我们将非常感激您提供该材料的地址或网站名称。请通过 `copyright@packt.com` 联系我们，并附上该材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专长，并且有兴趣撰写或参与编写书籍，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。当您阅读并使用本书后，为什么不在您购买的站点上留下评论呢？潜在的读者可以看到并根据您的公正意见做出购买决策，我们在 Packt 可以了解您的想法，我们的作者也能看到您对他们书籍的反馈。谢谢！

若要获取有关 Packt 的更多信息，请访问 [packt.com](http://www.packt.com/)。

# 前言

云技术目前是 IT 行业内最受欢迎的趋势之一。每天都有新公司开始其云计算之旅，逐渐远离传统的本地部署方式，而后者往往妨碍快速开发和扩展运营。由于现代世界要求我们迅速适应不断变化的期望和动态工作负载，了解如何在云中开发应用变得越来越重要。

你手中拿着的这本书将引导你了解微软 Azure 这一流行云服务平台的不同功能和服务。我们将主要关注**平台即服务**（**PaaS**）组件，这些组件让你跳过繁琐的基础设施配置过程，直接专注于配置各种功能和部署代码，从而让你的应用具备可扩展性、高可用性和弹性。本书每一章的目标是帮助你更好地理解多种云模式、连接和集成，使你能迅速开始自己的项目，并了解在特定架构中应该使用哪个 Azure 服务。

# 本书适合谁阅读

本书旨在作为一次穿越不同 Azure PaaS 服务的旅程。它涵盖了多种基本和中级概念，这些概念对于基于微软云平台开发可靠且强大的解决方案至关重要。主要读者是刚开始接触 Azure 或有意开始使用 Azure 的开发人员和 IT 专业人士，书中的详细指南将帮助他们拓展云计算技能。

# 如何充分利用本书

激活你的 Azure 订阅（无论是试用版、MSDN 订阅功能，还是商业版）。

由于大多数示例基于 .NET 技术栈，了解一些相关技术将让你更加轻松，尽管在可能的情况下，其他技术栈也有所涉及，同时，基本的 HTTP 概念（例如协议或通信模型）也将是一个优势。对容器相关主题的基础理解同样有帮助。更高级的主题和详细的操作说明通常会出现在 *进一步阅读* 部分。确保在完成本书的练习后覆盖所有内容。

# 下载示例代码文件

你可以从你的 [www.packt.com](http://www.packt.com) 账户下载本书的示例代码文件。如果你是在其他地方购买本书，可以访问 [www.packt.com/support](http://www.packt.com/support) 注册，并直接通过电子邮件获取文件。

你可以按照以下步骤下载代码文件：

1.  在 [www.packt.com](http://www.packt.com) 登录或注册。

1.  选择“SUPPORT”标签。

1.  点击“Code Downloads & Errata”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用最新版本解压或提取文件夹：

+   适用于 Windows 的 WinRAR/7-Zip

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，链接为[`github.com/PacktPublishing/Hands-On-Azure-for-Developers`](https://github.com/PacktPublishing/Hands-On-Azure-for-Developers)。如果代码有更新，它将会在现有的 GitHub 仓库中更新。

我们还有其他来自丰富书籍和视频目录的代码包，详情请访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快来看看吧！

# 下载彩色图像

我们还提供了一个 PDF 文件，包含本书中截图/图表的彩色图像。你可以在此下载：`www.packtpub.com/sites/default/files/downloads/9781789340624_ColorImages.pdf`.

# 使用的规范

本书中使用了多种文本规范。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。举个例子：“在应用程序的`Main()`方法中，添加以下代码。”

代码块的格式如下：

```
using System;

namespace MyFirstWebJob
{
    class Program
    {
```

当我们希望你注意代码块中特定部分时，相关行或项目会以粗体显示：

```
            var config = new JobHostConfiguration();
            config.UseTimers();

            var host = new JobHost(config);
```

所有命令行输入或输出均按以下方式书写：

```
docker images
```

**粗体**：表示新术语、重要词汇或屏幕上显示的词汇。例如，菜单或对话框中的词汇在文本中会以这种方式出现。这里有一个例子：“为此，你需要右键点击 Azure App Service 的 WebJobs 部分，并选择在门户中打开。”

警告或重要提示以这种方式显示。

提示和技巧以这种方式出现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请在邮件主题中注明书名，并通过`customercare@packtpub.com`联系我们。

**勘误**：尽管我们已尽最大努力确保内容的准确性，但错误难免。如果你发现本书中的错误，我们将非常感激你报告给我们。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择你的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果你在互联网上发现我们作品的任何非法复制品，我们将非常感激你提供该位置地址或网站名称。请通过`copyright@packt.com`与我们联系，并附上相关资料的链接。

**如果你有兴趣成为作者**：如果你在某个主题上有专业知识，并且有兴趣撰写或参与编写书籍，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评论。当你阅读并使用完本书后，为什么不在你购买书籍的网站上留下评论呢？潜在的读者可以看到并利用你的客观评价做出购买决定，我们在 Packt 也能了解你对我们产品的看法，而我们的作者则能看到你对他们书籍的反馈。谢谢！

欲了解有关 Packt 的更多信息，请访问 [packt.com](http://www.packt.com/)。

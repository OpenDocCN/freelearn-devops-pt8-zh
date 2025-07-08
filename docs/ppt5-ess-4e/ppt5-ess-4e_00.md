# 前言

Puppet 是一款配置管理工具，它能够自动化管理你所有的 IT 配置，赋予你对每个 Puppet 代理在网络中的控制权，以及何时和如何进行操作。在这个数字化交付和无处不在的互联网时代，实施可扩展且可移植的解决方案变得越来越重要，这不仅限于软件，还包括运行这些软件的系统。

本书旨在传授所需的知识，不仅包括 Puppet 的基础，还涉及其核心。书中将探讨并解释基于 Puppet 的设计理念和基本原则。通过使用一个高效且富有生产力的工具来实现目标。

# 本书内容

第一章，*编写你的第一个清单*，讲解了基于资源的 Puppet 声明式配置管理，以及如何实现它们。

第二章，*Puppet 服务器与代理*，涵盖了 Puppet 服务器的安装与配置，以及如何将代理连接到服务器。

第三章，*深入了解 Puppet 的 Ruby 部分——Facts、Types 和 Providers*，解释了 Facter 及其 Facts、Types 和 Providers 在 Puppet 中的基本功能。

第四章，*在类和定义类型中组合资源*，讲解了自定义资源的使用，帮助你简化重复的代码。

第五章，*将类、配置文件和扩展结合到模块中*，解释了 Puppet 环境和节点分类的概念。

第六章，*Puppet 初学者的高级部分*，涵盖了 Puppet 的一些特性，如可读性、灵活性、EPP 模板、虚拟与导出的资源以及资源默认值等改进。

第七章，*Puppet 4 和 5 的新特性*，解释了 Puppet 环境和节点分类的概念。

第八章，*通过 Hiera 实现代码和数据的分离*，介绍了一种 Puppet 方式来分离代码和数据，从而可以管理数据。

第*9 章*，*Puppet 角色与配置文件*，提供了一种工作流，允许独立升级上游模块和本地 Puppet 实现。

# 本书所需条件

你需要 Debian 8+ 或 Ubuntu 14+，以及物理/虚拟的 x86 系统。

# 本书适用对象

本书面向有经验的 IT 专业人员和新的 Puppet 用户。本书将帮助你从安装到高级自动化全面了解 Puppet。你将快速入门，并学习如何建立 Puppet 的最佳实践，进行高级自动化。

# 约定

本书中，您将看到多种文本样式，用来区分不同类型的信息。以下是这些样式的一些示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名等会以如下方式显示：“文件资源类型将返回`mtime`和`ctime`。”

代码块会如下所示：

```
node 'agent' {
     $packages = [ 'apache2', 'libapache2-mod-php5', 'libapache2-mod-passenger', ]
     package { $packages:
       ensure => 'installed',
       before => Service['apache2'],
      }
     service { 'apache2':
       ensure => 'running',
       enable => true,
     }
   }
```

当我们希望特别提醒您注意代码块中的某部分时，相关的行或项目会以粗体显示：

```
service { 'puppet': enable => false }
cron { 'puppet-agent-run':
  user    => 'root',
  command =>
    'puppet agent --no-daemonize --onetime --
     logdest=syslog',
   minute => fqdn_rand(60),
   hour   => absent,
} 
```

任何命令行输入或输出都如下所示：

```
puppet:///modules/ntp/ntp.conf puppet:///modules/my_app/opt/scripts/find_my_app.sh
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的内容，会像这样出现在文本中：“如果您正在寻找一个能通过额外的资源类型和提供程序增强您的代理的模块，请在模块详情页上查找 Types 标签。”

警告或重要的说明会像这样显示。

提示和技巧会以这种方式出现。

# 读者反馈

我们欢迎读者的反馈。让我们知道您对本书的看法——您喜欢或不喜欢的内容。读者反馈对我们非常重要，因为它帮助我们开发出您真正能从中受益的书籍。如果您有任何意见或建议，直接通过邮件发送给`feedback@packtpub.com`，并在邮件主题中提及书名。如果您在某个领域有专业知识，并且有兴趣撰写或为书籍贡献内容，请参考我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，作为一本 Packt 书籍的骄傲拥有者，我们为您提供了许多帮助，帮助您从购买中获得最大收益。

# 下载示例代码

您可以从您的账户下载本书的示例代码文件，访问[`www.packtpub.com`](http://www.packtpub.com)。如果您是在其他地方购买的本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，直接将文件发送到您的邮箱。您可以按照以下步骤下载代码文件：

1.  使用您的邮箱地址和密码登录或注册我们的网站。

1.  将鼠标悬停在顶部的 SUPPORT 标签上。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的地方。

1.  点击“代码下载”。

一旦文件下载完成，请确保使用最新版本的工具解压或提取文件夹：

+   Windows 版 WinRAR / 7-Zip

+   Mac 版 Zipeg / iZip / UnRarX

+   Linux 版 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/Puppet-Essentials-Third-Edition`](https://github.com/PacktPublishing/Puppet-Essentials-Third-Edition)。我们还有其他来自我们丰富书籍和视频目录的代码包，您可以在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 查看！

# 下载本书的彩色图片

我们还为您提供了一份包含本书中截图/图表彩色图片的 PDF 文件。这些彩色图片将帮助您更好地理解输出中的变化。您可以从 [`www.packtpub.com/sites/default/files/downloads/PuppetEssentialsThirdEdition_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/PuppetEssentialsThirdEdition_ColorImages.pdf) 下载此文件。

# 勘误

尽管我们已尽一切努力确保内容的准确性，但错误仍然会发生。如果您在我们的书籍中发现任何错误——可能是文本错误或代码错误——我们将非常感激您能向我们报告。通过这样做，您可以帮助其他读者避免困扰，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata) 进行报告，选择您的书籍，点击勘误提交表格链接，输入您的勘误详细信息。一旦您的勘误被验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该书籍的勘误列表中。要查看之前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索栏中输入书名。所需信息将出现在勘误部分。

# 盗版

互联网版权材料的盗版问题在各类媒体中普遍存在。在 Packt，我们非常重视版权和许可证的保护。如果您在互联网上发现任何我们作品的非法复制品，请立即向我们提供该位置地址或网站名称，以便我们采取措施。请通过 `copyright@packtpub.com` 联系我们，并提供涉嫌盗版材料的链接。感谢您的帮助，保护我们的作者以及我们为您带来有价值内容的能力。

# 问题

如果您在本书的任何方面遇到问题，可以通过 `questions@packtpub.com` 联系我们，我们将尽力解决问题。

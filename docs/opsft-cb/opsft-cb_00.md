# 前言

OpenShift 是一个由 Red Hat 推出的开源、多语言支持、可扩展的云平台即服务（PaaS）。在写作时，OpenShift 正式支持 Java、Ruby、Python、Node.js、PHP 和 Perl 编程语言运行时，以及 MySQL、PostgreSQL 和 MongoDB 数据库。它还提供了 Jenkins CI、RockMongo、Mongo Monitoring Service 代理、phpMyAdmin 以及许多其他功能。由于 OpenShift 具有可扩展性，开发者可以通过添加对当前 OpenShift 不支持的运行时、数据库和其他服务的支持来扩展它。开发人员可以通过命令行工具、IDE 集成或 Web 控制台来使用 OpenShift。OpenShift 使用一个名为 Git 的流行版本控制系统来管理应用程序的部署。OpenShift PaaS 使得云端 Web 应用程序的开发变得更加简便。部署现有或新应用程序到 OpenShift 非常直接。全球许多开发者正在利用 OpenShift 的功能来加速开发和部署。

开始使用 OpenShift 很容易，但就像许多我们用来开发 Web 应用程序的工具一样，可能需要一些时间才能真正领略 OpenShift 的全部功能。OpenShift 平台及其客户端工具充满了你可能从未想过的功能。一旦你了解它们，它们将使你更加高效，并有助于编写可扩展的 Web 应用程序。

*OpenShift Cookbook* 以简明易懂的方式展示了 100 多个实例。它将带领你学习多个实例，展示 OpenShift 的特性，并演示如何在上面部署特定的技术或框架。你可以快速学习并立即开始在 OpenShift 上部署应用程序。该手册还涵盖了水平扩展、应用程序日志记录和监控等主题。所覆盖的实例解决了在 OpenShift 上有效运行应用程序时常遇到的常见问题。假设读者已经熟悉 PaaS 和云计算的概念。此书不需要从头到尾阅读，允许读者选择自己感兴趣的章节和实例。*OpenShift Cookbook* 是一本易读的书，充满了实用的实例和有帮助的截图。

# 本书内容概述

第一章，*开始使用 OpenShift*，首先介绍了 OpenShift 及如何创建 OpenShift Online 账户。你将使用 Web 控制台创建你的第一个 OpenShift 应用程序，并了解常见的 OpenShift 术语，如“齿轮”和“卡带”。Web 控制台通常是开发人员使用的 OpenShift 主要界面。本章还讨论了如何安装 rhc OpenShift 命令行工具以及如何使用它执行基本操作。

第二章, *管理域名*，讨论了域名和命名空间的概念。你将学习如何执行诸如创建、重命名、查看和删除域名等操作。此外，本章还介绍了成员身份的概念，帮助团队协作。

第三章, *创建和管理应用程序*，介绍了如何使用 rhc OpenShift 命令行工具创建应用程序。rhc 命令行客户端是与 OpenShift 交互的最强大方式。你将学习如何执行各种应用程序管理操作，如启动、停止、清理和删除应用程序，并使用 rhc 进行管理。还将讨论 OpenShift 的高级功能，如部署跟踪、回滚、配置二进制文件和源代码部署。此外，你还将学习如何为 OpenShift 应用程序使用自己的域名。

第四章, *使用 MySQL 与 OpenShift 应用程序*，教读者如何将 MySQL 数据库与他们的应用程序一起使用。还将介绍如何更新默认的 MySQL 配置以满足应用程序的需求。

第五章, *使用 PostgreSQL 与 OpenShift 应用程序*，提供了一些示例，展示如何开始使用 OpenShift 的 PostgreSQL 数据库组件。你将学习如何添加和管理 PostgreSQL 组件，备份 PostgreSQL 数据库，列出并安装 PostgreSQL 扩展，并使用 EnterpriseDB PostgreSQL 云数据库服务与 OpenShift 应用程序配合使用。

第六章, *使用 MongoDB 和第三方数据库组件与 OpenShift 应用程序*，提供了多个示例，展示如何开始使用 OpenShift 的 MongoDB 组件。你还将学习如何使用 MariaDB 和远程字典服务器（Redis）等可下载组件。

第七章, *OpenShift for Java 开发者*，介绍了 Java 开发者如何有效使用 OpenShift 开发和部署 Java 应用程序。你将学习如何在 OpenShift 上部署 Java EE 6 和 Spring 应用程序。OpenShift 与各种 IDE 有着一流的集成，因此你将学习如何使用 Eclipse 来开发和调试 OpenShift 应用程序。

第八章，*Python 开发者的 OpenShift 指南*，讲解了 Python 开发者如何有效地使用 OpenShift 开发和部署 Python 应用程序。本章将教你如何在 OpenShift 上开发 Flask 框架的 Web 应用程序。你还将学习如何管理应用程序的依赖，访问应用程序的 virtualenv，并使用独立的 WSGI 服务器，如 Gunicorn 或 Gevent。

第九章，*Node.js 开发者的 OpenShift 指南*，介绍了如何使用 OpenShift 构建 Node.js 应用程序。你将学习如何使用 Express 框架构建 Web 应用程序。本章还将介绍如何使用 npm 管理应用程序依赖，如何使用 WebSocket，以及如何在 OpenShift Node.js 应用程序中使用 CoffeeScript。

第十章，*OpenShift 应用程序的持续集成*，教你如何与 OpenShift 应用程序进行持续集成。你将学习如何将 Jenkins 插件添加到应用程序中，并自定义 Jenkins 任务以满足需求。本章还介绍了如何安装 Jenkins 插件，构建托管在 GitHub 上的项目，并为 OpenShift 应用程序定义自定义 Jenkins 工作流。

第十一章，*日志记录与扩展 OpenShift 应用程序*，包含了有助于处理应用程序日志的方案。你将学习如何创建自动可扩展的应用程序。你将学习如何禁用自动扩展，并使用 rhc 命令行工具手动扩展 OpenShift 应用程序。

附录，*在虚拟机上运行 OpenShift*，解释了如何在虚拟化环境中运行 OpenShift 实例。

# 本书所需的内容

所有的示例都包含了每个方案所需的工具参考。预计你是一个熟练的 Web 开发人员，精通所使用的 Web 框架。你应该具备 Git 和 Bash 的基本知识。如果你是 Java 开发人员，你需要最新版本的 Java 和 Eclipse。如果你是 Python 开发人员，你需要 Python、virtualenv 和一个文本编辑器。如果你是 Node.js 开发人员，你需要 Node.js 和一个文本编辑器。

# 本书适合谁阅读

本书面向那些希望使用 OpenShift 构建下一个大创意的读者。读者可能是已经在使用 OpenShift 或计划将来使用它的 web 开发者。书中的配方提供了完成各种任务所需的信息。假设你已经熟悉你希望用来开发 web 应用程序的编程语言中的 web 开发。例如，如果你是 Java 开发者，那么你应该已经掌握了 Java EE 或 Spring 的基础知识。本书不会涉及 Java EE 或 Spring 的基础内容，而是讲解如何在 OpenShift 上部署 Java EE 或 Spring 应用程序。

# 约定

本书中，你会看到多种文本样式，用于区分不同类型的信息。以下是一些样式的示例，以及它们的含义说明。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号显示如下：“代替 `osbook`，消息将引用你的域名。”

代码块的格式如下：

```
[remote "origin"]
  url = ssh://52bbf209e0b8cd707000018a@myapp-osbook.rhcloud.com/~/git/blog.git/
  fetch = +refs/heads/*:refs/remotes/origin/*
```

任何命令行输入或输出都如下所示：

```
$ ssh 52b823b34382ec52670003f6@blog-osbook.rhcloud.com ls
app-deployments
app-root
git
mysql
php
phpmyadmin

```

**新术语** 和 **重要词汇** 会以粗体显示。你在屏幕上、菜单或对话框中看到的词汇会像这样出现在文本中：“点击 **I Accept** 按钮，浏览器将重定向到入门网页。”

### 注意

警告或重要说明会以框的形式出现。

### 提示

小贴士和技巧会像这样显示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们你对本书的看法——你喜欢哪些内容，或是不喜欢哪些内容。读者的反馈对我们开发能真正为你带来最大收益的书籍至关重要。

若想给我们提供一般反馈，请发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提到书名。

如果你在某个领域具有专业知识，并且有兴趣为书籍撰写或贡献内容，请参见我们在 [www.packtpub.com/authors](http://www.packtpub.com/authors) 上的作者指南。

# 客户支持

既然你已经成为了 Packt 图书的骄傲拥有者，我们提供了许多帮助你从购买中获得最大收益的内容。

## 下载示例代码

你可以从你的账户中下载所有已购买的 Packt 图书的示例代码文件，网址为 [`www.packtpub.com`](http://www.packtpub.com)。如果你是从其他地方购买的本书，你可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，将文件直接通过电子邮件发送给你。

## 勘误

虽然我们已经尽一切努力确保内容的准确性，但错误偶尔还是会发生。如果您在我们的任何一本书中发现错误——可能是文本或代码的错误——我们将不胜感激您向我们报告。通过这样做，您可以避免其他读者的挫折，并帮助我们改进此书的后续版本。如果发现任何勘误，请访问 [`www.packtpub.com/support`](http://www.packtpub.com/support)，选择您的书籍，点击**勘误提交表单**链接，输入您的勘误细节。一旦您的勘误被验证，您的提交将被接受，并且勘误将被上传到我们的网站，或者添加到该书籍的现有勘误列表中的勘误部分。

## 盗版

在互联网上盗版版权材料是所有媒体的长期问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何形式的非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请联系我们，使用链接到疑似盗版材料的地址`<copyright@packtpub.com>`。

我们感谢您帮助保护我们的作者，以及我们为您带来有价值内容的能力。

## 问题

如果您在书籍的任何方面遇到问题，请联系我们，您可以发送电子邮件至`<questions@packtpub.com>`，我们将尽力解决。

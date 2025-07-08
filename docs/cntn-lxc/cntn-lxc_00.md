# 前言

不久前，我们通常在单一服务器上部署应用程序，通过增加更多硬件资源来进行扩展——我们称之为“单体架构”。实现高可用性通常是通过在负载均衡器后面增加更多的单用途服务器/单体来完成，往往最终形成一个资源利用率低的系统集群。编写和部署应用程序也遵循这种单体架构——软件通常是一个大型二进制文件，提供了大部分甚至是全部的功能。我们要么从源代码编译它并使用某种安装程序，要么将其打包并发送到仓库。

随着虚拟机和容器的出现，我们摆脱了服务器单体架构，通过在隔离且资源受限的实例中运行应用程序，充分利用了可用的计算资源。应用程序的扩展变成了在服务器集群上添加更多虚拟机或容器，然后找到一种方法自动部署它们。我们还将单一的二进制应用拆分成微服务，它们通过消息总线/队列相互通信，充分利用容器所提供的低开销。部署完整的应用堆栈现在只是将服务打包到各自的容器中，创建一个单一的、完全隔离、依赖完整的工作单元，准备好进行部署。使用持续集成模式和工具（如 Jenkins）使我们能够进一步自动化构建和部署过程。

本书讲解的是 LXC 容器以及如何在其中运行应用程序。与其他容器解决方案（如 Docker）不同，LXC 旨在运行整个 Linux 系统，而不仅仅是单个进程，尽管后者也是可能的。即使 LXC 容器可以包含整个 Linux 文件系统，底层的宿主内核仍然是共享的，无需虚拟化层。

本书采用直接而实用的方式介绍 LXC。你将学习如何安装、配置和操作 LXC 容器，并通过多个示例讲解如何在 LXC 中运行高度可扩展且高可用的应用程序。你将使用监控和部署应用程序以及其他第三方工具。你还将学习如何编写自己的工具，扩展 LXC 及其各种库提供的功能。最后，你将看到一个完整的 OpenStack 部署，它为管理一组计算资源提供了智能化，使得在 LXC 容器中轻松部署你的应用程序。

# 本书内容涵盖的内容

第一章，*Linux 容器简介*，深入探讨了 Linux 内核中容器的历史，并介绍了一些基本术语。在了解基本内容后，你将详细了解内核命名空间和控制组（cgroups）的实现，并能够尝试一些 C 系统调用。

第二章，*在 Linux 系统上安装和运行 LXC*，涵盖了在 Ubuntu 和 Red Hat 系统上安装、配置和运行 LXC 所需的所有内容。你将学习所需的软件包和工具，并了解不同的 LXC 配置方法。通过本章的学习，你将拥有一个运行 LXC 容器的 Linux 系统。

第三章，*使用原生工具和 Libvirt 工具进行命令行操作*，专注于如何在命令行中运行和操作 LXC。本章将涵盖各种工具和软件包，并演示与容器化应用程序交互的不同方式。重点将放在 libvirt 和原生 LXC 库提供的功能上，控制 LXC 容器的整个生命周期。

第四章，*LXC 与 Python 代码集成*，将展示如何使用 Python 库编写工具并自动化 LXC 的配置和管理。你还将学习如何使用 Vagrant 和 LXC 创建开发环境。

第五章，*使用 Linux 桥接和 Open vSwitch 进行 LXC 网络配置*，将深入探讨容器化世界中的网络配置——将 LXC 连接到 Linux 桥接，使用直接连接、NAT 和其他多种方法。它还将演示使用 Open vSwitch 进行流量管理的更高级技巧。

第六章，*使用 LXC 进行集群和横向扩展*，在前面章节的基础上构建，展示如何构建 Apache 容器集群，并演示如何通过使用 Open vSwitch 的 GRE 隧道将它们连接起来。本章还展示了如何在最小化根文件系统容器中运行单进程应用程序的示例。

第七章，*容器化世界中的监控与备份*，讨论如何备份你的 LXC 应用容器并部署监控解决方案来进行警报和触发操作。我们将看到使用 Sensu 和 Monit 进行监控的示例，以及使用 iSCSI 和 GlusterFS 创建热备份和冷备份的示例。

第八章，*将 LXC 与 OpenStack 结合使用*，演示如何使用 OpenStack 配置 LXC 容器。首先介绍 OpenStack 的各个组件，以及如何使用 LXC nova 驱动程序在计算资源池中自动配置 LXC 容器。

附录，*LXC 与 Docker 和 OpenVZ 的替代方案*，通过展示其他流行的容器解决方案如 Docker 和 OpenVZ 的由来，以及它们之间的相似性和差异，结束本书。它还探讨了如何在 LXC 旁边安装、配置和运行这些解决方案的实际示例。

# 本书所需的内容

对 Linux 和命令行有初学者水平的知识应该足以跟随并运行示例代码。虽然本书并非关于软件开发，但要完全理解和实验代码片段需要一定的 Python 和 C 语言知识。如果不感兴趣，你可以跳过第四章，*LXC 与 Python 的代码集成*。

在硬件和软件要求方面，本书中的大多数示例已在虚拟机中测试过，虚拟机利用了各种云服务提供商，如 Amazon AWS 和 Rackspace Cloud。考虑到 Canonical 公司参与 LXC 项目，我们推荐使用最新版本的 Ubuntu，但在安装/操作方法有所不同的情况下，我们也提供了 CentOS 的示例。

# 本书适合谁阅读

本书适合任何对 Linux 容器感兴趣的人，从寻求深入理解 LXC 如何工作的 Linux 管理员，到需要在没有完整虚拟机开销的隔离环境中快速简便地原型化代码的软件开发人员。想要从头到尾阅读本书的最合适职位是 DevOps 工程师。

# 约定

本书中会有多种文本样式，用于区分不同类型的信息。以下是一些样式示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示：“手动使用`debootstrap`和`yum`等工具构建根文件系统和配置文件。”

一段代码如下所示：

```
#define _GNU_SOURCE
#include<stdlib.h>
#include<stdio.h>
#include<signal.h>
#include<sched.h>

staticintchildFunc(void *arg)
{
  printf("UID inside the namespace is %ld\n", (long) geteuid());
  printf("GID inside the namespace is %ld\n", (long) getegid());
}
```

当我们希望引起你对代码块中特定部分的注意时，相关行或项会以粗体显示：

```
<head> 
#define _GNU_SOURCE
#include
#include
#include
#include

staticintchildFunc(void *arg)
{
 printf("UID inside the namespace is %ld\n", (long) geteuid());  printf("GID inside the namespace is %ld\n", (long) getegid());
}

```

任何命令行输入或输出如下所示：

```
root@ubuntu:~# lsb_release -dc
Description:   	Ubuntu 14.04.5 LTS
Codename:      	trusty
root@ubuntu:~#

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词汇，例如菜单或对话框中的内容，会以如下方式出现在文本中：“导航到**网络支持** | **网络选项** | **802.1d 以太网桥接**，并选择**Y**以在内核中编译桥接功能，或选择**M**将其作为模块编译。”

### 注意

警告或重要提示会显示在像这样的框中。

### 提示

提示和技巧如这样显示。

# 读者反馈

我们非常欢迎读者的反馈。告诉我们您对本书的看法——您喜欢或不喜欢的部分。读者的反馈对我们非常重要，因为它帮助我们开发出您真正能从中受益的书籍。如果您有某个领域的专长，并且有兴趣撰写或参与编写书籍，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，作为一本 Packt 书籍的骄傲拥有者，我们提供了一些帮助您充分利用购买的资源。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户中下载此书的示例代码文件。如果您是在其他地方购买的此书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件将文件发送给您。

您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的官方网站。

1.  将鼠标指针悬停在顶部的**SUPPORT**标签上。

1.  点击**Code Downloads & Errata**。

1.  在**Search**框中输入书名。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的地方。

1.  点击**Code Download**。

一旦文件下载完成，请确保使用以下最新版本解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址是[`github.com/PacktPublishing/Containerization-with-LXC`](https://github.com/PacktPublishing/Containerization-with-LXC)。我们还有其他书籍和视频的代码包，您可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。赶快去看看吧！

## 下载本书的彩色图片

我们还为您提供了一份包含本书中使用的截图/图表的彩色图片的 PDF 文件。这些彩色图片将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/ContainerizationwithLXC_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ContainerizationwithLXC_ColorImages.pdf)下载该文件。

## 勘误

尽管我们已经尽力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现任何错误——无论是文本还是代码错误——我们将非常感激您向我们报告。这样，您可以帮助其他读者避免困扰，并且帮助我们改进后续版本。如果您发现任何勘误，请访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并填写勘误的详细信息。一旦您的勘误得到验证，您的提交将被接受，并且勘误将会被上传到我们的网站或添加到该书的勘误部分的现有列表中。

若要查看之前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，在搜索框中输入书名。相关信息将出现在**勘误**部分。

## 盗版

网络上版权材料的盗版问题是一个持续存在的全球性问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现任何形式的非法复制作品，请立即向我们提供其位置地址或网站名称，以便我们采取措施解决。

如果你发现疑似盗版的内容，请通过电子邮件联系我们，邮箱地址为 copyright@packtpub.com，并提供相关链接。

我们感谢您帮助保护我们的作者，并支持我们继续为您提供有价值的内容。

## 问题

如果你对本书的任何内容有疑问，可以通过电子邮件联系 questions@packtpub.com，我们会尽力解决问题。

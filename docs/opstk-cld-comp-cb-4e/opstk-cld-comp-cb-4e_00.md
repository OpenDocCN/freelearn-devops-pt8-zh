# 序言

OpenStack 是用于构建公共和私有云以及私有托管软件定义基础设施服务的开源软件。它是一个全球性的开源成功案例，由全球成千上万的人开发和支持，并得到了云计算领域领先企业的支持。本书专门设计旨在快速帮助你掌握 OpenStack，并提供信心与理解，以便将其部署到自己的数据中心。本书涵盖了使用 Ansible 安装 OpenStack 到数据中心的过程，并提供了运行的步骤，帮助你快速获取所需的知识，以便当下能够安装并操作 OpenStack。本书由 Rackspace 的四位高级 OpenStack 工程师和架构师编写，涵盖了广泛的主题，帮助你安装和配置 OpenStack 环境。本书将指导你完成以下内容：

+   如何使用 OpenStack-Ansible 安装和配置 OpenStack 的所有核心组件

+   如何掌握完整的私有云技术栈，从扩展计算资源到管理面向高冗余、高可用存储的对象存储服务

+   使用 Heat 和 Ansible 编排在 OpenStack 上运行的工作负载的示例

+   每一章都有实际的、基于真实场景的服务示例，确保你在自己的环境中也能顺利实现。

本书为你提供清晰、逐步的指导，帮助你成功安装和运行自己的私有云。内容中包含了大量实用且可操作的方案，使你能够使用 OpenStack 的最新功能并加以实现。

# 本书适用对象

本书面向从虚拟化环境转向云环境的系统管理员和技术架构师，要求读者熟悉云计算平台。预期读者具备虚拟化和管理 Linux 环境的知识。尽管有助于理解，但不要求具备 OpenStack 的前置知识或经验。

# 本书涵盖内容

第一章，*使用 Ansible 安装 OpenStack*，带领你通过实际操作安装 OpenStack，使用 OpenStack-Ansible。

第二章，*OpenStack 客户端*，介绍了如何安装操作 OpenStack 所需的工具，并提供了常用命令的快速参考。

第三章，*Keystone – OpenStack 身份认证服务*，介绍了如何使用 Keystone 配置和维护 OpenStack 中的服务和项目。

第四章，*Neutron – OpenStack 网络*，涵盖了软件定义网络服务及其插件的介绍与使用。

第五章，*Nova – OpenStack 计算*，通过一系列示例讲解如何运行和操作虚拟机实例以运行您的应用程序。

第六章，*Glance – OpenStack 镜像服务*，探讨了用于启动实例的机器镜像的使用方法。

第七章，*Cinder – OpenStack 块存储*，介绍了如何使用块存储卷并将其附加到您的实例。

第八章，*Swift – OpenStack 对象存储*，展示了如何使用高可用且冗余的对象存储服务。

第九章，*使用 Heat 和 Ansible 进行 OpenStack 编排*，讨论了如何开始使用 Heat 和 Ansible 在 OpenStack 内编排工作负载。

第十章，*使用 OpenStack 仪表盘*，展示了如何浏览 Horizon 用户界面仪表盘。

# 为了充分利用本书

使用本书时，您需要访问具有硬件虚拟化功能的计算机或服务器。预计您具有 Linux 操作系统的基础知识，并能通过 SSH 访问 Linux 服务器。

要设置第一章，*使用 Ansible 安装 OpenStack*，末尾描述的实验环境，您需要安装并使用 Oracle 的 VirtualBox 和 Vagrant。您可以访问[`github.com/OpenStackCookbook/vagrant-openstack`](https://github.com/OpenStackCookbook/vagrant-openstack)查看如何使用 VirtualBox 和 Vagrant 设置计算机的详细信息。

获取更多启动实验环境的附加资源，请访问[`www.openstackcookbook.com`](http://www.openstackcookbook.com)。参考该网站以获取有关安装支持软件（如 MariaDB/MySQL）的信息。更多信息请见[`bit.ly/OpenStackCookbookPreReqs`](http://bit.ly/OpenStackCookbookPreReqs)。

## 下载示例代码文件

你可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果你在其他地方购买了本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，注册后直接将文件发送到你的邮箱。

你可以通过以下步骤下载代码文件：

1.  登录或注册 [`www.packtpub.com`](http://www.packtpub.com)。

1.  选择**支持**标签。

1.  点击**代码下载与勘误表**。

1.  在**搜索**框中输入书名，并按照屏幕上的指示操作。

一旦文件下载完成，确保使用以下最新版本的其中一个解压或解压文件夹：

+   适用于 Windows 的 WinRAR / 7-Zip

+   适用于 Mac 的 Zipeg / iZip / UnRarX

+   适用于 Linux 的 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/OpenStack-Cloud-Computing-Cookbook-Fourth-Edition`](https://github.com/PacktPublishing/OpenStack-Cloud-Computing-Cookbook-Fourth-Edition)。我们还拥有其他来自我们丰富书籍和视频目录的代码包，网址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。快来看看吧！

## 下载彩色图片

我们还提供了一个 PDF 文件，里面包含了本书中使用的截图/图表的彩色图片。你可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/OpenStackCloudComputingCookbookFourthEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/OpenStackCloudComputingCookbookFourthEdition_ColorImages.pdf)。

## 使用的约定

本书中使用了一些文本约定。

`CodeInText`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账户名。这里有一个例子：在 Ubuntu 系统上，主机网络配置通过编辑`/etc/network/interfaces`文件来完成。代码块如下所示：

```
# Shared infrastructure parts
shared-infra_hosts:
  controller-01:
    ip: 172.29.236.110
  controller-02:
    ip: 172.29.236.111
  controller-03:
    ip: 172.29.236.112
# Compute Hosts
compute_hosts:
  compute-01:
    ip: 172.29.236.113
  compute-02:
    ip: 172.29.236.114
```

当我们希望特别提醒你注意代码块的某一部分时，相关行或项目会用粗体显示：

```
# Shared infrastructure parts
shared-infra_hosts:
 controller-01:
    ip: 172.29.236.110
  controller-02:
    ip: 172.29.236.111
  controller-03:
    ip: 172.29.236.112
# Compute Hosts
compute_hosts:
  compute-01:
    ip: 172.29.236.113
  compute-02:
    ip: 172.29.236.114
```

任何命令行输入或输出都如下所示：

```
cd /opt/openstack-ansible/scripts

```

**粗体**：表示一个新术语、一个重要的词或你在屏幕上看到的词，例如在菜单或对话框中，也会像这样出现在文本中。这里有一个例子：“接下来从左侧菜单中选择**高级系统设置**。”

### 注意

警告或重要说明将以框的形式显示，如下所示。

### 提示

小贴士和技巧将以这种形式出现。

# 各部分

在本书中，你会经常看到一些标题（*准备工作*、*如何做...*、*它是如何工作的...*、*还有更多...* 和 *另见*）。

为了清楚地说明如何完成一个食谱，请使用以下部分：

## 准备工作

本节告诉你在食谱中可以期待什么，并描述如何设置所需的软件或任何预备设置。

## 如何操作……

本节包含遵循食谱所需的步骤。

## 工作原理…

本节通常包含对上一节内容的详细解释。

## 还有更多…

本节包含有关该食谱的额外信息，帮助你更好地了解该食谱。

## 另见

本节提供有关食谱的有用链接，帮助你获得更多相关信息。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：发送电子邮件至 `<feedback@packtpub.com>`，并在邮件的主题中提及书名。如果你对本书的任何方面有疑问，请通过 `<questions@packtpub.com>` 与我们联系。

**勘误**：虽然我们已经尽力确保内容的准确性，但错误仍然可能发生。如果你在本书中发现了错误，我们将非常感激你向我们报告。请访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击勘误提交表单链接，并输入相关细节。

**盗版**：如果你在互联网上发现任何形式的非法复制品，我们将感激你提供相关的地址或网站名称。请通过 `copyright@packtpub.com` 联系我们，并附上相关资料的链接。

**如果你有兴趣成为一名作者**：如果你在某个领域有专业知识，并且有兴趣写书或为书籍做贡献，请访问 [`authors.packtpub.com`](http://authors.packtpub.com)。

## 评论

请留下评论。阅读并使用本书后，为什么不在你购买书籍的网站上留下评论呢？潜在读者可以通过你的公正意见做出购买决策，我们 Packt 也可以了解你对我们产品的看法，而我们的作者则能看到你对他们书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问 [packtpub.com](http://packtpub.com)。

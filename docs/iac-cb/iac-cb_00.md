# 前言

在不断发展的环境中，运营和开发团队越来越多地共同合作，使用工具和技术，分享作为 DevOps 运动一部分而流行的共同文化。从开发到生产，一个共同的工具和方法逐渐形成——这通常借鉴了开发人员和敏捷技术。

现在，API 在数据中心无处不在，自动化已经接管了曾经是系统管理员或 IT 工作的一切方面——基础设施现在基本上就是代码，在开发或分布式团队的生产环境中工作时，应该像对待代码一样对待它。

学习适合基础设施即代码描述的最重要工具、技术和工作流可能是一个艰巨的任务，许多团队可能会因需要大量信息、变更和知识来转向基础设施即代码而感到迷茫或沮丧。

本书的写作考虑到了我们过去几年在各自的工作中遇到的所有团队——这些团队对 DevOps、自动化和代码感兴趣，有些团队已经在某些方面做得相当好，但他们愿意探索其他工具和技术，发现如何通过提高代码质量、基础设施稳定性、服务可扩展性、部署速度、团队工作效率和反馈循环，来做得更好。

本书是一次谦逊的尝试，旨在涵盖与基础设施即代码相关的一切，基于我们在实际工作中的经验，从使用 Vagrant 的开发工作流，到使用 Terraform 或 Ansible 的复杂生产基础设施部署，从使用 Chef 和 Puppet 的配置管理基础，到高级的测试驱动开发（TDD）技术，以及全面的基础设施代码覆盖测试。本书还将提供深入的 Docker 技术和更多内容。只要可能或相关，我们尽力展示了使用其他工具或方法完成相同任务的替代方式，让任何对该主题有一定了解的人，在本书的任何部分都能找到学习的内容。

我们希望你能从本书中获得很多收获，并且希望使用基础设施即代码（infrastructure-as-code）来进行自动化和测试，能像我们写这本书时那样让你感到有趣。

# **本书内容概述**

第一章，*Vagrant 开发环境*，专注于使用 Vagrant 进行自动化开发环境。启动简单或复杂的环境，模拟各种虚拟网络配置，结合 Vagrant 和 Docker 或 Amazon 云，将虚拟机的配置交给 Chef 和 Ansible。所有示例都是自包含的现实生活中的小项目。

第二章，*使用 Terraform 配置 IaaS*，提供了在 Amazon Web Services 上开始使用 Terraform 所需的所有内容，从托管数据库服务器到日志处理、存储、凭证、Docker 注册表和 EC2 实例。

第三章，*深入使用 Terraform*，阐述了使用 Terraform 代码的一些更高级技术，如动态数据源、独立环境、Docker、GitHub 或 StatusCake 集成、团队协作，以及代码检查工具的工作原理。

第四章，*使用 Terraform 自动化完整基础设施*，将展示和描述针对 Amazon Web Services、Digital Ocean、OpenStack、Heroku、Packet 和 Google Cloud 等平台的完整实际 Terraform 代码。我们将为容器部署一个 Docker Swarm 集群在裸金属 CoreOS 集群上，构建一个 n 层 Web 基础设施，或搭建一个 GitLab + CI 组合。

第五章，*使用 Cloud-Init 配置最后一公里*，探索了我们可以使用 cloud-init 代码做的所有事情——文件管理、服务器配置、添加用户和密钥、仓库和软件包，或如 Chef、CoreOS 和 Docker 等扩展的示例。

第六章，*使用 Chef 和 Puppet 管理服务器的基础知识*，展示了使用 Chef 代码自动化基础设施的要点。从工作站设置到编写我们自己的食谱，再到管理外部食谱，本章涵盖了所有内容——我们将使用代码管理软件包、服务、文件、动态模板、依赖关系、关系、共享数据等。还展示了使用 Puppet 代码执行类似操作的替代方法，以便你更好地了解这个生态系统。

第七章，*使用 Chef 和 Puppet 测试和编写更好的基础设施代码*，专注于测试代码质量和可持续性的高级技术。它还涵盖了单元和集成测试、代码检查工具以及 Chef 和 Puppet 的相关工具，确保你能够编写出最佳的基础设施代码。

第八章，*使用 Chef 和 Puppet 维护系统*，展示了 Chef 或 Puppet 代码所能实现的高级功能，如计划收敛、加密的机密、环境、实时系统信息检索、应用程序部署以及安全的工作流或实践。

第九章，*使用 Docker*，是从开发者的角度来使用 Docker 容器——选择基础镜像、优化、标签、版本管理、部署 Ruby-on-Rails 或 Go 应用、网络、安全、代码检查，以及使用我们自己的持久私有注册表——这一切都通过简单的 Docker 指令来实现——作为代码。

第十章，*维护 Docker 容器*，展示了开发人员和工程师使用更高级的 Docker 功能，如代码测试、自动构建流水线和持续集成、自动化漏洞扫描、监控和调试。

# 本书所需的条件

基本要求是能够运行 Linux 虚拟机的计算机和互联网连接。作者的电脑是运行 Mac OS 10.11 和 Fedora 25 的笔记本，配有 VirtualBox 5，但任何其他 Linux 发行版也可以使用。Vagrant、Terraform、Chef 开发工具包和 Docker 也可以在 Windows 平台上运行，尽管作者未对此进行测试。

由于我们在处理基础设施即服务（IaaS），因此还需要拥有有效的 Amazon Web Services (AWS)、Google Cloud、Digital Ocean、Packet、Heroku 或 OpenStack 部署的账户。

在本书的各个章节中，我们还将使用一些免费的软件即服务（SaaS）账户，如 GitHub、Travis CI、Docker Hub、Quay.io、Hosted Chef 和 StatusCake。

# 本书适用对象

本书面向的是那些在跨职能团队或运维部门工作的 DevOps 工程师和开发人员，适合那些希望切换到 IAC 以管理复杂基础设施的人员。

# 章节

在本书中，你会看到一些经常出现的标题（准备工作、如何操作……、如何运作……、还有更多……以及参见）。

为了清楚地说明如何完成一个食谱，我们使用以下几个部分：

## 准备工作

本节告诉你在食谱中会遇到什么，并描述如何设置所需的软件或任何前期设置。

## 如何操作…

本节包含完成该食谱所需的步骤。

## 如何运作…

本节通常详细解释上一节中发生的事情。

## 还有更多…

本节包含有关该食谱的附加信息，旨在让读者更深入地了解该食谱。

## 参见

本节提供了有关该食谱的有用链接，以帮助获取更多有用的信息。

# 约定

本书中，你会发现多种文本样式，它们用来区分不同类型的信息。以下是这些样式的一些例子及其含义的解释。

文本中的代码词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名的展示方式如下：“包含前一个食谱中的 NGINX 配置和`docker-compose.yml`文件，然后你就可以开始了。”

一段代码设置如下：

```
Vagrant.configure("2") do |config|
  # all your Vagrant configuration here
end
```

当我们希望引起您对代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```
    config.vm.provision "ansible_local" do |ansible|
 ansible.version = "1.9.6"
 ansible.install_mode = :pip
      ansible.playbook = "playbook.yml"
    end
```

任何命令行输入或输出都按以下方式书写：

```
$ vagrant plugin list
vagrant-vbguest (0.13.0)

```

**新术语**和**重要词汇**以粗体显示。在屏幕上看到的词语，例如在菜单或对话框中，文本中会这样显示：“您可以通过登录到 AWS 控制台并导航到**EC2 仪表板** | **网络与安全** | **安全组**来查看您新创建的安全组。”

### 注意

警告或重要说明通常会以这种框架显示。

### 小贴士

小贴士和技巧以这种方式出现。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中受益的书籍。

要向我们提供一般反馈，请直接发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您擅长某个主题并有兴趣编写或贡献一本书，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有很多资源可以帮助您充分利用您的购买。

## 下载示例代码

您可以从您的账户中下载本书的示例代码文件，网址为：[`www.packtpub.com`](http://www.packtpub.com)。如果您是从其他地方购买的本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，并注册以便将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**SUPPORT**标签上。

1.  点击**代码下载与勘误表**。

1.  在**搜索**框中输入书名。

1.  选择您想下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的来源。

1.  点击**代码下载**。

一旦文件下载完成，请确保使用最新版本的工具解压或提取文件夹：

+   Windows 版 WinRAR / 7-Zip

+   Mac 版 Zipeg / iZip / UnRarX

+   Linux 版 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，网址为：[`github.com/PacktPublishing/Infrastructure-as-Code-IAC-Cookbook`](https://github.com/PacktPublishing/Infrastructure-as-Code-IAC-Cookbook)。我们还有来自我们丰富的书籍和视频目录中的其他代码包，可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。快去看看吧！

## 下载本书的彩色图像

我们还为您提供了本书中使用的截图/图表的彩色图片 PDF 文件。彩色图片将帮助您更好地理解输出中的变化。您可以通过以下链接下载该文件：[`www.packtpub.com/sites/default/files/downloads/InfrastructureasCode_IAC_Cookbook_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/InfrastructureasCode_IAC_Cookbook_ColorImages.pdf)

## 勘误

尽管我们已经尽力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现错误——例如文本或代码中的错误——我们非常感谢您向我们报告。通过这样做，您可以避免其他读者的困扰，并帮助我们改进后续版本的书籍。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata) 提交，选择您的书籍，点击**勘误提交表格**链接，输入勘误的详细信息。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该书籍的勘误部分。

要查看之前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。相关信息将显示在**勘误**部分。

## 盗版

互联网上的版权材料盗版问题是所有媒体中的持续问题。在 Packt，我们非常重视版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即提供相关地址或网站名称，以便我们采取相应的措施。

请通过电子邮件 `<copyright@packtpub.com>` 与我们联系，提供涉嫌盗版材料的链接。

我们感谢您帮助保护我们的作者及我们为您带来有价值内容的能力。

## 问题

如果您在本书的任何部分遇到问题，您可以通过 `<questions@packtpub.com>` 与我们联系，我们会尽力解决问题。

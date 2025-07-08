# 序言

Puppet 5 仍然是首选的软件配置管理工具，尤其适用于大规模配置。

以下是一些当前现实世界中使用 Puppet 的例子：

+   Twitter 使用 Puppet 管理其目前为止最大的社交网络基础设施之一([`blog.twitter.com/engineering/en_us/topics/infrastructure/2017/the-infrastructure-behind-twitter-scale.html`](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2017/the-infrastructure-behind-twitter-scale.html))。Facebook 则使用 Opscode Chef，这是软件配置管理领域的竞争产品。

+   Uber 使用 Puppet 进行其标准配置管理([`eng.uber.com/uchat/`](https://eng.uber.com/uchat/))。

+   Walmart 也是一个非常大的 Puppet 用户([`puppet.com/blog/how-walmart-scaled-puppet-55K-nodes-and-beyond`](https://puppet.com/blog/how-walmart-scaled-puppet-55K-nodes-and-beyond))。

尽管有 Ansible 和 Salt 等更新的产品，但 Puppet 依然是——我相信——首选工具，尤其适用于大型基础设施（10,000+台服务器）。值得一提的是，Ansible 也变得非常流行，这可能与其较浅的学习曲线以及被 Red Hat 采纳有关。

处理如此规模和复杂度的问题并非易事。在本书《**Mastering Puppet 5**》中，我们希望为你提供所需的技术，让你能够应对自己的大规模挑战，达到精通级别。

Puppet 5 版本是本书所涵盖的版本，它在去年的 PuppetConf（2017）上由新 CEO Sanjay Mirchandani 盛大发布，被宣布为<q>"*Puppet 有史以来最大的一次产品创新*"</q>。在本书中，我们详细讲解了这些新技术，包括 Puppet Discovery、Puppet Tasks 和 Puppet Pipelines，帮助你掌握使用 Puppet 5 的技能，让你在实际环境中自信使用。

# 本书适合的人群

如果你是一名已经使用过 Puppet 的系统管理员或开发人员，并希望在企业环境中、大规模使用 Puppet，掌握更高级的技能和最佳实践，那么这本书适合你。你需要具备一些使用 Puppet 的初级知识。

如果你在开始使用本书之前，希望先获得 Puppet 的入门知识，请查看 Puppet 的免费自学培训课程([`learn.puppet.com/category/self-paced-training`](https://learn.puppet.com/category/self-paced-training))，或者参加一些由讲师主导的培训或私人培训课程([`puppet.com/support-services/training`](https://puppet.com/support-services/training))。

# 本书内容

第一章，*编写模块*，将真正帮助你迈上正确的道路，编写更高质量的 Puppet 模块和清单，并介绍了 12 个模块编写的最佳实践。

第二章，*角色和配置文件*，介绍了两种额外的抽象层和改进的接口，使您的层次化业务数据更容易集成，使系统配置更易于阅读，并且使重构变得更加容易。

第三章，*扩展 Puppet*，涵盖了生态系统的三个部分，这些部分仍然可以通过 Ruby 层访问，以扩展 Puppet 以适应更高级的使用场景；即自定义事实、自定义函数以及类型和提供者。

第四章，*Hiera 5*，涵盖了 Hiera 的最新版本，它允许我们将所有特定于站点和业务的数据与我们的清单分开，从而使我们的 Puppet 模块更具可移植性。在本章中，我们还简要查看了配置和数据的三层结构：全局、环境和模块。我们还介绍了如何设置加密的 YAML 后端。最后，我们简要了解了如何使用 Jerakia 扩展 Hiera。

第五章，*代码管理*，涵盖了 r10k 和 Code Manager 的使用，允许我们将所有 Puppet 代码存储在 Git 仓库中，并提供版本控制和回滚功能。我们讨论了目录环境，它们使我们能够在单个主机上使用多个版本的代码，并且 r10k 和 Code Manager 支持这种方式。我们将构建一个 Puppetfile，并积极将代码部署到我们的 Puppet Master。

第六章，*工作流*，涵盖了一个基本的 Puppet 工作流。我们将更加深入地将 PDK 融入我们的基本工作流中，使我们能够更高效地编写代码。

第七章，*持续集成*，涵盖了将 Puppet 与 Jenkins 集成作为**持续集成**（**CI**）系统。我们将讨论**CI/持续部署**（**CI/CD**）管道的组件，实现达到这些里程碑所需的条件，并在 CI 系统中积极改进我们的 Puppet 代码。

第八章，*通过任务和发现扩展 Puppet*，涵盖了 Puppet 任务和 Puppet 发现。Puppet 任务允许我们运行临时命令并将其作为命令式脚本的构建块。我们将构建一个任务来检查日志文件，并计划为我们的 Puppet Master 构建一个聚合日志文件。Puppet 发现允许我们检查现有的基础设施，并确定虚拟机或容器中包、服务、用户以及各种其他组件的真实情况。

第九章，*导出资源*，介绍了 Puppet 中的虚拟和导出资源。我们将探索如何在清单中导出和收集资源，并介绍导出和收集资源的一些常见使用场景，包括以下内容：动态 `/etc/hosts` 文件、负载均衡和自动数据库连接。我们还将探索 `file_line` 和 `concat` 资源，允许我们基于这些导出资源构建动态配置文件。

第十章，*应用编排*，介绍了多个节点运行的顺序。我们将构建应用编排清单，使我们能够将节点连接在一起，并在多个节点之间提供配置信息，确保我们的多节点应用按所需顺序运行，并且拥有所需的信息。

第十一章，*Puppet 扩展性*，介绍了 Puppet 的水平和垂直扩展。我们将探索一些常见的调优设置，并检查如何水平扩展 Puppet 服务。

第十二章，*故障排除与分析*，介绍了在使用 Puppet 时常见的故障排除案例。我们将重点讨论 Puppet 服务错误和清单编译错误，并检查日志文件的调优和配置。

# 为了充分利用本书的内容

对于有一定 Puppet 经验的用户，本书的收益最大，但每一课都是为了帮助任何阶段的学习者。为了跟随本书的内容，用户应该安装 Puppet Enterprise 的试用版，并将一些节点附加到 Puppet Master。每一章都会帮助配置 Puppet Master，使其能够利用现有的基础设施。

Puppet Enterprise 安装说明可以在 [`puppet.com/docs/pe/latest/installing_pe.html`](https://puppet.com/docs/pe/latest/installing_pe.html) 找到。

# 下载示例代码文件

您可以从您的账户在 [www.packt.com](http://www.packtpub.com) 下载本书的示例代码文件。如果您从其他地方购买了本书，您可以访问 [www.packt.com/support](http://www.packtpub.com/support) 并注册，将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  登录或注册 [www.packt.com](http://www.packtpub.com/support)。

1.  选择 SUPPORT 标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用以下最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/Mastering-Puppet-5`](https://github.com/PacktPublishing/Mastering-Puppet-5)。如果代码有更新，它将在现有的 GitHub 仓库中进行更新。

我们还有来自我们丰富书籍和视频目录的其他代码包，您可以在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**查看。请查看！

# 下载彩色图片

我们还提供了一个包含本书中截图/图表彩色图片的 PDF 文件。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781788831864_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781788831864_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。以下是一个示例：“Hiera 的早期版本（版本 3 或更早）使用的是单一的、完全全局的 `hiera.yaml`。”

一段代码如下所示：

```
lookup({
   'name'  => 'classification',
   'merge' => {
     'strategy'        => 'deep',
     'knockout_prefix' => '--',
   },
 })
```

当我们希望引起您对代码块中特定部分的注意时，相关行或项会以粗体显示：

```
lookup({
   'name'  => 'classification',
   'merge' => {
     'strategy' => 'deep',
     'knockout_prefix' => '--',
   },
 })
```

任何命令行输入或输出如下所示：

```
$ sudo /opt/puppetlabs/puppet/bin/gem install hiera-eyaml
```

**粗体**：表示一个新术语、重要单词或屏幕上显示的单词。例如，菜单或对话框中的词语会像这样出现在文本中。以下是一个示例：“我们可以通过点击屏幕左侧的 Manage Jenkins 进入插件页面。”

警告或重要提示如下所示。

提示和技巧如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请通过电子邮件 `customercare@packtpub.com` 联系，并在邮件主题中提及书名。如果您对本书的任何内容有疑问，请通过电子邮件联系我们：`customercare@packtpub.com`。

**勘误表**：虽然我们已尽力确保内容的准确性，但错误难免发生。如果您在本书中发现错误，我们将非常感激您向我们报告。请访问 [www.packt.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误表提交表格链接，并输入详细信息。

**盗版**：如果您在互联网上遇到我们作品的任何非法复制形式，我们将非常感激您提供相关地址或网站名称。请通过 `copyright@packt.com` 与我们联系，并提供相关材料的链接。

**如果您有兴趣成为作者**：如果您对某个话题有专业知识，并且有兴趣撰写或参与书籍的编写，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评价。一旦你阅读并使用了这本书，为什么不在你购买书籍的网站上留下评价呢？潜在的读者可以看到并参考你的客观意见来做出购买决策，我们在 Packt 也能了解你对我们产品的看法，而我们的作者也能看到你对他们书籍的反馈。谢谢！

欲了解更多有关 Packt 的信息，请访问[packtpub.com](https://www.packtpub.com/)。

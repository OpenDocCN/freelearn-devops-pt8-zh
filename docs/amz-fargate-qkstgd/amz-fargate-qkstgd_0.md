# 前言

在 AWS re:Invent 2017 上，宣布了一种名为 Fargate 的新容器启动类型。本书内容就是关于 Amazon Fargate，这个针对 Docker 容器的 Amazon **弹性容器服务**（**ECS**）的新启动类型。

Docker 是事实上的标准容器化框架，彻底改变了软件的打包和部署。Amazon Fargate 使 ECS 平台实现了无服务器化；无需配置、运行或管理 EC2 实例。Amazon Fargate 在 Docker 容器上运行 ECS 服务，并将服务任务直接暴露给用户。Fargate 通过实现无服务器化，大大简化了 ECS 平台。

Amazon Fargate 提供以下好处：

+   它在 AWS ECS 上得到支持，AWS EKS 也将在未来添加支持

+   它与其他 AWS 服务完全集成，包括 IAM、CloudWatch 日志和 EC2 负载均衡

+   Amazon Fargate 是无服务器的，减少了 ECS 集群的管理；无需配置、创建或管理 EC2 实例

+   ECS 服务任务直接暴露给用户

+   每个任务都会创建一个弹性网络接口

+   支持 ECS 任务的自动扩展

+   提供了一个 ECS CLI，用于创建 ECS 集群并使用 Docker Compose 部署任务

Amazon Fargate 通过 Fargate 平台启动集群，而不是在 EC2 实例上启动 ECS 集群。

# 本书适合谁阅读

主要的读者群体是 Docker 用户和开发者。如果你从事以下角色，本书对你会很有帮助：

+   DevOps 架构师

+   Docker 专家

+   DevOps 工程师（Docker）

# 本书内容

第一章，*开始使用 Amazon ECS 和 Amazon Fargate*，是本书的引言部分，讨论了使用 Fargate 启动类型的好处，如何使用 Fargate 启动类型分配和配置计算资源，Fargate 中的 ECS 对象与 EC2 启动类型相同，以及 Fargate 中的新特性。

第二章，*网络配置*，介绍了 Fargate 启动类型在 ECS 集群中的使用方式。我们将使用 Hello World Docker 镜像创建一个包含容器、任务和服务定义的集群。随后，我们将使用与任务相关联的弹性网络接口的 IPv4 公共 IP 调用 Hello World 应用程序。

第三章，*使用 CloudWatch 日志*，讨论了配置 ECS 容器进行日志记录。我们通过 ECS 服务为 MySQL 数据库演示了 CloudWatch 日志。Fargate 启动类型唯一支持的日志驱动程序是 awslogs 驱动程序。

第四章，*使用自动扩展*，介绍了与 Fargate 启动类型一起使用的 ECS 服务自动扩展。配置自动扩展需要设置任务数量的范围（最小值和最大值），在该范围内应用自动扩展。

第五章，*使用 IAM*，讨论了为 ECS 配置 IAM 用户。根用户不需要进行权限配置，并且可以访问 ECS 资源、ECS 控制台、ECS API 及所有必需的 AWS 服务。如果要使用 IAM 用户进行 ECS 操作，必须为该 IAM 用户添加所需的 IAM 策略。

第六章，*使用应用程序负载均衡器*，讨论了如何配置使用 Fargate 启动类型的 ECS 服务，并通过应用程序负载均衡器平衡 Hello World 服务的 HTTP 请求。

第七章，*使用 Amazon ECS CLI*，讨论了如何使用 ECS CLI 创建一个 Fargate 启动类型的集群。接着，我们将在集群上部署一个 Docker Compose 文件，运行一个 WordPress Docker 镜像的任务。

# 为了充分利用本书

你应该具有一定的 Docker 基础知识，因为 Fargate 平台是 Amazon ECS 的一种启动模式，ECS 是一个用于 Docker 容器的托管服务。你还应该具备一些 AWS 和 ECS 的基本知识。

如果你还没有 AWS 账户，你需要在[`aws.amazon.com/resources/create-account/`](https://aws.amazon.com/resources/create-account/)创建一个。

# 下载示例代码文件

你可以从你的账户在[www.packtpub.com](http://www.packtpub.com)下载本书的示例代码文件。如果你是在其他地方购买的这本书，你可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，代码文件会直接通过邮件发送给你。

你可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择“SUPPORT”标签。

1.  点击“代码下载与勘误”

1.  在搜索框中输入书名并按照屏幕上的指示操作。

文件下载后，请确保使用最新版本的解压缩工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Amazon-Fargate-Quick-Start-Guide`](https://github.com/PacktPublishing/Amazon-Fargate-Quick-Start-Guide)。如果代码有更新，它会在现有的 GitHub 仓库中进行更新。

我们还提供了其他代码包，来自我们丰富的书籍和视频目录，可以在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**找到。快去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的截图/图表的彩色图片。你可以在此下载：[`www.packtpub.com/sites/default/files/downloads/AmazonFargateQuickStartGuide_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/AmazonFargateQuickStartGuide_ColorImages.pdf)。

# 使用的约定

本书中使用了一些文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账户。示例：“验证版本为 3.0，使用`get-host`命令。”

一段代码如下所示：

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

任何命令行输入或输出如下所示：

```
 Set-ExecutionPolicy RemoteSigned
```

**粗体**：表示新术语、重要词汇或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中通常这样显示。示例：“`编辑容器`对话框会显示出来。”

警告或重要提示如下所示。

提示和技巧如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请通过电子邮件 `feedback@packtpub.com` 联系我们，并在邮件主题中注明书名。如果您对本书的任何内容有疑问，请通过电子邮件 `questions@packtpub.com` 与我们联系。

**勘误**：尽管我们已尽力确保内容的准确性，但错误仍然会发生。如果您在本书中发现了错误，我们将不胜感激您能向我们报告。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们的作品的任何非法复制版本，我们将不胜感激您能提供该材料的地址或网站名称。请通过 `copyright@packtpub.com` 联系我们，并附上该材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题领域具有专长，并且有兴趣撰写或参与书籍编写，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。阅读并使用本书后，为什么不在购买该书的网站上留下评论呢？潜在的读者可以看到并利用您的公正意见做出购买决策，我们在 Packt 可以了解您对我们产品的看法，我们的作者也能看到您对他们书籍的反馈。谢谢！

欲了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。

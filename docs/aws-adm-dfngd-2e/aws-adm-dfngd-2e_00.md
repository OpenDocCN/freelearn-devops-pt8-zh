# 前言

亚马逊云服务现在已经成为客户和企业的首选云服务很长一段时间了。这家云服务提供商已经从仅仅是基础设施即服务提供商发展为一切和任何服务，帮助开发应用程序、游戏开发、物联网、大数据分析、客户参与服务、AR-VR 等等！然而，随着每年推出这么多服务和产品，对于初学者来说，很难知道从哪里开始以及如何开始使用这些服务。

这本书是一个一站式商店，您可以找到所有关于开始使用 AWS 服务的内容，其中包括 EC2 系统管理器，弹性 Beanstalk，EFS，CloudTrail，EMR，IoT 等等！如果您是系统管理员、架构师或只是想学习和探索 AWS 服务的各个方面的人，那么这本书就是您的正确选择！本书的每一章都旨在帮助您理解各个服务的概念，并通过练习简单易行的步骤获得实践经验。该书还强调了一些您在使用 AWS 时应该牢记的关键最佳实践和建议。

# 本书适合对象

本书适用于希望学习和实施 AWS 用于其自身环境和应用程序托管的所有 IT 专业人员。虽然不需要任何先前经验或知识，但对于您具有基本的 Linux 知识和对网络概念和服务器虚拟化有一定了解将是有益的。

# 要充分利用本书

要开始使用本书，您需要在本地桌面上安装以下软件：

+   诸如 PuTTY 这样的 SSH 客户端、PuTTYgen 这样的密钥生成器以及 WinSCP 这样的文件传输工具

+   任何现代网络浏览器，最好是 Mozilla Firefox

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)注册，以便直接通过电子邮件接收文件。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名并按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩文件夹：

+   用于 Windows 的 WinRAR/7-Zip

+   用于 Mac 的 Zipeg/iZip/UnRarX

+   用于 Linux 的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/AWS-Administration-The-Definitive-Guide-Second-Edition`](https://github.com/PacktPublishing/AWS-Administration-The-Definitive-Guide-Second-Edition)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还提供了丰富的书籍和视频资源，您可以在 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)** 找到更多的代码包。快来查看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号。例如：“该文档包括两个主要部分：一个是 `Parameters` 部分，其中包含文档要执行的操作列表，接着是 `mainSteps` 部分，指定文档要执行的操作，在此案例中为 `aws:configurePackage`。在此情况下，当文档被调用时，系统将提示用户从下拉列表中选择 `apache2`、`mysql-server`或 `php`，并可以选择软件版本号。”

代码块格式如下：

```
{ 
    "Effect": "Allow", 
    "Action": [ 
      "ec2messages:AcknowledgeMessage", 
      "ec2messages:DeleteMessage", 
      "ec2messages:FailMessage", 
      "ec2messages:GetEndpoint", 
      "ec2messages:GetMessages", 
      "ec2messages:SendReply" 
    ], 
    "Resource": "*" 
}, 
```

当我们希望引起您注意某个代码块的特定部分时，相关行或项目会用粗体显示：

```
{ 
    "Effect": "Allow", 
    "Action": [ 
      "ec2messages:AcknowledgeMessage", 
      "ec2messages:DeleteMessage", 
      "ec2messages:FailMessage", 
      "ec2messages:GetEndpoint", 
      "ec2messages:GetMessages", 
      "ec2messages:SendReply" 
    ], 
    "Resource": "*" 
}, 
```

所有命令行输入或输出如下所示：

```
# wget https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/debian_amd64/amazon-ssm-agent.deb
```

**粗体**：表示一个新术语、一个重要的词汇，或是你在屏幕上看到的词汇。例如，菜单或对话框中的词汇会以这种形式出现在文本中。这里有一个例子：“在 Create Role 向导中，从 AWS 服务角色类型中选择 EC2 选项，如下图所示。接下来，选择 EC2 作为此活动的*用例*，然后点击 Next: Permissions 按钮继续。”

警告或重要注意事项如下所示。

小贴士如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请通过电子邮件发送至`feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`联系我们。

**勘误**：虽然我们已尽一切努力确保内容的准确性，但错误难免。如果您在本书中发现任何错误，我们将非常感激您向我们报告。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接并输入相关信息。

**盗版**：如果您在互联网上遇到我们作品的任何非法复制品，恳请您提供相关位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并附上相关链接。

**如果您有兴趣成为作者**：如果您在某个领域拥有专业知识，并且有兴趣撰写或参与书籍的创作，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 书评

请留下评价。一旦您阅读并使用了本书，为什么不在您购买书籍的网站上留下评价呢？潜在读者可以看到并参考您的公正意见来做出购买决策，我们在 Packt 可以了解您对我们产品的看法，我们的作者也能看到您对其书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。

# 第十一章：编排云资源

|   | *休息不是懒散，有时在夏日的树荫下躺在草地上，听着水流的低语，或看着云朵在天空中漂浮，这绝不是浪费时间。* |   |
| --- | --- | --- |
|   | --*约翰·拉博克* |

在本章中，你将学习如何使用`puppetlabs/aws`模块来创建和管理 Amazon AWS 云实例，以及关联的资源，如子网、安全组和 VPC。你还将学习如何直接从 Hiera 数据构建整个云基础设施。

![编排云资源](img/8880_11_01.jpg)

# 云的介绍

在探索云计算的优势之前，也许我们应该先定义一下它是什么。在云计算普及之前，如果你需要计算能力，你会购买一台实际的、物理存在的计算机。但从客户的角度来看，我们并不一定想要一台计算机：我们只想要计算能力。我们希望能够根据需要，随时购买所需的计算资源，而不是为一台专用计算机支付高昂的固定费用。

进入**虚拟化**。一台物理服务器可以提供大量的虚拟服务器，每台虚拟服务器（理论上）彼此完全隔离。托管服务提供商建立了一个平台（由许多物理服务器通过网络连接而成），从客户的角度来看，这个平台提供了一个庞大的、无形的**云**虚拟计算资源（因此得名）。

## 自动化云资源配置

创建新的云实例比购买物理硬件更便宜、更容易，但你仍然需要做出一些选择：实例的 CPU 或内存大小，硬盘空间的多少，硬盘的种类（物理硬盘、固态硬盘、网络附加存储），应安装的操作系统，实例是否需要公共 IP 地址，应该配置哪些防火墙规则等等。

如果你已经阅读到本书的这一部分，你应该已经认识到这是一个**配置管理**问题。你可能已经有了一些关于我将推荐什么方案来解决它的想法，但首先让我们来看一下几种可用的选项。

## 使用 CloudFormation

CloudFormation 是专门针对**Amazon Web Services**（**AWS**）的模板语言。它以声明的方式描述 AWS 资源，有点像 Puppet 资源。你将 CloudFormation 模板上传到 AWS 门户（或 API），应用它，AWS 会创建所有指定的资源。以下示例展示了一段 CloudFormation 代码：

```
 "Resources" : {
    "EC2Instance" : {
      "Type" : "AWS::EC2::Instance",
      "Properties" : {
        "InstanceType" : { "Ref" : "InstanceType" },
        "SecurityGroups" : [ { "Ref" : "InstanceSecurityGroup" } ],
        "KeyName" : { "Ref" : "KeyName" },
        "ImageId" : { "Fn::FindInMap" : [ "AWSRegionArch2AMI", { "Ref" : "AWS::Region" },
                          { "Fn::FindInMap" : [ "AWSInstanceType2Arch", { "Ref" : "InstanceType" }, "Arch" ] } ] }
      }
    },
```

坦率地说，这个编程环境并不怎么有趣。尽管它从技术上来说是基础设施即代码，但它相当基础。尽管如此，它仍然比通过网页浏览器手动设置 AWS 基础设施有所进步。

## 使用 Terraform

Terraform 是一个相对复杂的云资源配置工具。它允许你以声明性方式描述资源，像 CloudFormation 一样，但在一个略高的抽象层次上，并且不是 AWS 特定的。以下示例展示了 Terraform 代码的样子：

```
resource "aws_instance" "web" {
  ami           = "${data.aws_ami.ubuntu.id}"
  instance_type = "t2.micro"

  tags {
    Name = "HelloWorld"
  }
}
```

Terraform 是一项有前景的技术，但可以说它仍处于开发的早期阶段。

## 使用 Puppet

用于管理云基础设施的独立工具是可以的，但如果我们用 Puppet 做其他所有事情，引入一个全新的工具仅仅为了这个似乎有些不值。所以我们能否用 Puppet 来管理云资源呢？

幸运的是，Puppet 提供了一个优秀的 Forge 模块（`puppetlabs/aws`），它正是为此而设计的。在本章的其余部分，我们将通过一些示例，展示如何使用 `puppetlabs/aws` 来管理 AWS 云资源。

# 设置 Amazon AWS 账户

如果你已经拥有 AWS 账户，可以跳到下一部分。否则，你可以按照以下说明设置一个新账户，并获取你开始使用 Puppet 构建基础设施所需的凭证。

## 创建 AWS 账户

按照以下步骤创建一个新的 AWS 账户：

1.  浏览到以下 URL：

    [`aws.amazon.com/`](https://aws.amazon.com/)

1.  点击 **登录到控制台**。

1.  按照指示创建并验证你的账户。

为了使用 Puppet 管理 AWS 资源，我们将创建一个额外的 AWS 用户账户专门用于 Puppet，使用 Amazon 的 **身份与访问管理**（**IAM**）框架。接下来的章节将演示如何操作。

## 创建一个 IAM 策略

在创建 Puppet 用户账户之前，我们需要为它需要执行的任务授予特定权限，例如读取和创建 EC2 实例。这涉及到创建一个 **IAM 策略**，它是一组你可以与用户账户关联的命名权限。

IAM 策略以 JSON 格式文档的形式表达。在示例仓库中有一个策略 JSON 文件，名为 `/examples/iam_policy.json`。打开此文件并复制其内容，准备粘贴到你的网页浏览器中。

按照以下步骤创建策略并将其与 Puppet 用户关联：

1.  在 AWS 控制台中，选择 **服务** | **IAM**。

1.  选择 **策略**。

1.  点击 **创建策略**。

1.  在 **创建策略** 屏幕上，选择 **创建你自己的策略**。

1.  输入 **策略名称**（例如，`puppet`）。

1.  在 **策略文档** 文本框中，粘贴从 `iam_policy.json` 文件中复制的文本。

1.  点击底部的 **创建策略** 以保存此设置。![创建 IAM 策略](img/8880_11_02.jpg)

## 创建一个 IAM 用户

要创建 Puppet IAM 用户并将其与策略关联，请按照以下步骤操作：

1.  登录 AWS 控制台。

1.  选择 **服务** | **IAM** | **用户**。

1.  点击 **添加用户**。

1.  输入你希望用于该账户的用户名（例如，`puppet`）。

1.  在 **访问类型** 部分，选择 **编程访问**。![创建 IAM 用户](img/8880_11_03.jpg)

1.  点击 **下一步：权限**。

1.  直接创建**附加现有策略**。

1.  在**策略类型**搜索框中输入`puppet`并按*回车*。![创建 IAM 用户](img/8880_11_04.jpg)

1.  您应该会看到我们在上一节中创建的策略，勾选旁边的框并点击**下一步：审查**。

1.  检查设置是否正确，然后点击**创建用户**。

当您完成创建 IAM 用户和策略后，应该会看到**成功**屏幕，其中列出了您的访问凭证。复制访问密钥 ID 和秘密访问密钥（点击**显示**查看秘密访问密钥）。您将在接下来的步骤中需要这些凭证（但请妥善保管）。

## 存储您的 AWS 凭证

按照以下步骤配置您的虚拟机以使用新生成的凭证访问 AWS：

1.  在您的 Vagrant 虚拟机上，运行以下命令以创建一个目录来存放凭证文件：

    ```
    mkdir /home/ubuntu/.aws

    ```

1.  创建一个名为`/home/ubuntu/.aws/credentials`的文件，内容如下（从 AWS 控制台屏幕中替换您的访问密钥 ID 和秘密访问密钥值）：

    ```
    [default]
    aws_access_key_id = AKIAINSZUVFYMBFDJCEQ
    aws_secret_access_key = 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
    ```

### 提示

手动创建文件在本例中是可以的，但对于生产环境，您应该使用 Puppet 来管理凭证文件，使用加密的 Hiera 数据，如第六章，*使用 Hiera 管理数据*中所示。

# 准备使用 puppetlabs/aws

在接下来的部分中，我们将演示如何生成一个 SSH 密钥对以连接到您的 EC2 实例，并安装`puppetlabs/aws`模块及其依赖项。

## 创建密钥对

您需要一个 SSH 密钥对才能连接到您创建的任何 EC2 实例。在本节中，我们将生成并下载您的密钥对。

1.  在 AWS 控制台中，进入**EC2**部分，在左侧窗格中选择**密钥对**下的**网络与安全**。

1.  点击**创建密钥对**按钮。![创建密钥对](img/8880_11_05.jpg)

1.  系统会提示您输入密钥对的名称。对于本示例，请输入`pbg`。

1.  一个名为`pbg.pem`的文件将由浏览器自动下载。将该文件移动到您计算机的`~/.ssh`目录（或者，如果您更愿意从那里访问 AWS 实例，也可以将其复制到 Vagrant 虚拟机中`ubuntu`用户的`~/.ssh`目录）。

1.  使用以下命令设置密钥文件的正确权限：

    ```
    chmod 600 ~/.ssh/pbg.pem

    ```

## 安装 puppetlabs/aws 模块

按照以下步骤安装`puppetlabs/aws`模块：

如果您已经设置了`r10k`模块管理工具，如第七章，*掌握模块*中所示，所需的模块应该已经安装。如果没有，请运行以下命令来安装：

```
cd /etc/puppetlabs/code/environments/pbg
sudo r10k puppetfile install

```

## 安装 AWS SDK gem

`puppetlabs/aws`模块需要一些 gems，我们可以使用 Puppet 轻松安装，使用以下清单（`aws_sdk.pp`）：

```
ensure_packages([
  'aws-sdk-core',
  'retries'
],
  { provider => puppet_gem })
```

### 提示

你注意到示例中的 `provider => puppet_gem` 吗？你可能还记得在第四章，*理解 Puppet 资源* 中提到过，`puppet_gem` 会在 Puppet 环境中安装 Ruby gem（与系统 Ruby 环境完全独立）。Puppet 模块所需的 gem 需要这样安装，否则 Puppet 无法加载它们。

1.  使用以下命令应用清单：

    ```
    sudo puppet apply --environment pbg /examples/aws_sdk.pp

    ```

1.  创建 `/home/ubuntu/.aws/config` 文件，并添加以下内容：

    ```
    [default]
    region=us-east-1
    ```

# 使用 Puppet 创建 EC2 实例

尽管你可以使用 Puppet 管理许多不同类型的 AWS 资源，但最重要的资源是 EC2 实例（虚拟服务器）。在这一部分，我们将学习如何创建你的第一个 EC2 实例。

## 选择 Amazon Machine Image (AMI)

为了运行 EC2 实例，即 AWS 虚拟机，你需要从众多可用的虚拟机中选择一个。每个虚拟机快照称为 **Amazon Machine Image**（**AMI**），并且有一个唯一的 ID。你将使用这个 ID 添加到 Puppet 清单中，以告知它要启动哪种类型的实例。

对于本示例，选择哪个 AMI 并不太重要，但我们将使用官方的 Ubuntu 镜像。按照以下步骤查找：

1.  浏览到以下 URL：

    [`cloud-images.ubuntu.com/locator/ec2/`](https://cloud-images.ubuntu.com/locator/ec2/)

1.  在 **Search** 框中，输入 `us-east-1 xenial`

1.  你应该能看到 `us-east-1` 区域中列出的 Ubuntu Xenial AMI，它们有不同的实例类型，类似以下截图：![选择 Amazon Machine Image (AMI)](img/8880_11_06.jpg)

1.  在列表中找到 **Instance Type** 为 `ebs-ssd` 的 AMI。在之前的截图中，列表中的第三个 AMI（`ami-26d6d131`）是合适的。

`AMI-ID` 列中的十六进制代码，以 `ami-` 开头，就是 AMI ID。请记下这个 ID 以备后用。点击链接查看 AWS 实例类型选择页面，并确认你选择的 AMI 上有标记 **Free tier eligible**；这些 AMI 不会产生费用。如果启动非免费层的 AMI 实例，你将会被收费。

## 创建 EC2 实例

现在我们已经选择了合适的 AMI，准备使用 Puppet 创建 EC2 实例。

然而，在此之前，我们需要对 AWS 设置做一些更改，请按照以下步骤操作：

1.  在 AWS 控制台中，从 **Services** 菜单中选择 **VPC**。

1.  在左侧窗格中选择 **Your VPCs**。

1.  将只列出一个 VPC。点击 **Name** 字段并将其名称设置为 `default-vpc`

1.  在左侧窗格中选择 **Subnets**。

1.  会列出几个子网，每个子网对应一个可用区。找到与 `us-east-1a` 可用区相关联的子网。

1.  点击子网的 **Name** 字段，并将名称设置为 `default-subnet`

    ### 提示

    为什么在运行示例之前我们必须为 VPC 和子网设置名称？`puppetlabs/aws` 模块通过资源的“名称”来引用它们，而不是通过其 ID（像 AMI ID 这样的长十六进制代码）。尽管 AWS 会自动为你创建默认的 VPC 和子网，但它不会为它们分配名称，这意味着在我们为它们设置名称之前，我们无法在 Puppet 代码中引用它们。只要你在 Puppet 代码中的名称与 AWS 控制面板中分配的名称相同，名称的具体内容并不重要。我们将在本章后面了解更多关于 VPC 和子网的功能以及如何使用它们。

1.  编辑文件 `/examples/aws_instance.pp`，并将第一行中的 `$ami` 的值更改为你之前选择的 AMI ID（在我们的示例中是 `ami-26d6d131`）：

    ```
    sudo vi /examples/aws_instance.pp
    $ami = 'ami-26d6d131'
    ```

1.  保存文件，并运行以下命令：

    ```
    sudo puppet apply --environment pbg /examples/aws_instance.pp

    ```

1.  你应该会看到类似下面的 Puppet 输出：

    ```
    Notice: /Stage[main]/Main/Ec2_securitygroup[pbg-sg]/ensure: created
    Notice: /Stage[main]/Main/Ec2_instance[pbg-demo]/ensure: changed absent to running
    ```

1.  如果你检查 AWS 控制台中的 **EC2** 部分，你应该看到你的新实例的状态为 **初始化中**，很快就会准备好使用。

## 访问你的 EC2 实例

一旦新创建的实例的状态从 **初始化中** 变为 **运行中**（你可能需要点击 AWS 控制台中的刷新按钮），你就可以使用 SSH 和之前下载的密钥文件连接到它。

1.  在 AWS 控制台中，查找实例的**公共 IP** 地址并复制它。

1.  从你自己的机器（或者如果你将 `pbg.pem` 文件复制到 Vagrant VM 上，从 Vagrant VM 中）运行以下命令（将 `YOUR_INSTANCE_IP` 替换为实例的公共 IP 地址）：

    ```
    ssh -i ~/.ssh/pbg.pem -l ubuntu YOUR_INSTANCE_IP
    The authenticity of host 'YOUR_INSTANCE_IP (YOUR_INSTANCE_IP)' can't be established.
    ECDSA key fingerprint is SHA256:T/pyWVJYWys2nyASJVHmDqOkQf8PbRGru3vwwKH71sk.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'YOUR_INSTANCE_IP' (ECDSA) to the list of known hosts.
    Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-1030-aws x86_64)
    ```

### 提示

现在你已经获得了实例的 SSH 访问权限，你可以像对待物理节点一样使用 Puppet 对其进行引导，或者手动安装 Puppet 和 Git，并检查清单仓库。（我们将在第十二章中开发一个完整的自动化引导过程，*整合所有内容*。）

恭喜你！你刚刚用 Puppet 创建了你的第一个 EC2 实例。在下一节中，我们将查看代码并详细检查资源。

## VPC、子网和安全组

让我们通过示例清单来看一下它是如何工作的。但首先，我们需要了解一些关于 AWS 资源的信息。

一个 EC2 **实例**位于一个**子网**中，子网是一个自包含的虚拟网络。所有位于子网内的实例可以相互通信。子网是**虚拟私有云**（**VPC**）的划分，VPC 是特定于你 AWS 账户的私有内部网络。

实例还拥有一个**安全组**，它是一组管理网络访问实例的防火墙规则。

当你创建 AWS 账户时，你会获得一个默认的 VPC，该 VPC 被划分为多个子网，每个 AWS **可用区**（**AZ**）一个。我们在示例实例中使用的是默认 VPC 和其中一个默认子网，但由于我们还需要一个安全组，因此我们首先在 Puppet 代码中创建它。

## ec2_securitygroup 资源

示例清单的第一部分创建了所需的`ec2_securitygroup`资源（`aws_instance.pp`）：

```
ec2_securitygroup { 'pbg-sg':
  ensure      =>  present,
  description => 'PBG security group',
  region      => $region,
  vpc         => 'default-vpc',
  ingress     => [
    {
      description => 'SSH access from world',
      protocol    => 'tcp',
      port        => 22,
      cidr        => '0.0.0.0/0',
    },
    {
      description => 'Ping access from world',
      protocol    => 'icmp',
      cidr        => '0.0.0.0/0',
    },
  ],
}
```

首先，`ec2_securitygroup`有一个标题（`pbg-sg`），我们将使用这个标题从其他资源（如`ec2_instance`资源）引用它。它还有一个`description`，仅仅是为了提醒我们它的用途。

它属于一个`region`和一个`vpc`，并具有一组`ingress`规则。这些是你的防火墙规则。你希望允许的每个防火墙端口或协议都需要单独的入口规则。

每个入口规则都是类似以下内容的哈希：

```
{
  description => 'SSH access from world',
  protocol    => 'tcp',
  port        => 22,
  cidr        => '0.0.0.0/0',
}
```

`protocol`指定了流量类型（`tcp`、`udp`等）。

`port`是要打开的端口号（`22`是 SSH 端口，我们需要它来登录到实例）。

最后，`cidr`键指定了允许访问的网络地址范围。 (`0.0.0.0/0`表示“所有地址”。)

## `ec2_instance`资源

`ec2_instance`资源，顾名思义，用于管理单个 EC2 实例。以下是示例清单（`aws_instance.pp`）中的相关部分：

```
ec2_instance { 'pbg-demo':
  ensure                      => present,
  region                      => $region,
  subnet                      => 'default-subnet',
  security_groups             => 'pbg-sg',
  image_id                    => $ami,
  instance_type               => 't1.micro',
  associate_public_ip_address => true,
  key_name                    => 'pbg',
}
```

首先，`ensure => present`告诉 AWS 该实例应该处于运行状态。（你也可以使用`running`作为`present`的同义词。）将`ensure => absent`设置为会终止并删除实例（以及任何附加的临时存储）。

EC2 实例还可以处于第三种状态，即`stopped`。停止的实例会保留其存储，并可以重新启动。由于 AWS 按实例小时计费，因此停止的实例不需要付费，所以最好停止不需要立即运行的实例。

实例属于一个`region`和一个`subnet`，并且具有一个或多个`security_groups`。

`image_id`属性告诉 AWS 为实例使用哪个 AMI ID。

`instance_type`属性从 AWS 提供的多种类型中进行选择，这些类型大致对应于实例的计算能力（不同类型在内存大小、虚拟 CPU 数量以及其他一些因素上有所不同）。

由于我们处于一个私有网络中，实例无法从互联网访问，除非我们为其分配公共 IP 地址。将`associate_public_ip_address`设置为`true`启用此功能。（除非实例确实需要暴露端口到互联网，否则应将其设置为`false`。）

最后，实例有一个`key_name`属性，它告诉 AWS 我们将使用哪个 SSH 密钥来访问该实例。在这种情况下，我们使用的是在本章前面创建的名为`pbg`的密钥。

### 提示

在进入下一个示例之前，请终止实例以避免浪费免费小时。你可以通过在 AWS 控制面板中选择实例，然后点击**操作** | **实例状态** | **终止**，或者重新应用 Puppet 清单，将实例的`ensure`属性设置为`absent`来实现。

# 管理自定义 VPC 和子网

在前面的示例中，我们使用了预先存在的默认 VPC 和子网来创建实例。对于演示目的来说这样是可以的，但在生产环境中，您会希望使用专用的 VPC 来管理您的 Puppet 资源，以便将其与 AWS 账户中的其他资源以及其他 Puppet 管理的 VPC 区分开来。例如，您可以有一个暂存 VPC 和一个生产 VPC。

默认情况下，新的 VPC 无法访问 Internet；我们还需要一个**Internet 网关**（它将 Internet 流量路由到 VPC 并从 VPC 返回）和一个**路由表**（它告诉某个子网将非本地流量发送到网关）。`puppetlabs/aws`模块提供了用于创建和管理这些实体的 Puppet 资源。

## 在自定义 VPC 中创建实例

在本节中，我们将使用一个更复杂的示例清单来创建一个新的 VPC 和子网，并与之关联一个 Internet 网关和路由表，然后添加一个安全组和 EC2 实例。

按照以下步骤应用清单：

1.  编辑文件`/examples/aws_vpc.pp`，并将第一行中的`$ami`值更改为您之前选择的 AMI ID（在我们的示例中，`ami-26d6d131`）：

    ```
    sudo vi /examples/aws_vpc.pp
    $ami = 'ami-26d6d131'
    ```

1.  保存文件并运行以下命令：

    ```
    sudo puppet apply --environment pbg /examples/aws_vpc.pp

    ```

1.  您应该会看到如下 Puppet 输出：

    ```
    Notice: /Stage[main]/Main/Ec2_vpc[pbg-vpc]/ensure: created
    Notice: /Stage[main]/Main/Ec2_vpc_internet_gateway[pbg-igw]/ensure: created
    Notice: /Stage[main]/Main/Ec2_vpc_routetable[pbg-rt]/ensure: created
    Notice: /Stage[main]/Main/Ec2_vpc_subnet[pbg-vpc-subnet]/ensure: created
    Notice: /Stage[main]/Main/Ec2_securitygroup[pbg-vpc-sg]/ensure: created
    Notice: /Stage[main]/Main/Ec2_instance[pbg-vpc-demo]/ensure: changed absent to running
    ```

1.  如果您检查 AWS 控制台的**EC2**部分，您应该会看到新实例的状态是**初始化中**，很快就可以使用了。

## ec2_vpc 资源

让我们详细看一下示例清单。以下是`ec2_vpc`资源（`aws_vpc.pp`）：

```
ec2_vpc { 'pbg-vpc':
  ensure     => present,
  region     => $region,
  cidr_block => '10.99.0.0/16',
}
```

VPC 需要一个`region`属性和`cidr_block`，即 VPC 将使用的网络地址范围。（实际上，这不是必须的，因为如果不指定，AWS 会随机为您分配一个。我们在这里指定一个仅用于演示。）

### 提示

实际上，您的网络范围并不重要，因为它完全是内部的。然而，采用官方分配给私有网络的地址范围（如`10.x.y.z`）是一种好习惯。为了减少与您组织中其他分配地址范围发生冲突的可能性，您可以为`x`选择一个随机数字（我们在示例中使用了`99`）。

## ec2_vpc_internet_gateway 资源

我们之前看到，VPC 默认情况下是无法连接到 Internet 的。将 Internet 流量引入 VPC 的方式有很多，包括 VPN 和 Amazon **Elastic** **Load Balancers**（**ELB**），但在本示例中，我们将使用`ec2_vpc_internet_gateway`资源，其格式如下：

```
ec2_vpc_internet_gateway { 'pbg-igw':
  ensure => present,
  region => $region,
  vpc    => 'pbg-vpc',
}
```

网关有一个标题（`pbg-igw`），并且与特定的`region`和`vpc`关联。

## ec2_vpc_routetable 资源

在配置了`ec2_vpc_internet_gateway`后，我们现在需要设置一个路由表，以确定哪些流量需要发送到它。以下是示例中的`ec2_vpc_routetable`资源：

```
ec2_vpc_routetable { 'pbg-rt':
  ensure => present,
  region => $region,
  vpc    => 'pbg-vpc',
  routes => [
    {
      destination_cidr_block => '10.99.0.0/16',
      gateway                => 'local'
    },
    {
      destination_cidr_block => '0.0.0.0/0',
      gateway                => 'pbg-igw'
    },
  ],
}
```

和往常一样，路由表有一个标题，`region`和`vpc`。它还包含一个或多个路由的数组。

**路由**就像是网络数据包的路标。它告诉你：“如果你要去这个目的地，请走这个路口。”每一条路由都是一个哈希表，包含`destination_cidr_block`和`gateway`键。

在我们示例中的第一条路由是用于本地流量的（目的地为`10.99.0.0/16`网络，这是我们分配给 VPC 的网络）：

```
{
  destination_cidr_block => '10.99.0.0/16',
  gateway                => 'local'
}
```

这告诉目的地为`10.99.0.0/16`的流量它是本地流量；也就是说，不需要使用网关，因为它已经在所需的网络上。

第二条路由是针对所有其他流量的：

```
{
  destination_cidr_block => '0.0.0.0/0',
  gateway                => 'pbg-igw'
}
```

网络地址`0.0.0.0/0`匹配所有可能的网络地址（`10.99.0.0/16`的流量已经通过之前的路由被过滤掉，所以剩下的所有流量必须是互联网流量）。指定的网关是`pbg-igw`，即我们之前创建的`ec2_vpc_internet_gateway`。

所以，这个路由表等同于以下的流量路由指令：

+   目的地为`10.99.0.0/16`的流量，保持在此网络内

+   所有其他流量，请通过`pbg-igw`网关

这些路由对于单个 VPC 来说已经足够；如果你在 AWS 中有更复杂的网络配置，你将需要更复杂的路由表，但基本原则是相同的。

## `ec2_vpc_subnet`资源

如我们所见，子网是 VPC 网络的一个子部分，它使你能够为不同的资源组逻辑地划分 VPC。例如，你可能有一个子网，它可以通过互联网访问，供公共节点使用，另一个子网则供内部资源使用，如数据库或日志服务器。

在这个示例中，我们只有一个子网：

```
ec2_vpc_subnet { 'pbg-vpc-subnet':
  ensure            => present,
  vpc               => 'pbg-vpc',
  region            => $region,
  cidr_block        => '10.99.0.0/24',
  availability_zone => "${region}a",
  route_table       => 'pbg-rt',
}
```

它有一个标题，`vpc`和`region`。因为它是 VPC 网络的一个子部分，它还需要一个`cidr_block`来指定它占用网络地址空间的具体部分。这必须是你分配给包含 VPC 的网络地址的子集，就像在这个示例中一样。

子网存在于 AWS 的可用区内（相当于数据中心）。这些可用区以区域命名；例如，`us-east-1`区域有`us-east-1a`、`us-east-1b`等可用区。这使得你可以在不同的可用区中配置冗余资源，以便在一个可用区故障时，另一个可用区能够接管。然而，在这个示例中，我们只使用了一个可用区`us-east-1a`，并将其传递给`availability_zone`属性。

默认情况下，子网中的资源只能在子网内进行通信。为了允许子网内外的流量通信，我们需要将其与`route_table`关联。通过使用我们之前创建的`pbg-rt`路由表，我们可以通过`pbg-igw`网关发送互联网流量。

就是这样。`ec2_securitygroup`和`ec2_instance`资源和我们之前的示例差不多，唯一不同的是使用了新的子网。

## 其他 AWS 资源类型

Puppet 不仅仅限于管理 EC2 实例；`puppetlabs/aws`模块还支持 ELB 负载均衡器、Cloudwatch 警报、自动扩展组、弹性 IP、DHCP、VPN、IAM 用户和策略、RDS 数据库、S3 存储桶、SQS 队列、Route 53 DNS 管理和**EC2 容器服务**（**ECS**）。由于空间、时间和精力的限制，我没有提供所有这些的示例，但你可以查阅该模块极为全面的文档，网址如下：

[`forge.puppet.com/puppetlabs/aws`](https://forge.puppet.com/puppetlabs/aws)

# 从 Hiera 数据中配置 AWS 资源

直接在代码中管理 AWS 资源没有任何问题，就像我们在前面的示例中做的那样，但我们可以做得稍微更好一点。

在第六章，*使用 Hiera 管理数据*中，我们展示了如何直接从 Hiera 数据创建 Puppet 资源。在那个例子中（*从 Hiera 哈希创建资源*），我们将所有基础设施的用户存储在一个名为`users`的 Hiera 哈希中，然后使用`each`关键字迭代该哈希，为每个用户创建一个用户资源。以下是再次展示的示例代码（`hiera_users2.pp`）：

```
lookup('users2', Hash, 'hash').each | String $username, Hash $attrs | {
  user { $username:
    * => $attrs,
  }
}
```

魔术字符`*`（**属性展开操作符**）告诉 Puppet 将`$attrs`哈希的内容作为资源的属性。

将资源描述为 Hiera 数据的优点在于，当我们需要添加一个新用户或更改现有用户的详细信息时，根本不需要触及 Puppet 代码。一切都在 Hiera 中定义。

## 遍历 Hiera 数据以创建资源

读者可能会问：“我们能不能用这些 AWS 资源做同样的事情？能不能直接在 Hiera 哈希中定义所有内容，然后让 Puppet 迭代它来创建资源？”

实际上，我们可以这么做。用于创建所有这些资源的清单出奇地简洁（`aws_hiera.pp`）：

```
$aws_resources = lookup('aws_resources', Hash, 'hash')
$aws_resources.each | String $r_type, Hash $resources | {
  $resources.each | String $r_title, Hash $attrs | {
    Resource[$r_type] { $r_title:
      * => $attrs,
    }
  }
}
```

要应用清单，请按照以下步骤操作：

1.  编辑 Hiera 数据文件`aws.yaml`，并将第一行中`ami:`设置的值更改为你之前选择的 AMI ID（在我们的示例中是`ami-26d6d131`）：

    ```
    sudo vi /etc/puppetlabs/code/environments/pbg/data/aws.yaml
    ami: 'ami-26d6d131'
    ```

1.  保存文件并运行以下命令：

    ```
    sudo puppet apply --environment pbg /examples/aws_hiera.pp

    ```

如果你已经运行了前面的示例并且 AWS 资源仍然存在，你将不会看到 Puppet 的任何输出，因为这些资源完全相同。

### 注意

记住，如果系统的状态已经与清单中表达的期望状态相同，Puppet 将不会做任何事情。

如果你想证明示例清单确实有效，可以使用 AWS 控制面板删除资源（或通过将 Hiera 数据中的`present`更改为`absent`来使用 Puppet 删除它们），然后重新应用清单。

如果你将该清单与 Hiera 用户示例中的清单进行比较，你会发现它由两个嵌套的循环组成，而不是一个单独的循环。外部循环遍历`$aws_resources`哈希的内容：

```
$aws_resources = lookup('aws_resources', Hash, 'hash')
$aws_resources.each | String $r_type, Hash $resources | {
  ...
  }
}
```

`$aws_resources`哈希的每个键都是一个 Puppet 资源类型的名称。以下是第一个（来自`hiera_aws.yaml`）：

```
 'ec2_vpc':
      ...

```

因此，在第一次循环时，`$r_type`的值将是`ec2_vpc`，而`$resources`的值将是这个哈希：

```
'pbg-vpc':
  ensure: present
  region: "%{lookup('region')}"
  cidr_block: '10.99.0.0/16'
```

现在我们进入内循环，创建所有类型为`$r_type`的资源：

```
$resources.each | String $r_title, Hash $attrs | {
  Resource[$r_type] { $r_title:
    * => $attrs,
  }
}
```

实际情况是，只有一个`ec2_vpc`资源，因此在第一次循环时，`$r_title`的值将是`pbg-vpc`，而`$attrs`的值将是这个哈希：

```
ensure: present
region: "%{lookup('region')}"
cidr_block: '10.99.0.0/16'
```

所以 Puppet 将创建这个资源：

```
ec2_vpc { 'pbg-vpc':
  ensure     => present,
  region     => 'us-east-1',
  cidr_block => '10.99.0.0/16',
}
```

这与前面示例中的`ec2_vpc`资源相同，在我们遍历外部循环时，我们将以相同的方式创建其他资源。

`Resource[$r_type]`是什么？这是一种 Puppet 的技巧。问题在于我们需要声明一个其类型我们还不知道的 Puppet 资源；它将由`$r_type`变量提供。你可能最初会尝试使用类似以下的语法：

```
$r_type = 'ec2_vpc'
$r_type { 'pbg-vpc':
  ...
}
```

不幸的是，Puppet 不允许这种语法，但有一种方法可以绕过这个问题。抽象数据类型`Resource`匹配任何资源类型（你可以在第八章，*类、角色和配置文件*中了解更多关于 Puppet 数据类型的信息）。

我们可以通过在方括号中包含实际的资源类型来使`Resource`更具体：`Resource['ec2_vpc']`。这是声明资源的有效语法。

所以这就是我们声明一个其类型来自变量的资源的方法：

```
$r_type = 'ec2_vpc'
Resource[$r_type] { 'pbg-vpc':
  ...
}
```

现在，由 Hiera 数据描述的 AWS 资源应该更容易维护和扩展，当你在生产中使用 Puppet 时。

## 清理未使用的资源

为了关闭你的 EC2 实例，从而避免使用你的免费小时数或被收费，编辑你的 Hiera 数据，将`ec2_instance`资源的`ensure`设置为`absent`：

```
'ec2_instance':
  'pbg-vpc-demo':
    ensure: absent
    region: "%{lookup('region')}"
    subnet: 'pbg-vpc-subnet'
    security_groups: 'pbg-vpc-sg'
    image_id: "%{lookup('ami')}"
    instance_type: 't1.micro'
    associate_public_ip_address: true
    key_name: 'pbg'
```

当你重新应用清单时，Puppet 将停止该实例。你可以保持其他资源不变，因为它们不会产生费用。

# 总结

在本章中，我们介绍了云计算的基本概念，并考察了管理云资源的一些选项，包括 CloudFormation 和 Terraform，然后介绍了`puppetlabs/aws`模块。

我们已经完成了创建 AWS 账户、设置 IAM 用户和策略、生成凭证和 SSH 密钥、安装 AWS SDK gem 以及选择合适的 AMI（Amazon 机器映像）的过程。

使用 Puppet，我们创建了一个 EC2 实例和安全组，并且看到了如何使用 SSH 连接到运行中的实例。更进一步，我们从头开始创建了整个 VPC，包括子网、互联网网关、路由表、安全组和 EC2 实例。

最后，我们已经看到如何直接从 Hiera 数据构建所有这些云资源，这种方式是描述 Puppet 资源最灵活和强大的方法。

在接下来的最后一章，我们将汇集本书前面各章的思路和技巧，创建一个完整、可操作的 Puppet 基础设施示例，您可以将其作为自己项目的基础。

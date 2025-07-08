# 第二章 使用 Terraform 提供 IaaS

在本章中，我们将覆盖以下教程：

+   配置 Terraform AWS 提供商

+   创建和使用一个 SSH 密钥对用于 AWS

+   使用 Terraform 管理 AWS 安全组

+   使用 Terraform 创建一个 Ubuntu EC2 实例

+   使用 Terraform 生成有意义的输出

+   使用 Terraform 的上下文默认值

+   使用 Terraform 管理 S3 存储

+   使用 Terraform 创建私有 Docker 仓库

+   使用 Terraform 创建一个 PostgreSQL RDS 数据库

+   使用 Terraform 启用 Docker 的 CloudWatch 日志

+   使用 Terraform 管理 IAM 用户

# 介绍

现代基础设施通常使用多个提供商（**Amazon Web Services**（**AWS**）、OpenStack、Google Cloud、Digital Ocean 等），并结合多个外部服务（DNS、邮件、监控等）。许多提供商提出了他们自己的自动化工具，但 Terraform 的优势在于它允许你从一个地方管理所有内容，全部使用代码。使用它，你可以根据环境动态创建两个 IaaS 提供商上的机器，在另一个 DNS 提供商处注册它们的名称，在第三方监控公司启用监控，同时配置公司 GitHub 帐户并将应用日志发送到适当的服务。此外，它还可以将配置委托给那些擅长此事的工具（如 Chef、Puppet 等），一切都能用同一个工具完成。你的基础设施状态被描述、存储、版本化并共享。

在本章中，我们将学习如何使用 Terraform 在 AWS 上引导一个完整的基础设施。你将学习如何从启动精细调优的 EC2 实例和在不同区域动态优化 RDS 数据库，到创建紧密的安全组、部署 SSH 密钥对和保护 IAM 访问密钥、启用 CloudWatch 日志存储、生成有用的输出、处理无限的**简单存储服务**（**S3**）存储以及使用私有 Docker 仓库。

### 注意

本书使用的 Terraform 版本是 0.7.2。

# 配置 Terraform AWS 提供商

我们可以与许多 IaaS 提供商一起使用 Terraform，例如 Google Cloud 或 Digital Ocean。在这里，我们将配置 Terraform 以便与 AWS 一起使用，并在本章的其余部分继续使用该提供商。

为了让 Terraform 与 IaaS 交互，它需要配置一个*提供商*。

## 准备工作

要执行这个教程，你需要以下内容：

+   一个具有密钥的 AWS 账户

+   一个工作中的 Terraform 安装

+   一个空目录来存储你的基础设施代码

+   一个互联网连接

## 如何操作…

为了在 Terraform 中配置 AWS 提供商，我们需要以下三个文件：

+   一个声明我们变量的文件，一个可选的描述，以及每个变量的可选默认值（`variables.tf`）

+   一个为整个项目设置变量的文件（`terraform.tfvars`）

+   一个提供商文件（`provider.tf`）

让我们在 `variables.tf` 文件中声明我们的变量。我们可以从声明通常称为 `AWS_DEFAULT_REGION`、`AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY` 的环境变量开始：

```
variable "aws_access_key" {
  description = "AWS Access Key"
}

variable "aws_secret_key" {
  description = "AWS Secret Key"
}

variable "aws_region" {
  default     = "eu-west-1"
  description = "AWS Region"
}
```

在 `terraform.tfvars` 文件中设置与 AWS 账户匹配的两个变量。不建议将此文件提交到源代码管理中：最好使用示例文件（即：`terraform.tfvars.example`）。同时，建议为 AWS 使用专门的 Terraform 用户，而不是 root 账户的密钥：

```
aws_access_key = "< your AWS_ACCESS_KEY >"
aws_secret_key = "< your AWS_SECRET_KEY >"
```

现在，让我们把所有这些整合到一个文件中，即 `provider.tf`：

```
provider "aws" {
  access_key = "${var.aws_access_key}"
  secret_key = "${var.aws_secret_key}"
  region     = "${var.aws_region}"
}
```

应用以下 Terraform 代码：

```
$ terraform apply

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

```

这仅意味着代码有效，而不意味着它能真正与 AWS 进行身份验证（尝试使用一对无效的密钥）。为此，我们需要在 AWS 上创建一个资源。

### 注意

现在你有了一个名为 `terraform.tfstate` 的新文件，它已经在你的仓库根目录下创建。这个文件非常关键：它是你基础设施的存储状态。不要犹豫去查看它，它是一个文本文件。

## 它是如何工作的…

这是第一次接触**HashiCorp 配置语言**（**HCL**），这是 Terraform 以及其他 HashiCorp 产品使用的语言，看起来相当熟悉：它是与 JSON 完全兼容的结构化语言。我们可以在这里找到关于 HCL 的更多信息：[`github.com/hashicorp/hcl`](https://github.com/hashicorp/hcl)。在这个例子中，我们声明了变量，并为参考提供了可选的描述。我们也可以简单地用以下方式声明它们：

```
variable "aws_access_key" { }
```

所有变量都参考以下结构：

```
${var.variable_name}
```

如果变量已经声明了默认值，就像我们的 `aws_region` 声明了 `eu-west-1` 的默认值；如果在 `terraform.tfvars` 文件中没有覆盖该值，它将会使用这个默认值。

如果我们没有为变量提供安全的默认值，会发生什么情况？ Terraform 在执行时会要求我们输入一个值：

```
$ terraform apply
var.aws_region
 AWS Region

 Enter a value:

```

## 还有更多内容…

我们直接在 Terraform 代码中使用了值来配置我们的 AWS 凭证。如果你已经在命令行中使用 AWS，很可能你已经有一组标准的环境变量：

```
$ echo ${AWS_ACCESS_KEY_ID}
<your AWS_ACCESS_KEY_ID>
$ echo ${AWS_SECRET_ACCESS_KEY}
<your AWS_SECRET_ACCESS_KEY>
$ echo ${AWS_DEFAULT_REGION}
eu-west-1

```

如果没有，你可以按以下方式简单设置它们：

```
$ export AWS_ACCESS_KEY_ID="123"
$ export AWS_SECRET_ACCESS_KEY="456"
$ export AWS_DEFAULT_REGION="eu-west-1"

```

然后 Terraform 可以直接使用它们，你唯一需要输入的代码就是声明你的提供者！当使用不同工具时，这非常方便。

`provider.tf` 文件看起来就会像这样简单：

```
provider "aws" { }
```

# 创建并使用一对 SSH 密钥以在 AWS 上使用

现在我们在 Terraform 中配置了 AWS 提供者，接下来让我们为即将启动的虚拟机的默认账户添加一对 SSH 密钥。

## 准备就绪

要跟随这个步骤，你将需要以下内容：

+   一个可工作的 Terraform 安装

+   已在 Terraform 中配置的 AWS 提供者

+   在你能记住的地方生成一对 SSH 密钥，例如，在你的仓库根目录下的 `keys` 文件夹中：

    ```
    $ mkdir keys
    $ ssh-keygen -q -f keys/aws_terraform -C aws_terraform_ssh_key -N ''

    ```

+   需要一个互联网连接

## 如何操作…

我们想要的资源名为 `aws_key_pair`。让我们在 `keys.tf` 文件中使用它，并粘贴公钥内容：

```
resource "aws_key_pair" "admin_key" {
  key_name   = "admin_key"
  public_key = "ssh-rsa AAAAB3[…]"
}
```

这将简单地将你的公钥上传到你的 AWS 账户，名为 `admin_key`：

```
$ terraform apply
aws_key_pair.admin_key: Creating...
 fingerprint: "" => "<computed>"
 key_name:    "" => "admin_key"
 public_key:  "" => "ssh-rsa AAAAB3[…]"
aws_key_pair.admin_key: Creation complete

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

```

如果你手动访问你的 AWS 账户，在 **EC2** | **网络与安全** | **密钥对** 中，你现在应该能找到你的密钥：

![如何操作……](img/B05671_02_01.jpg)

另一种使用我们的密钥与 Terraform 和 AWS 的方法是直接从文件中读取，这将展示如何在 Terraform 中使用文件插值。

为了做到这一点，让我们在 `variables.tf` 中声明一个新的空变量来存储我们的公钥：

```
variable "aws_ssh_admin_key_file" { }
```

在 `terraform.tfvars` 中初始化变量，指定密钥的路径：

```
aws_ssh_admin_key_file = "keys/aws_terraform"
```

现在让我们用 `file()` 插值替换之前的 `keys.tf` 代码：

```
resource "aws_key_pair" "admin_key" {
  key_name   = "admin_key"
  public_key = "${file("${var.aws_ssh_admin_key_file}.pub")}"
}
```

这是一种更清晰、更简洁的方式，来访问 Terraform 资源中的公钥内容。它也更易于维护，因为更换密钥时只需替换文件，无需其他操作。

## 如何运作……

我们的第一个资源 `aws_key_pair` 接受两个参数（一个密钥名称和公钥内容）。这就是 Terraform 中所有资源的工作方式。

我们使用了第一个 *文件* 插值，使用变量，展示了如何使用更动态的代码来管理我们的基础设施。

## 还有更多……

使用 Ansible，我们可以创建一个 *角色* 来做同样的工作。下面是如何使用变量来管理 EC2 密钥对，名称为 `admin_key`。为简化，我们使用了三个常见的环境变量——`AWS_ACCESS_KEY_ID`，`AWS_SECRET_ACCESS_KEY`，和 `AWS_DEFAULT_REGION`：

这里是一个典型的 Ansible 文件层次结构：

```
├── keys
│   ├── aws_terraform
│   └── aws_terraform.pub
├── main.yml
└── roles
 └── ec2_keys
 └── tasks
 └── main.yml

```

在主文件（`main.yml`）中，让我们声明我们的主机（`localhost`）将应用用于管理密钥的角色：

```
---
- hosts: localhost
  roles:
  - ec2_keys
```

在 `ec2_keys` 主任务文件中，创建 EC2 密钥（`roles/ec2_keys/tasks/main.yml`）：

```
---
  - name: ec2 admin key
    ec2_key:
      name: admin_key
      key_material: "{{ item }}"
    with_file: './keys/aws_terraform.pub'
```

使用以下命令执行代码：

```
$ ansible-playbook -i localhost main.yml
TASK [ec2_keys : ec2 admin key] ************************************************
ok: [localhost] => (item=ssh-rsa AAAA[…] aws_terraform_ssh)

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0

```

# 在 Terraform 中使用 AWS 安全组

亚马逊的安全组类似于传统防火墙，适用于 EC2 实例的入站（传入流量）和出站（传出流量）规则。这些规则可以按需更新。我们将创建一个初始的安全组，只允许从我们自己的 IP 地址访问 **安全外壳协议**（**SSH**）流量，同时允许所有的出站流量。

## 正在准备中

为了执行这个配方，你需要以下资源：

+   一个正常工作的 Terraform 安装

+   在 Terraform 中配置的 AWS 提供者（请参阅之前的配方）

+   一个互联网连接

## 如何操作……

我们使用的资源叫做 `aws_security_group`。这是基本的结构：

```
resource "aws_security_group" "base_security_group" {
  name        = "base_security_group"
  description = "Base Security Group"

  ingress { }

  egress { }

}
```

我们知道我们只想允许从我们自己的 IP 地址访问 TCP/22 端口（将 1.2.3.4/32 替换为你的 IP 地址！），并允许所有出站流量。它看起来是这样的：

```
ingress {
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["1.2.3.4/32"]
 }

egress {
  from_port   = 0
  to_port     = 0
  protocol    = "-1"
  cidr_blocks = ["0.0.0.0/0"]
}
```

你可以添加一个名称标签，以便以后更容易参考：

```
tags {
  Name = "base_security_group"
}
```

应用这个，你就可以开始了：

```
$ terraform apply
aws_security_group.base_security_group: Creating...
[…]
aws_security_group.base_security_group: Creation complete

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

```

你可以通过登录到 AWS 控制台并导航到 **EC2 仪表盘** | **网络与安全** | **安全组** 来查看你新创建的安全组：

![如何操作……](img/B05671_02_02.jpg)

另一种访问相同 AWS 控制台信息的方法是通过 AWS 命令行：

```
$ aws ec2 describe-security-groups --group-names base_security_group
{...}

```

## 还有更多……

我们也可以使用 Ansible 来实现相同的结果。以下是使用 Terraform 在本配方中所做操作的等效 Ansible 实现：

```
---
  - name: base security group
    ec2_group:
      name: base_security_group
      description: Base Security Group
      rules:
        - proto: tcp
          from_port: 22
          to_port: 22
          cidr_ip: 1.2.3.4/32
```

# 使用 Terraform 创建一个 Ubuntu EC2 实例

我们之前已经创建了在 AWS EC2 上启动标准虚拟机的必要条件（一个 SSH 密钥对和一个安全组）。现在，让我们在 EC2 上启动这个虚拟机，使用指定的 SSH 密钥对登录并将其放入安全组中，这样（在我们的案例中）SSH 只会从特定的 IP 地址可用。

### 注意

本示例使用的是 AWS 免费套餐中提供的 `t2.micro` 实例。

## 准备工作

要按步骤操作此方法，你需要以下内容：

+   一个正常工作的 Terraform 安装

+   一个 AWS 提供程序，一个 SSH 密钥对，以及在 Terraform 中配置的安全组（参见前面的配方）

+   一个互联网连接

## 如何操作…

首先，你需要找到适合你机器的 AMI。AMI 就像是 AWS 的系统磁盘映像，并通过其 ID 来引用（例如：ami-df3bceb0 或 ami-f2fc9d81）。以 Ubuntu 为例，你可以通过访问其 Amazon EC2 AMI 定位器页面（[`cloud-images.ubuntu.com/locator/ec2/`](https://cloud-images.ubuntu.com/locator/ec2/)）来找到你需要的 AMI。在本例中，我选择了一个 Xenial 版本（16.04 LTS），位于欧盟西部（爱尔兰）的 eu-west-1 区域，使用 HVM 虚拟化，并且由 SSD 磁盘支持。这样我们得到了一个结果——`ami-ee6b189d`：

![如何操作…](img/B05671_02_03.jpg)

从 `variables.tf` 文件中声明此变量开始，该文件在第一个配方中已经创建，使用一个与我们之前找到的 AMI ID 对应的默认值：

```
variable "ami" {
  default = "ami-ee6b189d"
}
```

现在让我们声明实例类型，并将其指定为默认值：

```
variable "aws_instance_type" {
  default = "t2.micro"
}
```

让我们使用这些变量来创建 Terraform `aws_instance` 资源。局部声明的变量可以通过 `${var.variable_name}` 结构来访问，而内部资源属性则通过 `${resource_type.resource_name.attribute}` 结构来访问：

```
resource "aws_instance" "dev" {
  ami                         = "${var.ami}"
  instance_type               = "${var.aws_instance_type}"
  key_name                    = "${aws_key_pair.admin_key.key_name}"
  security_groups             = ["${aws_security_group.base_security_group.name}"]
  associate_public_ip_address = true

  tags {
    Name = "Ubuntu launched by Terraform"
  }
}
```

应用以下代码：

```
$ terraform apply
aws_key_pair.admin_key: Creating...
[…]
aws_security_group.base_security_group: Creating...
[…]
aws_instance.dev: Creating...
[…]
```

导航到 AWS EC2 仪表盘下的 **实例** | **实例**，选择你的实例并记录下公共 IP：

![如何操作…](img/B05671_02_04.jpg)

尝试登录：

```
$ ssh -i keys/aws_terraform ubuntu@52.210.12.27
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-36-generic x86_64)
ubuntu@ip-172-31-18-156:~$
```

你可以通过刷新其状态来应用 Terraform，因为 Terraform 知道远程和本地状态是相同的，因此它不会每次都重新创建新的虚拟机。

你已经成功使用可重复的 Terraform 代码启动了第一个 AWS EC2 实例！

### 扩展实例数量

如果你想启动两个相似的实例，可能是为了调试，或者是为了在负载均衡器后面进行即时操作，应该怎么做？使用 Terraform 很容易，只需在 `aws_instance` 资源中使用 `count` 选项，它会启动所需数量的实例：

```
count = 2
```

接下来，`terraform apply` 这个配置并观察 Terraform 根据计数器自动创建一台新机器：

```
$ terraform apply
aws_key_pair.admin_key: Refreshing state... (ID: admin_key)
aws_security_group.base_security_group: Refreshing state... (ID: sg-d3dbd8b4)
aws_instance.dev.0: Refreshing state... (ID: i-0018b1044953371ae)
aws_instance.dev.1: Creating...
[...]
aws_instance.dev.1: Creation complete

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

```

第二台服务器出现在 AWS 控制台中：

![扩展实例数量](img/B05671_02_05.jpg)

### 注意

记住，销毁 Terraform 基础设施的命令是 `terraform destroy`。

## 还有更多……

我们可以使用 Ansible 达到类似的效果。以下是使用在前面教程中创建的 `admin_key` 和 `base_security_group` 的示例：

```
---
  - name: dev instance
    ec2:
      key_name: admin_key
      group: base_security_group
      instance_type: t2.micro
      image: ami-ee6b189d
      wait: yes
```

# 使用 Terraform 生成有意义的输出

如果 Terraform 在成功运行后能够显示一些有用的信息输出，那该多好？根据我们到目前为止所做的内容，了解如何连接到实例、查看本地和公网 IP 地址，或查看使用的安全组将是非常有帮助的。这正是 Terraform 输出的用途。

## 准备工作

要完成这个教程，你需要以下内容：

+   一个有效的 Terraform 安装

+   一个 AWS 提供者和一个 EC2 实例（使用 SSH 密钥对和安全组），都在 Terraform 中配置（参见前面的教程）

+   一个互联网连接

## 如何实现……

幸运的是，我们可以使用我们已经使用的相同语法来访问变量和引用的属性，这次是在 `output` 资源中。

让我们通过在 `outputs.tf` 中简单地添加一行，来显示如何连接到我们的虚拟机，使用我们 `dev` EC2 实例的 `public_ip` 属性：

```
output "login" {
  value = "ssh ubuntu@${aws_instance.dev.public_ip} -i ${var.aws_ssh_admin_key_file}"
}
```

下次应用 Terraform 时，它将显示以下内容：

```
login = ssh ubuntu@52.51.242.17 -i keys/aws_terraform
```

毋庸置疑，这比登录 AWS 控制台，找到实例并复制粘贴 IP 到终端要快速得多。

如果我们想快速了解我们的 EC2 实例运行在哪些安全组下，怎么办？我们知道安全组可以有多个，所以它是一个数组。我们可以使用 `formatlist` 插值语法来访问这个数组的内容，方法如下：

```
output "security_groups" {
  value = "${formatlist("%v", aws_instance.dev.security_groups)}"
}
```

所以现在，在下次执行 `terraform apply` 时，我们会立即知道我们的安全组：

```
security_groups = [
    base_security_group
]
```

此外，如果我们有很多来自多个来源的信息需要显示，我们也可以使用相同的语法：

```
output "instance_information" {
  value = "${formatlist("instance: %v public: %v private: %v", aws_instance.dev.*.id, aws_instance.dev.*.public_ip, aws_instance.dev.*.private_ip)}"
}
```

这将显示实例 ID 及其本地和公网 IP 地址。

## 还有更多……

注意，我们在第一次输出中使用了`${aws_instance.dev.public_ip}`，而在最后一次输出中使用了`aws_instance.dev.*.public_ip`。如果使用后者，输出将遍历所有可用的机器。如果你使用 `aws_instance` Terraform 资源的 `count=n` 参数启动多个实例，这将非常有用。

# 使用 Terraform 中的上下文默认值

我们已经看到如何在 Terraform 代码中声明和使用默认值，比如为我们区域选择的 Ubuntu AMI 或我们的虚拟机大小。Terraform 中一个有趣的功能是能够声明和使用*映射*值，因此，根据键的不同，变量可以有不同的值。我们将看到如何将它应用到对应 AWS 的正确 AMI 上。

## 准备工作

要完成这个教程，你需要以下内容：

+   一个有效的 Terraform 安装

+   一个 AWS 提供者和一个 EC2 实例（使用 SSH 密钥对和安全组），都在 Terraform 中配置（参见前面的教程）

+   一个互联网连接

## 如何实现……

下面是我们在`variables.tf`文件中为`eu-west-1`区域声明我们希望使用的 AMI：

```
variable "ami" {
  default = "ami-ee6b189d"
}
```

我们可以像这样在`instances.tf`文件中轻松访问它：

```
ami = "${var.ami}"
```

一种相似但更加明确的方法是使用映射，这样我们就能知道值所指向的具体区域：

```
variable "ami" {
  default = {
    eu-west-1 = "ami-ee6b189d"
  }
}
```

下面是我们如何在映射中访问相同的值：

```
ami = "${var.ami["eu-west-1"]}"
```

现在让我们为其他区域添加更多有效的 AMI ID：

```
variable "ami" {
  default = {
    eu-west-1 = "ami-ee6b189d"
    us-east-1 = "ami-4f680658"
    us-west-1 = "ami-68a9e408"
  }
}
```

如果在`instances.tf`文件中正确访问，`ami`变量现在可以适用于这三个区域中的任何一个：

```
ami = "${var.ami["us-east-1"]}"
```

现在是时候直接在代码中管理 AWS 区域，以提高可移植性了。将以下内容添加到`variables.tf`中，以将`eu-west-1`作为默认区域：

```
variable "aws_region" {
  default = "eu-west-1"
}
```

你现在可以在`provider.tf`文件中使用此变量来设置区域：

```
provider "aws" {
  region = "${var.aws_region}"
}
```

现在区域变量在全局范围内可用，让我们在`instances.tf`中使用它来访问我们的映射：

```
ami = "${var.ami["${var.aws_region}"]}"
```

现在我们有了一个易于地理部署的基础设施，团队中的任何人都可以在不修改代码的情况下，将其部署到离他们最近的位置。

## 还有更多…

我们可以使用 Terraform 中的`lookup()`函数对映射进行动态访问：

```
ami = "${lookup(var.ami, var.aws_region)}"
```

# 使用 Terraform 管理 S3 存储

轻松且可扩展地存储和访问文件是现代基础设施的一个重要部分。Amazon S3 是 Amazon 对此需求的回应。S3 将“对象”存储在“桶”中，并且没有存储限制（唯一的例外是桶名称：它必须在 Amazon S3 上唯一，因为命名空间是共享的）。我们将看看如何利用 Terraform 充分使用 S3。

## 准备工作

要执行此步骤，你需要以下内容：

+   一个正常工作的 Terraform 安装

+   一个在 Terraform 中配置的 AWS 提供商（请参考之前的配方）

+   一个互联网连接

## 如何操作…

我们将从在 S3 上创建一个简单且明确公开的桶 `iac-book` 开始，使用 `aws_s3_bucket` 资源（并附带一个标签以示说明）：

```
resource "aws_s3_bucket" "iac_book" {
  bucket = "iac-book"
  acl    = "public-read"

  tags {
    Name = "IAC Book Bucket in ${var.aws_region}"
  }
}
```

在执行`terraform apply`之后，你的桶立即可用于存储对象。你可以在 AWS S3 控制台上看到它（[`console.aws.amazon.com/s3/`](https://console.aws.amazon.com/s3/)）：

![如何操作…](img/B05671_02_06.jpg)

现在让我们存储第一个对象，一个非常简单的文件，包含一个简单的字符串（`"Hello Infrastructure-as-Code Cookbook!"`）。该资源名为 `aws_s3_bucket_object`，你需要引用之前创建的桶、目标名称（`index.html`）及其内容。ACL 在这里再次明确为公开：

```
resource "aws_s3_bucket_object" "index" {
  bucket = "${aws_s3_bucket.iac_book.bucket}"
  key = "index.html"
  content = "<h1>Hello Infrastructure-as-Code Cookbook!</h1>"
  content_type = "text/html"
  acl    = "public-read"
}
```

你也可以直接提供一个文件，而不是其内容：

```
source = "index.html"
```

如果你访问 AWS S3 控制台，你可以看到它有一些扩展的信息：

![如何操作…](img/B05671_02_07.jpg)

如果我们能直接从 Terraform 获取到文件的 URL 并提供给其他人，那就太棒了。不幸的是，目前没有简单的函数来做到这一点。然而，我们知道 URL 是如何构建的：`http://s3-<region>.amazonaws.com/bucket_name/object_name`。让我们创建一个输出，包含这些信息：

```
output "S3" {
  value = "http://s3-${aws_s3_bucket.iac_book.region}.amazonaws.com/${aws_s3_bucket.iac _book.id}/${aws_s3_bucket_object.index.key}"
}
```

将链接粘贴到网页浏览器中，你将能够访问你的文件。

一种解决方法是通过简单地在你的 `aws_s3_bucket` 资源中添加以下内容，使用 S3 的 *静态网站托管* 功能：

```
website {
  index_document = "index.html"
}
```

一个可选输出将为你提供其静态托管 URL（在我们的案例中，是 [iac-book.s3-website-eu-west-1.amazonaws.com](http://iac-book.s3-website-eu-west-1.amazonaws.com) 而不是 [`s3-eu-west-1.amazonaws.com/iac-book/index.html`](http://s3-eu-west-1.amazonaws.com/iac-book/index.html)）：

```
output "S3 Endpoint" {
  value = "${aws_s3_bucket.iac_book.website_endpoint}"
}
```

## 还有更多……

使用 Ansible，有很多方法可以创建一个存储桶。以下是一个简单的存储桶，具有公共读取权限，使用经典的`s3`模块：

```
---
- name: create iac-book bucket
  s3:
    bucket: iac-book
    mode: create
    permission: public-read
```

### 注意

请注意，Ansible 2.2 还带有一个`s3_website`模块，专门用于处理 S3 网站。

下面是我们如何使用相同的 `s3` 模块简单上传我们之前的 `index.html` 文件：

```
- name: create index.html file
  s3:
    bucket: iac-book
    object: index.html
    src: index.html
    mode: put
    permission: public-read
```

# 使用 Terraform 创建私有 Docker 仓库

要托管 Docker 镜像，你需要一个被称为 **注册表** 的东西。这个注册表可以是你自己运行的，也可以是作为服务提供的。它为你存储镜像，有时也会构建它们。Docker Hub 和 CoreOS 的 Quay.io 是你可以订阅的主要 Docker 管理注册表。它们在功能或定价上都很有吸引力。然而，一个有趣的替代方案是 AWS **弹性容器注册表** (**ECR**)：定价不同，并且完全集成在 AWS 生态系统中。让我们通过 Terraform 简单地创建无数个仓库！

## 做好准备

要执行此操作，你将需要以下内容：

+   一个正在运行的 Terraform 安装

+   已在 Terraform 中配置的 AWS 提供程序（参考前面的食谱）

+   已配置的 AWS CLI ([`docs.aws.amazon.com/cli/latest/userguide/installing.html`](http://docs.aws.amazon.com/cli/latest/userguide/installing.html))

+   一个互联网连接

## 如何操作……

假设你想将你的应用容器存储在一个名为`myapp`的仓库中，以便轻松部署。使用 Terraform 非常简单。将以下代码添加到一个名为`ecr.tf`的文件中：

```
resource "aws_ecr_repository" "myapp" {
  name = "myapp"
}
```

如果你想知道访问新仓库的 URL，可以使用相应的导出属性创建输出：

```
output "ECR" {
  value = "${aws_ecr_repository.myapp.repository_url}"
}
```

如果你习惯了其他 Docker 注册表，第一步是进行身份验证，以便创建私有仓库。这里，AWS 不提供登录或密码。我们需要使用官方的 AWS 命令行进行身份验证，这将为我们提供临时的 Docker 凭证。此命令的输出是需要输入的 Docker 命令：

```
$ aws ecr get-login --region eu-west-1
docker login -u AWS -p AQECAHh... -e none https://<account_number>.dkr.ecr.eu-west-1.amazonaws.com

```

现在我们可以随意进行 `docker build`、`tag` 和 `push` 镜像！（关于使用 Docker 镜像的更多内容，请参阅本书的专门章节。）

一个很好的高级功能是能够为每个创建的仓库使用精细的策略。

# 使用 Terraform 创建 PostgreSQL RDS 数据库

**Amazon 关系型数据库服务** (**RDS**) 是一种按需、可随时使用并可调整大小的 EC2 实例，专门定制和配置来运行所请求的数据库服务器。你可以在 RDS 上启动多种不同的关系型数据库服务器，我们在这个例子中将重点使用 PostgreSQL。

## 准备就绪

要完成此教程，你需要以下内容：

+   一个有效的 Terraform 安装

+   在 Terraform 中配置一个 AWS 提供程序（参考之前的教程）

+   一个互联网连接

## 如何操作…

在数据库部署中有许多参数需要考虑，即使是一个简单的部署。为了确保我们将部署的内容，我们将从填写一个简单的表格开始，列出数据库需求，并在此基础上进行构建：

| 参数 | 变量名 | 值 |
| --- | --- | --- |
| RDS 数据库引擎 | `rds_engine` | `postgresql` |
| RDS 数据库引擎版本 | `rds_engine_version` | `9.5.2` |
| RDS 实例名称 | `rds_identifier` | `db` |
| RDS 实例类型 | `rds_instance_type` | `db.t2.micro` |
| RDS 存储大小（GB） | `rds_storage_size` | `5` |
| RDS 第一个数据库名称 | `rds_db_name` | `iac_book_db` |
| RDS 管理员用户名 | `rds_admin_user` | `dbadmin` |
| RDS 管理员密码 | `rds_admin_password` | `super_secret_password` |
| RDS 是否公开可访问 | `rds_publicly_accessible` | `true` |

让我们在 `variables.tf` 文件中设置所有这些变量：

```
variable "rds_identifier" {
  default = "db"
}

variable "rds_instance_type" {
  default = "db.t2.micro"
}
variable "rds_storage_size" {
  default = "5"
}

variable "rds_engine" {
  default = "postgres"
}

variable "rds_engine_version" {
  default = "9.5.2"
}

variable "rds_db_name" {
  default = "iac_book_db"
}

variable "rds_admin_user" {
  default = "dbadmin"
}

variable "rds_admin_password" {
  default = "super_secret_password"
}

variable "rds_publicly_accessible" {
  default = "true"
}
```

由于我们运行的是 PostgreSQL 并且希望它能够通过互联网访问（虽然一般不推荐用于生产环境），我们需要一个安全组，允许仅允许我们的 IP 地址通过默认的 PgSQL 端口（TCP/5432）访问（参考 *使用 AWS 安全组与 Terraform* 教程），在 `securitygroups.tf` 文件中：

```
resource "aws_security_group" "rds_security_group" {
  name        = "rds_security_group"
  description = "RDS Security Group"

  ingress {
    from_port   = 5432
    to_port     = 5432
    protocol    = "tcp"
    cidr_blocks = ["1.2.3.4/32"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags {
    Name = "rds_security_group"
  }
}
```

现在我们已经具备了构建 `aws_db_instance` 资源所需的一切：

```
resource "aws_db_instance" "db" {
  engine            = "${var.rds_engine}"
  engine_version    = "${var.rds_engine_version}"
  identifier        = "${var.rds_identifier}"
  instance_class    = "${var.rds_instance_type}"
  allocated_storage = "${var.rds_storage_size}"
  name              = "${var.rds_db_name}"
  username          = "${var.rds_admin_user}"
  password          = "${var.rds_admin_password}"
  publicly_accessible    = "${var.rds_publicly_accessible}"
  vpc_security_group_ids = ["${aws_security_group.rds_security_group.id}"]
  tags {
    Name = "IAC Database in ${var.aws_region}"
  }
}
```

正如我们之前所做的，快速输出我们的新数据库的 FQDN 将帮助我们快速使用它，输出内容放在 `outputs.tf` 中：

```
output "RDS" {
  value = "address: ${aws_db_instance.db.address}"
}
```

现在让我们运行 `terraform apply` 并查看结果：

```
# psql -h <your_db_address> -d iac_book_db -U dbadmin
Password for user dbadmin:
psql (9.5.4, server 9.5.2)
[...]

iac_book_db=> \l
 List of databases
 Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-------------+----------+----------+-------------+-------------+-----------------------
 iac_book_db | dbadmin  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres    | dbadmin  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 rdsadmin    | rdsadmin | UTF8     | en_US.UTF-8 | en_US.UTF-8 | rdsadmin=CTc/rdsadmin
 template0   | rdsadmin | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/rdsadmin          +
 |          |          |             |             | rdsadmin=CTc/rdsadmin
 template1   | dbadmin  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/dbadmin           +
 |          |          |             |             | dbadmin=CTc/dbadmin
(5 rows)

```

还有更多你可以使用或设置的有用选项，如维护窗口、备份保留期限、专用数据库子网、存储加密和主从配置等。

## 还有更多…

当使用 Ansible 做类似的工作，且值相同的时候，这该如何运作？和往常一样简单：

```
---
- name: create RDS PgSQL
  rds:
    command: create
    instance_name: db
    db_engine: postgres
    engine_version: 9.5.2
    db_name: iac_book_db
    size: 5
    instance_type: db.t2.micro
    username: dbadmin
    password: super_secure_password
    publicly_accessible: yes
    tags:
      Name: IAC Database
```

执行完这个 playbook 后，类似的 PostgreSQL 服务器将运行在 RDS 上，就像我们之前使用 Terraform 所做的那样。

# 使用 Terraform 为 Docker 启用 CloudWatch 日志

**CloudWatch Logs** 是 Amazon 提供的一项日志聚合服务，你可以使用它将日志发送到 CloudWatch。它非常有用，可以将一些日志集中存储，分享访问权限，收到错误发生时的警报，或者仅仅是安全地存储它们。我们将学习如何创建 CloudWatch 日志组，并使用它来流式传输来自 Docker 容器中的日志。

## 准备就绪

要完成此教程，你需要以下内容：

+   一个有效的 Terraform 安装

+   在 Terraform 中配置一个 AWS 提供程序（参考之前的教程）

+   一个互联网连接

+   一个在 Linux 上运行的 Docker 引擎，用于可选的使用演示

## 如何操作...

假设我们希望日志组命名为 `docker_logs`，并且希望将这些日志保存七天。在 `variables.tf` 文件中，这看起来是这样的：

```
variable "log_group_name" {
  default = "docker_logs"
}

variable "log_retention_days" {
  default = "7"
}
```

同样，在一个新的 `cloudwatch.tf` 文件中，我们可以使用简单的 `aws_cloudwatch_log_group` 资源：

```
resource "aws_cloudwatch_log_group" "docker_logs" {
  name              = "${var.log_group_name}"
  retention_in_days = "${var.log_retention_days}"
}
```

在 `terraform apply` 后，如果你进入 AWS CloudWatch 页面，你会在左侧的 **日志组** 项下看到新创建的组（[`eu-west-1.console.aws.amazon.com/cloudwatch/`](https://eu-west-1.console.aws.amazon.com/cloudwatch/)）。

![如何操作...](img/B05671_02_08.jpg)

### Amazon CloudWatch Logs Docker 日志驱动程序

现在，你可以使用此组从应用程序或容器创建日志流。按照 AWS 推荐的方式使用它是有详细文档说明的，所以我们来改用 Docker。只需要给 Docker 守护进程访问 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY` 环境变量的权限（配置 Docker 守护进程超出了本章的范围，但对于基于 Red Hat 的系统，如 Fedora 或 CentOS，它位于 `/etc/sysconfig/docker`，对于 Debian/Ubuntu 系统，它位于 `/etc/default/docker`）。重新启动守护进程并开始使用新的 Docker 日志驱动程序记录容器输出，使用 Terraform 中之前指定的日志组名称（`docker_logs`）：

```
$ docker run -it --rm -p 80:80 --log-driver=awslogs --log-opt awslogs-region=eu-west-1 --log-opt awslogs-group=docker_logs --log-opt awslogs-stream=nginx nginx:stable

```

在容器上生成一些活动：

```
$ curl -IL http://localhost
HTTP/1.1 200 OK

```

刷新 AWS CloudWatch 页面，你会看到一个名为 `nginx` 的新条目，里面有容器日志。你可以像这样在你的基础设施中运行所有容器，并轻松获得集中式日志记录！

![Amazon CloudWatch Logs Docker 日志驱动程序](img/B05671_02_09.jpg)

# 使用 Terraform 管理 IAM 用户

使用 AWS 的一个关键部分是控制对资源的访问。我们在之前的所有示例中看到，我们经常需要使用 AWS 访问密钥，显然，使用单个密钥进行所有操作并不是一个好主意。试想，如果你其中一个服务被黑客入侵——入侵者将获得主 AWS 密钥，并且可以代表你执行任何操作。

一个好的安全设置是为团队中的每个人和基础设施中的每个服务分配专用密钥和专用访问权限范围。

幸运的是，**身份与访问管理**（**IAM**）正是为此而存在。我们将看到如何使用它与 Terraform 配合。

## 准备就绪

要执行此示例，你需要以下内容：

+   一个可用的 Terraform 安装

+   在 Terraform 中配置的 AWS 提供程序（参考之前的示例）

+   一个互联网连接

## 如何操作...

让我们从一个简单的案例开始：一个团队的两名成员（Mary 和 Joe）需要访问 AWS 上的资源。他们目前共享相同的主密钥，一旦发生泄漏，将是灾难性的。所以让我们问问他们到底需要在 AWS 空间中访问哪些内容：

| **Mary** | S3 读写权限 |
| --- | --- |
| **Joe** | EC2 只读 |

正如预期的那样，两个用户实际上都不需要完全访问权限！

亚马逊通过提供预构建的 IAM 安全策略来帮助我们。如果这些不够，你还可以定制你需要的策略：

![How to do it…](img/B05671_02_10.jpg)

### 注意

你可以在[`console.aws.amazon.com/iam/home#policies`](https://console.aws.amazon.com/iam/home#policies)找到所有 AWS 托管的 IAM 策略。

### 用于 S3 访问的 IAM 用户

让我们在新的`iam.tf`文件中为 Mary 创建第一个 IAM 用户，使用`aws_iam_user`资源：

```
resource "aws_iam_user" "mary" {
  name = "mary"
  path = "/team/"
}
```

`path`是纯粹可选的，仅供参考，我只是建议使用结构化的路径。所以以后我们还会有`/apps/`路径。

现在，我们可以为 Mary 的用户创建一个 AWS 访问密钥，使用`aws_iam_access_key`资源并引用我们的用户：

```
resource "aws_iam_access_key" "mary" {
  user = "${aws_iam_user.mary.name}"
}
```

最后，正如我们所知，我们希望将`AmazonS3FullAccess`托管策略附加到这个用户上，我们使用专用资源：

```
resource "aws_iam_user_policy_attachment" "mary_s3full" {
  user = "${aws_iam_user.mary.name}"
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3FullAccess"
}
```

让我们在`outputs.tf`中写一个`output`，这样我们就知道密钥的两部分：

```
output "mary" {
  value = "ACCESS_KEY: ${aws_iam_access_key.mary.id}, SECRET: ${aws_iam_access_key.mary.secret}"
}
```

此外，`terraform apply`此操作以创建`mary`用户：

```
[...]
Outputs:
mary = ACCESS_KEY: AKIAJPQB7HBK2KLAARRQ, SECRET: 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
```

### 测试限制

现在，`terraform apply`这个，然后通过 S3 浏览器确认你可以访问 S3！下面是使用`s3cmd`创建简单 S3 桶的示例：

```
$ s3cmd --access_key=<mary_access_key> --secret_key=<mary_secret_key> mb s3://iacbook-iam-bucket
Bucket 's3://iacbook-iam-bucket/' created

```

这个账户真的仅限于 S3 吗？就像它所声称的那样吗？让我们尝试使用 Mary 的账户通过`aws`命令行列出 EC2 主机（前提是你已按要求配置了`aws`工具）：

```
$ aws --profile iacbook-mary ec2 describe-hosts
An error occurred (UnauthorizedOperation) when calling the DescribeHosts operation: You are not authorized to perform this operation.

```

所以一切看起来都很好且安全！Mary 可以安全地在 S3 上完成她的工作。

### 用于 EC2 只读访问的 IAM 用户

有没有类似的托管策略可以为 Joe 提供只读 EC2 访问权限？幸运的是，有！它被创意地命名为`AmazonEC2ReadOnlyAccess`。

让我们创建第二个用户，使用`iam.tf`文件中的这个 IAM 策略：

```
resource "aws_iam_user" "joe" {
  name = "joe"
  path = "/team/"
}

resource "aws_iam_access_key" "joe" {
  user = "${aws_iam_user.joe.name}"
}

resource "aws_iam_user_policy_attachment" "joe_ec2ro" {
  user = "${aws_iam_user.joe.name}"
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess"
}
```

别忘了附带的有用输出：

```
output "joe" {
  value = "ACCESS_KEY: ${aws_iam_access_key.joe.id}, SECRET: ${aws_iam_access_key.joe.secret}"
}
```

接下来，再次`terraform apply`这个，那么 Joe 用户能看到 S3 上的内容吗？不，他不能：

```
$ s3cmd --access_key=<joe_access_key> --secret_key=<joe_secret_key> ls
ERROR: S3 error: 403 (AccessDenied): Access Denied

```

但是，Joe 用户能否像他需要的那样列出 EC2 虚拟机，使用 Mary 被禁止使用的同一个命令？是的，他可以：

```
$ aws --profile iacbook-joe ec2 describe-hosts
{
  "Hosts": []
}
```

我们正在通过代码安全地管理我们的基础设施访问！

### 应用程序用户 IAM – CloudWatch Logs

我们在之前的配方中使用了 CloudWatch 日志服务。如果你还记得，你需要在 Docker 引擎配置中再次输入你的密钥。如果你有 100 台服务器，你的主密钥将会出现在每一台服务器上。如果考虑到 Docker 中这个配置的范围只是发送日志，这就显得有些不必要了。幸运的是，有一个名为`CloudWatchLogsFullAccess`的托管 IAM 策略可以解决这个问题。

所以我们再创建一个用户，和之前为 Mary 和 Joe 创建的完全一样，只不过这个是为我们的 Docker 引擎创建的，而不是为一个真实用户创建的，位于`iam.tf`中。我建议使用不同的路径，只是为了区分真实用户和应用程序用户。然而，这完全是可选的，取决于个人偏好：

```
resource "aws_iam_user" "logs" {
  name = "logs"
  path = "/apps/"
}

resource "aws_iam_access_key" "logs" {
  user = "${aws_iam_user.logs.name}"
}

resource "aws_iam_user_policy_attachment" "logs_cloudwatch_full" {
  user = "${aws_iam_user.logs.name}"
  policy_arn = "arn:aws:iam::aws:policy/CloudWatchLogsFullAccess"
}
```

在`outputs.tf`中相关的`output`如下：

```
output "logs" {
  value = "ACCESS_KEY: ${aws_iam_access_key.logs.id}, SECRET: ${aws_iam_access_key.logs.secret}"
}
```

现在，使用`terraform apply`并再次尝试使用这些凭据而不是主密钥来执行*使用 Terraform 启用 CloudWatch 日志记录*配方：它仍然可以在 CloudWatch 范围内正常工作，但如果出现问题，它将永远不会让你的其余基础设施处于危险之中。在这一领域，最糟糕的情况就是日志的完全浪费。

```
[...]
Outputs:

joe = ACCESS_KEY: AKIAJQPSXBKSD3DY47BQ, SECRET: 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
logs = ACCESS_KEY: AKIAISIUXTG5RIJZAEYA, SECRET: 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
mary = ACCESS_KEY: AKIAJPQB7HBK2KLAARRQ, SECRET: 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
```

![应用程序用户 IAM – CloudWatch 日志](img/B05671_02_11.jpg)

## 还有更多…

如果你更愿意看到使用 Ansible 的实现方式，它稍微有些不同。IAM 的支持并不完全相同，因为没有 IAM 托管策略的支持。然而，你可以像这样简单地创建用户：

```
---
- name: create mary user
  iam:
    iam_type: user
    name: mary
    state: present
    access_key_state: create
    path: /team/
```

由于当前没有 IAM 托管策略的支持，一个解决方法是使用我们想要的 IAM 策略的 JSON 文件，例如给我们的用户 Mary 使用`AmazonS3FullAccess`。在 AWS 控制台的**策略**部分（[`console.aws.amazon.com/iam/home#policies`](https://console.aws.amazon.com/iam/home#policies)）可以很容易找到。将以下 JSON 内容粘贴到`Ansible`文件夹根目录下的`AmazonS3FullAccess.json`文件中：

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

在`iam_policy`模块中使用此本地策略：

```
- name: Assign a AmazonS3FullAccess policy to mary
  iam_policy:
    iam_type: user
    iam_name: mary
    policy_name: AmazonS3FullAccess
    state: present
    policy_document: AmazonS3FullAccess.json
```

# 3

# 构建你的第一个 Terraform 项目

如果你是 Terraform 新手并希望开始你的第一个项目，那么这一章适合你。在本章中，我们将介绍如何为 AWS 构建你的第一个 Terraform 配置。我们将从讨论如何安装 Terraform 并准备它与 AWS 一起使用开始。然后，我们将引导你完成构建第一个 Terraform 配置和模板的过程。最后，我们还会展示如何配置和测试你的第一个模板，这样你就可以看到你的基础设施开始生效。通过本章的学习，你将掌握使用 Terraform 构建基础设施所需的基础知识和技能。

在本章中，我们将涵盖以下主题：

+   如何安装 Terraform

+   如何安装/准备 Terraform 以便与 AWS 配合使用

+   构建你的第一个 Terraform 配置

+   构建你的第一个 Terraform 模板

+   配置和测试你的模板

# 如何安装 Terraform

在开始使用 Terraform 之前，了解如何正确安装和管理 Terraform 安装非常重要。安装 Terraform 可能有一定挑战，但有许多在线资源可以帮助你完成安装过程，包括官方的 Terraform 文档。Terraform 由 HashiCorp 作为二进制包发布，也可以通过流行的包管理器进行安装。安装 Terraform 是开始使用 Terraform 创建第一个项目的第一步。

接下来我们来看看不同的安装方法。

## 手动安装

对于手动安装，你可以选择从 Terraform **下载** 页面下载预编译的二进制文件，或者从源代码编译二进制文件。

### 预编译二进制文件

要安装 Terraform，你需要下载适合你操作系统的正确包，这些包以 ZIP 压缩包的形式提供。你可以在 Terraform 网站上选择你的操作系统，找到适合的安装包：[`www.terraform.io/downloads`](https://www.terraform.io/downloads)。

一旦你下载了适合你系统的 Terraform 包，下一步是解压这个包。在包内，你会找到一个名为 `terraform` 的二进制文件，它是 Terraform 的主要可执行文件。你可以安全地删除包内的其他文件，Terraform 仍然可以正常工作。

为了使用 Terraform，你还需要确保将 `terraform` 二进制文件添加到系统的 `PATH` 环境变量中，这样你就可以从终端的任何目录执行它。设置此环境变量的过程因操作系统不同而有所不同，但通常可以在网上或 Terraform 文档中找到相应的说明。

#### Mac 或 Linux 的 PATH 配置

打印出 `PATH` 中位置的冒号分隔列表：

```
echo $PATH
```

将 Terraform 二进制文件移动到列出的某个位置。这个命令假设二进制文件目前在你的下载文件夹中，并且你的 `PATH` 包含 `/usr/local/bin`，如果位置不同，你可以根据实际情况进行调整：

```
mv ~/Downloads/terraform /usr/local/bin/
```

#### Windows PATH 配置

`PATH` 配置存储在注册表中，你可以通过以下界面进行编辑：

1.  转到**控制面板** | **系统** | **系统设置** | **环境变量**。

1.  向下滚动系统变量，直到找到 `PATH`。

    点击**编辑**并根据需要进行更改。

    确保在前一个变量的末尾添加分号，因为这些分号用作分隔符。

1.  启动一个新的控制台以使设置生效。

### 从源代码编译

如果你希望从源代码编译 Terraform 二进制文件，你可以克隆 HashiCorp 的 Terraform 仓库：

```
git clone https://github.com/hashicorp/terraform.git
```

导航到新的目录：

```
cd terraform
```

然后，编译二进制文件。此命令将编译二进制文件并将其存储在 `$GOPATH/bin/terraform` 目录下：

```
go install
```

安装 Terraform 后，确保将 `terraform` 二进制文件添加到系统的 `PATH` 环境变量中，以便从任何地方执行它。这个过程可能会根据操作系统的不同有所不同，但通常包括使用命令或手动编辑系统文件将 `terraform` 二进制文件的位置添加到 `PATH` 变量中。

## 流行的包管理器

有几个流行的包管理器可以简化在不同操作系统上安装 Terraform 的过程。这些包管理器允许你通过一个**命令行界面**（**CLI**）管理和更新多个软件包。在本节中，我们将介绍一些最流行的 Terraform 包管理器，包括 Windows 上的 Chocolatey、macOS 上的 Homebrew，以及 Linux 上的 APT 和 Yum。

### macOS 上的 Homebrew

要在 Mac macOS 上安装 Terraform，你可以使用被称为 Homebrew 的免费开源包管理系统。在安装 Terraform 之前，你需要先安装 HashiCorp `tap`，它是一个包含所有 Homebrew 包的仓库：

```
brew tap hashicorp/tap
```

现在，通过 `hashicorp/tap/terraform` 安装 Terraform：

```
brew install hashicorp/tap/terraform
```

注意

这将安装一个签名的二进制文件，并且会随着每次官方新版本发布而自动更新。

要更新到最新版本的 Terraform，首先更新 Homebrew：

```
brew update
```

然后，运行 `upgrade` 命令以下载并使用最新的 Terraform 版本：

```
brew upgrade hashicorp/tap/terraform
```

### Windows 上的 Chocolatey

要在 Windows 上使用 Chocolatey 安装 Terraform，你可以使用命令行界面（CLI）。Chocolatey 是一个免费开源的 Windows 包管理器，它简化了在系统上安装和管理软件的过程：

```
choco install terraform
```

注意

Chocolatey 和 Terraform 包*不是*由 HashiCorp 直接维护的。Terraform 的最新版本始终可以进行手动安装。

### Linux

HashiCorp 官方维护并签署了适用于 Ubuntu/Debian、CentOS/RHEL、Fedora 和 Amazon Linux 发行版的包。

#### Ubuntu/Debian

确保你的系统是最新的，并且已经安装了 `gnupg`、`software-properties-common` 和 `curl` 包。这些包是验证 HashiCorp 的 GPG 签名并安装其 Debian 包仓库所必需的：

```
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```

安装 HashiCorp 的 GPG 密钥：

```
gpg --no-default-keyring \
    --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    --fingerprint
```

使用适合你系统的命令将官方 HashiCorp 仓库添加到系统中。该命令使用`lsb_release -cs`查找当前系统的发行版代号，如`buster`、`groovy`或`sid`，并将其添加到仓库文件中：

```
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
    https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
    sudo tee /etc/apt/sources.list.d/hashicorp.list
```

从 HashiCorp 下载包信息：

```
sudo apt update
```

从新的仓库安装 Terraform：

```
sudo apt-get install terraform
```

#### CentOS/RHEL

安装`yum-config-manager`以管理你的仓库：

```
sudo yum install -y yum-utils
```

使用`yum-config-manager`添加官方 HashiCorp Linux 仓库：

```
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
```

从新的仓库安装 Terraform：

```
sudo yum -y install terraform
```

#### Fedora

安装`dnf config-manager`以管理你的仓库：

```
sudo dnf install -y dnf-plugins-core
```

使用`dnf config-manager`添加官方 HashiCorp Linux 仓库：

```
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
```

从新的仓库安装 Terraform：

```
sudo dnf -y install terraform
```

#### Amazon Linux

安装`yum-config-manager`以管理你的仓库：

```
sudo yum install -y yum-utils
```

使用`yum-config-manager`添加官方 HashiCorp Linux 仓库：

```
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
```

从新的仓库安装 Terraform：

```
sudo yum -y install terraform
```

## 验证安装

完成安装过程后，验证 Terraform 是否正确安装是非常重要的。你可以打开一个新的终端会话，列出 Terraform 的可用子命令。这将帮助你确保 Terraform 正常工作，并且你可以开始使用 Terraform 创建和管理基础设施：

```
terraform -help
```

如果你收到类似以下的响应，说明你的安装成功：

```
> terraform -help
Usage: terraform [-version] [-help] <command> [args]
The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you're just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.
```

现在我们已经覆盖了如何安装 Terraform，接下来让我们通过配置 AWS 账户，使 Terraform 能够与 AWS 服务进行交互，来准备 Terraform 用于 AWS。

# 如何为 AWS 安装/准备 Terraform

一旦安装了 Terraform，你可以在 AWS 中创建你的第一个基础设施。Terraform 会生成一个执行计划，列出达到目标基础设施状态所需的步骤，并执行它。随着你对配置进行更改，Terraform 可以识别差异并生成增量执行计划来应用这些更改。

在接下来的步骤中，让我们在 AWS 中创建一个 S3 桶。首先，为了在 Terraform 中配置 AWS 资源，有一些前提条件。

## 前提条件

在成功安装 Terraform 后，你需要准备以下内容进行下一步操作：

+   AWS CLI 已安装

+   一个 AWS 账户以及具有创建资源权限的相关凭证

## AWS CLI 安装

**AWS CLI**是一个全面的工具，使你可以通过单一统一的界面管理 AWS 服务。它简化了多个 AWS 服务的管理，并通过脚本实现自动化。使用 AWS CLI，你可以直接从命令行执行 AWS 命令，更高效地管理 AWS 资源。

### Linux

要安装 `aws-cli`，你的系统必须能够解压或解开下载的包。如果你的操作系统没有内置的 `unzip` 命令，你可以使用等效的工具。AWS CLI 需要 `glibc`、`groff` 和 `less`。这些组件通常在大多数主流 Linux 发行版中默认包含。

以如下方式下载安装包：

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

通过解压安装程序来提取它。如果你的 Linux 发行版没有内置的 `unzip` 命令，你可以使用其他工具来解压。以下命令是解压包并在当前目录下创建一个名为 `aws` 的目录的示例：

```
unzip awscliv2.zip
```

使用提取后的 AWS 目录中的安装文件执行安装命令。默认安装路径是 `/usr/local/aws-cli`，并且会在 `/usr/local/bin` 创建一个符号链接。该命令可能需要使用 `sudo` 来授予这些目录的写权限：

```
sudo ./aws/install
```

使用以下命令确认安装：

```
aws –version
```

你应该会收到类似于以下的响应：

```
aws-cli/2.7.24 Python/3.8.8 Linux/4.14.133-113.105.amzn2.x86_64 botocore/2.4.5
```

### macOS

如果你拥有 `sudo`/管理员权限，可以使用以下命令为计算机上的所有用户安装 AWS CLI。

使用 `curl` 命令下载文件。`-o` 选项指定了下载的包文件保存的文件名。在这个示例中，文件被保存为当前文件夹中的 `AWSCLIV2.pkg`：

```
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
```

这个过程在 macOS 系统上有所不同，你可以运行标准的 macOS 安装程序来安装 Terraform。你只需使用 `-pkg` 参数指定已下载的 `.pkg` 文件作为源，并使用 `-target /` 参数指定安装驱动器。在安装过程中，文件将被安装到 `/usr/local/aws-cli`，并且会在 `/usr/local/bin` 自动创建一个符号链接。但是，要为这些文件夹授予写权限，你必须在命令中加入 `sudo`：

```
sudo installer -pkg ./AWSCLIV2.pkg -target /
```

为了验证 shell 是否能够在 `$PATH` 中找到并运行 `aws` 命令，可以使用以下命令：

```
which aws
```

返回的响应应该是以下内容：

```
/usr/local/bin/aws
```

验证 `PATH` 配置和版本：

```
aws –version
```

返回的响应应该类似于以下内容：

```
aws-cli/2.7.24 Python/3.8.8 Darwin/18.7.0 botocore/2.4.5
```

如果找不到 `aws` 命令，可能需要重新启动终端并重新验证。

### Windows

你需要管理员权限才能在 Windows 系统上安装 `aws-cli`。

下载并运行适用于 Windows（64 位）的 AWS CLI MSI 安装程序：[`awscli.amazonaws.com/AWSCLIV2.msi`](https://awscli.amazonaws.com/AWSCLIV2.msi)

或者，你可以运行 `msiexec` 命令从命令提示符运行 MSI 安装程序：

```
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

你可以通过打开命令提示符窗口并运行 `aws --version` 命令，来验证 AWS CLI 是否已安装在你的 Windows 机器上：

```
aws –version
```

这将显示已安装的 AWS CLI 版本，确认其已成功安装到你的系统上。

返回的响应应该类似于以下内容：

```
aws-cli/2.7.24 Python/3.8.8 Windows/10 exe/AMD64 prompt/off
```

如果 Windows 无法找到该程序，您可能需要关闭并重新打开命令提示符窗口以刷新路径并重新验证。

### AWS 账户

如果您没有 AWS 账户，可以按照以下步骤创建一个；如果您已经拥有账户，请跳过 AWS 凭证配置部分：

1.  打开 AWS 首页 [`aws.amazon.com/`](https://aws.amazon.com/)。

1.  选择**创建一个** **AWS 账户**。

1.  确保提供准确的账户信息，包括您的电子邮件地址，并选择**继续**。输入错误的电子邮件地址可能会导致无法访问您的 AWS 账户。

1.  选择**个人**或**专业**。

1.  输入您的公司或个人信息。

1.  阅读并接受 AWS 客户协议。

1.  选择**创建账户**并**继续**。

1.  在**支付信息**页面提供您的支付信息，并选择**验证并添加**以继续。必须提供有效的支付方式才能完成注册过程。

1.  完成支付信息后，您需要验证您的电话号码。您需要从列表中选择国家或地区代码，并提供一个您可以在接下来的几分钟内访问的电话号码以供验证。

1.  输入 CAPTCHA 显示的验证码，然后提交。

1.  在自动系统联系您后，您需要输入收到的 PIN 并选择**继续**选项。

1.  在**选择支持计划**页面，选择一个可用的 AWS 支持计划。

1.  最后，等待您的新账户激活。这通常需要几分钟，但最长可能需要 24 小时。

1.  一旦您的账户完全激活，AWS 会向您发送确认邮件。请务必检查您的电子邮件收件箱和垃圾邮件文件夹以查收确认信息。收到邮件后，您将可以完全访问所有 AWS 服务。

### AWS 凭证

为了通过编程方式与 AWS 进行交互，您需要提供 AWS 访问密钥。这些密钥在进行编程请求时验证您的身份。访问密钥可以是有效期较短的临时凭证，也可以是长期凭证，例如与 IAM 用户或 AWS 账户根用户关联的凭证。

为了与 AWS 一起使用 Terraform，建议创建一个具有特定权限的专用用户，并为 AWS CLI 和 Terraform CLI 生成访问密钥，以便它们能够通信并配置资源。

## 创建 IAM 用户和 Terraform 凭证

创建 IAM 用户是 AWS 中的标准做法，用于安全性和资源访问管理。可以创建多个 IAM 用户并为他们提供特定的 AWS 资源权限。创建 IAM 用户的过程简单，通常是在新团队成员加入公司时或新的应用程序需要访问 AWS 资源时进行的。对于 Terraform，您需要创建一个 IAM 用户并分配适当的权限以便进行资源配置，如下所示：

1.  登录到 AWS 管理控制台，并在[`console.aws.amazon.com/iam/`](https://console.aws.amazon.com/iam/)打开 IAM 控制台。

1.  在左侧导航窗格中，选择**用户**，然后选择**添加用户**。

1.  使用`terraform`作为新用户的用户名。这是 AWS 的登录名。

1.  选择该用户将拥有的访问类型。您可以选择程序化访问和访问 AWS 管理控制台；我建议只选择程序化访问，这样您就不需要维护程序化用户的密码生命周期。

1.  选择**下一步：权限**。

1.  将`terraform`用户添加到一个组，复制现有用户的权限，或者直接附加策略。如果计划使用此用户提供所有资源，可以将**AdministratorAccess**策略附加到该用户，但这会对账户中的所有资源提供广泛的权限，因此请确保保持凭证的机密性。

1.  选择**下一步：标签**。

1.  （可选）在**标签**页面，通过附加键值对标签来为用户添加元数据。

1.  选择**下一步：查看**。验证要添加到新用户的权限。当您准备好继续时，选择**创建用户**。

1.  通过选择每个密码和访问密钥旁边的**显示**选项，您可以查看该用户的访问密钥。要保存访问密钥，请选择**下载 .csv**选项，并将文件保存到安全位置。

    现在，您已经有了用户和凭证，可以继续进行接下来的步骤。

现在，您已经安装并准备好 Terraform 用于 AWS，是时候开始构建您的第一个 Terraform 配置了。凭借您的 AWS 访问密钥和用户权限，您现在可以开始编写 Terraform 代码，以在云中提供您的基础设施。

# 构建您的第一个 Terraform 配置

在安装了 Terraform 和 AWS CLI 之后，是时候配置它们之间的连接，以便能够使用 Terraform 创建资源。

要使用 IAM 凭证对 Terraform AWS 提供程序进行身份验证，您需要通过在终端中将您的密钥添加到`=`符号后，设置`AWS_ACCESS_KEY_ID`环境变量：

```
export AWS_ACCESS_KEY_ID=
```

然后，按照以下方式再次添加您的密钥，仍然是在`=`符号后：

```
export AWS_SECRET_ACCESS_KEY=
```

您可以使用以下命令验证您的凭证和连接性：

```
aws sts get-caller-identity
```

您应该会收到类似以下的输出：

```
{
    "Account": "1234567890",
    "UserId": "ABCDEFGHJIKLM",
    "Arn": "arn:aws:iam:: 1234567890:user/erol_kavas"
}
```

现在您已经成功构建了第一个 Terraform 配置，让我们继续构建第一个 Terraform 模板。

# 构建您的第一个 Terraform 模板

我们已经创建了 AWS 账户和一个 IAM 用户，并设置了 Terraform 和 AWS CLI 之间通信所需的凭证，以便提供基础设施。现在，让我们开始开发第一个 Terraform 模板。

每个 Terraform 配置都需要一个专用的工作目录。首先，为您的第一个 Terraform 项目创建一个新目录。可以使用任何代码编辑器或终端，我们将提供终端命令以供您参考：

```
mkdir my-first-terraform-project
```

切换到该目录，以便我们可以开始在其中创建文件：

```
cd my-first-terraform-project
```

创建一个空文件来定义你的基础设施：

```
touch main.tf
```

打开你喜欢的文本编辑器中的`main.tf`文件，并将以下配置复制到该文件中。添加完配置后，保存该文件：

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}
# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
}
resource "aws_vpc" "example" {
  cidr_block = "10.0.0.0/16"
}
```

在这里，我们明确声明我们正在使用 Terraform 的 AWS 提供者插件，并要求版本大于 4.0。我们没有提供我们在 IAM 中为用户创建的访问密钥/秘密，因为它们已经导入到终端中，并且你永远不应该在 Terraform 模板中硬编码凭证！我们还提供了我们希望进行所有更改的区域。在这个示例中，我们使用的是`us-east-1`。

然后，我们使用`aws_vpc`资源标识符来声明我们要启动一个 VPC 实例，并跟随`example`名称标识符。（这个名字可以是你喜欢的任何名字。）

我们提供`cidr_block`信息，以为 VPC 提供 cidr（IP 地址空间）信息。

现在，我们将在创建`main.tf`文件的目录中运行`terraform init`命令，以下载并初始化适当的提供者插件。在此情况下，我们将下载在`main.tf`文件中指定的 AWS 提供者插件：

```
terraform init
```

输出应如下所示：

```
Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 4.0"...
- Installing hashicorp/aws v4.36.1...
- Installed hashicorp/aws v4.36.1 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.
Terraform has been successfully initialized!
You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.
If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

现在，提供者已经安装，我们已初始化项目。让我们使用以下命令验证我们的模板：

```
terraform validate
```

输出必须如下所示：

```
Success! The configuration is valid.
```

验证我们的配置后，我们可以使用`terraform fmt`来格式化我们的配置文件，以使用正确的格式和风格。在 Terraform 的较新版本中，`terraform fmt`中新引入的格式化规则不被视为破坏性更改。然而，我们的目标是尽量减少对已经符合 Terraform 文档提供的样式指南的配置文件进行更改。

# 配置和测试你的模板

完成验证后，让我们运行`terraform plan`命令。这将让我们在决定是否应用之前看到 Terraform 将要执行的操作：

```
terraform plan
```

输出应如下所示：

```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
Terraform will perform the following actions:
  # aws_vpc.example will be created
  + resource "aws_vpc" "example" {
      + arn                                  = (known after apply)
      + cidr_block                           = "10.0.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_classiclink                   = (known after apply)
      + enable_classiclink_dns_support       = (known after apply)
      + enable_dns_hostnames                 = (known after apply)
      + enable_dns_support                   = true
      + enable_network_address_usage_metrics = (known after apply)
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags_all                             = (known after apply)
    }
Plan: 1 to add, 0 to change, 0 to destroy.
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

Terraform 通过使用声明式配置文件提供了一种可靠和安全的方式来管理基础设施生命周期。在处理**基础设施即代码**（**IaC**）时，遇到的最大障碍是所谓的漂移现象。漂移是指基础设施的实际状态与配置中所述状态之间的差异。

`terraform plan`是一个非常重要的命令，用于检测 Terraform 管理的资源的漂移——你必须能够理解每一个输出以及计划输出中的所有变化，特别是包含摘要的那一行。在我们的示例中，输出如下：

```
Plan: 1 to add, 0 to change, 0 to destroy.
```

在检查了更改并验证它们是我们计划和预期的内容后，让我们继续执行`terraform apply`：

```
terraform apply
```

运行`apply`后，Terraform 将运行另一个计划，并要求我们验证并批准预测的更改。输出应类似于以下内容：

```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
Terraform will perform the following actions:
  # aws_vpc.example will be created
  + resource "aws_vpc" "example" {
      + arn                                  = (known after apply)
      + cidr_block                           = "10.0.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_classiclink                   = (known after apply)
      + enable_classiclink_dns_support       = (known after apply)
      + enable_dns_hostnames                 = (known after apply)
      + enable_dns_support                   = true
      + enable_network_address_usage_metrics = (known after apply)
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags_all                             = (known after apply)
    }
Plan: 1 to add, 0 to change, 0 to destroy.
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.
  Enter a value:
```

你应该仔细检查输出内容，如果一切正常，可以通过输入`yes`来批准。除`yes`外，基础设施的配置将被丢弃，`apply`将不会在你的环境中部署任何内容。

在你批准后，`terraform-cli`将开始部署你请求的资源，附加输出应类似于以下内容：

```
aws_vpc.example: Creating...
aws_vpc.example: Creation complete after 2s [id=vpc-xxxxxx]
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

最后一行给出了你的`terraform apply`命令的总结，帮助你轻松查看已部署、添加、修改或销毁的内容。

在 AWS 管理控制台验证资源后，你可以通过运行`terraform destroy`命令来销毁在本练习中创建的示例资源：

```
terraform destroy
```

当你使用 Terraform 来管理基础设施时，`terraform destroy`命令可以用来终止由 Terraform 项目创建的资源。它是`terraform apply`命令的相反操作，因为它删除了 Terraform 状态中指定的所有资源。然而，值得注意的是，`terraform destroy`命令不会销毁在其他地方运行且未由当前 Terraform 项目管理的资源。

输出应类似于此：

```
aws_vpc.example: Refreshing state... [id=vpc-xxxx]
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy
Terraform will perform the following actions:
  # aws_vpc.example will be destroyed
  - resource "aws_vpc" "example" {
      - arn                                  = "arn:aws:ec2:us-east-1:xxxx:vpc/vpc-xxxx" -> null
      - assign_generated_ipv6_cidr_block     = false -> null
      - cidr_block                           = "10.0.0.0/16" -> null
      - default_network_acl_id               = "acl-0cbfcf0156e3eec97" -> null
      - default_route_table_id               = "rtb-0933253f8baad1cb2" -> null
      - default_security_group_id            = "sg-09aa1459d60ec7939" -> null
      - dhcp_options_id                      = "dopt-26ad5a5f" -> null
      - enable_classiclink                   = false -> null
      - enable_classiclink_dns_support       = false -> null
      - enable_dns_hostnames                 = false -> null
      - enable_dns_support                   = true -> null
      - enable_network_address_usage_metrics = false -> null
      - id                                   = "vpc-xxxx" -> null
      - instance_tenancy                     = "default" -> null
      - ipv6_netmask_length                  = 0 -> null
      - main_route_table_id                  = "rtb-xxxx" -> null
      - owner_id                             = "xx" -> null
      - tags                                 = {} -> null
      - tags_all                             = {} -> null
    }
Plan: 0 to add, 0 to change, 1 to destroy.
Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.
  Enter a value:
```

`terraform destroy`中的`-`前缀表示实例将被销毁。与`apply`类似，Terraform 显示其执行计划并在做出任何更改之前等待批准。仔细审查所有更改并予以批准非常重要，因为一旦批准，资源将无法恢复。

输入`yes`来执行此计划并销毁基础设施：

```
aws_vpc.example: Destroying... [id=vpc-xxxx]
aws_vpc.example: Destruction complete after 1s
Destroy complete! Resources: 1 destroyed.
```

类似于`apply`命令，Terraform 将确定销毁资源的顺序。如果有多个资源存在依赖关系，Terraform 将按照这些依赖关系的适当顺序销毁它们。在这种情况下，Terraform 识别到一个没有依赖关系的 VPC 并将其销毁。

# 总结

本章中，我们学习了如何安装 Terraform 并准备好与 AWS 一起使用。我们介绍了包括下载二进制文件、使用包管理器和从源代码编译等多种安装方法。此外，我们还讨论了如何设置 AWS 账户并为 Terraform 创建 IAM 用户。接着，我们介绍了如何创建我们的第一个 Terraform 项目的目录，粘贴配置代码，并使用`terraform apply`命令来部署资源。最后，我们学习了如何使用`terraform destroy`命令来销毁我们 Terraform 项目创建的资源。通过本章学到的技能，你现在应该能够使用 Terraform 在 AWS 上创建和管理基础设施。

在接下来的章节中，我们将探讨 Terraform 在**IaC**项目中的应用。

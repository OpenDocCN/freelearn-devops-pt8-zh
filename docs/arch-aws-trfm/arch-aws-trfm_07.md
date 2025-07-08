# 第七章：7

# 在项目中实现 Terraform

您准备好开始使用 Terraform 开发您的 AWS 基础设施了吗？在本章中，您将学习 Terraform 的基础知识，并了解如何在 AWS 中部署您的第一个模板。我们将介绍选择合适的 AWS 提供商和选择满足您项目需求的公共模块的过程。您还将学习如何为特定的使用场景编写自定义的 Terraform AWS 模块。

本章结束时，您将使用 Terraform 开发和部署您的 AWS 基础设施。您还将获得有关提供商决策和选择适合项目需求的公共模块的宝贵技能。此外，您将学习如何以及何时开发自定义的 AWS 模块，并了解如何有效地使用它们。

在本章中，我们将深入探讨 Terraform，了解它如何用于开发和部署 AWS 基础设施项目。以下是我们将要覆盖的主题：

+   用于开发 AWS 基础设施项目的 Terraform 基础知识

+   选择 AWS 提供商

+   为您的需求选择 AWS 公共模块

+   如何编写自定义 Terraform AWS 模块

# 用于开发 AWS 基础设施项目的 Terraform 基础知识

Terraform 是一个用于安全高效地构建、修改和版本化基础设施的工具。它可以管理多种不同类型云提供商的基础设施，包括 AWS。

让我们来看一些 Terraform 的基本概念。

## 资源

资源是您基础设施的一个元素，如 EC2 实例、S3 存储桶或安全组。

资源通常是通过在 Terraform 配置中使用资源块创建的。资源块有一个类型和一个名称，并指定资源的期望状态。例如，以下块将在 AWS 中创建一个 EC2 实例：

```
resource "aws_instance" "web_server" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```

这个资源块创建了一个类型为 `aws_instance`、名称为 `web_server` 的 EC2 实例。它指定该实例应使用指定的 AMI 和实例类型来创建。

当您运行 `terraform apply` 时，Terraform 将创建 EC2 实例，并将其设置为资源块中指定的期望状态。如果 EC2 实例已经存在并且其状态与期望状态不同，Terraform 将更新实例以使其与期望状态匹配。

您还可以使用资源属性来指定资源的附加细节，例如应该创建资源的 VPC、它应该关联的安全组等。

## 提供商

提供商是 Terraform 用来与特定云提供商（如 AWS）的基础设施进行交互的插件。每个提供商都有自己的一套资源，您可以使用这些资源来创建和管理基础设施。

要在 Terraform 配置中使用提供商，您需要在提供商块中指定它。例如，要使用 AWS 提供商，您需要在配置中添加以下块：

```
provider "aws" {
  region = "us-east-1"
}
```

该块指定您希望使用 AWS 提供者，并且您希望使用 `us-east-1` 区域。您还可以指定特定于提供者的配置选项，例如用于通过提供者的 API 进行身份验证时使用的访问密钥和秘密密钥。

一旦在您的配置中指定了一个提供者，您就可以使用该提供者的资源来创建和管理基础设施。例如，您可以使用 `aws_instance` 资源在 AWS 中创建 EC2 实例：

```
resource "aws_instance" "web_server" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```

## 状态

Terraform 维护一个状态文件，存储您基础设施的当前配置。这使它能够跟踪更改，并知道在您更改配置时应该采取哪些操作。

状态文件是 Terraform 工作方式的重要部分，因为它使 Terraform 知道在您更改基础设施时应该执行哪些操作。例如，如果您使用 Terraform 创建一个新的 EC2 实例，状态文件将会更新，以反映新实例的存在。如果您随后使用 Terraform 修改该实例，状态文件将会更新，以反映实例的新配置。

有几种不同的方式可以存储您的状态文件：

+   `terraform.tfstate`。这是最简单的选项，但如果您在团队中工作并需要共享状态文件，这可能会很不方便。

+   **远程状态文件**：您也可以将状态文件存储在远程位置，例如 S3 存储桶或 Terraform Cloud 工作区。这样可以与团队的其他成员共享状态文件，并且如果您的本地计算机损坏，它也有助于防止数据丢失。

+   **锁定状态文件**：在使用远程状态文件时，您可以启用状态文件锁定，以防止多个用户同时修改状态文件。这有助于避免当多个用户同时更改基础设施时发生冲突。

## 模块

模块是自包含的 Terraform 配置包，可以共享和重用。您可以使用模块以更模块化、可重用的方式定义基础设施。

例如，假设您想要创建一个带负载均衡器和数据库的 Web 服务器集群。您可以为这些组件创建一个模块，然后使用这些模块来构建 Web 服务器集群。这样，您可以在其他基础设施项目中重用这些模块，并且可以使您的代码更容易理解和维护。

要创建一个模块，您需要创建一个包含一个或多个 Terraform 配置文件以及一个 `module.tf` 文件的目录，该文件指定模块的输入和输出。然后，您可以通过在另一个 Terraform 配置文件中使用模块块来调用该模块，并将其用于您的基础设施。

这是一个调用名为 `web_server_cluster` 模块的模块块示例：

```
module "web_server_cluster" {
  source = "./web_server_cluster"
  num_web_servers = 3
  web_server_size = "t2.micro"
  database_size = "t2.micro"
}
```

这个块指定了模块的源（一个名为`web_server_cluster`的本地目录），并为模块设置了输入变量（`num_web_servers`、`web_server_size`和`database_size`）。然后，模块可以使用这些输入变量来创建所需的基础设施。

## 变量

变量允许你对配置进行参数化，使其更加灵活。你可以使用变量来定义在多个地方使用的值，或者你希望能够轻松调整的值。

要在 Terraform 配置中定义变量，可以使用变量块。以下是一个定义名为`image_id`的变量的变量块示例：

```
variable "image_id" {
  type = string
}
```

这个块定义了一个类型为字符串的变量，名为`image_id`。然后，你可以在配置中通过`${var.name}`语法引用此变量，例如：

```
resource "aws_instance" "web_server" {
  ami           = "${var.image_id}"
  instance_type = "t2.micro"
}
```

你还可以使用`default`属性为变量指定默认值，例如：

```
variable "image_id" {
  type    = string
  default = "ami-12345678"
}
```

这将`image_id`变量的默认值设置为`ami-12345678`。如果在运行 Terraform 时没有为变量指定值，它将使用默认值。

你可以通过以下方法之一设置变量的值：

+   **硬编码值**：你可以直接在配置中使用默认属性设置变量的值。这对于简单配置或测试很有用。

+   **使用环境变量**：你可以使用环境变量设置变量的值。这对于存储敏感信息或管理多个环境（例如测试和生产环境）非常有用。

+   `terraform.tfvars`)并在配置中通过`-var-file`标志引用该文件。这对于在多个配置之间共享值或存储不希望提交到版本控制的值非常有用。

## 输出

输出是从 Terraform 配置中导出的值，可以从其他配置中访问，或被外部系统使用。

要在 Terraform 配置中定义输出，可以使用`output`块。以下是一个`output`块示例，导出 EC2 实例的公共 IP 地址：

```
output "public_ip" {
  value = "${aws_instance.web_server.public_ip}"
}
```

这个`output`块导出了一个名为`public_ip`的值，并将其设置为`web_server` EC2 实例的公共 IP 地址。然后，你可以使用`terraform output`命令访问此输出的值，或者在另一个 Terraform 配置中通过`${output.name}`语法引用它。

输出对于显示关于基础设施的重要信息非常有用，例如资源的 IP 地址或应用程序的 URL。它们还可以用于在多个配置之间传递信息，例如在一个配置中创建的 S3 存储桶的 ID，并在另一个配置中使用。

## 提供程序

Provisioner 用于在资源创建后执行脚本或进行 API 调用。这对于安装 EC2 实例上的软件或上传文件到 S3 存储桶等任务非常有用。

Provisioner 是在 `resource` 块中的 `provisioner` 块内定义的。例如，以下的 `resource` 块包括一个 provisioner，该 provisioner 在 EC2 实例创建后运行一个 shell 脚本：

```
resource "aws_instance" "web_server" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
  provisioner "remote-exec" {
    inline = [
      "apt-get update",
      "apt-get install -y nginx",
    ]
  }
}
```

该 provisioner 使用 `remote-exec` provisioner 类型，它允许你通过 SSH 在远程主机上执行命令。inline 属性指定了要运行的命令。在这种情况下，provisioner 在 EC2 实例上安装了 `nginx` web 服务器。

Terraform 支持几种不同类型的 provisioner，包括 `file`、`remote-exec` 和 `local-exec`。你可以根据需求和管理的资源类型选择不同的 provisioner。

# 选择 AWS 提供商

在 Terraform 中，提供商是一个插件，它将 Terraform 与特定的基础设施平台（如 AWS、Google Cloud 或 Azure）集成。提供商负责了解基础设施平台的 API，并暴露可以使用 Terraform 创建、修改和销毁的资源。

Terraform 提供商有两种类型——官方提供商和第三方提供商：

+   **官方提供商**由 HashiCorp 开发和维护，HashiCorp 是 Terraform 的背后公司。这些提供商被认为是最稳定和可靠的，因为它们由 HashiCorp 支持，并定期更新。

+   **第三方提供商**由外部组织或个人开发和维护。这些提供商不受 HashiCorp 官方支持，但它们可以扩展 Terraform，支持更多的基础设施平台或工具。

你可以在 Terraform Registry 上找到所有可用的 Terraform 提供商的列表。([`registry.terraform.io/browse/providers`](https://registry.terraform.io/browse/providers))。该注册表列出了官方提供商和第三方提供商，并包含关于提供商与 Terraform 兼容性及其支持的资源的信息。

要在 Terraform 中选择一个提供商，你需要在配置中指定它的 `provider` 块。例如，要使用 AWS 提供商，你可以将以下块添加到你的配置中：

```
provider "aws" {
  region = "us-east-1"
}
```

该块指定了你想要使用 AWS 提供商，并选择 `us-east-1` 区域。你还可以指定特定提供商的配置选项，例如在使用提供商 API 进行身份验证时使用的访问密钥和秘密密钥。

你可以在一个配置中使用多个 `provider` 块来管理跨多个提供商的资源。例如，你可能会使用 AWS 提供商来管理你的 EC2 实例，使用 Google Cloud 提供商来管理你的计算引擎实例。

要指定使用的提供商版本，可以在 `provider` 块中使用 `version` 属性，例如：

```
provider "aws" {
  version = "~> 2.0"
}
```

这指定了你希望使用与 2.0 版本兼容的最新 AWS 提供者版本。

通过运行`terraform init`命令，Terraform 将自动下载配置中指定的提供者所需的插件。要强制使用特定版本的提供者，你可以在`provider`块中添加`version`属性。

你还可以在运行`terraform init`时使用`-upgrade`标志来指定提供者的版本。这将强制 Terraform 下载最新版本的提供者，即使你已经下载了一个兼容的版本。

如果你想使用特定版本的提供者，可以在`version`属性中指定版本号，例如：

```
provider "aws" {
  version = "2.23.0"
}
```

这指定了你希望使用版本 2.23.0 的 AWS 提供者。

如果你想使用最新版本的提供者，可以将`version`属性设置为`latest`，例如：

```
provider "aws" {
  version = "latest"
}
```

这指定了你希望使用 AWS 提供者的最新版本。

你还可以在运行`terraform init`时使用`-get-plugins`标志来下载配置中所有提供者的最新版本。

目前在 Terraform 中有两个 AWS 提供者：

+   **aws 提供者**是传统的提供者，使用 Go 语言编写。它使用 AWS 的 Go SDK 来向 AWS 发送 API 请求，长期以来它一直是 Terraform 的默认 AWS 提供者。

+   **aws.sdk 提供者**是一个较新的提供者，使用 TypeScript 编写。它使用 AWS 的 JavaScript SDK 来向 AWS 发送 API 请求。它在 Terraform v0.13 中作为实验性提供者引入，并在 Terraform v0.14 中成为稳定提供者。

一般来说，`aws.sdk`提供者比`aws`提供者更受欢迎，因为它功能更全面，对 AWS 服务的支持更好。然而，`aws`提供者仍然被广泛使用，并且在可预见的未来可能会继续得到支持。

# 根据你的需求选择 AWS 公共模块

要在 Terraform 中使用公共模块，你需要在配置中的`module`块中指定模块的来源。你可以通过本地路径、Terraform 公共注册表、Git 仓库网址或压缩归档文件的 URL（例如`.zip`或`.tar.gz`文件）来指定模块的来源。

下面是一些示例，展示了如何在`module`块中指定公共模块的来源：

`source`属性，例如：

```
module "web_server_cluster" {
  source = "./web_server_cluster"
}
```

这指定了`web_server_cluster`模块位于名为`web_server_cluster`的本地目录中。

`source`属性，例如：

```
module "web_server_cluster" {
  source = "git::https://github.com/example/web_server_cluster.git"
}
```

这指定了`web_server_cluster`模块位于 Git 仓库中，网址为[`github.com/example/web_server_cluster.git`](https://github.com/example/web_server_cluster.git)。

`source`属性，例如：

```
module "web_server_cluster" {
  source = "http://example.com/web_server_cluster.zip"
}
```

这指定了`web_server_cluster`模块位于 URL `http://example.com/web_server_cluster.zip`的`.zip`文件中。

`module` 块并将 `source` 属性设置为模块在 registry 上的 URL。

下面是一个调用 Terraform Registry 上托管模块的 `module` 块示例：

```
module "web_server_cluster" {
  source = "my-terraform-modules/web_server_cluster"
}
```

你可以在 Terraform Registry 上找到可用的公共模块列表（[`registry.terraform.io/browse/modules`](https://registry.terraform.io/browse/modules)）。该 Registry 包含官方和第三方模块，并提供模块与 Terraform 的兼容性信息以及它支持的资源。你可以使用搜索栏查找特定模块，或者浏览类别找到针对特定基础设施平台或用例的模块。

## 如何决定选择哪个 Terraform 模块

在决定在你的基础设施中使用哪些 Terraform 模块时，有几个因素需要考虑：

+   **兼容性**：确保该模块与你正在使用的 Terraform 版本兼容。你可以在模块的 Terraform Registry 页面查看兼容性信息。

+   **支持的资源**：检查模块是否支持你需要创建或管理的资源。你可以在 Terraform Registry 上的模块页面找到该模块支持的资源列表。

+   **输入变量**：确保模块包含你需要的输入变量，以自定义模块的行为。你可以在 Terraform Registry 上的模块页面找到模块的输入变量列表。

+   **输出值**：检查模块是否有你需要的输出值，以便访问创建的资源或在模块之间传递信息。你可以在 Terraform Registry 上的模块页面找到该模块的输出值列表。

+   **维护**：考虑模块的维护状态。检查模块最后更新的时间以及它是否在积极维护。你可能想选择一个更积极维护的模块，以确保它保持最新，并且能够修复 bug 和添加新功能。

# 如何编写自定义 Terraform AWS 模块

要编写一个 Terraform 模块，你需要创建一个配置文件或一组配置文件，定义你想要创建或管理的资源。模块本质上是一个可重用的配置，可以从其他配置或模块中调用。

以下是编写 Terraform 模块的步骤：

1.  `resource` 块用于定义你想要创建或管理的资源。例如，你可能会使用 `aws_instance` 资源来创建 EC2 实例，或使用 `aws_s3_bucket` 资源来创建 S3 桶。

1.  `variable` 块用于定义模块的输入变量。输入变量允许模块的用户在调用模块时自定义其行为。例如，你可能会定义一个变量来指定要创建的 EC2 实例数量，或指定 S3 桶的名称。

1.  `output` 块用于定义模块的输出值。输出值允许模块的用户访问创建的资源或在模块之间传递信息。例如，你可能会定义一个输出值，表示 EC2 实例的公共 IP 地址或 S3 桶的 URL。

1.  使用 `terraform plan` 和 `terraform apply` 测试模块，并确保它按照预期创建资源。

1.  **模块文档**：在模块的 README 文件中记录输入变量、输出值以及关于该模块的其他重要信息。

# 总结

在本章中，我们介绍了使用 Terraform 开发 AWS 基础设施项目的基础知识。我们学习了如何在 AWS 中部署第一个 Terraform 模板，选择 AWS 提供商，并选择最适合我们需求的公共 AWS 模块。我们还探讨了如何编写自定义的 Terraform AWS 模块，并如何有效地使用它们。本章结束时，你应该能够开发并部署自己的 AWS 基础设施，做出关于提供商的决策，选择最合适的公共 AWS 模块，并创建和使用自定义的 Terraform AWS 模块。这些技能将为你继续探索 Terraform 和 AWS 基础设施开发打下基础。

无论你是无服务器架构的新手还是经验丰富的开发者，使用 Terraform 部署无服务器项目都能为你的开发工作流带来变革。在下一章中，我们将深入探讨无服务器架构，并学习如何使用 Terraform 在 AWS Lambda 上部署无服务器应用。从配置 AWS API 和身份验证到处理事件触发器和自动扩展，你将掌握成功部署和管理无服务器项目所需的技能。

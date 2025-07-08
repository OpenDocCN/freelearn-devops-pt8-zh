# 第三章：3

# 利用 HashiCorp 实用程序提供程序

正如我们在第一章中讨论的，当我们了解 Terraform 的架构时，Terraform 被设计为可扩展的。在上一章中，我们花了很多时间研究 **HashiCorp 配置语言** (**HCL**)，它提供了许多工具，可以帮助我们定义 **基础设施即代码** (**IaC**)。然而，这些语言工具并不总是足够的。这就是为什么 HashiCorp 构建了一套实用的提供程序，它提供了一种基础类库或一套可重用的功能，适用于特定场景，无论你使用什么云平台来构建你的 IaC 解决方案。

本章涵盖以下主题：

+   处理现实

+   适配与集成

+   文件系统

+   操作系统和网络

# 处理现实

当我们用 IaC 构建架构时，产物不是代码，而是活生生的环境。虽然代码存在于我们大脑的抽象领域中，但这些环境在现实世界中运作，就像我们的最佳计划会被现实击碎一样——我们的环境也会如此。

因此，我们需要一些工具来准备我们的环境，以面对现实并与之应对。`random` 和 `time` 提供程序使我们能够避免资源和环境之间的冲突——无论是某个事物的名称，还是某个事物的过期时间。这些都是我们解决方案设计中的关键元素，当架构遇到现实世界时，它们可能决定成败。

## 随机化

`random` 提供程序提供了多种方法来为你的 Terraform 解决方案添加随机性。每种 `random` 资源类型可能会生成不同类型的随机值，并具有其他属性来控制输出。然而，除了少数几个例外，它们都通过一个叫做 `result` 的输出生成随机值。它们还至少有一个名为 `keepers` 的属性，用于触发 Terraform 重新创建资源。当你有频繁替换的临时资源时，这个属性特别有用，它可以帮助你确保在销毁并重新创建资源时没有名称冲突。

### 随机字符串

生成随机字符串是保证部署唯一性的好方法，特别是在动态生成短生命周期环境的情况下。根据不同的场景，有两种生成字符串的方式——一种用于非敏感数据，如资源名称，另一种用于敏感数据，如访问密钥和密码。

生成非敏感动态名称可以通过 `random_string` 来完成。同时，`random_password` 可以生成敏感值，你应该通过将其标记为敏感来防止泄漏，如果你输出它们，还需要通过保护你的状态来确保安全，因为 Terraform 会将结果值存储在状态中：

```
    resource "random_string" "name_suffix" {
      length           = 6
      special          = true
      override_special = "/@£$"
    }
```

上述代码生成一个随机字符串，可以在项目中生成唯一的资源名称。当您的资源名称长度要求较小时，使用短的随机字符串嵌入到资源名称中是一个很好的策略，因为当一个或两个资源需要异常小的名称长度时，创建一致的命名约定会变得很有挑战性。这种情况通常出现在资源需要具有全球唯一名称时，比如 S3 存储桶或 Azure 存储帐户：

```
    locals {
      resource_name = "foobar${random_string.name_suffix.result}"
    }
```

当您将随机名称后缀与命名约定的部分结合时，仍然可以获得一个相对合理的资源名称：

```
    resource "random_password" "database" {
      length           = 16
      special          = true
      override_special = "!#$%&*()-_=+[]{}<>:?"
    }
```

#### 唯一标识符

您还可以生成`random_uuid`。这对于您的资源支持非常长的名字时很有帮助，因为这些不区分大小写的字母数字值采用以下格式：`00000000-0000-0000-0000-000000000000`。您可能需要这个来生成唯一的关联标识符，以便在部署中使用公共标签将资源链接起来。

#### 纯粹为了娱乐

还有一个有趣的小资源叫做`random_pet`，它向古老的*宠物与牲畜*笑话致敬，您可以用它生成宠物名字。这个资源可能对生产环境没有太大用处，但在开发或实验室环境中，当您可以更加自由地为资源命名时，它是很有帮助的。`random_pet`资源的`id`输出将生成形容词-名词格式的名字。以下是我通过书中这一章的示例生成的一些示例值：

+   `notable-coyote`

+   `quiet-parakeet`

+   `pure-woodcock`

+   `healthy-monkey`

+   `mint-foal`

+   `pet-serval`

+   `ideal-lab`

+   `special-urchin`

您可以看到几乎所有的名字都没有什么意义，但有些可能很有趣。

### 随机数

生成随机数还可以帮助生成随机名称或从数组中生成随机索引。`random_integer`提供了一个简单的解决方案，允许您从指定的`min`和`max`值之间选择一个数字。

考虑以下 AWS 可用区数组：

```
    locals {
      azs = [
        "us-west-1a",
        "us-west-1c",
        "us-west-1d",
        "us-west-1e"
      ]
    }
```

如果我们想从这个`list`中选择一个随机的可用区，我们可以使用`random_integer`来生成一个随机索引：

```
    resource "random_integer" "az" {
      min = 0
      max = length(local.azs) – 1
    }
```

上面的代码将允许我们在`0`和`list`长度减去`1`之间生成一个随机整数，即`4 - 1 = 3`。因此，我们将随机生成`0`、`1`、`2`或`3`。

我们可以通过以下代码访问随机的可用区：

```
    locals {
      selected_az = local.azs[random_integer.az.result]
    }
```

最后，我们可以使用可用区名称来配置我们的 AWS 资源：

```
    resource "aws_elb" "foo" {
      availability_zones = [local.selected_az]
      # ... and other aws_elb arguments ...
    }
```

#### 超越简单的整数

当简单的整数不足以满足需求时，您可以使用`random_id`来生成更复杂的输出。唯一的输入是`byte_length`，它控制生成的随机数的大小。这个资源与其他`random`提供者的资源不同，因为它没有`result`输出，而是有几个其他输出，以不同格式呈现随机数，包括十进制、十六进制和 Base64：

```
    resource "random_id" "foo" {
      byte_length = 8
    }
```

上述代码生成了一个长度为 8 字节的随机数。不同格式的输出值示例如下：

+   `IpVgeF7uUY0`

+   `2492004038924456333`

+   `229560785eee518d`

+   `IpVgeF7uUY0=`

+   `=` `IpVgeF7uUY0`

同样，根据你的命名规范，这对创建独特标识资源的名称或标签非常有用。

### 混洗

在前面的例子中，当从列表中选择可用区时，如果我们想从列表中随机选择多个项，我们需要生成多个 `random_integer` 资源。使用 `random_integer` 来做到这一点已经相当繁琐，如果我们有要求确保第二个 `random_integer` 的结果不与第一个相同，问题会更加复杂。

幸运的是，选择数组索引的替代方法是使用内建资源来从 `list` 中选择随机子集。你可以使用 `random_shuffle` 资源来实现此方法，通过 `list` 和你想要的项数（通过 `result_count` 属性传递）来实现。输出的 `result` 将是一个你可以使用的字符串 `list`。如果我们希望 AWS **Elastic Load Balancer** (**ELB**) 跨多个可用区展开，这种方法会极大简化我们的解决方案。

请看以下 AWS 可用区的数组：

```
    locals {
      azs = [
        "us-west-1a",
        "us-west-1c",
        "us-west-1d",
        "us-west-1e"
      ]
    }
```

我们将使用 `random_shuffle` 来为我们的 ELB 生成两个可用区：

```
    resource "random_shuffle" "azs" {
      input        = local.azs
      result_count = 2
    }
```

最后，我们使用 `random_shuffle` 的 `result` 来设置 `availability_zones` 属性，因为它的输出是正确类型的 `list(string)`：

```
    resource "aws_elb" "foo" {
      availability_zones = random_shuffle.azs.result
      # ... and other aws_elb arguments ...
    }
```

这个资源非常有用，但使用时需要小心，因为它可能导致你的解决方案变得非确定性——意味着 Terraform 需要等到 `random_shuffle` 资源创建完成后，才能确定如何创建计划。这可能需要你使用有针对性的 `terraform apply` 操作，以避免第一次应用失败。

## 与时间相关的工作

在 Terraform 中，`time` 提供商提供了几项功能，使得在资源生命周期管理由时间决定的各种场景下更容易处理。

尽管大多数云提供商为资源调度提供了更好的解决方案，但在一些情况下，时间仍然在资源配置中起着至关重要的作用。这种情况通常涉及证书，你需要为证书设置固定的或滚动的过期时间窗口。在这种情况下，你可以使用未来的特定日期/时间，或者相对于当前日期/时间的日期/时间。

### 当前日期/时间

有时，你可能想捕捉当前的日期和时间，这些信息可能用于一个秘密值的生效日期。Terraform 提供了两种获取当前日期的方法——一个是函数，另一个是 `time_static` 资源：

```
    locals {
      option1 = timestamp()
    }
```

上述代码演示了如何使用 `timestamp()` 函数。以下代码展示了如何使用 `time` 提供商中的 `time_static` 资源：

```
    resource "time_static" "current_time" {}
```

这两种方法都会在你第一次运行 `apply` 时生成当前的日期/时间。不同之处在于，`timestamp()` 函数每次运行 `apply` 时都会生成当前的日期/时间戳。这使得它在一些场景中更加理想，比如给资源打上最后修改日期的标签，这有助于确定资源最后一次被 Terraform 修改的时间。另一个常见的场景是触发资源更新，你希望每次都触发更新，但要避免这种做法，因为这会导致解决方案中的持续变化。

与此同时，`time_static` 资源将在 State 中维护第一次 Terraform `apply` 时的原始日期/时间戳。这对于资源的生命周期管理很有用，可以帮助确定部署最初创建的时间，或根据年龄设置备份、扩展或退役策略。

### 固定日期/时间

可以使用日期/时间的字符串表示形式来创建一个未来的特定时间，方法是使用绝对的日期/时间：

```
    locals {
      future_date = "2024-05-04T00:00:00Z"
    }
```

上述代码会将过期日期设置为 2024 年 5 月 4 日。日期/时间的字符串表示格式为 `YYYY-MM-DDTHH:MM:SSZ`。

另一种选择是使用 `time_static` 资源并设置 `rfc3339` 属性，但由于其相较于简单设置本地时间的有限值，通常很少使用：

```
    resource "time_static" "may_the_fourth" {
      rfc3339 = "2024-05-04T00:00:00Z"
    }
```

### 时间偏移

可以使用相对于当前日期的周期来创建未来的特定时间，方法是使用 `time_offset`：

```
    resource "time_offset" "certificate_expiration" {
      offset_years = 1
    }
```

上述代码会将过期日期设置为恰好一年后的未来日期。有不同的属性可以调整偏移日期/时间戳，单位包括年份、月份、天数、小时、分钟和秒。你可以设置 `base_rfc3339` 属性来改变偏移的日期/时间的基准。这是动态设置某些过期日期的好方法。但你需要确保定期运行 Terraform 来保持目标日期在未来。

#### 旋转

在某些情况下，你可能需要定期重建资源。这个密钥可能需要每 90 天或 XXX 天更新一次。在这些情况下，`time_rotating` 资源相较于其静态同类资源 `time_static` 和 `time_offset` 提供了优势。

时间偏移看起来是旋转的解决方案，因为它是相对于当前日期的，但就像 `time_static` 一样，它只不过是另一种计算 Terraform 将存储在 State 中的静态日期/时间戳的方法。`time_rotating` 资源的超级能力是，当 `rotation_days` 期限相对于原始日期到期时，你会看到该资源触发替换：

```
    resource "time_rotating" "certificate" {
      rotation_days = 90
    }
```

这也要求你定期运行 Terraform 以保持未来的值。如果你使用了这样的资源，确保与变更管理流程进行协调，因为当你执行 `terraform plan` 时，它们可能会在不经意间出现在你面前，而你可能会发现自己已经错过了那个关键日期。

在本节中，我们学习了如何使用 `random` 提供者随机化资源的名称，甚至生成我们可以在 Terraform 在为其提供密码之前可以使用的密钥的部分自动化环境。我们还学习了在*第二章*中查看的 `timestamp()` 函数和其他创建时间段和旋转窗口的高级场景中何时使用 `time` 提供者。

接下来，我们将查看一些实用程序提供者，这些提供者帮助我们克服 Terraform 遇到不具备内置解决方案或已有提供者解决该问题的情况时的一些限制。

# 适应和集成

正如我们讨论过的，Terraform 及其提供者是开源项目，因此它可能存在其无法执行或无法获取的限制。因此，我们经常需要找到克服这些限制的方法，即使是暂时性的。在本节中，我们将查看几个提供者，这些提供者帮助 Terraform 扩展外部并利用可以增强 Terraform 并帮助其克服缺乏内置解决方案的情况的程序和系统。

## 访问外部资源

像许多实用程序提供者一样，`external` 提供者很小。它只有一个与提供者同名的数据源：`external`。顾名思义，此数据源允许您与第三方组件集成。它使您能够执行本地程序，传递其输入并处理输出。当您希望从外部源获取动态配置，对来自其他提供者的输入执行复杂转换，并集成您希望与 Terraform 集成的第三方工具时，此功能可能非常有利。

此提供者非常关注您指定的程序的运行时要求。首先，程序必须成功运行并以零退出码退出。如果程序返回非零代码，提供者将向 Terraform 发出警报并散播错误消息。其次，提供者期望输入和输出都以 JSON 格式提供。

当您的第三方程序满足所有这些要求时，使用 `external` 提供者非常完美。这确实是一个幸运的巧合！如果您正在集成这样的程序，那太棒了。但是，当您编写明确满足这些合同义务的自定义脚本或程序时，此提供者是正确的选择。

为了尽可能跨平台，编写这些自定义脚本的理想编程语言是 Python 或 Go。使用这些编程语言，您可以创建一个轻量级脚本，专门设计和构建，适合与您选择的外部系统进行通信，并提供 Terraform 友好的输出和错误处理：

```
    data "external" "example" {
      program = ["python", "${path.module}/example-data-source.py"]
      query = {
        # arbitrary map from strings to strings, passed
        # to the external program as the data query.
        id = "abc123"
      }
    }
```

在上面的代码中，我们正在本地机器上执行 Python 程序——这可能是我们的笔记本电脑，或者是我们**持续集成/持续交付**（**CI/CD**）管道的构建代理。

## 当你想从无到有地创建某样东西时

Terraform 及其提供者是开源项目。这意味着我们依赖于那些积极跟进平台和技术变化的友好互联网陌生人，他们在帮助我们实现自动化时提供支持。尽管 Terraform 对于广泛的公共云平台和技术有着出色的支持，但有时它还是需要一些帮助。可能有一些小特性没有得到支持，而你无法通过提供者中的资源原生配置。那些小小的调整有时在配置中发挥着关键作用，我们需要从其他可以通过 Terraform 提供者原生配置的资源中引入它们的依赖。这就是 `null_resource` 的作用所在。

### 空资源

`null_resource` 允许我们利用元参数如 `provisioner` 来执行本地和远程脚本执行。这使你能够执行一些关键的命令行脚本，这些脚本必须在 Terraform 继续其计划之前完成。因此，`null_resource` 没有像其他 Terraform 资源那样的属性。它唯一的属性是一个名为 `triggers` 的 `list(string)` 类型。当该数组中的任何字符串发生变化时，`null_resource` 就会被替换。这是一个重要的生命周期控制因素，在配置你附加的 `provisioner` 块时，你需要考虑这一点。

### 时间睡眠

还有一种什么都不做的技巧。在某些情况下，你试图触发的操作是非确定性的，这意味着你无法准确知道它何时完成。这可能是上下文之外的，或者是你使用的资源或适配器的真正技术限制。`time` 提供者提供了一个名为 `time_sleep` 的资源，可以让你创建一个睡眠定时器。你必须声明 `depends_on` 元参数，以确保在所需资源之间调用睡眠定时器：

```
    # This resource will destroy (potentially immediately) after null_resource.next
    resource "null_resource" "previous" {}
    resource "time_sleep" "wait_30_seconds" {
      depends_on = [null_resource.previous]
      create_duration = "30s"
    }
    # This resource will create (at least) 30 seconds after null_resource.previous
    resource "null_resource" "next" {
      depends_on = [time_sleep.wait_30_seconds]
    }
```

延迟 `destroy` 可以通过一个名为 `destroy_duration` 的不同属性来完成。

## 发起 HTTP 请求

有时，你可以不使用本地脚本或命令行工具，通过直接访问 REST API 端点来访问外部数据。当你希望从静态 HTTP 端点获取配置信息，访问管理在 Terraform 之外的资源的信息，或直接通过 REST API 与云提供商或外部服务集成，甚至将健康检查集成到 Terraform 流程中时，这种方法特别有利。

`http` 提供者提供了一个名为 `http` 的单一数据源，允许你进行 HTTP `GET` 操作。唯一必需的输入是 `url`，但你可以提供一些你期望在 HTTP 请求中设置的属性，比如 HTTP 请求头和正文内容：

```
    data "http" "foo" {
      url = "https://foo"
    }
```

在 Terraform 发出 HTTP 请求后，你可以访问 HTTP 响应状态码、头信息和正文内容。

在本节中，我们学习了如何与各种外部组件集成——本地程序或脚本、远程服务器或完全不使用任何组件。

接下来，我们将查看一些有助于我们创建和访问文件的实用程序提供者。我们已经遇到了一些能够启用这些场景的函数，但一些额外的场景只有在使用实用程序提供者时才可能。

# 文件系统

在使用 Terraform 构建基础设施即代码（IaC）解决方案时，有许多情况需要使用现有文件或创建新文件。在本节中，我们将查看使更高级场景成为可能的实用程序提供者，这些场景超出了 `file` 和 `templatefile` 函数的范围，以及何时应该使用提供者而不是函数。

## 读取和写入本地文件

有许多情况下，读取或写入文件可以大大简化 Terraform 如何与其他工具集成，例如通过创建配置文件、脚本或任何其他基础设施部署所需的工件。为了生成所需的输出，你可以在 HCL 代码中使用模板文件、输入变量或其他表达式来定义内容。

Terraform 有一个名为 `local` 的实用程序提供者，提供了此功能。该提供者有两个资源和两个数据源，分别名为 `local_file` 和 `local_sensitive_file`。

为什么不直接使用函数呢？函数不会参与依赖图，因此当你使用 `file` 或 `template_file` 函数时，不能与在 Terraform 操作过程中动态生成的文件一起使用。因此，如果你计划在 Terraform 中生成并使用文件，应该始终使用 `local` 提供者的资源。

### 写入文件

`local_file` 资源（以及相应的 `local_sensitive_file` 资源）允许你在 `filename` 属性中指定的目标位置创建一个新文件。有几种选项可以用于获取内容，可以通过使用 Terraform 内部动态生成的内容或现有文件来提供：

+   `content`：此属性允许你传递任何 UTF-8 编码的字符串，可以使用资源输出、局部变量或函数中的简单字符串来生成该字符串。

+   `content_base64`：此属性允许你传递作为 Base64 编码字符串的二进制数据。

+   `source`：此属性允许你传递一个现有文件的路径，用于读取文件内容。当使用此属性时，你将原始文件复制到新位置。

在以下代码中，我们将 `content` 属性设置为一个简单的常量字符串，并使用 `${path.module}` 特殊标记指定当前模块的工作目录作为名为 `foo.bar` 文件的输出位置：

```
    resource "local\_file" "foo" {
      content  = "foo!"
      filename = "${path.module}/foo.bar"
    }
```

在销毁时要小心，特别是当你在 Terraform 中动态生成文件内容时，使用`jsonencode`、`yamlencode`或任何其他方法时——你可能会遇到问题，因为 Terraform 处理这种资源类型依赖关系的方式。

同时，当将敏感数据写入文件时要小心，因为这可能会带来安全风险。同样，文件系统访问或 I/O 失败可能会为执行 Terraform 时创造额外的故障点。

鉴于这些常见的陷阱，使用 Terraform 生成文件内容仍然是非常有效的。一个这样的场景是为 Ansible 生成 YAML 清单，这是将 Terraform 和 Ansible 集成作为更广泛的维护过程的一部分的一个极好的方法，适用于需要操作系统级别配置管理变更的长期环境。

### 读取文件

`local_file`数据源（以及相应的`local_sensitive_file`数据源）允许你读取现有文件的内容，并以各种格式输出其内容，供你在代码库中的其他资源和模块中作为输入。这一功能与`file`函数类似，但提供了一些优势。

首先，它可以创建一个独立的块，从多个资源中引用，而不必在等效的函数调用中重复文件名的路径。创建一个对本地文件的集中引用可以使你的代码更易于维护，因为它让 Terraform 和人类对文件的依赖关系更加明确。

其次，通过利用数据源，你可以立即获得多种输出数据的方式，包括 Base64 字符串、SHA 和 MD5 选项。通过利用这些输出选项，你可以避免进行额外的嵌套函数调用，以执行相同的编码操作。

在以下代码中，我们通过`foo.bar`文件名访问模块当前目录中的现有文件：

```
    data "local_file" "foo" {
      filename = "${path.module}/foo.bar"
    }
```

然后，我们可以通过使用数据源的`content`输出或之前提到的其他编码选项来访问文件的原始内容。

## 模板文件和目录

在上一章中，我们介绍了模板文件，并调查了 HCL 内置函数，包括名为`template_file`的函数。当你处理单个文件时，应该使用这个函数，但`template`提供者提供了一个资源，你可以用它在给定目录中的所有文件上应用模板化。

该资源从`source_dir`属性获取输入文件，并且对于每个文件，它会将指定的输入变量替换为相应的占位符，将每个输出文件写入已建立的`destination_dir`。使用这个资源是更新跨多个文件的配置的一个好方法，但你必须确保所有文件都使用一致的占位符集，以便模板引擎以与`template_file()`函数相同的方式替换。

## 生成文件归档

有时，有必要将 Terraform 的输出打包成压缩归档文件，以便用于多个不同的目的，例如将配置高效地传输到应用部署管道的下一个阶段，将配置打包以便轻松分发到其他外部存储库，生成文档或其他工件以跟踪部署历史、环境变化或配置快照，以便随着时间的推移进行变化。

当处理机密或敏感数据时，应避免使用此方法，因为对于这些场景存在更合适的解决方案。与基础设施相关的元数据，由 Terraform 生成并需要由其他工具以不同格式使用，是这种方法的理想场景。

`archive_file` 资源在指定的 `output_path` 位置生成一个 ZIP 文件，并包含一个或多个文件，使用现有文件或动态生成的文件。

### 包含现有文件

当你想要引用存储在 Terraform 模块目录中的现有文件或 Terraform 使用 `local` 提供程序生成的文件时，应该使用 `source_file` 或 `source_dir` 来分别将单个文件或整个目录的文件包含到归档中：

```
    data "archive_file" "init" {
      type        = "zip"
      source_file = "${path.module}/foo.txt"
      output_path = "${path.module}/files/out.zip"
    }
```

### 包含动态生成的文件

当你想输出 Terraform 动态生成的内容，使用本地变量或其他方式构造对象时，需要使用一个或多个 `source` 块，并指定文件的 `content` 和 `filename`，将其包含到归档中：

```
    data "archive_file" "dotfiles" {
      type        = "zip"
      output_path = "${path.module}/files/out.zip"
      source {
        content  = "foobar"
        filename = "foo.txt"
      }
    }
```

如果你只想在归档中包含一个动态生成的文件，你可以使用顶级的 `source_content` 和 `source_filename` 来生成一个文件：

```
    data "archive_file" "dotfiles" {
      type        = "zip"
      output_path = "${path.module}/files/out.zip"
      source_content          = "foobar"
      source_content_filename = "foo.txt"
    }
```

前面的示例生成相同的输出，但后者在单文件归档场景中稍显简洁。同时，前者允许你在未来使用额外的 `source` 块向归档中添加任意数量的文件。

需要注意的是，将文件包含到归档中的每种方法是互斥的，因此必须选择一种方法，并且只能选择一种方法。

在本节中，我们学习了如何利用 `local`、`archive` 和 `template` 提供程序来处理更复杂的文件系统访问场景，在这些场景中，利用现有功能可能并不理想。

最后，我们将看看一些实用程序提供程序，帮助我们设置操作系统配置、安全性和网络访问控制。

# 操作系统和网络

HashiCorp 创建了 Terraform 的实用程序提供程序，以解决跨云平台的常见问题。因此，一些实用程序提供程序解决了与设置或连接到服务器时会遇到的常见架构场景相关的问题。

## 生成证书和 SSH 密钥

在 Terraform 中，`tls` 提供程序提供与 **传输层安全性** (**TLS**) 和加密相关的一般功能。你应该与证书颁发机构一起使用此提供程序来生成生产工作负载的签名证书。然而，许多功能在开发和实验环境中非常有用，可以简化工作效率。

### SSH 密钥

在处理虚拟机时，常见的用例是需要生成 `ssh-keygen`，但 Terraform 有一个提供此任务的资源，这使得将 SSH 密钥生成与将要使用它的基础设施封装在一起变得极其简单和方便。这是一个非常适合短期实验环境的工具，但在使用这种方法时，你需要保护你的 Terraform 状态。

```
    resource "tls_private_key" "ssh_key" {
      algorithm = "RSA"
      rsa_bits  = 4096
    }
```

现在 Terraform 已经生成了你的资源，你可以将其放入你选择的密钥管理器中：

```
    resource "azurerm_key_vault_secret" "ssh_private_key" {
      name         = "ssh-key"
      value        = tls_private_key.ssh_key.private_key_openssh
      key_vault_id = azurerm_key_vault.main.id
    }
```

在前面的示例中，我们将 `private_key_openssh` 值放入 Azure Key Vault 秘密中，这样我们就可以使用 Azure 门户中的 SSH 密钥直接连接到机器，或者使用 Azure Bastion。

`RSA` 是最常用的 SSH 密钥算法。它经过验证，但你应该使用至少 4,096 位的密钥大小。较新的算法，如 `ECDSA` 和 `ED25519` 也受支持，但在广泛应用于你的组织之前，你应确保你的客户端支持这些算法。

当你生成 SSH 密钥时，不需要将其保存到文件系统。你应该将其保存到一个证书管理服务中，如 **AWS 证书管理器** (**ACM**)、Azure Key Vault 或 Google Cloud 的证书管理器。

由于 Terraform 规划的工作方式，为了让 Terraform 知道 SSH 密钥资源是否存在，以及你的证书管理器是否有该资源，它需要在状态文件中维护关键属性。这些关键属性中有许多是高度敏感的信息，包括公钥和私钥。因此，保护 Terraform 状态的后端对于防止未授权访问至关重要。这个任务本身并不困难，但在你有一个安全的后端策略之前，可能最好不要在生产工作负载中使用这种方法。

### 证书

在生成证书时，通常需要生成一个 `PEM` 文件，并提供有关证书主题的详细信息，包括一些人类可读的信息，如组织名称、物理地址和网络地址信息，如域名或 IP 地址。

Terraform 有一个资源 `tls_cert_request`，可以生成 CSR。与生成 SSH 密钥的 `tls_private_key` 资源类似，该资源执行的是人类操作员通过图形界面或命令行界面生成 CSR 的任务。该资源的输出结果需要传递给 CA 以生成签名证书。

你可以使用`foo`资源在本地生成证书。CSR 提供程序将在运行 Terraform 的本地机器上尝试处理它。

首先，你需要一个私钥，可以使用我们用来生成 SSH 密钥的`tls_private_key`资源来创建：

```
    resource "tls_private_key" "foo" {
      algorithm = "RSA"
    }
```

然后，我们需要使用`tls_cert_request`资源生成 CSR：

```
    resource "tls_cert_request" "foo" {
      private_key_pem = tls_private_key.foo.private_key_pem
      subject {
        common_name  = "foo.com"
        organization = "Foobar, Inc"
      }
    }
```

最后，我们可以使用私钥和 CSR 生成证书：

```
    resource "tls_locally_signed_cert" "vault" {
      cert_request_pem = tls_cert_request.foo.cert_request_pem
      ca_key_algorithm   = tls_private_key.foo.algorithm
      ca_private_key_pem = tls_private_key.foo.private_key_pem
      ca_cert_pem        = tls_self_signed_cert.foo.cert_pem
      validity_period_hours = 17520
      allowed_uses = [
        "server_auth",
        "client_auth",
      ]
    }
```

## 生成 CloudInit 配置

**Cloud Init**是一个开源的跨平台工具，用于向云托管的虚拟机提供启动配置。它通过云的元数据和用户数据来配置新实例。例如，你可以设置主机名、设置用户和组、挂载和格式化磁盘、安装包以及运行自定义脚本。

### 基本使用方法

有时候，你可以使用数据源来生成内容，而不是与外部系统交互。`cloudinit_config`就是一个完美的例子。它通过一种模式帮助简化生成有时冗长的 Cloud-Init 配置文件，这些文件会作为输入传递给新创建的虚拟机。

Cloud-Init 支持几种不同的部分类型。每种类型都会推断出不同的模式和格式来传递内容，有时使用 JSON、YAML、bash 脚本或纯文本。Cloud-Init 能够做的全部内容超出了本书的范围，但我鼓励你查看在线文档以获取更多详情。我将介绍一些常见的用例，展示如何使用现有的 Cloud-Init 知识，并在使用`cloudinit` Terraform 提供程序时应用它：

```
    data "cloudinit_config" "foo" {
      gzip          = false
      base64_encode = false
    }
```

为了将输出作为用户数据附加到 AWS 上的新**弹性计算云**(**EC2**)实例，我们将使用以下代码：

```
    resource "aws_instance" "web" {
      # other ec2 attributes
      user_data = data.cloudinit_config.foo.rendered
    }
```

### 加载外部内容

当你有大量外部内容需要下载到实例时，可以使用`x-include-url`和`x-include-once-url`：

```
    data "cloudinit_config" "foo" {
      gzip          = false
      base64_encode = false
      part {
        content_type = "text/x-include-url"
        content      = "http://foo.com/bar.sh"
      }
    }
```

### 使用自定义脚本

当你想要执行存储在 Terraform 模块目录中的自定义脚本时，可以使用几种不同的部分类型在其他条件下运行这些脚本：

+   `x-shellscript`：这个脚本将在每次实例启动时运行。也就是说，无论是创建后的第一次启动还是随后的重启，它都会执行。

+   `x-shellscript-per-boot`：这与`x-shellscript`相同。它也将在每次实例启动时运行。

+   `x-shellscript-per-instance`：这个脚本将只在每个实例上运行一次。也就是说，脚本将在实例创建后的第一次启动时运行，但在随后的重启中不会再运行。这部分对于只需要每个实例做一次的初始化任务非常有用，例如设置跨重启持续的软件下载或用户。

+   `x-shellscript-per-once`：这个脚本将只在所有实例中运行一次。如果你创建多个带有相同脚本的实例，这个脚本只会在第一个启动的实例上运行。这个功能对于只需要在一组实例中执行一次的任务很有用，例如设置数据库或集群中的主节点。

考虑以下脚本，它存储在 Terraform 模块根文件夹中的一个名为`foo.sh`的 bash 脚本文件里：

```
    #!/bin/bash
    sudo apt-get update -y
    sudo apt-get install nginx -y
    echo '<h1>Hello from Terraform Cloud-Init!</h1>' | sudo tee /var/www/html/index.html
```

我们可以将其嵌入到`cloudinit_config`数据源中，以生成我们传递给新创建虚拟机的用户数据：

```
    data "cloudinit_config" "foo" {
      gzip          = false
      base64_encode = false
      part {
        content_type = "text/x-shellscript"
        content      = file("${path.module}/foo.sh")
      }
    }
```

### Cloud 配置文件

Cloud-Init 支持一种自定义模式，用于执行各种日常任务。支持多种不同的部分类型，允许你在 Cloud-Init 包中以多种格式包含 cloud config 数据：

+   `cloud-config`：这是最常用的内容类型，用于标准的 cloud-init YAML 配置文件。你可以使用`cloud-config`内容类型来执行通用实例配置任务，例如设置用户和组、管理软件包、运行命令和写入文件。

+   `cloud-config-archive`：这种内容类型在一个文件中提供多个 cloud-config 部分。`cloud-config-archive`文件是一个 YAML 文件，包含一个 cloud-config 部分的列表，每个部分是一个包含文件名、内容类型和内容本身的映射。你应该在按特定顺序应用多个 cloud-config 文件时使用它。它们在列表中的顺序决定了应用的顺序。

+   `cloud-config-jsonp`：这种内容类型允许你编写**带填充的 JSON**（**JSONP**）响应。JSONP 通常用于绕过网页浏览器的跨域策略。如果你在编写一个需要与不同域上的服务器交互并使用 JSONP 来绕过同源策略的 Web 应用程序，你可能会使用这种内容类型。

cloud config 的完整功能超出了本书的范围，但我鼓励你通过在线文档更详细地探索它们。以下配置演示了我们如何使用`cloud init`来生成一个用户、一个组、将用户分配到该组，并为机器设置权限和配置：

```
    #cloud-config
    groups:
      - bar
    users:
      - name: foo
        groups: sudo, bar
        shell: /bin/bash
        sudo: ['ALL=(ALL) NOPASSWD:ALL']
        ssh_authorized_keys:
          - ssh-rsa your-public-key
```

我们可以将其嵌入到`cloudinit_config`数据源中，以生成我们传递给新创建虚拟机的用户数据：

```
    data "cloudinit_config" "foo" {
      gzip          = false
      base64_encode = false
      part {
        content_type = "text/cloud-config"
        content      = file("${path.module}/users.yaml")
      }
    }
```

## 配置 DNS 记录

使用自动化管理**域名系统**（**DNS**）对于某些发布策略（如蓝绿部署）至关重要。Terraform 提供了一个可扩展的框架，可以轻松处理这种重要配置。

虽然大多数云服务提供商都提供自己的 DNS 服务，从 Amazon 的 Route 53 到 Azure 的私有 DNS 区域，通常都有针对你选择的云的第一方 DNS 管理解决方案。许多第三方公共 DNS 注册商提供 DNS 服务，如 Cloudflare、Akamai、GoDaddy 和 DynDNS。

然而，由于 Terraform 提供了如此可扩展的基础，管理 DNS 并不仅限于通过各自的提供商管理公共云平台。你还可以管理本地 DNS 服务器或任何在你选择的公共或私有云中使用基础设施即服务构建的自定义 DNS 基础设施。

您可以使用支持秘密密钥（`RFC 2845`）或 GSS-TSIG（`RFC 3645`）身份验证方法的任何 DNS 服务器与`DNS`提供商配合使用。

# 摘要

在本章中，我们深入探讨了 HashiCorp 构建的实用提供商，帮助我们增强我们的 IaC 解决方案。我们学习了如何与外部系统集成，如何处理存储在文件系统上的资产，以及如何在 Terraform 中随机化和填充空白。这些 Terraform 提供商非常多才多艺，随着你深入探索，你会发现它们越来越有价值。

如同使用其他提供商一样，始终在`required_providers`块中明确指定版本号来引用提供商。不要隐式使用最新版本。除此之外，这些提供商的资源可以嵌入到任何 Terraform 模块中。在设计可重用模块时，需要特别注意，确保上游客户端模块对提供商的要求尽可能少，因为如果模块需要声明多个不同的提供商，这可能会增加希望重用该模块的用户的复杂度！

在下一章中，我们将建立一些可以应用于我们的 IaC 的架构概念，无论我们瞄准的是哪个云平台。毕竟，我们的 IaC 的好坏取决于它所定义的架构。因此，我们必须充分理解云服务解剖的典型元素，以及虚拟机和容器等不同计算范式的机制。接下来，我们将通过多云视角来审视虚拟机架构。

# 第二部分：云架构与自动化概念

如果没有扎实的云架构和软件开发流程的基础，进入基础设施即代码（IaC）的旅程将是徒劳的。幸运的是，许多这些概念跨越了云平台，一旦你理解了关键概念，就能将这些知识应用到你选择的云平台中——无论是 AWS、Azure 还是 GCP。

本部分包括以下章节：

+   *第四章*，*云架构基础——虚拟机与基础设施即服务*

+   *第五章*，*超越虚拟机——容器与 Kubernetes 的核心概念*

+   *第六章*，*将一切连接在一起——GitFlow、GitOps 与* *CI/CD*

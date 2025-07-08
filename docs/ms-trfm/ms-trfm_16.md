

# 第十六章：已经配置过？导入现有环境的策略

本书前九章主要集中于如何使用多种云计算范式，在多个云平台上实现新的云架构。现在，我们将稍微调整一下方向，重点讨论如何处理现有环境。不幸的是，有时（实际上是很多时候），你所管理的云环境并不是最初通过**基础设施即代码**（**IaC**）使用 Terraform 进行配置的。它们可能是通过其他工具配置的，甚至是手动配置的，现在你正试图使用 Terraform 整合你的云操作。

本章涵盖以下主题：

+   导入单个资源

+   识别需要导入的资源

+   导入现有环境

+   最佳实践

# 导入单个资源

Terraform 支持两种导入资源到状态中的方式。一种方式本质上是命令式和程序化的，通常在 GitOps 流程之外，使用 Terraform 的**命令行界面**（**CLI**）执行。还有另一种更新的方式，允许我们在代码中声明导入操作，并按照标准的 GitFlow 流程将这些更改推送到生产环境。

## 导入命令

`import`命令允许你导入一个已经通过其他方式（而非 Terraform）配置好的现有资源：

```
terraform import [options] ADDRESS ID
```

`Terraform import`命令（[`developer.hashicorp.com/terraform/cli/commands/import`](https://developer.hashicorp.com/terraform/cli/commands/import)）有两个关键参数。各种选项超出了本书的范围。我建议你查阅文档，了解所有可用选项的详细信息。

第一个参数，即 Terraform 代码库中资源的地址，非常关键。它是我们用来访问 Terraform 工作区中资源的相同引用。与我们在 HashiCorp 配置语言代码库中工作时不同，我们不再受到当前 Terraform 模块范围的限制。该地址遵循你的 Terraform 提供者的命名约定。例如，你需要资源类型和对象引用才能导入一个虚拟机。

第二个参数是资源在目标云平台上的唯一标识符。不同云平台之间，这个唯一标识符会有很大不同。在接下来的部分中，我们将看到每个云平台之间的差异。

`import`命令非常适合用于在`terraform apply`过程中由于临时问题失败的单个资源。如果你需要导入整个解决方案，那么为每个资源编写一个`import`命令将会非常繁琐。即使是一个简单的虚拟机，也可能包含十多个资源。

## 导入块

`import`命令很有用且可用，但它需要通过命令行通过人工操作来对你的 IaC 代码库进行更改。导入块是在 Terraform 1.5.0 版本中引入的，它允许通过源代码更改来完成这些更改，这对于维持 GitFlow 流程至关重要。这反过来是 GitOps 模型的关键组成部分。

与使用 Terraform CLI 执行命令不同，你需要在代码库中嵌入一个像这样的导入块：

```
import {
  to = ADDRESS
  id = ID
}
```

它看起来与`import`命令的参数非常相似，但它利用你执行 Terraform 时所处的现有上下文。它还使用 HashiCorp 配置语言（HCL）来定义导入操作。

这种技术不仅允许我们将状态管理操作作为 GitOps 流程的一部分进行，还简化了这个过程。导入资源只需要两个拉取请求：第一个是引入我们希望导入的资源的导入块，第二个是在成功执行`Terraform Apply`后删除导入块，当资源被导入到 Terraform 状态中时。

## 导入多个资源

`Import`命令和导入块支持使用`for_each`和`count`元参数导入资源。

要导入通过`for_each`块配置的资源，你只需要定义一个包含你希望导入的资源唯一标识符的`map`：

```
locals {
  resources = {
    "zone1" = "ID-for-zone1"
    "zone2" = "ID-for-zone2"
  }
}
```

导入块的唯一标识符将来自你定义的`map`。然后，在导入块中使用匹配的`for_each`，该`for_each`引用与你的资源块相同的`map`，并通过`each.key`引用相应的资源：

```
import {
  for_each = local.resources
  to = ADDRESS[each.key]
  id = each.value
}
```

同样，当导入通过`count`元参数配置的资源时，我们必须声明一个包含唯一标识符的数组：

```
locals { resources = [ "ID-for-zone1", "ID-for-zone2" ] }
```

最后，我们可以在导入块上使用`count`元参数，并像处理资源块一样进行迭代：

```
import {
  count = length(local.resources)
  to = ADDRESS[count.index]
  id = local.resources[count.index]
}
```

使用`import`命令稍微复杂一些。你需要为`map`中的每个项目执行`terraform import`命令，引用正确的`key`并将其映射到相应的值：

```
terraform import 'ADDRESS["key"]' ID
```

使用类似的技巧导入通过`count`配置的资源：

```
terraform import 'ADDRESS[index]' ID
```

当处理`for_each`配置的资源时，我们需要为数组中的每个项目执行`terraform import`命令，并手动将索引与正确的唯一标识符关联。

尽管通过一些高级的 bash 脚本技术可以实现，但推荐的方法是使用 HashiCorp 配置语言中的导入块，因为这种方法更容易实现且更不容易出错。

我们已经分别研究了通过 `import` 命令和导入块将现有资源导入 Terraform 的命令式和声明式方法。现在，让我们研究如何在本书中覆盖的三个云平台——**Amazon Web Services**（**AWS**）、Microsoft Azure 和 Google Cloud Platform——中识别每个现有资源的正确唯一标识符。

# 确定要导入的资源

就像在本书前几章中我们开发的每种云架构之间存在细微差别一样，现有资源导入到 Terraform 的方式也受到了云平台之间结构性和不太明显差异的影响。

## AWS

AWS 对 EC2 实例的命名约定通常是这样的：`i-abcd1234`。它通常由两个组件组成：前缀和标识符，前缀在不同的 AWS 服务之间有所不同。

`i-` 前缀表示这是一个 `vol-`（卷）或 `sg-`（安全组）。

在这种情况下，`abcd1234` 标识符是实例的唯一标识符。AWS 通常为每个实例分配一个十六进制字符串，以区分它与其他资源。这种命名约定帮助用户和 AWS 服务在 AWS 生态系统内识别和引用资源。你需要识别正确的唯一标识符，以便将资源从 AWS 及其他云平台导入 Terraform。

使用 AWS 的导入命令时，它看起来是这样的：

```
terraform import aws_instance.foo i-abcd1234
```

相应的导入块看起来像这样：

```
import {
  to = aws_instance.foo
  id = i-abcd1234
}
```

理解地址和唯一标识符之间的区别非常重要。地址是 Terraform 内部的对象引用，而唯一标识符是目标云平台上资源的外部引用。这一理解将帮助你更有效地导航导入过程。

## Azure

在 Azure 中，唯一标识符称为 **Azure 资源 ID**。它采用一种完全不同的格式，使用 Azure 中云资源位置的多个不同地标来构建。它遵循一个结构化的格式，包含几个组件：订阅、资源组、资源提供者、资源类型和本地化的资源名称：

```
/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{resource-provider}/{resource-type}/{resource-name}
```

例如，Azure 虚拟机的 Azure 资源 ID 看起来像这样：

```
/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-foo/providers/Microsoft.Compute/virtualMachines/vmfoo001
```

在此示例中，我们可以看到资源 ID 路径中每个组件的具体值：

+   `00000000-0000-0000-0000-000000000000` 订阅的 GUID。

+   `rg-foo`。

+   `Microsoft.Compute` 是 Azure 计算服务的资源提供者，其中包括 Azure 虚拟机。

+   `virtualMachines` 用于 Azure 虚拟机。资源提供者与资源类型结合，创建一个完全限定的 Azure 资源类型：`Microsoft.Compute\virtualMachines`。

+   `vmfoo001`。

每个资源提供者中的资源类型也有子类型。这些子类型通过额外的斜杠分隔（例如虚拟机扩展：`Microsoft.Compute/virtualMachines/{vm-name}/extensions/{extension-name}`）。Azure 的资源 ID 命名约定使用了资源路径策略，而不是像 AWS 那样使用前缀和唯一标识符策略。因此，Azure 的资源 ID 可能会相当长，但它们确实有一种合理的方式来解构，从而收集关于特定资源部署上下文的有价值信息，避免了额外的查找。

当在 Azure 上使用导入命令时，命令如下所示：

```
terraform import azurerm_linux_virtual_machine.foo "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-foo/providers/Microsoft.Compute/virtualMachines/vmfoo001"
```

对应的导入块如下所示：

```
import {
  to = azurerm_linux_virtual_machine.foo
  id = "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-foo/providers/Microsoft.Compute/virtualMachines/vmfoo001"
}
```

重要的是要记住，地址是 Terraform 中的内部对象引用。唯一标识符是指向目标云平台上资源的外部引用。

## Google Cloud Platform

在 Google Cloud 中，资源的唯一标识符称为 **资源路径**，就像在 Azure 中一样，它由 Google Cloud 中资源位置的一些重要标志组成。这些标志与 Azure 的不同，因为两个平台的结构差异以及其他设计考虑：

```
projects/{{project}}/zones/{{zone}}/instances/{{name}}
```

例如，Google 计算实例的 Google 资源路径如下所示：

```
projects/proj-foo/zones/us-central1-a/instances/vmfoo001
```

在这个例子中，我们看到了资源路径中每个组件的具体值：

+   `proj-foo`。

+   **区域**：这表示资源在 Google Cloud 区域和可用区内的物理位置。

+   `vmfoo001`。

尽管 Google Cloud 确实具有更高层次的组织结构，如 Google Cloud 组织及其下的文件夹，但资源路径只包括 Google Cloud 项目 ID。这类似于 Azure 的资源 ID，它包括 Azure 订阅和资源组，因为这些是平台中资源的逻辑容器。Google 选择了更简化的路径，只包含项目 ID。Google Cloud 资源路径与 Azure 资源 ID 之间的一个主要区别是，Google 资源路径中包括了区域信息。区域表示资源在 Google Cloud 区域中的物理位置。Azure 的资源 ID 仅包括逻辑结构，如订阅、资源组、资源提供者和类型，不包括诸如 Azure 区域或可用区这样的物理位置。

当在 Google Cloud Platform 上使用导入命令时，命令如下所示：

```
terraform import google_compute_instance.foo "projects/proj-foo/zones/us-central1-a/instances/vmfoo001"
```

对应的导入块如下所示：

```
import {
  to = google_compute_instance.foo
  id = "projects/proj-foo/zones/us-central1-a/instances/vmfoo001"
}
```

重要的是要记住，地址是 Terraform 中的内部对象引用，唯一标识符是指向目标云平台上资源的外部引用。

现在，我们已经了解了一些关于如何识别现有资源以及需要映射到我们代码库中定义的资源的唯一标识符的信息，我们完全准备好开始手动导入资源到我们的 Terraform 代码中。然而，这真的是唯一的方法吗？是否有一种更具成本效益或时间敏感性的方法，能够让我们批量导入资源？在下一部分中，我们将探索一些工具，帮助我们找到并导入现有资源，并生成相应的 Terraform 代码。

# 导入现有环境

正如我们在本章前面的部分所看到的，Terraform 包含了广泛的导入机制，这些机制允许我们将单个资源和大量现有资源导入到我们的 Terraform 代码库中。这些工具可以帮助我们克服暂时性错误，避免产生需要通过现有 Terraform 代码库和 Terraform 状态文件管理的孤立资源。

然而，当我们没有编写任何 Terraform 代码，并且许多现有资源已经在云环境中配置好时，如何处理呢？从头开始手动反向工程所有 Terraform 代码似乎不是一个有用的时间投入方式。这就是为什么会有工具来帮助自动化这个过程！

在本节中，我们将检查解决这个问题的几个最流行的开源工具。

## Terraformer

**Terraformer** 是由 Google 开发的一个开源工具，帮助将现有的云基础设施导入到 Terraform 配置和状态中。它支持多种云服务提供商，包括我们在本书中探索过的那些。当然，Google Cloud 得到了很好的支持，包括它的主要竞争对手（AWS 和 Azure），但许多其他 Terraform 提供商也得到了支持。与 Terraform 内建功能不同，这个工具的设计目的是根据分布在云环境中的现有资源生成 Terraform 代码和状态。

这个工具以及类似的工具，通过利用云提供商的 REST API 来收集有关已配置的各种资源的信息。你只需要将其指向正确的方向，并为其设置一些限制，以便缩小其视野范围。你只需选择想要打包在同一个 Terraform 工作区和状态文件中的资源。

允许你将 Terraformer 限定为仅处理你感兴趣的资源的关键命令行参数包括资源类型、区域和标签。根据提供商的不同，可能会存在资源类型支持的限制，因此最好使用以下命令检查当前支持的资源列表：

```
terraformer list --provider=aws
```

这将帮助你了解如何查询特定的云平台。例如，在从 AWS 导入资源时，我们可以确定 `s3` 和 `ec2_instance` 是支持的资源类型：

```
terraformer import aws --resources=s3,ec2_instance --regions=us-west-1
```

在 Azure 上，我们将使用 Azure 特定的资源类型，并经常使用`--resource-group`参数来指定这种 Azure 特定的逻辑结构以导入资源：

```
terraformer import azure --resources=resource_group,vm --resource-group=your-resource-group
```

同样，在 Google Cloud 上，我们将使用 Google Cloud 项目，这是对应于 Azure 资源组的逻辑结构，以缩小领域：

```
terraformer import google --resources=gcs,compute_instance --projects=your-project-id --regions=your-region
```

标签起着重要作用，因为它们提供了一种非常精细的方式来将我们想要的内容导入到我们的 Terraform 工作空间中：

```
terraformer import google --filter="Name=tags.Environment;Value=Production"
```

我们可以指定一组非常具体的标签，这些标签我们预先在我们的环境中设置，以在导入过程中获得最高效率。

## Azure 导出工具

有其他商业和平台特定的工具，可能比像 Google 的 Terraformer 这样的通用工具做得更好。其中一个例子是`azurerm`提供者和`azapi`提供者，这两个 Terraform 提供者可以用来配置和管理 Azure 资源。

与 Terraformer 一样，Azure 导出工具有几种查询现有资源的机制，应包括在代码生成过程中。它支持其他导入选项，如全订阅导入，并消除了指定资源类型的需求。这可以通过使用`azurerm`和`azapi`提供者的组合来加快 Azure 代码生成过程。由于`azapi`提供者支持每个 Azure 资源的完全支持，因此在`azurerm`资源不可用时，它可以作为多功能填充使用，而不会出现基于资源类型的兼容性问题。

导入给定 Azure 资源组内所有资源的命令将简单地如下所示：

```
aztfexport resource-group rg-foo
```

它可以在交互模式或非交互模式下运行。交互模式允许最终用户查看将被导入并映射到其相应引用的资源在 Terraform 代码中的资源。

尽管 Azure 导出工具不像 Terraformer 项目那样广为人知，但它确实在 Azure 和更广泛的 Terraform 社区中有一些有趣的功能。例如，**附加**功能允许您执行有针对性的代码生成，并将现有资源附加到现有的 Terraform 工作空间中。

## 限制

一个高效的基础设施即代码（IaC）和 Terraform 代码生成工具的吸引力确实存在。但是，在涉足这一领域时，你应该意识到它并非没有局限性和常见陷阱。

Terraform 的代码生成工具面临的最大挑战，并非 Terraform 和 IaC 领域独有的问题，而是普遍存在于逆向工程或代码生成方法中的问题。使用逆向工程工具生成的代码通常缺乏手写代码从一开始就融入的工艺感。这不仅可能导致功能缺陷需要排查，还可能出现无数的代码质量和可读性问题，这些问题必须在代码库真正用于其预期目的（即通过 IaC 维护云环境）之前解决。

在导入的 Terraform 代码库中，常见的一个功能性问题是过度定义的显式依赖关系，使用了 `depends_on` 元数据参数。`depends_on` 子句是解决 Terraform 无法自动识别的资源之间隐式依赖关系的有价值工具。然而，在大多数情况下，显式定义这些资源之间的依赖关系是没有必要的，会增加代码库的冗余和复杂性，还会影响代码的可读性。

另一个例子是，当从云平台提取资源配置时，其值通常作为硬编码值导入，这些值分散在所有声明的资源中。这会立即产生技术债务，需对相关常量值进行合理化处理，并提取一组合理且理想的输入变量，以便用于定义相关的配置设置。

最后，Terraform 资源上常常存在仅可写属性，这些属性不会通过云平台的 REST API 返回，因为它们包含敏感或机密信息。出于保护机密信息泄露的设计考虑，这是正常现象。如果该资源最初是通过 Terraform 配置的，这就不是问题，因为这些敏感值会存储在状态文件中。然而，这会带来一定的重构过程，因为在大多数情况下，你的 Terraform 代码库将无法通过 `terraform validate`，更不用说 `terraform plan`，因为会有需要解决的错误。

在生成代码并导入资源后立即运行 `plan`，有助于发现导入过程中可能存在的细微差异和不规则之处。这是有可能发生的，因为 Terraform 代码生成的准确度远未达到 100%。

如我们所见，在工具领域中，有一些相当不错的选项可以自动化大规模云资源的代码生成和 Terraform 状态文件创建，包括我们在本书中重点讨论的三大云平台。然而，虽然代码生成可以加快过程中的某些部分，但它也可能带来需要解决的挑战。在下一节中，我们将权衡取舍，并讨论一些最佳实践和替代方案，用于将现有环境纳入 Terraform 管理。

# 最佳实践

我们已经看过 Terraform 内置的能力，能够导入单个资源，并且了解了如何识别在不同云平台上要导入的现有资源。我们认识到内置功能的一些局限性，并且看了 3rd party 替代方案，这些替代方案提供了批量导入整个环境的选项，以及这些选项的当前局限性。现在，我们将探讨最佳实践，如何以及何时使用这些不同的方法将现有资源和环境导入并纳入 Terraform 管理。

## 爆炸半径

在导入现有资源并将其纳入 Terraform 管理时，仔细思考这些资源的组织方式以及你希望如何将它们划分为长期有效的 IaC 解决方案是很重要的。这就是最小化**爆炸半径**的设计原则。当我们导入资源时，实际上是在确定我们根模块或 Terraform 工作空间的边界。

这是进行此设计的理想时机，因为工作空间尚未组织好。考虑清楚这一点很重要，因为它会影响你如何管理、更新和复制基础架构的部分内容，这取决于你如何将资源分组。

你应该考虑资源将要发挥的功能以及谁负责管理它们。假设一个中央团队负责维护架构中的某个部分，那么你可能希望考虑将这些资源组织到同一个 Terraform 工作空间内，以便更容易控制访问并减少团队之间的摩擦。

在使用 Terraformer 或其他工具在 Terraform 工作空间中生成代码时，使用标签来缩小资源筛选范围。为云资源预先添加合适用途的标签，将有助于你最大化使用 Terraform 导入工具的效果。这一点在 AWS 中尤为重要，因为与 Azure 和 Google Cloud 上的资源组和项目不同，AWS 缺乏类似的资源逻辑容器。

## 有时，慢慢移动就是快速前进

作为使用导入工具批量导入资源的替代方法，你可以使用一种轻量级的技术，利用内建的导入工具。这个过程有点繁琐，但有时缓慢推进让你能够更加有目的和深思熟虑。这一过程仅仅涉及使用查询技术来识别你计划导入的资源，然后使用最基本的 Terraform 资源定义将它们搭建起来。这个资源定义只是一个占位符，很可能与之前配置的资源的配置不匹配——但这并不是重点。

关键在于将现有资源导入状态，然后运行`terraform plan`来确定配置差异。接着，你可以使用生成的计划来调整资源定义的配置，以匹配计划输出，直到不再需要任何更改。

采用这种方法，你是在采取与批量工具导入相反的策略。与其挥舞砍刀穿越丛林，不如挥舞手术刀，做出极其细致的切割。你需要手动配置它，但这将让你对所导入并纳入管理的组件有一个更系统化、一步步的理解。这种更深入的理解可以帮助你识别出在批量导入过程中可能被忽略的依赖关系和设计缺陷。

## 蓝绿部署

另一种选择是考虑替代导入的方法。导入是一个繁琐且容易出错的过程。如果你有一些手动配置的关键基础设施，你可能想要考虑用已经由 Terraform 管理的新环境替换它们。

这种方法被称为**蓝绿部署**。这是一种广为人知的发布管理策略，通过构建一个新的**绿**环境来替换现有的**蓝**环境。在绿环境经过全面测试并准备就绪后，我们执行切换操作，从蓝环境切换到绿环境。

你可以设置新环境，并将工作负载和应用程序迁移到其中。这将允许你在手动配置且没有适当治理的环境与遵循最佳实践的环境之间实现清晰的分离。你可以逐步地、一步一步地将工作负载迁移到新的有序环境，直到旧环境完全关闭。

使用代码生成器可能会产生质量极差的代码，需要大量重构。虽然其中一些只是简单的输入变量提取，但随着环境复杂性的增加，将资源移入模块将变得异常繁琐。执行导入、重构和转换的工作量实际上可能比从零开始编写代码并逐步切换还要大。

当你权衡将遗留环境置于“保持灯火通明”模式的成本时，同时构建新的世界秩序，这样可以让你的组织在保持一定常态的同时，逐渐适应使用 IaC 管理环境的变化，而不是一蹴而就。

在本节中，我们讨论了一些在 Terraform 管理下导入现有资源和环境的重要经验法则。如果你计划进行批量导入，首先要认识到你将使用的工具的局限性，并为重构预留充足的时间。最重要的是，通过定义一个集中的爆炸半径来缩小关注范围，围绕你的部署进行管理。

如果翻遍一大堆杂乱的代码并通过广泛重构清理它们听起来不太符合你的兴趣，可以考虑通过高度集中的逐步导入过程慢慢重建环境，或者直接进行蓝绿部署。我的首选方法是蓝绿部署，但你必须仔细评估对生产环境的影响，以确定这是否是最适合你的选择。

# 摘要

在本章中，我们探讨了 Terraform 内置的现有资源导入 Terraform 状态的能力，采用命令式和声明式的方法。尽管内置的导入功能缺乏任何形式的代码生成，但我们也介绍了一些开源工具，这些工具分析现有环境并生成 HashiCorp 配置语言（HCL）代码来管理资源，并将其导入到状态中。我们讨论了这些不同导入技术的相关权衡以及何时考虑使用它们，这将帮助你为组织和团队决定最佳的行动方案。在下一章中，我们将讨论如何使用 Terraform 管理和操作现有环境。

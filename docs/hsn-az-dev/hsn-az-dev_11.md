# 使用 Azure 存储 - 表格、队列、文件和 Blob

Azure 中的 PaaS 不仅仅是关于应用服务或容器。这个云平台提供了更多功能，尤其是在存储、消息传递解决方案或监控方面。通过 Event Hub、Azure 存储或 Application Insights 等服务，我们得到了一个完整的云组件集，它们提供了极大的灵活性，并简化了开发完整、可扩展且易于维护的应用程序。

本章将涵盖以下主题：

+   使用 Azure 存储解决方案

+   使用 Azure 存储表格存储结构化数据

+   实现 Azure 存储文件的完全托管文件共享

+   使用 Azure 存储队列的队列

+   使用 Azure 存储 Blob 进行对象存储

# 技术要求

要执行本章的练习，您需要以下内容：

+   一个 Azure 订阅

+   Visual Studio 2017

+   Azure 存储资源管理器，网址：[`azure.microsoft.com/en-us/features/storage-explorer/`](https://azure.microsoft.com/en-us/features/storage-explorer/)

+   Azure 存储模拟器，网址：[`azure.microsoft.com/en-us/downloads/`](https://azure.microsoft.com/en-us/downloads/)

# 在解决方案中使用 Azure 存储

大多数应用程序在没有存储解决方案的情况下无法工作。存储解决方案可以是任何类型的数据库——关系型、文档型、文件型或图形型。大多数情况下，它们都需要一些技能来配置并开始使用它们。目前，我们已经介绍了 Azure 中的一种存储解决方案，即 Azure Cosmos DB，它是一个无服务器数据库，唯一需要做的就是设置正确的吞吐量值。当然，Azure 提供了更多存储服务，其中最常见的是 Azure 存储。它是一个 PaaS 云组件（虽然有些人将其定义为无服务器，主要是因为没有服务器），可以通过四种不同的方式使用。在本章中，我们将覆盖所有这些方式，帮助您熟悉它们的功能和特点。

# 不同的 Azure 存储服务

Azure 存储由四个不同的服务组成：

+   表格存储

+   队列存储

+   Blob 存储

+   Azure 文件

它们各自有不同的用途，提供不同的功能和限制。虽然它们的名称直观易懂，但您会发现每个服务都是完全不同的，虽然它们可以互相配合使用，但需要不同的技能才能高效地做到这一点，并且必须遵循最佳实践。此外，Azure 存储还提供了一个额外的服务——磁盘存储，这是虚拟机使用的功能。由于这个原因，本书将不涉及此部分内容。不过，您可以在*进一步阅读*部分找到相关文档的链接。

# 不同类型的存储账户

Azure 存储提供三种不同类型的存储账户：

+   **通用标准型**：支持表格、Blob、文件和队列，且支持三种不同类型的 Blob：块 Blob、页 Blob 和追加 Blob

+   **通用高级**：仅限于块存储，支持页块存储

+   **支持热/冷访问层的 Blob 存储**：仅限于块存储，并支持块存储和追加块

你将在接下来的章节中学习更多关于不同种类块存储的内容。目前的问题是：标准帐户和高级帐户之间的区别是什么——当然，除了定价之外？你可以这样定义它们：

+   标准：最常见的选择，具有合理的性能并支持所有类型的数据。这些帐户使用磁盘存储。

+   高级：性能更好的帐户，得益于使用 SSD 硬盘——推荐用于虚拟机以及当你需要快速访问存储在其中的数据时。

如果你想比较两种帐户的性能，以下是 128 GB 磁盘的对比：

+   标准：500 I/O 操作/秒，吞吐量 50 MB/秒，月费 €3,78

+   高级：500 I/O 操作/秒，吞吐量 100 MB/秒，月费 €15,12

所以，如你所见，高级选项的吞吐量大约是标准选项的两倍。

# 保护 Azure 存储

一般来说，有两种方法可以保护对存储帐户的访问：

+   Azure AD 与 RBAC

+   SAS 令牌

此外，块存储可以公开访问（当然，前提是你决定这么做）。根据你的需求，某个选项可能更符合你的要求——这当然取决于你的应用程序特点。以下是这两种保护 Azure 存储的方法的区别：

+   **RBAC**：此方法用于保护对帐户的管理操作。你可以限制对服务特定功能的访问，仅限于 Azure AD 中定义的特定组。然而，你无法使用此方法保护块存储或表存储（尽管你可以通过保护对 SAS 令牌的访问间接做到这一点）。

+   **SAS 令牌**：这些是长字符串，存储描述访问资源的不同参数。它们指定服务类型、权限和令牌的有效期，或限制对 IP 地址的访问。

以下是一个 SAS 令牌的示例：

```
https://myaccount.blob.core.windows.net/securecontainer/blob.txt?sv=2015-04-05&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D
```

如你所见，它限制了对存储在 `securecontainer` 容器中的 `blob.txt` 文件的访问。它定义了诸如服务版本（`sv`）、过期时间（`se`）或令牌的实际签名（`sig`）等参数。一般来说，使用 SAS 令牌，你可以限制对帐户或服务的访问（因此，也可以限制对表存储中一系列实体的访问）。

# 复制

使用云服务时，你必须预期任何服务都可能随时宕机。尽管 Azure 存储被认为是最耐用的服务之一（因为许多 Azure 服务依赖于它），但它仍然可能会发生故障。为了缓解与此类故障相关的问题，它提供了四种不同的复制方式：

+   **本地冗余存储 (LRS)**：在同一数据中心内有三份数据副本

+   **区域冗余存储 (ZRS)**：在同一区域内有三份数据副本

+   **地理冗余存储（GRS）**：在同一数据中心内存储三份数据，再加上在另一区域内存储三份数据

+   **读取访问地理冗余存储（RA-GRS）**：在同一数据中心内存储三份数据，再加上在另一区域内存储三份数据，并且可以从该区域进行读取

在使用 Azure Storage 构建应用时，你必须仔细设计其可用性要求。根据你的期望，可能需要选择不同的模型。

当使用将数据复制到另一个数据中心的模型时（基本上是 GRS 和 RA-GRS），需要考虑不同区域之间传输数据的成本。

你可能会想知道 LRS 与其他复制模型相比有多耐用。要了解这一点，你需要理解数据是如何在单个数据中心内存储的。实际上，Azure Storage 的磁盘被安装在机架内，这些机架构成了一个更大的概念，叫做 stamp。Stamps 被配置成使用不同的电源线和网络，正因为如此的设置，数据可以存储在不同的故障域中，确保如果一个故障，其他两个仍然能工作。微软表示，LRS 的设计目标是提供至少 99.999999999%的耐用性。如果这还不够，你可以考虑其他模型。

使用 RA-GRS 时，不要轻视如果发生故障时轻松写入备用区域的能力。启动故障转移是微软的责任（与 Azure Cosmos DB 不同，后者由你来决定），因此**恢复时间目标**（**RTO**）包括微软做出决策的时间和更改 DNS 条目以指向另一区域的时间。

# 使用 Azure Storage Tables 存储数据

我们将通过了解 Table Storage 开始我们的 Azure Storage 能力之旅。如果你希望存储几乎无限容量的非结构化数据，并且对可用性和耐用性有很高的要求，那么这个服务适合你。在这一部分，你将学习如何开始使用 Table Storage 开发应用程序，并了解存储数据和实现最佳性能的最佳实践，无论是写入还是读取。你还将学习如何高效地查询数据，以及在设计使用此 Azure Storage 功能的服务时需要注意的事项。

# 创建 Azure Storage 服务

要开始使用，首先我们需要实际创建一个 Azure Storage 实例。为此，请按照以下步骤操作：

1.  进入 Azure 门户并点击+创建资源。搜索`storage account`，然后点击创建按钮。

1.  你会看到一个典型的表单，你需要在其中配置一个新的服务实例。以下是我选择的一个示例：

![](img/a3843f54-4e7d-423a-aa0e-b724dfe360b5.png)

现在我想介绍一些这里提供的更神秘的选项：

+   部署模型：根据当前需求，您可以选择不同的部署模型。一般来说，几乎每个新的存储账户，默认选项是使用**资源管理器**。`经典`模式设计用于传统部署，使用经典虚拟网络。这个选择还限制了在选择`账户种类`和一些额外功能，如`性能`层时可用的选项。

+   账户种类：这里有三个选项可用（通用、V1/V2 和 Blob）。如果想要使用包括表、队列、Blob 在内的多种功能的存储账户，请选择存储。选择`V2`允许您定义访问层（冷或热），这直接关联到访问账户内数据的频率。

+   需要安全传输：使用**Azure 存储**可以要求启用安全连接选项。为了您的生产工作负载，务必打开此选项，以防止任何人通过例如 HTTP 而非 HTTPS 访问存储在账户中的数据。

+   性能：可以选择`标准`或`高级`性能层。正如前面提到的，这会影响使用的硬件，标准层使用常规磁盘，而高级层使用 SSD。

+   虚拟网络：与许多其他服务一样，可以在虚拟网络中配置**Azure 存储**，以进一步限制对其的访问。

一切准备就绪后，您可以点击`创建`按钮并稍等片刻——您的账户将被创建，并很快您将能够开始使用它。

# 管理表存储

当您转到`概述`刀片时，您将看到一个包含有关您账户的基本信息的仪表板：

![图片](img/8c59ccb6-4783-4420-8643-b67536590506.png)

正如您所见，它显示了您在创建时定义的信息，如位置、性能层或复制类型。此外，当您向下滚动时，还将看到`监视`部分，其中显示整个服务的运行情况：

![图片](img/d7c6d840-3b9d-485f-90bd-6431d6bbc62f.png)

在本节中，我们介绍表存储，因此请在左侧找到`表`刀片并点击。最初，您可能根本看不到任何表——当然，这是我们预期的情况，因为此服务的实例刚刚被配置。尽管如此，这是一种检查账户内实际存储内容的方法之一：

![图片](img/4192d265-7acd-4613-b5d8-2bb2a954734b.png)

要创建新表，只需点击`+表`按钮——您将被要求提供表名，这是开始的全部内容。也许您还记得，我描述过表存储作为存储非结构化数据的能力。这就是为什么在开始一个表时没有其他选项的原因——您只需依赖此服务工作的内部规范。以下展示了创建新容器时的情况：

![图片](img/9e23615b-2b65-4680-b3ee-424c792f4a30.png)

上面的截图展示了一个`orders`表及其 URL——你可能会想知道这个 URL 到底是什么。由于有多种方式来管理和使用 Azure 服务，Azure 存储允许你通过不同的方法使用其功能，如 REST、Powershell、Azure CLI 或 Azure 门户。当你使用 SDK 并阅读其源代码时，你会发现它们实际上只是对简单 REST API 的封装。这使得该服务非常容易入门并在日常工作中使用。我们已经谈了些关于表存储的基础知识——现在是时候描述它们的架构了。

# 在表存储中存储数据

表存储中的每条记录都有一个包含多个列的行结构。每一行有以下基本列：

+   `PartitionKey`：行分区的标识符

+   `RowKey`：行的标识符

+   `Timestamp`：此列表示行最近被修改的时间

+   `ETag`：表存储实现了乐观并发控制模型，并使用 ETag 来控制是否修改实体

当然，你并不局限于上述列——你可以创建任何额外的列，并为每个列指定类型。然而，在进一步进行之前，你需要完全理解这种设计的含义。这里有一个例子，展示了存储在同一个表中的实体：

2018-07-13T11:56:11.108Z

| `PartitionKey` | `RowKey` | `Timestamp` | `Name` | `Price` | `Created` | `CustomerId` | **Quantity** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 订单 | 16Hbs6gs8s | 2018-07-13T11:56:11.108Z |  |  | 2018-07-13T11:36:11.108Z | customer-001 |  |
| 16Hbs6gs8s | 1 | 2018-07-13T11:57:17.108Z | 海绵 | 3.00 | 2018-07-13T11:36:11.108Z |  | 3 |

在前面的例子中，你的数据存储在多个分区中，尽管只使用了一个表，但仍然可以使用多个架构，因此无需使用额外的容器。此外，我使用了一个简单的模式，这使你能够引入 1:n 关系——每个订单都有一个唯一的`RowKey`，可以用作与之相关的实体的分区键（这使得查询数据变得非常简单）。

# PartitionKey

表存储通过分区来分配、加载和处理请求。表中分区键的数量会影响它们的平衡能力。虽然可以为每个表使用单个分区，但在大多数情况下，这是一种无效的方法，会降低存储帐户的性能。分区键的大小限制为 1 KB，并且在表中必须唯一（因此，一旦实体分配了分区键，所有使用相同值的实体将存储在同一存储中）。它们也必须是字符串类型。

# RowKey

每个行键是分区内一行的唯一标识符（因此，只要它们有不同的`PartitionKey`，你可以拥有使用相同`RowKey`列值的多行）。更重要的是，每个表按行键值升序排序。这要求你在设计时要聪明，例如，当你只需要读取部分顶端行而不想提供它们的行键时（我们将在本章后面讲到）。像`PartitionKey`一样，`RowKey`也有 1 KB 的限制，并且必须是字符串。

# 时间戳

这个列是由服务器端维护的，并且是一个`DateTime`值，每次实体被修改时都会更改。它也被内部用于提供乐观并发控制，并且不能被修改。即使你设置了它，值也会被忽略。

# 实体的一般规则

表存储在存储数据时有一些硬性限制：

+   最大列数为 255

+   实体的最大大小为 1 MB

+   默认情况下，每个实体列被创建为字符串类型——在创建时可以覆盖这个默认值

+   不可能存储`null`值——如果你没有提供列的值，实体会被认为没有这个列

# 查询表存储中的数据

要在表存储中查询数据，你需要一个简单的应用程序（可以是控制台应用程序）和该服务的 SDK。你还需要一个 Azure 存储实例——它可以是 Azure 中配置的存储实例，也可以是本地的存储实例（如果你已安装存储模拟器）。

要开始使用存储模拟器，只需搜索可执行文件（例如，`开始` | 输入`Storage Emulator`）并运行它。它会初步创建一个用于存储数据的数据库，并在后台运行，所以你不必担心意外关闭它。

为了开始，我们需要使用 NuGet 包管理器安装`WindowsAzure.Storage`包。它包含了开始在.NET 中使用 Azure 存储所需的所有内容。这里有一个创建表的代码示例：

```
using Microsoft.WindowsAzure.Storage;

namespace TableStorage
{
    internal class Program
    {
        private static void Main()
        {
            var storageAccount = CloudStorageAccount.Parse("<connection-string>");
            var tableClient = storageAccount.CreateCloudTableClient();
            var table = tableClient.GetTableReference("orders");

            table.CreateIfNotExists();
        }
    }
}
```

我们可以简要描述一下这段代码的作用：

1.  它解析连接字符串，以便在接下来的方法中使用

1.  它创建了一个`CloudTableClient`类的实例，这是与表存储交互的主要类

1.  它获取到一个`order`表的引用，无论它是否存在

1.  最后，它会创建一个`orders`表（如果表尚不存在的话）

你也可以使用`Create()`方法来代替`CreateIfNotExists()`，不过如果表已经创建，使用`Create()`方法可能会出错。

现在我们需要获取连接字符串，具体取决于你想要使用的存储账户，你可以：

+   需要进入 Azure 门户，找到你的存储账户，然后从`Access keys`选项卡中复制连接字符串

+   使用`UseDevelopmentStorage=true`值来连接存储模拟器

当你执行一个应用程序时，表应该能够顺利创建。现在，当我们有了表后，我们希望实际往表中插入一些数据。为此，你需要以下代码：

```
var op = TableOperation.Insert(new DynamicTableEntity("orders", Guid.NewGuid().ToString(), "*",
  new Dictionary<string, EntityProperty>
  {
    {"Created", EntityProperty.GeneratePropertyForDateTimeOffset(DateTimeOffset.Now)},
    {"CustomerId", EntityProperty.GeneratePropertyForString("Customer-001")}
  }));

table.Execute(op);
```

在这里，我们创建了一个新的`TableOperation`，它接受一个参数，即`TableEntity`的实例。`TableEntity`是一个包含所有行属性的基类，必须传递给表（如`PartitionKey`或`RowKey`）。当然，除了使用`DynamicTableEntity`，你还可以从`TableEntity`派生并引入自定义实体类。

在前面的示例中，我们使用了`Insert()`操作，但对于并发请求来说，这可能不是最好的选择。在这种情况下，有时更好的做法是使用`InsertOrReplace()`或`InsertOrMerge()`。

最后需要做的是查询表格。要在.NET 中执行此操作，你将需要像这样的一些代码：

```
var query = new TableQuery();
var result = table.ExecuteQuery(query);

foreach (var entity in result)
{
  Console.WriteLine($"{entity.PartitionKey}|{entity.RowKey}|{entity.Timestamp}|{entity["Created"].DateTimeOffsetValue}|{entity["CustomerId"].StringValue}");
}
```

我们刚刚执行了一个基本查询，这将返回表中的所有行。虽然它现在能够工作，但使用这样的查询来查询表中的所有数据并不是最佳选择——在大多数情况下，你将使用以下类似的查询：

```
var query =
  new TableQuery().Where(
    TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "orders"));
```

上述查询将返回表中所有具有`orders`分区键的行。你可以通过生成进一步的过滤条件，按照自己的需求扩展这些查询。

记住，为了获得最佳性能，你的查询应该同时包括`PartitionKey`和`RowKey`。仅使用`PartitionKey`会导致结果较差，但仍然是可以接受的。仅使用`RowKey`将导致读取整个分区。没有使用这些列将导致读取整个表。

你也可以使用 Azure Storage Explorer 查看表中存储的内容：

![](img/3480ecc7-bd3c-4579-9a09-89352ee0fc39.png)

# Azure Cosmos DB 中的表 API

可以使用 Azure Cosmos DB 来利用表存储的高级服务。使用这个选项有以下优点：

+   自动和手动故障转移。

+   二级索引（对行内所有属性进行索引的能力）。

+   跨多个区域的独立扩展。

+   不同的一致性级别。

+   每个表的专用吞吐量。

虽然仅使用表存储也可以实现故障转移，但其余的功能仅适用于 Azure Cosmos DB，并且当你喜欢这个服务的简易性，同时希望面对更复杂的场景时，它将是一个很好的解决方案。

# 使用 Azure Files 实现完全托管的文件共享

当你需要创建一个可以供不同人访问的文件共享时，你通常需要购买一些硬件，这些硬件将为这种功能进行设置和配置，或者使用第三方解决方案，这些解决方案可能难以定制或者价格昂贵。使用 Azure Storage，你可以快速开发一个几乎无限制容量、提供行业标准协议、并且可以快速配置和投入使用的解决方案。

# Azure Files 概念

Azure Files 有一些基本概念，构成了整个服务的框架。实际上，它旨在取代当前的本地文件服务器，在功能和性能方面表现优异。Azure Files 与“传统”解决方案的主要区别在于可访问性（因为您可以设置访问令牌并将 URL 设置为私有）。此外，它是操作系统无关的，允许您在不同的机器上使用相同的文件共享，无论是 Linux、Windows 还是 macOS。当然，它还继承了其他 Azure 存储的概念，因此您可以以同样的可靠性和耐用性来使用它。Azure Files 的主要特点是支持 SMB 协议。这是一种非常常见的协议（也是一个成熟的协议，因其设计于 1980 年代中期），用于共享计算机资源，也用于打印机和其他网络设备。我们可以这样总结 Azure Files：

+   **完全托管**：这是一项完整的云服务，您无需担心操作系统或其配置。

+   **耐用性和弹性**：使用 Azure Files，您无需担心无法访问存储的数据，或担心电力故障等停机事件对资源的安全性造成影响。

+   **常用开发工具**：由于系统 I/O API、适当的 SDK，甚至 REST API，访问 Azure Files 变得非常简单。

# 使用 Azure Files

当您进入 Azure 门户并打开您的 Azure 存储实例时，您可以找到“文件”刀片。它与将要讨论的 Blob 存储非常相似，显示了可用的文件共享列表，如下图所示：

![](img/e4d8165d-ecc6-4010-8cb5-97b5386923ba.png)

在此屏幕上，您可以通过点击+文件共享按钮来创建新的文件共享。这里最重要的是“配额”字段的值，它决定了文件共享的最大容量。

文件共享的配额最大值为 5,120 GB。

要获取有关如何连接到文件共享的信息，您可以点击右侧的更多按钮并选择连接：

![](img/ef04fe98-5092-4c54-9679-888d21ea26e5.png)

它会显示一些简短的指令，说明如何快速从计算机连接到特定的文件共享。这里您可以看到一个在 PowerShell 中为 Windows 编写的命令示例：

```
$acctKey = ConvertTo-SecureString -String "<key>" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\handsonazurestore", $acctKey
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\handsonazurestore.file.core.windows.net\handsonazure" -Credential $credential -Persist
```

您可以使用`-Name`参数指定驱动器的字母（在前面的示例中，它是`Z`）。

映射驱动器是一项可能需要额外权限的操作——请确保以管理员身份运行所有这些命令。

现在，我可以比较在 Azure 门户中显示的文件共享内容：

![](img/d6487b6f-934d-4d69-b010-676d194e71a7.png)

我在虚拟机上挂载的磁盘：

![](img/695b1b20-a5df-42a2-b0e5-3ab4ecee9f02.png)

整个设置只用了几分钟——这是该服务的优势，因为通常我需要花费很多小时来设置所有内容并达到相同的可移植性和可靠性水平。它还为你提供了无限的存储容量——没有任何东西会阻止你附加多个 Azure Files 共享空间并将所有文件存储在其中。

# Blob 存储与 Azure Files

事实上，Azure Blob 存储和 Azure Files 都有类似的目的——它们用于存储和共享文件。然而，在使用场景上，它们之间有一些根本性的差异，例如：

+   如果你想为公司创建一个通用的文件共享空间，你将使用 Azure Files

+   如果你希望为用户通过你的网站上传的文件提供一个存储空间，你将使用 Blob 存储

+   如果你希望文件完全私密，你将使用 Azure Files

+   如果你想在 Blob 或容器级别配置安全性，你将使用 Blob 存储

这两项服务也有不同的定价模型（例如，当按每 GB 数据收费时，Azure Files 要比 Blob 存储贵得多）。

# Azure 队列存储中的队列

Azure 存储——除了作为存储多种不同类型数据的服务外——还可以用作队列。队列存储是另一个功能，允许你快速开发一个需要简单队列解决方案的应用程序，并且能够在队列中存储数百万条消息而不影响性能。在本节中，你将了解如何使用队列存储开发应用程序，以及使用该功能时需要注意的事项。此外，我假设你已经拥有一个存储账户。如果没有，可以查看**使用 Azure 存储表存储数据**一节，里面描述了创建账户的过程。

# 队列存储功能

一般来说，队列存储有两个使用场景：

+   异步处理消息

+   在不同服务之间交换通信（Azure Functions、传统的 Web 角色/Worker 角色）

这是一个非常简单的队列解决方案，能够存储和处理任何格式的消息，每条消息的大小限制为 64 KB。消息的保留时间为七天——超过这个时间，消息将丢失。队列的容量基本上等于你的存储账户的容量。通常，你不必担心存储空间用尽的问题。队列存储与其他 Azure 存储功能共享许多附加功能，如虚拟网络、SAS 令牌等。因此，我们在本节中不再重新介绍这些功能。

# 使用队列存储开发应用程序

为了展示队列存储，我创建了两个应用程序：

+   生产者

+   消费者

生产者将创建并推送消息，随后将被消费者消费。这里是`生产者`应用程序的代码：

```
using System;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Queue;

namespace QueueStorage.Producer
{
    internal class Program
    {
        private static void Main()
        {
            var storageAccount = CloudStorageAccount.Parse("UseDevelopmentStorage=true");
            var queueClient = storageAccount.CreateCloudQueueClient();
            var queue = queueClient.GetQueueReference("orders");

            queue.CreateIfNotExists();

            var message = new CloudQueueMessage($"New order ID: {Guid.NewGuid()}");
            queue.AddMessage(message);
        }
    }
}
```

当然还有`消费者`应用程序：

```
using System;
using Microsoft.WindowsAzure.Storage;

namespace QueueStorage.Consumer
{
    internal class Program
    {
        private static void Main()
        {
            var storageAccount = CloudStorageAccount.Parse("UseDevelopmentStorage=true");
            var queueClient = storageAccount.CreateCloudQueueClient();
            var queue = queueClient.GetQueueReference("orders");

            var message = queue.GetMessage();
            Console.WriteLine(message.AsString);
            Console.ReadLine();
        }
    }
}
```

当您向队列发布消息时，您可以随时取回它—如前所述，您有七天时间从队列中取出消息。这里您可以看到存储在队列中的消息样子：

![](img/0dba0131-21e1-425d-952a-e4c890c5919c.png)

# 对象存储解决方案 – Azure 存储 Blob

Azure 存储的最后一个功能是 Blob 存储。在前面的章节中，我们使用该服务通过表存储存储非结构化数据，通过队列存储推送消息，并通过文件存储创建文件共享。在本章的最后一部分，我们将重点开发存储所谓 blob 的解决方案。您可能会想知道什么是 blob—其实并没有一个单一的定义。一般来说，blob 是各种类型的文件，比如文本文件、图像或音频文件。您将了解如何在应用程序中使用它们，如何保护它们，以及如何实现最大性能。

# Blob 存储概念

在深入了解服务之前，您需要了解 Blob 存储的基本概念。这里有一个图表，清楚地定义了三个主要主题：

![](img/91943c42-c9e8-41ea-aca9-fd880be20ff3.png)

如您所见，我们有三个不同的概念：

+   **账户**：即您的存储账户，存储所有 Blob 存储中的数据。

+   **容器**：一个逻辑实体，可以容纳无限数量的 blob。一个账户可以拥有无限数量的容器。

+   **Blob**：存储在容器中的文件。

此外，还有三种不同类型的 blob：

+   **块 blob**：最大大小为 4.7 TB 的文本或二进制数据。这样的 blob 由更小的块组成。

+   **附加 blob**：一种更为特定的 blob 类型，最适合用于日志数据、存储事件或事务日志等场景。它们针对追加操作进行了优化。

+   **页面 blob**：用于存储虚拟机（VM）使用的 VHD 文件。

使用最新版本的存储账户（v2），可以使用该服务的最新功能。最有趣的新增功能之一是访问层。现在可以选择使用热层或冷层，选择取决于数据访问的频率—如果你希望频繁读取数据，热层是最佳选择；否则，使用冷层或通用账户会更好。

上述的各个存储层在选择 Blob 作为存储账户类型时可用，普通账户不可使用这些存储层。

还有一个层级：归档层—用于存储很少访问的 blob—尽管它仅在 blob 层级可用。您可能会想知道这些层级之间的差异。这里有一张表格，定义了它们的定价：

|  | **热层** | **归档层** | **冷层** |
| --- | --- | --- | --- |
| 前 50 TB/月 | 每 GB $0.0184 | 每 GB $0.01 | 每 GB $0.002 |
| 下一步 450 TB/月 | 每 GB $0.0177 | 每 GB $0.01 | 每 GB $0.002 |
| 每月超过 500 TB | 每 GB $0.017 | 每 GB $0.01 | 每 GB $0.002 |

就存储而言，你可以看到热存储是最贵的，其他层级要便宜得多，特别是归档存储。现在让我们来看一下 10,000 次读取操作的价格：

+   **热存储**：$0.004

+   **冷存储**：$0.01

+   **归档**：$5

哎呀—这里的差别可大了！这就是选择正确层级如此重要的原因——如果你误用了 Blob 存储层级，你最终可能会得到一个花费非常高的解决方案。

# 向 Blob 存储中插入数据

现在我们将尝试向 Blob 存储中实际添加一些内容。这里有一段代码，可以帮助你上传一个文件到容器：

```
using System;
using Microsoft.WindowsAzure.Storage;

namespace BlobStorage
{
    internal class Program
    {
        private static void Main()
        {
            var storageAccount = CloudStorageAccount.Parse("UseDevelopmentStorage=true");
            var cloudBlobClient = storageAccount.CreateCloudBlobClient();
            var container = cloudBlobClient.GetContainerReference("handsonazure");

            container.CreateIfNotExists();

            var blob = container.GetBlockBlobReference("foo.txt"); 
            blob.UploadText("This is my first blob!");

            Console.ReadLine();
        }
    }
}
```

和前面几个例子一样，这个也看起来非常相似。你需要按照以下步骤操作：

1.  首先，你必须创建一个 `CloudStorageAccount` 实例。

1.  然后，你需要获取容器的引用，如果容器不存在，则创建它。

1.  最后，你必须获取一个 Blob 的引用，并上传一些内容。

如果我打开 Azure Storage Explorer，我可以看到一个新的 Blob 已经上传到容器中：

![](img/984feda9-76bc-48c1-b707-84ca85263ea0.png)

当然，如果我打开文件，我会看到它包含我上传的文本：

![](img/c88f981d-9174-4377-ac5c-80d51ebc730c.png)

# 容器和权限

在访问 Blob 存储中的容器时，你可以选择适当的访问级别。如果你进入 Azure 门户并打开你的 Azure Storage 服务，你可以找到 Blobs 磁贴。在其中，你可以点击 + 容器按钮，这将打开一个小窗口：

![](img/081f7f18-611f-4311-a615-42f8d9025b0a.png)

如你所见，除了为容器提供名称外，你还可以选择公共访问级别。目前，你有三种不同的选项可供选择：

+   私有：禁止匿名访问

+   Blob：Blob 层级的匿名访问

+   容器：容器级别的匿名访问

你可以点击你创建的容器，看到另一个屏幕，在那里你可以进行管理。我将用它来实际上传一个文件，看看有哪些其他选项可用。这里展示的是文件上传后，在门户中点击它的样子：

![](img/45bd6c11-2d52-4c9e-8842-a556438b9df2.png)

现在我可以看到关于文件的附加元数据，管理如获取租约或生成 SAS 令牌等操作。

如果你想将文件设为只读，点击“获取租约”按钮——虽然仍然可以更改它，但这种操作需要提供租约 ID。

更重要的是，存在一个可用的 URL 属性，可以直接访问 Blob，例如，使用浏览器访问。在我的例子中，它是这样的：

`https://handsonazurestore.blob.core.windows.net/blob/11047_01_01.PNG`

现在你可能想知道 Blob 和容器访问之间的区别是什么。为了找出答案，我们将使用以下代码：

```
using System;
using Microsoft.WindowsAzure.Storage.Blob;

namespace BlobStorage
{
    internal class Program
    {
        private static void Main()
        {
            var container = new CloudBlobContainer(new Uri("<container-uri>"));
            var blobs = container.ListBlobs();

            foreach (var blob in blobs)
            {
                Console.WriteLine(blob.Uri);
            }

            Console.ReadLine();
        }
    }
}
```

我已经创建了两个不同的容器——一个具有 Blob 访问权限，一个具有容器访问权限。如果我执行前面的代码，针对具有完全公共访问权限的容器，我将看到如下内容：

![](img/36fd3500-e727-4b41-9f2f-6c6f26bd4c53.png)

现在让我们对一个仅具有 Blob 公共访问权限的容器运行它：

![](img/c6135bf3-10fe-4ad2-b5ae-202df734164a.png)

正如你所看到的，当容器的访问级别是 Blob 或私有时，容器级别的操作不可用。当然，如果你使用例如访问密钥进行授权，你可以列出容器中的所有 Blob，即使它是私有的。当然，也可以直接在代码中设置容器级别的权限：

```
using System;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;

namespace BlobStorage
{
    internal class Program
    {
        private static void Main()
        {
            var storageAccount = CloudStorageAccount.Parse("UseDevelopmentStorage=true");
            var cloudBlobClient = storageAccount.CreateCloudBlobClient();
            var container = cloudBlobClient.GetContainerReference("blob");

            container.CreateIfNotExists(BlobContainerPublicAccessType.Blob);

            var blob = container.GetBlockBlobReference("foo.txt"); 
            blob.UploadText("This is my first blob!");

            Console.ReadLine();
        }
    }
}
```

# Blob 存储：附加功能

Blob 存储的一个最新且最酷的特性是软删除功能。它允许你执行称为软删除的操作。这意味着什么？在某些情况下，你可能希望删除一个文件，但又能在固定时间内轻松恢复该删除操作。在 Blob 存储中，通过软删除功能可以实现这一点：

![](img/ea110677-c180-478e-9e60-f086e5ce6dbf.png)

如果你启用此功能，任何已删除的 Blob 将在存储中保留一段设定的天数（但不可检索或修改）。Blob 存储还具有两个其他功能，可以与两个其他 Azure 服务一起使用：

+   Azure CDN：一个为客户提供静态内容的内容分发网络服务——我们将在本书后续章节中详细介绍。

+   Azure 搜索：如前所述，你可以轻松地将 Blob 存储设置为搜索引擎的数据源。

正如你所看到的，这是一个非常灵活且实用的 Azure 存储功能，可以用于文件存储、作为 Azure 搜索文档存储、日志数据库等多种用途。

# 总结

在这一章中，你学习了关于 Azure 最重要的服务之一——Azure 存储的一些基础知识。我们为表格、队列、文件和 Blob 开发了一些解决方案——每种方案都可以实现不同的功能，从异步消息处理到创建文件共享。你还了解了不同的冗余模型，以及这个服务的可靠性和耐用性。在*进一步阅读*部分，你会找到更多的资源，这些资源将帮助你在使用这个 Azure 服务时提升技能，如表格存储模式、性能目标和 REST API 参考。在接下来的章节中，你将学习一些数据处理服务，比如 Azure 事件中心和 Azure 流分析。

# 问题

1.  在创建帐户时，选择 Blob 作为帐户类型时，提供哪些存储层？

1.  要在查询表格存储时实现最佳性能，必须包含哪些内容？

1.  存储帐户有哪些可用的冗余模型？

1.  Blob 存储和文件存储有什么区别？

1.  你能使用 Blob 存储来存储二进制文件吗？

1.  消息在队列存储中能存活多久才会被移除？

1.  Queue 存储中消息的最大大小是多少？

1.  `PartitionKey` 列值的最大大小是多少？

1.  表存储中实现了什么并发模型？

1.  Azure Files 存储和本地文件系统之间有什么区别？

# 深入阅读

+   磁盘存储: [`docs.microsoft.com/en-us/azure/virtual-machines/windows/about-disks-and-vhds`](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/about-disks-and-vhds)

+   SAS 令牌参考: [`docs.microsoft.com/en-us/azure/storage/common/storage-dotnet-shared-access-signature-part-1`](https://docs.microsoft.com/en-us/azure/storage/common/storage-dotnet-shared-access-signature-part-1)

+   ARM 与经典部署: [`docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-deployment-model`](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-deployment-model)

+   表存储数据模型: [`docs.microsoft.com/en-us/rest/api/storageservices/Understanding-the-Table-Service-Data-Model`](https://docs.microsoft.com/en-us/rest/api/storageservices/Understanding-the-Table-Service-Data-Model)

+   Blob 存储定价: [`azure.microsoft.com/en-us/pricing/details/storage/blobs/`](https://azure.microsoft.com/en-us/pricing/details/storage/blobs/)

+   文件存储性能目标: [`docs.microsoft.com/en-us/azure/storage/files/storage-files-scale-targets`](https://docs.microsoft.com/en-us/azure/storage/files/storage-files-scale-targets)

+   表存储指南: [`docs.microsoft.com/en-us/azure/storage/tables/table-storage-design-guidelines`](https://docs.microsoft.com/en-us/azure/storage/tables/table-storage-design-guidelines)

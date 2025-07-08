# 第九章：瑞士军刀 - Azure Cosmos DB

在存储方面，我们经常需要使用多个数据库存储多个数据模式。由于需要使用多个服务，管理我们的解决方案变得繁琐，并且需要具备相当的技能才能以正确的方式进行。感谢 Azure Cosmos DB，我们可以使用不同的数据库模型（如 MongoDB、表存储或 Gremlin）存储记录，同时仅为我们商定的内容付费——吞吐量、延迟、可用性和一致性，这一切都得益于无服务器模型。

本章将涵盖以下主题：

+   Azure Cosmos DB 是什么，它与其他存储系统相比如何

+   分区、吞吐量和一致性

+   不同的 Azure Cosmos DB 数据库模型

+   安全功能

# 技术要求

要执行本章中的练习，您需要以下内容：

+   Microsoft Visual Studio 或 Visual Studio Code

+   Azure 订阅

# 理解 Cosmos DB

在处理存储时，您可能听说过不同种类的存储：关系型数据库、NoSQL 数据库、图数据库、文档数据库。存储数据时，有很多不同的模型可供选择，每种模型具有不同的特点。如果您需要轻松地维护表之间的关系，通常情况下您会选择像 SQL Server 这样的数据库。另一方面，也许您想将每个记录保存为 JSON 文件格式，那么最合适的解决方案就是 MongoDB 实例。虽然选择完全取决于您，但最大的问题是，您需要有不同种类的服务来实现相同的目的——存储数据。这就是 Azure Cosmos DB 发挥作用的地方。凭借其多模型功能、灵活性和可扩展性，它是全球分布式和高响应应用的理想选择。在本节中，您将学习如何开始使用此服务及其主要功能。

# 在门户中创建 Cosmos DB 实例

我们将通过在 Azure 门户中创建 Azure Cosmos DB 开始我们的旅程：

1.  当您点击 + 创建资源并搜索 `Azure Cosmos DB` 时，您将看到一个简单的表单，允许您选择该服务的基本功能：

![](img/eef48642-0d4b-48da-92bd-37a50979efe7.png)

然而，也有一些不太明显的功能，需要稍微解释一下：

+   **API**：如前所述，Azure Cosmos DB 允许您在创建时使用几种不同的 API。目前，有五种可用的 API：SQL、MongoDB、Cassandra、Azure Table 和 Gremlin。根据所选择的 API，您将拥有不同的功能（而且更重要的是，应用程序代码中与数据库通信时需要不同的软件包）。

+   启用地理冗余：通过选择此选项，您的数据将分布在两个配对区域之间（取决于您在位置下拉菜单中选择的区域），例如西欧和北欧，或美国中部和美国东部 2。

+   启用多主模式：这是一个新的（目前处于预览阶段）功能，你可以在全球范围内拥有多个主数据库，而不仅仅是单一的主数据库。这大大减少了读取已保存数据时的延迟（因为你不必等待数据传播），并提高了一致性和数据完整性（因为你可以将数据写入特定区域的主实例）。

+   虚拟网络：根据你选择的模型，你可以通过将 Azure Cosmos DB 实例放入特定的虚拟网络和子网中，来限制对其的访问。目前，这对于两种数据库模型是可行的：SQL 和 MongoDB。

1.  当你对所有输入的数据感到满意时，可以点击“创建”按钮来创建它。服务创建完成后，你可以访问“概览”面板，查看它的初始工作状态：

![](img/5d0b4b56-f433-4249-8e9a-02d106e47757.png)

如你所见，中心显示了一个地图，告诉你数据在不同区域是如何复制的。如果你点击地图，你将能够重新配置初始设置。如果你点击“添加新区域”按钮，你将能够搜索特定区域并将其选择为额外的读取区域。或者，你可以直接点击一个区域图标：

![](img/ed8e4a92-228e-483c-b54e-0aa5a3b9a7cd.png)

在当前设置中，你无法添加额外的写入区域。要做到这一点，你必须使用我之前提到的多主模式功能。

一旦保存了额外的区域，**手动故障转移**和**自动故障转移**将会变为激活状态。故障转移的概念很简单——如果你的写入区域出现故障并变得不可用，另一个可用的读取区域可以代替它。唯一的区别是，你是希望手动执行故障转移，还是自动执行。

如果你选择了自动故障转移，你可以决定读取和写入区域之间的切换顺序。如果你想，比如，首先从北欧切换到西欧，那么西欧必须在列表中成为最高优先级。

如果你返回到“概览”面板，你会注意到一些额外的功能：

+   监控：在这里，你可以轻松找到所有对数据库的请求及其状态。

+   启用地理冗余：如果你在创建 Azure Cosmos DB 实例时没有启用此功能，你现在可以启用。

+   数据浏览器：点击此按钮，你可以轻松访问一个浏览器，允许你插入和修改数据。

此外，你还可以前往“快速入门”面板，在那里你将能够开始使用此 Azure 服务开发应用程序：

![](img/069d2761-adae-4f90-a18f-afcc76a422a3.png)

根据所选的数据库模型，您将能够访问不同的初始配置。此外，如您所见，您可以选择是否使用 .NET、Node.js、Java 或 Python——这些语言都可以轻松与 Azure Cosmos DB 集成，使其在创建多平台应用程序时成为一个更好的选择。

# 在 Visual Studio 中使用 Azure Cosmos DB

除了在门户中控制 Azure Cosmos DB，您还可以直接在您的代码和 IDE 中访问它，例如 Visual Studio。像许多其他服务一样，您可以使用 Cloud Explorer 浏览您的订阅中所有可用的数据库实例：

![](img/f6e91c8e-ea8f-471d-9470-d37ce4371171.png)

现在让我们尝试通过一个简单的应用程序与它进行通信。虽然 Cosmos DB 的实例最初是空的，但我们可以迅速为其添加一个表。

在本节中，我们将使用 Cosmos DB 中的表 API。如果您想使用其他类型，您需要查看 *进一步阅读* 部分的教程。

请参考以下代码片段：

```
using System;
using Microsoft.Azure.CosmosDB.Table;
using Microsoft.Azure.Storage;

namespace HandsOnAzureCosmosDB
{
    internal class Program
    {
        private static void Main()
        {
            // You can get the connection string from the Quick start blade mentioned previously
            var connectionString = "<connection-string>";
            var storageAccount = CloudStorageAccount.Parse(connectionString);
            var tableClient = storageAccount.CreateCloudTableClient();

            var reference = tableClient.GetTableReference("handsonazure");
            var result = reference.CreateIfNotExists();

            Console.ReadLine();
        }
    }
}
```

在前面的代码中，我们正在创建一个空表，该表应该会在 Cosmos DB 实例中立即可用。现在，如果我再次检查 Cloud Explorer，我看到实际上它是正确的：

![](img/1f0a5e35-91dc-46ac-ba88-283742d706fd.png)

现在我们可以向其中添加一条记录。我们稍微修改代码如下：

```
using System;
using Microsoft.Azure.CosmosDB.Table;
using Microsoft.Azure.Storage;

namespace HandsOnAzureCosmosDB
{
    internal class Program
    {
        private static void Main()
        {
            // You can get the connection string from the Quick start blade mentioned previously
            var connectionString = "<connection-string>";
            var storageAccount = CloudStorageAccount.Parse(connectionString);
            var tableClient = storageAccount.CreateCloudTableClient();

            var reference = tableClient.GetTableReference("handsonazure");
            var result = reference.CreateIfNotExists();

            var executionResult = reference.Execute(TableOperation.Insert(new TableEntity("handsonazure", Guid.NewGuid().ToString())));
            Console.WriteLine(executionResult.Result);

            Console.ReadLine();
        }
    }
}
```

现在我们通过门户中的 Data Explorer 来查看 Visual Studio 和 Azure 门户之间的差异。我们应该能够看到我们刚刚插入的实体：

![](img/2c3adb3f-1415-424f-8e49-502ed27d37ce.png)

如您所见，开始使用具有多种数据库模型的数据库，且能够快速配置以实现地理冗余并在全球范围内进行扩展，只需几行代码。

# Azure Cosmos DB 的定价

Azure Cosmos DB 是 Azure 中的一部分无服务器服务。这意味着配置和提供服务器以运行它的机会要么极为有限，要么不可用。如您所见，我们无法定义希望运行多少实例的服务（或节点或集群）。相反，我们必须为每个集合单独定义 **吞吐量**：

![](img/17c2f81b-fe7d-4097-9d5f-8162bde5b368.png)

一个简单的计算器显示屏还估算了每小时和每天的集合成本。Azure Cosmos DB 中的吞吐量单位是 **请求单位**（**RUs**）。在创建容器（或集合——您可以使用这两种定义）时，您还需要指定其类型——是 **固定** 容量还是 **无限制**。

一旦定义了集合类型，您无法再更改它。

通过选择不同的选项，您可以为 RUs 选择不同的限制：

+   对于 **固定**，您可以选择从 400 到 10,000 的 RUs。

+   对于 **无限制**，您可以从 1,000 RUs 到最多 100,000 RUs 之间进行选择。

关于定价，你支付的是存储的数据量（$0.25 GB/月）和预留的 RUs（每 100 RUs 每小时 $0.008）。有了这些值，我们可以迅速计算出最小账单金额——大约是 $23。现在有一个非常重要的警告：你是按每个集合/表/容器收费的。这意味着，如果你有例如 20 个不同的表，你需要支付 20 * $23 = $462。在这种情况下，有时最好将数据库建模为可以将所有数据存储在一个容器中的方式。

尽管 Azure Cosmos DB 看起来是一项相当昂贵的服务，但请记住，它为你做了很多事情，比如地理冗余、多个读取区域、多主模式等。你始终需要计算出最适合你的选项（并且如果能通过类似的方式获得相同的结果）。为此，可以查看 *进一步阅读* 部分中的容量规划器。

# 分区、吞吐量和一致性

现在我们已经了解了有关 Azure Cosmos DB 的一些基本知识——它是如何工作的以及其最常见的功能——接下来我们可以稍微关注一下这个服务中的三个非常重要的主题：分区、吞吐量和一致性。这些因素在选择数据库引擎来支持你的应用程序时至关重要。它们直接告诉你系统的性能表现、能够处理多少请求以及在数据完整性方面适用的保证。

# Azure Cosmos DB 中的分区

分区与 Azure Cosmos DB 中的扩展性密切相关，因为它可以为传入请求进行负载均衡。事实上，在此服务中有两种不同类型的分区：

+   **物理**：这些是固定存储和可变计算资源的组合。这种类型的分区由 Cosmos DB 完全管理——你无法直接影响数据是如何物理分区的，也无法控制服务如何处理这些分区。实际上，你也不知道当前有多少分区正在被使用。因此，你不应该针对这个特定概念设计容器。

+   **逻辑**：这种分区包含具有相同分区键的数据。因为你可以定义该键（通过在每个实体中指定它），所以你可以控制数据的分区方式。

请记住，逻辑分区的最大限制为 10 GB。此外，所有属于一个逻辑分区的数据必须存储在同一个物理分区内。

现在你可能会想知道 Azure Cosmos DB 中的分区是如何工作的。这可以通过几个步骤来描述：

1.  每次创建一个新容器时（并且你提供了若干 RUs），Cosmos DB 都需要创建物理分区，这些分区将能够处理指定数量的请求。

1.  结果可能是指定的 RUs 数量超过了每个分区每秒处理的最大请求数。在这种情况下，Cosmos DB 将预配所需的分区数量，以满足你的要求。

1.  接下来需要做的是为分区键哈希分配空间。所有已预配的分区都必须分配相同的空间（以便均匀分配）。

1.  现在，如果某个分区在一段时间后达到了存储限制，它将被拆分为两个新的分区，数据将在它们之间均匀分配。

当然，用于分区的分区键因数据库模型而异——对于表来说，它是分区键，对于 SQL，它将是一个自定义路径。一般而言，这个操作根据数据库类型略有不同，但整体概念是相同的。

记住，如果你的容器对其中所有实体只有一个分区键，那么 Cosmos DB 将无法对该分区进行拆分。这意味着，你可能会达到每个分区最大 10GB 的限制，并且无法再添加更多数据。

分区键的最佳值完全取决于你的应用数据规范。通常，你需要选择一个具有较大差异性的值（这样才可以进行分区）。另一方面，你不应为每个记录创建唯一的分区键（尽管可能做到，但成本非常高）。对于这个问题没有统一的解决方案——你总是需要分析每个场景，并选择最适合你的方案。

在大多数情况下，值得在过滤数据时包括分区键，因为它可以支持高并发。

# Azure Cosmos DB 中的吞吐量

在*进一步阅读*部分，你将找到一个容量计算器——这是一种工具，可以帮助你规划你的 Cosmos DB 实例并估算所需的 RUs。如前所述，在这个特定服务中，你并不需要定义实例或集群的数量。相反，在创建容器时，你需要指定该特定集合（或一组集合）期望的吞吐量。得益于 Azure Cosmos DB 的**服务水平协议**（**SLA**），该值将得到保障。此外，即使你将数据库复制到另一个区域，你也可以预期一个区域的问题不会影响其他区域。

1 个请求单位有一个重要的定义——它是处理能力，允许你使用简单的 GET 请求读取 1KB 的实体。对于插入或删除等操作则不同，因为这些操作需要更多的计算能力来执行。

如果你想知道一个特定操作消耗了多少 RUs，你需要查看来自 Cosmos DB 实例的响应中的`x-ms-request-charge`头部。它将告诉你此操作的成本——当然，你需要记住，这个成本可能会根据返回的记录数量而有所不同。在文档中，你可以找到以下表格：

| 操作 | 请求单位费用 |
| --- | --- |
| 创建项 | ~15 RU |
| 读取项 | ~1 RU |
| 按 ID 查询项 | ~2.5 RU |

这些是针对大小为 1 KB 的实体执行操作时的值。如你所见，值会根据操作类型的不同而完全不同。你还可以看到，仔细检查所有操作要求是至关重要的——如果你没有做到这一点，可能会遇到 HTTP 429 响应，表示你已经超过了预留的吞吐量限制。在这种情况下，你应该遵守`x-ms-retry-after-ms`头响应，这样可以轻松实施重试策略。

# Azure Cosmos DB 中的一致性

除了不同的数据库模型外，Azure Cosmos DB 还提供了不同级别的一致性。你可能会想知道一致性是什么，它如何影响你的数据。

我们可以按如下定义它：

一致性是数据库系统的一个参数，反映了事务如何影响数据。它定义了当不同的约束或/和触发器影响写入数据库的数据时所应用的规则。

基本上，它告诉你，如果一组操作影响了你的数据，它将不会被损坏，你可以依赖它。以下是 Cosmos DB 中可用的一致性模型：

+   STRONG

+   BOUNDED STATELESS

+   SESSION

+   CONSISTENT PREFIX

+   EVENTUAL

在上述列表中，每个低于 STRONG 的一致性级别都会给你带来较少的一致性。这对于 EVENTUAL 尤其如此，你可能通过一个名为“最终一致性”的主题有所了解。一般来说，你为你的账户设置默认的一致性级别——然后可以针对每个请求覆盖它（当然，如果你愿意的话）。如果你想了解每个一致性级别具体如何工作，请参阅*进一步阅读*部分。要在你的 Cosmos DB 实例中设置特定级别，请点击“默认一致性”选项卡：

![](img/f2f6e600-c9ec-43f2-8d6f-df1417e8691f.png)

如你所见，它允许你根据需求轻松切换到另一个一致性级别。更重要的是，它显示了一个漂亮的动画，描述了在多个区域中读取/写入如何适应这个特定级别。下面的截图展示了最终一致性的动画：

![](img/0a21d58f-400e-4c64-93d7-ad1e0ca31e47.png)

在此截图中，每个单独的节点代表在特定区域中的单次读取或写入操作。此外，在这个界面中，你还可以在选择了 BOUNDED STALENESS 级别时设置最大延迟（时间）。

# CosmosDB 数据模型和 API

如前所述，Azure Cosmos DB 提供了五种不同的数据库模型，它们共享相同的基础设施和概念。这是一个很棒的特点，使得该服务非常灵活，可以服务于多种不同的目的。在本节中，我将简要描述每个数据库模型，以便你能够选择最适合你需求的模型。

# SQL

如果你想到 SQL，你可能会想到一个包含表格、关系和存储过程的关系型数据库。当你使用 Cosmos DB 中的**SQL API**时，实际上你会使用可以通过 SQL 语法查询的文档。假设你想使用以下调用查询文档：

```
SELECT * FROM dbo.Order O WHERE O.AccountNumber = "0000-12-223-12"
```

这里是一个用 C#编写的查询示例：

```
var order = 
  client.CreateDocumentQuery<Order>(collectionLink)
    .Where(so => so.AccountNumber == "0000-12-223-12")
    .AsEnumerable()
    .FirstOrDefault();
```

如你所见，这完全是通过一个简单的 LINQ 查询，它允许你使用特定的属性来过滤数据。由于所有 Cosmos DB 中的记录都是以 JSON 文档的形式存储的，你可以轻松地将它们从表格形式转换为文档表示（并可能进行非规范化处理）。

使用文档数据库与存储关系型数据库中的数据完全不同。始终记住要根据数据库的能力恰当地建模你的数据。

# MongoDB

由于 Cosmos DB 实现了 MongoDB 的协议，你可以轻松将当前使用 MongoDB 文档数据库的所有应用程序迁移到新的 Azure Cosmos DB 实例，而无需更改任何内容（除了连接字符串）。虽然它目前还无法完全模拟 MongoDB（支持的操作的完整列表可以在*进一步阅读*部分找到），但在大多数情况下，你将能够无缝地使用它。由于 Cosmos DB 在安全性方面有严格要求，因此你在与其通信时必须使用 SSL：

```
mongodb://username:password@host:port/[database]?ssl=true
```

这里你可以看到一个连接字符串的模板，`ssl=true`已经包含在其中——它是在与此 Azure 服务通信时必需的。而且，你将无法在未认证请求的情况下设置通信。

# 图形

Azure Cosmos DB 支持 Gremlin 作为图形数据库模型。如果你不熟悉图形数据库，你可以将其看作由顶点和边组成的结构。它们能够非常方便地展示图形中不同元素之间的关系，因为你可以迅速遍历这些连接，发现元素 A 通过元素 C 间接了解了元素 B。更具体地说， Cosmos DB 支持一种被称为**属性图**的更具体的图形数据库模型。以下是 Gremlin 查询的示例：

```
:> g.V('thomas.1').out('knows').out('uses').out('runsos').group().by('name').by(count())
```

上述示例取自文档，直观地回答了这个问题：用户 ID 为 `thomas.1`的关系使用了哪些操作系统？图形数据库非常适用于社交媒体门户或物联网中心等应用。

# 表格

虽然你可以将 Azure Storage Table 用于你的应用程序（这一部分将在接下来的章节中讲解），你也可以利用 Cosmos DB 的 Table API，并考虑使用该服务处理更复杂的场景。这两种服务之间存在一些差异：

+   当前， Azure Storage Tables 的最大操作限制为每秒 20,000 次操作，而 Cosmos DB 则能够达到数百万次操作

+   你不能为 Storage Table 启动故障转移

+   在 Cosmos DB 中，数据会根据所有属性进行索引，而不仅仅是分区键和行键

+   不同的定价（存储与吞吐量）

+   不同的一致性级别

使用 Cosmos DB Table API 进行开发与使用 Azure Table Storage 是相同的。以下是一个 C#代码示例，从表中检索实体：

```
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
CloudTable table = tableClient.GetTableReference("people");
TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

foreach (CustomerEntity entity in table.ExecuteQuery(query))
{
    Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
        entity.Email, entity.PhoneNumber);
}
```

# Cassandra

在 Azure Cosmos DB 中，最后一个可用的模型是 Cassandra。Cassandra 是一个可扩展、持久化和去中心化的数据库，用于存储海量数据。现在，如果你将其与 Cosmos DB 一起使用，你可以专注于开发，而不是操作、性能管理或一致性选择。虽然当前此模型处于预览阶段，但你可以进行测试，并查看它带给你的优势。底层使用 Cassandra API，因此可以使用 Cassandra 查询语言与数据进行交互。此模型与 MongoDB 相似——你可以使用与当前 Cassandra 实例相同的工具，且不应注意到任何差异。

# Cosmos DB 的不同功能

Azure Cosmos DB 具有多个不同的功能，可以帮助你降低费用、保障实例安全或与其他服务集成。在本节中，我们将快速浏览其中的大部分功能，帮助你全面理解此服务的基础知识，并能自主推进。

# 账户级吞吐量

有时，你可能希望为整个账户设置一个固定的吞吐量值，而不是为每个集合单独定义吞吐量。如果你有许多不同的容器，且不希望为每个容器单独付费（正如你记得的——每月超过$20），你可以选择账户级吞吐量并为整个账户设置吞吐量：

![](img/9f751068-bbf5-4766-922d-5bec4effbc33.png)

该功能的唯一限制是，你目前不能在账户中创建表。如果启用此功能，所有对所有表的请求将共享相同数量的吞吐量（这样你可以支付更少的费用，但如果遇到“贪婪”的集合，你可能会耗尽 RUs）。该功能的缺点是，无论是否创建了集合，你都将为已配置的吞吐量付费。

请注意，上述功能仅适用于 Table API。

# 数据库级吞吐量

在 Azure Cosmos DB 中，也可以直接为数据库配置吞吐量。为此，你需要在创建数据库时选中“配置吞吐量”复选框：

![](img/46f2bcf5-8708-4699-9272-2098d9bc4794.png)

启用该功能后，所有配置的 RUs 将跨数据库中所有可用的集合共享。

# 防火墙和虚拟网络

如果你在创建 Cosmos DB 时配置了虚拟网络功能，在此面板中你将能够进一步配置它。更重要的是，还可以配置防火墙——因此你可以限制特定 IP 范围的访问或禁止来自其他 Azure 数据中心的连接。一般来说，你不希望数据库被任何人访问，因此如果此功能可用，我强烈建议你使用它。

请注意，目前防火墙和虚拟网络（VNets）仅适用于 SQL API 和 Mongo API。其他 API 的支持应该很快就会提供。

# Azure Functions

您可以通过使用添加 Azure Function 面板，轻松将 Azure Cosmos DB 与 Azure Functions 集成：

![](img/0154b628-5612-42f5-b34d-150a53842fc8.png)

创建一个来自 Cosmos DB 的函数将向您的函数应用程序添加以下代码：

```
#r "Microsoft.Azure.Documents.Client"

using Microsoft.Azure.Documents;
using System.Collections.Generic;
using System;

public static async Task Run(IReadOnlyList<Document> input, TraceWriter log)
{
    log.Verbose("Document count " + input.Count);
    log.Verbose("First document Id " + input[0].Id);
}
```

这是 CSX 代码，我们没有涵盖过——不过，除了语法上的一些微小变化，它是纯 C#。这个函数将监听您在创建过程中选择的集合的变化——接下来它做什么由您决定。一般来说，这是将这两个服务集成的快捷且简单的方式。更重要的是，您可以为您的集合或表生成不止一个 Azure Function。现在，如果我添加一个文档，我可以看到它触发了一个函数：

```
 Pause  Clear  Copy logs  Expand
2018-08-07T06:01:28  Welcome, you are now connected to log-streaming service.
2018-08-07T06:01:34.556 [Info] Function started (Id=5fb63ab3-e128-45ad-b7a8-4ccfdad38c82)
2018-08-07T06:01:34.573 [Verbose] Document count 1
2018-08-07T06:01:34.573 [Verbose] First document Id test_document
2018-08-07T06:01:34.591 [Info] Function completed (Success, Id=5fb63ab3-e128-45ad-b7a8-4ccfdad38c82, Duration=21ms)
```

# 存储过程

Azure Cosmos DB 允许创建可以单独执行的存储过程，并可以包含您不想共享的额外逻辑。

如果您进入数据资源管理器中的集合，您将看到“新建存储过程”标签和创建存储过程的选项：

![](img/00ad3946-c75e-4b1d-91fe-7b3b5966ef94.png)

存储过程是用 JavaScript 编写的——这使您可以轻松访问文档的架构（因为它们都是 JSON 格式的）。更重要的是，它们是按集合注册的。这里有一个最简单的存储过程示例：

```
function sample(prefix) {
    var context = getContext();
    var response = context.getResponse();

    response.setBody("Hello, World");
}
```

# 用户定义的函数和触发器

为了扩展查询语言，您可以编写自己的**用户定义函数**（**UDF**）并在查询中使用。请注意，您不能在存储过程中使用这些函数。UDF 用于扩展 Azure Cosmos DB 中的 SQL 查询语言，并且只能在查询内部调用。而触发器则分为两类：

+   前触发器

+   后触发器

此外，您可以选择此触发器所引用的操作：

+   全部

+   创建

+   删除

+   替换

这里您可以找到一个触发器的示例，它会在文档创建之前更新文档中的时间戳：

```
var context = getContext();
var request = context.getRequest();
var documentToCreate = request.getBody();

if (!("timestamp" in documentToCreate)) {
  var ts = new Date();
  documentToCreate["my timestamp"] = ts.getTime();
}

request.setBody(documentToCreate);
```

触发器当然也可以从数据资源管理器中获取：

![](img/43a704b5-66c8-495d-9bc9-ebae4e0079f4.png)

# 总结

在这一章中，您了解了另一个无服务器 Azure 组件——Azure Cosmos DB。您看到了该服务支持的多种数据库模型，以及多个不同的功能，例如地理冗余和轻松扩展、引入新的读取区域（数据将在这些区域中进行复制）。更重要的是，您现在了解了多个一致性模型，并且知道如何在 Azure 门户中更改它们。

在下一章中，您将学习另一个热门话题：与 Azure Event Grid 的反应式架构。

# 问题

1.  目前 Azure Cosmos DB 支持哪些 API？

1.  Azure Table Storage 和 Cosmos DB 中的 Table API 的功能之间有差异吗？

1.  可用的一致性模型有哪些？

1.  哪种一致性模型更一致——界限一致性、过时一致性，还是最终一致性？

1.  是否可以将 Azure Cosmos DB 的访问限制为仅来自单一 IP 的访问？

1.  SQL API 和 SQL Server 是一样的吗？

1.  使用存储过程的原因是什么？

1.  是否可以为整个账户而不是每个集合单独配置 Azure Cosmos DB 的吞吐量？

# 进一步阅读

+   在 Azure Cosmos DB 中分区数据: [`docs.microsoft.com/en-us/azure/cosmos-db/partition-data`](https://docs.microsoft.com/en-us/azure/cosmos-db/partition-data)

+   容量规划器: [`www.documentdb.com/capacityplanner`](https://www.documentdb.com/capacityplanner)

+   Azure Cosmos DB RUs: [`docs.microsoft.com/en-us/azure/cosmos-db/request-units`](https://docs.microsoft.com/en-us/azure/cosmos-db/request-units)

+   一致性级别: [`docs.microsoft.com/en-us/azure/cosmos-db/consistency-levels`](https://docs.microsoft.com/en-us/azure/cosmos-db/consistency-levels)

+   Mongo DB 支持: [`docs.microsoft.com/en-us/azure/cosmos-db/mongodb-feature-support#mongodb-protocol-support`](https://docs.microsoft.com/en-us/azure/cosmos-db/mongodb-feature-support#mongodb-protocol-support)

+   图形 API 和 Gremlin: [`tinkerpop.apache.org/docs/current/reference/#intro`](http://tinkerpop.apache.org/docs/current/reference/#intro)

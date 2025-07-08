# 第六章：使用 Hiera 管理数据

|   | *What you don't know can't hurt me.* |   |
| --- | --- | --- |
|   | --*Edward S. Marshall* |

在本章中，你将学习为什么将数据和代码分开是有用的。你将看到如何设置 Puppet 内置的 Hiera 机制，如何使用它存储和查询配置信息，包括加密的秘密信息如密码，以及如何使用 Hiera 数据创建 Puppet 资源。

![使用 Hiera 管理数据](img/8880_06_01.jpg)

# 为什么选择 Hiera？

我们所说的**配置信息**是什么意思？在你的清单中会有很多我们可以视为配置信息的内容：例如，你所有资源属性的值。看下面的例子：

```
package { 'puppet-agent':
  ensure => '5.2.0-1xenial',
}
```

上面的清单声明了要安装版本为 `5.2.0-1xenial` 的 `puppet-agent` 包。但是当 Puppet 发布新版本时，会发生什么呢？当你想要升级时，你必须找到这段代码，可能在多个目录层级的深处，并且编辑它来改变所需的版本号。

## 数据需要维护

将这一点乘以你在整个清单中管理的所有软件包，问题已经显现。但这只是需要维护的一部分数据，还有很多其他数据需要维护：例如，cron 任务的时间、报告发送的邮件地址、从网页上获取的文件 URL、监控检查的参数、为数据库服务器配置的内存大小等等。如果这些值嵌入在成百上千个清单文件中的代码里，你将为未来埋下麻烦。

如何使你的配置信息更容易查找和维护？

## 设置依赖于节点

将数据和代码混合在一起会使得查找和编辑数据变得更加困难。但还有另一个问题。如果你有两个节点需要用 Puppet 来管理，而且有一个配置值需要在每个节点上不同怎么办？例如，它们可能都有一个 cron 任务来运行备份，但任务需要在每个节点上不同的时间运行。

如何为不同的节点使用不同的值，而不在清单中加入大量复杂的逻辑？

## 操作系统有所不同

如果你有一些节点运行 Ubuntu 16，而有些运行 Ubuntu 18 呢？如果你曾经需要升级节点上的操作系统，你就会知道，从一个版本到下一个版本，许多事情都会发生变化。例如，数据库服务器包的名称可能从 `mysql-server` 改成了 `mariadb-server`。

如何根据节点运行的操作系统找到在清单中使用的正确值？

## Hiera 方法

我们希望的是在 Puppet 中拥有一个类似于中央数据库的地方，可以查找配置设置。数据应当与 Puppet 代码分开存储，并且使得查找和编辑值变得容易。应该能够通过 Puppet 代码或模板中的简单函数调用来查找值。此外，我们还需要根据节点的主机名、操作系统或其他因素来指定不同的值。我们还希望能够对值强制执行特定的数据类型，如 String 或 Boolean。数据库应当为我们完成所有这些工作，并返回需要的适当值。

幸运的是，Hiera 正是如此操作的。Hiera 允许你将配置数据存储在简单的文本文件中（实际上是 YAML、JSON 或 HOCON 文件，这些文件使用流行的结构化文本格式），其内容如下所示：

```
---
  test: 'This is a test'
  consul_node: true
  apache_worker_factor: 100
  apparmor_enabled: true
  ...
```

在你的清单中，你可以使用 `lookup()` 函数查询数据库，如以下示例所示（`lookup.pp`）：

```
file { lookup('backup_path', String):
  ensure => directory,
}
```

`lookup` 的参数是你想要检索的 Hiera 键的名称（例如 `backup_path`）以及预期的数据类型（例如 `String`）。

# 设置 Hiera

在开始使用 Hiera 之前，Hiera 需要知道一两个信息，这些信息在 Hiera 配置文件中指定，文件名为 `hiera.yaml`（请不要将其与 Hiera 数据文件混淆，后者也是 YAML 文件，我们将在本章后面了解它们）。每个 Puppet 环境都有自己的本地 Hiera 配置文件，位于环境目录的根目录下（例如，对于 `production` 环境，本地 Hiera 配置文件的位置为 `/etc/puppetlabs/code/environments/production/hiera.yaml`）。

### 提示

Hiera 还可以使用位于 `/etc/puppetlabs/puppet/hiera.yaml` 的全局配置文件，该文件优先于每个环境的配置文件，但 Puppet 文档建议仅在某些特殊情况下使用此配置层，例如临时覆盖；所有常规的 Hiera 数据和配置应该保存在环境层中。

以下示例展示了一个最小的 `hiera.yaml` 文件（`hiera_minimal.config.yaml`）：

```
---
version: 5

defaults:
  datadir: data
  data_hash: yaml_data

hierarchy:
  - name: "Common defaults"
    path: "common.yaml"
```

YAML 文件以三个破折号和换行符开始（`---`）。这是 YAML 格式的一部分，而不是 Hiera 的特性；它是指示新 YAML 文档开始的语法。

`defaults` 部分中最重要的设置是 `datadir`。它告诉 Hiera 在哪个目录查找数据文件。通常情况下，这些文件位于 Puppet 清单目录的 `data/` 子目录中，但如果需要，你可以更改这个设置。

### 提示

大型组织可能会发现将 Hiera 数据文件与 Puppet 代码分开管理非常有用，例如，可以将其放在一个单独的 Git 仓库中（例如，你可能希望授予某些人编辑 Hiera 数据的权限，但不允许编辑 Puppet 清单）。

`hierarchy` 部分也很有趣。它告诉 Hiera 要读取哪些文件及其顺序。在示例中，仅定义了 `Common defaults`，指示 Hiera 在名为 `common.yaml` 的文件中查找数据。我们将在本章后面看到，您还可以在 `hierarchy` 部分做更多操作。

# 将 Hiera 数据添加到您的 Puppet 仓库

您的 Vagrant 虚拟机已经配置好了适当的 Hiera 配置和示例数据文件，位于 `/etc/puppetlabs/code/environments/pbg` 目录中。现在试试看：

运行以下命令：

```
sudo puppet lookup --environment pbg test
--- This is a test
```

### 提示

我们之前没有看到过 `--environment` 参数，因此现在简要介绍一下 Puppet 环境。Puppet **环境**是一个包含 Hiera 配置文件、Hiera 数据和一组 Puppet 清单的目录——换句话说，这是一个完整的、独立的 Puppet 配置。每个环境都位于 `/etc/puppetlabs/code/environments` 下的一个命名目录中。默认环境是 `production`，但是您可以通过给 `puppet lookup` 命令传递 `--environment` 参数来使用任何您喜欢的环境。在示例中，我们告诉 Puppet 使用 `/etc/puppetlabs/code/environments/pbg` 目录。

当您将 Hiera 数据添加到自己的 Puppet 环境时，可以使用示例中的 `hiera.yaml` 和数据文件作为起点。

## Hiera 故障排除

如果您没有得到结果 `This is a test`，说明您的 Hiera 设置未正常工作。如果您看到警告 `Config file not found, using Hiera defaults`，请检查您的 Vagrant 虚拟机是否包含 `/etc/puppetlabs/code/environments/pbg` 目录。如果没有，请销毁并重新配置您的 Vagrant 虚拟机，使用以下命令：

```
vagrant destroy
scripts/start_vagrant.sh

```

如果您看到类似以下的错误，一般意味着 Hiera 数据文件的语法存在问题：

```
Error: Evaluation Error: Error while evaluating a Function Call, (/etc/puppetlabs/code/environments/pbg/hiera.yaml): did not find expected key while parsing a block mapping at line 11 column 5  at line 1:8 on node ubuntu-xenial
```

如果出现这种情况，请检查您的 Hiera 数据文件的语法。

# 查询 Hiera

在 Puppet 清单中，您可以使用 `lookup()` 函数查询 Hiera 中指定的键（您可以将 Hiera 看作一个键值数据库，其中键是字符串，值可以是任何类型）。

通常，您可以在 Puppet 清单中任何可能使用文字值的地方使用 `lookup()` 调用。以下代码展示了这方面的一些示例（`lookup2.pp`）：

```
notice("Apache is set to use ${lookup('apache_worker_factor', Integer)} workers")

unless lookup('apparmor_enabled', Boolean) {
  exec { 'apt-get -y remove apparmor': }
}

notice('dns_allow_query enabled: ', lookup('dns_allow_query', Boolean))
```

要在示例环境中应用此清单，请运行以下命令：

```
sudo puppet apply --environment pbg /examples/lookup2.pp
Notice: Scope(Class[main]): Apache is set to use 100 workers
Notice: Scope(Class[main]): dns_allow_query enabled:  true
```

## 类型化查找

正如我们所看到的，`lookup()` 函数有第二个参数，指定要检索的值的预期类型。虽然这是可选的，但您应该始终指定它，以帮助捕获错误。如果您不小心查询了错误的键，或在数据文件中错误地输入了值，您会看到类似这样的错误：

```
Error: Evaluation Error: Error while evaluating a Function Call, Found value has wrong type, expects a Boolean value, got String at /examples/lookup_type.pp:1:8 on node ubuntu-xenial
```

# Hiera 数据的类型

正如我们所看到的，Hiera 数据存储在文本文件中，采用名为**YAML Ain't Markup Language**的格式进行结构化，这是组织数据的一种常见方式。以下是来自我们示例 Hiera 数据文件的另一段代码，您可以在虚拟机的 `/etc/puppetlabs/code/environments/pbg/data/common.yaml` 文件中找到：

```
  syslog_server: '10.170.81.32'
  monitor_ips:
    - '10.179.203.46'
    - '212.100.235.160'
    - '10.181.120.77'
    - '94.236.56.148'
  cobbler_config:
    manage_dhcp: true
    pxe_just_once: true
```

实际上，存在三种不同类型的 Hiera 数据结构：**单一值**、**数组**和**哈希**。稍后我们将详细讨论这些内容。

## 单一值

大多数 Hiera 数据由一个与单一值相关联的键组成，如前面的示例所示：

```
syslog_server: '10.170.81.32'
```

值可以是任何合法的 Puppet 值，例如字符串（如本例所示），或者它也可以是整数：

```
apache_worker_factor: 100
```

## 布尔值

你应该在 Hiera 中将布尔值指定为`true`或`false`，且不加引号。然而，Hiera 对布尔值的解释较为宽松：`true`、`on`或`yes`（无论是否带引号）都会被解释为真值，而`false`、`off`或`no`则会被解释为假值。但为了清晰起见，最好遵循以下格式：

```
consul_node: true
```

当你在 Puppet 代码中使用`lookup()`返回布尔值时，可以将其用作条件表达式，例如在`if`语句中：

```
if lookup('is_production', Boolean) {
  ...
```

## 数组

有用的是，Hiera 也可以存储与单一键关联的值数组：

```
monitor_ips:
  - '10.179.203.46'
  - '212.100.235.160'
  - '10.181.120.77'
  - '94.236.56.148'
```

键（`monitor_ips`）后面跟着一个值的列表，每个值单独列在一行并以短横线（`-`）开头。当你在代码中调用`lookup('monitor_ips', Array)`时，值会作为 Puppet 数组返回。

## 哈希

正如我们在第五章，*变量、表达式与事实*中看到的，哈希（在某些编程语言中也叫做**字典**）就像一个数组，其中每个值都有一个标识名称（称为**键**），如以下示例所示：

```
cobbler_config:
  manage_dhcp: true
  pxe_just_once: true
```

哈希中的每对键值都会列出，并缩进在各自的行上。`cobbler_config`哈希包含两个键，`manage_dhcp`和`pxe_just_once`。与这些键关联的值是`true`。

当你在清单中调用`lookup('cobbler_config', Hash)`时，数据会作为 Puppet 哈希返回，你可以使用常规的 Puppet 哈希语法引用其中的单个值，正如我们在第五章，*变量、表达式与事实*（`lookup_hash.pp`）中看到的那样：

```
$cobbler_config = lookup('cobbler_config', Hash)
$manage_dhcp = $cobbler_config['manage_dhcp']
$pxe_just_once = $cobbler_config['pxe_just_once']
if $pxe_just_once {
  notice('pxe_just_once is enabled')
} else {
  notice('pxe_just_once is disabled')
}
```

由于 Hiera 数据常常是哈希中的哈希，你可以通过以下“点符号”(`lookup_hash_dot.pp`)从多层嵌套的哈希中检索值：

```
$web_root = lookup('cms_parameters.static.web_root', String)
notice("web_root is ${web_root}")
```

# Hiera 数据中的插值

Hiera 数据不仅限于字面值；它还可以包括 Facter 事实或 Puppet 变量的值，如以下示例所示：

```
 backup_path: "/backup/%{facts.hostname}"
```

任何位于`%{}`定界符中的内容，都会被 Hiera 解析并插值。在这里，我们使用点符号引用`$facts`哈希中的一个值。

## 使用 lookup()

有帮助的是，你还可以通过将`lookup()`函数作为值的一部分来在 Hiera 数据中插值。这可以避免多次重复相同的值，并使数据更易于阅读，如以下示例（也来自`hiera_sample.yaml`）所示：

```
ips:
  home: '130.190.0.1'
  office1: '74.12.203.14'
  office2: '95.170.0.75'
firewall_allow_list:
  - "%{lookup('ips.home')}"
  - "%{lookup('ips.office1')}"
  - "%{lookup('ips.office2')}"
```

这比单纯列出一组 IP 地址而不说明它们代表什么要易读得多，而且它可以防止你在一个地方更新值却忘记在另一个地方更新，避免引入错误。使用 Hiera 插值使你的数据自文档化。

## 使用 alias()

当你在 Hiera 字符串值中使用 `lookup()` 函数时，结果总是一个字符串。如果你处理的是字符串数据，或者你想将一个 Hiera 值插入到包含其他文本的字符串中，这没问题。然而，如果你处理的是数组、哈希或布尔值，你需要使用 `alias()` 函数。这样，你就可以通过引用其名称在 Hiera 中重用任何 Hiera 数据结构：

```
firewall_allow_list:
  - "%{lookup('ips.home')}"
  - "%{lookup('ips.office1')}"
  - "%{lookup('ips.office2')}"
vpn_allow_list: "%{alias('firewall_allow_list')}"
```

不要被周围的引号所迷惑：看起来 `vpn_allow_list` 可能是一个字符串值，但由于我们使用了 `alias()`，它实际上将是一个数组，就像它所别名的值（`firewall_allow_list`）一样。

## 使用 literal()

因为百分号字符（`%`）告诉 Hiera 进行插值，你可能会想知道如何在数据中指定字面上的百分号。例如，Apache 在其配置中使用百分号来表示像 `%{HTTP_HOST}` 这样的变量名。要在 Hiera 数据中写入这样的值，我们需要使用 `literal()` 函数，它专门用于表示字面上的百分号字符。例如，要将值 `%{HTTP_HOST}` 写为 Hiera 数据，我们需要写成：

```
%{literal('%')}{HTTP_HOST}
```

你可以在示例 Hiera 数据文件中看到一个更复杂的例子：

```
force_www_rewrite:
  comment: "Force WWW"
  rewrite_cond: "%{literal('%')}{HTTP_HOST} !^www\\. [NC]"
  rewrite_rule: "^(.*)$ https://www.%{literal('%')}{HTTP_HOST}%{literal('%')}{REQUEST_URI} [R=301,L]"
```

# 层次结构

到目前为止，我们只使用了一个 Hiera 数据源（`common.yaml`）。实际上，你可以有任意多个数据源。每个数据源通常对应一个 YAML 文件，它们在 `hiera.yaml` 文件的 `hierarchy` 部分列出，优先级最高的源在前，优先级最低的源在后：

```
hierarchy:
  ...
  - name: "Host-specific data"
    path: "nodes/%{facts.hostname}.yaml"
  - name: "OS release-specific data"
    path: "os/%{facts.os.release.major}.yaml"
  - name: "OS distro-specific data"
    path: "os/%{facts.os.distro.codename}.yaml"
  - name: "Common defaults"
    path: "common.yaml"
```

然而，一般来说，你应该尽可能将更多的数据保存在 `common.yaml` 文件中，因为如果数据集中在一个地方，查找和维护会更加方便，而不是分散在多个文件中。

例如，如果你有一些只在 `monitor` 节点上使用的 Hiera 数据，你可能会倾向于将其放在 `nodes/monitor.yaml` 文件中。但是，除非它需要覆盖 `common.yaml` 中的一些设置，否则你只是在让它更难找到和更新。将你能放入的所有内容都放进 `common.yaml`，而把其他数据源只保留用于覆盖公共值。

## 处理多个值

你可能会想知道，如果同一个键出现在多个 Hiera 数据源中会发生什么。例如，假设第一个数据源包含以下内容：

```
consul_node: false
```

此外，假设 `common.yaml` 包含：

```
consul_node: true
```

当你用这些数据调用 `lookup('consul_node', Boolean)` 时，会发生什么？`consul_node` 在两个不同的文件中有两个不同的值，那么 Hiera 会返回哪个值？

答案是，Hiera 会按照`hierarchy`部分中列出的顺序搜索数据源；也就是说，按照优先级顺序。它返回第一个找到的值，因此，如果有多个值，只有来自第一个——即优先级最高——数据源的值会被返回（这就是“层级”部分）。

## 合并行为

我们在前一节中提到，如果有多个值匹配指定的键，第一个匹配的数据源优先于其他数据源。这是默认行为，也是你通常希望的行为。然而，有时你可能希望`lookup()`返回所有匹配值的并集，遍历整个层级。Hiera 允许你指定在多个值匹配时，它应该使用哪种策略。

这被称为**合并行为**，你可以在`lookup()`的第三个参数中指定你希望使用的合并行为，紧跟着键和值类型（`lookup_merge.pp`）：

```
notice(lookup('firewall_allow_list', Array, 'unique'))
```

默认的合并行为称为`first`，它仅返回一个值，即第一个找到的值。相对而言，`unique`合并行为会返回所有找到的值，作为一个扁平化的数组，去除重复值（因此称为`unique`）。

如果你查找哈希数据，可以使用`hash`合并行为来返回一个合并后的哈希，包含所有匹配哈希中的键值对。如果 Hiera 找到两个具有相同名称的哈希键，则仅返回第一个的值。这被称为**浅合并**。如果你需要深度合并（即，匹配的哈希会在所有层级上进行合并，而不仅仅是顶层），请使用`deep`合并行为。

如果这一切听起来有点复杂，别担心。默认的合并行为通常是你大多数时候所需要的，如果你确实需要其他合并行为，可以在 Puppet 文档中找到更多信息。

## 基于 fact 的数据源

层级机制允许你为所有情况设置通用的默认值（通常在`common.yaml`中），但可以在特定情况下覆盖这些值。例如，你可以根据 Puppet fact 的值在层级中设置数据源，比如主机名：

```
  - name: "Host-specific data"
    path: "nodes/%{facts.hostname}.yaml"
```

Hiera 将查找指定 fact 的值，并在`nodes/`目录中搜索具有该名称的数据文件。在前面的例子中，如果节点的主机名是`web1`，Hiera 将会在 Hiera 数据目录中查找数据文件`nodes/web1.yaml`。如果该文件存在并包含指定的 Hiera 键，则`web1`节点将获得该值，而其他节点则会从`common`中获取默认值。

### 注意

请注意，如果你愿意，可以将 Hiera 数据文件组织在`data/`主目录下的子目录中，例如`data/nodes/`。

层次结构中另一个有用的事实是操作系统的主要版本或代号。当你需要让清单在多个操作系统版本上工作时，这非常有用。如果你有多个节点，迁移到最新的操作系统版本通常是一个逐步的过程，每次升级一个节点。如果从一个版本到下一个版本有所变化，影响到你的 Puppet 清单，你可以使用 `os.distro.codename` 事实来选择适当的 Hiera 数据，示例如下：

```
  - name: "OS-specific data"
    path: "os/%{facts.os.distro.codename}.yaml"
```

或者，你可以使用 `os.release.major` 事实：

```
  - name: "OS-specific data"
    path: "os/%{facts.os.release.major}.yaml"
```

例如，如果你的节点正在运行 Ubuntu 16.04 Xenial，Hiera 会查找名为 `os/xenial.yaml`（如果使用 `os.distro.codename`）或 `os/16.04.yaml`（如果使用 `os.release.major`）的数据文件，文件位于 Hiera 数据目录中。

有关 Puppet 中事实的更多信息，请参见第五章，*变量、表达式和事实*。

## 什么应该放在 Hiera 中？

你应该将哪些数据放在 Hiera 中，哪些应该放在 Puppet 清单中？一个很好的经验法则是在考虑何时分离数据和代码时，问问自己将来可能会**变化**的是什么。例如，包的确切版本是 Hiera 数据的一个很好的候选项，因为它很可能需要在未来进行更新。

属于 Hiera 的数据的另一个特点是它**特定**于你的网站或公司。如果你将 Puppet 清单交给其他公司或组织中的某个人，并且她需要修改代码中的某些值以使其在她的网站上运行，那么这些值可能应该放在 Hiera 中。这样做可以更容易地共享和重用代码；你只需要在 Hiera 中编辑一些值。

如果在多个地方都需要相同的数据，最好将该数据存储在 Hiera 中。否则，你要么不得不重复数据，这会增加维护的难度，要么使用全局变量，这在任何编程语言中都不推荐，尤其是在 Puppet 中。

如果在将清单应用到不同的**操作系统**时需要更改数据值，这也可以是 Hiera 数据的一个候选项。正如我们在本章中所看到的，你可以使用层次结构根据事实（例如操作系统或版本）选择正确的值。

另一类属于 Hiera 的数据是类和模块的参数值；我们将在第七章中进一步了解，*掌握模块*。

# 使用 Hiera 数据创建资源

当我们开始使用 Puppet 时，我们是通过在清单中直接使用字面属性值来创建资源的。在这一章中，我们看到如何使用 Hiera 数据填充清单中资源的标题和属性。现在，我们可以更进一步，**直接从 Hiera** 查询中创建资源。这种方法的优势在于，我们可以根据数据创建任何类型的资源，且数量不限。

## 从 Hiera 数组构建资源

在第五章中，*变量、表达式和事实*，我们学习了如何使用 Puppet 的 `each` 函数来迭代数组或哈希，并在此过程中创建资源。让我们将这种技术应用到一些 Hiera 数据中。在我们的第一个示例中，我们将从 Hiera 数组中创建一些用户资源。

运行以下命令：

```
sudo puppet apply --environment pbg /examples/hiera_users.pp
Notice: /Stage[main]/Main/User[katy]/ensure: created
Notice: /Stage[main]/Main/User[lark]/ensure: created
Notice: /Stage[main]/Main/User[bridget]/ensure: created
Notice: /Stage[main]/Main/User[hsing-hui]/ensure: created
Notice: /Stage[main]/Main/User[charles]/ensure: created
```

这是我们使用的数据（来自 `/etc/puppetlabs/code/environments/pbg/data/common.yaml` 文件）：

```
users:
  - 'katy'
  - 'lark'
  - 'bridget'
  - 'hsing-hui'
  - 'charles'
```

这是读取数据并创建相应用户实例的代码（`hiera_users.pp`）：

```
lookup('users', Array[String]).each | String $username | {
  user { $username:
    ensure => present,
  }
}
```

将 Hiera 数据与资源迭代结合起来是一个强大的想法。这个简短的清单可以管理你基础设施中的所有用户，而无需编辑 Puppet 代码来进行更改。要添加新用户，只需编辑 Hiera 数据。

## 从 Hiera 哈希构建资源

当然，现实生活中的情况从来不会像编程语言示例那样简单。如果你真的用 Hiera 数据来管理用户，你需要包含比名字更多的数据：你需要能够管理用户的 shell、UID 等，还需要能够在必要时删除用户。为此，我们需要为 Hiera 数据添加一些结构。

运行以下命令：

```
sudo puppet apply --environment pbg /examples/hiera_users2.pp
Notice: Compiled catalog for ubuntu-xenial in environment pbg in 0.05 seconds
Notice: /Stage[main]/Main/User[katy]/uid: uid changed 1001 to 1900
Notice: /Stage[main]/Main/User[katy]/shell: shell changed '' to '/bin/bash'
Notice: /Stage[main]/Main/User[lark]/uid: uid changed 1002 to 1901
Notice: /Stage[main]/Main/User[lark]/shell: shell changed '' to '/bin/sh'
Notice: /Stage[main]/Main/User[bridget]/uid: uid changed 1003 to 1902
Notice: /Stage[main]/Main/User[bridget]/shell: shell changed '' to '/bin/bash'
Notice: /Stage[main]/Main/User[hsing-hui]/uid: uid changed 1004 to 1903
Notice: /Stage[main]/Main/User[hsing-hui]/shell: shell changed '' to '/bin/sh'
Notice: /Stage[main]/Main/User[charles]/uid: uid changed 1005 to 1904
Notice: /Stage[main]/Main/User[charles]/shell: shell changed '' to '/bin/bash'
Notice: Applied catalog in 0.17 seconds
```

与之前示例的第一个不同点是，这里数据不是简单的数组，而是哈希中的哈希：

```
users2:
  'katy':
    ensure: present
    uid: 1900
    shell: '/bin/bash'
  'lark':
    ensure: present
    uid: 1901
    shell: '/bin/sh'
  'bridget':
    ensure: present
    uid: 1902
    shell: '/bin/bash'
  'hsing-hui':
    ensure: present
    uid: 1903
    shell: '/bin/sh'
  'charles':
    ensure: present
    uid: 1904
    shell: '/bin/bash'
```

这是处理数据的代码（`hiera_users2.pp`）：

```
lookup('users2', Hash, 'hash').each | String $username, Hash $attrs | {
  user { $username:
    * => $attrs,
  }
}
```

`users2` 哈希中的每个键是一个用户名，每个值是一个包含用户属性（如 `uid` 和 `shell`）的哈希。

当我们对这个哈希调用 `each` 时，我们为循环指定了两个参数，而不是一个：

```
| String $username, Hash $attrs |
```

正如我们在第五章中看到的，*变量、表达式和事实*，在迭代哈希时，这两个参数分别接收哈希的键和值。

在循环内部，我们为哈希中的每个元素创建一个用户资源：

```
user { $username:
  * => $attrs,
}
```

你可能还记得上一章中提到的 `*` 运算符（属性展开运算符），它告诉 Puppet 将 `$attrs` 视为属性-值对的哈希。因此，在第一次循环时，对于用户 `katy`，Puppet 将创建一个等效于以下清单的用户资源：

```
user { 'katy':
  ensure => present,
  uid    => 1900,
  shell  => '/bin/bash',
}
```

每次我们在循环中处理 `users` 的下一个元素时，Puppet 将使用指定的属性创建另一个用户资源。

## 使用 Hiera 数据管理资源的优势

上面的示例使得在网络中管理用户变得更加简便，而无需编辑 Puppet 代码：例如，如果你想删除一个用户，你只需将她的 `ensure` 属性在 Hiera 数据中修改为 `absent`。尽管每个用户恰好都有相同的属性集，但这并非必须；你可以将 Puppet `user` 资源支持的任何属性添加到数据中的任何用户。此外，如果有一个属性的值对所有用户始终相同，则无需在每个用户的 Hiera 数据中列出它。你可以将其作为 `user` 资源中的文字属性值添加到循环内，这样每个用户都会拥有它。

这使得定期添加和更新用户变得更加容易，但也有其他优点：例如，你可以编写一个简单的 Web 应用程序，允许人力资源人员使用浏览器界面添加或编辑用户，并且它只需要输出一个包含所需数据的 YAML 文件。这比自动生成 Puppet 代码要容易得多、也更加稳健。更好的是，你可以从 LDAP 或 **Active Directory**（**AD**）服务器中提取用户数据，并将其转换为 Hiera YAML 格式，供该清单使用。

这是一种非常强大且灵活的技术，当然你可以用它来管理任何类型的 Puppet 资源：文件、软件包、Apache 虚拟主机、MySQL 数据库——你能用资源做的任何事情都可以通过 Hiera 数据和 `each` 来实现。你还可以使用 Hiera 的重写机制为不同的节点、角色或操作系统创建不同的资源集。

然而，你不应该过度使用这种技术。根据 Hiera 数据创建资源增加了一层抽象，使得任何试图阅读或维护代码的人更难理解它。使用 Hiera 时，从检查中也很难确切知道在特定情况下节点将获得哪些数据。尽量保持你的层次结构尽可能简单，并将数据驱动的资源技巧保留用于需要频繁更新的大量可变资源的场景。在第十一章，*编排云资源*，我们将看到如何使用相同的技术来管理云实例。

# 管理机密数据

Puppet 经常需要了解你的秘密；例如，密码、私钥和其他凭据需要在节点上配置，并且 Puppet 必须能够访问这些信息。问题是如何确保没有其他人能访问这些信息。如果你将这些数据提交到 Git 仓库，它将对所有有权限访问该仓库的人可见，而如果是公共的 GitHub 仓库，任何人都能看到它。

显然，能够以某种方式加密秘密数据是至关重要的，以便 Puppet 能够在需要的各个节点上解密它，但对没有密钥的人来说是无法解密的。流行的 GnuPG 加密工具是一个不错的选择，它允许您使用公开密钥加密数据，并且可以广泛分发，但只有拥有对应私钥的人才能解密信息。

Hiera 具有可插拔的**后端**系统，允许它支持多种不同的存储数据方式。一个这样的后端叫做`hiera-eyaml-gpg`，它允许 Hiera 使用 GnuPG 加密的数据存储。与加密整个数据文件不同，`hiera-eyaml-gpg`允许您在同一个 YAML 文件中混合加密和明文数据。这样，即使没有私钥的人也可以编辑和更新 Hiera 数据文件中的明文值，尽管加密的数据值对他们来说是不可读的。

## 配置 GnuPG

首先，我们需要安装 GnuPG 并创建一个用于 Hiera 的密钥对。以下说明将帮助您完成这项工作：

1.  运行以下命令：

    ```
    sudo apt-get install gnupg rng-tools

    ```

1.  一旦安装了 GnuPG，运行以下命令生成一个新的密钥对：

    ```
    gpg --gen-key

    ```

1.  当系统提示时，选择 RSA 和 RSA 密钥类型：

    ```
    Please select what kind of key you want:
       (1) RSA and RSA (default)
       (2) DSA and Elgamal
       (3) DSA (sign only)
       (4) RSA (sign only)
    Your selection? 1

    ```

1.  选择 2048 位的密钥大小：

    ```
    RSA keys may be between 1024 and 4096 bits long.
    What keysize do you want? (2048) 2048

    ```

1.  输入`0`作为密钥过期时间：

    ```
    Key is valid for? (0) 0
    Key does not expire at all
    Is this correct? (y/N) y

    ```

1.  当系统提示输入真实姓名、电子邮件地址和用于密钥的评论时，输入适合您网站的内容：

    ```
    Real name: Puppet
    Email address: puppet@cat-pictures.com
    Comment:
    You selected this USER-ID:
        "Puppet <puppet@cat-pictures.com>"

    Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o

    ```

1.  当系统提示输入密码短语时，直接按*Enter*（该密钥不能有密码短语，因为 Puppet 无法提供它）。

生成密钥可能需要几分钟时间，但一旦完成，GnuPG 会打印出密钥指纹和详细信息（您的密钥指纹会与此不同）：

```
pub   2048R/40486112 2016-09-30
      Key fingerprint = 6758 6CEE D221 7AA0 8369  FF3A FEC1 0055 4048 6112
uid                  Puppet <puppet@cat-pictures.com>
sub   2048R/472954EB 2016-09-30
```

该密钥现在存储在您的 GnuPG 密钥环中，Hiera 将能够在该节点上使用它来加密和解密您的秘密数据。稍后我们将看到如何将此密钥分发到 Puppet 管理的其他节点。

## 添加一个加密的 Hiera 数据源

使用 GPG 加密数据的 Hiera 源需要几个额外的参数。以下是示例`hiera.yaml`文件中的相关部分：

```
  - name: "Secret data (encrypted)"
    lookup_key: eyaml_lookup_key
    path: "secret.eyaml"
    options:
      gpg_gnupghome: '/home/ubuntu/.gnupg'
```

与普通数据源一样，我们有`name`和`path`指向数据文件，但我们还需要指定`lookup_key`函数，在这种情况下是`eyaml_lookup_key`，并将`options['gpg_gnupghome']`设置为指向 GnuPG 目录，解密密钥存放在该目录下。

## 创建加密的秘密

现在，您已经准备好将一些秘密数据添加到您的 Hiera 存储中。

1.  使用以下命令创建一个新的空 Hiera 数据文件：

    ```
    cd /etc/puppetlabs/code/environments/pbg
    sudo touch data/secret.eyaml

    ```

1.  运行以下命令，使用`eyaml`编辑器编辑数据文件（保存时，它会自动为您加密数据）。请用您在创建 GPG 密钥时输入的电子邮件地址替代`puppet@cat-pictures.com`。

    ```
    sudo /opt/puppetlabs/puppet/bin/eyaml edit --gpg-always-trust --gpg-recipients=puppet@cat-pictures.com data/secret.eyaml

    ```

1.  如果系统提示你选择默认编辑器，请选择你喜欢的编辑器。如果你熟悉 Vim，建议选择 Vim，但如果不熟悉，你可能会发现`nano`是最简单的选择。（你应该学习 Vim，但那是另一本书的内容。）

1.  你选择的编辑器将启动，并且以下文本已经插入到文件中：

    ```
    #| This is eyaml edit mode. This text (lines starting with #| at the top of the
    #| file) will be removed when you save and exit.
    #|  - To edit encrypted values, change the content of the DEC(<num>)::PKCS7[]!
    #|    block (or DEC(<num>)::GPG[]!).
    #|    WARNING: DO NOT change the number in the parentheses.
    #|  - To add a new encrypted value copy and paste a new block from the
    #|    appropriate example below. Note that:
    #|     * the text to encrypt goes in the square brackets
    #|     * ensure you include the exclamation mark when you copy and paste
    #|     * you must not include a number when adding a new block
    #|    e.g. DEC::PKCS7[]! -or- DEC::GPG[]!
    ```

1.  在注释信息下方输入以下文本，完全按照显示的内容输入，包括开头的三个短横线：

    ```
    ---
      test_secret: DEC::GPG[This is a test secret]!
    ```

1.  保存文件并退出编辑器。

1.  运行以下命令以测试 Puppet 是否能够读取并解密你的机密数据：

    ```
    sudo puppet lookup --environment pbg test_secret
    --- This is a test secret
    ```

## Hiera 如何解密机密数据

为了验证数据确实是加密的，可以运行以下命令查看磁盘上数据文件中的内容：

```
cat data/secret.eyaml
---
  test_secret: ENC[GPG,hQEMA4+8DyxHKVTrAQf/QQPL4zD2kkU7T+FhaEdptu68RAw2m2KAXGujjnQPXoONrbh1QjtzZiJBlhqOP+7JwvzejED0NXNMkmWTGfCrOBvQlZS0U9Vrgsyq5mACPHyeLqFbdeOjNEIR7gLP99aykAmbO2mRqfXvns+cZgaTUEPXOPyipY5Q6w6/KeBEvekTIZ6ME9Oketj+1/zyDz4qWH+0nLwdD9L279d7hnokpts2tp+gpCUc0/qKsTXpdTRPE2R0kg9Bl84OP3fFlTSTgcT+pS8Dfa1/ZzALfHmULcC3hckG9ZSR+0cd6MyJzucwiJCreIfR/cDfqpsENNM6PNkTAHEHrAqPrSDXilg1KtJSAfZ9rS8KtRyhoSsk+XyrxIRH/S1Qg1dgFb8VqJzWjFl6GBJZemy7z+xjoWHyznbABVwp0KXNGgn/0idxfhz1mTo2/49POFiVF4MBo/6/EEU4cw==]
```

当然，实际的密文会对你有所不同，因为你使用了不同的加密密钥。关键点在于，消息完全被打乱了。GnuPG 的加密算法极其强大；即使同时使用地球上所有的计算机，解密使用 2,048 位密钥加密的数据平均需要比当前宇宙年龄多很多倍的时间。（换句话说，在合理的时间内解密数据的概率是几乎不可能的。）

当你在清单中引用像`test_secret`这样的 Hiera 密钥时，接下来会发生什么？Hiera 会查询在`hiera.yaml`中配置的数据源列表。层次结构中的第一个源是`secret.eyaml`，它包含我们感兴趣的密钥（`test_secret`）。以下是该值：

```
ENC[GPG,hQEMA4 … EEU4cw==]
```

`ENC`告诉 Hiera 这是一个加密值，而`GPG`标识正在使用的加密类型（`hiera-eyaml`支持多种加密方法，其中 GPG 是其中之一）。Hiera 调用 GPG 子系统处理加密数据，GPG 在密钥环中搜索找到适当的解密密钥。如果找到密钥，GPG 会解密数据并将结果传回 Hiera，Hiera 再返回给 Puppet，最终的结果是明文：

```
This is a test secret
```

系统的美妙之处在于所有这些复杂性都被隐藏起来；你需要做的只是调用`lookup('test_secret', String)`函数在你的清单中，然后你就能得到答案。

## 编辑或添加加密的机密数据

如果机密数据以加密形式存储，你可能会想知道当你想更改机密值时该如何编辑它。幸运的是，有方法可以做到这一点。记得当你第一次输入机密数据时，你使用了以下命令：

```
sudo /opt/puppetlabs/puppet/bin/eyaml edit --gpg-always-trust --gpg-recipients=puppet@cat-pictures.com data/secret.eyaml

```

如果你再次运行相同的命令，你会发现你正在查看原始的明文（以及一些解释性注释）：

```
---
  test_secret: DEC(1)::GPG[This is a test secret]!
```

你可以编辑`This is a test secret`字符串（确保其他部分保持不变，包括`DEC::GPG[]!`分隔符）。当你保存文件并关闭编辑器时，如果密钥发生变化，数据将使用你的密钥重新加密。

不要删除`DEC`后面括号中的`(1)`，它告诉 Hiera 这是一个已存在的密钥，而不是新的密钥。随着你向此文件添加更多的密钥，它们将被依次编号。

为了方便编辑，我建议你制作一个名为`/usr/local/bin/eyaml_edit`的 shell 脚本，用于运行`eyaml edit`命令。你可以在 Vagrant 盒子中的`/examples/eyaml_edit.sh`找到一个示例，复制到`/usr/local/bin`并进行编辑（像之前一样，用与你的 GPG 密钥关联的邮箱地址替换`gpg-recipients`）：

```
#!/bin/bash
/opt/puppetlabs/puppet/bin/eyaml edit --gpg-always-trust --gpg-recipients=puppet@cat-pictures.com /etc/puppetlabs/code/environments/pbg/data/secret.eyaml
```

现在，每当你需要编辑秘密数据时，可以简单地运行以下命令：

```
sudo eyaml_edit

```

若要添加新的密钥，请添加一行如下所示：

```
  new_secret: DEC::GPG[Somebody wake up Hicks]!
```

当你保存并退出编辑器时，新的加密秘密将被存储在数据文件中。

## 分发解密密钥

现在你的 Puppet 清单使用了加密的 Hiera 数据，你需要确保每个运行 Puppet 的节点都拥有解密密钥的副本。使用以下命令将密钥导出到文本文件中（当然，使用你的密钥邮箱地址）：

```
sudo sh -c 'gpg --export-secret-key -a puppet@cat-pictures.com >key.txt'

```

将`key.txt`文件复制到需要密钥的节点，并运行以下命令导入密钥：

```
sudo gpg --import key.txt
sudo rm key.txt

```

确保在导入密钥后删除所有文本文件的副本。

### 注意

**重要提示**

由于所有 Puppet 节点都有解密密钥的副本，因此此方法只保护你的秘密数据不被没有访问节点权限的人获取。它仍然比将秘密数据以明文形式放入清单中要好得多，但它的缺点是，如果有人能够访问某个节点，他就可以解密、修改并重新加密秘密数据。为了提高安全性，你应该使用一个秘密管理系统，在该系统中节点没有密钥，而 Puppet 只能以只读方式访问秘密。一些可选的系统包括 Hashicorp 的 Vault 和 Conjur 的 Summon。

# 总结

在本章中，我们概述了一些在 Puppet 清单中维护配置数据的问题，并介绍了 Hiera 作为一个强大的解决方案。我们已经看到如何配置 Puppet 使用 Hiera 数据存储，并且如何在 Puppet 清单中使用`lookup()`查询 Hiera 键。

我们已经了解了如何编写 Hiera 数据源，包括字符串、数组和哈希数据结构，以及如何使用`lookup()`将值插入 Hiera 字符串，包括 Puppet 事实和其他 Hiera 数据，还了解了如何使用`alias()`复制 Hiera 数据结构。我们已经学习了 Hiera 的层级结构是如何工作的，以及如何使用`hiera.yaml`文件进行配置。

我们已经看到我们的示例 Puppet 基础设施是如何配置使用 Hiera 数据的，并通过在 Puppet 清单中查找数据值演示了这个过程。在出现问题时，我们还查看了一些常见的 Hiera 错误，并讨论了将数据放入 Hiera 的经验法则。

我们已经探索了使用 Hiera 数据创建资源的方法，使用 `each` 循环遍历数组或哈希。最后，我们介绍了如何使用加密数据与 Hiera，使用 `hiera-eyaml-gpg` 后端，并展示了如何创建一个 GnuPG 密钥并用它加密一个秘密值，然后通过 Puppet 再次检索它。我们探讨了 Hiera 查找和解密秘密数据的过程，开发了一个简单的脚本来方便地编辑加密数据文件，并概述了一种将解密密钥分发到多个节点的基本方法。

在下一章中，我们将学习如何从 Puppet Forge 查找和使用公共模块；如何使用公共模块管理软件，包括 Apache、MySQL 和归档文件；如何使用 `r10k` 工具部署和管理第三方模块；以及如何编写和构建自己的模块。

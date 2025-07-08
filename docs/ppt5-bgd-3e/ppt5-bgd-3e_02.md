# 第二章：创建你的第一个清单

|   | *开始是如此脆弱的时刻。* |   |
| --- | --- | --- |
|   | --*弗兰克·赫伯特，《沙丘》* |   |

在本章中，你将学习如何用 Puppet 编写第一个清单，并如何利用 Puppet 配置服务器。你还将了解 Puppet 如何编译和应用清单。你将看到如何使用 Puppet 管理文件内容，如何安装软件包，以及如何控制服务。

![创建你的第一个清单](img/B08880_02_01.jpg)

# 你好，Puppet——你的第一个 Puppet 清单

任何编程语言中的第一个示例程序，按照传统，都会打印`hello, world`。虽然我们在 Puppet 中可以轻松做到这一点，但让我们做一些更有挑战性的事情，让 Puppet 在服务器上创建一个包含该文本的文件。

在你的 Vagrant 盒子中，运行以下命令：

```
sudo puppet apply /examples/file_hello.pp
Notice: Compiled catalog for ubuntu-xenial in environment production in 0.07 seconds
Notice: /Stage[main]/Main/File[/tmp/hello.txt]/ensure: defined content as '{md5}22c3683b094136c3398391ae71b20f04'
Notice: Applied catalog in 0.01 seconds
```

我们可以暂时忽略 Puppet 的输出，但如果一切顺利，我们应该能够运行以下命令：

```
cat /tmp/hello.txt
hello, world
```

## 理解代码

让我们看一下示例代码，看看发生了什么（运行`cat /example/file_hello.pp`，或者在文本编辑器中打开文件）：

```
file { '/tmp/hello.txt':
  ensure  => file,
  content => "hello, world\n",
}
```

代码术语`file`开始了一个**资源声明**，声明了一个`file`资源。**资源**是你希望 Puppet 管理的一部分配置：例如，文件、用户账户或软件包。资源声明遵循以下模式：

```
RESOURCE_TYPE { TITLE:
  ATTRIBUTE => VALUE,
  ...
}
```

资源声明将构成你几乎所有的 Puppet 清单，因此理解它们是如何工作的非常重要：

+   `RESOURCE_TYPE`表示你声明的资源类型；在本例中，它是`file`。

+   `TITLE`是 Puppet 用来在内部标识资源的名称。每个资源必须具有唯一的标题。对于`file`资源，通常这个标题是文件的完整路径：在这个例子中，是`/tmp/hello`。

这段代码的其余部分是描述如何配置资源的属性列表。可用的属性取决于资源的类型。对于文件，你可以设置`content`、`owner`、`group`和`mode`等属性，但每个资源都支持的一个属性是`ensure`。

再次强调，`ensure`的可能值是特定于资源类型的。在这种情况下，我们使用`file`来表示我们想要一个常规文件，而不是目录或符号链接：

```
ensure  => file,
```

接下来，为了在文件中添加一些文本，我们指定`content`属性：

```
content => "hello, world\n",
```

`content`属性将文件的内容设置为你提供的字符串值。在这里，文件的内容被声明为`hello, world`，后跟一个换行符（在 Puppet 字符串中，我们用`\n`表示换行符）。

请注意，`content`指定了文件的全部内容；你提供的字符串将替换文件中已有的内容，而不是将其附加到文件中。

## 修改现有文件

如果 Puppet 运行时文件已经存在且包含其他内容，会发生什么？Puppet 会修改它吗？

```
sudo sh -c 'echo "goodbye, world" >/tmp/hello.txt'
cat /tmp/hello.txt
goodbye, world
sudo puppet apply /examples/file_hello.pp
cat /tmp/hello.txt
hello, world
```

答案是肯定的。如果文件的任何属性，包括其内容，与清单不匹配，Puppet 将更改它，以使其一致。

如果你手动编辑 Puppet 管理的文件，这可能会导致一些意外的结果。如果你在不更改 Puppet 清单的情况下修改文件，Puppet 在下次运行时会覆盖该文件，你的更改将丢失。

因此，最好在 Puppet 管理的文件中添加注释，内容可以类似于以下示例：

```
# This file is managed by Puppet - any manual edits will be lost
```

在你首次部署文件时，将此内容添加到 Puppet 的文件副本中，它将提醒你和其他人不要手动更改文件。

## 模拟运行 Puppet

因为你不能事先确定应用 Puppet 清单会对系统做出哪些更改，所以最好先进行模拟运行。将 `--noop` 标志添加到 `puppet apply` 中，可以在不做任何实际更改的情况下，展示 Puppet 会执行的操作：

```
sudo sh -c 'echo "goodbye, world" >/tmp/hello.txt'
sudo puppet apply --noop /examples/file_hello.pp
Notice: Compiled catalog for ubuntu-xenial in environment production in 0.04 seconds
Notice: /Stage[main]/Main/File[/tmp/hello.txt]/content: current_value {md5}7678..., should be {md5}22c3... (noop)
```

Puppet 会根据文件的 MD5 哈希值决定是否需要更新 `file` 资源。在上一个例子中，Puppet 报告 `/tmp/hello.txt` 当前的哈希值是 `7678...`，而根据清单，它应该是 `22c3...`。因此，下一次 Puppet 运行时，该文件将被更改。

如果你想查看 Puppet 实际上会对文件做出什么更改，可以使用 `--show_diff` 选项：

```
sudo puppet apply --noop --show_diff /examples/file_hello.pp
Notice: Compiled catalog for ubuntu-xenial in environment production in 0.04 seconds
Notice: /Stage[main]/Main/File[/tmp/hello.txt]/content:
--- /tmp/hello.txt      2017-02-13 02:27:13.186261355 -0800
+++ /tmp/puppet-file20170213-3671-2yynjt        2017-02-13 02:30:26.561834755 -0800
@@ -1 +1 @@
-goodbye, world
+hello, world
```

当你想确保你的 Puppet 清单只会影响你预期的内容时，这些选项非常有用——有时，它们也能帮助你检查某些内容是否在 Puppet 外部发生了变化，而无需实际撤销更改。

## Puppet 如何应用清单

这是清单的处理过程。首先，Puppet 读取清单及其中包含的资源列表（在这个例子中，只有一个资源），并将其编译成目录（节点所需状态的内部表示）。

接下来，Puppet 按顺序应用目录中的每个资源：

1.  首先，它检查资源是否存在于服务器上。如果不存在，Puppet 将创建它。在这个例子中，我们声明了文件 `/tmp/hello.txt` 应该存在。第一次运行 `sudo puppet apply` 时，文件不存在，因此 Puppet 将为你创建该文件。

1.  然后，对于每个资源，它检查目录中每个属性的值与服务器上实际存在的内容是否匹配。在我们的例子中，只有一个属性：`content`。我们指定了文件的内容应该是 `hello, world\n`。如果文件为空或包含其他内容，Puppet 将用目录中指定的内容覆盖该文件。

在这种情况下，第一次应用目录时，文件将为空，因此 Puppet 会将字符串 `hello, world\n` 写入文件中。

在后面的章节中，我们将更详细地探讨 `file` 资源。

## 创建你自己的文件

创建你自己的清单文件（你可以随意命名，只要文件扩展名是`.pp`）。使用`file`资源在服务器上创建一个文件，并设置你喜欢的内容。应用清单并使用 Puppet 检查文件是否已创建并包含你指定的文本。

直接编辑文件并更改内容，然后重新应用 Puppet，检查它是否将文件更改回清单中指定的内容。

# 管理包

Puppet 中的另一个关键资源类型是**package**。手动配置服务器的一个重要部分就是安装包，因此我们在 Puppet 清单中也会经常使用包。尽管每个操作系统都有自己的包格式，而且不同的格式在功能上差异较大，但 Puppet 通过一个单一的`package`类型来表示所有这些可能性。如果你在 Puppet 清单中指定一个给定的包应该被安装，Puppet 会使用适当的包管理器命令，在其运行的任何平台上安装它。

如你所见，Puppet 中所有资源声明都遵循这种格式：

```
RESOURCE_TYPE { TITLE:
  ATTRIBUTE => VALUE,
  ...
}
```

`package`资源没有什么不同。`RESOURCE_TYPE`是`package`，你通常需要指定的唯一属性是`ensure`，而它通常需要的唯一值是`installed`：

```
package { 'cowsay':
  ensure => installed,
}
```

尝试这个示例：

```
sudo puppet apply /examples/package.pp
Notice: Compiled catalog for ubuntu-xenial in environment production in 0.52 seconds
Notice: /Stage[main]/Main/Package[cowsay]/ensure: created
Notice: Applied catalog in 29.53 seconds
```

让我们来看看`cowsay`是否已安装：

```
cowsay Puppet rules!
 _______________
< Puppet rules! >
 ---------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

这可真是个有用的包！

## Puppet 如何应用清单

`package`资源的标题是`cowsay`，因此 Puppet 知道我们在谈论一个名为`cowsay`的包。

`ensure`属性决定了包的安装状态：不出所料，`installed`告诉 Puppet 该包应该被安装。

正如我们在前面的示例中看到的，Puppet 通过逐个检查每个资源并检查它在服务器上的属性与清单中指定的属性是否匹配来处理这个清单。在这种情况下，Puppet 会查找`cowsay`包，看看它是否已安装。它没有，但清单中说应该安装它，因此 Puppet 执行所有必要的操作，以使现实与清单一致，这里意味着安装该包。

### 提示

本书仍处于早期阶段，但你已经可以使用 Puppet 做很多事情了！如果你能安装包并管理文件内容，那么你就能在很大程度上完成你可能需要的任何服务器配置。如果你现在就停止阅读（这会有点可惜，但我们都很忙），你仍然可以使用 Puppet 来自动化你将遇到的很多配置工作。但 Puppet 的功能远不止如此。

## 练习

创建一个清单，使用`package`资源安装你认为在服务器管理中有用的任何软件。以下是一些建议：`tmux`、`sysdig`、`atop`、`htop`和`dstat`。

## 使用 puppet 资源查询资源

如果你想查看 Puppet 认为你已安装的包的版本，可以使用`puppet resource`工具：

```
puppet resource package openssl
package { 'openssl':
  ensure => '1.0.2g-1ubuntu4.8',
}
```

`puppet resource TYPE TITLE` 将输出一个表示系统中指定资源当前状态的 Puppet 清单。如果你省略 `TITLE`，则会得到 `TYPE` 类型的所有资源的清单。例如，如果你运行 `puppet resource package`，你将看到系统上所有已安装包的 Puppet 代码。

### 提示

`puppet resource` 甚至有一个交互式配置功能。要使用它，请运行以下命令：

```
puppet resource -e package openssl

```

如果你运行此命令，Puppet 将生成资源的当前状态清单，并在编辑器中打开它。如果你现在进行更改并保存，Puppet 将应用该清单并对系统进行更改。这是一个有趣的小功能，但如果用这种方式进行整个配置，会相当耗时。

# 服务

第三种最重要的 Puppet 资源类型是**服务**：一个长期运行的进程，或者做一些持续性的工作，或者等待请求并对其做出响应。例如，在大多数系统上，`sshd` 进程会一直运行，并监听 SSH 登录请求。

Puppet 用 `service` 资源类型来建模服务。`service` 资源看起来像下面这个例子（你可以在 `/examples/` 目录下的 `service.pp` 中找到这个例子。从现在起，我将只提供每个例子的文件名，因为它们都在同一个目录下）：

```
service { 'sshd':
  ensure => running,
  enable => true,
}
```

`ensure` 参数控制服务是否应运行。如果它的值是 `running`，那么如你所料，Puppet 将启动服务，如果它没有运行的话。如果你将 `ensure` 设置为 `stopped`，Puppet 将停止服务，如果它正在运行的话。

服务也可以设置为在系统启动时启动，使用 `enable` 参数。如果 `enable` 设置为 `true`，服务将在启动时启动。如果 `enable` 设置为 `false`，则不会启动。一般来说，除非有充分的理由，否则所有服务都应该设置为在启动时启动。

## 使用 `puppet describe` 获取资源帮助

如果你记不住所有不同资源的各种属性，Puppet 有一个内置的帮助功能可以提醒你。例如，运行以下命令：

```
puppet describe service

```

这将提供 `service` 资源的描述，并列出所有属性和允许的值。这适用于所有内置资源类型以及许多第三方模块提供的资源类型。要查看所有可用资源类型的列表，可以运行以下命令：

```
puppet describe --list

```

## 包文件服务模式

给定的软件通常需要这三种 Puppet 资源类型：`package` 资源安装软件，`file` 资源部署软件所需的一个或多个配置文件，`service` 资源运行软件本身。

这是一个使用 MySQL 数据库服务器的例子（`package_file_service.pp`）：

```
package { 'mysql-server':
  ensure => installed,
  notify => Service['mysql'],
}

file { '/etc/mysql/mysql.cnf':
  source => '/examples/files/mysql.cnf',
  notify => Service['mysql'],
}

service { 'mysql':
  ensure => running,
  enable => true,
}
```

`package` 资源确保 `mysql-server` 包被安装。

MySQL 的配置文件是`/etc/mysql/mysql.cnf`，我们使用`file`资源从 Puppet 仓库复制该文件，以便我们可以控制 MySQL 设置。

最后，`service`资源确保`mysql`服务正在运行。

## 通知链接资源

你可能注意到在前面的示例中，`file`资源中新增了一个属性，叫做`notify`：

```
file { '/etc/mysql/mysql.cnf':
  source => '/examples/files/mysql.cnf',
  notify => Service['mysql'],
}
```

这是什么意思呢？假设你对`mysql.cnf`文件进行了修改，并使用 Puppet 应用了这个修改。更新后的文件将被写入磁盘，但因为`mysql`服务已经在运行，它无法知道配置文件已经更改。因此，直到重新启动服务，修改才会生效。不过，如果你在`file`资源上指定了`notify`属性，Puppet 会为你处理这个问题。`notify`的值是通知的资源，具体操作取决于被通知资源的类型。当它是一个服务时，默认操作是重新启动该服务。（我们将在第四章，*理解 Puppet 资源*中了解其他选项。）

通常，在包-文件-服务模式中，文件会通知服务，因此每当 Puppet 更改文件内容时，它会重新启动已通知的服务以应用新配置。如果有多个文件影响该服务，它们应该都通知服务，而 Puppet 足够智能，只会在有多个依赖资源更改时重新启动一次服务。

要通知的资源名称以资源类型（大写）为前缀，后跟资源标题，标题被引号括起来并放在方括号内：`Service['mysql']`。

## 使用`require`进行资源顺序排列

在包-文件-服务示例中，我们声明了三个资源：`mysql-server`包、`/etc/mysql/mysql.cnf`文件和`mysql`服务。如果你考虑一下，它们需要按这个顺序应用。没有安装`mysql-server`包，就不会有`/etc/mysql/`目录来放置`mysql.cnf`文件。没有包或配置文件，`mysql`服务将无法运行。

一个完全合理的问题是，“Puppet 是否按照清单中声明的顺序应用资源？”答案通常是“是”，除非你明确使用`require`属性指定了不同的顺序。

所有资源都支持`require`属性，且其值是清单中声明的另一个资源的名称，指定方式与使用`notify`时相同。下面是再次给出的包-文件-服务示例，这次使用`require`明确指定了资源顺序（`package_file_service_require.pp`）：

```
package { 'mysql-server':
  ensure => installed,
}

file { '/etc/mysql/mysql.cnf':
  source  => '/examples/files/mysql.cnf',
  notify  => Service['mysql'],
  require => Package['mysql-server'],
}

service { 'mysql':
  ensure  => running,
  enable  => true,
  require => [Package['mysql-server'], File['/etc/mysql/mysql.cnf']],
}
```

你可以看到，`mysql.cnf`资源需要`mysql-server`包。`mysql`服务则依赖于其他资源，这些资源以方括号中的数组形式列出。

当资源已经按正确的顺序排列时，您无需使用`require`，因为 Puppet 会按您声明的顺序应用资源。然而，显式指定顺序仍然可能是有用的，尤其是在清单文件中有大量资源时，这样做有助于代码阅读者理解。

在 Puppet 的早期版本中，资源是按较为任意的顺序应用的，因此使用`require`来表达依赖关系非常重要。而现在，您不需要经常使用它，通常只会在旧版代码中遇到。

# 总结

在本章中，我们已经看到了清单是如何由 Puppet 资源构成的。您学习了如何使用 Puppet 的`file`资源来创建和修改文件，如何使用`package`资源安装软件包，以及如何用`service`资源管理服务。我们还了解了常见的包-文件-服务模式，并学习了如何使用资源的`notify`属性向另一个资源发送消息，指示其配置已被更新。我们还讨论了在必要时使用`require`属性来明确资源之间的依赖关系。

您还学会了如何使用`puppet resource`检查系统的当前状态，并使用`puppet describe`获得有关所有 Puppet 资源的命令行帮助。为了检查 Puppet 会对系统进行哪些更改，而不实际进行更改，我们介绍了`--noop`和`--show_diff`选项，用于`puppet apply`命令。

在下一章，我们将学习如何使用版本控制工具 Git 来跟踪您的清单文件，我们将介绍 Git 的基本概念，例如仓库（repo）和提交（commit），并学习如何将代码分发到您将用 Puppet 管理的每一台服务器。

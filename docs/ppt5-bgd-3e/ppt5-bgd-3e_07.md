# 第七章. 掌握模块

|   | *没有大问题，只有很多小问题。* |   |
| --- | --- | --- |
|   | --*亨利·福特* |

在本章中，你将了解 Puppet Forge，这是 Puppet 模块的公共仓库，你将学习如何使用 `r10k` 模块管理工具安装和使用来自 Puppet Forge 的第三方模块。你将看到如何使用三个重要的 Forge 模块的示例：`puppetlabs/apache`、`puppetlabs/mysql` 和 `puppet/archive`。你还将了解 `puppetlabs/stdlib` 提供的一些有用功能，这是 Puppet 的标准库。最后，通过一个完整的示例，你将学习如何从零开始开发自己的 Puppet 模块，如何为模块添加适当的元数据，以及如何将其上传到 Puppet Forge。

![掌握模块](img/8880_07_01.jpg)

# 使用 Puppet Forge 模块

虽然你可以为你想要管理的所有内容编写自己的清单，但通过尽可能使用公共 Puppet 模块，你可以节省大量的时间和精力。在 Puppet 中，**模块** 是一个自包含的可共享、可重用的代码单元，通常设计用来管理某一特定服务或软件，例如 Apache Web 服务器。

## 什么是 Puppet Forge？

**Puppet Forge** 是一个公共的 Puppet 模块仓库，其中许多模块由 Puppet 官方支持并维护，所有这些模块都可以下载和使用。你可以通过以下网址浏览 Forge：

[`forge.puppet.com/`](https://forge.puppet.com/)

使用像 Puppet 这样的成熟工具的一个优势是，存在大量成熟的公共模块，涵盖了你可能需要的大多数常见软件。例如，以下是你可以使用 Puppet Forge 中的公共模块管理的一小部分内容：

+   MySQL/PostgreSQL/SQL Server

+   Apache/Nginx

+   Java/Tomcat/PHP/Ruby/Rails

+   HAProxy

+   Amazon AWS

+   Docker

+   Jenkins

+   Elasticsearch/Redis/Cassandra

+   Git 仓库

+   防火墙（通过 iptables）

## 寻找你需要的模块

Puppet Forge 首页顶部有一个搜索框。在此框中输入你要查找的内容，网站会显示所有与你搜索关键词匹配的模块。通常会有多个结果，那么你该如何决定使用哪个模块呢？

最好的选择是 **Puppet 支持的** 模块，如果有的话。这些模块由 Puppet 官方支持和维护，你可以确信，支持的模块将与广泛的操作系统和 Puppet 版本兼容。支持的模块在搜索结果中会显示黄色的 **SUPPORTED** 标志，或者你可以通过以下网址浏览所有支持模块的列表：

[`forge.puppet.com/modules?endorsements=supported`](https://forge.puppet.com/modules?endorsements=supported)

下一个最佳选择是**Puppet 认证**模块。虽然这些模块没有官方支持，但它们是 Puppet 推荐的，并且已经过检查，确保遵循最佳实践并符合一定的质量标准。认证模块在搜索结果中会显示绿色的**认证通过（APPROVED）**标志，或者你可以通过以下链接浏览所有认证模块的列表：

[`forge.puppet.com/modules?endorsements=approved`](https://forge.puppet.com/modules?endorsements=approved)

假设没有 Puppet 支持或 Puppet 认证的模块，另一种选择模块的有用方法是查看下载数量。在 Puppet Forge 搜索结果页面上选择**最多下载**标签，将根据下载量对结果进行排序，最受欢迎的模块排在前面。当然，下载量最多的模块不一定是最好的，但它们通常是一个不错的起点。

检查模块的最新发布日期也是值得的。如果你正在查看的模块一年多没有更新，最好选择一个维护更活跃的模块（如果有的话）。点击**最新发布**标签将按最新更新的顺序对搜索结果进行排序。

你也可以通过操作系统支持和 Puppet 版本兼容性来过滤搜索结果；这对于找到与您的系统兼容的模块非常有用。

选择好你想要的模块后，就可以将其添加到你的 Puppet 基础架构中了。

## 使用 r10k

过去，许多人习惯直接下载 Puppet Forge 模块，并将其复制到他们的代码库中，实际上是分叉了模块的仓库（至今仍有人这么做）。这种方法有很多缺点。其中一个缺点是，你的代码库中会充斥着不是你写的代码，这会让你很难找到你需要的代码。另一个问题是，测试你的代码与不同版本的公共模块兼容时很困难，你需要创建自己的 Git 分支、重新下载模块等等。你也无法获得 Puppet Forge 模块的未来错误修复和改进，除非你手动更新你的副本。在很多情况下，你需要对模块做一些小的更改或修复，以便在你的环境中使用，这样你的模块版本就会与上游版本产生差异，未来会积累维护问题。

因此，一个更好的模块管理方法是使用 `r10k` 工具，它可以解决这些问题。`r10k` 不直接下载所需的模块并将其添加到代码库中，而是通过一个名为 **Puppetfile** 的特殊文本文件，在每个 Puppet 管理的节点上安装所需的模块。`r10k` 会完全根据 Puppetfile 中的元数据来管理 `modules/` 目录的内容。模块代码从不直接被检查到代码库中，而是在需要时从 Puppet Forge 下载。因此，如果你愿意，你可以保持模块的最新版本，或者将每个模块固定在一个已知与清单兼容的特定版本。

`r10k` 是 Puppet 部署的事实标准模块管理工具，在本书的后续内容中，我们将使用它来管理模块。

在这个例子中，我们将使用 `r10k` 安装 `puppetlabs/stdlib` 模块。示例仓库中的 Puppetfile 列出了本书中将使用的所有模块。它如下所示（稍后我们会详细分析语法）：

```
forge 'http://forge.puppetlabs.com'

mod 'garethr/docker', '5.3.0'
mod 'puppet/archive', '1.3.0'
mod 'puppet/staging', '2.2.0'
mod 'puppetlabs/apache', '2.0.0'
mod 'puppetlabs/apt', '3.0.0'
mod 'puppetlabs/aws', '2.0.0'
mod 'puppetlabs/concat', '4.0.1'
mod 'puppetlabs/docker_platform', '2.2.1'
mod 'puppetlabs/mysql', '3.11.0'
mod 'puppetlabs/stdlib', '4.17.1'
mod 'stahnma/epel', '1.2.2'

mod 'pbg_ntp',
  :git => 'https://github.com/bitfield/pbg_ntp.git',
  :tag => '0.1.4'
```

按照以下步骤操作：

1.  运行以下命令来清空 `modules/` 目录中的内容（确保你已经备份了想要保留的任何内容）：

    ```
    cd /etc/puppetlabs/code/environments/pbg
    sudo rm -rf modules/

    ```

1.  运行以下命令，让 `r10k` 处理这里的示例 Puppetfile 并安装你所请求的模块：

    ```
    sudo r10k puppetfile install --verbose

    ```

`r10k` 会将 Puppetfile 中列出的所有模块下载到 `modules/` 目录中。该目录中的所有模块会被 Puppet 自动加载，并可在你的清单中使用。为了测试 `stdlib` 模块是否正确安装，运行以下命令：

```
sudo puppet apply --environment pbg -e "notice(upcase('hello'))"
Notice: Scope(Class[main]): HELLO
```

`upcase` 函数将其字符串参数转换为大写字母，是 `stdlib` 模块的一部分。如果此功能无法正常工作，说明 `stdlib` 模块未正确安装。如前面的示例所示，我们使用 `--environment pbg` 开关告诉 Puppet 在 `/etc/puppetlabs/code/environments/pbg` 目录中查找代码、模块和数据。

## 理解 Puppetfile

示例 Puppetfile 以以下内容开始：

```
forge 'http://forge.puppetlabs.com'
```

`forge` 语句指定了模块应从哪个仓库获取。

紧接着会出现一组以 `mod` 开头的行：

```
mod 'garethr/docker', '5.3.0'
mod 'puppet/archive', '1.3.0'
mod 'puppet/staging', '2.2.0'
...
```

`mod` 语句指定了模块的名称（`puppetlabs/stdlib`）以及要安装的模块的具体版本（`4.17.0`）。

## 使用 `generate-puppetfile` 管理依赖关系

`r10k` 不会自动管理模块之间的依赖关系。例如，`puppetlabs/apache` 模块依赖于安装 `puppetlabs/stdlib` 和 `puppetlabs/concat`。除非你指定它们，`r10k` 不会自动为你安装这些模块，因此你还需要将它们添加到 Puppetfile 中。

然而，你可以使用 `generate-puppetfile` 工具来找出所需的依赖关系，然后将它们添加到你的 Puppetfile 中。

1.  运行以下命令来安装 `generate-puppetfile` gem：

    ```
    sudo gem install generate-puppetfile

    ```

1.  运行以下命令生成指定模块列表的 Puppetfile（在命令行中列出所有需要的模块，模块之间用空格分隔）：

    ```
    generate-puppetfile puppetlabs/docker_platform
    Installing modules. This may take a few minutes.
    Your Puppetfile has been generated. Copy and paste between the markers:
    =============================================
    forge 'http://forge.puppetlabs.com'

    # Modules discovered by generate-puppetfile
    mod 'garethr/docker', '5.3.0'
    mod 'puppetlabs/apt', '3.0.0'
    mod 'puppetlabs/docker_platform', '2.2.1'
    mod 'puppetlabs/stdlib', '4.17.1'
    mod 'stahnma/epel', '1.2.2'
    =============================================
    ```

1.  运行以下命令生成现有 Puppetfile 的更新版本和依赖项列表：

    ```
    generate-puppetfile -p /etc/puppetlabs/code/environments/pbg/Puppetfile

    ```

这是一个非常有用的工具，既可以用来查找你需要在 Puppetfile 中指定的依赖项，也可以帮助你保持 Puppetfile 与所使用的所有模块的最新版本同步。

# 在清单中使用模块

现在我们知道如何查找和安装公共 Puppet 模块，接下来看看如何使用它们。我们将通过几个示例来演示，使用 `puppetlabs/mysql` 模块来设置 MySQL 服务器和数据库，使用 `puppetlabs/apache` 模块来设置 Apache 网站，以及使用 `puppet/archive` 下载并解压一个压缩包。完成这些示例后，你应该会对如何找到合适的 Puppet 模块、将其添加到 `Puppetfile` 中并通过 `r10k` 部署感到非常有信心。

## 使用 puppetlabs/mysql

按照以下步骤运行 `puppetlabs/mysql` 示例：

1.  如果你之前已经按照 *使用 r10k* 部分的步骤操作，那么所需的模块应该已经安装。如果没有，请运行以下命令安装它：

    ```
    cd /etc/puppetlabs/code/environments/pbg
    sudo r10k puppetfile install

    ```

1.  运行以下命令应用清单：

    ```
    sudo puppet apply --environment=pbg /examples/module_mysql.pp
    Notice: Compiled catalog for ubuntu-xenial in environment pbg in 0.89 seconds
    Notice: /Stage[main]/Mysql::Server::Config/File[/etc/mysql]/ensure: created
    Notice: /Stage[main]/Mysql::Server::Config/File[/etc/mysql/conf.d]/ensure: created
    Notice: /Stage[main]/Mysql::Server::Config/File[mysql-config-file]/ensure: defined content as '{md5}44e7aa974ab98260d7d013a2087f1c77'
    Notice: /Stage[main]/Mysql::Server::Install/Package[mysql-server]/ensure: created
    Notice: /Stage[main]/Mysql::Server::Root_password/Mysql_user[root@localhost]/password_hash: password_hash changed '' to '*F4AF2E5D85456A908E0F552F0366375B06267295'
    Notice: /Stage[main]/Mysql::Server::Root_password/File[/root/.my.cnf]/ensure: defined content as '{md5}4d59f37fc8a385c9c50f8bb3286b7c85'
    Notice: /Stage[main]/Mysql::Client::Install/Package[mysql_client]/ensure: created
    Notice: /Stage[main]/Main/Mysql::Db[cat_pictures]/Mysql_database[cat_pictures]/ensure: created
    Notice: /Stage[main]/Main/Mysql::Db[cat_pictures]/Mysql_user[greebo@localhost]/ensure: created
    Notice: /Stage[main]/Main/Mysql::Db[cat_pictures]/Mysql_grant[greebo@localhost/cat_pictures.*]/ensure: created
    Notice: Applied catalog in 79.85 seconds
    ```

让我们来看一下示例清单（`module_mysql.pp`）。第一部分通过包含 `mysql::server` 类来安装 MySQL 服务器：

```
# Install MySQL and set up an example database
include mysql::server
```

`mysql::server` 类接受多个参数，其中大多数我们暂时不需要关注，但我们希望在此示例中设置其中几个。虽然你可以像设置资源属性一样直接在 Puppet 清单代码中为类参数设置值，但我将展示一种更好的方法：使用 Hiera 的自动参数查找机制。

### 提示

我们在第六章中简要提到过，*使用 Hiera 管理数据*，Hiera 可以为类和模块参数提供值，但它到底是如何工作的呢？当你包含一个类 `x` 并传入一个参数 `y` 时，Puppet 会自动在 Hiera 中搜索任何与 `x::y` 名称匹配的键。如果找到了，它就会使用该值作为参数的值。就像其他 Hiera 数据一样，你可以通过层次结构为不同的节点、角色或操作系统设置不同的值。

在这个示例中，我们的参数设置在示例 Hiera 数据文件（`/etc/puppetlabs/code/environments/pbg/data/common.yaml`）中：

```
mysql::server::root_password: 'hairline-quotient-inside-tableful'
mysql::server::remove_default_accounts: true
```

`root_password` 参数，如你所料，用来设置 MySQL `root` 用户的密码。我们还启用了 `remove_default_accounts`，这是一个安全特性。MySQL 默认带有各种用于测试的用户账户，在生产环境中应当关闭。此参数禁用了这些默认账户。

### 提示

请注意，尽管为了清晰起见我们在此明文指定了密码，但在生产环境的清单中，这些密码应该被加密，就像任何其他凭证或机密数据一样（参见 第六章，*使用 Hiera 管理数据*）。

接下来是资源声明：

```
mysql::db { 'cat_pictures':
  user     => 'greebo',
  password => 'tabby',
  host     => 'localhost',
  grant    => ['SELECT', 'UPDATE'],
}
```

正如你所看到的，这看起来和我们之前使用的内置资源一样，比如 `file` 和 `package` 资源。实际上，`mysql` 模块为 Puppet 添加了一个新的资源类型：`mysql::db`。这个资源表示一个特定的 MySQL 数据库：在我们的例子中是 `cat_pictures`。

资源的标题是数据库的名称，在本例中是 `cat_pictures`。接下来是属性列表。`user`、`password` 和 `host` 属性指定用户 `greebo` 应该被允许从 `localhost` 使用密码 `tabby` 连接到数据库。`grant` 属性指定用户应该具有的 MySQL 权限：对数据库的 `SELECT` 和 `UPDATE` 权限。

当应用此清单时，Puppet 将创建 `cat_pictures` 数据库并设置 `greebo` 用户账户以便访问它。这是管理应用程序的 Puppet 清单中非常常见的模式：通常，应用程序需要某种数据库来存储其状态，并且需要用户凭据来访问它。`mysql` 模块让你非常容易地配置这一点。

现在我们可以看到使用 Puppet Forge 模块的一般原则：

+   我们将模块及其依赖项添加到我们的 `Puppetfile` 中，并使用 `r10k` 部署它。

+   我们在清单中`include`该类，并提供任何需要的参数作为 Hiera 数据。

+   可选地，我们添加一个或多个由模块定义的自定义资源类型的资源声明（在本例中是 MySQL 数据库）。

几乎所有的 Puppet 模块都以类似的方式工作。在接下来的章节中，我们将介绍一些关键模块，这些模块在使用 Puppet 管理服务器时，你很可能会用到。

## 使用 puppetlabs/apache

大多数应用程序都有某种类型的 Web 界面，通常需要一个 Web 服务器，而久负盛名的 Apache 仍然是一个流行的选择。`puppetlabs/apache` 模块不仅安装和配置 Apache，还允许你管理虚拟主机（各个网站，例如应用程序的前端）。

这是一个示例清单，它使用 `apache` 模块来创建一个简单的虚拟主机来服务一个图像文件（`module_apache.pp`）：

```
include apache

apache::vhost { 'cat-pictures.com':
  port          => '80',
  docroot       => '/var/www/cat-pictures',
  docroot_owner => 'www-data',
  docroot_group => 'www-data',
}

file { '/var/www/cat-pictures/index.html':
  content => "<img 
    src='http://bitfieldconsulting.com/files/happycat.jpg'>",
  owner   => 'www-data',
  group   => 'www-data',
}
```

按照以下步骤应用清单：

1.  如果你之前已经按照 *使用 r10k* 章节的步骤操作，所需的模块应该已经安装。如果没有，请运行以下命令进行安装：

    ```
    cd /etc/puppetlabs/code/environments/pbg
    sudo r10k puppetfile install

    ```

1.  运行以下命令来应用清单：

    ```
    sudo puppet apply --environment=pbg /examples/module_apache.pp

    ```

1.  要测试新的网站，请将浏览器指向（对于 Vagrant 用户；如果你没有使用 Vagrant box，请浏览你用 Puppet 管理的服务器上的 `80` 端口）`http://localhost:8080/`

你应该看到一只快乐猫的图片：

![使用 puppetlabs/apache](img/8880_07_02.jpg)

让我们仔细查看清单，了解它是如何工作的。

1.  它从 `include` 声明开始，实际上在服务器上安装了 Apache（`module_apache.pp`）：

    ```
    include apache
    ```

1.  你可以为 `apache` 类设置许多参数，但在本示例中，我们只需要设置一个，像其他示例一样，我们通过示例 Hiera 文件中的 Hiera 数据来设置它：

    ```
    apache::default_vhost: false
    ```

    这将禁用默认的 **Apache 2 测试页面** 虚拟主机。

1.  接下来是一个 `apache::vhost` 资源的声明，它创建了一个 Apache 虚拟主机或网站。

    ```
    apache::vhost { 'cat-pictures.com':
      port          => '80',
      docroot       => '/var/www/cat-pictures',
      docroot_owner => 'www-data',
      docroot_group => 'www-data',
    }
    ```

    资源的标题是虚拟主机响应的域名（`cat-pictures.com`）。`port` 告诉 Apache 听哪个端口上的请求。`docroot` 指定 Apache 在服务器上查找网站文件的目录路径。最后，`docroot_owner` 和 `docroot_group` 属性指定应该拥有 `docroot/` 目录的用户和组。

1.  最后，我们创建一个 `index.html` 文件，为网站添加一些内容，在这种情况下是一个快乐猫咪的图片。

    ```
    file { '/var/www/cat-pictures/index.html':
      content => "<img 
        src='http://bitfieldconsulting.com/files/happycat.jpg'>",
      owner   => 'www-data',
      group   => 'www-data',
    }
    ```

### 注意

请注意，Vagrant 虚拟机中的端口 `80` 映射到本地机器上的端口 `8080`，因此浏览 `http://localhost:8080` 相当于直接访问 Vagrant 虚拟机上的端口 `80`。如果由于某些原因需要更改此端口映射，请编辑您的 `Vagrantfile`（在 Puppet 初学者指南仓库中），并查找以下行：

```
config.vm.network "forwarded_port", guest: 80, host: 8080
```

根据需要更改这些设置，并在本地机器的 PBG 仓库目录中运行以下命令：

```
vagrant reload

```

## 使用 puppet/archive

从软件包安装软件是常见的任务，但有时你也需要从归档文件中安装软件，例如 tarball（`.tar.gz` 文件）或 ZIP 文件。`puppet/archive` 模块在这方面非常有帮助，因为它提供了一种简单的方法来从互联网下载归档文件，并且还可以为你解压这些文件。

在下面的示例中，我们将使用 `puppet/archive` 模块来下载并解压流行的 WordPress 博客软件的最新版本。按照以下步骤应用清单：

1.  如果你之前已经按照 *使用 r10k* 部分中的步骤操作，所需的模块应该已经安装。如果没有，请运行以下命令来安装它：

    ```
    cd /etc/puppetlabs/code/environments/pbg
    sudo r10k puppetfile install

    ```

1.  运行以下命令以应用清单：

    ```
    sudo puppet apply --environment=pbg /examples/module_archive.pp
    Notice: Compiled catalog for ubuntu-xenial in environment production in 2.50 seconds
    Notice: /Stage[main]/Main/Archive[/tmp/wordpress.tar.gz]/ensure: download archive from https://wordpress.org/latest.tar.gz to /tmp/wordpress.tar.gz and extracted in /var/www with cleanup
    ```

与本章之前的模块不同，`archive` 不需要安装任何东西，因此我们不需要包含该类。你只需要声明一个 `archive` 资源。让我们详细看看这个示例，了解它是如何工作的（`module_archive.pp`）：

```
archive { '/tmp/wordpress.tar.gz':
  ensure       => present,
  extract      => true,
  extract_path => '/var/www',
  source       => 'https://wordpress.org/latest.tar.gz',
  creates      => '/var/www/wordpress',
  cleanup      => true,
}
```

1.  标题给出了你希望归档文件下载到的位置（`/tmp/wordpress.tar.gz`）。假设你在解压后不需要保留归档文件，通常最好将其放在 `/tmp` 中。

1.  `extract` 属性决定 Puppet 是否应该解压归档文件；通常情况下，这应该设置为 `true`。

1.  `extract_path`属性指定了解压归档内容的位置。在本例中，将其解压到`/var/www/`的子目录是有意义的，但根据归档的性质，这个路径会有所不同。例如，如果归档文件包含需要编译和安装的软件，最好将其解压到`/tmp/`，这样在下次重启后，文件会被自动清理。

1.  `source`属性告诉 Puppet 从哪里下载归档文件，通常（如本例所示）是一个网页 URL。

1.  `creates`属性的作用与`exec`资源中的`creates`完全相同，我们在第四章，*理解 Puppet 资源*中有详细讲解。它指定了一个文件，这个文件会在解压归档文件时创建。如果这个文件已经存在，Puppet 就知道归档文件已经被解压，因此不需要再次解压。

1.  `cleanup`属性告诉 Puppet 是否在解压完归档文件后删除该文件。通常，这会设置为`true`，除非你需要保留归档文件，或者你根本不需要解压它。

### 提示

一旦文件被`cleanup`删除，Puppet 在下次应用清单时不会重新下载归档文件`/tmp/wordpress.tar.gz`，即使它的`ensure => present`。`creates`子句告诉 Puppet 这个归档文件已经下载并解压过。

# 探索标准库

`puppetlabs/stdlib`是最古老的 Puppet Forge 模块之一，也是官方的 Puppet 标准库。我们在本章前面简要地看过它，当时用它作为使用`r10k`安装模块的示例，但现在让我们更仔细地看看它提供了什么功能，以及你可以在哪里使用它。

标准库的目标并不是管理某些特定的软件或文件格式，而是提供一组可以在任何 Puppet 代码中使用的功能和资源。因此，编写良好的 Forge 模块会利用标准库的功能，而不是自己实现相同功能的工具函数。

你应该在自己的 Puppet 代码中做到这一点：当你需要某个特定功能时，先检查标准库，看看它是否能解决你的问题，而不是自己实现。

在尝试本节中的示例之前，请确保按照以下步骤安装`stdlib`模块：如果你之前按照*使用 r10k*一节的步骤操作过，所需的模块已经安装。如果没有，请运行以下命令来安装它：

```
cd /etc/puppetlabs/code/environments/pbg
sudo r10k puppetfile install

```

## 安全安装软件包与 ensure_packages

如你所知，你可以使用`package`资源来安装软件包，像这样（`package.pp`）：

```
package { 'cowsay':
  ensure => installed,
}
```

如果你在清单的另一个类中安装了相同的软件包，会发生什么呢？Puppet 会拒绝运行，并出现类似这样的错误：

```
Error: Evaluation Error: Error while evaluating a Resource Statement, Duplicate declaration: Package[cowsay] is already declared in file /examples/package.pp:1; cannot redeclare at /examples/package.pp:4 at /examples/package.pp:4:1 on node ubuntu-xenial
```

如果你的两个类都确实需要这个包，那么你就会遇到问题。你可以创建一个简单声明包的类，然后将它包含在这两个类中，但对于一个包来说，这会带来很大的开销。更糟糕的是，如果重复声明存在于第三方模块中，可能无法，也不建议修改该代码。

我们需要的是一种声明包的方法，如果该包也在其他地方声明，则不会导致冲突。标准库在`ensure_packages()`函数中提供了这一功能。调用`ensure_packages()`并传入一个包名数组，如果这些包尚未在其他地方声明，它们将被安装（`package_ensure.pp`）：

```
ensure_packages(['cowsay'])
```

要应用这个示例，请运行以下命令：

```
sudo puppet apply --environment=pbg /examples/package_ensure.pp

```

你可以以相同的方式尝试本章中的所有剩余示例。确保将`--environment=pbg`开关传递给`puppet apply`，因为所需的模块仅在`pbg`环境中安装。

如果你需要向`package`资源传递额外的属性，可以将它们作为第二个参数通过哈希传递给`ensure_packages()`，像这样（`package_ensure_params.pp`）：

```
ensure_packages(['cowsay'],
  {
    'ensure' => 'latest',
  }
)
```

为什么这比直接使用`package`资源更好？当你在多个地方声明相同的`package`资源时，Puppet 会给出错误消息并拒绝执行。然而，如果该包是通过`ensure_packages()`声明的，Puppet 将成功执行。

由于它提供了一种安全的方式来安装包而不会发生资源冲突，你应该始终使用`ensure_packages()`，而不是内置的`package`资源。如果你正在编写公开发布的模块，这肯定是必不可少的，但我建议你在所有代码中都使用它。我们将在本书的其余部分使用它来管理包。

## 使用`file_line`就地修改文件

通常，在使用 Puppet 管理配置时，我们希望改变或添加文件中的某一行，而不必承担用 Puppet 管理整个文件的开销。有时候，管理整个文件本来就不可行，因为另一个 Puppet 类或另一个应用程序可能已经在管理它。我们可以编写一个`exec`资源来修改文件，但标准库提供了一个正是为此目的而设计的资源类型：`file_line`。

这是使用`file_line`资源向系统配置文件添加一行的示例（`file_line.pp`）：

```
file_line { 'set ulimits':
  path => '/etc/security/limits.conf',
  line => 'www-data         -       nofile          32768',
}
```

如果有可能某个其他 Puppet 类或应用程序需要修改目标文件，请使用`file_line`，而不是直接管理文件。这可以确保你的类不会与任何其他试图控制文件的操作发生冲突。

你还可以使用`file_line`查找并修改现有行，使用`match`属性（`file_line_match.pp`）：

```
file_line { 'adjust ulimits':
  path  => '/etc/security/limits.conf',
  line  => 'www-data         -       nofile          9999',
  match => '^www-data .* nofile',
}
```

`match` 的值是一个正则表达式，如果 Puppet 在文件中找到匹配此表达式的行，它将用 `line` 的值替换该行。（如果你需要更改多行，请将 `multiple` 属性设置为 `true`，否则当多行匹配表达式时，Puppet 会报错。）

你还可以使用 `file_line` 来删除文件中的某一行，如果该行存在的话（`file_line_absent.pp`）：

```
file_line { 'remove dash from valid shells':
  ensure            => absent,
  path              => '/etc/shells',
  match             => '^/bin/dash',
  match_for_absence => true,
}
```

请注意，当使用 `ensure => absent` 时，如果希望 Puppet 实际删除匹配的行，还需要将 `match_for_absence` 属性设置为 `true`。

## 介绍其他一些有用的函数

`grep()` 函数将在数组中搜索正则表达式，并返回所有匹配的元素（`grep.pp`）：

```
$values = ['foo', 'bar', 'baz']
notice(grep($values, 'ba.*'))

# Result: ['bar', 'baz']
```

`member()` 和 `has_key()` 函数如果给定的值分别在指定的数组或哈希中，则返回 `true`（`member_has_key.pp`）：

```
$values = [
  'foo',
  'bar',
  'baz',
]
notice(member($values, 'foo'))

# Result: true

$valuehash = {
  'a' => 1,
  'b' => 2,
  'c' => 3,
}
notice(has_key($valuehash, 'b'))

# Result: true
```

`empty()` 函数在其参数为空字符串、数组或哈希时返回 `true`（`empty.pp`）：

```
notice(empty(''))

# Result: true

notice(empty([]))

# Result: true

notice(empty({}))

# Result: true
```

`join()` 函数将提供的数组元素连接成一个字符串，使用指定的分隔符字符或字符串（`join.pp`）：

```
$values = ['1', '2', '3']
notice(join($values, '... '))

# Result: '1... 2... 3'
```

`pick()` 函数是一个很好的方式，当变量为空时提供默认值。它接受任意数量的参数，并返回第一个非未定义或非空的参数（`pick.pp`）：

```
$remote_host = ''
notice(pick($remote_host, 'localhost'))

# Result: 'localhost'
```

有时你需要解析来自外部源的结构化数据。如果数据是 YAML 格式的，可以使用 `loadyaml()` 函数将其读取并解析为本地 Puppet 数据结构（`loadyaml.pp`）：

```
$db_config = loadyaml('/examples/files/database.yml')
notice($db_config['development']['database'])

# Result: 'dev_db'
```

`dirname()` 函数非常有用，如果你有一个指向文件或目录的路径字符串，并且你想引用它的父目录，例如将其声明为 Puppet 资源（`dirname.pp`）：

```
$file = '/var/www/vhosts/mysite'
notice(dirname($file))

# Result: '/var/www/vhosts'
```

## Pry 调试器

当 Puppet 清单没有按预期工作时，故障排除可能会很困难。使用 `notice()` 打印变量和数据结构的值可能有帮助，或者运行 `puppet apply -d` 查看详细的调试输出，但如果这些方法都失败了，你可以使用标准库的 `pry()` 方法进入交互式调试会话（`pry.pp`）：

```
pry()
```

在 Puppet 的上下文中安装了 `pry` gem 后，你可以在代码的任何位置调用 `pry()`。当你应用清单时，Puppet 会在调用 `pry()` 函数的地方启动一个交互式 Pry shell。你可以运行 `catalog` 命令来检查 Puppet 的目录，该目录包含当前在清单中声明的所有资源：

```
sudo puppet apply --environment=pbg /examples/pry_install.pp
sudo puppet apply --environment=pbg /examples/pry.pp
...
[1] pry(#<Puppet::Parser::Scope>)> catalog
=> #<Puppet::Resource::Catalog:0x00000001bbcf78
...
 @resource_table={["Stage", "main"]=>Stage[main]{}, ["Class", "Settings"]=>Class[Settings]{}, ["Class", "main"]=>Class[main]{}},
 @resources=[["Stage", "main"], ["Class", "Settings"], ["Class", "main"]],
...
```

完成目录检查后，键入 `exit` 退出调试器并继续应用 Puppet 清单。

# 编写你自己的模块

如我们所见，Puppet 模块是一种将一组相关代码和资源组合在一起的方式，执行某些特定任务，比如管理 Apache web 服务器或处理归档文件。但你到底如何创建一个模块呢？在这一部分，我们将开发一个自己的模块来管理 NTP 服务，这对于大多数系统管理员来说是最简单的将服务器时钟与互联网时间标准同步的方式。（当然，你不必为此编写自己的模块，因为 Puppet Forge 上已经有一个非常好的现成模块。但为了学习的目的，我们还是自己写一个。）

## 为你的模块创建一个仓库

如果我们打算将新模块与从 Puppet Forge 安装的其他模块一起使用，那么我们应该仅为我们的模块创建一个新的 Git 仓库。然后我们可以将其详细信息添加到 Puppetfile 中，并使用`r10k`为我们安装它。

如果你已经完成了第三章，*使用 Git 管理你的 Puppet 代码*，你应该已经创建了一个 GitHub 帐户。如果没有，请前往该章节并按照*创建 GitHub 帐户和项目*部分的指示操作，然后再继续：

1.  登录到你的 GitHub 帐户，点击**开始项目**按钮。

1.  在**创建新仓库**页面，输入一个合适的仓库名称（我使用`pbg_ntp`作为 Puppet 初学者指南的 NTP 模块名称）。

1.  勾选**初始化该仓库时添加 README**框。

1.  点击**创建仓库**。

1.  GitHub 将引导你进入新仓库的项目页面。点击**克隆或下载**按钮。如果你使用的是带 SSH 密钥的 GitHub，如我们在第三章，*使用 Git 管理你的 Puppet 代码*中讨论过的那样，请复制**通过 SSH 克隆**链接。否则，点击**使用 HTTPS**并复制**通过 HTTPS 克隆**链接。

1.  在你的计算机上，或者在你开发 Puppet 代码的地方，运行以下命令以克隆新仓库（请使用你在上一步复制的 GitHub URL，而不是这里的 URL）：

    ```
    git clone https://github.com/bitfield/pbg_ntp.git

    ```

当克隆操作成功完成后，你就准备好开始创建你的新模块了。

## 编写模块代码

正如你在查看已安装的 Puppet Forge 模块时会看到的那样，模块具有标准的目录结构。这是为了让 Puppet 能够自动找到模块中的清单文件、模板和其他组件。虽然复杂的模块有许多子目录，但在本例中，我们只关心`manifests`和`files`。在本部分中，我们将创建必要的子目录，编写管理 NTP 的代码，并添加一个配置文件，代码将安装它。

### 注意

该模块的所有代码和文件都可以在以下 GitHub 仓库的 URL 中找到：

[`github.com/bitfield/pbg_ntp`](https://github.com/bitfield/pbg_ntp)

1.  运行以下命令以创建`manifests`和`files`子目录：

    ```
    cd pbg_ntp
    mkdir manifests
    mkdir files

    ```

1.  创建文件 `manifests/init.pp`，并包含以下内容：

    ```
    # Manage NTP
    class pbg_ntp {
      ensure_packages(['ntp'])

      file { '/etc/ntp.conf':
        source  => 'puppet:///modules/pbg_ntp/ntp.conf',
        notify  => Service['ntp'],
        require => Package['ntp'],
      }

      service { 'ntp':
        ensure => running,
        enable => true,
      }
    }
    ```

1.  创建文件 `files/ntp.conf`，并包含以下内容：

    ```
    driftfile /var/lib/ntp/ntp.drift

    pool 0.ubuntu.pool.ntp.org iburst
    pool 1.ubuntu.pool.ntp.org iburst
    pool 2.ubuntu.pool.ntp.org iburst
    pool 3.ubuntu.pool.ntp.org iburst
    pool ntp.ubuntu.com

    restrict -4 default kod notrap nomodify nopeer noquery limited
    restrict -6 default kod notrap nomodify nopeer noquery limited
    restrict 127.0.0.1
    restrict ::1
    ```

1.  运行以下命令将你的更改添加、提交并推送到 GitHub（如果没有使用 SSH 密钥，你需要输入你的 GitHub 用户名和密码）：

    ```
    git add manifests/ files/
    git commit -m 'Add module manifest and config file'
    [master f45dc50] Add module manifest and config file
     2 files changed, 29 insertions(+)
     create mode 100644 files/ntp.conf
     create mode 100644 manifests/init.pp
    git push origin master

    ```

请注意，`ntp.conf` 文件的 `source` 属性如下所示：

```
puppet:///modules/pbg_ntp/ntp.conf
```

我们以前没有见过这种文件来源，它通常仅在模块代码中使用。`puppet://` 前缀表示文件来自 Puppet 仓库内，而路径 `/modules/pbg_ntp/` 告诉 Puppet 在 `pbg_ntp` 模块中查找它。尽管 `ntp.conf` 文件实际上位于目录 `modules/pbg_ntp/files/` 中，但我们无需指定 `files` 部分：这是默认的，因为这是一个 `file` 资源。（这不仅仅是你：这让每个人都感到困惑。）

我们不是通过 `package` 资源安装 `ntp` 包，而是使用标准库中的 `ensure_packages()`，正如本章之前所述。

## 创建并验证模块元数据

每个 Puppet 模块应该在其顶层目录中包含一个名为 `metadata.json` 的文件，该文件包含有关该模块的有用信息，供模块管理工具使用，包括 Puppet Forge。

创建文件`metadata.json`，并包含以下内容（请使用你自己的名字和 GitHub URL）：

```
{
  "name": "pbg_ntp",
  "version": "0.1.1",
  "author": "John Arundel",
  "summary": "Example module to manage NTP",
  "license": "Apache-2.0",
  "source": "https://github.com/bitfield/pbg_ntp.git",
  "project_page": "https://github.com/bitfield/pbg_ntp",
  "tags": ["ntp"],
  "dependencies": [
    {"name":"puppetlabs/stdlib",
      "version_requirement":">= 4.17.0 < 5.0.0"}
  ],
  "operatingsystem_support": [
    {
      "operatingsystem": "Ubuntu",
      "operatingsystemrelease": [ "16.04" ]
    }
  ]
}
```

这些大多是比较显而易见的。`tags` 是一个字符串数组，它将帮助人们在 Puppet Forge 上找到你的模块，通常会使用你的模块所管理的软件或服务的名称来标记模块（在本例中是 `ntp`）。

如果你的模块依赖于其他 Puppet 模块，这很有可能（例如，本模块依赖于 `puppetlabs/stdlib` 提供 `ensure_packages()` 函数），你可以使用 `dependencies` 元数据来记录这一点。你应该列出模块使用的每个模块，以及与模块兼容的最早和最新版本。（如果当前发布的版本有效，请指定下一个主要版本作为最新版本。例如，如果你的模块与 `stdlib` 版本 4.17.0 兼容且这是当前最新版本，请指定 5.0.0 作为最高兼容版本。）

最后，`operatingsystem_support` 元数据允许你指定模块支持的操作系统及其版本。这对于那些寻找与自己操作系统兼容的 Puppet 模块的人非常有帮助。如果你知道你的模块与 Ubuntu 16.04 兼容，就像示例模块一样，你可以在 `operatingsystem_support` 部分列出这一点。你的模块支持的操作系统越多越好，所以如果可能的话，在其他操作系统上测试你的模块，并在确认它们可以正常工作后将其列入元数据中。

### 提示

关于模块元数据的详细信息以及如何使用它，请参阅 Puppet 文档：

[`docs.puppet.com/puppet/latest/reference/modules_metadata.html`](https://docs.puppet.com/puppet/latest/reference/modules_metadata.html)

确保模块的元数据正确非常重要，有一个小工具可以帮助你完成这项工作，叫做`metadata-json-lint`。

1.  运行以下命令安装`metadata-json-lint`并检查你的元数据：

    ```
    sudo gem install metadata-json-lint
    metadata-json-lint metadata.json

    ```

1.  如果`metadata-json-lint`没有产生输出，说明你的元数据是有效的，可以继续进行下一步。如果看到错误信息，修复问题后再继续。

1.  运行以下命令将你的元数据文件添加、提交并推送到 GitHub：

    ```
    git add metadata.json
    git commit -m 'Add metadata.json'
    git push origin master

    ```

## 给模块打标签

就像使用第三方 Puppet Forge 模块一样，能够在 Puppetfile 中指定要安装的模块的确切版本是非常重要的。你可以通过使用 Git 标签，将版本标签附加到模块仓库中的特定提交上来实现这一点。随着模块的进一步开发并发布新版本，你可以为每个发布添加一个新的标签。

对于模块的首次发布，根据元数据，它的版本是 0.1.1，运行以下命令来创建并推送发布标签：

```
git tag -a 0.1.1 -m 'Release 0.1.1'
git push origin 0.1.1

```

## 安装你的模块

我们可以使用`r10k`来安装我们的新模块，就像我们使用 Puppet Forge 模块一样，唯一的不同之处是，由于我们的模块尚未在 Puppet Forge 上，因此仅在 Puppetfile 中指定模块名称是不够的；我们需要提供 Git URL，以便`r10k`可以从 GitHub 克隆该模块。

1.  将以下`mod`语句添加到你的 Puppetfile 中（用你的 GitHub URL 代替我的）：

    ```
    mod 'pbg_ntp',
      :git => 'https://github.com/bitfield/pbg_ntp.git',
      :tag => '0.1.1'
    ```

1.  因为该模块还依赖于`puppetlabs/stdlib`，所以也要添加这个`mod`语句：

    ```
    mod 'puppetlabs/stdlib', '4.17.0'
    ```

1.  现在以正常方式使用`r10k`安装模块：

    ```
    sudo r10k puppetfile install --verbose

    ```

`r10k`可以从你有访问权限的任何 Git 仓库安装模块；你只需要在 Puppetfile 中的`mod`语句中添加`:git`和`:tag`参数。

## 应用你的模块

现在你已经创建、上传并安装了模块，我们可以在清单中使用它：

```
sudo puppet apply --environment=pbg -e 'include pbg_ntp'

```

如果你使用的是 Vagrant 盒子或较新的 Ubuntu 版本，服务器很可能已经在运行 NTP，因此你在 Puppet 应用时唯一看到的变化将是`ntp.conf`文件。尽管如此，它仍然确认了你的模块有效。

## 更复杂的模块

当然，我们开发的模块是一个非常简单的示例。然而，它展示了 Puppet 模块的基本要求。随着你成为一个更高级的 Puppet 开发者，你将创建和维护更加复杂的模块，类似于你从 Puppet Forge 下载并使用的那些模块。

现实世界中的模块通常包含以下一个或多个组件：

+   多个清单文件和子目录

+   参数（可以直接提供或从 Hiera 数据中查找）

+   自定义事实和自定义资源类型与提供者

+   示例代码，展示如何使用该模块

+   开发者可以用来验证更改的规范和测试

+   依赖其他模块（必须在模块的元数据中声明）

+   支持多个操作系统

### 提示

你可以在 Puppet 文档中找到关于模块及模块高级功能的更详细信息：

[`docs.puppet.com/puppet/latest/reference/modules_fundamentals.html`](https://docs.puppet.com/puppet/latest/reference/modules_fundamentals.html)

## 上传模块到 Puppet Forge

上传模块到 Puppet Forge 非常简单：你只需要注册一个账户，使用 `puppet module build` 命令创建模块的归档文件，然后通过 Puppet Forge 网站上传即可。

然而，在决定编写一个模块之前，你应该先检查是否已经有一个模块在 Puppet Forge 上可以实现你的需求。写本文时，Puppet Forge 上已有超过 4,500 个模块，因此你很有可能可以使用现有的 Puppet Forge 模块，而不是编写自己的模块。如果已有模块存在，贡献一个新的模块只会增加用户选择的难度。例如，目前有 150 个管理 Nginx Web 服务器的模块。显然，这至少多了 149 个，所以只有在确认 Puppet Forge 上没有类似模块时，才提交一个新模块。

如果已有模块覆盖了你想管理的软件，但不支持你的操作系统或版本，可以考虑改进这个模块，而不是开始一个新模块。联系模块作者，看看你是否能帮助改进他们的模块，并扩展对你操作系统的支持。类似地，如果你发现模块中有 bug 或想要对其进行改进，可以在模块的 issue 跟踪系统中打开一个问题（如果有的话），或者在 GitHub 上分叉该模块仓库（如果模块有 GitHub 版本），或者联系作者，了解如何提供帮助。绝大多数 Puppet Forge 模块都是由志愿者编写和维护的，因此你的支持和贡献将有利于整个 Puppet 社区。

如果你不想为现有模块进行分支或贡献，可以考虑编写一个小的包装模块，扩展或覆盖现有模块，而不是从头开始创建一个新模块。

如果你决定编写并发布自己的模块，尽量使用标准库中的功能，例如 `ensure_packages()`。这将使你的模块有更高的兼容性，与其他 Forge 模块兼容的机会更大。

### 注意

如果你想为 Puppet 模块社区做出更多贡献，可以考虑加入 Vox Pupuli 小组，该小组维护着超过一百个开源 Puppet 模块：

[`voxpupuli.org/`](https://voxpupuli.org/)

# 总结

在本章中，我们了解了 Puppet 模块，包括 Puppet Forge 模块库的介绍。我们已了解如何搜索所需的模块以及如何评估结果，包括 **Puppet Approved** 和 **Puppet Supported** 模块、操作系统支持以及下载量。

我们已经介绍了如何使用`r10k`工具下载和管理基础设施中的 Puppet 模块，并且如何在 Puppetfile 中指定所需的模块和版本。我们通过详细的示例展示了如何使用三个重要的 Forge 模块：`puppetlabs/apache`、`puppetlabs/mysql`和`puppet/archive`。

引入 Puppet 标准库后，我们介绍了如何使用`ensure_packages()`避免模块之间的包冲突，`file_line`资源（用于对配置文件进行行级编辑），以及一系列用于数据操作的有用函数，同时还介绍了 Pry 调试器。

为了充分理解模块的工作原理，我们从零开始开发了一个简单的模块来管理 NTP 服务，该模块托管在自己的 Git 仓库中，并通过 Puppetfile 和`r10k`进行管理。我们了解了模块所需的元数据，以及如何使用`metadata-json-lint`创建和验证这些元数据。

最后，我们探讨了一些更复杂模块的特性，讨论了将模块上传到 Puppet Forge，并概述了在决定是开始一个新模块还是扩展和改进现有模块时需要考虑的事项。

在下一章中，我们将探讨如何将 Puppet 代码组织为类，如何将参数传递给类，如何创建定义的资源类型，以及如何使用角色、配置文件来结构化清单，并且如何使用 Hiera 数据在节点上包含类。

# 第九章：Puppet 角色与配置文件

现在我们已经对 Puppet DSL 及其概念有了完整的了解，是时候看看如何基于 Puppet 构建反映你基础设施设置和需求的实现了。

在 Puppet 的早期，通常的做法是将资源和变量添加到节点分类中。这通常导致代码重复，使重构几乎不可能。这种模式大多反映了通常的管理员工作，即配置单个系统。

为了避免难以管理和维护的代码，Puppet 模块周围出现了一个社区。该社区致力于将系统的技术部分实现为 Puppet 模块。模块的好处在于可以通过参数重用，并且由于共享的努力，可以更快地修复错误和实现新功能。

由于我们现在有大量可用的模块，我们必须重新思考结合模块的节点分类模式。在这里，角色和配置文件模式发挥了作用。

本章将涵盖以下主题：

+   技术组件模块

+   在配置文件中实现组件

+   从配置文件构建角色

+   业务用例和节点分类

+   将代码放置在 Puppet 服务器上

# 技术组件模块

直到现在，我们一直将模块称为严格的目录结构，包含类、静态文件、模板和扩展。现在我们必须区分上游或通用模块与我们的平台实现模块。

现在负责特定技术组件的模块被称为技术组件模块。技术组件本身是为在系统上运行的某些软件（如 Nginx、Postgres 或 LDAP）配置的一组配置。

一直存在一个问题，即模块是否是技术组件模块。有一些模式可以帮助你识别技术组件模块：

+   在上游开发，拥有活跃的社区

+   开源，包含`README`和`LICENSE`文件

+   仅管理所需内容

+   清晰描述的入口类，包含采纳和可重用的参数

+   允许与其他技术组件类堆叠在一起

+   使用与配置技术相关的模块名称

+   通常支持多个操作系统

+   拥有公共信息，如包和配置文件名

+   不包含私人数据，例如内部 DNS 服务器 IP

# 在配置文件中实现组件

Puppet 代码如果不是从上游获取，而是内部开发的，用于描述你的基础设施，通常是资源和上游类的实现。这种实现被称为配置文件类。

从技术上讲，配置文件是一个包含类的模块，通常还会有参数、定义、文件和模板。在极少数情况下，将自定义事实或自定义函数也包含在配置文件中可能是有价值的。

在一个配置文件内部，指定数据和资源。数据可以是静态数据，适用于整个平台，或者放入 Hiera 中。资源可以是任何东西，比如类、文件、包和服务。

将这些内容组合成一个配置文件，构建出另一个抽象层：

+   数据通过 Hiera 进行抽象。

+   CLI 命令通过资源类型进行抽象。

+   资源类型通过技术组件模块进行抽象。

+   技术组件模块通过配置文件进行抽象。

当你在互联网上搜索配置文件时，你大多数会找到关于 Apache、MySQL 和 WordPress 安装的简短介绍。

配置文件不适合公开发布，因为它们通常包含关于你基础设施的私人信息。相反，你需要自己开发配置文件。

让我们从一个例子开始：一个基于 `phpmyadmin` 的数据库管理系统。

系统由多个基础技术组件组成：远程登录、备份、防火墙、Web 服务器、数据库、`php` 和 `phpmyadmin`。这些组件中的每一个都由上游开发的技术组件模块进行管理。我们希望的实现方式是通过配置文件完成的：

```
# Class profile::login
#
# manages ssh access
# company policy requires the following settings:
# - forbid root login
# - forbid x11 forwarding
# - allow login based on group (admins)
#
# We have several other settings which are put into our own
# sshd_config template file
#
class profile::login {
  class { 'ssh':
    sshd_x11_forwarding     => false,
    sshd_config_template    => 
    epp('profile/login/sshd_config.epp'),
    sshd_config_allowgroups => ['admins'],
    permit_root_login       => 'no',
  }
}
```

不必为每个不同的设置写一个单独的配置文件，你可以选择为配置文件添加参数并利用 Hiera 查找，或者将组件堆叠在一起：

```
# Class profile::login::secure
#
# Reuses profile::login classes
# adds known_host_file based on template
#
class profile::login::secure {
  include profile::ssh
  file { '/etc/ssh/ssh_known_hosts':
    ensure  => file,
    content => epp('profile/login/ssh_known_hosts.epp'),
  }
}
```

对 MySQL 使用相同的模式。主`mysql`配置文件仅使用`puppetlabs-mysql`模块安装一个 MySQL 实例：

```
# Class profile::database::mysql
#
# Needs data in hiera:
# - mysql_root_password (String), defaults to 123456
# - mysql_database (Hash)
#
class profile::database::mysql {
  $mysql_root_password = lookup('mysql_root_password', String, 'first', '123456')
  $mysql_database = lookup('mysql_database', Hash, 'deep', '')
  class { 'mysql':
    root_password           => $mysql_root_password,
    remove_default_accounts => true,
  }
  class { 'mysql::bindings':
    php_enable => true,
  }
  $mysql_database.each |$db, $options| {
    mysql::db { $db:
      * => $options,
    }
  }
}
```

相同的模式适用于使用上游模块安装 `php` 和 `phpmyadmin`。

```
# Class profile::scripting::php
#
# uses puppet/php mdoule
# basic installation only
#
class profile::scripting::php {
  include ::php
} 
# Class profile::apps::phpmyadmin
#
# uses jlondon/phpmyadmin
# configures the application and application vhost
#
class profile::apps::phpmyadmin {
  class { 'phpmyadmin': }
  phpmyadmin::server{ 'default': }
  phpmyadmin::vhost { 'internal.domain.net':
    vhost_enabled => true,
    priority      => '20',
    docroot       => $phpmyadmin::params::doc_path,
    ssl           => true,
   }
}
# Class profile::apps::phpmyadmin::db
#
# uses jlondon/phpmyadmin module
# exports the setting for phpmyadmin
# 
class profile::apps::phpmyadmin::db {
  @@phpmyadmin::servernode { "${::ipaddress}":
    server_group => 'default',
  }
}
```

在目录结构中对配置文件进行分组没有技术上的必要。想象一下一个基础设施，包含大量配置文件，甚至是大量相似的配置文件，如 PostgreSQL、MySQL、MariaDB、Galera Cluster、Oracle DB 和 MSSQL。在这种情况下，分组比使用平面文件结构更合适，因为许多平面文件会导致难以阅读的目录结构：

```
profile/
|- manifests/
|   |- apps/
|   |   |─ phpmyadmin/
|   |   |   \- db.pp
|   |   \- phpmyadmin.pp
|   |- database/
|   |   \- mysql.pp
|   |- login/
|   |   \- secure.pp
|   |- login.pp
|   \- scripting/
|   \─ php.pp
\- templates/
    \- login/
        |- sshd_config.epp
        \- ssh_known_hosts.epp 
```

# 业务使用场景和节点分类。

随着所有实现的就位，我们开始考虑如何进行节点分类。

有多种可用的选项，哪个是最佳解决方案，主要取决于你的平台。

当你有一个非常多样化的平台时，角色作为另一层抽象的概念并不是非常有用，因为它通常会导致代码重复。在这种情况下，大多数人选择使用配置文件进行节点分类。

当你有大量相同配置的系统时，最好采用角色模式并按业务使用场景对系统进行分类。

业务使用场景允许你描述系统时，不仅仅是按它们的功能来描述，更是按它们的使用目的来描述。

想一想`phpmyadmin`的安装。根据使用场景和业务方的不同，可能会有不同的分类名称：

技术人员会使用“`数据库控制面板`”这个术语。如果`phpmyadmin`安装是由销售团队使用的，可能他们会将这个系统称为`crm 数据管理系统`。

最好的解决方案是确定应用程序的相关方，并询问该应用程序的用途。这样可以带来一个积极的副作用，即获得所有业务使用案例的概览。如果你发现一个系统有多个业务使用案例，那么在系统发生故障时，就能更容易理解其业务影响。

过去，人们将许多应用程序堆叠到一个系统上，以便最大化硬件使用率。这些是单个节点上有多个业务使用案例的基础设施。随着虚拟机概念的出现，这种做法已经过时。如今，一个虚拟机应该只服务于一个业务使用案例。

# 从配置文件构建角色

让我们继续使用`phpmyadmin`的例子，它是为销售部门构建的，目的是让他们管理他们的 CRM 数据库。

在这种情况下，我们根据实现配置文件为该系统构建一个角色。角色的名称反映了业务使用案例：

```
class role::crm_db_control_panel {
  contain profile::login::secure
  contain profile::database::mysql
  contain profile::scripting::php
  contain profile::apps::phpmyadmin::db
  contain profile::apps::phpmyadmin
}
```

在角色中，只需要声明配置文件。不需要编写代码逻辑、资源或数据查找。这使得角色的使用更加灵活。不要尝试构建几乎相同的角色，因为这样会导致重复代码。相反，更好的做法是创建带有数据查找的配置文件，以便反映个人使用情况。

前面提到的角色可以用于节点分类：

```
node 'dbcrmmgmt.domain.com' {
  contain role::crm_db_control_panel
}
```

如你所见，我们有一个单一实例和一个单一角色。这在从头开始构建系统时非常有用。在现有的基础设施中，人们可以使用这一概念来识别当一个系统不可用时，哪些业务单元会受到影响。除此之外，它还帮助我们了解在需要时应该在哪里分离服务。

有些环境中，角色和配置文件的概念并不适用。通常，这些是现有平台，在这些平台上，多个服务（角色）运行在同一系统上，并且同一配置文件有许多不同的实现。在这些情况下，应该验证仅依靠实现层（配置文件）是否足够。

# 将代码放在 Puppet 服务器上

从技术角度看，角色和配置文件是模块中的类。通常，模块会放入环境的`modules`目录中。但是角色和配置文件不同于模块，它们是模块的实现和实现的集合。

为了反映这种不同的行为，通常的做法是向环境中添加另一个`module`目录。这个配置可以在环境中的`environment.conf`文件中完成：

```
#/etc/puppetlabs/code/environments/production/environment.conf
modulepath = site:modules:$basemodulepath
```

在我们的例子中，我们已经向模块路径设置中添加了一个新路径：site。该目录位于我们的环境中（`/etc/puppetlabs/code/environments/production/site`）。该目录将包含我们所有的角色和配置文件：

```
/etc/puppetlabs/code/environment/production/site/
  |- profile/
  | |- manifests/
  | | |- apps/
  | | | |- phpmyadmin/
  | | | | \- db.pp
  | | | \- phpmyadmin.pp
  | | |- database/
  | | | \- mysql.pp
  | | |- login/
  | | | \- secure.pp
  | | |- login.pp
  | | \ - scripting/
  | | \- php.pp
  | \- templates/
  | \- login/
  | |- sshd_config.epp
  | \- ssh_known_hosts.epp
\- role/
       \- manifests/
            \- db_control_panel.pp
```

这样，我们就能将角色和配置文件保存在一个单独的目录结构中，并且模块可以单独存放。

# Puppet 控制仓库

通常，Puppet 模块与上游开发的库相同。我们希望确保在 Puppet 代码中使用的模块以一种可以升级的方式进行存储。因此，我们不能将模块直接放入我们的环境 Git 仓库中。除此之外，我们还希望在将 Puppet 代码更新投入生产之前进行测试。

最佳实践是拥有一个控制代码库，其中包含我们的角色和配置文件、清单的节点分类、环境配置文件和 Hiera v5 配置。现在我们再添加另一个文件：`Puppetfile`。

`Puppetfile` 引用模块，并可选地指定它们的源位置和版本：

```
# Third Party modules
mod "puppetlabs/concat", '3.0.0' # postgresql requires concat < 3.0.0
mod "puppetlabs/stdlib", :latest
mod "puppetlabs/aws", :latest
mod "jdowning/rbenv", :latest
mod "puppet/archive", :latest
mod "puppetlabs/inifile", :latest
# Used by profile::puppet::server
mod 'puppetlabs/postgresql', :latest
mod 'puppetlabs/puppetdb', :latest
mod 'puppet/puppetserver',
  :git => "https://github.com/voxpupuli/puppet-puppetserver.git",
  :tag => '2.1.0'
mod 'puppetlabs/puppetserver_gem', :latest
mod 'puppet/r10k', :latest
# mod 'puppet/puppetboard', :latest
```

当没有给定源时，模块将从 Puppet Forge 安装（[`forge.puppet.com`](https://forge.puppet.com)）。由于大多数生产系统无法连接到互联网，因此在私有 Git 服务器上拥有上游模块开发仓库的克隆是很有用的。

Puppet 控制代码库可以具有以下文件和目录结构：

```
 control-repo/
  |- environment.conf
  |- hieradata/
  |- hiera.yaml
  |- manifests/
  | |- dmz.pp
  | |- internal.pp
  | \- site.pp
  |- Puppetfile
  |- README.md
  \- site/
       |- profile/
       | |- manifests/
       | | |- apps/
       | | | |- phpmyadmin/
       | | | | \- db.pp
       | | | \- phpmyadmin.pp
       | | |- database/
       | | | \- mysql.pp
       | | |- login/
       | | | \- secure.pp
       | | |- login.pp
       | | \- scripting/
       | | \- php.pp
       | \- templates/
       | \- login/
       | |- sshd_config.epp
       | \- ssh_known_hosts.epp
       \- role/
            \- manifests/
                 \- db_control_panel.pp  
```

# 同步上游模块

通常，可以使用工作站进行同步。首先，在本地 Git 服务器上创建一个空的代码库并克隆到工作站。在这个本地代码库中，添加一个新的远程位置：

```
git remote add github https://github.com/puppetlabs/puppetlabs-concat.git
```

现在获取上游代码：

```
$ git pull github master
remote: Counting objects: 2871, done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 2871 (delta 1), reused 7 (delta 1), pack-reused 2859
Receiving objects: 100% (2871/2871), 634.98 KiB | 648.00 KiB/s, done.
Resolving deltas: 100% (1394/1394), done.
From https://github.com/puppetlabs/puppetlabs-concat
* branch master -> FETCH_HEAD
```

Git 将代码与对象分离。通常，上游使用标签来标识其模块的版本发布。标签是非代码对象的一部分。接下来，获取非代码对象：

```
git fetch --all
Fetching origin
Fetching github
remote: Counting objects: 33, done.
remote: Compressing objects: 100% (29/29), done.
remote: Total 33 (delta 8), reused 28 (delta 3), pack-reused 0
Unpacking objects: 100% (33/33), done.
From https://github.com/puppetlabs/puppetlabs-concat
* [new branch] 1.0.x -> github/1.0.x
[...]
   * [new tag] 3.0.0 -> 3.0.0
   * [new tag] 4.0.0 -> 4.0.0
   * [new tag] 4.0.1 -> 4.0.1 
```

现在，代码被推送到本地代码库服务器：

```
$ git push origin master
   Counting objects: 2871, done.
Compressing objects: 100% (1368/1368), done.
    Writing objects: 100% (2871/2871), 634.76 KiB | 0 bytes/s, done.
Total 2871 (delta 1394), reused 2871 (delta 1394)
To /var/repositories/puppetlabs-concat.git
* [new branch] master -> master 
```

别忘了也要 `push` 标签：

```
$ git push origin master --tags
Counting objects: 19, done.
Compressing objects: 100% (19/19), done.
Writing objects: 100% (19/19), 11.01 KiB | 0 bytes/s, done.
Total 19 (delta 0), reused 0 (delta 0)
To /var/repositories/puppetlabs-concat.git
 * [new tag] 0.1.0 -> 0.1.0
[...]
 * [new tag] 3.0.0 -> 3.0.0
 * [new tag] 4.0.0 -> 4.0.0
 * [new tag] 4.0.1 -> 4.0.1
```

# R10K 代码部署

通常，公司会有一个暂存系统，其中新的开发会在投入生产前进行测试。开发团队有一个开发阶段。从这个阶段，变更将被部署到质量门阶段，并在成功测试后放入生产阶段。

许多人使用这些名称来命名 Puppet 代码环境。但是，如果你的 Puppet 更改会破坏整个开发阶段会发生什么呢？在这种情况下，开发团队将无法继续进行紧急的改进或修复。这可能导致耗时且成本高昂的工作，需要从头开始重新构建开发阶段。

但我们如何开发和测试 Puppet 代码更改呢？通常，这需要为基础设施开发人员设置另一个阶段。所有其他基础设施阶段（开发、QA、生产）随后会使用稳定的生产 Puppet 环境代码进行部署。

这就导致了两个 Puppet 环境：**开发**和**生产**。

但我们不希望这些是两个独立的 Git 仓库，因为这会使更改的分阶段变得非常困难。这就是 R10K 的作用所在。R10K 使用 Puppet 控制代码库中的分支，并将这些分支部署到 Puppet Master 上作为环境。现在，代码更改可以通过从一个分支合并到另一个分支来完成。

分支名称可以自由选择，但有一些特殊名称不应使用：master、agent、main。特别是对于 Git 仓库，其中默认分支为 master，需要重新配置默认分支名称。

必须在 Puppet Master 上安装并配置 R10K。我们可以使用 Puppet 来进行安装：

```
$ puppet resource package r10k ensure=present provider=puppet_gem
package { 'r10k':
  ensure => ['2.5.5'],
}  
```

`r10k` 配置文件必须放置在 `/etc/puppetlabs/r10k/r10k.yaml` 目录下。在该文件中，我们激活所需的 `cache` 目录，`r10k` 会在其中存储所有仓库的本地副本。记住保持这些缓存的干净状态，并在与本地 Git 仓库缓存相关的行为异常时，删除整个缓存。

R10K 允许使用多个控制仓库。这些仓库可以轻松地在一个环境中共存，因为它们会被加上提供的源名称作为前缀。控制仓库被放置在 `sources` 部分，并获得一个唯一的名称。然后，我们指定 R10K 获取代码的远程 URL。代码被部署到 `basedir` 设置中提到的路径。

最后的两个设置与获取代码相关。第一个设置涉及 Git 访问。在 Git 设置中，可以设置提供程序。提供程序有两个选项：`shellgit` 和 rugged。`shellgit` 提供程序使用 `git` 二进制文件，必须在 Puppet 主机上可用。运行 `R10K` 命令的用户必须配置好 `git` Shell 环境，并指定用于授权的 `ssh` 密钥。Rugged 是一个 Ruby 实现，可以直接在 `r10k.yaml` 文件中指定 Git `ssh` 设置。通常，使用 `shellgit` 就足够了。最后一个设置指定了 R10K 如何从 Puppet Forge 获取模块。在这里，我们只能指定一个代理服务器：

```
# /etc/puppetlabs/r10k/r10k.yaml
---
 cachedir: '/var/cache/r10k'
  sources:
  infrastructure:
   remote: 'https://gitserver/infra/r10k-control-repo.git'
  basedir: '/etc/puppetlabs/code/environments'
  qa:
   remote: 'https://gitserver/security/r10k-control-repo.git'
  basedir: '/etc/puppetlabs/code/environments'
   prefix: true
git:
 provider: shellgit
    forge:
  # proxy: 'http://proxyserver:port'
```

从控制仓库部署分支和安装模块通过运行 `r10k` 命令来完成。请确保仅以需要获取代码的用户身份运行此命令。以 `root` 身份运行此命令时，可能是 ssh 凭据错误，或者环境之后归属于 `root` 用户。参数 -v 启用详细模式：

```
# r10k deploy environment -v# r10k deploy environment -vINFO -> Deploying environment /etc/puppetlabs/code/r10k/developmentINFO -> Environment development is now at db43d907d5b39d6197e42fc5c8edb4c0a4db27d6[...]INFO -> Deploying environment /etc/puppetlabs/code/r10k/productionINFO -> Environment production is now at 149a903d027d69d88a50ef3a563cb432c1e087c4[…]INFO -> Deploying environment /etc/puppetlabs/code/r10k/qa_developmentINFO -> Environment qa_development is now at db43d907d5b39d6197e42fc5c8edb4c0a4db27d6[…]INFO -> Deploying environment /etc/puppetlabs/code/r10k/qa_productionINFO -> Environment production is now at 149a903d027d69d88a50ef3a563cb432c1e087c4
```

可以通过安装来自 `voxpupuli` 的 `puppet-r10k` 模块来替代手动运行 `r10k` 命令行工具 ([`github.com/voxpupili/puppet-r10k`](https://github.com/voxpupuli/puppet-r10k)):

```
class profile::puppet::master::r10k_webhook {
  class {'r10k::webhook::config':
    enable_ssl      => false,
    use_mcollective => false,
  }
  class {'r10k::webhook':
    use_mcollective => false,
    user            => 'root',
    group           => '0',
  }
}
```

此 webhook 可以由 Git 服务器或 CI/CD 测试/部署工具链触发，例如 Jenkins 或 GoCD。通常，两者的组合是一种有效的设置，您可以尽快部署功能分支，并且仅在测试成功后才在开发或生产环境中运行更新。

# 总结

在构建和维护 Puppet 代码库时，最好实施角色和配置文件模式。它要求您定义角色以覆盖所有机器使用场景。角色通过混合和匹配配置文件类，这些类基本上是从自定义和开源模块中收集的类，用来管理实际资源。

在设置您的第一个大型 Puppet 安装时，最好从第一天起就遵循这种模式，因为这样可以让您在不陷入复杂结构的情况下扩展清单。

Puppet 代码的部署通过`r10k`管理，其中 Git 分支名称反映了您的 Puppet 代码质量。使用`Puppetfile`可以将您自己的 Puppet 代码开发与上游模块开发分离开来。

这就是我们对*Puppet Essentials*的介绍。我们已经覆盖了相当多的内容，但正如您可以想象的那样，我们仅仅触及了一些话题的皮毛，比如 Puppet 代码测试、提供者开发，或是利用 PuppetDB。您所学的内容最有可能满足您的即时需求。如需了解更多内容，请随时查阅[`docs.puppet.com/`](https://docs.puppet.com/)上的精彩在线文档，或者加入社区，在聊天、Slack 或邮件列表中提出您的问题。

感谢阅读，希望您在使用 Puppet 及其管理工具家族时能充满乐趣。

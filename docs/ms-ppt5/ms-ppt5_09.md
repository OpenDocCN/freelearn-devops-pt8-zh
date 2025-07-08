# 第九章：导出资源

导出资源提供了一种让系统声明资源的方式，但不一定要实现它。它们的设计目的是让节点将有关自己信息发布到一个中央数据库（PuppetDB），这样另一个节点就可以收集 Puppet 资源并在系统上实现它。导出资源主要为创建一个能够感知环境中其他基础架构的基础架构提供了方法。它们对于必须最终与由自动化过程动态创建的基础架构中的信息进行合并的基础架构提供了最大的价值。

本章将涵盖以下主题：

+   虚拟资源和导出资源

+   一些用例

# 虚拟资源和导出资源

要理解导出资源，首先需要理解虚拟资源。虚拟资源声明了一种状态，该状态可能会被提供，但只有在使用`realize`函数声明时才会强制执行。这些资源的设计目的是让你提前发布资源，但只有在满足其他条件时才会强制执行。虚拟资源有助于克服 Puppet 代码中可能出现的单一声明挑战，特别是当你有多个清单需要生成相同资源时——你可能需要在单一节点上包括多个这样的清单。如果多个模块需要管理同一文件，可以考虑将资源虚拟化，并使该资源在多个模块中可用。

# 虚拟资源

虚拟资源的一个常见使用示例是使用特定访问权限的管理员用户。通过健全的安全策略，你可能不希望任何单一用户拥有基础架构中所有系统的管理员访问权限。此时，你可能希望将管理员用户声明为虚拟资源，并允许在适当的情况下由配置文件实现这些用户。在以下示例中，我将把自己添加为 Linux 系统的管理员用户，然后在多个清单中实现该资源，既不会导致资源冲突，又能让我将自己精确地放置在适当的系统上。

首先，我需要将自己以及可能的其他用户声明为虚拟资源：

```
# modules/admins/manifests/infrastructure.pp
# This manifest declares the virtual resource for my administrative user
class admins::infrastructure {
  @user {'rrussellyates':
    ensure => present,
    comment => 'Ryan Russell-Yates',
    groups => ['wheel']
  }
}
```

我希望在多个配置文件中使用`realize`函数从目录调用用户对象，并确保它已经存在于系统上。注意引用中的大写字母：`User['rrussellyates']`。这个对象已经在目录中存在，因此我调用的是一个已经存在的对象。我需要确保包含声明该虚拟用户的清单，以便该虚拟用户已经在目录中并由配置文件实现：

```
# modules/profile/manifests/monitoring_support.pp
# Assume I'm a member of a monitoring team, that monitors critical applications
class profile::monitoring_support {
  include admins::infrastructure
  include profile::nagios
  include profile::monitoring_baseline

  realize User['rrussellyates']
}

# modules/profile/manifests/team/baseline.pp
# This profile combines our multiple required classes for the application
class profile::my_app
 {

  include admins::infrastructure
  include security
  include ntp
  include dns

  realize User['rrussellyates']
}
```

现在，我的生产级应用程序需要两个清单来调用我作为管理员用户。由于这是一个只声明过一次的虚拟资源，因此这两个清单可以独立或一起调用该用户而不会发生冲突：

```
# The role for our production application with special SLA monitoring
# Notice that both my_app and monitoring support require me as an administrative
# user. A development version of the application needs my support, as well as
# anything with a production-level SLA for monitoring. If I attempted to declare
# myself as a resource in both of these profiles, we'd have a duplicate resource
# declaration.
class role::production_app {
  include profile::my_app
  include profile::monitoring_support
}
```

然后，我们将把这个角色应用到我们的节点，使用我们的`site.pp`：

```
# site.pp

node 'appserver' {
  include role::production_app
}
```

当我们在系统上运行这个时，管理员用户会被实现而不会出现重复的资源声明错误，即使该用户在两个个人资料中都被实现。我们可以在多个地方成功调用此用户，而不会发生资源冲突：

```
[root@wordpress vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Loading facts
Info: Caching catalog for wordpress
Info: Applying configuration version '1529120853'
Notice: /Stage[main]/Admins::Infrastructure/User[rrussellyates]/ensure: created
Notice: Applied catalog in 0.10 seconds
```

# 标签

不是所有虚拟资源都是相互独立的。有时，我们希望生成一组可以一起实现的资源。Puppet 提供了一个叫做标签（tag）的元参数，使我们能够将资源分类在一起。标签允许我们使用 `puppet agent -t --tags <tag>` 运行一部分 Puppet 代码。它们为资源提供了用户特定的标记，以便构建类似对象的集合。标签是一个数组，因此你可以为一个资源应用多个标签，但仍然分别调用它们。带有标签的虚拟资源可以通过资源收集器调用，这个收集器有时被称为 *飞船操作符*。通过标签调用资源的简单格式是 `Resource <| |> `。在两个竖线之间，你可以在目录中搜索任何参数或元参数。

通过使用标签，我们可以用 `User <| tag == 'monitoring_admin' |>` 调用这个管理员用户。这使我们能够将资源捆绑在一起并作为一个组调用，而不是作为一个独立的部分。让我们以之前的示例为基础，扩展成一个基于标签的系统：

```
class admins::infrastructure {
  @user {'rrussellyates':
    ensure  => present,
    comment => 'Ryan Russell-Yates',
    groups  => ['wheel'],
    tag     => ['infrastructure_admin','monitoring_support'],
  }
  @user {'jsouthgate':
    ensure  => present,
    comment => 'Jason Southgate',
    groups  => ['wheel'],
    tag     => ['infrastructure_admin'],
  }
  @user {'chuck':
    ensure  => present,
    comment => 'Our Intern',
    groups  => ['wheel'],
    tag     => ['monitoring_support'],
  }
}
```

现在，我们已经将 `chuck` 标记为监控支持的成员，将 Jason 标记为基础设施管理员的成员，而我自己则同时是这两个团队的成员。我的清单将调用一个组的用户，而不是单独调用每个用户：

```
class profile::my_app {
  include admins::infrastructure
  include security
  include ntp
  include dns

  # This line calls in all Monitoring Support and Infrastructure Admin users.
  User <| tag == 'monitoring_support' or tag == 'infrastructure_admin' |>
}
```

当我们更改个人资料以使用这两个标签时，将会添加两个额外的用户：`jsouthgate` 和 `chuck`。管理员用户 `russellyates` 已经在系统中，因此没有重新创建他：

```
[root@wordpress vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Loading facts
Info: Caching catalog for wordpress
Info: Applying configuration version '1529120940'
Notice: /Stage[main]/Admins::Infrastructure/User[jsouthgate]/ensure: created
Notice: /Stage[main]/Admins::Infrastructure/User[chuck]/ensure: created
Notice: Applied catalog in 0.11 seconds
```

# 导出的资源

虚拟资源允许我们将资源分配到它们所创建的节点上。进一步来说，导出的资源允许一个节点创建虚拟资源并与基础设施中的其他节点共享。导出的资源是设计自动化系统的一种有用方式，尤其是当这些系统需要来自其他自动化系统的信息时。在实现中，你可以将导出的资源视为已虚拟化并宣布的。该资源并未在系统上实现（尽管它可能会），而是与基础设施的其他部分共享。这使我们能够构建基于了解其他节点状态的系统来管理事物。

声明一个导出的资源与声明虚拟资源的方式相同，不同的是我们使用两个 `@` 符号而不是一个：

```
class profile::fillsuptmp 
{
  # Exported Resource. Virtual and Shared
  # Notice that only the @@ is different!
  @@file {
"/tmp/${::fqdn}":
    ensure  => present,
    content => $::ipaddress,
    tag     => ['Fillsuptmp']
  }
}
```

虽然我们可能不希望使用这个特定的示例，但我们可以使用文件`<| |>`在本地系统上实现这个资源，并仅接收本地资源。通过添加一组额外的括号，`<<| |>>`，我们的基础设施会将此文件作为每个节点所描述的内容来接收。以下示例展示了如何从导出的目录中检索我们的资源：

```
class profile::filltmp {
# This simple declaration will call our file from above and place one from
# each machine on the system. Notice the title containing $::fqdn, so a new
# file in /tmp will be created with the FQDN of each machine known to PuppetDB.

  include profile::fillsuptmp

  File <<| tag == 'Fillsuptmp' |>>
}
```

我们将在 `site.pp` 中添加这个配置文件，而不放在节点定义内部，这样所有节点都能使用它：

```
# site.pp

include profile::filltmp
```

当我们运行代理时，我们会看到每个节点在基础设施中的 `/tmp` 目录下生成一个文件。对于每个检查到 master 的节点，其他所有节点也将获得一个新的文件：

```
[root@wordpress vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Loading facts
Info: Caching catalog for wordpress
Info: Applying configuration version '1529121275'
Notice: /Stage[main]/Profile::Fillsuptmp/File[/tmp/wordpress]/ensure: defined content as '{md5}c679836c51e9e0e92191c7d2d38f5fe5'
Notice: /Stage[main]/Profile::Filltmp/File[/tmp/pe-puppet-master]/ensure: defined content as '{md5}c679836c51e9e0e92191c7d2d38f5fe5'
Notice: Applied catalog in 0.10 seconds
```

导出的资源是从最后一次报告到 PuppetDB 的目录存储中收集的。如果一个节点在目录中发生了变化，而导出的资源在上次运行中不再可用，则该资源将无法用于资源收集。

需要注意的是，关于导出的资源：实现这些资源的节点将*最终*趋向于基础设施的预期状态。直到节点将该资源报告给 Puppet Master，它才会意识到该节点的存在。假设你有一个新节点，并且已将其与外部资源分类到目录中，直到该节点第一次检查到 master，这个资源才会出现在 PuppetDB 中。即使它检查到 PuppetDB，执行该资源的节点也必须再次运行 `puppet agent run`。这意味着，对于刚刚运行了默认 30 分钟计时器的节点，它可能需要 30 分钟才能将其导出资源报告给 Puppet Master。然后，你的节点收集这些资源可能需要再等待 30 分钟才能与 master 检查并接收这些更改。导出的资源不是即时的，但你的基础设施最终会根据提供给 PuppetDB 的新信息进行收敛。

Puppet 代理的内置默认设置是每 30 分钟与 Puppet Master 进行一次检查。如果你有一台简单配置的机器，像负载均衡器，应该更快地找到此信息，可以考虑让它更频繁地检查与 master 的连接。每 5 分钟检查一次的负载均衡器，在通过 Puppet 完成初始配置后，应该很快就能将节点报告为在线。

# 使用案例

导出的资源最好在基础设施中谨慎使用。我们将讨论一些使用案例，并在过程中介绍可能使用这些信息的类似应用程序。我们将使用 Forge 模块，在合理的地方使用，但也会构建一些自定义导出的资源，以便提供一个功能性示例。在这一部分，我们将讨论几个导出资源的示例：

+   动态的 `/etc/hosts` 文件

+   将一个节点添加到 `haproxy` 负载均衡器

+   在应用服务器上为数据库服务器构建外部数据库

+   使用 `concat` 和 `File_line` Puppet 资源的自定义配置文件

# 主机文件

第一个示例易于理解和解释，但绝对不应替代真正的**域名服务器**(**DNS**)。几年前，我有一个客户正在使用公共云，但该云被一家非常大的公司收购，该公司有一个专门的团队来管理公司 DNS。DNS 记录的处理时间通常为 4 天，而他们启动的许多应用程序的生命周期只有几天，然后就会被新节点替换。如果节点的网络信息在一段时间内发生变化，这种解决方案会遇到一些问题，但它在 DNS 政策放宽之前为客户提供了有效的短期解决方案。

在下面的示例中，我们将使用一个单独的配置文件，导出每个由`profile::etchosts`分类的系统的 FQDN 和 IP 地址，这些信息将被环境中的每个节点（包括源节点）消费和处理：

```
class profile::etchosts {
# A host record is made containing the FQDN and IP Address of the classified node
  @@host {$::fqdn:
    ensure => present,
    ip     => $::ipaddress,
    tag    => ['shoddy_dns'], }
# The classified node collects every shoddy_dns host entry, including its own,
# and adds it to the nodes host file. This even works across environments, as
# we haven't isolated it to a single environment.
  Host <<| tag == 'shoddy_dns' |>>
}
```

如果我们希望确保仅收集处于相同 Puppet 环境中的主机条目，我们可以简单地将清单更改为读取`Host <<| tag == 'shoddy_dns'`和`environment == $environment |>>`。

这个清单包含了导出的资源和资源收集调用。我们在其上包含此清单的任何节点将报告其主机记录并检索所有主机记录（包括它自身）。由于我们希望将此代码应用于基础设施中的所有节点，因此我们会将其放在`site.pp`中的任何节点定义之外，使其适用于基础设施中的所有节点：

```
# site.pp
include profile::etchosts
# Provided so nodes don't fail to classify anything
node default { }
```

当我们运行代理时，我们会单独检索每个主机条目并将其放入`/etc/hosts`。在下面的示例中，我将这个目录应用于我的 Puppet Master。Puppet Master 从 PuppetDB 中检索每个主机条目并将其放入`/etc/hosts`。报告的节点是`haproxy`、`appserver`和`mysql`。这些节点将在本章其余的示例中使用：

```
[root@pe-puppet-master vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Loading facts
Info: Caching catalog for pe-puppet-master
Info: Applying configuration version '1529033713'
Notice: /Stage[main]/Profile::Etchosts/Host[mysql]/ensure: created
Info: Computing checksum on file /etc/hosts
Notice: /Stage[main]/Profile::Etchosts/Host[appserver]/ensure: created
Notice: /Stage[main]/Profile::Etchosts/Host[haproxy]/ensure: created
Notice: Applied catalog in 14.07 seconds

[root@pe-puppet-master vagrant]# cat /etc/hosts
# HEADER: This file was autogenerated at 2018-06-15 03:36:02 +0000
# HEADER: by puppet. While it can still be managed manually, it
# HEADER: is definitely not recommended.
127.0.0.1 localhost
10.20.1.3 pe-puppet-master
10.20.1.6 mysql
10.20.1.5 appserver
10.20.1.4 haproxy
```

如前所述，这不应被视为真正的 DNS 的替代品。它是一个简单且实用的示例，展示了如何构建和使用 Puppet 导出的资源。

# 负载均衡

负载均衡器是一个常见的系统，它在 Puppet 中使用导出的资源模式。负载均衡器用于将流量转发到多个节点，提供通过冗余实现的高可用性，以及通过水平扩展提供的性能。像 HAProxy 这样的负载均衡器还允许设计将用户引导到距离他们更近的数据中心的应用程序，以提高性能。

负载均衡器本身将接收传统配置，而负载均衡器的每个成员将导出一个资源供负载均衡器消费。负载均衡器随后使用每个导出资源中的每个条目来构建一个合并的配置文件。

以下示例使用了 Puppet Forge 中的`puppetlabs-haproxy`模块（更多信息请访问[`forge.puppet.com/puppetlabs/haproxy`](https://forge.puppet.com/puppetlabs/haproxy)）。HAProxy 是一个免费的开源负载均衡器，可以在家中无需许可证使用。Forge 上还有其他负载均衡器的模块可用，用户可以根据企业需求创建自定义模块。

我们需要创建两个配置文件来支持这个用例：一个用于负载均衡器，一个用于负载均衡成员。负载均衡成员配置文件是一个简单的导出资源，声明它将使用的监听服务，并将其主机名、IP 地址和可用端口报告给 HAProxy。`loadbalancer`配置文件将配置一个非常简单的默认`loadbalancer`，提供一个监听服务用于转发流量，最重要的是收集所有从导出资源来的配置：

```
class profile::balancermember {
  @@haproxy::balancermember { 'haproxy':
    listening_service => 'myapp',
    ports             => ['80','443'],
    server_names      => $::hostname,
    ipaddresses       => $::ipaddress,
    options           => 'check',
  }
}
class profile::loadbalancer {
  include haproxy

  haproxy::listen {'myapp':
    ipaddress => $::ipaddress,
    ports => ['80','443']
  }

  Haproxy::Balancermember <<| listening_service == 'myapp' |>>
}
```

然后，我们将这些配置文件放置在两个不同的主机上。在以下示例中，我将`balancermember`配置文件放置在`appserver`上，而`loadbalancer`配置文件放置在`haproxy`上。我们将继续扩展之前的`site.pp`，并逐步添加代码：

```
#site.pp
include profile::etchosts

node 'haproxy' {
  include profile::loadbalancer
}

node 'appserver' {
  include profile::balancermember
}

# Provided so nodes don't fail to classify anything
node default { }
```

在以下示例中，负载均衡器已经配置为负载均衡器，但没有转发的负载均衡成员。`appserver`也已经完成了一次运行，并将其导出的`haproxy`配置报告到 PuppetDB。最后，HAProxy 服务器收集并实现这个资源，并将其作为一行放入配置文件中，使得`haproxy`能够将流量转发到`appserver`：

```
root@haproxy vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Loading facts
Info: Caching catalog for haproxy
Info: Applying configuration version '1529036882'
Notice: /Stage[main]/Haproxy/Haproxy::Instance[haproxy]/Haproxy::Config[haproxy]/Concat[/etc/haproxy/haproxy.cfg]/File[/etc/haproxy/haproxy.cfg]/content:
--- /etc/haproxy/haproxy.cfg 2018-06-15 04:27:25.398339144 +0000
+++ /tmp/puppet-file20180615-17937-6bt84x 2018-06-15 04:28:05.100339144 +0000
@@ -27,3 +27,5 @@
 bind 10.0.2.15:443
 balance roundrobin
 option tcplog
+ server appserver 10.0.2.15:80 check
+ server appserver 10.0.2.15:443 check

Info: Computing checksum on file /etc/haproxy/haproxy.cfg
Info: /Stage[main]/Haproxy/Haproxy::Instance[haproxy]/Haproxy::Config[haproxy]/Concat[/etc/haproxy/haproxy.cfg]/File[/etc/haproxy/haproxy.cfg]: Filebucketed /etc/haproxy/haproxy.cfg to puppet with sum dd6721741c30fbed64eccf693e92fdf4
Notice: /Stage[main]/Haproxy/Haproxy::Instance[haproxy]/Haproxy::Config[haproxy]/Concat[/etc/haproxy/haproxy.cfg]/File[/etc/haproxy/haproxy.cfg]/content: content changed '{md5}dd6721741c30fbed64eccf693e92fdf4' to '{md5}b819a3af31da2d0e2310fd7d521cbc76'
Info: Haproxy::Config[haproxy]: Scheduling refresh of Haproxy::Service[haproxy]
Info: Haproxy::Service[haproxy]: Scheduling refresh of Service[haproxy]
Notice: /Stage[main]/Haproxy/Haproxy::Instance[haproxy]/Haproxy::Service[haproxy]/Service[haproxy]: Triggered 'refresh' from 1 event
Notice: Applied catalog in 0.20 seconds
```

这个模块使用`Concat`片段来构建整个文件。如果一个节点在某次运行中没有报告其`haproxy::balancermember`导出资源，它将在下一次运行时从`loadbalancer`中移除，后者会实现这些资源。

当用户请求 HAProxy 服务器的端口`80`（`http`）或端口`443`（`https`）时，它将自动从我们的`appserver`获取并转发流量。如果我们有多个应用服务器，它甚至会在两个服务器之间分配负载，实现横向扩展。

# 数据库连接

许多应用程序需要数据库来存储会话间的信息。在许多大型组织中，通常将这些数据库集中化，并将应用程序指向它们。以下示例将包含两个配置文件：一个用于数据库，一个用于需要使用该数据库的应用程序。

以下示例使用了 Puppet Forge 中的`puppetlabs-mysql`模块。MySQL 是一个免费的开源 SQL 数据库，可以在家中无需许可证使用。还有其他数据库的模块可用，如 SQL Server、OracleDB、Kafka 和 MongoDB。这些模块可以以类似的方式使用，为外部数据库提供导出资源。

在下面的示例中，`appserver::database`配置文件提供了一个非常简单的 MySQL 安装。它还获取所有标记为`ourapp`的`mysql::db`资源，并在中央服务器上实现它们。`appserver`配置文件接受一个密码参数，该密码可以通过`hiera`提供，或使用`eyaml`加密。使用此密码，它将导出一个数据库资源，并将在数据库服务器上进行收集和实现。还可以对该服务器上的应用程序进行其他配置，以确保它使用这个导出的资源提供的数据库：

```
class profile::appserver (
  $db_pass = lookup('dbpass')
) {
  @@mysql::db { "appdb_${fqdn}":
    user     => 'appuser',
    password => $db_pass,
    host     => $::fqdn,
    grant    => ['SELECT', 'UPDATE', 'CREATE'],
    tag      => ourapp,
  }
}

class profile::appserver::database {

  class {'::mysql::server':
    root_password => 'suP3rP@ssw0rd!',
  }

  Mysql::Db <<| tag == 'ourapp' |>>
}
```

我们将把这些配置文件插入到`appserver`节点的定义中，并为`mysql`创建一个新的节点定义。此配置将确保我们的`appserver`在`mysql`服务器上有一个相关的数据库，并且它的应用程序能够通过`haproxy`正确地转发。注意在`appserver`节点上传递密码：

```
#site.pp

include profile::etchosts

node 'haproxy' {
  include profile::loadbalancer
}
node 'appserver' {
  include profile::balancermember
  class {'profile::appserver': db_pass => 'suP3rP@ssw0rd!' }
}
node 'mysql' {
  include profile::appserver::database
}
# Provided so nodes don't fail to classify anything
node default { }
```

当应用于一个已配置的数据库服务器时，使用从`appserver`新导出的资源，数据库服务器上会创建一个新的数据库、数据库用户和一组权限：

```
[root@mysql vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Loading facts
Info: Caching catalog for mysql
Info: Applying configuration version '1529037526'
Notice: /Stage[main]/Profile::Appserver::Database/Mysql::Db[appdb_appserver]/Mysql_database[appdb_appserver]/ensure: created
Notice: /Stage[main]/Profile::Appserver::Database/Mysql::Db[appdb_appserver]/Mysql_user[appuser@appserver]/ensure: created
Notice: /Stage[main]/Profile::Appserver::Database/Mysql::Db[appdb_appserver]/Mysql_grant[appuser@appserver/appdb_appserver.*]/ensure: created
Notice: Applied catalog in 0.34 seconds
```

该系统的一个缺陷是从启动`appserver`到数据库在系统上实现之间有长达 30 分钟的间隙。在下一章中，我们还将讨论应用程序编排，它通过将节点链接在一起并协调代理运行来帮助解决这个问题。如果你有时间让基础设施收敛，这个导出的资源可以仅用于数据库和应用程序。

# Concat，文件行，以及你！

之前的示例依赖于现有资源，例如主机，或现有的 forge 模块，如`haproxy`和`mysql`。在某些情况下，我们需要使用导出的资源在系统上构建自定义配置文件。我们将展示使用`concat`和`file_line`的示例。`concat`用来声明文件的全部内容，使用一个有序字符串列表。文件行是`puppetlabs-stdlib`的一部分（更多信息请访问[`forge.puppet.com/puppetlabs/stdlib`](https://forge.puppet.com/puppetlabs/stdlib)），它将缺失的行插入到现有文件中，也可以使用`regex`匹配现有行。

# Concat – 锤子

我们将构建一个名为`/tmp/hammer.conf`的文件，该文件由一个头部部分和由导出的资源提供的可变数量的部分组成。这个类设计用来在基础设施中的所有机器上使用，但也可以通过将导出的资源与`concat`资源和头部`concat::fragment`分开，轻松转变为单个服务器配置文件。

以下示例使用了 Puppet Forge 中的`puppetlabs-concat`。（更多信息请访问[`forge.puppet.com/puppetlabs/concat`](https://forge.puppet.com/puppetlabs/concat)）。`concat`模块允许我们声明一个由多个片段组成的文件，并按顺序在系统上创建该文件。这使得我们能够定义一个或多个头部和尾部，同时为动态行的添加留出空间。

在以下示例中，我们正在构建一个清单来构建`hammer.conf`。`concat`资源为每个片段提供了构建的位置。`hammer time`的`concat::fragment`用于管理头部，如参数中的顺序`01`所示。每台机器将导出一个`concat`片段，详细描述 FQDN、IP 地址、操作系统和版本，并作为文件中的一行，以模拟一个全局配置文件或清单文件。最后，每台机器将使用`concat`片段的资源收集器实现所有这些导出的片段：

```
class files::hammer {

  $osname = fact('os.name')
  $osrelease = fact('os.release')

  concat {'/tmp/hammer.conf':
    ensure => present,
  }

  concat::fragment {'Hammer Time':
    target => '/tmp/hammer.conf',
    content => "This file is managed by Puppet.It will be overwritten",
    order => '01',
  }

  @@concat::fragment {"${::fqdn}-hammer":
    target => '/tmp/hammer.conf',
    content => "${::fqdn} - ${::ipaddress} - ${osname} ${osrelease}",
    order => '02',
    tag => 'hammer',
  }

  Concat::Fragment <<| tag == 'hammer' |>>

}
```

我们将在任何节点定义之外添加`files::hammer`，这样这个清单文件将在我们基础设施中的所有机器上创建：

```
include profile::etchosts
include files::hammer

node 'haproxy' {
  include profile::loadbalancer
}

node 'appserver' {
  include profile::balancermember
  class {'profile::appserver': db_pass => 'suP3rP@ssw0rd!' }
}

node 'mysql' {
  include profile::appserver::database
}

# Provided so nodes don't fail to classify anything
node default { }
```

当在我们基础设施中的`mysql`节点上运行时，作为基础设施中的最后一个节点，`/tmp/hammer.conf`文件将被创建，并包含每个节点提供的 Facter facts：

```
root@mysql vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Loading facts
Info: Caching catalog for mysql
Info: Applying configuration version '1529040374'
Notice: /Stage[main]/Files::Hammer/Concat[/tmp/hammer.conf]/File[/tmp/hammer.conf]/ensure: defined content as '{md5}f3f0d7ff5de10058846333e97950a7b9'
Notice: Applied catalog in 0.33 seconds

# /tmp/hammer.conf
This file is managed by Puppet. It will be overwritten
haproxy - 10.0.2.15 - CentOS 7
mysql - 10.0.2.15 - CentOS 7
pe-puppet-master - 10.0.2.15 - CentOS 7
appserver - 10.0.2.15 - CentOS 7
```

大多数配置文件应该以这种方式构建，并由 Puppet 整体管理。如果某个文件的全部内容未被管理，我们可以使用`stdlib`提供的`file_line`来代替。

# file_line – 手术刀

在本练习中，我们将使用 Puppet 文件资源来构建文件，但仅在系统中尚未存在该文件时才会构建。然后，我们将使用文件行将我们想要的值插入到文件中，这些值是从所有系统的导出资源中收集的。

以下示例使用了 Puppet Forge 中的`puppetlabs-stdlib`（更多信息请访问[`forge.puppet.com/puppetlabs/stdlib`](https://forge.puppet.com/puppetlabs/stdlib)）。`stdlib`包含了大量可以在清单中使用的函数，以及资源`file_line`。`file_line`允许单独管理文件中的一行，也可以作为查找和替换未管理文件时使用的正则表达式匹配。如果是针对 INI 文件，建议改用`puppetlabs-inifile`。（更多信息请访问[`forge.puppet.com/puppetlabs/inifile`](https://forge.puppet.com/puppetlabs/inifile)）。

在这个示例中，我们将首先创建一个名为`/tmp/scalpel.conf`的文件。我们会确保这个文件存在，并且其所有权属于 root。我们将设置替换标志，告知 Puppet 如果该文件已经存在于系统中则不要替换其内容，从而确保文件中已有的内容不会被覆盖。如果文件当前不在系统中，内容行将提供默认值。接下来，我们将创建一个导出的`file_line`来模拟一个配置行，并使用匹配语句确保我们替换的是配置错误的行，而不是创建新行。最后，我们将在所有分类了此文件的节点上实现这些资源：

```
class files::scalpel {

  $arch = fact('os.architecture')
  file {'/tmp/scalpel.conf':
    ensure => file,
    owner => 'root',
    group => 'root',
    content => 'This file is editable, with individually managed settings!',
    replace => false,
  }
  @@file_line {"$::fqdn - setting":
    path => '/tmp/scalpel.conf',
    line => "${::fqdn}: $arch - ${::kernel} - Virtual: ${::is_virtual}",
    match => "^${::fqdn}:",
    require => File['/tmp/scalpel.conf'],
    tag => 'scalpel',
  }
  File_line <<| tag == 'scalpel' |>>
}
```

scalpel 配置文件旨在在基础设施中的每台机器上使用，因此它也被放置在`site.pp`中的节点定义外：

```
include profile::etchosts

include files::scalpel

node 'haproxy' {
  include profile::loadbalancer
}

node 'appserver' {
  include profile::balancermember
  class {'profile::appserver': db_pass => 'suP3rP@ssw0rd!' }
}

node 'mysql' {
  include profile::appserver::database
}

# Provided so nodes don't fail to classify anything
node default { }
```

最后，我们的节点捕捉到配置更改，并创建了文件。请注意，文件已被创建，然后通过我们在导出`file_line`资源中使用的`require`参数插入了文件行：

```
[root@haproxy vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Loading facts
Info: Caching catalog for haproxy
Info: Applying configuration version '1529041736'
Notice: /Stage[main]/Files::Scalpel/File[/tmp/scalpel.conf]/ensure: defined content as '{md5}2d3ebc675ea9c8c43677c9513f820db0'
Notice: /Stage[main]/Files::Scalpel/File_line[haproxy - setting]/ensure: created
Notice: /Stage[main]/Files::Scalpel/File_line[mysql - setting]/ensure: created
Notice: /Stage[main]/Files::Scalpel/File_line[appserver - setting]/ensure: created
Notice: /Stage[main]/Files::Scalpel/File_line[pe-puppet-master - setting]/ensure: created
Notice: Applied catalog in 0.18 seconds

# /tmp/scalpel.conf
This file is editable, with individually managed settings!
haproxy: x86_64 - Linux - Virtual: true
mysql: x86_64 - Linux - Virtual: true
appserver: x86_64 - Linux - Virtual: true
pe-puppet-master: x86_64 - Linux - Virtual: true
```

与我们的`concat`示例不同，除由清单管理的单独行外，这个文件在 Puppet 之外仍然是可编辑的。在下面的示例中，我已将文件顶部添加了注释，并将`haproxy`的虚拟设置更改为 false：

```
# Our comments now stay in this file, because we're not managing
# The whole file, just individual lines. This methodology can
# come in useful once in a great while. This is still configuration
# drift, so make sure to use it sparingly!
This file is editable, with individually managed settings!
haproxy: x86_64 - Linux - Virtual: false
mysql: x86_64 - Linux - Virtual: true
appserver: x86_64 - Linux - Virtual: true
pe-puppet-master: x86_64 - Linux - Virtual: true
```

当代理再次运行时，`haproxy`行会被修正，但我们的注释仍然保持在文件顶部。用户甚至可以向该文件添加自己的配置行，只要这些配置没有被 Puppet 导出的资源报告，它们就会保留在配置文件中：

```
[root@haproxy vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Loading facts
Info: Caching catalog for haproxy
Info: Applying configuration version '1529042980'
Notice: /Stage[main]/Files::Scalpel/File_line[haproxy - setting]/ensure:
created
Notice: Applied catalog in 0.15 seconds

# Our comments now stay in this file, because we're not managing
# The whole file, just individual lines. This methodology can
# come in useful once in a great while. This is still configuration
# drift, so make sure to use it sparingly!

This file is editable, with individually managed settings!
haproxy: x86_64 - Linux - Virtual: true
mysql: x86_64 - Linux - Virtual: true
appserver: x86_64 - Linux - Virtual: true
pe-puppet-master: x86_64 - Linux - Virtual: true
```

这种方法确实允许基础设施中存在大量的配置漂移。如果你的团队致力于提供受控的自服务资源，这是一个有效的方式，允许你的客户修改配置文件，前提是这些设置没有被你的基础设施团队专门管理。

# 总结

在这一章中，我们讨论了虚拟资源和导出资源。虚拟资源允许我们描述一个资源应该是什么，并在其他条件下实现它。导出资源允许我们通过使用 PuppetDB 将我们的虚拟资源公布给基础设施中的其他节点。我们探讨了如何为管理用户编写虚拟资源，并为基础设施中的所有其他节点在`/tmp`目录中放置一个文件。接着我们使用导出资源创建了一个`/etc/hosts`文件，一个负载均衡器，一个数据库，以及使用`concat`和`file_line`构建自定义配置文件的示例。

当我们将这些导出资源应用到我们的系统时，我们注意到这些资源的主要限制是时间。我们的基础设施最终会收敛，但它并不会以协调和及时的方式发生。我们的下一章将讨论应用程序编排，它允许我们将多层应用程序连接起来，并编排 Puppet 在这些节点上的运行顺序。

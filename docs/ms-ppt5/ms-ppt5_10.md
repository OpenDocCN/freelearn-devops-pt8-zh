# 第十章：应用程序编排

应用程序编排为 Puppet 语言提供了几个关键特性。应用程序编排扩展了导出资源的概念，针对更具体的应用程序，允许节点之间共享配置项。此外，该特性还提供了一种方式，按顺序安排 Puppet 运行，确保依赖节点在需要它们的节点之前完成构建或汇聚。应用程序编排允许我们将多个节点按顺序组合在一起运行。最重要的是，配置更新不是在检查时随机应用的，而是按照特定的、顺序的模式应用的。

应用程序编排仅适用于 Puppet Enterprise。Puppet 开源用户可以使用语言构造，但有序运行是 Puppet Enterprise 提供的。

应用程序编排有三个新的语言构造，我们需要使用它们来创建有序运行，并自动共享彼此的信息：

+   **应用程序定义**：描述整个应用程序堆栈的组件集合的端到端描述

+   **应用程序组件**：整个应用程序堆栈的单个组件

+   **服务资源**：旨在跨应用程序组件共享信息的资源

# 应用程序定义

应用程序定义看起来很像定义的资源，但也类似于传统的 Puppet 配置文件。它们描述了一组组件，构成整个系统，但与配置文件不同，它们不局限于单一节点。这些应用程序定义描述了一个或多个节点的已配置状态，按应用程序组件拆分。

应用程序定义将类似于定义类型，但有几个关键区别：

+   它们被命名为`application`，而不是`define`。

+   每个资源必须在模块内进行命名空间划分：

```
# Application used instead of 'class' or 'define'
application 'example' (
  $var,
) {

# app1 exports its database configuration items
  example::app1 {
    config => $var,
    export => Database['app1'],
  }

# app2 both imports the previous database and exports its own type: Application
  example::app2 {
    config  => $var,
    consume => Database['app1'],
    export  => Application['app1'],
  }
```

需要注意的是，应用程序中的每个资源都可以与我们网站定义中的一个完全不同的节点相关联。我们还可以使用我们的网站定义来传递这些共享的配置项，这些项在前面的代码中由`$var`表示：

```
#/etc/puppetlabs/code/environments/production/manifests/site.pp
site {
  example {'app1':
    var => 'config',
    nodes => {
      Node['database'] => [ Example::App1['app']],
      Node['app']      => [ Example::App2['app']],
    }
  }
}
```

在节点的哈希值中，注意到`Node`对象和`Example::App<X>`对象是大写的。

# 应用程序组件

应用程序组件提供了多节点应用程序的单个部分。它们通常是定义类型（以便重用），但在非常简单的情况下，也可以是类或甚至本地资源，例如文件。应用程序组件由`export`、`consume`或`require`元参数在应用程序声明中创建。

应用程序组件写成通用类或定义类型。它们遵循与所有其他 Puppet 代码相同的自动加载格式。`example::app2`的清单仍然位于`manifests/example/app2.pp`。应用程序组件可以通过在清单底部添加一个额外的语句，显式列出它们在各自清单中导出和消费的值：

```
class example::app2 (
# $db_host is provided by the consume of the Database
  $db_host,
) {
# Any resources, defined types or class calls in a regular manifest would be placed here.
}
# Note that the consume is outside of the class declaration
Example::App2 consumes Database {
  db_host => $host,
}
# Note that the produces is outside of both the class declaration, and above consume
Example::App2 produces Http {
  host => $::fqdn,
  port => '80',
}
```

在上面的示例中，`$db_host`是一个可以传递给清单中任何资源的值。我们不是通过 Hiera 或 Puppet DSL 传递它，而是通过另一个应用程序提供的`host`参数来消费该值。我们还导出节点自身的 FQDN 和主机名，以便后续应用程序可以使用这些值指向由`example::app2`创建的 Web 服务。`Database`和`Http`都是服务资源，描述在应用程序之间共享的信息。

# 服务资源

服务资源是由应用组件填充和查看的环境范围信息池。服务资源类似于导出资源，提供关于 PuppetDB 中其他节点的信息。服务资源的独特之处在于它们建立节点之间的依赖关系。服务资源被声明为 Puppet 类型，用 Ruby 编写。提供程序是可选的，并允许进行导出资源的可用性测试。

服务资源类型提供了通过应用编排的`consume`和`export`元参数存储和传输信息的框架。服务资源需要一个类型，使用 Ruby 代码声明信息的结构。它们始终存储在模块中，路径为`lib/puppet/type/<resource>.rb`，并在部署时发送到环境中的所有节点，但未使用该资源的节点不会执行操作。以下示例类型可以包含由`app1`导出并由`app2`消费的数据库资源：

```
#lib/puppet/type/database.rb

# Notice the :is_capability => true. This property creates this type as an
# environment-wide service, to be produced and consumed.
Puppet::Type.newtype :database, :is_capability => true do
  newparam :host
end
```

这个简单的示例为我们提供了一种导出关于数据库信息的方式，特别是`host`参数。这可以通过`export`参数填充，并通过`consume`参数读取。

# 建模应用程序

在本章的其余部分，我们将专注于构建一个简单和一个更复杂的编排应用程序示例。我们的第一阶段将是创建一个单一的数据库和一个单一的 Web 服务器。

# 应用程序和数据库

在我们的第一个示例中，我们将从一个节点的数据库导出信息，并在 WordPress 实例上检索它。这个简单的示例将允许我们成对部署节点，并确保在依赖于它的 Web 应用程序之前构建数据库。

# 依赖关系

在开始编写代码之前，我们将查找 Forge 上的相关支持或开源模块。WordPress 需要一个 SQL 服务器和一个 Web 主机，我们将通过 Apache HTTPD 提供。在开始之前，我们需要从 Forge 安装以下模块：

+   `puppetlabs-mysql`

+   `puppetlabs-apache`

+   `hunner-wordpress`

```
[root@pe-puppet-master myapp]# puppet module install puppetlabs-mysql
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└─┬ puppetlabs-mysql (v5.4.0)
 ├── puppet-staging (v3.2.0)
 ├── puppetlabs-stdlib (v4.25.1)
 └── puppetlabs-translate (v1.1.0)
[root@pe-puppet-master myapp]# puppet module install puppetlabs-apache
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└─┬ puppetlabs-apache (v3.1.0)
 ├── puppetlabs-concat (v4.2.1)
 └── puppetlabs-stdlib (v4.25.1)
[root@pe-puppet-master myapp]# puppet module install hunner-wordpress
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└─┬ hunner-wordpress (v1.0.0)
 ├── puppetlabs-concat (v4.2.1)
 ├── puppetlabs-mysql (v5.4.0)
 └── puppetlabs-stdlib (v4.25.1)
```

# 构建

我们将从顶部开始编写代码。在学习代码的同时，想象代码的最终状态，并学习在此过程中启用它的各个部分是很有帮助的。

# 节点声明

我们的第一步将是节点声明。这将放在我们的`site.pp`中，每个应用程序将放在特定的站点调用下。在下面的示例中，请注意以下内容：

+   所有应用都在顶级 `site{}` 声明中声明。

+   `myapp {'myapp': }` 只是可以放入 `site.pp` 中的一种应用。我们还可以在它下面再添加一个，如 `myapp.{'myapp2': }`，它位于站点中，并且拥有此应用的第二个独立实例。

+   `Node['<nodename>']` 和 `Myapp::<app>` 是大写的。

+   我仍然可以使用 `site.pp` 来处理其他事项，正如 Puppet Master 的分类所示，具体如下：

```
site {
  myapp { 'myapp':
      nodes => {
        Node['mysql'] => [ Myapp::Db['myapp']],
        Node['appserver'] => [ Myapp::Web['myapp']],
        Node['haproxy'] => [ Myapp::Lb['myapp']],
    }
  }
}

node 'puppetmaster' {
  include role::puppetmaster
}

# To keep the sample simple, firewalls have been disabled on all machines.
service {'firewalld': ensure => stopped }
service {'iptables':  ensure => stopped }
```

这个特定配置将确保 `mysql` 节点获取数据库，`appserver` 获取 WordPress，而 HAProxy 获取负载均衡器配置。

# 应用声明

在前面的示例中，我们在 `site{}` 声明下方调用了一个名为 `myapp` 的资源。此清单位于 `myapp` 模块中的 `manifests/init.pp`，声明了该应用，描述了一些可覆盖的参数，并使用 `export` 和 `consume` 元参数来编排应用。请注意以下内容：

+   在第一行中，应用 `myapp` 被用作类或定义的替代。

+   `myapp::db` 导出 SQL 资源。

+   `myapp::web` 消耗 SQL 资源。

+   `myapp::db` 会在 `myapp::web` 之前运行，因为 `myapp::web` 通过 consume 依赖于它。

+   我们使用 `$name` 变量，以便每个组件接收 `myapp` 作为名称，该名称来自 `myapp {'myapp':}`。

```
application myapp (
  $dbuser = 'wordpress',
  $dbpass = 'w0rdpr3ss!',
  $webpath = '/var/www/wordpress',
  $vhost = 'appserver',
) {
  myapp::db { $name:
    dbuser => $dbuser,
    dbpass => $dbpass,
    export => Sql[$name],
  }
  myapp::web { $name:
    webpath => $webpath,
    consume => Sql[$name],
    vhost => $vhost,
  }
}
```

# DB 服务资源

我们将为这个简单的用例构建自己的自定义 DB 类型。它将允许我们从数据库向 WordPress 应用传递值。这个简单的示例确保类型命名为 `db`，并将其标记为服务资源，向数据库服务资源提供五个可用参数。此文件放置在 `lib/puppet/type/db.rb` 中：

```
# lib/puppet/type/db.rb
# Adding :is_capability to the custom type marks the resources as service resources
Puppet::Type.newtype :db, :is_capability => true do
  newparam :name, :is_namevar => true
  newparam :user
  newparam :password
  newparam :port
  newparam :host
end
```

# 应用组件

我们的 `myapp` 定义类型将利用我们在上一节创建的 `db` 资源，将四个值传递给 PuppetDB，直接从 `myapp::db` 中。我们将使用此清单来构建 MySQL 服务器，并向另一个节点上的 WordPress 实例提供信息。请注意以下示例中的内容：

+   一个常规的定义类型，使用标准的 Puppet DSL。我们构建了一个服务器和一个数据库来支持该应用。

+   `$host` 在清单中没有使用，但会传递给产生的 `Db` 资源。

+   `Myapp::Db 产生 Db` 直接放置在定义之后，在同一个清单中：

```
define myapp::db (
  $dbuser,
  $dbpass,
  $host = $::fqdn,
){

  class {'::mysql::server':
    root_password => 'Sup3rp@ssword!',
    override_options => {
      'mysqld' => {
        'bind-address' => '0.0.0.0'
      }
    }
  }

  mysql::db { $name:
    user => $dbuser,
    password => $dbpass,
    host => '%',
    grant => ['ALL PRIVILEGES'],
  }
}
Myapp::Db produces Db {
  dbuser => $dbuser,
  dbpass => $dbpass,
  dbhost => $host,
  dbname => $name,
}
```

`Myapp::Web` 是一个定义类型，用于消耗由 `Myapp::Db` 产生的 `Db`。它安装所需的包，安装 Apache，构建 vhost，并将 WordPress 部署到 vhost 的 `docroot`。请注意以下内容：

+   `$vhost` 和 `$webpath` 是由应用 `myapp` 提供的。

+   `$dbuser`、`$dbpass`、`$dbhost` 和 `$dbname` 由消耗 `Db {}` 提供。

+   因为我们的清单使用了 `dbpass`、`dbhost`、`dbuser` 和 `dbname` 的值，我们的映射不需要再声明。以下示例将直接声明变量：

```
define myapp::web (
  $webpath,
  $vhost,
  $dbuser,
  $dbpass,
  $dbhost,
  $dbname,
  ) {

    package {['php',
              'mysql',
              'php-mysql',
              'php-gd'
             ]:
      ensure => installed,
    }

    class {'apache':
      default_vhost => false
    }

    include ::apache::mod::php

    apache::vhost { $vhost:
      port => '80',
      docroot => $webpath,
      require => File[$webpath],
    }

    file { $webpath:
      ensure => directory,
      owner => 'apache',
      group => 'apache',
      require => Package['httpd'],
    }

    class { '::wordpress':
      db_user => $dbuser,
      db_password => $dbpass,
      db_host => $dbhost,
      db_name => $dbname,
      create_db => false,
      create_db_user => false,
      install_dir => $webpath,
      wp_owner => 'apache',
      wp_group => 'apache',
    }
  }
Myapp::Web consumes Db { }
```

我们可以使用前面的代码集合来排序和部署我们的多层应用程序。我们当前的模块应如下所示：

```
myapp
├── lib
│   └── puppet
│       └── type
│           ├── sql.rb
├── manifests
    ├── db.pp
    ├── init.pp
    └── web.pp
```

然后我们可以使用`puppet app`和`puppet job`命令来部署我们的应用程序。

# 部署

要查看列在`site.pp`中的应用程序，我们可以使用命令`puppet app show`。此命令读取我们的主清单，并列出所有应用程序及其组件。在以下示例中，从前面的代码来看，我们正在将`Myapp::Db`部署到`mysql`，并将`Myapp::Web`部署到`appserver`：

在运行此实验时，可能会收到一条消息：*应用程序管理已禁用。要启用它，请在 orchestrator 服务配置中设置`app-management: true`*。为了解决这个问题，你可以登录到 Puppet Enterprise 控制台，进入 Puppet Master 配置，并将`puppet_enterprise::profile::master::app-management`的值更改为`true`。

```
[root@pe-puppet-master manifests]# puppet app show
Myapp[myapp]
 Myapp::Db[myapp] => mysql
 + produces Sql[myapp]
 Myapp::Web[myapp] => appserver
 consumes Sql[myapp]
```

为了模拟部署，我们可以使用`puppet job plan`命令。我们为它提供`application`和`environment`标志，以便应用程序协调器知道使用哪个版本的`site.pp`。此命令主要显示排序，你可以从以下结果中看到，`mysql`将在`appserver`之前进行配置：

```
[root@pe-puppet-master manifests]# puppet job plan --application Myapp --environment production

+-------------------+------------+
| Environment | production |
| Target | Myapp |
| Concurrency Limit | None |
| Nodes | 2 |
+-------------------+------------+

Application instances: 1
 - Myapp[myapp]

Node run order (nodes in level 0 run before their dependent nodes in level 1, etc.):
0 -----------------------------------------------------------------------
mysql
 Myapp[myapp] - Myapp::Db[myapp]

1 -----------------------------------------------------------------------
appserver
 Myapp[myapp] - Myapp::Web[myapp]

Use `puppet job run --application 'Myapp' --environment production` to create and run a job like this.
Node catalogs may have changed since this plan was generated.
```

通过从`puppet job plan`切换到`puppet job show`，我们实际上是按顺序部署我们的代码。运行首先发生在`mysql`服务器上，产生的信息将被`appserver`节点使用。此运行确保在尝试部署依赖于它们的应用程序之前，必要的组件已完全部署：

```
Use `puppet job run --application 'Myapp' --environment production` to create and run a job like this.
Node catalogs may have changed since this plan was generated.
[root@pe-puppet-master manifests]# puppet job run --application 'Myapp' --environment production
Starting deployment ...

+-------------------+------------+
| Job ID | 8 |
| Environment | production |
| Target | Myapp |
| Concurrency Limit | None |
| Nodes | 2 |
+-------------------+------------+

Application instances: 1
 - Myapp[myapp]

Node run order (nodes in level 0 run before their dependent nodes in level 1, etc.):
0 -----------------------------------------------------------------------
mysql
 Myapp[myapp] - Myapp::Db[myapp]

1 -----------------------------------------------------------------------
appserver
 Myapp[myapp] - Myapp::Web[myapp]

New job created: 8
Started puppet run on mysql ...
Finished puppet run on mysql - Success!
 Resource events: 0 failed 4 changed 32 unchanged 0 skipped 0 noop
 Report: https://pe-puppet-master/#/run/jobs/8/nodes/mysql/report
Started puppet run on appserver ...
Finished puppet run on appserver - Success!
 Resource events: 0 failed 3 changed 130 unchanged 0 skipped 0 noop
 Report: https://pe-puppet-master/#/run/jobs/8/nodes/appserver/report

Success! 2/2 runs succeeded.
```

我们现在已经部署了一个非常简单的有序应用程序。在配置我们的`wordpress`服务器之前，数据库将完全启动。在下一个示例中，我们将允许多个`wordpress`服务器和多个负载均衡器来为我们的应用程序提供扩展。

# 添加负载均衡器并提供水平扩展

在许多情况下，我们希望我们的应用程序能够水平扩展。构建更多的节点可以让我们为更多的客户提供服务。这将是对之前应用程序的完全重写，并且还将整合`puppetlabs/app_modeling`模块来自 Forge。

# 依赖项

为了提供新的功能，我们需要从 Forge 获取`puppetlabs-haproxy`模块和`puppetlabs/app_modeling`模块。如果你正在使用`Puppetfile`，只需将它们添加到`Puppetfile`中。在以下示例中，我正在手动安装这些依赖项到现有的主服务器上：

```
[root@pe-puppet-master myapp]# puppet module install puppetlabs-haproxy
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└─┬ puppetlabs-haproxy (v2.1.0)
 ├── puppetlabs-concat (v4.2.1)
 └── puppetlabs-stdlib (v4.25.1)
[root@pe-puppet-master myapp]# puppet module install puppetlabs/app_modeling
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└─┬ puppetlabs-app_modeling (v0.2.0)
 └── puppetlabs-stdlib (v4.25.1)
```

我们现在可以通过`app_modeling`构建`haproxy`节点和新的应用程序编排功能。

# 构建

我们将从`site.pp`开始，再次从端点建模我们的应用程序。我添加了两条额外的服务行，确保为了本节课的目的防火墙被禁用。我们还可以考虑使用`puppetlabs/firewall`来管理我们的防火墙，并为防火墙生成和使用 FQDN。在以下示例中，你会注意到一些事情：

+   我们将`dbpass`变量传递给应用程序。这个变量可以存储在 Hiera 中，并通过 EYAML 加密。

+   我们有两个`wordpress`节点和两个`haproxy`节点，它们在`appserver`中都有各自独特的名称。

```
# For the purposes of this demo, the next two lines can be used to ensure firewalls
# are off for all CentOS nodes.

service {'iptables': ensure => stopped }
service {'firewalld': ensure => stopped }

site {
  myapp { 'myapp':
    dbpass => 'rarypass',
    nodes => {
      Node['mysql']       => [ Myapp::Db['myapp']],
      Node['wordpress']   => [ Myapp::Web['myapp-1']],
      Node['wordpress-2'] => [ Myapp::Web['myapp-2']],
      Node['haproxy']     => [ Myapp::Lb['myapp-1']],
      Node['haproxy-2']   => [ Myapp::Lb['myapp-2']],
    }
  }
}
```

在声明应用程序后，我们可以建模我们的`init.pp`来声明整个应用程序。这个应用程序包含了很多内容，因此请注意以下几点：

+   提供了五个变量，`db`变量在`DB`和`App`中都被使用。

+   `Myapp::Db`生成一个数据库。

+   `Myapp::Web`消耗一个数据库并生成一个 HTTP 服务资源。

+   我们使用来自`puppetlabs/app_modeling`的`collect_component_titles`函数提供一个我们可以迭代的数组。我们通过`$nodes`收集附加到`Myapp::Web`和`Myapp::Lb`的节点。这些值被命名为`allwebs`和`alllbs`。

+   我们使用来自`puppetlabs/stdlib`的`map`函数操作`$allwebs`。在这个`map`函数中，我们将每个节点名称转化为`Http["web-${wordpress_name}"]`的值，其中`$wordpress_name`是附加到`Myapp::Web`应用程序的每个节点的名称。我们将这个值作为每个`MyApp::Web`声明的导出。

+   我们将`$http (Http["web-${wordpress_name}"])`的值返回给`$https`数组，以便我们可以在负载均衡器上使用这些值。

+   我们的负载均衡器使用`each`语句代替`map`语句，因为我们不需要转换任何数据：

```
application myapp (
  $dbuser = 'wordpress',
  $dbpass = 'w0rdpr3ss!',
  $dbname = 'wordpress',
  $webpath = '/var/www/wordpress',
  $webport = '80'
) {

  myapp::db { $name:
    dbuser => $dbuser,
    dbpass => $dbpass,
    dbname => $dbname,
    export => Database["db-${name}"],
  }

# This section can be confusing, but here is essentially what's going on
# $allwebs is an array full of every node assigned to Myapp::Web in our application
# $https takes that $allwebs array of every node, creates a service resource,
# adds myapp::web to each node providing values for that service resource, and then
# returns all transformed service resource names back to the array.

# We're transforming each node listed in our site.pp into an array of Http[<nodename>]
# resource calls. And on each node we'll apply our defined type inside of the
# same map.

  $allwebs = collect_component_titles($nodes, Myapp::Web)

  $https = $allwebs.map |$wordpress_name| {

    $http = Http["web-${wordpress_name}"]

    myapp::web { "$wordpress_name":
      dbuser => $dbuser,
      dbpass => $dbpass,
      dbname => $dbname,
      webport => $webport,
      webpath => $webpath,
      consume => Database["db-${name}"],
      export => $http,
    }

    $http

  }

# We'll use an each statement here instead of a map, because we don't need
# any Load balancer values returned. They're the end of the chain. Our each
# statement covers each node, and $https from before is used to add nodes
# to the load balancer

  $alllbs = collect_component_titles($nodes, Myapp::Lb)

  $alllbs.each |$load_balancer| {

    myapp::lb { "${load_balancer}":
      balancermembers => $https,
      require => $https,
      port => '80',
      balance_mode => 'roundrobin',
    }

  }

}
```

我们的`myapp::db`生成一个 MySQL 服务器，并创建一个数据库来为我们的应用程序提供服务。我们在`init.pp`中使用`dbuser`、`dbpass`和`dbname`的值。请特别注意生成行，使用`app_modeling`服务资源来生成清单底部的数据库：

+   生成一个主机，从将被 Web 应用程序消耗的机器的 FQDN 中提取。

+   生成一个端口，虽然我们的 Web 清单中不使用该端口，但它提供了一个可用性测试，用于我们的应用程序编排节点。在能够通过 FQDN 上的端口`3306`连接到数据库之前，Web 的应用程序编排不会触发。如果没有这个声明，它将默认为`5432`，这是 Postgres 服务器的默认端口：

```
define myapp::db (
  $dbuser,
  $dbpass,
  $dbname,
){

  class {'::mysql::server':
    root_password => 'Sup3rp@ssword!',
    override_options => {
      'mysqld' => {
        'bind-address' => '0.0.0.0'
      }
    }
  }

  mysql::db { $dbname:
    user => $dbuser,
    password => $dbpass,
    host => '%',
    grant => ['ALL'],
  }
}
# This produces line is producing 2 values: host and port. We'll use host directly
# on Myapp::Web, but the port designator is used to pass the Resource Type test for
# Database using puppetlabs/app_modeling. Without the port, the test will fail to find
# the upstream Database and won't finish the agent run.
Myapp::Db produces Database {
  host => $::fqdn,
  port => '3306',

}
```

我们的`Myapp::Web`调用将使用来自初始应用程序的五个变量，但从所消耗的资源中接收数据库主机。请注意以下几点：

+   `$dbhost`的值由消耗的数据库填充。在底部，我们明确地将`$dbhost`的值映射到`Myapp::Web`消耗的`$host`值，并将其传递给`Myapp::Db`。

+   我们将由消费提供的`$dbhost`传递给`wordpress`类，实现与远程数据库的自动连接。

+   `Myapp::Web`生成一个 HTTP 资源，提供主机、端口、IP 和状态码。我们将使用主机、端口和 IP 为我们的负载均衡器提供服务，而`status_codes`则是另一个可用性测试，以确保由`haproxy`提供服务的网站状态码为`302`或`200`：

```
define myapp::web (
  $webpath,
  $webport,
  $dbuser,
  $dbpass,
  $dbhost,
  $dbname,
  ) {

    package {['php','mysql','php-mysql','php-gd']:
      ensure => installed,
    }

    class {'apache':
      default_vhost => false
    }

    include ::apache::mod::php

    apache::vhost { $::fqdn:
      port => $webport,
      docroot => $webpath,
      require => [File[$webpath]],
    }

    file { $webpath:
      ensure => directory,
      owner => 'apache',
      group => 'apache',
      require => Package['httpd'],
    }

    class { '::wordpress':
      db_user => $dbuser,
      db_password => $dbpass,
      db_host => $dbhost,
      db_name => $dbname,
      create_db => false,
      create_db_user => false,
      install_dir => $webpath,
      wp_owner => 'apache',
      wp_group => 'apache',
    }
  }
Myapp::Web consumes Database {
  dbhost => $host,
}
Myapp::Web produces Http {
  host => $::clientcert,
  port => $webport,
  ip => $::networking['interfaces']['enp0s8']['ip'],
  # Like the port parameter in the Database provider, we'll need to send the status_codes
  # flag to the Http provider to ensure we don't only accept a 302 status code.
  # A new wordpress application sends status code 200, so we'll let it through as well.
  status_codes => ['302','200'],
}
```

`Myapp::Lb` 实际上并不会消费或导出任何资源。我们构建了一个`haproxy::listen`服务，然后对于每个`balancermember`，我们导入了上述的值。在我们的应用声明中，我们将每个语句应用于`$https`数组中的每个成员，以下代码将这些数据转化为相关的负载均衡器。我们从每个`myapp::web`中获取主机、端口和 IP，并将其作为成员添加到我们的`haproxy::listen`中：

```
define myapp::lb (
  $balancermembers,
  String $ipaddress = '0.0.0.0',
  String $balance_mode = 'roundrobin',
  String $port = '80',
) {

  include haproxy

  haproxy::listen {"wordpress-${name}":
    collect_exported => false,
    ipaddress => $::networking['interfaces']['enp0s8']['ip'],
    mode => 'http',
    options => {
      'balance' => $balance_mode,
    },
    ports => $port,
  }

  $balancermembers.each |$member| {
    haproxy::balancermember { $member['host']:
      listening_service => "wordpress-${name}",
      server_names => $member['host'],
      ipaddresses => $member['ip'],
      ports => $member['port'],
    }
  }

}
```

# 部署

部署我们的新应用程序使用与之前相同的命令。我们将使用`puppet app show`来提供带排序的节点列表。你会看到，我们的单个数据库会产生一个数据库；每个 webapp 使用该数据库并生成一个 HTTP 服务资源，最终被每个负载均衡器使用：

```
[root@pe-puppet-master manifests]# puppet app show
Myapp[myapp]
 Myapp::Db[myapp] => mysql
 + produces Database[db-myapp]
 Myapp::Web[myapp-1] => appserver
 + produces Http[web-myapp-1]
 consumes Database[db-myapp]
 Myapp::Web[myapp-2] => appserver2
 + produces Http[web-myapp-2]
 consumes Database[db-myapp]
 Myapp::Lb[myapp-1] => haproxy
 consumes Http[web-myapp-1]
 consumes Http[web-myapp-2]
 Myapp::Lb[myapp-2] => haproxy2
 consumes Http[web-myapp-1]
 consumes Http[web-myapp-2]
```

在我们启动应用程序之前，我们可以运行一个`puppet job plan`来了解运行过程中排序的情况：

```
[root@pe-puppet-master manifests]# puppet job plan --application Myapp --environment production

+-------------------+------------+
| Environment | production |
| Target | Myapp |
| Concurrency Limit | None |
| Nodes | 5 |
+-------------------+------------+

Application instances: 1
 - Myapp[myapp]

Node run order (nodes in level 0 run before their dependent nodes in level 1, etc.):
0 -----------------------------------------------------------------------
mysql
 Myapp[myapp] - Myapp::Db[myapp]

1 -----------------------------------------------------------------------
wordpress
 Myapp[myapp] - Myapp::Web[myapp-1]
wordpress2
 Myapp[myapp] - Myapp::Web[myapp-2]

2 -----------------------------------------------------------------------
haproxy
 Myapp[myapp] - Myapp::Lb[myapp-1]
haproxy2
 Myapp[myapp] - Myapp::Lb[myapp-2]

Use `puppet job run --application 'Myapp' --environment production` to create and run a job like this
```

最后，我们运行我们的应用程序，看到 MySQL 被先配置，然后是我们的`wordpress`实例，接着是负载均衡器。感谢`puppetlabs/app_modeling`提供的服务资源，我们也知道，在`wordpress`服务器之前，数据库已经被主动访问，而且我们的`wordpress`服务器在负载均衡器配置之前会产生 302 状态码：

```
[root@pe-puppet-master production]# puppet job run --application Myapp --environment production --verbose
Starting deployment ...

+-------------------+------------+
| Job ID | 42 |
| Environment | production |
| Target | Myapp |
| Concurrency Limit | None |
| Nodes | 5 |
+-------------------+------------+

Application instances: 1
 - Myapp[myapp]

Node run order (nodes in level 0 run before their dependent nodes in level 1, etc.):
0 -----------------------------------------------------------------------
mysql
 Myapp[myapp] - Myapp::Db[myapp]

1 -----------------------------------------------------------------------
wordpress
 Myapp[myapp] - Myapp::Web[myapp-1]
wordpress-2
 Myapp[myapp] - Myapp::Web[myapp-2]

2 -----------------------------------------------------------------------
haproxy
 Myapp[myapp] - Myapp::Lb[myapp-1]
haproxy-2
 Myapp[myapp] - Myapp::Lb[myapp-2]

New job created: 42
Started puppet run on mysql ...
Finished puppet run on mysql - Success!
 Resource events: 0 failed 9 changed 27 unchanged 0 skipped 0 noop
 Report: https://pe-puppet-master/#/run/jobs/42/nodes/mysql/report
Started puppet run on wordpress-2 ...
Started puppet run on wordpress ...
Finished puppet run on wordpress-2 - Success!
 Resource events: 0 failed 81 changed 66 unchanged 0 skipped 0 noop
 Report: https://pe-puppet-master/#/run/jobs/42/nodes/wordpress-2/report
Finished puppet run on wordpress - Success!
 Resource events: 0 failed 81 changed 66 unchanged 0 skipped 0 noop
 Report: https://pe-puppet-master/#/run/jobs/42/nodes/wordpress/report
Started puppet run on haproxy-2 ...
Started puppet run on haproxy ...
Finished puppet run on haproxy - Success!
 Resource events: 0 failed 4 changed 30 unchanged 0 skipped 0 noop
 Report: https://pe-puppet-master/#/run/jobs/42/nodes/haproxy/report
Finished puppet run on haproxy-2 - Success!
 Resource events: 0 failed 4 changed 30 unchanged 0 skipped 0 noop
 Report: https://pe-puppet-master/#/run/jobs/42/nodes/haproxy-2/report

Success! 5/5 runs succeeded.
Duration: 58 sec
```

# 总结

在本章中，我们学习了如何使用应用程序编排来排序我们的应用程序。这建立在我们编写 Puppet 代码时学到的基础知识上，甚至在使用导出资源时也是如此。当我们构建更多应用程序和对象进行配置时，我们需要确保我们的 Puppet Master 能够为所有这些节点提供服务。

在下一章中，我们将讨论如何水平和垂直扩展 Puppet Enterprise。

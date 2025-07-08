# 第三章。编写更好的清单

|   | *“通过代码行数衡量编程进展，就像通过重量衡量飞机建造进展一样。”* |   |
| --- | --- | --- |
|   | --*比尔·盖茨* |

在本章中，我们将涵盖：

+   使用资源数组

+   使用资源默认值

+   使用定义类型

+   使用标签

+   使用运行阶段

+   使用角色和配置文件

+   向类传递参数

+   从 Hiera 传递参数

+   编写可重用的跨平台清单

+   获取有关环境的信息

+   导入动态信息

+   向 shell 命令传递参数

# 介绍

你的 Puppet 清单是你整个基础设施的活文档。保持它们整洁和有序是维护和理解的好方法。Puppet 为你提供了以下工具来实现这一点：

+   数组

+   默认值

+   定义类型

+   依赖关系

+   类参数

我们将看到如何使用这些工具以及更多内容。在阅读本章时，尝试运行示例并查看你自己的清单，看看这些功能如何帮助你简化和改进 Puppet 代码。

# 使用资源数组

你可以对资源做的任何事，也可以对资源数组做。利用这个思路来重构你的清单，使它们更简洁明了。

## 如何操作…

下面是使用资源数组重构的步骤：

1.  在你的清单中找出一个包含多个相同类型资源实例的类，例如，软件包：

    ```
      package { 'sudo' : ensure => installed }
      package { 'unzip' : ensure => installed }
      package { 'locate' : ensure => installed }
      package { 'lsof' : ensure => installed }
      package { 'cron' : ensure => installed }
      package { 'rubygems' : ensure => installed }
    ```

1.  将它们组合在一起，并用一个数组替换为单个软件包资源：

    ```
      package
      {
        [ 'cron',
        'locate',
        'lsof',
        'rubygems',
        'sudo',
        'unzip' ]:
        ensure => installed,
      }
    ```

## 如何操作…

Puppet 的大多数资源类型可以接受一个数组，而不是单个名称，并会为数组中的每个元素创建一个实例。你为资源提供的所有参数（例如，`ensure => installed`）都会分配给每个新的资源实例。这个简写方法只有在所有资源具有相同属性时才有效。

## 另见

+   第一章中的 *迭代多个项目* 配方，*Puppet 语言与风格*

# 使用资源默认值

Puppet 模块是一组相关的资源，通常用于配置特定服务。在模块中，你可以定义多个资源；资源默认值允许你为资源指定默认属性值。在此示例中，我们将展示如何为 `File` 类型指定资源默认值。

## 如何操作…

为了展示如何使用资源默认值，我们将创建一个 apache 模块。在这个模块中，我们将指定默认的拥有者和组是 `apache` 用户，如下所示：

1.  创建一个 apache 模块并为 `File` 类型创建一个资源默认值：

    ```
      class apache {
        File {
          owner => 'apache',
          group => 'apache',
          mode => 0644,
        }
      }
    ```

1.  在 `/var/www/html` 目录下创建 html 文件：

    ```
      file {'/var/www/html/index.html':
        content => "<html><body><h1><a
          href='cookbook.html'>Cookbook!
          </a></h1></body></html>\n",
      }
      file {'/var/www/html/cookbook.html':
        content =>
          "<html><body><h2>PacktPub</h2></body></html>\n",
      }

    ```

1.  将此类添加到你的默认节点定义中，或者使用`puppet apply`将模块应用到你的节点。我将使用我们在上一章中配置的方法，将代码推送到 Git 仓库，并使用 Git 钩子将代码部署到 Puppet 主节点，如下所示：

    ```
    t@mylaptop ~/puppet $ git pull origin production
    From git.example.com:repos/puppet
     * branch            production -> FETCH_HEAD
    Already up-to-date.
    t@mylaptop ~/puppet $ cd modules
    t@mylaptop ~/puppet/modules $ mkdir -p apache/manifests
    t@mylaptop ~/puppet/modules $ vim apache/manifests/init.pp
    t@mylaptop ~/puppet/modules $ cd ..
    t@mylaptop ~/puppet $ vim manifests/site.pp 
    t@mylaptop ~/puppet $ git status
    On branch production
    Changes not staged for commit:
    modified:   manifests/site.pp
    Untracked files:
    modules/apache/
    t@mylaptop ~/puppet $ git add manifests/site.pp modules/apache
    t@mylaptop ~/puppet $ git commit -m 'adding apache module'
    [production d639a86] adding apache module
     2 files changed, 14 insertions(+)
     create mode 100644 modules/apache/manifests/init.pp
    t@mylaptop ~/puppet $ git push origin production
    Counting objects: 13, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (6/6), done.
    Writing objects: 100% (8/8), 885 bytes | 0 bytes/s, done.
    Total 8 (delta 0), reused 0 (delta 0)
    remote: To puppet@puppet.example.com:/etc/puppet/environments/puppet.git
    remote:    832f6a9..d639a86  production -> production
    remote: Already on 'production'
    remote: From /etc/puppet/environments/puppet
    remote:    832f6a9..d639a86  production -> origin/production
    remote: Updating 832f6a9..d639a86
    remote: Fast-forward
    remote:  manifests/site.pp                |    1 +
    remote:  modules/apache/manifests/init.pp |   13 +++++++++++++
    remote:  2 files changed, 14 insertions(+)
    remote:  create mode 100644 modules/apache/manifests/init.pp
    To git@git.example.com:repos/puppet.git
     832f6a9..d639a86  production -> production

    ```

1.  将模块应用到节点或运行 Puppet：

    ```
    Notice: /Stage[main]/Apache/File[/var/www/html/cookbook.html]/ensure: defined content as '{md5}493473fb5bde778ca93d034900348c5d'
    Notice: /Stage[main]/Apache/File[/var/www/html/index.html]/ensure: defined content as '{md5}184f22c181c5632b86ebf9a0370685b3'
    Notice: Finished catalog run in 2.00 seconds
    [root@hiera-test ~]# ls -l /var/www/html
    total 8
    -rw-r--r--. 1 apache apache 44 Sep 15 12:00 cookbook.html
    -rw-r--r--. 1 apache apache 73 Sep 15 12:00 index.html

    ```

## 它是如何工作的……

我们定义的资源默认值指定了此类中的所有文件资源的所有者、组和模式（也称为此范围内的所有者、组和模式）。除非你特别覆盖资源默认值，否则属性的值将从默认值中获取。

## 还有更多……

你可以为任何资源类型指定资源默认值。你还可以在`site.pp`中指定资源默认值。我发现指定`Package`和`Service`资源的默认操作非常有用，如下所示：

```
  Package { ensure => 'installed' }
  Service {
    hasrestart => true,
    enable     => true,
    ensure     => true,
  }
```

使用这些默认值时，每当你指定一个软件包时，该软件包将被安装。每当你指定一个服务时，该服务将启动并启用在启动时运行。这些是你指定软件包和服务的常见原因，大多数时候这些默认值会做你希望的事情，你的代码也会更加简洁。当你需要禁用一个服务时，只需覆盖默认值。

# 使用定义类型

在上一个示例中，我们看到如何通过将相同的资源分组到数组中来减少冗余代码。然而，这种技术仅限于所有参数相同的资源。当你有一组共享一些参数的资源时，你需要使用定义类型将它们组合在一起。

## 如何操作……

以下步骤将向你展示如何创建定义：

1.  将以下代码添加到你的清单中：

    ```
      define tmpfile() {
        file { "/tmp/${name}": content => "Hello, world\n",
        }
      }
      tmpfile { ['a', 'b', 'c']: }
    ```

1.  运行 Puppet：

    ```
    [root@hiera-test ~]# vim tmp.pp
    [root@hiera-test ~]# puppet apply tmp.pp 
    Notice: Compiled catalog for hiera-test.example.com in environment production in 0.11 seconds
    Notice: /Stage[main]/Main/Tmpfile[a]/File[/tmp/a]/ensure: defined content as '{md5}a7966bf58e23583c9a5a4059383ff850'
    Notice: /Stage[main]/Main/Tmpfile[b]/File[/tmp/b]/ensure: defined content as '{md5}a7966bf58e23583c9a5a4059383ff850'
    Notice: /Stage[main]/Main/Tmpfile[c]/File[/tmp/c]/ensure: defined content as '{md5}a7966bf58e23583c9a5a4059383ff850'
    Notice: Finished catalog run in 0.09 seconds
    [root@hiera-test ~]# cat /tmp/{a,b,c}
    Hello, world
    Hello, world
    Hello, world

    ```

## 它是如何工作的……

你可以把定义类型（通过`define`关键字引入）看作一个模具。它描述了一个模式，Puppet 可以用来创建许多相似的资源。每次你在清单中声明一个`tmpfile`实例时，Puppet 会插入所有包含在`tmpfile`定义中的资源。

在我们的例子中，`tmpfile`的定义包含一个`file`资源，其内容为`Hello, world\n`，路径为`/tmp/${name}`。如果你声明了一个名为`foo`的`tmpfile`实例：

```
tmpfile { 'foo': }
```

Puppet 会创建一个路径为`/tmp/foo`的文件。换句话说，定义中的`${name}`将被 Puppet 被要求创建的任何实际实例的`name`替换。它几乎就像我们创建了一种新类型的资源：`tmpfile`，它有一个参数——它的`name`。

就像常规资源一样，我们不必仅传递一个标题；如前面的示例所示，我们可以提供一个标题数组，Puppet 将创建所需数量的资源。

### 提示

关于名称，namevar：你创建的每个资源必须有一个唯一的名称，namevar。这与标题不同，标题是 Puppet 在内部引用资源的方式（尽管它们通常是相同的）。

## 还有更多……

在这个例子中，我们创建了一个定义，其中唯一在实例之间变化的参数是`name`参数。但我们可以添加任何我们想要的参数，只要我们在`name`参数后面的括号中声明它们，如下所示：

```
  define tmpfile($greeting) {
    file { "/tmp/${name}": content => $greeting,
    }
  }
```

接下来，在声明资源实例时为其传递值：

```
  tmpfile{ 'foo':
    greeting => "Good Morning\n",
  }
```

你可以将多个参数声明为逗号分隔的列表：

```
  define webapp($domain,$path,$platform) {
    ...
  }
  webapp { 'mywizzoapp':
    domain   => 'mywizzoapp.com',
    path     => '/var/www/apps/mywizzoapp',
    platform => 'Rails',
  }
```

你还可以为任何未提供的参数声明默认值，从而使它们成为可选参数：

```
  define tmpfile($greeting,$mode='0644') {
    ...
  }
```

这是一种强大的技术，用于抽象出某些资源的共同部分，并将其保存在一个地方，以便你*不要重复自己*。在前面的例子中，`webapp`中可能包含许多独立的资源：软件包、配置文件、源代码检出、虚拟主机等。但它们对于每个`webapp`实例都是相同的，除了我们提供的参数。这些参数可能会在模板中引用，例如，用于设置虚拟主机的域名。

## 另见

+   本章中的*将参数传递给类*示例

# 使用标签

有时，一个 Puppet 类需要了解另一个类，或者至少需要知道它是否存在。例如，一个管理防火墙的类可能需要知道该节点是否为 Web 服务器。

Puppet 的`tagged`函数会告诉你一个命名的类或资源是否出现在该节点的目录中。你还可以向节点或类应用任意标签，并检查这些标签的存在。标签是另一个元参数，类似于我们在第一章中介绍的`require`和`notify`，*Puppet 语言与风格*。元参数用于 Puppet 目录的编译，但不是附加到资源上的属性。

## 如何实现……

为了帮助你发现自己是否在特定节点或节点类上运行，所有节点都会自动被标记为节点名称及其包含的任何类的名称。以下是一个例子，展示如何使用`tagged`获取这些信息：

1.  将以下代码添加到你的`site.pp`文件中（将`cookbook`替换为你机器的`hostname`）：

    ```
      node 'cookbook' {
        if tagged('cookbook') {
          notify { 'tagged cookbook': }
        }
      }
    ```

1.  运行 Puppet：

    ```
    root@cookbook:~# puppet agent -vt
    Info: Caching catalog for cookbook
    Info: Applying configuration version '1410848350'
    Notice: tagged cookbook
    Notice: Finished catalog run in 1.00 seconds

    ```

    节点也会自动标记它们所包含的所有类的名称，以及其他一些自动标签。你可以使用`tagged`来查找节点上包含了哪些类。

    你不仅仅限于检查 Puppet 自动应用的标签。你还可以添加自己的标签。要在节点上设置任意标签，请使用`tag`函数，如以下示例所示：

1.  按照以下方式修改你的`site.pp`文件：

    ```
      node 'cookbook' {
        tag('tagging')
        class {'tag_test': }
      }
    ```

1.  添加一个`tag_test`模块，并提供以下`init.pp`（或者懒惰一些，将以下定义添加到你的`site.pp`中）：

    ```
      class tag_test {
        if tagged('tagging') {
          notify { 'containing node/class was tagged.': }
        }
      }
    ```

1.  运行 Puppet：

    ```
    root@cookbook:~# puppet agent -vt
    Info: Caching catalog for cookbook
    Info: Applying configuration version '1410851300'
    Notice: containing node/class was tagged.
    Notice: Finished catalog run in 0.22 seconds

    ```

1.  你还可以使用标签来确定应用清单的哪些部分。如果你在 Puppet 命令行中使用 `--tags` 选项，Puppet 将仅应用那些带有你指定标签的类或资源。例如，我们可以用两个类来定义 `cookbook` 类：

    ```
      node cookbook {
        class {'first_class': }
        class {'second_class': }
      }
      class first_class {
        notify { 'First Class': }
      }
      class second_class {
        notify {'Second Class': }
      }
    ```

1.  现在，当我们在 `cookbook` 节点上运行 `puppet agent` 时，我们会看到两个通知：

    ```
    root@cookbook:~# puppet agent -t
    Notice: Second Class
    Notice: First Class
    Notice: Finished catalog run in 0.22 seconds

    ```

1.  现在将 `first_class` 和 `add --tags` 函数应用于命令行：

    ```
    root@cookbook:~# puppet agent -t --tags first_class
    Notice: First Class
    Notice: Finished catalog run in 0.07 seconds

    ```

## 还有更多...

你可以使用标签创建一个资源集合，然后将该集合作为其他资源的依赖。例如，假设某个服务依赖于由多个文件片段构建的配置文件，如下所示：

```
  class firewall::service {
    service { 'firewall': ... 
    }
    File <| tag == 'firewall-snippet' |> ~> Service['firewall'] 
  }
  class myapp { 
    file { '/etc/firewall.d/myapp.conf': tag => 'firewall-snippet', ... 
    } 
  }
```

在这里，我们指定如果任何标记为 `firewall-snippet` 的文件资源被更新，`firewall` 服务应收到通知。我们只需要将特定应用或服务的配置片段标记为 `firewall-snippet`，Puppet 就会完成其余工作。

尽管我们可以为每个片段资源添加一个 `notify => Service["firewall"]` 函数，但如果 `firewall` 服务的定义发生变化，我们将不得不逐一查找并更新所有片段。标签允许我们将逻辑封装在一个地方，使得将来的维护和重构变得更容易。

### 注意

`<| tag == 'firewall-snippet' |>` 语法是什么？这是所谓的资源收集器，它通过搜索关于资源的一些数据来指定一组资源；在这个例子中，是标签的值。你可以在 Puppet Labs 网站上了解更多关于资源收集器和 `<| |>` 运算符（有时称为飞船操作符）的信息：[`docs.puppetlabs.com/puppet/3/reference/lang_collectors.html`](http://docs.puppetlabs.com/puppet/3/reference/lang_collectors.html)。

# 使用运行阶段

一个常见的需求是在其他组之前应用某一组资源（例如，安装软件包仓库或自定义 Ruby 版本），或者在其他组之后应用（例如，在安装完依赖项后部署应用）。Puppet 的运行阶段功能使你能够实现这一点。

默认情况下，清单中的所有资源都应用于一个名为 `main` 的单一阶段。如果你需要某个资源在其他资源之前应用，你可以将其分配到一个新的运行阶段，该阶段指定在 `main` 之前运行。类似地，你也可以定义一个在 `main` 之后运行的阶段。实际上，你可以根据需要定义任意数量的运行阶段，并告诉 Puppet 它们应该按什么顺序应用。

在这个示例中，我们将使用阶段来确保一个类首先应用，另一个类最后应用。

## 如何做...

以下是使用运行 `stages` 的示例步骤：

1.  创建文件 `modules/admin/manifests/stages.pp`，内容如下：

    ```
      class admin::stages {
        stage { 'first': before => Stage['main'] }
        stage { 'last': require => Stage['main'] }
        class me_first {
          notify { 'This will be done first': }
        }
        class me_last {
          notify { 'This will be done last': }
        }
        class { 'me_first':
          stage => 'first',
        }
        class { 'me_last':
          stage => 'last',
        }
      }
    ```

1.  修改你的 `site.pp` 文件，如下所示：

    ```
      node 'cookbook' {
        class {'first_class': }
        class {'second_class': }
        include admin::stages
      }
    ```

1.  运行 Puppet：

    ```
    root@cookbook:~# puppet agent -t
    Info: Applying configuration version '1411019225'
    Notice: This will be done first
    Notice: Second Class
    Notice: First Class
    Notice: This will be done last
    Notice: Finished catalog run in 0.43 seconds

    ```

## 它是如何工作的…

让我们详细检查一下这段代码，看看发生了什么。首先，我们声明了运行阶段 `first` 和 `last`，如下所示：

```
  stage { 'first': before => Stage['main'] }
  stage { 'last': require => Stage['main'] }
```

对于`first`阶段，我们已经指定它应该在`main`之前执行。也就是说，标记为`first`阶段的每个资源将在任何`main`阶段的资源之前应用（`main`是默认阶段）。

`last`阶段需要`main`阶段，因此在`main`阶段的每个资源都应用后，才能应用`last`阶段的资源。

然后我们声明一些类，稍后将为这些类分配运行阶段：

```
  class me_first {
    notify { 'This will be done first': }
  }
  class me_last {
    notify { 'This will be done last': }
  }
```

现在我们可以将所有内容结合起来，并在节点上包含这些类，同时为每个类指定运行阶段：

```
  class { 'me_first': stage => 'first',
  }
  class { 'me_last': stage => 'last',
  }
```

请注意，在`me_first`和`me_last`的`class`声明中，我们不需要指定它们采用`stage`参数。`stage`参数是另一个元参数，意味着它可以应用于任何类或资源，而无需显式声明。当我们在 Puppet 节点上运行`puppet agent`时，`me_first`类中的通知会在`first_class`和`second_class`中的通知之前应用。`me_last`类的通知会在`main`阶段之后应用，因此它会在`first_class`和`second_class`的两个通知之后应用。如果你多次运行`puppet agent`，你会看到`first_class`和`second_class`的通知可能不会总是按相同的顺序出现，但`me_first`类将始终最先出现，`me_last`类将始终最后出现。

## 还有更多内容…

你可以定义任意数量的运行阶段，并为它们设置任何顺序。这可以大大简化一个复杂的清单，否则需要在资源之间设置大量显式依赖关系。然而要小心，避免意外引入依赖循环；当你将某个资源分配到一个运行阶段时，你自动将它与之前阶段中的所有内容关联起来。

你可能希望在`site.pp`文件中定义阶段，这样在清单的顶部就能清楚地看到哪些阶段是可用的。

*Gary Larizza* 在他的网站上写了一个有用的关于使用运行阶段的介绍，包含一些现实世界的示例：

[`garylarizza.com/blog/2011/03/11/using-run-stages-with-puppet/`](http://garylarizza.com/blog/2011/03/11/using-run-stages-with-puppet/)

一个警告：许多人不喜欢使用运行阶段，觉得 Puppet 已经提供了足够的资源排序控制，并且不加区分地使用运行阶段会使您的代码变得非常难以理解。在可能的情况下，应尽量减少使用运行阶段。有几个关键的例子可以证明使用阶段可以减少复杂性。最明显的是当一个资源修改用于在系统上安装软件包的系统时。在默认的 `main` 阶段定义包时，您的清单可以依赖于存在更新的包管理配置信息。例如，对于基于 Yum 的系统，您可以创建一个在 `main` 阶段之前的 `yumrepos` 阶段。您可以使用链箭头指定这种依赖关系，如下面的代码片段所示：

```
  stage {'yumrepos': }
  Stage['yumrepos'] -> Stage['main']
```

然后，我们可以创建一个创建 Yum 仓库 (`yumrepo`) 资源并将其分配给 `yumrepos` 阶段的类，如下所示：

```
  class {'yums': stage => 'yumrepos',
  }
  class yums {
    notify {'always before the rest': }
    yumrepo {'testrepo': baseurl => 'file:///var/yum', ensure  => 'present',
    }
  }
```

对于基于 Apt 的系统，相同的例子将是一个定义 Apt 源的阶段。阶段的关键在于将其定义在您的 `site.pp` 文件中，这样它们就会非常可见，并且只在确保不会引入依赖循环的情况下才使用它们。

## 参见

+   *使用标签* 配方，在本章中

+   *绘制依赖图* 配方在 第十章，*监控、报告和故障排除*

# 使用角色和配置文件

组织良好的 Puppet 清单易于阅读；模块的目的应在其名称中明显。节点的目的应在一个单一类中定义。这个单一的类应包含执行该目的所需的所有类。Craig Dunn 写了一篇关于这种分类系统的文章，他将其称为 "角色和配置文件" ([`www.craigdunn.org/2012/05/239/`](http://www.craigdunn.org/2012/05/239/))。在这种模型中，角色是节点的单一目的，一个节点只能有一个角色，一个角色可以包含多个配置文件，而一个配置文件包含与单个服务相关的所有资源。在这个例子中，我们将创建一个使用多个配置文件的 web 服务器角色。

## 如何做到…

我们将创建两个模块来存储我们的角色和配置文件。角色将包含一个或多个配置文件。每个角色或配置文件将被定义为子类，例如 `profile::base`

1.  决定您的角色和配置文件的命名策略。在我们的例子中，我们将创建两个模块，`roles` 和 `profiles`，分别包含我们的角色和配置文件：

    ```
    $ puppet module generate thomas-profiles
    $ ln -s thomas-profiles profiles
    $ puppet module generate thomas-roles
    $ ln -s thomas-roles roles

    ```

1.  开始定义我们 `webserver` 角色的组成部分作为配置文件。为了保持这个例子简单，我们将创建两个配置文件。首先是一个 `base` 配置文件，包括我们的基本服务器配置类。其次是一个 `apache` 类来安装和配置 Apache Web 服务器 (`httpd`)，如下所示：

    ```
    $ vim profiles/manifests/base.pp
    class profiles::base {
     include base
    }
    $ vim profiles/manifests/apache.pp
    class profiles::apache {
     $apache = $::osfamily ? {
     'RedHat' => 'httpd',
     'Debian' => 'apache2',
     }
     service { "$apache":
     enable => true,
     ensure => true,
     }
     package { "$apache":
     ensure => 'installed',
     }
    }

    ```

1.  为我们的 `webserver` 角色定义一个 `roles::webserver` 类，如下所示：

    ```
    $ vim roles/manifests/webserver.pp
    class roles::webserver {
     include profiles::apache
     include profiles::base
    }

    ```

1.  将`roles::webserver`类应用到一个节点。在集中式安装中，你可以使用**外部节点分类器**（**ENC**）将类应用到节点，或者使用 Hiera 来定义角色：

    ```
      node 'webtest' {
        include roles::webserver
      }
    ```

## 它是如何工作的……

将 Web 服务器配置的各个部分分解成不同的配置文件，使我们能够独立地应用这些部分。我们创建了一个基本配置文件，可以扩展以包括我们希望应用到所有节点的所有资源。我们的`roles::webserver`类简单地包含了`base`和`apache`类。

## 还有更多内容……

正如我们将在下一节中看到的，我们可以将参数传递给类，以改变它们的工作方式。在我们的`roles::webserver`类中，我们可以使用类实例化语法，而不是`include`，并通过类中的`parameters`覆盖它。例如，要将参数传递给`base`类，我们可以使用：

```
  class {'profiles::base':
    parameter => 'newvalue'
  }
```

在我们之前使用过的地方：

```
include profiles::base
```

### 提示

在本书的早期版本中，节点和类继承用于实现类似的目标，即代码重用。节点继承在 Puppet 版本 3.7 及更高版本中已被弃用。应该避免使用节点和类继承。使用角色和配置文件可以达到相同的可读性，并且更容易跟随。

# 向类传递参数

有时候，将类的某些方面参数化非常有用。例如，你可能需要管理不同版本的`gem`包，而不必为每个不同版本号创建单独的类，你可以将版本号作为参数传入。

## 如何操作……

在这个例子中，我们将创建一个接受参数的定义：

1.  将参数声明为类定义的一部分：

    ```
      class eventmachine($version) {
        package { 'eventmachine': provider => gem, ensure   => $version,
        }
      }
    ```

1.  使用以下语法将类包含在节点上：

    ```
      class { 'eventmachine':
        version => '1.0.3',
      }
    ```

## 它是如何工作的……

类定义`class eventmachine($version) {`就像普通的类定义，唯一不同的是它指定了该类需要一个参数：`$version`。在类内部，我们定义了一个`package`资源：

```
  package { 'eventmachine':
    provider => gem,
    ensure   => $version,
  }

```

这是一个`gem`包，我们请求安装版本`$version`。

在节点上包含类，而不是使用通常的`include`语法：

```
include eventmachine
```

这样做时，会出现一个`class`语句：

```
  class { 'eventmachine':
    version => '1.0.3',
  }

```

这样做的效果是一样的，但同时也为参数`version`设置了一个值。

## 还有更多内容……

你可以为类指定多个参数，如下所示：

```
  class mysql($package, $socket, $port) {
```

然后以相同的方式提供它们：

```
  class { 'mysql':
    package => 'percona-server-server-5.5',
    socket  => '/var/run/mysqld/mysqld.sock',
    port    => '3306',
  }

```

### 指定默认值

你还可以为一些参数提供默认值。当你在不设置参数的情况下包含类时，将使用默认值。例如，如果我们创建了一个具有三个参数的`mysql`类，我们可以为任何或所有参数提供默认值，如代码片段所示：

```
class mysql($package, $socket, $port='3306') {
```

或者全部：

```
  class mysql(
    package = percona-server-server-5.5",
    socket  = '/var/run/mysqld/mysqld.sock',
    port    = '3306') {

```

默认值允许你使用一个默认值，并在需要时覆盖该默认值。

与定义不同，一个参数化类只能在一个节点上存在一个实例。因此，如果你需要多个不同的资源实例，应该改用`define`。

# 从 Hiera 传递参数

就像我们在上一章引入的 `defaults` 参数一样，Hiera 可用于为类提供默认值。此功能要求 Puppet 版本 3 及以上。

## 准备工作

安装并配置 `hiera`，就像我们在 第二章 *Puppet 基础设施* 中所做的那样。创建一个全局或公共的 `yaml` 文件；这将作为所有值的默认值。

## 如何操作…

1.  创建一个没有默认值的参数类：

    ```
    t@mylaptop ~/puppet $ mkdir -p modules/mysql/manifests t@mylaptop ~/puppet $ vim modules/mysql/manifests/init.pp
    class mysql ( $port, $socket, $package ) {
     notify {"Port: $port Socket: $socket Package: $package": }
    }

    ```

1.  在 Hiera 中更新你的公共 `.yaml` 文件，加入 `mysql` 类的默认值：

    ```
    ---
    mysql::port: 3306
    mysql::package: 'mysql-server'
    mysql::socket: '/var/lib/mysql/mysql.sock'

    ```

    将类应用于节点，你现在可以将 `mysql` 类添加到你的默认节点中。

    ```
    node default {
     class {'mysql': }
    }

    ```

1.  运行 `puppet agent` 并验证输出：

    ```
    [root@hiera-test ~]# puppet agent -t
    Info: Caching catalog for hiera-test.example.com
    Info: Applying configuration version '1411182251'
    Notice: Port: 3306 Socket: /var/lib/mysql/mysql.sock Package: mysql-server
    Notice: /Stage[main]/Mysql/Notify[Port: 3306 Socket: /var/lib/mysql/mysql.sock Package: mysql-server]/message: defined 'message' as 'Port: 3306 Socket: /var/lib/mysql/mysql.sock Package: mysql-server'
    Notice: Finished catalog run in 1.75 seconds

    ```

## 它是如何工作的...

当我们在清单中实例化 `mysql` 类时，并没有为任何属性提供值。Puppet 知道去 Hiera 查找与 `class_name::parameter_name:` 或 `::class_name::parameter_name:` 匹配的值。

当 Puppet 找到一个值时，它将其作为类的参数。如果 Puppet 未能在 Hiera 中找到值并且没有定义默认值，目录失败将导致以下命令行：

```
Error: Could not retrieve catalog from remote server: Error 400 on SERVER: Must pass package to Class[Mysql] at /etc/puppet/environments/production/manifests/site.pp:6 on node hiera-test.example.com

```

这个错误表示 Puppet 需要为参数 `package` 提供一个值。

## 还有更多...

你可以定义一个 Hiera 层次结构，并根据事实为参数提供不同的值。例如，你可以在层次结构中使用 `%{::osfamily}`，并根据 `osfamily` 参数（如 RedHat、Suse 和 Debian）有不同的 `yaml` 文件。

# 编写可重用的跨平台清单

每个系统管理员都梦想拥有统一的、同质化的基础设施，所有机器都运行相同版本的相同操作系统。然而，正如生活中的其他领域一样，现实通常是混乱的，并且与计划不符。

你可能负责一堆不同年龄和架构的服务器，这些服务器运行着不同内核的不同操作系统，通常分布在不同的数据中心和 ISP 上。

这种情况应该会让 SSH 中的系统管理员感到恐惧，因为在每台服务器上执行相同的命令可能会导致不同的、不可预测的，甚至是危险的结果。

我们当然应该努力使旧服务器保持最新，并尽可能在单一参考平台上工作，以简化管理、降低成本并提高可靠性。但在我们达到这个目标之前，Puppet 使得应对异构环境稍微容易一些。

## 如何操作…

下面是一些使你的清单更具可移植性的示例：

1.  当你需要将相同的清单应用于不同操作系统分发版的服务器时，主要的差异可能是包和服务的名称，以及配置文件的位置。尝试通过使用选择器设置全局变量，将所有这些差异捕捉到一个类中：

    ```
      $ssh_service = $::operatingsystem? { /Ubuntu|Debian/ => 'ssh', default         => 'sshd',
      }
    ```

    你不必担心清单中任何其他部分的差异；当你引用某些内容时，可以放心地使用变量，它将在每个环境中指向正确的内容：

    ```
      service { $ssh_service: ensure => running,
      }
    ```

1.  我们经常需要处理混合架构；这可能会影响共享库的路径，并且可能需要不同版本的软件包。同样，尽量将所有所需的设置封装在一个架构类中，该类设置全局变量：

    ```
      $libdir = $::architecture ? {
        /amd64|x86_64/   => '/usr/lib64', default => '/usr/lib',
      }
    ```

    然后你可以在清单中或甚至在模板中使用这些变量，任何需要架构相关值的地方：

    ```
    ; php.ini
    [PHP]
    ; Directory in which the loadable extensions (modules) reside.
    extension_dir = <%= @libdir %>/php/modules
    ```

## 它是如何工作的...

这种方法（可以称之为自上而下）的优点是你只需要做一次选择。另一种选择是自下而上的方法，即每次使用设置时，都需要有一个选择器或`case`语句：

```
  service { $::operatingsystem? {
    /Ubuntu|Debian/ => 'ssh', default         => 'sshd' }: ensure => running,
  }
```

这样不仅会导致大量重复，还会使代码更难以阅读。而且，当新操作系统被加入时，你需要在整个清单中进行修改，而不是仅仅在一个地方进行修改。

## 还有更多…

如果你正在为公共发布编写模块（例如，在 Puppet Forge 上），尽可能使你的模块跨平台，将使其对社区更有价值。尽可能在许多不同的发行版、平台和架构上进行测试，并添加适当的变量，以确保它能在各处运行。

如果你使用了一个公共模块并将其调整为适应自己的环境，考虑在认为这些更改可能对其他人有帮助时，将这些更改更新到公共版本中。

即使你不打算发布一个模块，也要记住它可能会在生产环境中使用很长时间，并且可能需要适应环境中的许多变化。如果从一开始就设计好应对这些变化的功能，将使你或最终维护你代码的人轻松许多。

|   | *"永远编写代码，假设维护你代码的人是一个暴力的精神病患者，知道你住在哪里。"* |   |
| --- | --- | --- |
|   | --*戴夫·卡哈特* |

## 另见

+   第七章中的*使用公共模块*配方，*管理应用程序*

+   第二章中的*配置 Hiera*配方，*Puppet 基础架构*

# 获取环境信息

在 Puppet 清单中，你经常需要了解你所在机器的一些本地信息。Facter 是与 Puppet 一起使用的工具，提供了一种标准的方法来从环境中获取信息（事实），例如以下内容：

+   操作系统

+   内存大小

+   架构

+   处理器数量

要查看系统上可用的事实的完整列表，运行：

```
$ sudo facter
architecture => amd64
augeasversion => 0.10.0
domain => compute-1.internal
ec2_ami_id => ami-137bcf7a
ec2_ami_launch_index => 0

```

### 注意

虽然从命令行获取这些信息很方便，但 Facter 的真正强大之处在于能够在 Puppet 清单中访问这些事实。

一些模块定义了自己的事实；要查看任何已本地定义的事实，可以在运行 facter 时添加`-p (pluginsync)`选项，如下所示：

```
$ sudo facter -p

```

## 如何实现…

以下是使用 Facter 事实在清单中的示例：

1.  像引用其他变量一样在清单中引用 Facter 事实。事实是 Puppet 中的全局变量，因此它们应以双冒号(`::`)为前缀，如以下代码片段所示：

    ```
    notify { "This is $::operatingsystem version $::operatingsystemrelease, on $::architecture architecture, kernel version $::kernelversion": }
    ```

1.  当 Puppet 运行时，它将为当前节点填充适当的值：

    ```
    [root@hiera-test ~]# puppet agent -t
    ...
    Info: Applying configuration version '1411275985'Notice: This is RedHat version 6.5, on x86_64 architecture, kernel version 2.6.32
    ...
    Notice: Finished catalog run in 0.40 seconds

    ```

## 它是如何工作的…

Facter 提供了一种标准方式，使清单能够获取它们所应用节点的信息。当你在清单中引用一个事实时，Puppet 将查询 Facter 以获取当前值并将其插入清单。Facter 事实是顶级作用域变量。

### 小贴士

始终使用前导的双冒号引用事实，以确保你使用的是事实而不是本地变量：

`$::hostname` 而不是 `$hostname`

## 还有更多…

你还可以在 ERB 模板中使用事实。例如，你可能想将节点的主机名插入到文件中，或者根据节点的内存大小更改应用程序的配置设置。当在模板中使用事实名称时，请记住它们不需要美元符号，因为这是 Ruby 而不是 Puppet：

```
$KLogPath <%= case @kernelversion when '2.6.31' then
'/var/run/rsyslog/kmsg' else '/proc/kmsg' end %>

```

引用事实时，请使用`@`语法。在与模板函数调用相同作用域中定义的变量也可以使用`@`语法引用。作用域外的变量应使用`scope`函数。例如，要引用我们在`mysql`模块中之前定义的`mysql::port`变量，请使用以下代码：

```
MySQL Port = <%= scope['::mysql::port'] %>
```

应用此模板会生成以下文件：

```
[root@hiera-test ~]# puppet agent -t
...
Info: Caching catalog for hiera-test.example.com
Notice: /Stage[main]/Erb/File[/tmp/template-test]/ensure: defined content as '{md5}96edacaf9747093f73084252c7ca7e67'
Notice: Finished catalog run in 0.41 seconds [root@hiera-test ~]# cat /tmp/template-test
MySQL Port = 3306

```

## 另见

+   第九章中的*创建自定义事实*示例，*外部工具与 Puppet 生态系统*

# 导入动态信息

尽管有些系统管理员喜欢通过堆积一堆旧打印机将自己与办公室的其他部分隔离开来，但我们都需要不时地与其他部门交换信息。例如，你可能希望将来自外部来源的数据插入到 Puppet 清单中。`generate`函数非常适合这种情况。函数在编译目录的机器上执行（对于集中式部署是主节点）；像这里示例的代码只适用于无主配置。

## 准备中

按照以下步骤准备运行示例：

1.  创建脚本`/usr/local/bin/message.rb`，其内容如下：

    ```
    #!/usr/bin/env ruby
    puts "This runs on the master if you are centralized"

    ```

1.  使脚本可执行：

    ```
    $ sudo chmod a+x /usr/local/bin/message.rb

    ```

## 如何做到这一点…

此示例调用我们之前创建的外部脚本并获取其输出：

1.  创建包含以下内容的`message.pp`清单：

    ```
    $message = generate('/usr/local/bin/message.rb')
    notify { $message: }

    ```

1.  运行 Puppet：

    ```
    $ puppet apply message.pp 
    ...
    Notice: /Stage[main]/Main/Notify[This runs on the master if you are centralized
    ]/message: defined 'message' as 'This runs on the master if you are centralized

    ```

## 它是如何工作的…

`generate`函数运行指定的脚本或程序并返回结果，在此例中是来自 Ruby 的一个愉快消息。

目前来看这并不是非常有用，但你明白其思路了。脚本可以做的任何事情，比如打印、获取或计算数据（例如数据库查询的结果），都可以通过`generate`引入到你的清单中。当然，你也可以运行标准的 UNIX 工具，例如`cat`和`grep`。

## 还有更多…

如果你需要向由`generate`调用的可执行文件传递参数，可以将它们作为额外的参数添加到函数调用中：

```
$message = generate('/bin/cat', '/etc/motd')

```

Puppet 会通过限制在调用中可以使用的字符来保护你免受恶意的 Shell 调用，例如，不允许使用 Shell 管道和重定向。最简单且最安全的方法是将所有逻辑放入脚本中，然后调用这个脚本。

## 另见

+   第九章中的*创建自定义事实*教程，*外部工具与 Puppet 生态系统*

+   第二章中的*配置 Hiera*教程，*Puppet 基础设施*

# 向 Shell 命令传递参数

如果你想将值插入命令行（例如由`exec`资源运行），它们通常需要加引号，尤其是当它们包含空格时。`shellquote`函数可以接收任意数量的参数，包括数组，并对每个参数加引号，然后将它们作为空格分隔的字符串返回，你可以将这个字符串传递给命令。

在这个示例中，我们想设置一个`exec`资源来重命名文件；但源文件名和目标文件名都包含空格，因此需要在命令行中正确加引号。

## 如何做到…

下面是使用`shellquote`函数的示例：

1.  使用以下命令创建`shellquote.pp`清单：

    ```
    $source = 'Hello Jerry'
    $target = 'Hello... Newman'
    $argstring = shellquote($source, $target)
    $command = "/bin/mv ${argstring}"
    notify { $command: }

    ```

1.  运行 Puppet：

    ```
    $ puppet apply shellquote.pp 
    ...
    Notice: /bin/mv "Hello Jerry" "Hello... Newman"
    Notice: /Stage[main]/Main/Notify[/bin/mv "Hello Jerry" "Hello... Newman"]/message: defined 'message' as '/bin/mv "Hello Jerry" "Hello... Newman"'

    ```

## 它是如何工作的…

首先，我们定义了`$source`和`$target`变量，这两个变量分别是我们在命令行中使用的文件名：

```
$source = 'Hello Jerry'
$target = 'Hello... Newman'

```

接着，我们调用`shellquote`将这些变量拼接成一个加引号的、空格分隔的字符串，如下所示：

```
$argstring = shellquote($source, $target)

```

然后，我们组合成最终的命令行：

```
$command = "/bin/mv ${argstring}"

```

结果将会是：

```
/bin/mv "Hello Jerry" "Hello... Newman"

```

现在，可以使用`exec`资源运行此命令行。如果我们不使用`shellquote`，会发生什么？

```
$source = 'Hello Jerry'
$target = 'Hello... Newman'
$command = "/bin/mv ${source} ${target}"
notify { $command: }

Notice: /bin/mv Hello Jerry Hello... Newman

```

这将无法正常工作，因为`mv`命令期望接收空格分隔的参数，因此它会将这条命令解释为将三个文件`Hello`、`Jerry`和`Hello...`移动到名为`Newman`的目录中，而这可能不是我们想要的结果。

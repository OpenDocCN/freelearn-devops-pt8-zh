# 第九章：外部工具与 Puppet 生态系统

|   | *“尽管如此，您可以随时偏离道路。道路的真正用途正是：到达个别选择的出发点。”* |   |
| --- | --- | --- |
|   | --*Robert Bringhurst，《排版风格元素》* |

在本章中，我们将覆盖以下内容：

+   创建自定义事实

+   添加外部事实

+   将事实设置为环境变量

+   使用 Puppet 资源命令生成清单

+   使用其他工具生成清单

+   使用外部节点分类器

+   创建您自己的资源类型

+   创建您自己的提供者

+   创建自定义函数

+   使用 rspec-puppet 测试您的 Puppet 清单

+   使用 librarian-puppet

+   使用 r10k

# 介绍

Puppet 本身是一个非常有用的工具，但如果将 Puppet 与其他工具和框架结合使用，您可以获得更大的收益。我们将讨论一些将数据输入 Puppet 的方法，包括自定义 Facter 事实、外部事实以及从现有配置自动生成 Puppet 清单的工具。

您还将学习如何通过创建自定义函数、资源类型和提供者来扩展 Puppet；如何使用外部节点分类器脚本将 Puppet 与您基础设施的其他部分集成；以及如何使用 rspec-puppet 测试您的代码。

# 创建自定义事实

虽然 Facter 内置的事实很有用，但实际上添加您自己的事实非常简单。例如，如果您有位于不同数据中心或托管服务商的机器，您可以为此添加自定义事实，这样 Puppet 就可以判断是否需要应用任何本地设置（例如，本地 DNS 服务器或网络路由）。

## 如何操作...

这是一个简单的自定义事实示例：

1.  创建目录 `modules/facts/lib/facter`，然后创建文件 `modules/facts/lib/facter/hello.rb`，并填入以下内容：

    ```
    Facter.add(:hello) do
      setcode do
        "Hello, world"
      end
    end
    ```

1.  修改您的 `site.pp` 文件如下：

    ```
    node 'cookbook' {
      notify { $::hello: }
    }
    ```

1.  运行 Puppet：

    ```
    [root@cookbook ~]# puppet agent -t
    Notice: /File[/var/lib/puppet/lib/facter/hello.rb]/ensure: defined content as '{md5}f66d5e290459388c5ffb3694dd22388b'
    Info: Loading facts
    Info: Caching catalog for cookbook.example.com
    Info: Applying configuration version '1416205745'
    Notice: Hello, world
    Notice: /Stage[main]/Main/Node[cookbook]/Notify[Hello, world]/message: defined 'message' as 'Hello, world'
    Notice: Finished catalog run in 0.53 seconds

    ```

## 它是如何工作的...

Facter 事实在与 facter 一起分发的 Ruby 文件中定义。Puppet 可以通过在模块的 `lib/facter` 子目录中创建文件，向 facter 添加附加事实。然后，这些文件会像我们之前看到的 `puppetlabs-stdlib` 模块一样被传输到客户端节点。要让命令行工具 facter 使用这些 `puppet` 事实，请像以下命令行所示一样在 facter 后添加 `-p` 选项：

```
[root@cookbook ~]# facter hello

[root@cookbook ~]# facter -p hello
Hello, world

```

### 提示

如果您使用的是旧版本的 Puppet（低于 3.0），您需要在 `puppet.conf` 文件中启用 `pluginsync`，如下所示命令行：

```
[main]
pluginsync = true

```

事实可以包含任何 Ruby 代码，并且在 `setcode do ... end` 块中评估的最后一个值将是该事实返回的值。例如，您可以创建一个更有用的事实，它返回当前登录系统的用户数量：

```
Facter.add(:users) do
  setcode do
    %x{/usr/bin/who |wc -l}.chomp
  end
end
```

要在清单中引用事实，只需像内置事实一样使用其名称：

```
notify { "${::users} users logged in": }
Notice:  2 users logged in

```

你可以向任何 Puppet 模块添加自定义事实。在创建多个模块会使用到的事实时，将它们放置在事实模块中可能更有意义。在大多数情况下，自定义事实与特定模块相关，应将其放置在该模块中。

## 还有更多……

持有事实定义的 Ruby 文件的名称并不重要。你可以随意命名这个文件；事实的名称来自于`Facter.add()`函数的调用。你也可以在单个 Ruby 文件中多次调用该函数，根据需要定义多个事实。例如，你可以`grep` `/proc/meminfo`文件，并根据内存信息返回多个事实，如以下代码片段中的`meminfo.rb`文件所示：

```
File.open('/proc/meminfo') do |f|
  f.each_line { |line|
  if (line[/^Active:/])
    Facter.add(:memory_active) do
      setcode do line.split(':')[1].to_i
      end
    end
  end
  if (line[/^Inactive:/])
    Facter.add(:memory_inactive) do
      setcode do line.split(':')[1].to_i
      end
    end
  end
  }
end
```

将此文件同步到节点后，`memory_active`和`memory_inactive`事实将如下面所示可用：

```
[root@cookbook ~]# facter -p |grep memory_
memory_active => 63780
memory_inactive => 58188

```

你可以扩展事实的使用，构建一个完全没有节点的 Puppet 配置；换句话说，Puppet 可以仅根据事实的结果决定应用哪些资源到机器上。Jordan Sissel 在[`www.semicomplete.com/blog/geekery/puppet-nodeless-configuration.html`](http://www.semicomplete.com/blog/geekery/puppet-nodeless-configuration.html)上写过关于这种方法的文章。

你可以在 Puppetlabs 网站上了解更多关于自定义事实的内容，包括如何确保操作系统特定的事实仅在相关系统上工作，以及如何加权事实，以确保它们按照特定顺序进行评估：

[`docs.puppetlabs.com/guides/custom_facts.html`](http://docs.puppetlabs.com/guides/custom_facts.html)

## 另请参见

+   在第三章的*导入动态信息*操作说明中，*编写更好的清单*。

+   在第二章的*配置 Hiera*操作说明中，*Puppet 基础设施*。

# 添加外部事实

*创建自定义事实*的操作说明描述了如何添加用 Ruby 编写的额外事实。你也可以选择从简单的文本文件或脚本中创建事实，使用外部事实替代。

外部事实存放在`/etc/facter/facts.d`目录中，并具有简单的`key=value`格式，如下所示：

```
message="Hello, world"
```

## 准备工作

这是你需要做的准备工作，以便为系统添加外部事实：

1.  你需要 Facter 版本 1.7 或更高版本才能使用外部事实，因此可以查找`facterversion`的值，或使用`facter -v`命令：

    ```
    [root@cookbook ~]# facter facterversion
    2.3.0
    [root@cookbook ~]# facter -v
    2.3.0

    ```

1.  你还需要创建外部事实目录，使用以下命令：

    ```
    [root@cookbook ~]# mkdir -p /etc/facter/facts.d

    ```

## 如何做到……

在这个示例中，我们将创建一个简单的外部事实，返回一条信息，如*创建自定义事实*操作说明所示：

1.  创建文件`/etc/facter/facts.d/local.txt`，并加入以下内容：

    ```
    model=ED-209
    ```

1.  运行以下命令：

    ```
    [root@cookbook ~]# facter model
    ED-209

    ```

    好了，这很简单！你当然可以像下面这样，向相同的文件或其他文件添加更多的事实：

    ```
    model=ED-209
    builder=OCP
    directives=4
    ```

    然而，如果你需要以某种方式计算事实，例如，计算已登录用户的数量怎么办？你可以创建可执行的事实来完成这个任务。

1.  创建文件`/etc/facter/facts.d/users.sh`，并包含以下内容：

    ```
    #!/bin/sh
    echo users=`who |wc -l`

    ```

1.  使用以下命令使这个文件变为可执行：

    ```
    [root@cookbook ~]# chmod a+x /etc/facter/facts.d/users.sh

    ```

1.  现在使用以下命令检查`users`值：

    ```
    [root@cookbook ~]# facter users
    2

    ```

## 它是如何工作的...

在这个例子中，我们将通过在节点上创建文件来创建一个外部事实。我们还将展示如何覆盖之前定义的事实。

1.  当前版本的 Facter 会查找`/etc/facter/facts.d`目录下的`.txt`、`.json`或`.yaml`类型的文件。如果 Facter 找到文本文件，它会解析文件中的`key=value`对，并将键作为一个新的事实：

    ```
    [root@cookbook ~]# facter model
    ED-209

    ```

1.  如果文件是 YAML 或 JSON 格式，Facter 会按照相应的格式解析文件中的`key=value`对。例如，对于 YAML 文件：

    ```
    ---
    registry: NCC-68814
    class: Andromeda
    shipname: USS Prokofiev
    ```

1.  结果输出如下：

    ```
    [root@cookbook ~]# facter registry class shipname
    class => Andromeda
    registry => NCC-68814
    shipname => USS Prokofiev

    ```

1.  对于可执行文件，Facter 会假设它们的输出是`key=value`对的列表。它将执行`facts.d`目录中的所有文件，并将它们的输出添加到内部事实哈希中。

    ### 提示

    在 Windows 中，批处理文件或 PowerShell 脚本可以像在 Linux 中使用可执行脚本一样使用。

1.  在`users`示例中，Facter 将执行`users.sh`脚本，生成以下输出：

    ```
    users=2

    ```

1.  然后，它会在这个输出中搜索`users`并返回匹配的值：

    ```
    [root@cookbook ~]# facter users
    2

    ```

1.  如果为您指定的键有多个匹配项，Facter 会根据权重属性决定返回哪个事实。在我使用的版本中，外部事实的权重为 10,000（在`facter/util/directory_loader.rb`中定义为`EXTERNAL_FACT_WEIGHT`）。这个高权重值是为了确保您定义的事实能够覆盖已提供的事实。例如：

    ```
    [root@cookbook ~]# facter architecture
    x86_64
    [root@cookbook ~]# echo "architecture=ppc64">>/etc/facter/facts.d/myfacts.txt
    [root@cookbook ~]# facter architecture
    ppc64

    ```

## 还有更多...

由于所有外部事实的权重为 10,000，它们在`/etc/facter/facts.d`目录中被解析的顺序决定了它们的优先级（最后被遇到的事实具有最高优先级）。要创建一个优先于其他事实的事实，您需要将它创建在按字母顺序最后的文件中：

```
[root@cookbook ~]# facter architecture
ppc64
[root@cookbook ~]# echo "architecture=r10000" >>/etc/facter/facts.d/z-architecture.txt
[root@cookbook ~]# facter architecture
r10000

```

### 调试外部事实

如果你在让 Facter 识别外部事实时遇到问题，可以以调试模式运行 Facter，看看发生了什么：

```
ubuntu@cookbook:~/puppet$ facter -d robin
Fact file /etc/facter/facts.d/myfacts.json was parsed but returned an empty data set

```

`X` JSON 文件被解析，但返回了一个空数据集错误，这意味着 Facter 没有在文件中找到任何`key=value`对，或者（在可执行事实的情况下）在其输出中没有找到。

### 注意

请注意，如果您有外部事实存在，Facter 每次查询时都会解析或运行`/etc/facter/facts.d`目录中的所有事实。如果其中一些脚本运行时间较长，这可能会显著拖慢任何使用 Facter 的程序（使用`--iming`开关运行 Facter 以排除故障）。除非某个特定的事实每次查询时都需要重新计算，否则考虑使用 cron 作业定期计算该事实并将结果写入 Facter 目录中的文本文件。

### 在 Puppet 中使用外部事实

你创建的任何外部事实都将同时对 Facter 和 Puppet 可用。要在 Puppet 清单中引用外部事实，只需像使用内建或自定义事实一样使用事实名称：

```
notify { "There are $::users people logged in right now.": }

```

除非你特别尝试覆盖已定义的事实，否则应避免使用预定义事实的名称。

## 另见

+   第三章中的*导入动态信息*食谱，*编写更好的清单*

+   第二章中的*配置 Hiera*食谱，*Puppet 基础设施*

+   本章中的*创建自定义事实*食谱

# 将事实设置为环境变量

另一种将信息传递给 Puppet 和 Facter 的便捷方式是使用环境变量。任何以`FACTER_`开头的环境变量都会被解释为一个事实。例如，使用以下命令询问 facter hello 的值：

```
[root@cookbook ~]# facter -p hello
Hello, world

```

现在通过环境变量覆盖值，并重新询问：

```
[root@cookbook ~]# FACTER_hello='Howdy!' facter -p hello
Howdy!

```

它同样适用于 Puppet，因此让我们通过一个示例来操作。

## 如何操作...

在这个示例中，我们将使用环境变量设置一个事实：

1.  保持 cookbook 的节点定义与我们上一个示例相同：

    ```
    node cookbook {
      notify {"$::hello": }
    }
    ```

1.  运行以下命令：

    ```
    [root@cookbook ~]# FACTER_hello="Hallo Welt" puppet agent -t
    Info: Caching catalog for cookbook.example.com
    Info: Applying configuration version '1416212026'
    Notice: Hallo Welt
    Notice: /Stage[main]/Main/Node[cookbook]/Notify[Hallo Welt]/message: defined 'message' as 'Hallo Welt'
    Notice: Finished catalog run in 0.27 seconds

    ```

# 使用 Puppet 资源命令生成清单

如果你有一台已经配置好的服务器，或者几乎配置完成，你可以将该配置捕获为 Puppet 清单。Puppet 资源命令会从系统的现有配置生成 Puppet 清单。例如，你可以使用`puppet resource`生成一个清单，创建系统中所有找到的用户。这对于拍摄一个正常工作的系统快照并将其配置快速导入 Puppet 非常有用。

## 如何操作...

下面是使用`puppet resource`从运行中的系统获取数据的一些示例：

1.  要为特定用户生成清单，运行以下命令：

    ```
    [root@cookbook ~]# puppet resource user thomas
    user { 'thomas':
     ensure           => 'present',
     comment          => 'thomas Admin User',
     gid              => '1001',
     groups           => ['bin', 'wheel'],
     home             => '/home/thomas',
     password         => '!!',
     password_max_age => '99999',
     password_min_age => '0',
     shell            => '/bin/bash',
     uid              => '1001',
    }

    ```

1.  对于某个特定的服务，运行以下命令：

    ```
    [root@cookbook ~]# puppet resource service sshd
    service { 'sshd':
     ensure => 'running',
     enable => 'true',
    }

    ```

1.  对于包，运行以下命令：

    ```
    [root@cookbook ~]# puppet resource package kernel
    package { 'kernel':
     ensure => '2.6.32-431.23.3.el6',
    }

    ```

## 还有更多...

你可以使用`puppet resource`来检查 Puppet 中可用的每种资源类型。在前面的示例中，我们为资源类型的特定实例生成了一个清单，但你也可以使用`puppet resource`来输出所有该资源类型的实例：

```
[root@cookbook ~]# puppet resource service
service { 'abrt-ccpp':
 ensure => 'running',
 enable => 'true',
}
service { 'abrt-oops':
 ensure => 'running',
 enable => 'true',
}
service { 'abrtd':
 ensure => 'running',
 enable => 'true',
}
service { 'acpid':
 ensure => 'running',
 enable => 'true',
}
service { 'atd':
 ensure => 'running',
 enable => 'true',
}
service { 'auditd':
 ensure => 'running',
 enable => 'true',
}

```

这将输出系统中每个服务的状态；这是因为每个服务都是可枚举的资源。当你尝试对一个不可枚举的资源执行相同命令时，会收到错误信息：

```
[root@cookbook ~]# puppet resource file
Error: Could not run: Listing all file instances is not supported.  Please specify a file or directory, e.g. puppet resource file /etc

```

让 Puppet 描述系统中的每个文件是行不通的；这最好交给像`tripwire`这样的审计工具（这是一个设计用来查找系统中每个文件变化的工具，[`www.tripwire.com`](http://www.tripwire.com)）。

# 使用其他工具生成清单

如果你想快速捕获一个正在运行的系统的完整配置作为 Puppet 清单，可以使用一些工具来帮助你。在这个例子中，我们将使用 Blueprint，它旨在检查一台机器并将其状态输出为 Puppet 代码。

## 准备工作

下面是你需要做的准备工作，以便在系统中使用 Blueprint。

运行以下命令以安装 Blueprint；我们将使用 `puppet resource` 来更改 `python-pip` 包的状态：

```
[root@cookbook ~]# puppet resource package python-pip ensure=installed
Notice: /Package[python-pip]/ensure: created
package { 'python-pip':
 ensure => '1.3.1-4.el6',
}
[root@cookbook ~]# pip install blueprint
Downloading/unpacking blueprint
 Downloading blueprint-3.4.2.tar.gz (59kB): 59kB downloaded
 Running setup.py egg_info for package blueprint
Installing collected packages: blueprint
 Running setup.py install for blueprint
 changing mode of build/scripts-2.6/blueprint from 644 to 755
...
Successfully installed blueprint
Cleaning up...

```

### 提示

如果你的食谱节点上尚未安装 Git，你可能需要先安装它。

## 如何操作...

这些步骤将向你展示如何运行 Blueprint：

1.  运行以下命令：

    ```
    [root@cookbook ~]# mkdir blueprint && cd blueprint
    [root@cookbook blueprint]# blueprint create -P blueprint_test
    # [blueprint] searching for APT packages to exclude
    # [blueprint] searching for Yum packages to exclude
    # [blueprint] caching excluded Yum packages
    # [blueprint] parsing blueprintignore(5) rules
    # [blueprint] searching for npm packages
    # [blueprint] searching for configuration files
    # [blueprint] searching for APT packages
    # [blueprint] searching for PEAR/PECL packages
    # [blueprint] searching for Python packages
    # [blueprint] searching for Ruby gems
    # [blueprint] searching for software built from source
    # [blueprint] searching for Yum packages
    # [blueprint] searching for service dependencies
    blueprint_test/manifests/init.pp

    ```

1.  阅读 `blueprint_test/manifests/init.pp` 文件以查看生成的代码：

    ```
    #
    # Automatically generated by blueprint(7).  Edit at your own risk.
    #
    class blueprint_test {
      Exec {
        path => '/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin',
      }
      Class['sources'] -> Class['files'] -> Class['packages']
        class files {
          file {
            '/etc':
              ensure => directory;
            '/etc/aliases.db':
    content => template('blueprint_test/etc/aliases.db'),
              ensure  => file,
    group   => root,
              mode    => 0644,
              owner   => root;
    '/etc/audit':
              ensure => directory;
    '/etc/audit/audit.rules':
              content => template('blueprint_test/etc/audit/audit.rules'),
              ensure  => file,
              group   => root,
              mode    => 0640,
              owner   => root;
            '/etc/blkid':
              ensure => directory;
    '/etc/cron.hourly':
              ensure => directory;
    '/etc/cron.hourly/run-backup':
              content => template('blueprint_test/etc/cron.hourly/run-backup'),
              ensure  => file,
              group   => root,
              mode    => 0755,
    owner   => root;
    '/etc/crypttab':
              content => template('blueprint_test/etc/crypttab'),
              ensure  => file,
              group   => root,
              mode    => 0644,
              owner   => root;
    ```

## 还有更多...

Blueprint 只是对当前系统进行快照；它不会做出智能的决策，并且 Blueprint 会捕获系统上的所有文件和所有软件包。它将生成一个可能比你实际需要的配置要大的配置。例如，当配置一个服务器时，你可能会指定要安装 Apache 包。Apache 包的依赖项将自动安装，而你不需要指定它们。当使用像 Blueprint 这样的工具生成配置时，你将捕获所有这些依赖项，并锁定当前安装在你系统上的版本。从我们生成的 Blueprint 代码来看，我们可以看到确实如此：

```
class yum {
  package {
    'GeoIP':
      ensure => '1.5.1-5.el6.x86_64';
    'PyXML':
      ensure => '0.8.4-19.el6.x86_64';
    'SDL':
      ensure => '1.2.14-3.el6.x86_64';
    'apr':
      ensure => '1.3.9-5.el6_2.x86_64';
    'apr-util':
      ensure => '1.3.9-3.el6_0.1.x86_64';
```

如果你自己创建这个清单，你可能会指定`ensure => installed`，而不是具体的版本号。

包安装默认版本的文件。Blueprint 并没有意识到这一点，会将所有文件添加到清单中，甚至那些没有变化的文件。默认情况下，Blueprint 会毫不分辨地捕获 `/etc` 中的所有文件作为文件资源。

Blueprint 和类似的工具通常有很小的应用场景，但它们可以帮助你熟悉 Puppet 语法，并给你一些关于如何指定自己清单的思路。然而，我并不建议盲目地使用这个工具来创建系统。

良好的配置管理没有捷径，那些希望通过复制和粘贴别人的代码（如公开模块）来节省时间和精力的人，可能会发现这既不能节省时间，也不能减少精力。

# 使用外部节点分类器

当 Puppet 在节点上运行时，它需要知道应该将哪些类应用于该节点。例如，如果它是一个 Web 服务器节点，可能需要包含一个 `apache` 类。将节点映射到类的正常方式是在 Puppet 清单本身中，例如，在你的 `site.pp` 文件中：

```
node 'web1' {
  include apache
}
```

或者，你可以使用**外部节点分类器（ENC）**来完成这项工作。ENC 是任何可执行程序，它可以接受完全限定域名（FQDN）作为第一个命令行参数（`$1`）。该脚本应返回一个类、参数以及一个可选的环境列表，用于应用到节点。输出应该是标准的 YAML 格式。在使用 ENC 时，你应该记住，通过标准`site.pp`清单应用的类将与 ENC 提供的类合并。

### 注意

ENC 返回的参数可作为顶级作用域变量提供给节点。

ENC 可以是一个简单的 Shell 脚本，或者是一个更复杂的程序或 API 的包装器，能够决定如何将节点映射到类。Puppet 企业版和 The Foreman 提供的 ENC（[`theforeman.org/`](http://theforeman.org/)）都是简单的脚本，它们连接到各自系统的 Web API。

在这个示例中，我们将构建最简单的 ENC，一个 Shell 脚本，简单地打印出包含的类列表。我们将从包含一个`enc`类开始，该类定义了`notify`，它将打印出一个顶级作用域变量`$enc`。

## 准备工作

我们将从创建我们的`enc`类开始，并与`enc`脚本一起使用：

1.  运行以下命令：

    ```
    t@mylaptop ~/puppet $ mkdir -p modules/enc/manifests

    ```

1.  创建文件`modules/enc/manifests/init.pp`，内容如下：

    ```
    class enc {
      notify {"We defined this from $enc": }
    }
    ```

## 如何操作...

这是如何构建一个简单的外部节点分类器的步骤。我们将在我们的 Puppet 主服务器上执行所有这些步骤。如果你是在无主模式下运行，请在节点上执行这些步骤：

1.  创建文件`/etc/puppet/cookbook.sh`，内容如下：

    ```
    #!/bin/bash
    cat <<EOF
    ---
    classes:
    enc:
    parameters:
      enc: $0
    EOF
    ```

1.  运行以下命令：

    ```
    root@puppet:/etc/puppet# chmod a+x cookbook.sh 

    ```

1.  按照以下方式修改你的`/etc/puppet/puppet.conf`文件：

    ```
    [main]
      node_terminus = exec
      external_nodes = /etc/puppet/cookbook.sh
    ```

1.  重启 Apache（重启主服务器）以使更改生效。

1.  确保你的`site.pp`文件中有以下默认节点的空定义：

    ```
    node default {}
    ```

1.  运行 Puppet：

    ```
    [root@cookbook ~]# puppet agent -t
    Info: Caching catalog for cookbook.example.com
    Info: Applying configuration version '1416376937'
    Notice: We defined this from /etc/puppet/cookbook.sh
    Notice: /Stage[main]/Enc/Notify[We defined this from /etc/puppet/cookbook.sh]/message: defined 'message' as 'We defined this from /etc/puppet/cookbook.sh'
    Notice: Finished catalog run in 0.17 seconds

    ```

## 它是如何工作的...

当在`puppet.conf`中设置 ENC 时，Puppet 将使用节点的 FQDN（技术上是`certname`变量）作为第一个命令行参数调用指定的程序。在我们的示例脚本中，这个参数被忽略，它只输出一个固定的类列表（实际上，只有一个类）。

很明显，这个脚本并不是特别有用；一个更复杂的脚本可能会检查数据库以查找类列表，或者在哈希表中查找节点，或者查阅外部文本文件或数据库（通常是一个组织的配置管理数据库，**CMDB**）。希望这个示例足以帮助你开始编写你自己的外部节点分类器。记住，你可以使用任何你喜欢的语言编写脚本。

## 还有更多内容...

ENC 可以提供一整类需要包含在节点中的类，格式如下（YAML）：

```
---
classes:
  CLASS1:
  CLASS2:
  CLASS3:
```

对于需要参数的类，你可以使用以下格式：

```
---
classes:
  mysql:
    package: percona-server-server-5.5
    socket:  /var/run/mysqld/mysqld.sock
    port:    3306
```

你也可以使用以下格式的 ENC 生成顶级作用域变量：

```
---
parameters:
  message: 'Anyone home MyFly?'
```

你以这种方式设置的变量将在清单中使用常规的顶级作用域变量语法，例如`$::message`，可以访问这些变量。

## 另见

+   参见 puppetlabs ENC 页面，了解更多关于编写和使用 ENC 的信息：[`docs.puppetlabs.com/guides/external_nodes.html`](http://docs.puppetlabs.com/guides/external_nodes.html)

# 创建你自己的资源类型

如你所知，Puppet 有许多有用的内置资源类型：包、文件、用户等。通常，你可以通过使用这些内置资源的组合，或者使用 `define`（你可以像使用资源一样使用它）来完成你需要做的所有事情（有关定义的更多信息，请参见第三章，*编写更好的清单*）。

在 Puppet 的早期，创建自定义资源类型更为常见，因为核心资源的列表比今天要短。在你考虑创建自己的资源类型之前，我建议你先在 Forge 中搜索替代解决方案。即使你只找到部分解决问题的项目，你也会通过扩展和帮助这个项目得到更好的服务，而不是试图创建自己的资源类型。然而，如果你确实需要创建自己的资源类型，Puppet 使这变得非常容易。原生类型是用 Ruby 编写的，因此你需要对 Ruby 有一定的基础了解才能创建自己的资源类型。

让我们回顾一下类型与提供者的区别。类型描述了一个资源及其可以拥有的参数（例如，`package` 类型）。提供者告诉 Puppet 如何为特定平台或情况实现资源类型（例如，`apt/dpkg` 提供者为类似 Debian 的系统实现了 `package` 类型）。

一个单一类型（`package`）可以有多个提供者（APT、YUM、Fink 等）。如果在声明资源时没有指定提供者，Puppet 将根据环境选择最合适的提供者。

在本节中，我们将使用 Ruby；如果你不熟悉 Ruby，可以访问[`www.ruby-doc.org/docs/Tutorial/`](http://www.ruby-doc.org/docs/Tutorial/)或[`www.codecademy.com/tracks/ruby/`](http://www.codecademy.com/tracks/ruby/)。

## 如何实现...

在本节中，我们将看到如何创建一个自定义类型，用于管理 Git 仓库，而在接下来的部分中，我们将编写一个提供者来实现这个类型。

创建文件 `modules/cookbook/lib/puppet/type/gitrepo.rb`，内容如下：

```
Puppet::Type.newtype(:gitrepo) do
  ensurable

  newparam(:source) do
    isnamevar
  end

  newparam(:path)
end
```

## 它是如何工作的...

自定义类型可以存在于任何模块中，位于 `lib/puppet/type` 子目录下，并且文件名应与类型名称相对应（在我们的例子中是 `modules/cookbook/lib/puppet/type/gitrepo.rb`）。

`gitrepo.rb` 的第一行告诉 Puppet 注册一个名为 `gitrepo` 的新类型：

```
Puppet::Type.newtype(:gitrepo) do
```

`ensurable` 这一行会自动为该类型添加 `ensure` 属性，类似于 Puppet 的内置资源：

```
ensurable
```

现在我们将为该类型添加一些参数。目前，我们只需要一个 `source` 参数来指定 Git 源 URL，以及一个 `path` 参数来告诉 Puppet 应该在哪个文件系统路径下创建该仓库：

```
newparam(:source) do
  isnamevar
end
```

`isnamevar` 声明告诉 Puppet `source` 参数是该类型的名称变量（namevar）。所以，当你声明此资源的实例时，不论你给定什么名字，它都会成为 `source` 的值，例如：

```
gitrepo { 'git://github.com/puppetlabs/puppet.git':
  path => '/home/ubuntu/dev/puppet',
}
```

最后，我们告诉 Puppet 该类型接受 `path` 参数：

```
newparam(:path)
```

## 还有更多...

在决定是否创建自定义类型时，你应该问自己一些问题，关于你试图描述的资源，例如：

+   资源是可枚举的吗？你能轻松地获取系统上所有该资源实例的列表吗？

+   资源是原子性的吗？你能确保系统上只存在一份该资源吗（当你想对该资源使用`ensure=>absent`时，这一点尤为重要）？

+   是否有其他资源描述此资源？在这种情况下，基于现有资源定义类型通常是更简单的解决方案。

### 文档

我们的示例故意保持简单，但当你开始为生产环境开发实际的自定义类型时，应该添加文档字符串来描述该类型及其参数的功能，例如：

```
Puppet::Type.newtype(:gitrepo) do
  @doc = "Manages Git repos"

  ensurable

  newparam(:source) do
    desc "Git source URL for the repo"
    isnamevar
  end

  newparam(:path) do
    desc "Path where the repo should be created"
  end
end
```

### 验证

你可以使用参数验证来生成有用的错误消息，当有人试图向资源传递无效值时。例如，你可以验证存储库创建目录是否实际存在：

```
newparam(:path) do
  validate do |value|
    basepath = File.dirname(value)
    unless File.directory?(basepath)
      raise ArgumentError , "The path %s doesn't exist" % basepath
    end
  end
end
```

你还可以指定该参数可以接受的值的列表：

```
newparam(:breakfast) do
  newvalues(:bacon, :eggs, :sausages)
end
```

# 创建你自己的提供者

在前一节中，我们创建了一个新的自定义类型 `gitrepo`，并告诉 Puppet 它需要两个参数，`source` 和 `path`。但是，到目前为止，我们还没有告诉 Puppet 如何实际检出该仓库；换句话说，就是如何创建此类型的特定实例。这时，提供者（provider）就派上用场了。

我们看到一个类型通常会有多个可能的提供者。在我们的示例中，实例化 Git 仓库只有一种合理的方法，所以我们只提供一个提供者：`git`。如果你要将此类型通用化——比如叫做仓库（repo）——可以很容易地想象，根据仓库的类型创建多个不同的提供者，例如 `git`、`svn`、`cvs` 等等。

## 如何操作...

我们将添加 `git` 提供者，并创建一个 `gitrepo` 资源实例来检查是否一切正常。为了让它工作，你需要安装 Git，但如果你正在使用在第二章中描述的基于 Git 的清单管理设置，*Puppet 基础设施*，我们可以安全地假设 Git 是可用的。

1.  创建文件 `modules/cookbook/lib/puppet/provider/gitrepo/git.rb`，内容如下：

    ```
    require 'fileutils'

    Puppet::Type.type(:gitrepo).provide(:git) do
      commands :git => "git"

      def create
        git "clone", resource[:source], resource[:path]
      end

      def exists?
        File.directory? resource[:path]
      end
    end
    ```

1.  按照以下方式修改你的 `site.pp` 文件：

    ```
    node 'cookbook' {
      gitrepo { 'https://github.com/puppetlabs/puppetlabs-git':
        ensure => present,
        path   => '/tmp/puppet',
      }
    }
    ```

1.  运行 Puppet：

    ```
    [root@cookbook ~]# puppet agent -t
    Notice: /File[/var/lib/puppet/lib/puppet/type/gitrepo.rb]/ensure: defined content as '{md5}6471793fe2b4372d40289ad4b614fe0b'
    Notice: /File[/var/lib/puppet/lib/puppet/provider/gitrepo]/ensure: created
    Notice: /File[/var/lib/puppet/lib/puppet/provider/gitrepo/git.rb]/ensure: defined content as '{md5}f860388234d3d0bdb3b3ec98bbf5115b'
    Info: Caching catalog for cookbook.example.com
    Info: Applying configuration version '1416378876'
    Notice: /Stage[main]/Main/Node[cookbook]/Gitrepo[https://github.com/puppetlabs/puppetlabs-git]/ensure: created
    Notice: Finished catalog run in 2.59 seconds

    ```

## 工作原理...

自定义提供程序可以存放在任何模块中，位于 `lib/puppet/provider/TYPE_NAME` 子目录下，并命名为提供程序的名称。（提供程序是实际在系统上运行的程序；在我们的示例中，程序是 Git，而提供程序位于 `modules/cookbook/lib/puppet/provider/gitrepo/git.rb`。请注意，模块的名称并不重要。）

在 `git.rb` 的初始 `require` 语句后，我们告诉 Puppet 使用以下语句注册 `gitrepo` 类型的新提供程序：

```
Puppet::Type.type(:gitrepo).provide(:git) do
```

当你在清单中声明 `gitrepo` 类型的实例时，Puppet 会首先检查该实例是否已经存在，通过调用提供程序的 `exists?` 方法。所以我们需要提供这个方法，并编写代码来检查 `gitrepo` 类型的实例是否已经存在：

```
def exists?
  File.directory? resource[:path]
end
```

这不是最复杂的实现；它仅仅在存在与实例的 `path` 参数匹配的目录时返回 `true`。更好的 `exists?` 实现可能会检查，例如，是否存在 `.git` 子目录，并且它包含有效的 Git 元数据。但现在这样已经足够。

如果 `exists?` 返回 `true`，那么 Puppet 将不再采取任何进一步的操作，因为指定的资源已经存在（至少 Puppet 是这么认为的）。如果返回 `false`，Puppet 假设资源尚不存在，并会尝试通过调用提供程序的 `create` 方法来创建它。

因此，我们提供一些代码给 `create` 方法，它调用 `git clone` 命令来创建仓库：

```
def create
  git "clone", resource[:source], resource[:path]
end
```

该方法可以访问实例的参数，我们需要了解从哪里检出代码仓库，并且在哪个目录下创建它。我们通过查看 `resource[:source]` 和 `resource[:path]` 来获得这些信息。

## 还有更多内容...

你可以看到，Puppet 中的自定义类型和提供程序非常强大。事实上，它们可以做任何事情——至少是 Ruby 可以做的任何事情。如果你正在使用复杂的 `define` 语句和 `exec` 资源来管理基础设施的某些部分，你可能想考虑用自定义类型来替代它们。然而，正如前面所说，在实现自己的之前，值得四处看看，看看是否有人已经做过类似的事情。

我们的示例非常简单，关于编写自定义类型还有很多内容需要学习。如果你打算将代码分发给别人使用，或者即使不打算分发，包含测试代码也是一个好主意。puppetlabs 有一个关于自定义类型与提供程序接口的有用页面：

[`docs.puppetlabs.com/guides/custom_types.html`](http://docs.puppetlabs.com/guides/custom_types.html)

关于实现提供程序：

[`docs.puppetlabs.com/guides/provider_development.html`](http://docs.puppetlabs.com/guides/provider_development.html)

以及一个完整的实例，展示了开发自定义类型和提供程序的过程，比书中展示的内容稍微复杂一些：

[`docs.puppetlabs.com/guides/complete_resource_example.html`](http://docs.puppetlabs.com/guides/complete_resource_example.html)

# 创建自定义函数

如果你已经阅读过第四章中的*使用 GnuPG 加密秘密*，*工作与文件和软件包*，那么你应该已经见过一个自定义函数的例子（在那个例子中，我们创建了一个`secret`函数，调用了 GnuPG）。现在我们更详细地了解`custom`函数，并构建一个示例。

## 如何实现...

如果你已经阅读过第六章中的*高效分发 cron 任务*，*管理资源与文件*，你可能还记得我们使用了`inline_template`函数根据节点的主机名设置随机时间来运行 cron 任务。在这个例子中，我们将这个想法转化为一个名为`random_minute`的自定义函数：

1.  创建文件`modules/cookbook/lib/puppet/parser/functions/random_minute.rb`，并包含以下内容：

    ```
    module Puppet::Parser::Functions
      newfunction(:random_minute, :type => :rvalue) do |args|
        lookupvar('hostname').sum % 60
      end
    end
    ```

1.  按如下方式修改你的`site.pp`文件：

    ```
    node 'cookbook' {
      cron { 'randomised cron job':
        command => '/bin/echo Hello, world >>/tmp/hello.txt',
        hour    => '*',
        minute  => random_minute(),
      }
    }
    ```

1.  运行 Puppet：

    ```
    [root@cookbook ~]# puppet agent -t
    Info: Retrieving pluginfacts
    Info: Retrieving plugin
    Notice: /File[/var/lib/puppet/lib/puppet/parser/functions/random_minute.rb]/ensure: defined content as '{md5}e6ff40165e74677e5837027bb5610744'
    Info: Loading facts
    Info: Caching catalog for cookbook.example.com
    Info: Applying configuration version '1416379652'
    Notice: /Stage[main]/Main/Node[cookbook]/Cron[custom fuction example job]/ensure: created
    Notice: Finished catalog run in 0.41 seconds

    ```

1.  使用以下命令检查`crontab`：

    ```
    [root@cookbook ~]# crontab -l
    # HEADER: This file was autogenerated at Wed Nov 19 01:48:11 -0500 2014 by puppet.
    # HEADER: While it can still be managed manually, it is definitely not recommended.
    # HEADER: Note particularly that the comments starting with 'Puppet Name' should
    # HEADER: not be deleted, as doing so could cause duplicate cron jobs.
    # Puppet Name: run-backup
    0 15 * * * /usr/local/bin/backup
    # Puppet Name: custom fuction example job
    15 * * * * /bin/echo Hallo, welt >>/tmp/hallo.txt

    ```

## 它是如何工作的...

自定义函数可以存在于任何模块中，存放在`lib/puppet/parser/functions`子目录中的一个文件里，文件名与函数名相同（在我们的例子中是`random_minute.rb`）。

函数代码放在一个`module ... end`块中，如下所示：

```
module Puppet::Parser::Functions
  ...
end
```

然后我们调用`newfunction`来声明我们的新函数，传递名称（`:random_minute`）和函数类型（`:rvalue`）：

```
newfunction(:random_minute, :type => :rvalue) do |args|
```

`:rvalue`部分仅表示此函数返回一个值。

最后，函数代码本身如下：

```
    lookupvar('hostname').sum % 60
```

`lookupvar`函数让你通过名称访问事实和变量；在这个例子中，使用`hostname`来获取我们运行的节点名称。我们使用 Ruby 的`sum`方法获取字符串中字符的数值总和，然后进行整数除法取模 60，以确保结果在`0..59`的范围内。

## 还有更多...

当然，你可以通过自定义函数做更多的事情。实际上，任何你能在 Ruby 中做的事情，都可以在自定义函数中做。你还可以访问在 Puppet 清单中函数被调用时作用域内的所有事实和变量，通过调用`lookupvar`就像示例中展示的那样。你还可以操作参数，例如创建一个通用的哈希函数，它接受两个参数：哈希表的大小和可选的哈希内容。创建`modules/cookbook/lib/puppet/parser/functions/hashtable.rb`，并包含以下内容：

```
module Puppet::Parser::Functions
  newfunction(:hashtable, :type => :rvalue) do |args|
    if args.length == 2
      hashtable=lookupvar(args[1]).sum
    else
      hashtable=lookupvar('hostname').sum
    end

    if args.length > 0
      size = args[0].to_i
    else
      size = 60
    end
    unless size == 0
      hashtable % size
    else
      0
    end
  end
end
```

现在我们将为`hashtable`函数创建一个测试，并按如下方式修改`site.pp`：

```
node cookbook {
  $hours = hashtable(24)
  $minutes = hashtable()
  $days = hashtable(30)
  $days_fqdn = hashtable(30,'fqdn')
  $days_ipaddress = hashtable(30,'ipaddress')
  notify {"\n hours=${hours}\n minutes=${minutes}\n days=${days}\n days_fqdn=${days_fqdn}\n days_ipaddress=${days_ipaddress}\n":}
}
```

现在，运行 Puppet 并观察返回的值：

```
Notice:  hours=15
 minutes=15
 days=15
 days_fqdn=4
 days_ipaddress=2

```

我们的简单定义在我们增加了参数功能后迅速增长。与所有编程一样，在处理参数时要特别小心，确保没有错误情况。在之前的代码中，我们专门检查了`size`变量为 0 的情况，以避免除零错误。

要了解更多关于自定义函数的内容，请参阅 puppetlabs 网站：

[`docs.puppetlabs.com/guides/custom_functions.html`](http://docs.puppetlabs.com/guides/custom_functions.html)

# 使用 rspec-puppet 测试你的 Puppet 清单

如果我们能验证我们的 Puppet 清单是否满足某些预期，而不必运行 Puppet，那将是极好的。`rspec-puppet`是一个很棒的工具，可以实现这一目标。基于 RSpec，一个 Ruby 程序的测试框架，`rspec-puppet`让你为 Puppet 清单编写测试用例，特别有助于捕捉回归问题（修复另一个 bug 时引入的 bug）和重构问题（重组代码时引入的 bug）。

## 准备工作

下面是安装`rspec-puppet`时你需要做的步骤：

运行以下命令：

```
t@mylaptop~ $ sudo puppet resource package rspec-puppet ensure=installed provider=gem
Notice: /Package[rspec-puppet]/ensure: created
package { 'rspec-puppet':
 ensure => ['1.0.1'],
}
t@mylaptop ~ $ sudo puppet resource package puppetlabs_spec_helper ensure=installed provider=gem
Notice: /Package[puppetlabs_spec_helper]/ensure: created
package { 'puppetlabs_spec_helper':
 ensure => ['0.8.2'],
}

```

## 如何操作...

让我们创建一个示例类`thing`，并为其编写一些测试。

1.  定义`thing`类：

    ```
    class thing {
      service {'thing':
        ensure  => 'running',
        enable  => true,
        require => Package['thing'],
      }
      package {'thing':
        ensure => 'installed'
      }
      file {'/etc/thing.conf':
        content => 'fubar\n',
        mode    => 0644,
        require => Package['thing'],
        notify  => Service['thing'],
      }
    }
    ```

1.  运行以下命令：

    ```
    t@mylaptop ~/puppet]$cd modules/thing
    t@mylaptop~/puppet/modules/thing $ rspec-puppet-init
     + spec/
     + spec/classes/
     + spec/defines/
     + spec/functions/
     + spec/hosts/
     + spec/fixtures/
     + spec/fixtures/manifests/
     + spec/fixtures/modules/
     + spec/fixtures/modules/heartbeat/
     + spec/fixtures/manifests/site.pp
     + spec/fixtures/modules/heartbeat/manifests
     + spec/fixtures/modules/heartbeat/templates
     + spec/spec_helper.rb
     + Rakefile

    ```

1.  创建文件`spec/classes/thing_spec.rb`，并写入以下内容：

    ```
    require 'spec_helper'

    describe 'thing' do
      it { should create_class('thing') }
      it { should contain_package('thing') }
      it { should contain_service('thing').with(
        'ensure' => 'running'
      ) }
      it { should contain_file('/etc/things.conf') }
    end
    ```

1.  运行以下命令：

    ```
    t@mylaptop ~/.puppet/modules/thing $ rspec
    ...F

    Failures:

     1) thing should contain File[/etc/things.conf]
     Failure/Error: it { should contain_file('/etc/things.conf') }
     expected that the catalogue would contain File[/etc/things.conf]
     # ./spec/classes/thing_spec.rb:9:in `block (2 levels) in <top (required)>'

    Finished in 1.66 seconds
    4 examples, 1 failure

    Failed examples:

    rspec ./spec/classes/thing_spec.rb:9 # thing should contain File[/etc/things.conf]

    ```

## 它是如何工作的...

`rspec-puppet-init`命令为你创建一个目录框架，以便你将你的测试程序（specs）放入其中。现在，我们只关心`spec/classes`目录。这里是你将放置每个类的测试文件的地方，每个文件以类名命名，例如，`thing_spec.rb`。

`spec`代码本身以以下语句开始，设置了运行测试的 RSpec 环境：

```
require 'spec_helper'
```

然后，跟着一个`describe`块：

```
describe 'thing' do
  ..
end
```

`describe`标识了我们要测试的类（`thing`），并将关于该类的断言列表放入一个`do .. end`块中。

断言是我们对`thing`类的预期。例如，第一个断言如下：

```
  it { should create_class('thing') }
```

`create_class`断言用于确保命名的类确实被创建。接下来的行：

```
  it { should contain_package('thing') }
```

`contain_package`断言意味着它字面上的意思：该类应该包含一个名为`thing`的包资源。

接下来，我们测试`thing`服务的存在：

```
it { should contain_service('thing').with(
  'ensure' => 'running'
) }
```

上述代码实际上包含了两个断言。首先，类包含一个`thing`服务：

```
contain_service('thing')
```

第二，服务具有一个`ensure`属性，值为`running`：

```
with(
  'ensure' => 'running'
)
```

你可以使用`with`方法指定任何你想要的属性和值，作为逗号分隔的列表。例如，以下代码断言了一个`file`资源的多个属性：

```
it { should contain_file('/tmp/hello.txt').with(
  'content' => "Hello, world\n",
  'owner'   => 'ubuntu',
  'group'   => 'ubuntu',
  'mode'    => '0644'
) }
```

在我们的`thing`示例中，我们只需要测试文件`thing.conf`是否存在，使用以下代码：

```
it { should contain_file('/etc/thing.conf') }
```

当你运行`rake spec`命令时，`rspec-puppet`将编译相关的 Puppet 类，运行它找到的所有测试，并显示结果：

```
...F
Failures:
 1) thing should contain File[/etc/things.conf]
 Failure/Error: it { should contain_file('/etc/things.conf') }
 expected that the catalogue would contain File[/etc/things.conf]
 # ./spec/classes/thing_spec.rb:9:in `block (2 levels) in <top (required)>'
Finished in 1.66 seconds
4 examples, 1 failure

```

如你所见，我们在测试中定义的文件是`/etc/things.conf`，但清单中的文件是`/etc/thing.conf`，因此测试失败。编辑`thing_spec.rb`，将`/etc/things.conf`改为`/etc/thing.conf`：

```
  it { should contain_file('/etc/thing.conf') }
```

现在再次运行 rspec：

```
t@mylaptop ~/.puppet/modules/thing $ rspec
....
Finished in 1.6 seconds
4 examples, 0 failures

```

## 还有更多...

使用 rspec，你可以验证许多条件。任何资源类型都可以通过 `contain_<资源类型>`(标题) 进行验证。除了验证你的类是否能正确应用外，你还可以通过在 spec 目录内使用相应的子目录（类、定义或函数）来测试函数和定义。

你可以在 [`rspec-puppet.com/`](http://rspec-puppet.com/) 上找到更多关于 `rspec-puppet` 的信息，包括可用断言的完整文档和教程。

当你想开始测试你的代码如何应用于节点时，你需要使用另一个工具——Beaker。Beaker 与各种虚拟化平台配合，创建临时虚拟机，并将 Puppet 代码应用到这些虚拟机上。然后，结果将用于 Puppet 代码的验收测试。这种一边开发一边测试的方法被称为 **测试驱动开发**（**TDD**）。有关 Beaker 的更多信息，请访问 GitHub 上的 [`github.com/puppetlabs/beaker`](https://github.com/puppetlabs/beaker)。

## 另见

+   第一章中的 *使用 puppet-lint 检查你的清单* 示例，*Puppet 语言与风格*

# 使用 librarian-puppet

当你开始在 Puppet 基础设施中包含来自 Forge 的模块时，跟踪你安装的版本并确保所有测试区域的一致性可能会变得有些棘手。幸运的是，我们将在接下来的两节中讨论的工具可以为你的系统带来秩序。我们首先将介绍 librarian-puppet，它使用一个名为 Puppetfile 的特殊配置文件来指定各种模块的源位置。

## 准备工作

我们将安装 librarian-puppet 来完成这个示例。

使用 Puppet 在 Puppet 主机上安装 `librarian-puppet`，当然是通过 Puppet 安装：

```
root@puppet:~# puppet resource package librarian-puppet ensure=installed provider=gem
Notice: /Package[librarian-puppet]/ensure: created
package { 'librarian-puppet':
 ensure => ['2.0.0'],
}

```

### 提示

如果你在一个无主机环境中工作，请在你将管理代码的机器上安装 `librarian-puppet`。如果 Ruby 开发包在主机上不可用，你的 gem 安装可能会失败；安装 `ruby-dev` 包可以解决这个问题（使用 Puppet 来安装）。

## 如何操作...

在这个示例中，我们将使用 librarian-puppet 下载并安装一个模块：

1.  为自己创建一个工作目录；librarian-puppet 默认会覆盖你的模块目录，因此我们暂时将在临时位置工作：

    ```
    root@puppet:~# mkdir librarian
    root@puppet:~# cd librarian

    ```

1.  创建一个新的 Puppetfile，内容如下：

    ```
    #!/usr/bin/env ruby
    #^syntax detection

    forge "https://forgeapi.puppetlabs.com"

    # A module from the Puppet Forge
    mod 'puppetlabs-stdlib'
    ```

    ### 注意

    另外，你可以使用 `librarian-puppet init` 来创建一个示例 Puppetfile，并编辑它以匹配我们的示例：

    ```
    root@puppet:~/librarian# librarian-puppet init
     create  Puppetfile

    ```

1.  现在，运行 librarian-puppet 来下载并安装 `puppetlabs-stdlib` 模块到 `modules` 目录：

    ```
    root@puppet:~/librarian# librarian-puppet install
    root@puppet:~/librarian # ls
    modules  Puppetfile  Puppetfile.lock
    root@puppet:~/librarian # ls modules
    stdlib

    ```

## 它是如何工作的...

`Puppetfile` 的第一行让 `Puppetfile` 看起来像一个 Ruby 源代码文件。这些完全是可选的，但它强迫编辑器将该文件当作 Ruby 文件来处理（事实上它就是 Ruby 文件）：

```
#!/usr/bin/env ruby
```

接下来我们定义 Puppet Forge 的位置；如果你有本地镜像，也可以在这里指定一个内部 Forge：

```
forge "https://forgeapi.puppetlabs.com"
```

现在，我们添加了一行以包括 `puppetlabs-stdlib` 模块：

```
mod 'puppetlabs-stdlib'
```

有了 `Puppetfile` 后，我们运行了 `librarian-puppet`，它从 Forge 行中提供的 URL 下载了模块。当模块下载时，`librarian-puppet` 创建了一个 `Puppetfile.lock` 文件，其中包含作为源使用的位置和下载模块的版本号：

```
FORGE
  remote: https://forgeapi.puppetlabs.com
  specs:
    puppetlabs-stdlib (4.4.0)

DEPENDENCIES
  puppetlabs-stdlib (>= 0)
```

## 还有更多...

`Puppetfile` 允许你从除 Forge 以外的来源拉取模块。你可以使用本地 Git URL，甚至是 GitHub URL，来下载那些不在 Forge 上的模块。关于 librarian-puppet 的更多信息可以在 GitHub 网站上找到：[`github.com/rodjek/librarian-puppet`](https://github.com/rodjek/librarian-puppet)。

请注意，librarian-puppet 会创建模块目录并默认删除你放入其中的任何模块。大多数使用 librarian-puppet 的安装选择将本地模块放在 `/local` 子目录中（`/dist` 或 `/companyname` 也常用）。

在下一节中，我们将讨论 r10k，它比 librarian 更进一步，管理整个环境目录。

# 使用 r10k

`Puppetfile` 是一种非常好的格式，用于描述你希望在环境中包含哪些模块。在 `Puppetfile` 的基础上，另一个工具是 **r10k**。r10k 是一个完整的环境管理工具。你可以使用 r10k 将本地 Git 仓库克隆到你的 `environmentpath` 中，然后将 `Puppetfile` 中指定的模块放入该目录。这个本地 Git 仓库被称为主仓库；r10k 期望在此找到你的 `Puppetfile`。r10k 还理解 Puppet 环境，并会将 Git 分支克隆到 `environmentpath` 的子目录中，从而简化多个环境的部署。r10k 特别有用的一点是它使用本地缓存目录来加速部署。通过配置文件 `r10k.yaml`，你可以指定缓存的存储位置以及主仓库的位置。

## 准备工作

我们将在控制机器（通常是主机）上安装 r10k。这将是我们控制所有已下载和已安装模块的地方。

1.  在你的 Puppet 主机上安装 r10k，或者在你希望管理 `environmentpath` 目录的任何机器上安装：

    ```
    root@puppet:~# puppet resource package r10k ensure=installed provider=gem
    Notice: /Package[r10k]/ensure: created
    package { 'r10k':
     ensure => ['1.3.5'],
    }

    ```

1.  创建一个新的 Git 仓库副本（可选，建议在你的 Git 服务器上进行）：

    ```
    [git@git repos]$ git clone --bare puppet.git puppet-r10k.git
    Initialized empty Git repository in /home/git/repos/puppet-r10k.git/

    ```

1.  在你的本地机器上检出新的 Git 仓库，并将现有的模块目录移动到新的位置。在本例中，我们使用 `/local`：

    ```
    t@mylaptop ~ $ git clone git@git.example.com:repos/puppet-r10k.git
    Cloning into 'puppet-r10k'...
    remote: Counting objects: 2660, done.
    remote: Compressing objects: 100% (2136/2136), done.
    remote: Total 2660 (delta 913), reused 1049 (delta 238)
    Receiving objects: 100% (2660/2660), 738.20 KiB | 0 bytes/s, done.
    Resolving deltas: 100% (913/913), done.
    Checking connectivity... done.
    t@mylaptop ~ $ cd puppet-r10k/
    t@mylaptop ~/puppet-r10k $ git checkout production
    Branch production set up to track remote branch production from origin.
    Switched to a new branch 'production'
    t@mylaptop ~/puppet-r10k $ git mv modules local
    t@mylaptop ~/puppet-r10k $ git commit -m "moving modules in preparation for r10k"
    [master c96d0dc] moving modules in preparation for r10k
     9 files changed, 0 insertions(+), 0 deletions(-)
     rename {modules => local}/base (100%)
     rename {modules => local}/puppet/files/papply.sh (100%)
     rename {modules => local}/puppet/files/pull-updates.sh (100%)
     rename {modules => local}/puppet/manifests/init.pp (100%)

    ```

## 如何操作...

我们将创建一个 `Puppetfile` 来控制 r10k 并在我们的主机上安装模块。

1.  在新的 Git 仓库中创建一个包含以下内容的 `Puppetfile`：

    ```
    forge "http://forge.puppetlabs.com"
    mod 'puppetlabs/puppetdb', '3.0.0'
    mod 'puppetlabs/stdlib', '3.2.0'
    mod 'puppetlabs/concat'
    mod 'puppetlabs/firewall'
    ```

1.  将 `Puppetfile` 添加到你的新仓库中：

    ```
    t@mylaptop ~/puppet-r10k $ git add Puppetfile
    t@mylaptop ~/puppet-r10k $ git commit -m "adding Puppetfile"
    [production d42481f] adding Puppetfile
     1 file changed, 7 insertions(+)
     create mode 100644 Puppetfile
    t@mylaptop ~/puppet-r10k $ git push
    Counting objects: 7, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (5/5), 589 bytes | 0 bytes/s, done.
    Total 5 (delta 2), reused 0 (delta 0)
    To git@git.example.com:repos/puppet-r10k.git
     cf8dfb9..d42481f  production -> production

    ```

1.  回到你的主机，创建 `/etc/r10k.yaml` 文件，内容如下：

    ```
    ---
    :cachedir: '/var/cache/r10k'
    :sources:
     :plops:
     remote: 'git@git.example.com:repos/puppet-r10k.git'
     basedir: '/etc/puppet/environments'

    ```

1.  运行 r10k 以填充`/etc/puppet/environments`目录（提示：首先备份你的`/etc/puppet/environments`目录）：

    ```
    root@puppet:~# r10k deploy environment -p

    ```

1.  确认你的`/etc/puppet/environments`目录下有一个`production`子目录。在该目录内，`/local`目录将存在，并且模块目录中会列出`Puppetfile`中所有的模块：

    ```
    root@puppet:/etc/puppet/environments# tree -L 2
    .
    ├── master
    │   ├── manifests
    │   ├── modules
    │   └── README
    └── production
     ├── environment.conf
     ├── local
     ├── manifests
     ├── modules
     ├── Puppetfile
     └── README

    ```

## 它是如何工作的...

我们首先创建了一个 Git 仓库的副本；这样做只是为了保留之前的工作，并不是必须的。需要记住的重要事项是，r10k 和 librarian-puppet 都假设它们控制着`/modules`子目录。我们需要将模块移开，并为模块创建一个新的位置。

在`r10k.yaml`文件中，我们指定了新仓库的位置。当我们运行 r10k 时，它首先将这个仓库下载到本地缓存中。一旦 Git 仓库被下载到本地，r10k 会遍历每个分支，并查找该分支中的`Puppetfile`。对于每个`branch/Puppetfile`组合，里面指定的模块首先下载到本地缓存目录（`cachedir`），然后下载到`r10k.yaml`中指定的`basedir`。

## 还有更多内容...

你可以使用`r10k`来自动化部署你的环境。我们用来运行`r10k`并填充我们的环境目录的命令，可以轻松地放入 Git 钩子中，自动更新你的环境。还有一个**marionette collective**（**mcollective**）插件（[`github.com/acidprime/r10k`](https://github.com/acidprime/r10k)），可以让`r10k`在任意一组服务器上运行。

使用这些工具中的任何一个将有助于保持你的网站一致，即使你没有利用 Forge 上可用的各种模块。

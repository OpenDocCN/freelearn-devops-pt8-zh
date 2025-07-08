# 编写你的第一个清单

配置管理已经成为 IT 世界中至关重要的一部分。使用敏捷方法进行更快的开发对 IT 运营产生了巨大影响，IT 运营需要跟上更快的系统部署速度。没有强大的管理基础设施，服务器操作几乎是不可行的。在众多可用工具中，Puppet 已经确立了自己作为最受欢迎和最广泛应用的解决方案之一。最初由 Luke Kanies 编写，这个工具现在在 Apache 2.0 许可证下分发，并由 Luke 的公司 Puppet Inc 维护。它拥有一个庞大且充满活力的社区，丰富的插件 API 和支持工具，出色的在线文档，以及基于 SSL 认证的强大安全模型。

和所有配置管理系统一样，Puppet 允许你维护一个基础设施定义的中央仓库，以及一个工具链，用于在受管系统上强制执行所需的状态。整个功能集非常令人印象深刻。本书将引导你通过一些步骤，快速掌握 Puppet 最重要的方面和原则。

在本章中，我们将涵盖以下主题：

+   开始使用

+   介绍资源、参数和属性

+   解释`puppet apply`命令的输出

+   使用变量

+   在清单中添加控制结构

+   控制执行顺序

+   实现资源交互

+   检查 Puppet 核心资源类型

# 开始使用

安装 Puppet 很简单。在大型 Linux 发行版上，你可以通过`apt-get`或`yum`直接安装 Puppet 软件包。

Puppet 的安装可以通过以下方式完成：

+   从默认的操作系统仓库安装

+   来自 Puppet Inc

前一种方式通常更简单。第二章，*Puppet 服务器和代理*，提供了简单的安装 Puppet Inc 软件包的说明。安装 Puppet 的跨平台方式是获取`puppet` Ruby gem。这种方式适合用于测试和管理单一系统，但不推荐用于生产环境。

安装 Puppet 后，你可以立即使用它。Puppet 由清单驱动，清单相当于脚本或程序，使用 Puppet 的**领域特定语言**（**DSL**）编写。我们从必备的`Hello, world!`清单开始：

```
# hello_world.pp
notify { 'Hello, world!':
} 
```

下载示例代码：

你可以从你在[`www.packtpub.com`](http://www.packtpub.com)的账户中下载你购买的所有 Packt Publishing 书籍的示例代码文件。如果你在其他地方购买了本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，并注册自己以便将文件直接通过电子邮件发送给你。

为了使清单生效，请使用以下命令（我们特意避免使用`execute`这个术语——清单不能被执行；更多细节将在本章中段介绍）：

```
root@puppetmaster:~# puppet apply hello_world.pp
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.45 seconds
Notice: Hello, world!
Notice: /Stage[main]/Main/Notify[Hello, world!]/message: defined 'message' as 'Hello, world!'
Notice: Applied catalog in 0.03 seconds 
```

Puppet Inc. 提供的软件包包含所有必需的软件组件，并安装到 `/opt/puppetlabs`。如果找不到 puppet 命令，你可以指定完整路径（`/opt/puppetlabs/bin/puppet`），或者刷新你的 shell 环境（`exec` bash，或注销并重新登录）。

在我们查看清单的结构和 `puppet apply` 命令的输出之前，先做一些有用的事情，作为示例。Puppet 自带其背景服务。假设你想在让它干扰系统之前先了解基本原理。你可以编写一个清单，让 Puppet 确保该服务当前没有运行，并且不会在系统启动时启动：

```
# puppet_service.pp
service { 'puppet':
  ensure => 'stopped',
  enable => false,
}
```

要控制系统进程、启动选项、软件安装等，Puppet 需要以 `root` 权限运行。这是调用该工具最常见的方式，因为 Puppet 通常会管理操作系统级别的设施。通过 `sudo` 或从 root shell 中，以 root 权限应用你的新清单，如下所示：

```
root@puppetmaster:~# puppet apply puppet_service.pp
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.61 seconds
Notice: /Stage[main]/Main/Service[puppet]/ensure: ensure changed 'running' to 'stopped'
Notice: Applied catalog in 0.15 seconds 
```

现在，Puppet 已为你禁用了其后台服务的自动启动。再次应用相同的清单没有效果，因为所需的步骤已经完成：

```
root@puppetmaster:~# puppet apply puppet_service.pp
Notice: Compiled catalog for puppetmaster.example.net in environment 
production in 0.62 seconds
Notice: Applied catalog in 0.07 seconds 
```

这反映了 Puppet 中的一个标准行为：Puppet 资源是**幂等的**，这意味着每个资源首先会将实际（系统）状态与期望（Puppet）状态进行比较，只有在存在差异（配置漂移）时才会启动操作。

你经常会看到 Puppet 输出这样的内容。它告诉你一切都按预期进行。因此，这是一个理想的结果，就像 git status 输出的“清洁”一样。

# 介绍资源、参数和属性

你在前一部分编写的每个清单都声明了一个相应的资源。资源是清单的基本构建块。每个资源都有一个类型（在此案例中分别为 `notify` 和 `service`）和一个名称或标题（`Hello, world!` 和 `puppet`）。每个资源对于清单来说是唯一的，可以通过其类型和名称的组合来引用，例如 `Service["puppet"]`。最后，一个资源还包括一个零个或多个属性的列表。属性是键值对，例如 `"enable => false"`。

属性名称不能随意选择。它们是 Puppet 资源类型的一部分。Puppet 区分两种不同的属性：参数和特性。每种资源类型支持一组特定的属性。参数描述 Puppet 应该如何处理资源类型，特性描述资源的特定设置。某些参数适用于所有资源类型（元参数），而一些名称则非常常见，例如 `ensure`。`service` 类型支持 `ensure` 特性，它表示受管理进程的状态。另一方面，它的 `enabled` 特性与系统引导配置相关（针对特定服务）。

我们已经使用了属性、特性和参数这几个术语，看似可以互换使用。不要被迷惑——它们之间有重要的区别。特性和参数是 Puppet 使用的两种不同的属性。

你已经看过两个特性的实际应用。现在让我们来看一个参数：

```
service { 'puppet':
  ensure   => 'stopped',
  enable   => false,
  provider => 'upstart',
}
```

`provider` 参数告诉 Puppet 需要与 `upstart` 子系统交互来控制其后台服务，而不是与 `systemd` 或 `init` 交互。如果不指定该参数，Puppet 会做出智能猜测。系统上有很多支持的工具来管理服务。稍后你将了解更多关于提供者及其自动选择的内容。

参数和属性的区别在于，参数仅指示 Puppet 应该如何管理资源，而不是期望的状态是什么。Puppet 只会对属性值进行操作。在这个例子中，它们是 `ensure => 'stopped'` 和 `enable => false`。对于每个这样的属性，Puppet 将执行以下任务：

+   测试资源是否已经与目标状态同步。

+   如果资源未同步，它将触发同步操作。

当由给定资源管理的系统实体（在此例中是 Puppet 的 `upstart` 服务配置）处于与属性值在清单中描述的状态一致时，属性被认为是同步的。在这个例子中，只有当 `puppet` 服务未运行时，`ensure` 属性才会是同步的。如果 `upstart` 没有配置为在系统启动时启动 Puppet，则 `enable` 属性是同步的。

作为一个帮助记忆的提示，记住，特性可以不同步，而参数则不可以。

Puppet 还允许你通过使用 `puppet resource` 命令读取现有的系统状态：

```
root@puppetmaster:~# puppet resource user root
user { 'root':
 ensure           => 'present',
 comment          => 'root',
 gid              => '0',
 home             => '/root',
 password         => '$6$17/7FtU/$TvYEDtFgGr0SaS7xOVloWXVTqQxxDUgH.
 eBKJ7bgHJ.hdoc03Xrvm2ru0HFKpu1QSpVW/7o.rLdk/9MZANEGt/',
 password_max_age => '99999',
 password_min_age => '0',
 shell            => '/bin/bash',
 uid              => '0',
} 
```

请注意，某些资源类型会返回只读属性（例如，文件资源类型会返回 `mtime` 和 `ctime`）。请参阅相关类型的文档。

# 解释 `puppet apply` 命令的输出。

正如你已经看到的，Puppet 输出的信息相当冗长。随着你对工具的熟悉，你会迅速学会识别关键信息。让我们首先看看信息性消息。再一次应用 `service.pp` 清单：

```
root@puppetmaster:~# puppet apply puppet_service.pp
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.48 seconds
Notice: Applied catalog in 0.05 seconds
```

Puppet 没有采取任何特定的行动。你只会得到两个时间戳：一个来自清单的编译阶段，另一个来自目录应用阶段。目录是编译清单的全面表示。Puppet 基于当前目录的内容来评估和同步资源。

现在，为了快速强制 Puppet 显示一些更有趣的输出，可以直接从 shell 传递一行清单。Ruby 或 Perl 的常用用户会认出这种调用语法：

```
# puppet apply -e'service { "puppet": enable => true, }'
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.62 seconds
Notice: /Stage[main]/Main/Service[puppet]/enable: enable changed 'false' to 'true'
Notice: Applied catalog in 0.12 seconds. 
```

我们更倾向于在作为命令行参数传递的清单中使用双引号，因为在 shell 中，清单应整体用单引号括起来。

你指示 Puppet 对 Puppet 服务执行另一次更改。输出反映了实际执行的更改。让我们分析一下这个日志消息：

+   行首的 `Notice:` 关键字表示日志级别。其他级别包括 `Warning`（警告）、`Error`（错误）和 `Debug`（调试）

+   更改的属性通过完整路径进行引用，从 `Stage[main]` 开始。Stages 超出了本书的范围，因此你在这里总是看到默认的 `main`。

+   下一个路径元素是 `Main`，它是另一个默认值。它表示声明该资源的类。你将在 第四章《*在类和定义类型中组合资源*》中学习类。

+   接下来是资源。你已经学到，`Service[puppet]` 是它的唯一引用

+   最后，`enable` 是相关属性的名称。当多个属性不同步时，通常每个属性都会有一行输出，表示该属性已同步。

+   日志行的其余部分表示 Puppet 认为合适应用的更改类型。表述方式取决于属性的性质。它可以像 `created`（已创建）一样简单，表示新添加到管理系统的资源，或者像 `changed false to true`（更改 false 为 true）这样简短的短语。

# 干运行你的清单

`puppet apply` 的另一个有用的命令行选项是 `--noop`。

它指示 Puppet 在资源不同步时不执行任何操作。

相反，你只会得到一条日志输出，表明在没有该选项时将会发生什么变化。这对于判断清单是否可能会破坏系统上的任何内容非常有用：

```
root@puppetmaster:~# puppet apply puppet_service.pp --noop
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.63 seconds
Notice: /Stage[main]/Main/Service[puppet]/enable: current_value true, should be false (noop)
Notice: Class[Main]: Would have triggered 'refresh' from 1 events
Notice: Stage[main]: Would have triggered 'refresh' from 1 events
Notice: Applied catalog in 0.06 seconds 
```

输出格式与之前相同，日志末尾带有一个标记为 (`noop`) 的标记，表示同步操作。这个日志可以视为应用清单时的预览，假如不使用 `--noop` 选项。

关于触发刷新时的额外通知将在后面描述，目前可以忽略。完成本章和第四章，*类和定义类型中的资源组合*后，你将更好地理解它们的意义。

# 使用变量

变量赋值的工作方式与大多数脚本语言相同。任何变量名都始终以 `$` 符号开头：

```
$download_server = 'img2.example.net'
$url = "https://${download_server}/pkg/example_source.tar.gz" 
```

同样，像大多数脚本语言一样，Puppet 在双引号中的字符串内执行变量值替换，但在单引号字符串中不进行任何插值。

变量对于使清单更简洁和易懂非常有用。它们帮助你实现保持源代码不冗余的总体目标。与命令式编程和脚本语言中的变量的一个重要区别是，Puppet 清单中的变量是不可变的。一旦一个值被赋值，它就不能被覆盖。

如果它是常量，为什么还叫做变量呢？人们不应将 Puppet 仅视为一个管理单一系统的工具。对于单一系统，Puppet 变量可能看起来像一个常量，但 Puppet 管理着多个操作系统不同的系统。在所有这些系统中，变量将会不同，而不是常量。

# 变量类型

从 Puppet 3.x 开始，只有四种变量类型：字符串、数组、哈希和布尔值。Puppet 4 引入了一个丰富的数据类型系统。新数据类型系统将在第七章，*Puppet 4 和 5 的新特性*的最后部分进行解释。这些基本的变量类型与其他语言中的相应类型工作方式类似。根据你的背景，你可能已经熟悉使用关联数组或字典作为 Puppet 的哈希类型的语义等效物：

```
$a_bool = true
$a_string = 'This is a string value'
$an_array = [ 'This', 'forms', 'an', 'array' ]
$a_hash = { 
  'subject'   => 'Hashes',
  'predicate' => 'are written',
  'object'    => 'like this',
  'note'      => 'not actual grammar!',
  'also note' => [ 'nesting is',
{ 'allowed'   => ' of course' } ], 
}
```

访问值同样简单。请注意，`hash`语法类似于 Ruby，而不是 Perl：

```
$x = $a_string
$y = $an_array[1]
$z = $a_hash['object'] 
```

字符串可以作为资源属性值使用，但值得注意的是，资源标题也可以是变量引用：

```
package { $apache_package:
  ensure => 'installed'
}
```

在这个上下文中，字符串值的含义直观清晰。但你也可以在此传递数组，用一条语句声明一组资源。以下清单管理三个包，确保它们都已安装：

```
$packages = [
  'apache2',
  'libapache2-mod-php5',
  'libapache2-mod-passenger',
 ]
package { $packages:
  ensure => 'installed'
}
```

你将在后面的章节中学习如何高效地使用哈希值。

数组不需要存储在变量中就可以使用，但在某些情况下，将其存储在变量中是一个好习惯。

# 数据类型

Puppet 4 中的数据类型系统允许你检查和验证一个变量是否属于特定的数据类型。这可以防止代码在（例如）预期为数组却接收到布尔值时表现不正确。

数据类型的完整功能将在 第七章，《Puppet 4 和 5 的新特性》中详细解释。在 Puppet 清单中，可以使用正则表达式控制结构来检查数据类型。

Puppet 有核心数据类型和抽象数据类型。核心数据类型是最常用的数据类型，例如字符串或整数，而抽象数据类型允许进行更复杂的类型验证，如可选类型或变体类型。

在处理数据类型之前，我们必须理解 Puppet 清单中的控制结构概念。

# 在清单中添加控制结构

到目前为止，你已经根据本章的指引编写了三个简单的清单。每个清单只包含一个资源，其中一个是通过命令行的 `-e` 选项传入的。当然，你不可能为每一种情况都编写不同的清单。相反，就像 Ruby 或 Perl 脚本会根据不同的条件分支出不同的代码路径一样，Puppet 代码也有一些结构，使其能够根据不同的情况变得灵活和可重用。

最常见的控制结构是 `if`/`else` 块。它与许多编程语言中的相应结构非常相似：

```
if 'mail_lda' in $needed_services {
  service { 'dovecot': enable => true }
} else {
  service { 'dovecot': enable => false }
}
```

Puppet 的 DSL 也有一个 `case` 语句，类似于其他语言中的 `case`：

```
case $role {
  ‘imap_server’: {
    package { ‘dovecot’: ensure => installed, }
    service { ‘dovecot’: ensure => running, }
  }
  /_webservers$/: {
    service { [‘apache’, ‘ssh’]: ensure => running, }
  }
  default: {
    service { ‘ssh’: ensure => running, }
  }
}
```

在第二个匹配器中，你可以看到如何使用正则表达式。

`case` 语句还可以根据变量的数据类型切换到特定的代码：

```
case $role {
  Array: {
    include $role[0]
  }
  String: {
    include $role
  }
  default: {
    notify { 'This nodes $role variable is neither an 
    Array nor a String':}
  }
}
```

`case` 语句的一种变体是选择器。它是一个表达式，而不是语句，可以像 C 类语言中的三元 `if`/`else` 操作符一样使用：

```
package { 'dovecot':
  ensure => $role ? {
    'imap_server' => 'installed',
    /desktop$/    => 'purged',
    default       => 'removed',
  },
}
```

类似于 `case` 语句，选择器也可以根据数据类型返回结果：

```
package { 'dovecot':
  ensure  => $role ? {
    Boolean => 'installed',
    String  => 'purged',
    default => 'removed',
  },
}
```

选择器的使用需要小心，因为在更复杂的清单中，这种语法会影响可读性。

# 控制执行顺序

到目前为止，你可能已经有了这样的印象：Puppet 的 DSL 是一种专业化的脚本语言。但实际上，这种印象是错误的。清单（manifest）不是脚本或程序。该语言是一种工具，用于通过一组资源（包括文件、软件包和定时任务等）来建模系统状态。

整个范式与脚本语言不同。Ruby 或 Perl 是命令式语言，基于一系列按严格顺序执行的语句。而 Puppet 的 DSL 是声明式的，这意味着清单声明了一组期望具有特定属性的资源。这些资源被放入一个目录中，Puppet 然后尝试在所有声明的资源中构建一条路径。编译器按顺序解析清单，但配置器以非常不同的方式应用资源。

换句话说，清单应该始终描述你期望的最终结果。达到这个结果所需的具体操作由 Puppet 决定。

为了更清楚地说明这一点，让我们看一个例子：

```
package { 'haproxy':
  ensure => 'installed',
}
file {'/etc/haproxy/haproxy.cfg':
  ensure => file,
  owner  => 'root',
  group  => 'root',
  mode   => '0644',
  source => 'puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
}
service { 'haproxy':
  ensure  => 'running',
}
```

通过这个清单，Puppet 将确保以下状态被实现：

1.  安装了`HAproxy`包。

1.  `haproxy.cfg`文件有特定的内容，这些内容已准备好并保存在`/etc/puppet/modules/`目录下的文件中。

1.  启动了`HAproxy`。

为了使这个工作生效，重要的是必须按照顺序执行必要的步骤：

+   配置文件通常无法在包之前安装，因为还没有目录来容纳它。

+   服务在安装之前无法启动。如果它在配置完成之前变为活动状态，它将使用来自包的默认设置。

强调这一点是因为前面的清单实际上并未包含 Puppet 指示严格排序的提示。没有显式的依赖关系，Puppet 可以自由地按其认为合适的顺序排列资源。

Puppet 的最新版本允许一种基于本地清单的排序方式，因此呈现的例子实际上会按原样工作。基于清单的排序可以在`puppet.conf`配置文件中按如下方式配置：

```
ordering = manifest
```

这个设置是 Puppet 4 的默认设置。仍然重要的是要了解排序原则，因为在更复杂的清单中，隐式顺序很难确定，而且正如你很快将学到的那样，还有其他因素会影响顺序。

# 声明依赖关系

将这种简单的清单整理成有序的方式最简单的方法是资源链式连接。其语法是在两个资源之间使用一个简单的 ASCII 箭头：

```
package { 'haproxy':
  ensure => 'installed',
}
->
file { '/etc/haproxy/haproxy.cfg':
  ensure => file,
  owner  => 'root',
  group  => 'root',
  mode   => '0644',
  source => 'puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
}
->
service {'haproxy':
  ensure => 'running',
}
```

只有当所有相关资源可以彼此紧挨着写时，这种方法才可行。换句话说，如果依赖关系的图形表示并不是一条直链，而是一个树形、星形或其他任何形状，这种语法就不够用了。

在内部，Puppet *将*构建一个有序的资源图，并在遍历该图时同步它们。

一种更通用且灵活的声明依赖关系的方法是通过特殊的元参数——这些参数可以与任何资源类型一起使用。有不同的元参数，其中大多数与排序无关（你在前面的例子中见过`provider`）。对于资源排序，Puppet 提供了`require`和`before`元参数。

两者都将一个或多个已声明资源的引用作为其值。如前所述，Puppet 引用有一个特殊的语法：

```
Type['title']
e.g.
Package['haproxy'] 
```

你只能构建指向在目录中声明的资源的引用。即使在被管理的系统中存在，你也不能构建和使用指向未被 Puppet 管理的东西的引用。

这是使用`require`元参数进行排序的`HAproxy`清单：

```
package { 'haproxy':
  ensure  => 'installed',
}
file {'/etc/haproxy/haproxy.cfg':
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  source  => 'puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
  require => Package['haproxy'],
}
 service {'haproxy':
  ensure  => 'running',
  require => File['/etc/haproxy/haproxy.cfg'],
}
```

以下清单在语义上是相同的，但它依赖的是 `before` 元参数，而不是 `require`：

```
package { 'haproxy':
  ensure => 'installed',
  before => File['/etc/haproxy/haproxy.cfg'],
}
file { '/etc/haproxy/haproxy.cfg':
  ensure => file,
  owner  => 'root',
  group  => 'root',
  mode   => '0644',
  source => 'puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
  before => Service['haproxy'],
}
service { 'haproxy':
  ensure => 'running',
}
```

当然，清单也可以混合使用这两种符号风格。这部分留给读者练习，没有专门的描述。

`require` 元参数通常能生成更易于理解的代码，因为它表达了被注解资源对另一个资源的依赖关系。另一方面，`before` 参数则暗示了一个引用资源对当前资源所形成的依赖关系。这可能会让人感到不直观，特别是对于经常使用软件包系统的人（这些系统通常实现了 `require` 风格的依赖声明）。

有时，决定是使用 `require` 还是 `before` 可能会很困难。在简单的情况下，大多数人更倾向于使用 `require`。在某些情况下，使用 `before` 更容易。想想那些有多个配置文件的服务。将有关配置文件和需求的信息保存在一个地方，可以减少在添加或移除附加配置文件时忘记同时修改服务所造成的错误。看一下以下示例代码：

```
file { '/etc/apache2/apache2.conf':
  ensure => file,
  before => Service['apache2'],
}
file { '/etc/apache2/httpd.conf':
  ensure => file,
  before => Service['apache2'],
}
service { 'apache2':
  ensure => running,
  enable => true,
}
```

在这个例子中，所有依赖关系都在文件资源声明中进行了声明。如果你改用 `require` 参数，那么在发生变化时，你将始终需要修改至少两个资源：

```
file { '/etc/apache2/apache2.conf':
  ensure => file,
}
file { '/etc/apache2/httpd.conf':
  ensure => file,
}
service { 'apache2':
  ensure => running,
  enable => true,
 require => [
    File['/etc/apache2/apache2.conf'],
    File['/etc/apache2/httpd.conf'],
  ],
}
```

每当你添加一个新文件并由 Puppet 管理时，你会记得更新服务资源声明吗？考虑另一个更简单的例子：

```
if $os_family == 'Debian' {
  file { '/etc/apt/preferences.d/example.net.prefs':
    content => '...',
    before  => Package['apache2'],
  }
}
package { 'apache2':
  ensure    => 'installed',
} 
```

`preferences.d` 目录中的文件仅对类似 Debian 的系统有意义；这就是为什么该软件包不能安全地 `require` 它的原因。如果清单应用到其他操作系统，例如 CentOS，`apt` 配置文件不会出现在目录中，感谢 `if` 条件语句。如果该软件包无论如何都将其作为依赖，最终的目录将不一致，Puppet 也无法应用它。在文件资源中指定 `before` 是安全的，并且语义上是等效的。

在像这种情况中，`before` 元参数是必不可少的，并且在其他情况下，它能使清单代码更加优雅和直接。熟悉 `before` 和 `require` 两者是很有益的。

# 错误传播

定义需求有另一个重要的目的。声明的资源上的引用，只有在依赖的资源成功完成时才会被验证为成功引用。如果所需资源未能成功同步，可以视为 Puppet DSL 代码中的一个停止点。

例如，如果 `source` 文件的 URL 被破坏，`file` 资源会失败：

```
file { '/etc/haproxy/haproxy.cfg':
  ensure => file,
  source => 'puppet:///modules/haproxy/etc/haproxy.cfg',
}  
```

这里缺少了一个路径段。Puppet 会报告文件资源未能同步：

```
root@puppetmaster:~# puppet apply typo.pp
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.62 seconds
Error: /Stage[main]/Main/File[/etc/haproxy/haproxy.cfg]: Could not evaluate: Could not retrieve information from environment production source(s) puppet:///modules/haproxy/etc/haproxy.cfg
Notice: /Stage[main]/Main/Service[haproxy]: Dependency File[/etc/haproxy/haproxy.cfg] has failures: true
Warning: /Stage[main]/Main/Service[haproxy]: Skipping because of failed dependencies
Notice: Applied catalog in 0.06 seconds 
```

在这个例子中，`Error` 行描述了由破损 URL 引起的错误。错误传播通过下面的 `Notice` 和 `Warning` 行表示。

Puppet 未能应用对配置文件的更改；它无法将当前状态与不存在的源进行比较。由于服务依赖于配置文件，Puppet 甚至不会尝试启动它。这是出于安全考虑：如果任何依赖关系无法进入定义的状态，Puppet 必须假定系统不适合应用该依赖资源。

这是另一个利用资源依赖的重要原因。记住，链式箭头和`before`元参数也暗示着错误传播。

# 避免循环依赖。

在了解资源之间另一种可能的相互关系之前，有一个问题你应该注意：依赖关系不能形成循环。让我们通过一个例子来可视化这个问题：

```
file { '/etc/haproxy':
  ensure  => 'directory',
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
}
file { '/etc/haproxy/haproxy.cfg':
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  source  => 'puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
}
service { 'haproxy':
  ensure  => 'running',
  require => File['/etc/haproxy/haproxy.cfg'],
  before  => File['/etc/haproxy'],
}
```

这个清单中的依赖循环有些隐藏（这很可能是你在使用 Puppet 时遇到的许多循环的情况）。

它由以下关系构成：

+   `File['/etc/haproxy/haproxy.cfg']`自动依赖父目录`File['/etc/haproxy']`。这是一个隐式的、内置的依赖关系。

+   父目录`File['/etc/haproxy']`由于其`before`元参数，要求`Service['haproxy']`。

+   `Service['haproxy']`服务依赖`File['/etc/haproxy/haproxy.cfg']`配置文件。

以下资源组合存在隐式依赖关系，等等：

+   如果声明了一个目录和该目录中的一个文件，Puppet 将首先创建目录，然后创建文件。

+   如果声明了用户及其主组，Puppet 将首先创建该组，然后创建用户。

+   如果声明了文件和所有者（用户），Puppet 将首先创建用户，然后创建文件。

诚然，前面的例子是人为设定的——在配置目录之前管理服务是没有意义的。尽管如此，即使是看似合理的清单设计，也可能导致循环依赖。这就是 Puppet 如何响应这种设计：

```
root@puppetmaster:~# puppet apply circle.pp
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.62 seconds
Error: Failed to apply catalog: Found 1 dependency cycle:
(File[/etc/haproxy/haproxy.cfg] => 
               Service[haproxy] => 
             File[/etc/haproxy] => File[/etc/haproxy/haproxy.cfg])
Try the '--graph' option and opening the resulting '.dot' file in OmniGraffle or GraphViz
```

输出帮助你定位有问题的关系。如果依赖循环非常大，涉及大量资源，那么文本渲染就很难分析。因此，Puppet 还提供了通过`--graph`选项获取依赖图的图形表示的机会。

如果你这样做，Puppet 将在其输出中包含新创建的`.dot`文件的完整路径。它的内容看起来类似于 Puppet 的输出：

```
digraph Resource_Cycles {
label = "Resource Cycles"
"File[/etc/haproxy/haproxy.cfg]" ->"Service[haproxy]" ->"File[/etc/haproxy]" ->"File[/etc/haproxy/haproxy.cfg]"
}  
```

这本身没有太大帮助，但它可以直接输入到像`dotty`这样的工具中，生成实际的图表。

总结一下，资源依赖有助于防止 Puppet 在意外或不可控的情况下对资源进行操作。它们还可以限制资源评估的顺序。

# 实现资源交互。

除了依赖关系外，资源还可以进入类似但不同的相互关系。记住我们之前跳过的输出部分。它们如下：

```
root@puppetmaster:~# puppet apply puppet_service.pp --noop
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.62 seconds
Notice: /Stage[main]/Main/Service[puppet]/ensure: current_value running, should be stopped (noop)
Notice: Class[Main]: Would have triggered 'refresh' from 1 events
Notice: Stage[main]: Would have triggered 'refresh' from 1 events
Notice: Applied catalog in 0.05 seconds 
```

Puppet 提到**刷新**会因为**事件**的原因而触发。每当 Puppet 需要同步操作时，资源会发出此类事件。如果没有显式的代码来接收并响应这些事件，它们将被丢弃。

设置此类事件接收器的机制是通过类比通用的发布/订阅队列来命名的；资源通过 `subscribe` 元参数配置为响应事件。没有 `publish` 关键字或参数，因为每个资源在技术上都是事件（消息）的发布者。相应的 `subscribe` 元参数的对立面叫做 `notify`，它明确地将生成的事件指向被引用的资源。

事件系统的一个常见实际应用是重新加载服务配置。当 `service` 资源接收事件（通常来自配置文件的更改）时，Puppet 会执行适当的操作以重启该服务。

如果你指示 Puppet 执行此操作，可能会由于重启操作导致短暂的服务中断。请注意，如果新配置导致错误，服务可能无法启动并保持离线状态。

以下代码示例展示了 `haproxy` 包、相应的 `haproxy` 配置文件和 `haproxy` 服务之间的关系：

```
file { '/etc/haproxy/haproxy.cfg':
  ensure  => file,
  owner   => ‘root’,
  group   => ‘root’
  mode    => ‘0644’
  source  => ‘puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
  require => Package['haproxy'],
}
service { 'haproxy':
  ensure    => 'running',
  subscribe => File['/etc/haproxy/haproxy.cfg'],
}
```

如果要使用 `notify` 元参数，它必须指定给发出事件的资源：

```
file { '/etc/haproxy/haproxy.cfg':
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  source  => 'puppet:///modules/haproxy/etc/haproxy/haproxy.cfg',
  require => Package['haproxy'],
  notify  => Service['haproxy'],
}
service { 'haproxy':
  ensure  => 'running',
}
```

这可能会让你想起 `before` 和 `require` 元参数，它们提供了对一对资源关系的对称表达方式。这并非巧合，这些元参数是紧密相关的：

+   订阅另一个资源的资源隐式地需要它。

+   通知另一个资源的资源在依赖图中会隐式地排在后一个资源之前。

换句话说，`subscribe` 与 `require` 相同，只是依赖资源接收来自同伴的事件。`notify` 和 `before` 也是如此。

链接语法也可以用于信号传递。为了在相邻资源之间建立信号关系，请使用带波浪符的 ASCII 箭头 `~>`，而不是 `->` 中的破折号：

```
file { '/etc/haproxy/haproxy.cfg': … }
~>
service { 'haproxy': … }
```

`service` 资源类型是两种在资源收到通知时支持刷新的重要类型之一（另一个将在下一节讨论）。还有其他类型，但它们没有这么普遍。

# 检查 Puppet 核心资源类型

为了完整了解清单的基本元素，我们来仔细看看你已经使用过的资源类型，以及一些你尚未遇到但属于 Puppet 基础安装的重要资源类型。

你可能已经对 `file` 类型有了很好的了解，它将确保文件和目录的存在及其权限。通过 `source` 参数从仓库（通常是 Puppet 模块）拉取文件也是一个常见的用例。

对于非常短的文件，将所需内容直接包含在清单中更加经济：

```
file { '/etc/modules':
  ensure  => file,
  content => "# Managed by Puppet!\n\ndrbd\n",
}
```

双引号允许扩展转义序列，例如`\n`。

另一个有用的功能是管理符号链接：

```
file { '/etc/apache2/sites-enabled/001-puppet-lore.org':
  ensure => 'link',
  target => '../sites-available/puppet-lore.org',
}
```

你应该知道，文件资源类型需要绝对路径和文件名。如果在标题中使用相对路径，Puppet 会报错：

```
file { '../demo.txt':
  ensure => file,
}
puppet apply file_error.pp
Notice: Compiled catalog for puppetmaster.demo.example42.com in environment production in 0.09 seconds
Error: Parameter path failed on File[../demo.txt]: File paths must be fully qualified, not '../demo.txt' at /root/file_error.pp:1
```

你已经知道的下一个类型是`package`，它的典型用法相当直观。确保包被安装或删除。你还没有看到的一个显著用例是使用基本的包管理器而不是`apt`或`yum/zypper`。如果某个包在仓库中不可用，这是非常有用的：

```
package { 'haproxy':
  ensure   => present,
  provider => 'dpkg',
  source   => '/opt/packages/haproxy-1.5.1_amd64.dpkg',
}
```

如果你努力设置一个简单的仓库，通常会提高效率，这样就可以使用主要的包管理器了。

最后但同样重要的是，有一个`service`类型，它的最重要的属性你已经了解。值得指出的是，在你不希望添加完整的`init`脚本或类似脚本的情况下，它可以作为一个简单的快捷方式。如果提供了足够的信息，`service`类型的`base`提供者将为你管理简单的后台进程：

```
service { 'count-logins':
  provider    => 'base',
  ensure      => 'running',
  enable      => true,
  binary      => '/usr/local/bin/cnt-logins',
  start       => '/usr/local/bin/cnt-logins –daemonize',
  has_status  => true,
  has_restart => true,
  subscribe   => File['/usr/local/bin/cnt-logins'],
}
```

Puppet 不仅会在某些原因导致脚本未运行时重启它，还会在引用的配置文件内容发生变化时重新启动它。

这仅在 Puppet 管理文件内容并且所有更改仅通过 Puppet 传播时有效。

如果 Puppet 更改了脚本文件的其他属性（例如文件模式），这也会导致进程重启。

让我们来看看你可能需要的其他类型。

# 用户和组类型

尤其是在缺乏中央注册表（如 LDAP）的情况下，能够管理每台机器上的用户账户是非常有用的。所有受支持平台都有提供者；然而，提供的属性有所不同。在 Linux 上，`useradd`提供者是最常见的。它允许管理`/etc/passwd`中的所有字段，如`uid`和`shell`，以及组成员关系：

```
group { 'proxy-admins':
  ensure     => present,
  gid        => 4002,
}
user { 'john':
  ensure     => present,
  uid        => 2014,
  home       => '/home/john',
  managehome => true, # <- adds -m to useradd
  gid        => 1000,
  shell      => '/bin/zsh',
  groups     => [ 'proxy-admins' ],
}
```

与所有资源一样，Puppet 不仅会确保用户和组存在，还会修复任何不同的属性，比如`home`目录。

即使用户依赖于组：（因为它不能在组存在之前被添加），也不需要在清单中显式表达。用户会自动要求所有必要的组，类似于文件自动要求其父目录。

Puppet 也可以愉快地管理你的 LDAP 用户账户。

前面提到过，根据操作系统的不同，提供了不同的属性。Linux（以及`useradd`提供者）支持设置密码，而在 HP-UX（使用`hp-ux`提供者）上，无法通过 Puppet 设置用户密码。

在这种情况下，Puppet 只会显示一个警告，指出用户资源类型正在使用不受支持的属性，并继续管理所有其他属性。换句话说，在 Puppet DSL 代码中使用不受支持的属性不会破坏 Puppet 的运行。

# `exec` 资源类型

在 Puppet 核心中有一种特殊的资源类型。记住我们之前提到的，Puppet 不是一个专业的脚本引擎，而是一个允许你用引人注目的 DSL 模型化系统部分状态的工具，并且能够根据定义的目标改变系统。这就是为什么你声明 `user` 和 `group`，而不是依次调用 `groupadd` 和 `useradd`。你之所以能这样做，是因为 Puppet 内置了对这些实体的管理支持。这非常有利，因为 Puppet 还知道在不同的平台上，使用的是不同的命令进行账户管理，并且某些系统上的参数可能会略有不同。

当然，Puppet 并不掌握任何受支持系统的所有细节。假设你希望管理一个 OpenAFS 文件服务器，Puppet 没有专门的资源类型来帮助你。理想的解决方案是利用 Puppet 的插件系统，编写你自己的类型和提供程序，这样你的清单就能直接反映 AFS 特定的配置。然而，这并不简单，而且在某些情况下，当你只需要 Puppet 在清单的少数地方调用一些特殊的命令时，这也不值得去做。

对于这种情况，Puppet 提供了 `exec` 资源类型，允许执行自定义命令来替代抽象的同步操作。

例如，它可以在没有合适的包时用来解压 tar 包：

```
exec { 'tar cjf /opt/packages/homebrewn-3.2.tar.bz2':
  cwd     => '/opt',
  path    => '/bin:/usr/bin',
  creates => '/opt/homebrewn-3.2',
}
```

`creates` 参数对于 Puppet 判断命令是否需要执行非常重要。一旦指定的路径存在，资源就视为已同步。对于那些不会创建明显文件或目录的命令，还有其他参数 `onlyif` 和 `unless`，它们允许 Puppet 查询同步状态：

```
exec { 'perl -MCPAN -e "install YAML"':
  path   => '/bin:/usr/bin',
  unless => 'cpan -l | grep -qP ^YAML\\b',
}
```

查询命令的退出代码决定了状态。在 `unless` 的情况下，查询失败时会执行 `exec` 命令。这就是 `exec` 类型保持幂等性的方法。Puppet 对大多数资源类型自动执行这一操作，但对于 `exec` 来说，这不可行，因为同步是如此任意地定义的。作为用户，定义每个资源的适当查询就成了你的责任。

最后，`exec` 类型的资源是使用 `notify` 和 `subscribe` 接收事件的第二个显著案例：

```
exec { 'apt-get update':
  path        => '/bin:/usr/bin',
  subscribe   => File['/etc/apt/sources.list.d/jenkins.list'],
  refreshonly => true,
}
```

你甚至可以以这种方式将多个 `exec` 资源链接在一起，使得每次调用都会触发下一个命令。然而，这种做法并不推荐，它会使 Puppet 降级为一个（相当不完善的）脚本引擎。尽可能避免使用 `exec` 资源，而应优先使用常规资源。一些不属于核心的资源类型可以作为插件从 Puppet Forge 获取。你将在第五章中了解更多关于这个主题的内容，*将类、配置文件和扩展结合成模块*。

由于 `exec` 资源可以用于执行几乎 *任何* 操作，它们有时被滥用来代替更合适的资源类型。这是 Puppet 清单中的典型反模式。最好将 `exec` 资源视为最后的手段或紧急出口，只有在所有其他选择都已用尽的情况下才使用。

理想情况下，你的 `exec` 资源类型应该构建为一次性命令。

所有 Puppet 安装都有类型文档内置在代码中，可以通过 `puppet describe` 命令在命令行打印出来：

**`puppet describe <type> [-s]`** 如果你不确定某个类型是否存在，可以通过 `puppet describe` 命令返回所有可用资源类型的完整列表：

`**puppet describe --list**`

让我们简要讨论一下另两个内置支持的类型。它们分别允许管理 cron 任务、挂载的分区和共享资源，这些都是服务器操作中常见的需求。

# cron 资源类型

一个 cron 任务主要由一个命令和指定运行该命令的时间和日期组成。Puppet 将命令和每个日期元素建模为一个资源的属性，资源类型为 `cron`：

```
cron { 'clean-files':
  ensure      => present,
  user        => 'root',
  command     => '/usr/local/bin/clean-files',
  minute      => '1',
  hour        => '3',
  weekday     => [ '2', '6' ],
  environment => 'MAILTO=felix@example.net',
}
```

`environment` 属性允许你为 `cron` 指定一个或多个变量绑定，以添加到任务中。

# 挂载资源类型

最后，Puppet 将为你管理所有可挂载的文件系统，包括它们的基本属性，如源设备和挂载点、挂载选项及当前状态。`fstab` 文件中的一行几乎可以直接转换为 Puppet 清单：

```
mount { '/media/gluster-data':
  ensure  => 'mounted',
  device  => 'gluster01:/data',
  fstype  => 'glusterfs',
  options => 'defaults,_netdev',
  dump    => 0,
  pass    => 0,
}
```

对于此资源，Puppet 会确保文件系统在运行后确实被挂载。当然，也可以确保文件系统处于 `unmounted` 状态；Puppet 还可以确保该条目在 `fstab` 文件中是 `present` 的，或在系统中完全 `absent`。

# 总结

在系统上安装 Puppet 后，你可以通过编写和应用清单来使用它。这些清单使用 Puppet 的 DSL 编写，并包含你系统所需状态的描述。尽管它们看起来像脚本，但不应视其为脚本。首先，它们由资源组成，而非命令。这些资源通常不是按它们编写的顺序进行评估的。相反，应通过 `require` 和 `before` 元参数来显式定义顺序。

每个资源都有一些属性：`parameters` 和 `properties`。每个属性都会单独进行评估；Puppet 会检测是否需要对系统进行更改，以将某个属性同步到清单中定义的状态，并且会执行这些更改。这被称为同步资源或属性。

`require` 和 `before` 这两个排序参数非常重要，因为它们建立了一个资源对一个或多个其他资源的依赖关系。这使得 Puppet 可以跳过某些目录中的部分内容，如果一个重要资源无法同步。必须避免循环依赖。

清单中的每个资源都有一个资源类型，用来描述所管理的系统实体的性质。一些最常用的类型包括文件（file）、软件包（package）和服务（service）。Puppet 提供了许多类型用于方便的系统管理，并且有很多插件可供选择以扩展功能。某些任务需要使用 `exec` 资源，但应谨慎使用。

在 第二章，*Puppet 服务器与代理* 中，我们将介绍主机/代理的设置。

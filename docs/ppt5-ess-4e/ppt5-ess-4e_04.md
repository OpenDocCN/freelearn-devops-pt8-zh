# 第四章：在类和定义类型中结合资源

到目前为止，您已经使用 Puppet 执行了一些生产级任务。您学习了如何编写独立的 Manifest，并使用 `puppet apply` 来应用它们。在设置您的第一个 Puppet 主控和代理时，您在主控上为节点 Manifest 创建了一个简单的示例。在 `node '<hostname>'` 块中，您创建了一个 Manifest 文件的等效物。这样，Puppet 主控就只使用特定代理节点的这个 Manifest。

尽管这些内容非常有用且基本重要，但显然不足以应对日常业务需求。通过使用包含一组资源的 `node` 块，您将不得不对类似节点执行大量的复制和粘贴操作，整个结构会迅速变得难以管理。这是一种不自然的方法来开发 Puppet Manifest。尽管与您熟悉的许多其他语言存在显著差异，但 Puppet DSL 是一种编程语言。仅仅通过 `node` 块和资源来构建 Manifest 与仅仅编写只有 `main` 函数的 C 程序或没有自定义类的 Ruby 代码是一样的。

您可以使用已经掌握的工具编写的 Manifest 并不是平面的。您学习了诸如 `if` 和 `case` 等常见的控制结构，您的 Manifest 可以使用这些结构根据 Facter 事实的值查询并根据结果进行分支。

然而，这些结构应该由语言工具来补充，以创建可重复使用的代码单元，类似于过程化语言中的函数或方法。本章通过以下主题介绍这些概念：

+   引入类和定义类型

+   设计模式

+   已定义类型的动态方面

+   类之间的排序和事件

+   通过参数使类更加灵活

# 引入类和定义类型

Puppet 的方法或函数的等效物有两种：一方面是**类**，另一方面是定义类型（也简称为定义）。

你会发现，对于类来说，函数类比有些薄弱，但对于定义类型来说非常合适。

乍一看它们很相似，因为它们都包含一块可重用的 Puppet DSL 代码块。但它们在使用方式上有很大的区别。让我们先看看类。

# 定义和声明类

Puppet 类可以被认为是包含 Puppet 资源声明集合的容器。它只创建一次（类定义），并被所有需要使用准备好的功能的节点使用。每个类代表系统配置的一个众所周知的子集，例如 `ntp`、`nginx` 和 `ssh`。

例如，一个经典的用例 `case` 是一个安装 Apache Web 服务器并应用一些基本设置的类。这个类看起来与以下内容相同：

```
class apache {
  package { 'apache2':
    ensure => present,
  }
  file { '/etc/apache2/apache2.conf':
    ensure => 'file',
    source =>
    'puppet:///modules/apache/etc/apache2/apache2.conf', 
  } 
  service { 'apache2': 
    ensure    => running,
    enable    => true, 
    subscribe => File'/etc/apache2/apache2.conf', 
  } 
} 
```

所有 Web 服务器节点将使用这个类。为此，它们的清单需要包含一个简单的声明：

```
include apache 
```

这称为包含一个类，或者声明它。如果你的 `apache` 类足够强大，可以完成所有必要的任务，那么这一行可能会完全构成 `node` 块的内容：

```
node 'webserver01' {
  include apache
}
```

在你自己的设置中，可能不会编写自己的 Apache 类。你可以使用通过 Puppet 模块提供的开源类。[第五章，*将类、配置文件和扩展组合成模块*，将为你提供所有详细信息。

这就是我们对类的简要概述。虽然还有更多内容要讨论，但我们还是先来看一下已定义类型。

# 创建和使用已定义类型

已定义类型可以看作是一个新的资源类型，它利用了现有的资源类型。当你有多个相同的现有资源类型实例时，使用已定义类型非常有用，因为你可以将它们包装在已定义类型中。作为一个类，它主要由一个包含清单代码的主体构成。然而，已定义类型接收参数，并将这些参数的值作为局部变量在其主体中使用。

这里是另一个典型的已定义类型示例，即 Apache 虚拟主机配置：

```
define virtual_host(
  String $content,
  String[3,3] $priority = '050'
) {
  file { "/etc/apache2/sites-available/${name}":
    ensure  => 'file',
    owner   => 'root',
    group   => 'root',
    mode    => '0644',
    content => $content
  }
  file { "/etc/apache2/sites-enabled/${priority}-${name}": 
    ensure => 'link', 
    target => "../sites-available/${name}";
  }
} 
```

数据类型如 `String` 从 Puppet 4 开始就已可用。在 Puppet 3 及之前的版本中，你会直接跳过这些；所有变量曾经都是无类型的。

这段代码可能看起来还是很晦涩。从实际使用它的其他地方来看，它会变得更加清晰；下面的代码展示了如何使用：

```
virtual_host { 'example.net': 
  content => file('apache/vhosts/example.net') 
}
virtual_host{ 'fallback': 
  priority => '999', 
  content  => file('apache/vhosts/fallback') 
}   
```

这就是为什么这个构造被称为已定义类型；你现在可以在清单中放置看似是资源的内容，但实际上你是在调用你自己定义的清单代码构造。

当声明多个相同类型的资源时，如前面的代码所示，你可以在一个块中进行声明，并用分号分隔它们：

```
virtual_host {
  'example.net':
    content  => 'foo';
  'fallback':
    priority => '999',
    content  => ...,
} 
```

官方风格指南禁止使用这种语法，但在某些情况下，它确实可以使清单更具可读性和可维护性。

`virtual_host` 类型接受两个参数：`content` 参数是必需的，因为它没有默认值，并且在配置文件资源中按原样使用。Puppet 将同步该文件的内容为清单中指定的内容。`priority` 参数是可选的，它的值将作为文件名的前缀。如果省略该参数，相应的虚拟主机定义将使用默认优先级 `050`。

本示例类型的两个参数都是 `String` 类型。有关 Puppet 变量类型系统的详细信息，请参见 第五章，*将类、配置文件和扩展组合成模块*。可以简要地说，你可以将参数限制为某些值类型。然而，这并非强制要求。你可以省略类型名，Puppet 将接受该参数的任何值。

此外，每个定义的类型可以隐式引用它被调用时的名称（或标题）。换句话说，您定义的每个实例都会有一个名称，您可以通过 `$name` 或 `$title` 变量访问它。

在定义类型的主体中，有一些其他的 *魔法* 变量是可用的。如果一个定义类型的资源声明了像 `require => ...` 这样的元参数，它的值可以通过 `$require` 变量在主体中访问。如果没有使用元参数，则该变量值为空。这适用于诸如 `before`、`notify` 等所有元参数，但您可能永远不需要使用这些。元参数会自动做出正确的处理。

# 理解并利用差异

Puppet 的类和定义类型各自的目的非常明确，通常不会重叠。

类声明了与系统某种方式相关的资源和属性。类是您系统中一个或多个方面的最终描述。无论类表示什么，它都只能以一种形式存在；对于 Puppet 来说，每个类本质上是一个单例，是一个固定的信息集，要么应用于您的系统（类被包含），要么不应用。

您将在类中封装并方便地包含在清单中的典型资源如下：

+   一个或多个应该安装（或移除）的包

+   `/etc` 中的特定配置文件

+   用于存储许多子系统脚本或配置文件的常用目录

+   应该在所有适用系统中几乎相同的 Cron 作业

`define` 用于所有存在多个实例的事物。所有在系统中以不同数量出现的方面都可以使用这种语言结构建模。从这个角度来看，`define` 与其声明语法所模拟的完整资源非常相似。定义类型的典型内容包括：

+   `conf.d` 风格目录中的文件

+   如 `/etc/hosts` 之类的易于解析文件中的条目

+   Apache 虚拟主机

+   数据库中的模式

+   防火墙中的规则

该类的单例特性尤为重要，因为它防止了多次资源声明的冲突。请记住，每个资源必须对一个目录是唯一的。例如，考虑 Apache 包的第二次声明：

```
package { 'apache2': }
```

该声明可以出现在您的一个 Web 服务器的清单中的任何位置（例如，直接在 `node` 块中，紧挨着 `include apache`）；这个额外的声明将防止目录的成功编译。

阻止成功编译的原因是 Puppet 目前无法确保两个声明表示相同的目标状态，或者可以合并成一个复合状态。可能多个相同资源的声明会在属性的期望值上发生冲突（例如，一个声明可能希望确保一个包是`absent`，而另一个声明则需要它是`present`）。你应该将 Puppet 视为对系统配置状态的声明性描述。

该类的优点是可以在清单中任意数量地添加相同类的`include`语句。Puppet 只会将该类的内容提交到目录一次。

资源的唯一性约束适用于已定义的类型。你自己定义的两个实例不能共享相同的名称。使用相同的名称两次或更多次会导致编译错误：

```
virtual_host { 'wordpress': 
  content  => file(...), 
  priority => '011', 
}
virtual_host { 'wordpress': 
  content  => '# Dummy vhost', 
  priority => '600', 
}
```

# 设计模式

你对类和定义类型的了解仍然偏学术化。你已经学习了它们的定义方面以及使用它们的语法，但我们还没有让你体会这些概念在不同实际场景中的应用。

以下部分将概述你可以使用这些语言工具做什么。

# 编写全面的类

许多类被编写来让 Puppet 在代理平台上执行重要任务。在这些类中，Apache 类可能是一个较为简单的例子。你可以设想一个类，它可以从任何机器的清单中包含，并确保满足以下条件：

+   防火墙软件已安装并配置了默认规则集。

+   恶意软件检测软件已安装。

+   定时任务按照设定的时间间隔运行扫描器。

+   邮件子系统已配置，以确保定时任务能够传递其输出。

创建此类规模的类有两种常见方法。它可以成为所谓的**单体**实现，一个包含所有资源的大型类，这些资源共同工作以形成期望的安全基线。这种方法的优点是能准确描述你的基础设施，但缺乏可维护性。另一方面，你可以选择**组合**设计，类的主体中几乎没有资源（或者根本没有），而是通过多个`include`语句来包含较简单的类。功能被模块化，中心类充当收集器。这是 Puppet 模块开发人员常用的最佳实践方式。

我们还没有涉及类包含其他类的能力。这是因为这非常简单。类的主体几乎可以包含任何清单，`include`语句也不例外。类中唯一不能出现的几个元素之一就是`node`块。

给描述增添一些生动的元素，大致上各个类将呈现如下样子：

```
class monolithic_security { 
  package { [ 'iptables', 'rkhunter', 'postfix' ]:
    ensure => 'installed';
  } 
  cron { 'run-rkhunter': 
    ... 
  } 
  file { '/etc/init.d/iptables-firewall': 
    source => ... 
    mode => 755 
  }
  file { '/etc/postfix/main.cf': 
    ensure => 'file', 
    content => ... 
  } 
  service { [ 'postfix', 'iptables-firewall' ]: 
    ensure => 'running', 
    enable => true 
  } 
}
class divided_security {
  include iptables_firewall
  include rkhunter
  include postfix
}
```

在开发自己的功能类时，您不应选择这两种极端中的任何一种。大多数类最终会位于这两者之间的某个位置。选择可以在很大程度上基于个人偏好。技术上的影响是微妙的，但它们各自的缺点如下：

+   因此，追求单体类可能会导致资源冲突，因为您几乎没有利用类的单例特性。

+   将类拆分得过于细碎会使得施加秩序和分配刷新事件变得困难，您可以参考本章稍后的“将类、配置文件和扩展组合成模块”部分。

在大多数情况下，这些方面并不是至关重要的。逐案设计选择将基于每个作者的经验和偏好。在不确定的情况下，首先倾向于选择组合设计。

# 编写组件类

还有一个类的常见使用案例。您不仅可以将一个类填充很多协同工作的方面来实现复杂的目标，还可以将类限制为非常特定的目的。有些类只包含一个资源。可以说，这个类将该资源包装起来。

这对于在不同上下文中需要的资源非常有用。通过将它们包装在一个类中，您可以确保这些上下文不会产生同一资源的多次声明。

例如，`netcat`包对防火墙服务器很有用，也可以对 Web 应用服务器有用。可能会有一个`firewall`类和一个`appserver`类。两个类都声明了`netcat`包：

```
package { 'netcat':  
  ensure => 'installed'  
}  
```

如果某个服务器同时具有两个角色（这可能由于预算原因或其他不可预见的情况发生），这将成为一个问题；当同时包含`firewall`和`appserver`类时，结果清单会声明两次`netcat`包。这是禁止的。为了解决这个问题，`netcat`包资源可以被包装在一个`netcat`类中，这个类由`firewall`和`appserver`类共同引用：

```
class netcat { 
  package { 'netcat':  
    ensure => 'installed'  
  }  
}  
```

让我们考虑另一个典型的组件类示例，它确保一个公共文件路径的存在。假设您的 IT 政策要求所有自定义脚本和应用程序都安装在`/opt/company/bin`目录下。许多类，如前面例子中的`firewall`和`appserver`，都需要在这里放置一些相关内容。每个类都需要确保在部署脚本之前，相关目录已经存在。可以通过包含一个组件类来实现，该类包装了`directory`树中的`file`资源：

```
class scripts_directory {  
  file { [ '/opt/company/', '/opt/company/bin' ]:  
    ensure => 'directory',  
    owner  => 'root',  
    group  => 'root',  
    mode   => '0755',  
  }  
} 
```

组件类是一个相当精确的概念。然而，正如您在前一节关于更强大类的部分中看到的那样，所有可能的类设计形式构成了一个细致的尺度，介于所呈现的例子之间。您编写的所有清单很可能会包含多个类。获得最佳实践的最佳方式是直接使用类来构建您所需要的清单。

**全面**类和**组件**类这些术语并不是官方的 Puppet 语言，社区也不会用它们来传达设计实践。我们随意选择它们来描述我们在这些章节中阐述的思想。对定义类型的用例描述也是如此，接下来的章节中将会看到这些用例。

接下来，我们来看一下定义类型的一些用途。

# 使用定义类型作为资源包装器

尽管定义类型与类看似相似，但它们的使用方式有所不同。例如，组件类被描述为*封装一个资源*。这在一个非常特定的上下文中是准确的，封装的资源是一个单例，并且在整个清单中只能以一种形式出现。

当将资源包装在定义类型中时，你最终会得到相应资源类型的变种。清单中可以包含任意数量的定义类型实例，每个实例将封装一个不同的资源。

为了使其生效，在定义类型的主体中声明的资源名称必须是动态创建的。它几乎总是相应定义类型实例的`$name`变量，或者是从它派生的值。

这里是另一个来自许多清单中的典型示例：大多数使用 Puppet 文件服务功能的用户，都会希望在某个时候包装`file`类型，这样就无需为每个文件单独输入相应的 URL：

```
define module_file(String $module) {  
  file { $title:  
    source => "puppet:///modules/${module}/${title}" 
  }  
} 
```

这使得 Puppet 能够轻松地将文件从主服务器同步到代理。主服务器上的副本必须正确放置在主服务器上的命名模块中：

```
module_file { '/etc/ntpd.conf':  
  module => 'ntp': 
}  
```

这个资源将使 Puppet 从`ntp`模块中检索`ntp.conf`文件。前面的声明比完全编写的文件资源和 Puppet URL（尤其是你可能需要同步的大量文件）更简洁、不冗余，后者可能类似于以下内容：

```
file { '/etc/ntpd.conf':  
  source => 'puppet:///modules/ntp/etc/ntpd.conf': 
}  
```

对于像`module_file`这样的包装器，它可能会被广泛使用，你需要确保它支持所有被包装资源类型的属性。在这种情况下，`module_file`包装器应该接受所有`file`属性。例如，以下是如何将`mode`属性添加到包装器类型中的方法：

```
define module_file( 
  String $module, 
  Optional[String] $mode = undef 
) {  
  if $mode != undef {  
    File { mode => $mode }  
  }  
  file { $title:  
    source => "puppet:///modules/${module}/${title}"  
  }  
} 
```

`File { ... }`块声明了相同作用域内所有`file`资源属性的默认值。`undef`值类似于 Ruby 中的`nil`，它是一个方便的默认参数值，因为用户不太可能需要将它作为实际值传递给被包装的资源。

你也可以使用覆盖语法，而不是默认语法：

```
File[$name] { mode => $mode }
```

这使得代码的意图稍微更明显，但在只有一个`file`资源的情况下并不是必须的。第六章，*Puppet 初学者进阶部分*，包含了更多关于覆盖和默认值的信息。

# 使用定义类型作为资源多路复用器

使用已定义类型包装单一资源是有用的，但有时你可能希望在你包装的资源类型之外添加额外的功能。在其他时候，你可能希望已定义类型统一很多功能，就像本节开始时的综合类一样。

对于这两种情况，你需要在已定义类型的主体中有多个资源。这里也有一个经典的例子：

```
define user_with_key(
  String $key, 
  Optional[String] $uid = undef, 
  String $group = 'users'
) {
  user { $title: 
    ensure     => present
    gid        => $group, 
    uid        => $uid,
    managehome => true, 
  }
  ssh_authorized_key { "key for ${title}": 
    ensure => present, 
    user   => $title, 
    type   => 'rsa', 
    key    => $key, 
  } 
}
```

这段代码允许你在一个资源声明中创建具有授权 SSH 密钥的用户帐户。这个代码示例有一些显著的特点：

+   由于你基本上是在包装多个资源类型，所有内部资源的标题都是从当前已定义类型实例的实例标题（或名称）派生的；实际上，这是所有已定义类型的必备实践。

+   你可以硬编码业务逻辑的部分；在这个例子中，我们取消了对非 RSA SSH 密钥的支持，并将 `users` 定义为默认组。

# 使用已定义类型作为宏

一些源代码需要许多重复的任务。假设你的站点使用一个子系统，该子系统依赖于某个位置的符号链接来启用配置文件，就像 `init` 使用 `rc2.d/` 及其兄弟目录中的符号链接，并指向 `../init.d/<service>`。

一个启用大量配置片段的清单可能会看起来像这样：

```
file { '/etc/example_app/conf.d.enabled/england': 
  ensure => 'link', 
  target => '../conf.d.available/england' 
}
file { '/etc/example_app/conf.d.enabled/ireland': 
  ensure => 'link', 
  target => '../conf.d.available/ireland' 
}
file { '/etc/example_app/conf.d.enabled/germany': 
  ensure => 'link', 
  target => '../conf.d.available/germany' 
  ... 
}
```

这段代码很难阅读且维护起来有点痛苦。在 C 程序中，人们会使用预处理器宏，仅使用链接和目标的基本名称，就能扩展为每个资源描述的三行代码。Puppet 不使用预处理器，但你可以使用已定义类型来实现类似的效果：

```
define example_app_config { 
  file { "/etc/example_app/conf.d.enabled/${title}": 
    ensure => 'link', 
    target => "../conf.d.available/${title}", 
  } 
} 
```

该已定义类型实际上更像是一个简单的函数调用，而不是一个真正的宏。

该定义不需要任何参数，它可以仅依赖于资源名称，因此前面的代码现在可以简化为以下内容：

```
example_app_config {'england': }
example_app_config {'ireland': }
example_app_config {'germany': }
... 
```

或者，以下代码更加简洁：

```
example_app_config { [ 'england', 'ireland', 'germany', ... ]: 
}
```

这种数组表示法引出了已定义类型的另一种用途。

# 利用已定义类型中的数组值

编程中一个比较常见的场景是需要接受某个来源的数组值，并对每个值执行任务。Puppet 清单也不例外。

假设之前示例中的符号链接实际上指向的是目录，并且每个这样的目录都包含一个子目录，用于存放指向区域的可选链接。Puppet 也应该管理这些链接。

当然，在了解了已定义类型的宏观概念后，你肯定不希望将每个区域都作为单独的资源添加到你的清单中。然而，你需要设计一种方法，将区域名称映射到国家。既然已经有一个为国家定义的资源类型，那么有一种非常直接的方法：将区域列表作为已定义类型的一个属性（或者说是一个参数）：

```
define example_app_config (
  Array $regions = []
) {
  file { "/etc/example_app/conf.d.enabled/${name}":
    ensure => link,
    target => "../conf.d.available/${name}",
  }
  # to do: add functionality for $regions
}
```

使用该参数非常简单：

```
example_app_config { 'england':
  regions => [ 'South East', 'London' ],
}
example_app_config { 'ireland':
  regions => [ 'Connacht', 'Ulster' ],
}
example_app_config { 'germany':
  regions => [ 'Berlin', 'Bayern', 'Hamburg' ],
}
... 
```

实际的挑战是将这些值付诸实践。一种天真的做法是将以下内容添加到`example_app_config`的定义中：

```
file { $regions:
  path   => "/etc/example_app/conf.d.enabled/${title}/ 
    regions/${name}",
  ensure => 'link',
  target => "../../regions.available/${name}";
}
```

然而，这样做是行不通的。`$name`变量并不引用正在声明的`file`资源的标题。它实际上就像`$title`一样，引用的是外部类或定义类型的名称（在这种情况下是国家名称）。不过，实际的结构应该是你非常熟悉的。唯一缺少的部分是另一个定义类型：

```
define example_app_region(String $country) { 
  file { "/etc/example_app/conf.d.enabled/${country}/regions/${title}":
    ensure => 'link', 
    target => "../../regions.available/${title}",
  } 
}  
```

`example_app_config`定义类型的完整定义应该如下所示：

```
define example_app_config(Array $regions = []) {
  file { "/etc/example_app/conf.d.enabled/${title}": 
    ensure => 'link', 
    target => "../conf.d.available/${title}", 
  } 
  example_app_region { $regions: 
   country => $title,
  } 
}
```

*外部*定义类型通过将其自己的资源名称作为参数值传递，来调整`example_app_region`类型以适应其各自的需求。

# 使用迭代器函数

使用 Puppet 4 及更高版本时，你可能不会像上一节中那样编写代码。得益于新语言特性，使用定义类型作为迭代器已经不再必要。我们将通过以下示例概述这一替代方法，更多详细内容请参考第七章，*来自 Puppet 4 和 5 的新特性*。

现在可以通过使用`each`函数从数组中声明普通的国家链接：

```
[ 'england', 'ireland', 'germany' ].each |$country| { 
  file { "/etc/example_app/conf.d.enabled/${country}": 
    ensure => 'link', 
    target => "../conf.d.available/${country}",
  } 
}   
```

区域可以通过结构化数据声明。对于这个用例，使用哈希就足够了：

```
$region_data = { 
  'england' => [ 'South East', 'London' ], 
  'ireland' => [ 'Connacht', 'Ulster' ], 
  'germany' => [ 'Berlin', 'Bayern', 'Hamburg' ], 
}
$region_data.each |$country, $region_array| {
  $region_array.each |$region| {
    file { "/etc/example_app/conf.d.enabled/${country}/ 
      regions/${region}":
      ensure => link,
      target => "../../regions.available/${region}",
    }
  }
} 
```

在新的清单文件中，你应该选择使用`each`和`map`函数进行迭代，而不是为了此目的使用定义类型。不过，你仍然可以在旧的清单代码中找到前者的示例。有关更多信息，请参见第七章，*来自 Puppet 4 和 5 的新特性*。

# 从定义类型中包含类

上一个示例中定义的`example_app_config`类型应该服务于一个非常特定的目的。因此，它假设基本目录`/etc/example_app`及其子目录在定义类型之外被独立管理。这是一个合理的设计，但许多定义类型是为了被多个独立类或其他定义类型使用的。此类定义需要自包含。

在我们的示例中，定义类型需要确保以下资源是清单的一部分：

```
file { [ '/etc/example_app', '/etc/example_app/config.d.enabled' ]:
  ensure => 'directory',
} 
```

只将此声明放入定义体内将导致重复资源错误。每个`example_app_config`实例都会尝试自己声明目录。然而，我们已经讨论过一种模式来避免这一问题，我们称之为组件类。

为了确保任何`example_app_config`类型的实例都是自包含并能够独立工作，请将前面的声明包裹在一个类中（例如，`class example_app_config_directories`），并确保将此类直接包含在定义体内：

```
define example_app_config(Array $regions = []) {
  include example_app_config_directories
...
} 
```

你可以参考本书附带的示例来查看类的定义。

# 类之间的排序和事件

Puppet 的类与 Java 或 Ruby 等面向对象编程语言中的类几乎没有相似之处。没有方法或属性。没有任何类的独立实例。你不能创建接口或抽象基类。

为数不多的共享特性之一是封装特性。就像面向对象编程（OOP）中的类一样，Puppet 的类隐藏了实现细节。要让 Puppet 开始管理某个子系统，你只需包含适当的类。

# 在类和定义类型之间传递事件

通过将所有资源分类到类中，你使得（你的同事或其他协作者）无需了解每个单独的资源。这是有益的。你可以将类和定义类型的集合视为你的接口。你不希望阅读项目中任何人写过的所有清单文件。

然而，封装在传递资源事件时不方便。假设你有一个守护进程，它从 Apache 日志文件中创建实时统计数据。它应该订阅 Apache 的配置文件，以便在有任何更改时重新启动（这些更改可能会影响该守护进程的操作）。在另一种情况下，你可能会让 Puppet 管理一些外部数据，用于一个自编译的 Apache 模块。如果 Puppet 更新了这些数据，你可能会希望触发 Apache 服务的重启以重新加载所有内容。

凭借已知在 `apache` 类的某处定义了一个服务 `Service['apache2']`，你可以直接让你的模块数据文件通知该资源。如果 Puppet 不对在外部类中声明的资源施加任何保护，它将有效。然而，这可能会带来一个小的可维护性问题。

资源的引用位置远离资源本身。稍后维护清单时，你或你的同事可能希望在遇到引用时查看资源。在 Apache 的情况下，找到查看的位置并不困难，但在其他情况下，引用目标的位置可能不那么明显。

通常不需要查找目标资源，但了解该资源实际做什么可能很重要。特别是在调试过程中，如果更改了清单后，引用的资源找不到时，这一点尤为重要。

此外，这种方法在另一种情况下无法使用，其中你的守护进程需要订阅配置更改。当然，你可以盲目地订阅中央的 `apache2.conf` 文件。然而，如果负责的类选择在 `/etc/apache2/conf.d` 中的片段里做大部分配置工作，这样做就无法达到预期的效果。

通过将 `notify` 或 `subscribe` 参数指向管理相关实体的整个类，可以优雅地解决两种情况：

```
file { '/var/lib/apache2/sample-module/data01.bin':
  source => '...',
  notify => Class['apache'],
}
service { 'apache-logwatch': 
  enable    => true, 
  subscribe => Class['apache'], 
}  
```

当然，现在信号是无差别地发送（或接收）的，文件不仅通知了 `Service['apache2']`，还通知了 `apache` 类中的每一个其他资源。这通常是可以接受的，因为大多数资源会忽略事件。

至于 `logwatch` 守护进程，如果 `apache` 类中的资源需要同步操作，它可能会不必要地刷新自己。此现象发生的概率取决于类的实现。为了获得理想的结果，可能需要将配置文件资源移到它们自己的类中，以便守护进程可以订阅那个类。

对于你定义的类型，你可以应用相同的规则：根据需要订阅和通知它们。这样做非常自然，因为它们本来就是像本地资源一样声明的。这就是你如何订阅多个定义类型实例（如 `symlink`）的方法：

```
$active_countries = [ 'England', 'Ireland', 'Germany' ]
service { 'example-app': 
  enable    => true, 
  subscribe => Symlink[$active_countries], 
} 
```

诚然，这个示例有些笨拙，因为它要求所有 `symlink` 资源标题都必须在一个数组变量中可用。在这种情况下，更自然的做法是让定义类型实例通知服务，而不是这样做：

```
symlink { [ 'England', 'Ireland', 'Germany' ]: 
  notify => Service['example-app'], 
} 
```

该表示法将一个元参数传递给已定义的类型。其结果是该参数值会应用于定义内部声明的所有资源。

如果一个已定义的类型包装或包含了 `service` 或 `exec` 类型的资源，也可能需要通知该定义的实例刷新包含的资源。以下示例假设 `service` 类型被一个名为 `protected_service` 的定义类型所包装：

```
file { '/etc/example_app/main.conf':  
  source => '...',  
  notify => Protected_service['example-app'],  
} 
```

# 容器排序

`notify` 和 `subscribe` 元参数并不是唯一可以直接应用于已定义类型的类和实例的参数，`before` 和 `require` 这两个元参数也适用。这些元参数允许你为资源定义相对于类的顺序，排列已定义类型的实例，甚至对类之间的顺序进行排序。

后者是通过链式操作符来实现的：

```
include firewall 
include loadbalancing 
Class['firewall'] -> Class['loadbalancing'] 
```

这段代码的效果是，所有来自 `firewall` 类的资源将在任何来自 `loadbalancing` 类的资源之前同步，如果前者的任何资源失败，后者的所有资源都将无法同步。

链接箭头不能仅仅放置在 `include` 语句之间。它只在资源定义或资源引用之间有效。

由于这些排序语义，要求整个类同步实际上是相当合理的。你实际上将相关资源标记为依赖于该类。因此，只有在类所建模的整个子系统成功同步后，相关资源才会同步。

# 限制

不幸的是，容器的排序和刷新事件的分发存在一个相当严重的问题：它们都不会超越后续类的 `include` 语句。考虑以下示例：

```
class apache {
  include apache::service
  include apache::package
  include apache::config
}
file { '/etc/apache2/conf.d/passwords.conf':
  source  => '...', 
  require => Class['apache'], 
} 
```

我经常提到，`apache` 类如何全面地模拟 Apache 服务器子系统的一切，在上一节中，我进一步解释了，如果将 `require` 参数指向这样一个类，将确保只有在子系统成功配置后，Puppet 才会接触到相关的依赖资源。

这一点大多是正确的，但由于类边界的限制，它在这个场景中无法实现预期效果。依赖的配置文件实际上应该需要在 `class apache::package` 中声明的 `Package['apache']` 包。然而，这个关系并不会跨越多个类包含，因此这个特定的依赖关系根本不会成为生成目录的一部分。

同样，发送到 `apache` 类的任何刷新事件都不会起作用；它们会被分发到类体内声明的资源（但其中没有任何资源），而不会传递给包含的类。订阅该类也没有意义，因为在包含类内部生成的任何资源事件都不会被 `apache` 类转发。

关键点是，不能完全忽视类的实现来构建类之间的关系。如果有疑问，您需要确保目标类中实际声明了您感兴趣的资源。

讨论围绕着类中的 `include` 语句展开，但由于在定义类型中也常常使用它们；相同的限制在这种情况下也适用。

这里也有一个亮点。根据之前解释的示例，Apache 配置文件的更正确实现应该依赖于该包，但在服务之前也会同步自身，并可能通知它（以便在必要时重启 Apache）。当所有资源都属于 `apache` 类，并且你只想遵循与容器交互的模式时，它将导致如下声明：

```
file { '/etc/apache2/conf.d/passwords.conf':  
  source  => '...',  
  require => Class['apache'],  
  notify  => Class['apache'],  
}  
```

这形成了一个即时的依赖循环：`file` 资源要求在处理之前，`apache` 类的所有部分都必须同步，但为了通知它们，它们必须排在 `file` 资源之后，这在顺序图中是行不通的。通过了解 `apache` 类的内部结构，用户可以选择真正有效的元参数值：

```
file { '/etc/apache2/conf.d/passwords.conf': 
  source  => '...',  
  require => Class['apache::package'],  
  notify  => Class['apache::service'],  
}  
```

对于好奇的人，前面的代码大致展示了内部类的样子。

另一个好消息是，调用定义类型不会像 `include` 语句那样引发问题。事件会正常传递给定义类型内部的资源，跨越任意数量的堆叠调用。排序也像预期的那样正常工作。让我们简要看看这个例子：

```
class apache {  
  virtual_host { 'example.net': ... } 
  ...  
} 
```

这个`apache`类还使用已定义类型`virtual_host`创建了一个虚拟主机。需要此类的资源将隐式地要求`virtual_host`实例中的所有资源。订阅该类的用户将接收到来自这些资源的事件，指向该类的事件也将到达该`virtual_host`的资源。

事实上，有充分的理由使`include`语句在这方面表现不同。由于类可以非常宽松地包含（得益于它们的单例特性），类通常会建立一个庞大的包含网络。通过在清单中添加一个`include`语句，你可能在不知情的情况下将数百个类拉入该清单。假设关系和事件超越了整个网络，随之而来的将是各种意外的效果。依赖循环几乎是不可避免的。整个构造将变得无法管理。此类关系的成本也将呈指数增长。请参阅下一节。

# 容器关系的性能影响

还有一个方面，你在引用容器类型以建立关系时应牢记。Puppet 代理需要从中构建一个依赖关系图。此图包含所有资源作为节点，所有关系作为边。类和已定义类型会扩展为它们所有的声明资源。与容器的所有关系都会扩展为与每个资源的关系。

如果关系的另一端是本地资源，那么这种情况通常是无害的。一个需要五个声明资源的类会导致五个依赖关系，这不会造成问题。如果相同的类被一个包含三个资源的已定义类型的实例所要求，那么每个资源都会与该类的资源建立关系，这样你就会在图中得到 15 条边。

当容器调用复杂的已定义类型时，成本会更高，可能还会是递归调用。

更复杂的图意味着 Puppet 代理需要更多的工作，其运行时间也会更长。当在调试或开发清单时交互式运行代理时，这尤其令人烦恼。为了避免不必要的努力，请仔细考虑你的关系声明，并仅在真正适当时使用它们。

# 缓解限制

Puppet 语言的架构师已经设计了两种替代方法来解决排序问题。我们将考虑这两种方法，因为你可能会在现有的清单中遇到它们。在新的设置中，你应该始终选择后一种方法。

# 锚点模式

`anchor`模式是解决递归类`include`语句中排序和信号传递问题的经典解决方法。可以通过以下示例类进行说明：

```
class example_app { 
  anchor { 'example_app::begin': 
    notify => Class['example_app_config'], 
  } 
  include example_app_config 
  anchor { 'example_app::end': 
   require => Class['example_app_config'],
  } 
} 
```

假设有一个资源被放置在`before`=> Class['example_app']之前。它会被放在链中的每个`anchor`之前，因此也会在`example_app_config`中的任何资源之前，尽管存在`include`的限制。这是因为`Anchor['example_app::begin']`伪资源会通知被包含的类，因此它会被排在所有资源之前。对于需要类的对象，`example::end`锚点也会起到类似的效果。

`anchor`资源类型是专门为此目的创建的。它不是 Puppet 核心的一部分，而是通过`stdlib`模块提供的（下一章会介绍模块）。由于它也转发刷新事件，因此甚至可以通知和订阅这个锚定类，事件将会在被包含的`example_app_config`类内外传播。

`stdlib`模块可以在 Puppet Forge 中找到，更多内容将在下一章介绍。关于`anchor`模式的描述性文档也可以在 Puppet 文档的[`projects.puppetlabs.com/projects/puppet/wiki/Anchor_Pattern`](https://docs.puppet.com/puppet/latest/lang_containment.html)中找到。鉴于锚点模式已经被 Puppet 的类容器功能所取代，这份文档有些过时。

# `contain`功能

为了让复合类能够绕过`include`语句的限制，你可以利用在 Puppet 版本 3.4.x 及更高版本中提供的`contain`功能。

如果之前的`apache`示例写成如下形式，关于顺序和刷新事件就不会再有问题了：

```
class apache {
  contain apache::service
  contain apache::package
  contain apache::config
}
```

官方文档描述了如下行为：

"一个被包含的类在包含类开始之前不会被应用，并且会在包含类结束之前完成。"

这可能让你觉得我们现在在讨论一个可以解决类顺序问题的灵丹妙药。从现在开始，你是否只需要使用`contain`替代`include`，然后再也不用担心类的顺序问题呢？当然不是；这样做会引入许多不必要的顺序约束，并且很快让你陷入无法修复的依赖循环中。使用`contain`类，但要确保这样做是合理的。被包含的类应当真正构成包含类所建模内容的核心部分。

被引用的文档仅提到了类，但类同样可以包含在定义的类型中。包含的效果不仅仅局限于顺序方面。刷新事件也会被正确地传播。

# 通过参数使类更具灵活性

直到目前为止，类和定义在灵活性方面被视为直接对立的；定义类型本身通过不同的参数值具有适应性，而类则建模了单一的静态状态。正如本节标题所示，这并不完全正确。类也可以有参数。在这种情况下，它们的定义和声明与定义类型的定义和声明非常相似：

```
class apache::config(Integer $max_clients=100) { 
  file { '/etc/apache2/conf.d/max_clients.conf':
    content => "MaxClients ${max_clients}\n",
  }
}
```

使用如上所示的定义，类可以用参数值进行声明：

```
class { 'apache::config':  
  max_clients => 120, 
}  
```

这能够实现一些非常优雅的设计，但也带来了一些缺点。

# 参数化类的注意事项

允许类参数的后果几乎显而易见：你会失去单例特性。嗯，这也不完全正确，但你在声明类时的自由度会被大大限制。

定义了所有参数默认值的类仍然可以使用`include`语句进行声明。在同一清单中，这仍然可以任意多次进行。

然而，`class { 'name': }`的类似资源声明不能在同一清单中对任何给定的类出现多于一次。这与资源的规则相一致，不应感到非常惊讶——毕竟，如果在清单的不同位置试图为类的参数绑定不同的值，会显得非常尴尬。

然而，当混合使用`include`和替代语法时，事情变得非常混乱。在使用类似资源的语法声明类后，可以任意次数地包含该类。然而，一旦使用`include`声明了类，就不能再使用资源风格的声明。这是因为参数将被认为采用默认值，而`class { 'name': }`声明与此相冲突。

总之，以下代码是有效的：

```
class { 'apache::config': } 
include apache::config 
```

然而，以下代码无法正常工作：

```
include apache::config 
class { 'apache::config': } 
```

结果是，你实际上不能为组件类添加参数，因为`include`语句在大量使用时不再安全。因此，参数本质上仅对综合类有用，而这些类通常不会从清单的不同部分进行包含。

在第五章，*将类、配置文件和扩展组合成模块*中，我们将讨论一些替代模式，其中一些利用了类参数。还要注意，第八章，*使用 Hiera 实现代码与数据的分离*，提出了一种解决方案，使你在使用参数化类时更加灵活。通过这种方法，你可以更自由地使用类接口。

# 更倾向于使用 include 关键字

自从类参数可用以来，一些 Puppet 用户感到有必要编写（示例）代码，故意放弃`include`关键字，转而使用类似资源的类声明，如下所示：

```
class apache {
  class { 'apache::service': }
  class { 'apache::package': }
  class { 'apache::config': }
}
```

这么做是一个非常糟糕的主意。我们无法过度强调这一点：Puppet 类的最强大概念之一是它们的单例特性，即可以在清单中任意包含类，而无需担心与其他代码的冲突。上述声明语法会剥夺你这种能力，即使相关的类不支持参数。

最安全的做法是尽可能使用`include`，并尽量避免使用替代语法。事实上，第八章，*通过 Hiera 分离代码和数据*，介绍了在没有与类声明相同的资源情况下使用类参数的能力。这样，即使在使用参数时，你也可以完全依赖`include`。这些是最安全的推荐做法，可以帮助你避免与不兼容的类声明发生冲突。

# 总结

类和定义类型是创建可重用 Puppet 代码的基本工具。类包含清单中不能重复的资源，而定义则能够在每次调用时管理一组独特的适配资源。它通过利用接收到的参数值来实现这一点。尽管类也支持参数，但仍有一些需要注意的注意事项。

在清单中使用定义的类型时，你可以像声明本地类型的资源一样声明实例。类主要通过`include`语句使用，尽管也有其他替代方式，例如`class { }`语法或`contain`函数。类的顺序问题也可以通过`contain`函数得到一定缓解。理论上，类和定义足以构建你所需的几乎所有清单。实际中，你可能希望将代码组织成更大的结构。

第五章，*将类、配置文件和扩展组合成模块*，将向你展示如何做到这一点，并向你介绍一系列超出此内容的有用功能。

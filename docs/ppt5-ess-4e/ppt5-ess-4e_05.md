# 第五章：将类、配置文件和扩展组合成模块

在上一章中，你了解了将类和定义类型模块化和可重用的工具。我们讨论了几乎所有 Puppet 资源都应该被分离到适当的类中，除非它们在逻辑上需要作为定义类型的一部分。这几乎提供了足够的语法来构建一个完整代理节点集群的 manifests；每个节点选择适当的复合类，而复合类又包含其他所需类，所有类会递归地实例化定义类型。

直到现在为止，还没有讨论过 manifests 在文件系统中的组织方式。显然，将所有代码塞进一个大的 `site.pp` 文件是不理想的。解决这个问题的方法是通过模块实现的，本章将对此进行解释。

除了组织类和定义，模块还是共享公共代码的一种方式。它们是 Puppet manifests 和插件的软件库。模块还提供了一个便捷的位置，用来存放在前一章中提到的接口描述。Puppet Labs 运营着一个专门的服务，用于托管开源模块，称为 Puppet Forge。

模块的存在和一般位置在 第三章 中简要提到过，*Puppet 中 Ruby 部分的初探 - Facts、Types 和 Providers*。现在是时候更详细地探讨这些内容以及其他方面了。本章将涵盖以下主题：

+   Puppet 模块的内容

+   管理环境

+   构建一个组件模块

+   查找有用的 Forge 模块

# Puppet 模块的内容

模块可以看作是一个更高级的组织单元。它将有助于实现共同管理目标的类和定义类型捆绑在一起（例如，特定的系统方面或软件）。这些 manifests 并不是模块组织的全部内容；大多数模块还会捆绑文件和文件模板。模块中也可以包含几种类型的 Puppet 插件。本节将解释模块的不同部分，并展示它们的存放位置。你还将了解模块文档的手段以及如何获取现有模块以供使用。

# 模块的组成部分

对于大多数模块，**manifests** 是最重要的部分——核心功能。manifests 由类和定义类型组成，这些类和类型共享一个命名空间，以模块名称为根。例如，`ntp` 模块只包含名称以 `ntp::` 前缀开头的类和定义。

许多模块包含可以同步到代理文件系统的文件。这通常用于配置文件或代码片段。你已经看过这些示例，但让我们再重复一遍。许多 manifests 中常见的资源之一是 `file` 资源，例如以下内容：

```
file { ‘/etc/ntp.conf’: 
  source => ‘puppet:///modules/ntp/ntp.conf’, 
}  
```

前述资源引用的是一个随虚拟`ntp`模块提供的文件。它已准备好提供一般适用的配置数据。然而，通常需要调整这类文件中的某些参数，以便节点清单可以为相应的代理声明定制的配置设置。用于此目的的工具是模板，将在第六章，*Puppet 初学者高级部分*中讨论。

模块的另一个可能组成部分是自定义事实 - 这是在请求目录之前同步到代理并运行的代码，以便将输出作为有关代理系统的事实提供。

这些事实并不是可以与模块一起提供的唯一 Puppet 插件。还有**解析器函数**（也称为**自定义函数**）。在许多情况下，它们是构建某些特定实现的最便捷方式，如果不是唯一方式。

在前文已经暗示过的最终插件类型是自定义本地类型和提供者，它们也方便地放置在模块中。

# 模块结构

所有提及的组件都需要位于特定的文件系统位置，以便主节点进行拾取。每个模块形成一个目录树。其根目录以模块本身命名。例如，`ntp`模块存储在名为`ntp/`的目录中。

所有清单都存储在名为`manifests/`的子目录中。每个类和定义类型都有各自的文件。例如，`ntp::package`类将在`manifests/package.pp`中找到，名为`ntp::monitoring::nagios`的定义类型将在`manifests/monitoring/nagios.pp`中找到。容器名称的第一个部分（`ntp`）始终是模块名称，其余描述了`manifests/`下的位置。您可以参考下文的模块树获取更多示例。

`manifests/init.pp`文件是特殊的。可以将其视为默认的清单位置，因为它被查找以获取来自相关模块的任何定义。

刚才提到的示例都可以放在`init.pp`中，仍然可以工作。不过，这样做会使得定位定义变得更加困难。

在实践中，`init.pp`应仅包含一个类，其名称与模块相同（例如`ntp`类），如果您的模块实现了这样的类。这是一种常见的做法，因为它允许清单使用简单语句来调用模块的核心功能：

```
include ntp 
```

您可以参考*模块最佳实践*部分获取有关此主题的更多注释。

模块为代理提供的文件和模板并没有严格分类到特定位置。重要的是它们分别被放置在 `files/` 和 `templates/` 子目录中。这些子树的内容可以按照模块作者的喜好进行结构化，并且清单必须正确引用它们。静态文件应始终通过 URL 进行访问，例如以下这些：

```
puppet:///modules/ntp/ntp.conf 
puppet:///modules/my_app/opt/scripts/find_my_app.sh 
```

这些文件位于 `files/` 的相应子目录中：

```
.../modules/ntp/files/ntp.conf 
.../modules/my_app/files/opt/scripts/find_my_app.sh 
```

URI 中的 `modules` 前缀是必须的，后面必须跟着模块名称。其余路径直接对应于 `files/` 目录中的内容。模板也有类似的规则。有关详细信息，请参阅第六章，*《Puppet 初学者进阶部分》*。

最后，所有插件都位于 `lib/` 子树中。自定义事实是存放在 `lib/facter/` 中的 Ruby 文件。解析器函数存储在 `lib/puppet/parser/functions/` 中，Puppet 4 API 函数位于 `lib/puppet/functions/`，而自定义资源类型和提供者分别位于 `lib/puppet/type/` 和 `lib/puppet/provider/` 中。这并非巧合；这些 Ruby 库由主节点和代理节点在相应的命名空间中查找。本章稍后会提供所有这些组件的示例。

简而言之，以下是树状视图中可能出现的模块内容：

```
/opt/puppetlabs/code/environments/production/modules/my_app
    |- templates/ # templates are covered in the next chapter
    |- files/
    | |- subdir1/ # puppet:///modules/my_app/subdir1/<filename>
    | |- subdir2/ # puppet:///modules/my_app/subdir2/<filename>
    | | \- subsubdir/ # puppet:///modules/my_app/subdir2/subsubdir/...
    |- manifests/
    | |- init.pp # class my_app is defined here
    | |- params.pp # class my_app::params is defined here
    | |- config/
    | | |- detail.pp # my_app::config::detail is defined here
    | | \- basics.pp # my_app::config::basics is defined here
    \- lib/
        |- facter/ # contains .rb files with custom facts
        \- puppet/
           |- functions # contains .rb files with Puppet 4 functions
           |- parser/
           | \- functions # contains .rb files with parser functions
           |- type/ # contains .rb files with custom types
           \- provider/ # contains .rb files with custom providers
```

# 模块中的文档

一个模块可以并且应该包含文档。Puppet 主节点不会自动处理任何模块文档。因此，模块作者在如何为其特定站点创建的模块结构化文档方面有很大的决定权。尽管如此，仍然存在一些常见的做法，遵循这些做法是一个好主意。此外，如果一个模块最终要在 Forge 上发布，适当的文档应当视为强制性的。

发布模块的过程超出了本书的范围。你可以在[`docs.puppetlabs.com/puppet/latest/reference/modules_publishing.html.`](https://docs.puppet.com/puppet/latest/modules_publishing.html)找到相关指南。

对于许多模块，文档的主要焦点集中在 `README` 文件上，该文件位于模块的根目录中。它通常以 Markdown 格式书写，文件名为 `README.md` 或 `README.markdown`。`README` 文件应包含解释说明，通常还包括参考文档。

Puppet DSL 接口也可以直接在清单中以 `rdoc` 和 `YARD` 格式进行文档化。这适用于类和定义的类型：

```
# Class: my_app::firewall
#
# @summary This class adds firewall rules to allow access to my_app.
#
# @example Declaring the class
# include my_app::firewall
#
# @param Parameters: none
class my_app::firewall {
  # class code here
}
```

你可以使用`puppet strings`子命令为所有模块生成 HTML 文档（包括导航）。安装 puppet-strings Ruby 扩展后，便可以使用此子命令：`puppet resource package puppet-strings provider=puppet_gem`。这个做法相对较为晦涩，因此这里不会详细讨论。然而，如果这个选项对你有吸引力，我们鼓励你阅读相关文档。

以下命令概述了可能的 puppet strings 功能：

```
puppet help strings 
```

# 管理环境

Puppet 并不是仅通过模块来组织内容的。还有一个更高层次的单元，叫做**环境**，它将模块进行分组和包含。一个环境主要由以下部分组成：

+   一个或多个站点清单文件

+   一个包含模块的`modules`目录

+   一个可选的`environment.conf`配置文件

当管理员为节点编译清单时，它仅使用一个环境来执行此任务。如第二章《Puppet 服务器和代理》中所述，它始终从`manifests/*.pp`开始，这些文件形成环境的站点清单。在我们查看实际操作是如何进行之前，先来看看一个示例`environment`目录：

```
/opt/puppetlabs/code/environments/
    \- production/
         |- environment.conf
         |- manifests/
         | |- site.pp
         | \- nodes.pp
         \- modules/
             |- my_app/
             \- ntp/
```

`environment.conf`文件可以自定义环境。通常，Puppet 使用`site.pp`和`manifests`目录中的其他文件。若要让 Puppet 读取另一个目录中的所有`pp`文件，可以在`environment.conf`中设置`manifest`选项：

```
#/opt/puppetlabs/code/environments/production/environment.conf 
manifest = puppet_manifests
```

在大多数情况下，无需更改`manifest`选项。

`site.pp`文件将包含来自模块的节点分类。Puppet 会在活动环境的`modules`子目录中查找模块。你可以通过在`environment.conf`中设置`modulepath`选项，定义包含模块的额外子目录：

```
#/opt/puppetlabs/code/environments/production/environment.conf 
modulepath = modules:site-modules 
```

目录结构可以做得更加具有区分度：

```
/opt/puppetlabs/code/environments/
     \- production/
         |- manifests/
         |- modules/
         | \- ntp/
         \- site-modules/
             \- my_app/
```

# 配置环境位置

Puppet 默认使用`production`环境。这个环境以及其他环境预期位于`/opt/puppetlabs/code/environments`目录中。你可以通过在`puppet.conf`中设置`environmentpath`选项来覆盖此默认设置：

```
[main]
environmentpath = /etc/local/puppet/environments
```

# 获取和安装模块

下载现有模块是非常常见的。Puppet Labs 拥有一个专门的站点，用于共享和获取模块——Puppet Forge。它的工作方式与 RubyGems 或 CPAN 相同，并且使用户可以通过命令行界面轻松获取指定模块。在 Forge 中，模块的命名方式是通过将作者的名字前缀到实际模块名称上，例如`puppetlabs-stdlib`或`ffrank-constraints`。

`puppet module install`命令会在活动环境中安装一个模块：

```
root@puppetmaster# puppet module install puppetlabs-stdlib
```

*测试你的模块*部分提供了关于如何使用不同环境的信息。

当前版本的`stdlib`模块（由用户`puppetlabs`编写）从 Forge 下载并安装到标准模块位置。这是当前环境的`modulepath`中的第一个位置，通常是`modules`子目录。具体来说，这些模块很可能会最终安装到`environments/production/modules`目录中。

特别是`stdlib`模块应被视为必需的；它为 Puppet 语言添加了大量有用的功能。比如`keys`、`values`和`has_key`函数，这些对于正确处理哈希结构至关重要，举几个例子。这些函数一旦模块安装完成，就可以在你的清单文件中使用，无需包含任何类或其他显式加载。如果你编写自己的模块来添加函数，这些函数也会以相同的方式自动加载。

# 模块最佳实践

在所有当前版本的 Puppet 中，你应该养成将所有清单代码放入模块中的习惯，只有以下几个例外：

+   `node`块

+   应包含那些应该始终存在的类的`include`语句（最常见的设计模式是在所谓的基础角色中进行处理；不过，具体的角色和配置模式请参考第九章，*Puppet 角色与配置文件*）

+   声明有用的变量，这些变量应该在你的清单文件中具有与 Facter 事实相同的可用性

本节提供了如何相应地组织你的清单文件的详细信息。它还建议了一些设计实践和策略，用以测试模块的更改。

# 将所有内容放入模块中

你可能会在一些非常老的安装中找到一些清单文件，它们将大量的清单文件聚集在一个或多个目录中，并在`site.pp`文件中使用`import`语句，例如：

```
import '/etc/puppet/manifests/custom/*.pp' 
```

这些文件中的所有类和定义的类型都将全局可用。

这种方法存在可扩展性问题，早已被弃用。`import`关键字在 Puppet 4 及以后的版本中已不再支持。

给类和定义类型起有意义的名称会更加高效，这样 Puppet 可以在模块集合中查找它们。这个方案在前面已经讨论过了，现在我们来看一个示例，Puppet 编译器遇到一个类名时，比如：

```
include ntp::server::component::watchdog 
```

Puppet 将继续在当前环境的所有已配置模块位置中查找`ntp`模块（即`modulepath`设置中的路径）。然后，它会尝试读取`ntp/manifests/server/component/watchdog.pp`文件，以查找类定义。如果失败，它将尝试`ntp/manifests/init.pp`。

这样会使得编译变得非常高效。Puppet 会动态识别所需的清单文件，并仅包括那些文件进行解析。它还帮助代码检查和开发，因为非常清楚在哪里可以找到特定的定义。

从技术上讲，确实可以将一个模块的所有清单放入其`init.pp`文件中，但这样做会失去结构化模块清单树所提供的优势。

# 避免泛化

每个模块理想情况下应当服务于一个特定的目的。在依赖 Puppet 来管理多样化服务器基础设施的网站上，可能会有针对每个服务的模块，比如`apache`、`ssh`、`nagios`、`nginx`等等。这些模块中的大多数将来自上游开发，并被称为“技术组件模块”。也可以有网站特定的模块，例如`users`或`shell_settings`，如果操作需要这种细粒度的控制。这些自定义模块有时只是以拥有它们的团队或公司命名。

理想的粒度取决于你设置的具体需求。你通常希望避免使用诸如`utilities`或`helpers`这样的模块名称，这些模块作为无法归入任何现有模块的想法的“集大成者”。这种缺乏组织的情况会对规范性产生不利影响，可能导致混乱的模块，包含本应成为各自独立模块的定义。

添加更多模块是很便宜的。一个模块通常不会对 Puppet 主控操作产生额外成本，而且随着模块数量的增加，用户体验通常会变得更加高效，而不是降低效率。当然，如果你的网站对每个模块有特殊的文档或其他处理要求，这种平衡可能会被打破。这些规则必须在模块组织决策时进行权衡。

# 测试你的模块

根据你的代理网络的规模，你的一些或许多个模块可以被各种各样的节点使用。尽管这些节点有很多共同之处，它们之间可能仍然存在显著差异。对一个中央模块的更改，例如`ssh`或`ntp`，可能会对大量代理产生广泛的后果。

测试工作最重要的工具是 Puppet 的`--noop`选项。它适用于`puppet agent`以及`puppet apply`。如果在命令行中给出该选项，Puppet 将不会执行任何必要的同步操作，而仅仅将相应的输出行呈现给你。在第一章中有这个的示例，*编写你的第一个清单*。

但是，当使用主控而不是本地使用`puppet apply`时，会出现一个新问题。所有代理都会查询主控。除非在你测试新清单时禁用所有代理，否则很可能会有代理检查并不小心运行未经测试的代码。

事实上，即使在你登录时，你的测试代理也可以在后台透明地触发其常规运行。

防范此类无法控制的清单应用非常重要。一个小错误可能会在短时间内损坏大量代理机器。最好的做法是，在主节点上定义多个环境并逐步进行代码更改。第九章*，Puppet 角色与配置文件*将进一步介绍相关内容。

# 使用环境进行安全测试

除了`production`环境之外，你应该至少创建一个测试环境。你可以称其为`testing`或其他任何你喜欢的名称。使用目录环境时，只需在`environmentpath`中创建其目录即可。

这样的附加环境在测试更改时非常有用。测试环境或多个环境应为生产数据的副本。首先在`testing`中准备所有清单更改。你可以让代理在将其复制到生产环境之前测试此更改：

```
root@agent# puppet agent --test --noop --environment testing
```

你甚至可以省略某些或所有代理上的`noop`标志，以便实际部署更改。某些在清单中出现的细微错误无法通过检查`noop`输出来检测出来，因此在发布之前，通常最好至少运行一次代码。

当环境与源代码控制结合使用时，它们的效果更为显著，特别是像`git`或`mercurial`这样的分布式系统。无论是否使用环境和测试，为你的 Puppet 代码进行版本控制都是一个好主意；这正是 Puppet 通过其基础设施即代码（Infrastructure as Code）理念为你提供的最大优势之一。

使用环境和`noop`模式形成了一个务实的测试方法，适用于大多数场景。当然，针对错误的 Puppet 行为的安全性是有限的。还有更正式的模块测试方式：

+   `rspec-puppet`工具允许模块作者基于`rspec`实现单元测试。你可以在[`rspec-puppet.com/`](http://rspec-puppet.com/)找到更多详细信息。

+   可以通过`beaker`进行验收测试。有关详细信息，请参考[`github.com/puppetlabs/beaker/wiki/How-To-Beaker`](https://github.com/puppetlabs/beaker/wiki/How-To-Beaker)

详细解释这些工具超出了本书的范围。

# 构建组件模块

本章已讨论了模块的许多理论和操作方面，但你尚未了解编写模块的过程。为此，本章的其余部分将一步一步带领你创建一个示例模块。

再次强调，大多数情况下，你会想从 Forge 上寻找通用模块。可用模块的数量不断增加，因此很有可能已经有某个模块可以帮助你完成所需的工作。

假设你想在你的网络中添加 Cacti：一个基于 RRD 工具的趋势监控和图形服务器，包括一个 web 界面。如果你首先检查 Forge，你确实会找到一些模块。但是，假设这些模块都没有符合你的需求，因为它们的功能集或实现方式不合适。如果连这些模块的接口都不符合要求，那么基于现有模块（如在 GitHub 上分叉）来开发自己的模块也没有多大意义。那你就需要从头开始编写自己的模块。

# 为你的模块命名

模块名称应该简洁且直观。如果你管理的是特定的软件，可以根据它来命名模块——`apache`、`java`、`mysql` 等等。避免使用动词，如 `install_cacti` 或 `manage_cacti`。如果模块名称需要由多个单词组成（因为目标子系统名称较长），它们应由下划线字符分隔。禁止使用空格、连字符和其他非字母数字字符。

在我们的示例中，模块应该被命名为 `cacti`。

通常，你不会编写像 apache、mysql、java 这样的模块名称，因为这些名称已经被上游开发使用了。当学习 Puppet 时，想要从一个简单的模块实现开始，可能上游模块太复杂，难以理解。在这种情况下，你应该用公司或团队的名称作为前缀来命名模块。记住不要使用连字符，而应使用下划线分隔公司/团队名称，例如 `packt_apache`、`infra_mysql`。这个命名模式可以保留原始的命名空间，并且在以后迁移到上游模块时更加容易。

# 使你的模块对 Puppet 可用

要使用你自己的模块，你不需要通过 `puppet module` 使其可安装。为此，你需要先将模块上传到 Forge，这将需要额外的工作。不过幸运的是，如果你只是将源代码放在主节点的适当位置，模块就能正常工作，而不需要进行所有这些准备工作。

要创建你自己的 `cacti` 模块，请创建基本目录：

```
root@puppetmaster# mkdir -p /opt/puppetlabs/code/environments/testing/packt_cacti/{manifests,files}  
```

一旦代理使用了它们，别忘了将所有的更改同步到 `production`。

# 实现基础的模块功能

大多数模块通过其清单文件完成所有工作。

有一些显著的例外，比如 `stdlib` 模块。它主要增加了解析器函数和一些通用资源类型。

在规划模块的类时，最直接的思考方式是考虑你希望如何使用完成的模块。接口设计的方式有很多种。事实上的标准规定，管理的子系统通过在代理系统中包含模块的主类进行初始化；该类与模块同名，并且实现于模块的 `init.pp` 文件中。

对于我们的 Cacti 模块，用户应该使用以下内容：

```
include packt_cacti 
```

结果，Puppet 将采取所有必要的步骤来安装软件，并在需要时执行任何额外的初始化。

首先创建`cacti`类，并以命令行方式实现设置，将命令替换为适当的 Puppet 资源。在 Debian 系统中，安装`cacti`包就足够了。其他所需的软件通过依赖项引入（完成 LAMP 堆栈），并且在包安装后，接口通过服务器机器上的 Web URI`/cacti/`可用：

```
# .../modules/packt_cacti/manifests/init.pp 
class packt_cacti { 
  package { 'cacti': 
    ensure => installed, 
  } 
} 
```

你的模块现在可以进行测试了。通过在`site.pp`或`nodes.pp`的`testing`环境中从代理的清单中调用它：

```
node 'agent' { 
  include packt_cacti 
} 
```

直接在代理上应用它：

```
root@agent# puppet agent --test --environment testing  
```

这在 Debian 上可以工作，Cacti 可以通过`http://<address>/cacti/`访问。

一些站点使用**外部节点分类器**（**ENC**），例如 Foreman。除了其他有用的功能外，它还可以集中分配环境到节点。在这种情况下，`--environment`开关将不起作用。

遗憾的是，当通过`/` URI 请求主页时，Cacti 的 Web 界面不会显示。为了启用此功能，请为模块提供配置适当重定向的能力。在模块中的`/opt/puppetlabs/code/environments/testing/packt_cacti/files/etc/apache2/conf.d/cacti-redirect.conf`准备一个 Apache 配置片段：

```
# Do not edit this file - it is managed by Puppet! 
RedirectMatch permanent ^/$ /cacti/ 
```

警告通知很有帮助，特别是当多个管理员可以访问 Cacti 服务器时。

添加一个专门的类来将此文件同步到代理机器是有意义的：

```
# .../modules/packt_cacti/manifests/redirect.pp
class packt_cacti::redirect {
  file { '/etc/apache2/conf.d/cacti-redirect.conf':
    ensure => file,
    source => 'puppet:///modules/packt_cacti/etc/apache2/conf.d/cacti-redirect.conf',
    require => Package['cacti'];
  }
}
```

这样简短的文件也可以通过`file`类型的`content`属性而不是`source`来管理：

```
$puppet_warning = '# Do not edit - managed by Puppet!'
$line = 'RedirectMatch permanent ^/$ /cacti/'
file { '/etc/apache2/conf.d/cacti-redirect.conf':
  ensure  => file,
  content => "${puppet_warning}\n${line}\n",
}
```

这样做更高效，因为内容是目录的一部分，因此代理无需通过另一个请求从主服务器获取校验和。

该模块现在允许用户`include packt_cacti::redirect`以获取此功能。这本身并不是一个糟糕的接口，但这种修改实际上非常适合成为`cacti`类的一个参数：

```
class packt_cacti( 
  $redirect = true,) 
{ 
  if $redirect { 
    contain packt_cacti::redirect 
  } 
  package { 'cacti': 
    ensure => installed, 
  } 
} 
```

当清单使用`include cacti`时，重定向现在默认安装。

如果 Web 服务器有其他虚拟主机提供非 Cacti 的内容，这可能是不希望的。在这种情况下，清单将声明带有以下参数的类：

```
class { 'packt_cacti': 
  redirect => false,
} 
```

说到最佳实践，大多数模块也会将安装过程拆分为独立的类。在我们的例子中，这几乎没有帮助，因为安装状态通过一个单一资源得到保证，但我们还是这样做：

```
class packt_cacti( 
  $redirect = true, 
) { 
  contain packt_cacti::install 
  if $redirect { 
    contain packt_cacti::redirect 
  } 
} 
```

在这里使用`contain`是明智的，以便使 Cacti 管理成为一个独立的单元。`cacti::install`类被放入一个单独的`install.pp`清单文件中：

```
# .../modules/packt_cacti/manifests/install.pp 
class packt_cacti::install { 
  package { 'cacti': 
    ensure => 'installed', 
  } 
} 
```

在 Debian 上，`cacti`包的安装过程会将另一个 Apache 配置文件复制到`/etc/apache2/conf.d`。由于 Puppet 执行的是常规的`apt`安装，因此会达到这一结果。然而，Puppet 并不会确保配置保持在期望的状态。

配置可能会被破坏，这确实存在实际风险。如果给定节点使用`puppetlabs-apache`模块，它通常会清除`/etc/apache2/`目录下的任何未管理的配置文件。在为现有服务器启用此模块时要非常小心。在`noop`模式下进行测试。如果需要，修改清单以包含现有的配置。

明智的做法是向清单中添加一个`file`资源，用来保存安装后状态下的配置片段。通常在 Puppet 中，这需要你将配置文件内容复制到模块中，就像重定向配置文件一样位于主服务器上的某个文件中。然而，由于 Cacti 的 Debian 包包含了位于`/usr/share/doc/cacti/cacti.apache.conf`的配置片段，你可以指示代理与此同步实际配置。在另一个事实标准模块`config`类中执行此操作：

```
# .../modules/packt_cacti/manifests/config.pp 
class packt_cacti::config {  
  file { '/etc/apache2/conf.d/cacti.conf':  
    mode   => '0644',  
    source => '/usr/share/doc/cacti/cacti.apache.conf',  
  }  
}  
```

这个类应该也包含在`packt_cacti`类中。重新运行代理现在不会产生任何效果，因为配置已经到位。

# 为派生清单创建实用工具

现在，你已经创建了几个类，这些类将模块的基本安装和配置工作进行了模块化。类非常适合实现与管理软件整体相关的全局设置。

然而，仅仅安装 Cacti 并使其网页界面可用，毕竟并不是一个特别强大的功能，模块做的事情不过是用户通过包管理器安装 Cacti 时能够完成的工作。Cacti 的一个更大的痛点是，它通常需要通过网页界面进行配置；添加服务器以及为每个服务器选择和配置图表可能是一个繁琐的任务，并且根据你的图表需求的复杂程度，可能需要每个服务器数十次点击。

这就是 Puppet 最有帮助的地方。期望状态的文本表示允许通过正则表达式快速地进行复制粘贴和名称替换。更好的是，一旦有了 Puppet 接口，用户可以自己设计定义类型，从而避免重复的复制粘贴工作。

说到定义类型，它们是你模块所需的，用于支持这种配置方式。Cacti 配置中的每台机器应该是一个定义类型的实例。图表也可以有自己的类型。

与类的实现一样，你首先需要问自己的是，如何通过命令行来完成这项任务。

事实上，更好的问题可能是应该使用哪个 API，最好是从 Ruby 出发。然而，这只有在你打算编写 Puppet 插件（资源类型和提供者）时才重要。我们将在本章稍后讨论这个问题。

Cacti 附带了一组 CLI 脚本。Debian 包将这些脚本放在 `/usr/share/cacti/cli` 目录下。让我们在实现 Puppet 接口的过程中探索这些脚本。目标是定义类型，这些类型将有效地封装命令行工具，以便 Puppet 始终能够通过适当的查询和更新命令保持定义的配置状态。

# 添加配置项

在为 Cacti 模块设计更多功能时，首先是能够注册一个用于监控的机器——或者更确切地说，是一个 **设备**，正如 Cacti 自身所称（网络基础设施，如交换机和路由器也经常被监控，而不仅仅是计算机）。因此，第一个定义类型的名称应该是 `cacti::device`。

来自 *命名你的模块* 小节的相同警告适用——除非你有非常充分的理由（如移除是不可能的），否则不要被诱惑给你的类型命名为 `create_device` 或 `define_domain` 等。即便如此，最好还是跳过动词。

用于注册设备的 CLI 脚本名为 `add_device.php`。它的帮助输出清楚地指示需要两个参数，即 `description` 和 `ip`。自定义实体的描述通常是相应 Puppet 资源标题的良好用途。现在类型几乎已经可以自然而然地写出来了：

```
# .../modules/packt_cacti/manifests/device.pp 
define packt_cacti::device ( 
  $ip, 
) { 
  $cli = '/usr/share/cacti/cli' 
  $options = "--description='${title}' --ip='${ip}'" 
  exec { "add-cacti-device-${title}": 
    command => "${cli}/add_device.php ${options}", 
    require => Class['cacti'], 
} 
```

实际上，通常不需要使用这么多变量，但它有助于在页面有限的水平空间中提高可读性。

这个 `exec` 资源赋予 Puppet 使用 CLI 在 Cacti 配置中创建新设备的能力。由于 PHP 是 Cacti 包的要求之一，因此只需要让 `exec` 资源 `require` `cacti` 类即可。注意 `$title` 的使用，不仅用于 `--description` 参数，还用于 `exec` 资源的资源名称。这确保每个 `packt_cacti::device` 实例都声明一个唯一的 `exec` 资源来创建自身。

`exec` 资源类型允许以 root 权限和空的 shell 环境运行任意命令。这使得能够灵活地在 Puppet DSL 中封装配置命令。但 `exec` 资源类型也有其缺点：`exec` 资源类型本身并不是幂等的，并且存在所有操作都通过运行命令完成的风险，这与 Puppet 作为声明式配置管理系统的性质相悖。最好的做法是将 `exec` 资源类型视为应急出口：只有在没有其他方法可以实现目标时才使用它。

通常，自定义资源类型更为合适，特别是在运行复杂的命令时需要处理复杂的检查选项。自定义资源类型将在本章后面讲解。

然而，这仍然缺少一个重要方面。如前面的示例所写，这个`exec`资源会使 Puppet 代理在任何情况下都运行 CLI 脚本。虽然这是不正确的——它只应在设备尚未添加时才运行。

每个`exec`资源应当包含`creates`、`onlyif`或`unless`中的一个参数。它定义了一个查询，供 Puppet 用来判断当前的同步状态。除非设备已经存在，否则必须执行`add_device`调用。现有设备的查询必须通过`add_graphs.php`脚本（这可能不太直观）进行。当使用`--list-hosts`选项时，它会打印一行头信息和一个设备列表，设备描述位于第四列。以下`unless`查询将找到相关资源：

```
$search = "sed 1d | cut -f4- | grep -q '^${title}\$'" 
exec { "add-cacti-device-${title}": 
  command => "${cli}/add_device.php ${options}", 
  path    => '/bin:/usr/bin', 
  unless  => "${cli}/add_graphs.php --list-hosts |  
              ${search}", 
  require => Class[cacti], 
} 
```

`path`参数非常有用，因为它允许调用核心实用程序时无需提供完整路径。

一般设置标准的搜索路径列表是个好主意，因为某些工具在`PATH`环境变量为空时无法正常工作。

如果在设备列表中找到了完全相同的资源标题，`unless`命令将返回`0`。最后的`$`符号被转义，以便 Puppet 在`$search`命令字符串中按字面意义包括它。

你现在可以通过将以下资源添加到代理机器的清单中来测试你新的定义：

```
# in manifests/nodes.pp 
node 'agent' { 
  include packt_cacti 
  packt_cacti::device { 'Puppet test agent (Debian 7)':  
    ip => $::ipaddress, 
  }  
} 
```

在下一次执行`puppet agent --test`时，你会收到通知，表示添加设备的命令已经执行。再次执行，Puppet 会判断所有内容已经与目录同步。

# 允许自定义

`add_device.php`脚本有一系列可选参数，允许用户自定义设备。Puppet 模块应当暴露这些参数。我们来选择其中一个并在`packt_cacti::device`类型中实现。每个 Cacti 设备都有一个默认为`tcp`的`ping_method`。使用这个模块时，我们甚至可以覆盖软件的默认值：

```
define packt_cacti::device(
  $ip,
  $ping_method='icmp'
){
  $cli = '/usr/share/cacti/cli'
  $base_opt = "--description='${title}' --ip='${ip}'"
  $ping_opt = "--ping_method=${ping_method}"
  $options = "${base_opt} ${ping_opt}"
  $search = "sed 1d | cut -f4- | grep -q '^${title}\$'"
  exec { "add-cacti-device-${title}":
    command => "${cli}/add_device.php ${options}",
    path    => '/bin:/usr/bin',
    unless  => "${cli}/add_graphs.php --list-hosts | ${search}",
    require => Class[cacti],
  }
}
```

该模块默认使用`icmp`而不是`tcp`。无论是否将其传递给`packt_cacti::device`实例，值都会始终传递给 CLI 脚本。在后一种情况下，使用的是参数默认值。

如果你计划发布你的模块，尽可能使用与托管软件相同的默认值会更加明智。

一旦你加入了所有可用的 CLI 开关，你将成功创建一个 Puppet API，用于将设备添加到你的 Cacti 配置中，用户将因此受益于便捷的复现、共享、隐式文档、简单的版本控制等功能。

# 移除不需要的配置项

仍然有一个剩下的问题。对于 Puppet 类型来说，无法删除它们创建的实体是不典型的。当前，这实际上是你模块所依赖的 CLI 的技术限制，因为它尚未实现`remove_device`功能。此类脚本已经在互联网上发布，但在撰写本文时，它们尚未正式成为 Cacti 的一部分。

为了使模块更具功能性，合理的做法是将额外的 CLI 脚本加入到模块的文件中。将适当的文件放入`modules/cacti/files/`下的正确目录，并向`cacti::install`类中添加另一个`file`资源：

```
file { '/usr/share/cacti/cli/remove_device.php': 
  ensure  => file, 
  mode    => '0755', 
  source  => 
       'puppet:///modules/packt_cacti/usr/share/cacti/cli/
     remove_device.php', 
  require => Package['cacti'], 
} 
```

然后，你可以向`cacti::device`类型添加一个`ensure`属性：

```
define packt_cacti::device( 
  $ensure='present', 
  $ip, 
  $ping_method='icmp', 
{ 
  $cli = '/usr/share/cacti/cli' 
  $search = "sed 1d | cut -f4- | grep -q '^${title}\$'" 
  case $ensure { 
  'present': { 
    # existing cacti::device code goes here 
  } 
  'absent': { 
    $remove = "${cli}/remove_device.php" 
    $get_id = "${remove} --list-devices | awk -F'\\t' 
       '\$4==\"${title}\" { print \$1 }'" 
    exec { "remove-cacti-device-${name}": 
        command => "${remove} --device-id=\$( ${get_id} 
      )", 
        path    => '/bin:/usr/bin', 
        onlyif  => "${cli}/add_graphs.php --list-hosts | 
           ${search}", 
        require => Class[cacti], 
      } 
    } 
  } 
} 
```

请注意，我们在这里对缩进做了一些处理，以避免断行过多。这个新的`exec`资源很复杂，因为`remove_device.php`脚本需要删除设备的数字 ID。这是通过执行`--list-devices`调用并将其传递给`awk`来获取的。为了进一步影响可读性，像双引号、`$`符号和反斜杠等内容必须进行转义，以便 Puppet 在清单中包含有效的`awk`脚本。

还要注意，查询这个`exec`资源的同步状态与`add`资源的查询是相同的，唯一的区别是现在它与`onlyif`参数一起使用：仅在配置中仍然找到相关设备时才执行操作。

# 处理复杂性

我们为`packt_cacti::device`定义实现的命令相当复杂。在这种复杂度的层面，Shell 单行命令变得不再适用于驱动 Puppet 的资源。当处理 Cacti 图表时情况变得更糟；`add_graphs.php` CLI 脚本不仅需要设备的数字 ID，还需要图表的数字 ID。在这种情况下，将复杂性移出清单并为实际的 CLI 编写包装脚本是合理的。我将简要描述这个实现。包装脚本将遵循这个大致模式。

```
#!/bin/bash 
DEVICE_DESCR=$1 
GRAPH_DESCR=$2 
DEVICE_ID=` #scriptlet to retrieve numeric device ID` 
GRAPH_ID=`  #scriptlet to retrieve numeric graph ID` 
GRAPH_TYPE=`#scriptlet to determine the graph type` 
/usr/share/cacti/cli/add_graphs.php \ 
  --graph-type=$GRAPH_TYPE \ 
  --graph-template-id=$GRAPH_ID \ 
  --host-id=$DEVICE_ID
```

使用这个，你可以添加一个简单的`graph`类型：

```
define packt_cacti::graph( 
  $device, 
  $graph=$title 
) { 
  $add = '/usr/local/bin/cacti-add-graph' 
  $find = '/usr/local/bin/cacti-find-graph' 
  exec { "add-graph-${title}-to-${device}": 
    command => "${add} '${device}' '${graph}'", 
    path    => '/bin:/usr/bin', 
    unless  => "${find} '${device}' '${graph}'", 
  } 
} 
```

这还需要一个额外的`cacti-find-graph`脚本。添加这个脚本带来了额外的挑战，因为当前的 CLI 没有列出已配置图表的功能。可以添加到`cacti`模块的功能还包括管理 Cacti 的数据源、改变设备选项，甚至可能是更改配置中已存在的其他对象的选项。

这样的商品超出了基本内容的范畴，暂时不做详细说明。让我们看看`cacti`模块的其他部分作为示例。

# 通过插件增强代理

可重用的类和定义使得使用你模块的清单更加具有表现力。现在安装和配置 Cacti 变得简洁，而执行这一操作的清单变得非常易读和可维护。

现在是时候探索模块的另一个更强大的方面：Puppet 插件了。插件的不同类型包括自定义事实（在第三章中讨论过，*Puppet Ruby 部分概览 - 事实、类型和提供者*）、解析器函数、资源类型和提供者。所有这些插件都存储在主机上的模块中，并会同步到所有代理上。代理不会使用解析器函数（虽然在同步后，`puppet apply` 的用户可以在代理机器上使用它们）；相反，事实和资源类型在代理上执行大部分工作。现在让我们集中讨论类型和提供者；其他插件将在后续的专门章节中讨论。

本节可以视为可选内容。许多用户永远不会接触任何资源类型或提供者的代码，因为清单本身已经提供了所需的所有灵活性。如果你不关心插件，可以跳过本节，直接阅读关于如何查找 Forge 模块的最后部分。另一方面，如果你对 Ruby 技能有信心，并希望在 Puppet 安装中利用它们，继续阅读，了解自定义类型和提供者如何为你提供帮助。

虽然自定义资源类型在主机和代理上都可以使用，但提供者将主要在代理端完成所有工作。虽然资源类型也主要通过代理执行，但它们对主机有一个影响；它们使清单能够声明该类型的资源。代码不仅描述了哪些属性和参数存在，还可以包含对各自值的验证和转换代码。该部分由代理调用。有些资源类型甚至会自行处理同步和查询，尽管通常至少会有一个提供者来负责这些操作。

在上一节中，你通过封装一些 `exec` 资源实现了一个定义类型，并完成了所有的同步工作。通过 Puppet 安装二进制文件和脚本，你可以通过这种方式实现几乎任何功能，且无需编写任何插件。然而，这种做法也有一些缺点：

+   理想情况下，输出是晦涩的，而在出现错误时则会让人不知所措。

+   Puppet 每个资源至少会调用一个外部进程；在许多情况下，需要多个子进程。

简而言之，你需要付出代价，无论是可用性还是性能方面。考虑 `packt_cacti::device` 类型。对于每个声明的资源，Puppet 在每次运行时都会运行 `exec` 资源的 `unless` 查询（或者在指定 `ensure => absent` 时，运行 `onlyif`）。这包括一次调用 PHP 脚本（可能会比较昂贵），以及几个必须解析输出的核心工具。在拥有数十或数百个受管设备的 Cacti 服务器上，这些调用会累积起来，使代理花费大量时间来生成和等待这些子进程。

另一方面，考虑一个提供者。它可以实现一个`instances`钩子，这将在初始化时创建一个已配置的 Cacti 设备的内部列表。总共只需要一个 PHP 调用，所有输出的处理可以直接在代理进程中的 Ruby 代码内完成。仅这些节省的操作就能使每次运行成本大大降低：已经同步的资源不会受到任何惩罚，因为不需要运行额外的外部命令。

在我们继续实现一个简单的类型/提供者配对之前，先快速看一下代理输出。以下是`cacti::device`类型创建设备时的输出：

```
Notice: /Stage[main]/Main/Node[agent]/Packt_cacti::Device[Agent_VM_Debian_7]/Exec[add-cacti-device-Agent_VM_Debian_7]/returns: executed successfully  
```

本地类型以更简洁的方式表达这些操作，例如来自`file`资源的输出：

```
Notice: /Stage[main]/Main/File[/usr/local/bin/cacti-search-graph]/ensure: created  
```

# 使用本地类型替换定义类型

创建一个与之匹配的提供者的自定义资源类型的过程

（或者多个提供者）并不容易。让我们一步步了解其中的步骤：

+   命名你的类型

+   创建资源类型的接口

+   设计合理的参数钩子

+   使用资源名称

+   添加提供者

+   声明管理命令

+   实现基本功能

+   允许提供者预取现有资源

+   在配置过程中使类型更强健

# 命名你的类型

本地类型和定义类型之间的第一个重要区别是命名。自定义类型没有像定义类型那样的模块命名空间，定义类型是基于清单的。本地类型来自所有已安装的模块，可以自由地混合使用。它们使用普通名称。因此，简单地将`packt_cacti::device`的本地实现命名为`device`是不明智的——这很容易与其他模块可能有的设备概念发生冲突。命名你第一个资源类型的明显选择是`cacti_device`。

该类型必须完全实现于`packt_cacti/lib/puppet/type/cacti_device.rb`。所有钩子和调用都将封装在`Type.newtype`块中：

```
Puppet::Type.newtype(:cacti_device) do 
  @doc = <<-EOD 
    Manages Cacti devices. 
    EOD 
end 
```

`@doc`中的文档字符串应视为强制性的，并且它应该比这个例子更有实质性。考虑包含一个或多个示例资源声明。将所有后续的代码片段放在`EOD`终止符和最终的`end`之间。

# 创建资源类型的接口

首先，类型应该具有`ensure`属性。Puppet 的资源类型有一个方便的助手方法，通过简单调用生成所有必要的类型代码：

```
ensurable 
```

在类型的主体中通过这个方法调用，你可以添加典型的`ensure`属性，包括所有必要的钩子。这个行在类型代码中是唯一需要的（实际实现将在提供者中跟进）。大多数属性和参数需要更多的代码，就像`ip`参数一样：

```
require 'ipaddr' 
newparam(:ip) do 
  desc "The IP address of the device." 
  isrequired 
  validate do |value| 
    begin 
      IPAddr.new(value) 
    rescue ArgumentError 
      fail "'#{value}' is not a valid IP address" 
    end 
  end 
  munge do |value| 
    value.downcase 
  end 
end
```

通常这应该是一个`ip`属性，但提供者将依赖于 Cacti CLI，而 Cacti CLI 无法更改已配置的设备。如果 IP 地址是一个属性，那么为了执行属性值同步，需要进行此类更改。

如你所见，IP 地址参数的代码主要由验证组成。

在文件顶部附近添加`require 'ipaddr'`行，而不是放在`Type.newtype`块内。

该参数现在可用于`cacti_device`资源，代理甚至会拒绝添加 IP 地址无效的设备。这对用户很有帮助，因为明显的地址错误将被早期检测到。在我们更仔细地查看`munge`钩子之前，先实现下一个参数。

# 设计合理的参数钩子

接下来是`ping_method`参数，它只接受来自有限集合的值，因此验证很容易：

```
newparam(:ping_method) do 
  desc "How the device's reachability is determined. 
    One of `tcp` (default), `udp` or `icmp`." 
  validate do |value| 
    [ :tcp, :udp, :icmp ].include?(value.downcase.to_sym) 
  end 
  munge do |value| 
    value.downcase.to_sym 
  end 
  defaultto :tcp 
end 
```

仔细查看`munge`块，你会注意到它们旨在统一输入值。对于参数来说，这比属性要不那么关键，但如果将来某个 Cacti 模块版本中这些参数被改为属性，它将不会尝试将`ping_method`的`tcp`同步为`TCP`。如果用户在清单中更喜欢使用大写字母，后者可能会出现。通过 munging，两个值都会变成`:tcp`。对于 IP 地址，调用`downcase`仅对 IPv6 有影响。

超出 Puppet 本身的范围，参数值的 munging 也很重要。它允许 Puppet 接受比被管理的子系统更方便的值。例如，Cacti 可能不接受`TCP`作为值，但 Puppet 会接受，并且会正确处理它。

# 使用资源名称

你需要处理最后一个要求：每个 Puppet 资源类型必须声明一个`name 变量`或简称`namevar`。如果资源本身未指定该参数，该参数将使用清单中的资源标题作为其值。例如，`exec`类型具有`command`参数作为其`namevar`。你可以将可执行命令放入资源标题中，或显式声明该参数：

```
exec { '/bin/true': } 
# same effect: 
exec { 'some custom name': command => '/bin/true' } 
```

若要将现有参数标记为名称变量，请在该参数的主体中调用`isnamevar`方法。如果类型有一个名为`:name`的参数，它会自动成为名称变量。这是一个安全的默认值。

```
newparam(:name) do 
  desc "The name of the device." 
  #isnamevar # → commented because automatically assumed 
end 
```

资源类型现在已经可以在清单中使用：

```
cacti_device { 'eth0': 
  ensure      => present, 
  ip          => $::ipaddress, 
  ping_method => 'icmp', 
} 
```

这段代码将被编译到一个目录中，但代理会抛出错误，因为没有提供者可用。

# 添加一个提供者

资源类型本身已经准备好执行操作，但缺少执行系统检查和执行同步的提供者。让我们一步步构建它，就像构建资源类型一样。提供者的名称不需要反映它所针对的资源类型，而应包含对其实现的管理方法的引用。由于你的提供者将依赖于 Cacti CLI，命名为`cli`即可。如果多个提供者为不同类型提供功能，共享相同的名称也是可以的。

在`packt_cacti/lib/puppet/provider/cacti_device/cli.rb`中创建框架结构：

```
Puppet::Type.type(:cacti_device).provide( 
  :cli, 
  :parent => Puppet::Provider 
  ) do 
end 
```

实际上，指定`:parent => Puppet::Provider`并不是必要的。`Puppet::Provider`是提供者的默认基类。如果你为一个子系统编写了几个类似的提供者（每个提供者针对不同的资源类型），而这些提供者都依赖相同的工具链，你可能希望实现一个基类提供者，作为所有兄弟提供者的父类。

现在，让我们集中精力构建一个自给自足的`cli`提供者，针对`cacti_device`类型。首先，声明你将需要的命令。

# 声明管理命令

提供者使用`commands`方法来方便地绑定可执行文件

Ruby 标识符：

```
commands :php => ‘php’
commands :add_device => ‘/usr/share/cacti/cli/add_device.php’
commands :add_graphs => ‘/usr/share/cacti/cli/add_graphs.php’
commands :rm_device => ‘/usr/share/cacti/cli/remove_device.php’
```

你不会直接调用`php`。它在这里被包含是因为声明命令有两个作用：

+   你可以通过生成的方法方便地调用命令

+   只有在找到所有命令后，提供者才会标记自己为`valid`

因此，如果在 Puppet 的搜索路径中找不到`php` CLI 命令，Puppet 会认为该提供者是无效的。用户可以通过 Puppet 的调试输出快速判断这一错误情况。

# 实现基本功能

提供者的基本功能现在可以通过三个实例方法来实现。这些方法的名称本身并没有什么特别的含义，但这些是默认`ensure`属性期望可用的方法（记住你在类型代码中使用了`ensurable`快捷方式）。

第一个方法是创建一个资源（如果它尚不存在）。它必须收集所有资源参数的值，并构建一个合适的`add_device.php`调用：

```
def create 
  args = [] 
  args << "--description=#{resource[:name]}" 
  args << "--ip=#{resource[:ip]}" 
  args << "--ping_method=#{resource[:ping_method]}" 
  add_device(*args) 
end 
```

不要像在命令行中那样引用参数值。Puppet 会为你处理这个问题。它还会转义参数中的任何引号，因此在这种情况下，Cacti 会接收到这些引号并将其包含在配置中。例如，这将导致标题不正确。

```
args << "--description='#{resource[:name]}'" 
```

提供者还必须能够删除或`销毁`实体：

```
def destroy 
  rm_device("--device-id=#{@property_hash[:id]}") 
end 
```

`property_hash` 变量是提供者的实例成员。每个资源都有其特定的提供者实例。继续阅读，了解它如何被初始化以包含设备的 ID 号。

在我们进入这部分之前，让我们先添加最后一个提供者方法，以实现`ensure`属性。这是一个查询方法，代理会用来判断资源是否已经存在：

```
def exists? 
  self.class.instances.find do |provider| 
    provider.name == resource[:name] 
  end 
end 
```

`ensure`属性依赖于提供者类方法`instances`，以获取系统上所有实体的`providers`列表。它将每个实体与`resource`属性进行比较，`resource`属性是当前提供者实例正在执行操作的资源类型实例。如果这让你感到困惑，请参考下一节的图示。

# 允许提供者预先获取现有资源

`instances`方法非常特殊——它在提供者初始化期间实现了系统资源的预取。你必须自己将其添加到提供者中。一些子系统不适合大规模获取所有现有资源（例如`file`类型）。这些提供者没有`instances`方法。而枚举 Cacti 设备则是完全可行的：

```
def self.instances
  return @instances ||= add_graphs(“--list-hosts”).
    split(“\n”).
    drop(1).
    collect do |line|
      fields = line.split(/\t/, 4)
      Puppet.debug “prefetching cacti_device #{fields[3]} 
      “ +
                   “with ID #{fields[0]}”
      new(:ensure => :present,
            :name => fields[3],
              :id => fields[0])
    end
end
```

提供者实例的`ensure`值反映了当前状态。该方法为系统中找到的资源创建实例，因此对于这些资源，值始终是`present`。还要注意，方法的结果会缓存在`@instances`类成员变量中。这一点非常重要，因为`exists?`方法会调用`instances`，这可能会频繁发生。

Puppet 需要另一种方法来执行正确的预取操作。你通过`instances`实现的大规模获取，向代理提供了一份代表系统上实体的提供者实例列表。代理从主节点接收资源类型实例的列表。然而，Puppet 还没有在资源（类型实例）和提供者之间建立关系。你需要在提供者类中添加一个`prefetch`方法来实现这一点：

```
def self.prefetch(resources)
  instances.each do |provider|
    if res = resources[provider.name]
      res.provider = provider
    end
  end
end 
```

代理将`cacti_device`资源作为哈希传递，资源标题作为相应的键。这使得查找非常简单（且快速）。

这完成了`cli`提供者对`cacti_device`类型的支持。你现在可以用`cacti_device`实例替换你的`cacti::device`资源，以享受更好的性能和更清晰的代理输出：

```
node "agent" {
  include cacti
  cacti_device { ‘Puppet test agent (Debian 7)":
    ensure => present,
    ip     => $::ipaddress,
  }
}
```

请注意，与您定义的类型`cacti::device`不同，本地类型不会为其`ensure`属性假设默认值`present`。因此，您必须为任何`cacti_device`资源指定此值。否则，Puppet 只会管理已存在资源的属性，而不关心实体是否存在。在`cacti_device`的特殊情况下，这样做永远不会有任何作用，因为没有其他属性（仅有参数）。

你可以参考第六章，*Puppet 初学者进阶部分*，了解如何使用资源默认值来避免重复的`ensure => present`规范。

# 在配置过程中增强类型的健壮性

`packt_cacti`模块还有一个小问题。它是自给自足的，并处理 Cacti 的安装和配置。然而，这意味着在 Puppet 的第一次运行中，`cacti`包及其 CLI 将不可用，代理会正确判断`cli`提供者尚不适用。由于它是`cacti_device`类型的唯一提供者，在`cacti`包之前同步的任何此类型的资源都会失败。

在`packt_cacti::device`定义类型的例子中，你只是将`require`元参数添加到了内部资源。为了在原生类型实例中实现相同的效果，你可以使用`autorequire`特性。就像文件自动依赖于它们所在的目录一样，Cacti 资源应该依赖于`cacti`包的成功同步。将以下代码块添加到`cacti_device`类型中：

```
autorequire :package do 
  catalog.resource(:package, 'cacti') 
end 
```

# 通过事实增强 Puppet 的系统知识

当在第三章中介绍事实时，*《Puppet 中 Ruby 部分概述 - 事实、类型和提供者》*，你对创建自定义事实的过程进行了简要了解。

我们在那时提到过模块，现在，我们可以更详细地了解事实代码如何部署，以下是`Cacti`模块的例子。我们将重点关注原生 Ruby 事实——它们比外部事实更具可移植性。由于后者容易创建，这里无需深入讨论它们。

关于外部事实的详细信息，你可以参考 Puppet Labs 网站上的自定义事实在线文档：[`docs.puppetlabs.com/facter/latest/custom_facts.html#external-facts`](http://docs.puppetlabs.com/facter/latest/custom_facts.html#external-facts)。

事实是 Puppet 插件的一部分，模块可以包含它们，就像前面章节中的类型和提供者一样。它们属于`lib/facter/`子树。对于`cacti`模块的用户，了解给定 Cacti 服务器上有哪些图表模板可能会很有帮助（当然前提是图表管理功能已经实现）。完整的列表可以通过一个事实传递。以下代码位于`packt_cacti/lib/facter/cacti_graph_templates.rb`，将执行此操作：

```
Facter.add(:cacti_graph_templates) do
  setcode do
    cmd = ‘/usr/share/cacti/cli/add_graphs.php’
    Facter::Core::Execution.exec(“#{cmd} --list-graph-
    templates”).
      split(“\n”).
      drop(1).
      collect do |line|
        line.split(/\t/)[1]
      end
  end
end
```

该代码将调用 CLI 脚本，跳过其输出的第一行，并将每一行其余部分的第二列的值合并成一个列表。清单可以通过全局变量`$cacti_graph_templates`访问此列表，正如访问任何其他事实一样。

# 通过自定义函数优化模块的接口

函数在保持你的清单干净且易于维护方面非常有帮助，有些任务甚至无法在没有 Ruby 函数的情况下实现。

自定义函数的一个常见用法（特别是在 Puppet 3 中）是输入验证。您可以在清单本身中执行此操作，但由于语言的限制，这可能会让人感到沮丧。生成的 Puppet DSL 代码可能很难阅读和维护。`stdlib` 模块提供了许多基本数据类型的 `validate_X` 函数，例如 `validate_bool`。Puppet 4 及更高版本中的类型化参数使这一过程更加方便和自然，因为对于支持的变量类型，已经不再需要验证函数。

与所有插件一样，这些函数不需要特定于模块的领域，它们会立即对所有清单可用。举个例子，`packt_cacti` 模块可以使用 `packt_cacti::device` 参数的验证函数。检查一个字符串是否包含有效的 IP 地址与 Cacti 完全没有关系。另一方面，检查 `ping_method` 是否是 Cacti 识别的那些方法，则不那么通用。

为了查看它是如何工作的，让我们实现一个函数，完成 `packt_cacti::device` 中 IP 地址参数的 `validate` 和 `munge` 钩子的功能。如果地址无效，编译应该失败；否则，它应该返回统一的地址值：

```
module Puppet::Parser::Functions
  require ‘ipaddr’
  newfunction(:cacti_canonical_ip, :type => :rvalue) do |args|
    ip = args[0]
    begin
      IPAddr.new(ip)
    rescue ArgumentError
      raise “#{@resource.ref}: invalid IP address ‘#{ip}’”
    end
    ip.downcase
  end
end
```

在异常消息中，`@resource.ref` 会扩展为出现问题的资源类型实例的文本引用，例如 `Packt_cacti::Device[Edge Switch 03]`。

以下示例说明了在没有 `ensure` 参数的简单版本 `cacti::device` 中使用该函数：

```
define packt_cacti::device($ip) {
  $cli = ‘/usr/share/cacti/cli’
  $c_ip = cacti_canonical_ip(${ip})
  $options = “--description=‘${name}’ --ip=‘${c_ip}’”
  exec { “add-cacti-device-${name}”:
    command => “${cli}/add_device.php ${options}”,
    require => Class[cacti],
  }
}
```

如果 IP 地址（巧妙地）存在，清单将无法编译：

转置数字：

```
ip => '912.168.12.13' 
```

IPv6 地址将被转换为全小写字母。

Puppet 4 引入了更强大的 API 用于定义自定义函数。请参考 第六章，*Puppet 初学者高级部分*，了解其优势。

# 使您的模块能够在多个平台上移植

不幸的是，我们的 `Cacti` 模块非常依赖 Debian 包。它期望在特定位置找到 CLI，在另一个位置找到 Apache 配置片段。这些位置很可能是特定于 Debian 包的。如果模块能够在 Red Hat 衍生系统上也能正常工作，那就更好了。

第一步是通过执行手动安装来了解差异。我选择使用运行 Fedora 18 的虚拟机进行测试。基本的安装与 Debian 相同，当然，使用的是 `yum` 而不是 `apt-get`。Puppet 会自动在这里执行正确的操作。`puppet::install` 类也包含一个 CLI 文件。Red Hat 包将 CLI 安装在 `/var/lib/cacti/cli`，而不是 `/usr/share/cacti/cli`。

如果模块应该支持两个平台，那么`remove_device.php`脚本的目标位置不再固定。因此，最好从模块的中心位置部署脚本，而在代理系统上的目标位置则成为一个模块参数，如果您愿意的话。这些值通常会在`params`类中收集：

```
# …/packt_cacti/manifests/params.pp
class packt_cacti::params {
  case $osfamily {
    ‘Debian’: {
      $cli_path = ‘/usr/share/cacti/cli’
    }
    ‘RedHat’: {
      $cli_path = ‘/var/lib/cacti/cli’
    }
    default: {
      fail “the cacti module does not yet support the 
      ${osfamily} 
        platform”
    }
  }
}
```

最好是针对不支持的代理平台失败编译。用户必须从其模块中删除`cacti`类的声明，而不是让 Puppet 尝试未经测试的安装步骤，这很可能不起作用（这可能涉及 Gentoo 或 BSD 变体）。

需要访问变量值的类必须包含`params`类：

```
class packt_cacti::install {
  include pack_cacti::params
  file { ‘remove_device.php’:
    ensure => file,
    path   => 
     “${packt_cacti::params::cli_path}/remove_device.php’,
    source => 
    ‘puppet:///modules/packt_cacti/cli/remove_device.php’,
    mode   => ‘0755’,
  }
}
```

对于`cacti::redirect`类和`cacti::config`类，可能需要类似的转换。只需向`params`类添加更多变量。这不仅限于清单；事实和提供者也必须按照代理平台的行为。

您经常会看到`params`类是继承而不是包含：

```
class packt_cacti(
  $redirect = ${packt_cacti::params::redirect}
)inherits packt_cacti::params{
  # ...
}
```

这是因为类主体中的`include`语句不允许使用`params`类中的变量值作为类参数的默认值，比如此示例中的`$redirect`参数。

您自己的自定义模块通常不需要可移植性实践。在理想情况下，您不会在多个平台上使用它们。但如果打算在 Forge 上共享它们，则应该考虑这种实践是强制性的。对于大多数 Puppet 需求，您不希望编写模块，而是从 Forge 下载现有的解决方案。

在 Puppet 4.9 及更高版本中，params 类模式将不再需要用于传递默认参数值。取而代之的是一种新的数据绑定机制。该机制在《第八章》中有详细解释，*使用 Hiera 分离代码与数据*。

# 查找有用的 Forge 模块。

使用[`forge.puppetlabs.com`](http://forge.puppetlabs.com/)的 Web 界面非常直观。通过填写软件、系统或服务的名称来搜索，通常会得到一列非常合适的模块，而它们的名称往往就是你搜索的关键词。事实上，对于常见术语，可用模块的数量可能会让人眼花缭乱。

您可以立即了解每个模块的成熟度和流行度。如果模块正在积极使用和维护：

+   它的分数接近 5。

+   它有一个版本号，表示超过 1.0.0（甚至 0.1.0）的发布。

+   最近的版本发布并不久，也许不到半年的时间。

+   它有大量的下载量。

然而，后面三个数字可能会有很大差异，这取决于模块实现的功能数量以及其主题的普及程度。更重要的是，仅仅因为某个模块获得了大量关注和定期贡献，并不意味着它是最适合你情况的选择。

你被鼓励评估那些访问量较少的模块——这样你可能会发现一些隐藏的宝藏。下一节详细介绍了一些质量的深层指标，供你参考。

如果你不能，或者不想花太多时间去挖掘最合适的模块，也可以参考 Puppet 支持和 Puppet 批准的模块侧边栏。所有在这些类别中展示的模块都得到了 Puppet Labs 的质量认证。

# 识别模块特性

在 Forge 中导航到模块的详细信息时，会显示其`README`文件。一个空的或非常简略的文档表明模块作者并未花费多少心思。`README`文件中的示例清单通常是快速启动模块的一个好起点。

如果你正在寻找能够通过额外的资源类型和提供者增强代理的模块，可以查看模块详细信息页面上的类型标签。点击模块描述顶部附近的项目 URL 链接也是一个有启发性的做法。这个链接通常指向 GitHub。在这里，你不仅可以方便地浏览`lib/`子树中的插件，还可以大致了解模块清单的结构。

另一个精心维护模块的标志是单元测试。这些测试可以在`spec/`子树中找到。大多数 Forge 模块都有这个子树。不过，这个子树通常没有实际的测试。可能会有所有类和定义类型的测试代码文件，这些文件通常位于`spec/classes/`和`spec/defines/`子目录中。对于插件，理想情况下会在`spec/unit/`和`spec/functions/`中有单元测试。

一些模块的`README`文件中包含一个绿色的标签，写着**构建通过**。这个标签有时会变红，显示**构建失败**。这些模块通过 GitHub 使用 Travis CI，因此它们很可能至少有一些单元测试。

# 摘要

所有的 Puppet 开发都应该在模块中进行，每个模块应尽可能服务于一个特定的目的。大多数模块只包含清单文件。这足以提供非常有效且易读的节点清单，通过包含恰当命名的类和实例化定义的类型，清晰简洁地表达其意图。

模块还可以包含 Puppet 插件，形式包括资源类型和提供者、解析函数或事实。这些通常都是 Ruby 代码。然而，外部事实可以用任何语言编写。虽然编写自己的类型和提供者不是必须的，但它可以提高你的性能和管理灵活性。

并不需要自己编写所有模块。相反，尽可能依赖于来自 Puppet Forge 的开源模块是一个明智的选择。Puppet Forge 是一个不断增长的有用代码集合，几乎涵盖了 Puppet 能够管理的所有系统和软件。特别是，由 Puppet Labs 策划的模块通常质量非常高。像所有开源软件一样，您非常欢迎自行添加任何缺失的需求到这些模块中。

在对 Puppet 的整体构建模块有了一个广泛的了解后，第六章，*Puppet 初学者进阶部分*，将进一步缩小范围。现在，您已经具备了构建和组织清单代码库的工具，接下来您将学习一些精细的技巧，以优雅地解决一些 Puppet 中独特的问题。

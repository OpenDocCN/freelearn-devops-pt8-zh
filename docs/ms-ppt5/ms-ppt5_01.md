# 编写模块

编写 Puppet 模块和清单是 Puppet 生态系统工作的真正核心。

所以，你可能已经为你的基础设施中的软件组件编写了至少几个模块，而且 Puppet 文档中有一个很好的入门指南：[`puppet.com/docs/pe/2017.3/quick_start_guides/writing_modules_nix_getting_started_guide.html`](https://puppet.com/docs/pe/2017.3/quick_start_guides/writing_modules_nix_getting_started_guide.html)，所以我不会浪费时间再讲解这些内容。但我敢肯定，在掌握 Puppet v5 的过程中，你真正想要做的是正确地编写这些模块。

让我们在本章中一起迈出提高模块质量的一步。我在过去几年里花了大量时间深入一线，收集来自欧洲一些最佳项目的最佳实践，并应用我从大学教育和 15 年以上的行业经验中学到的实践和软件原则。我希望能向你介绍一些捷径，让你的工作更轻松！

以下是我认为会真正帮助你走上高质量 Puppet 模块和清单正确路径的一些建议：

+   使用合适的 IDE 和插件

+   使用良好的模块类结构：

    +   遵循类命名约定

    +   为模块提供一个单一入口点

    +   使用高内聚和低耦合原则

    +   使用封装原则

    +   强类型化你的模块变量

+   使用新的 Puppet 开发工具包命令：

    +   创建模块框架和元数据

    +   创建 `init.pp`

    +   创建更多的类

    +   验证你的模块

    +   对模块进行单元测试

+   持续关注代码气味

+   确保你没有在处理死代码

+   与社区合作

+   使用 Puppet Forge

+   编写优秀的文档

+   添加模块依赖

+   为你的模块添加兼容性数据

    +   操作系统支持

    +   Puppet 和 PE 版本支持

+   使用新的 Hiera 5 模块级别数据

+   将模板从 ERB 语法升级到 ERP 语法

现在，让我们依次检查这些最佳实践。

# 使用合适的 IDE 和插件

使用一个合适的文本编辑器并配合插件进行编写，绝对是提高质量的好步骤。有很多选项可以选择，最好选择适合你个人写作风格的工具。就我个人而言，我最成功地使用了 Atom（[`atom.io`](https://atom.io)），并最近将其安装在我的工作站上。我多年前使用过 Eclipse（这个工具也曾被称为 Geppetto），不过我觉得它由于较大的内存占用显得笨重。保持一定的 Vim 熟练度也很不错，尤其是在服务器端的命令行操作，或者如果你在工作站上使用 Linux 操作系统的话。另外，还有适用于 macOS X 的 TextMate 编辑器，它拥有 Apple 风格的界面。

让我们来看一下作为 Puppet 开发者可用的各种 **集成开发环境（IDE）** 选项。

# Vim

Vim ([`www.vim.org`](http://www.vim.org)) 当然仍然是文本文件编辑的主流工具。它在 Unix 世界有着悠久的历史，是一个非常轻量级的命令行文本编辑器。Vim 可以说是最原始的文本编辑器之一。如果你有足够的内存并且有耐心学习各种键盘命令，它可以作为一个闪电般快速且高效的 IDE 使用。我的建议是从几个基本命令开始，每次使用 Vim 时都尽量学习一些新命令。

你可以优化你的 Vim，使其更适合编辑 Puppet 清单。假设你刚安装了一个全新的 Vim，并且已经安装了 Git，我们来看看如何操作。

进入你的主目录并使用以下命令克隆给定的仓库：

```
cd ~
git clone https://github.com/ricciocri/vimrc .vim
cd .vim
git pull && git submodule init && git submodule update && git submodule status
cd ~
ln -s .vim/.vimrc 
```

将仓库克隆到你的主目录的 `.vim` 目录中，将为你配置 Vim 设置。该仓库包含几个子模块，包含以下内容：

+   **Pathogen** ([`github.com/tpope/vim-pathogen`](https://github.com/tpope/vim-pathogen)) 是 Vim 大师 Tim Pope 的通用插件，它让你可以轻松管理 Vim 的 *runtimepath*，并将 Vim 插件和运行时文件安装到各自独立的目录中，避免文件冲突。

+   **Vim-puppet** ([`github.com/rodjek/vim-puppet`](https://github.com/rodjek/vim-puppet)) 是 Tim Sharpe 编写的原始 Vim 插件，使 Vim 更加适合 Puppet 开发。

+   **snipmate.vim** ([`github.com/msanders/snipmate.vim`](https://github.com/msanders/snipmate.vim)) 是一个 Vim 脚本，它实现了 TextMate 的一些代码片段功能。

+   **Syntastic** ([`github.com/vim-syntastic/syntastic`](https://github.com/vim-syntastic/syntastic)) 是一个语法检查插件，它通过外部语法检查器检查文件，并显示任何出现的错误。这可以通过命令行的 `pdk validate` 命令完成，也可以在文件保存时自动进行检查。

+   **Tabular** ([`github.com/godlygeek/tabular`](https://github.com/godlygeek/tabular)) 用于根据 *Puppet 风格指南*对你的箭头符号 (=>) 进行对齐，以便通过运行 `pdk validate` 命令（我们稍后会详细介绍 `pdk validate` 命令）。

+   **vim-fugitive** ([`github.com/tpope/vim-fugitive`](https://github.com/tpope/vim-fugitive)) 提供了 Vim 的深度 Git 集成功能。

我不能保证这将是一个完美符合你个人 Vim 风格的 Vim 设置，但它肯定会让你走上正确的道路，并且你将安装 Pathogen，这样你就可以进一步调整你的 Vim 设置，直到它符合你的需求。

你可能还想在 GitHub 上 fork 这个仓库，这样你可以保存所有的设置并与团队共享。

# TextMate

TextMate ([`macromates.com`](http://macromates.com)) 是一个仅适用于 macOS X 的编辑器，且有一个专门的 TextMate bundle 可用于编辑 Puppet 清单 ([`github.com/masterzen/puppet-textmate-bundle`](https://github.com/masterzen/puppet-textmate-bundle))。首先，安装 TextMate 和 Git（可以通过命令行开发者工具获取），然后按照以下命令设置 Puppet bundle：

```
$ mkdir ~/temp
$ cd ~/temp
$ git clone https://github.com/masterzen/puppet-textmate-bundle.git Puppet.tmbundle
$ mv ~/temp/Puppet.tmbundle ~/Library/Application\ Support/TextMate/Bundles/
$ rm -fr ~/temp
```

现在选择一个清单并用 TextMate 打开。在 TextMate 对话框中，选择 Puppet 并安装 Bundle，之后你就准备好开始了。

# Atom

这是我基于个人风格推荐的 IDE，使用我的 MacBook 作为主机操作系统。Atom ([`atom.io`](https://atom.io)) 是一个功能齐全的 IDE，被描述为 *一个适用于 21 世纪的可黑客化文本编辑器*，它包含了你期望的所有功能：跨平台、包（即插件）管理器、自动补全、文件浏览器、多个面板、查找和替换等。

GitHub 开发了 Atom，并且他们的目标是将一个完整的 IDE 的便利性与像 Vim 这样的经典复杂编辑器的深度可配置性结合起来。

Atom 上有成千上万的开源包，为其增加了新的功能，下面是我特别推荐用于 Puppet 开发的那些：

+   `language-puppet`（为 Puppet 文件添加语法高亮和片段）

+   `linter-puppet-lint`（为你的 Puppet 清单提供 linting 支持）

+   `aligner-puppet`（根据 Puppet 风格指南对胖箭头进行对齐）

+   `erb-snippets`（用于编写 Puppet ERB 模板的片段和快捷键）

+   `linter-js-yaml`（使用 JS-YAML 解析你的 YAML 文件）

+   `tree-view-git-status`（在树视图中显示文件的 Git 状态）

# Visual Studio

如果你是 Windows 和 .NET 世界的开发者，那么不妨看看 Visual Studio Code 插件中的 Puppet 语言支持 ([`marketplace.visualstudio.com/items?itemName=jpogran.puppet-vscode`](https://marketplace.visualstudio.com/items?itemName=jpogran.puppet-vscode))。

它包含了你在 Visual Studio IDE 中开发 Puppet 时所期望的所有功能：语法高亮、代码片段、文件验证、根据 Puppet 风格指南进行 linting、资源和参数的 IntelliSense、从 `puppet resource` 命令导入、节点图预览，现已集成 **Puppet Development Kit** (**PDK**)。

# 使用良好的模块和类结构

本节包含一系列关于良好的模块和类设计的建议。请记住，Puppet 开发本质上就像任何其他类型的软件开发一样，我们通过多年的软件开发经验，特别是在 O&O 软件公司，我们学到了一些模块化和类设计的原则，它们使我们的开发变得更好。我还认为，通向 *基础设施即代码* 的一部分是让我们的 Puppet 代码与其他应用程序代码一样设计良好、结构清晰且经过充分测试。

# 遵循类命名约定

在 Puppet 社区中，随着时间的推移，已经形成了一定的类命名约定，结构化类时考虑这些约定非常重要：

+   `init.pp`：`init.pp` 包含与模块同名的类，是模块的主要入口点。

+   `params.pp`：`params.pp` 模式（稍后将在本章中详细介绍）是一个优雅的小技巧，利用了 Puppet 的类继承行为。模块中的其他任何类都会继承 `params` 类，因此可以适当地设置它们的参数。

+   `install.pp`：与安装软件相关的资源应放在 `install` 类中。`install` 类必须命名为 `<modulename>::install`，并且必须位于 `install.pp` 文件中。

+   `config.pp`：与配置已安装软件相关的资源应放在 `config` 类中。`config` 类必须命名为 `<modulename>::config`，并且必须位于 `config.pp` 文件中。

+   `service.pp`：与管理软件服务相关的资源应放在 `service` 类中。服务类必须命名为 `<modulename>::service`，并且必须位于 `service.pp` 文件中。

对于配置为客户端/服务器风格的软件，请参见以下内容：

+   `<modulename>::client::install` 和 `<modulename>::server::install` 将是分别放置在 `client` 和 `server` 目录中的 `install.pp` 文件的类名

+   `<modulename>::client::config` 和 `<modulename>::server::install` 将是分别放置在 `client` 和 `server` 目录中的 `config.pp` 文件的类名

+   `<modulename>::client::service` 和 `<modulename>::server::service` 将是分别放置在 `client` 和 `server` 目录中的 `service.pp` 文件的类名

# 拥有一个单一的入口点来访问模块

`init.pp` 应该是模块的唯一入口点。这样，尤其是审查文档的人，以及查看 `init.pp` 代码的人，都能对模块的行为有一个完整的概览。

如果你有效地使用了封装并且使用了描述性的类名，那么仅通过查看 `init.pp` 你就可以很好地了解模块是如何实际管理软件的。

具有可配置参数的模块应该以单一方式并在此单一位置进行配置。唯一的例外是，例如，像 Apache 模块这样的模块，可能还需要配置一个或多个虚拟目录。

理想情况下，你可以通过一个简单的包含语句来使用你的模块，如下所示：

```
include mymodule
```

你也可以通过使用类声明来使用它，如下所示：

```
class {'mymodule':
  myparam => false,
}
```

*Apache 虚拟目录* 风格的配置方式将是使用你的新模块的第三种方式：

```
mymodule::mydefine {‘define1':
  myotherparam => false,
}
```

与此建议相反的反模式是，除了 `init.pp` 和你的定义类型外，还存在其他类，且这些类的参数期望被设置。

# 使用高内聚和低耦合原则

尽可能地，Puppet 模块应该由具有单一责任的类组成。在软件工程中，我们称之为高内聚性。软件工程中的内聚性是指一个模块的各个元素在多大程度上属于同一类别。尽量确保每个类只有一个责任，避免在类中随意混合不相关的功能。

# 使用封装原则

尽可能地，这些类应使用封装来隐藏实现细节，避免用户了解具体的资源名称。例如，你模块的用户无需了解个别资源的名称。在软件工程中，我们称之为封装。例如，在 `config` 类中，我们可以使用多个资源，但用户不需要知道所有细节。相反，他们只需要知道应该使用 `config` 类来正确配置软件。

将类包含在其他类中非常有用，尤其是在大型模块中，你希望提高代码的可读性。你可以将功能块移动到独立的文件中，然后使用 `contain` 关键字来引用这些分离的功能块。

请参阅 [`puppet.com/docs/puppet/5.3/lang_containment.html`](https://puppet.com/docs/puppet/5.3/lang_containment.html) 网站，获取关于 `contain` 关键字的提醒。

# 提供合理且经过深思熟虑的参数默认值

如果绝大多数使用你模块的人都将使用某个特定参数设置，那么当然有必要为该参数设置默认值。

仔细考虑你的模块是如何使用的，并站在非专家用户的角度来审视自己的模块。

按照合理的顺序展示可用的模块参数，将常用的设置放在最前面，而不是随意的顺序（如字母顺序）。

# 强烈类型化你的模块变量

在 Puppet 的早期版本中（在版本 4 发布之前的版本），我们会创建没有定义数据类型的 `class` 参数，然后，如果我们非常友好的话，我们会使用 `stdlib validate_<datatype>` 函数来检查这些变量的适当值：

```
class vhost (
  $servername,
  $serveraliases,
  $port
)
{ ...
```

Puppet 4 和 5 内建了定义参数化类接受的数据类型的方法。请参阅以下示例：

```
class vhost (
  String  $servername,
  Array   $serveraliases,
  Integer $port
)
{ ...
```

# 使用新的 Puppet 开发工具包命令

一些可以提高 Puppet 开发质量的功能，例如 `puppet-lint`、`puppet-rspec` 和像 `puppet module create` 这样的命令，已经存在了一段时间，但之前，你必须自己去发现这些工具，安装它们，并弄清楚如何有效地使用它们。

Puppet 在 2017 年 8 月决定将这些工具整合到客户端，并通过新的 Puppet 开发工具包版本 1.0 使其变得易于使用。我清楚地记得，`puppet-rspec` 总是需要一些时间来设置并使其正常工作。现在一切都变得非常简单。

让我们快速浏览一下使用新版本 PDK 1.0 的模块开发流程。

+   **创建模块框架和元数据**：`pdk new module` 命令的运行方式与旧的 `puppet module create` 命令相同，具体如下：

```
$ pdk new module zope –-skip-interview
```

+   **创建** `init.pp`：现在有一套创建模块内部清单的命令，具体如下：

    +   `pdk new class`（[`puppet.com/docs/pdk/1.0/pdk_reference.html#pdk-new-class-command`](https://puppet.com/docs/pdk/1.0/pdk_reference.html#pdk-new-class-command)）

    +   `pdk new defined_type`（[`puppet.com/docs/pdk/1.0/pdk_reference.html#pdk-new-definedtype-command`](https://puppet.com/docs/pdk/1.0/pdk_reference.html#pdk-new-definedtype-command)）

    +   `pdk new task`（[`puppet.com/docs/pdk/1.0/pdk_reference.html#pdk-new-task-command`](https://puppet.com/docs/pdk/1.0/pdk_reference.html#pdk-new-task-command)）——有关新 Puppet 任务功能的更多详细信息，请参见第六章，*工作流程*。

所以，只需使用模块的名称来创建 `init.pp`：

```
$ pdk new class zope
```

这些命令现在可以完全避免在文本编辑器中使用片段来创建注释、声明和其他样板代码。

+   **创建更多类**：使用相同的命令创建任何更多的类。请参见以下示例：

```
$ pdk new class params
```

# 验证您的模块

在工作过程中，您可以使用新的 `pdk validate` 命令（[`puppet.com/docs/pdk/1.0/pdk_reference.html#pdk-validate-command`](https://puppet.com/docs/pdk/1.0/pdk_reference.html#pdk-validate-command)）来帮助检查模块是否能编译、是否符合 Puppet 风格指南，以及是否拥有有效的元数据：

```
$ pdk validate
```

# 对您的模块进行单元测试

提升模块质量的最重要的一件事就是对它们进行测试！测试确实是任何软件开发领域中软件质量保证的一个重要方面。在敏捷开发社区，我们已经对自动化测试坚持了超过 10 年！

Puppet RSpec（[`rspec-puppet.com/tutorial`](http://rspec-puppet.com/tutorial)）已经让 Puppet 社区能够对其模块进行单元测试有一段时间了，但现在使用新版本 PDK 1.0 更加容易，因为所有设置都已准备好，您只需添加测试代码并运行测试。

从 Puppet 的角度来看，单元测试意味着 *检查编译器的输出*。编译后的关系资源目录中是否包含资源？它们的顺序是否符合预期，具体取决于传递的参数和/或存在的事实？

当你开始在 Puppet-RSpec 中编写测试时，最初看似你只是在用另一种类似 Ruby 的语言重写 Puppet 清单。然而，实际上远不止如此。如果模块的功能具有一定复杂性，例如测试 Puppet 模板生成的动态内容，支持多个操作系统，或根据传入的参数执行不同的操作，那么这些测试实际上形成了一张安全网，在编辑或添加新功能时可以防止回归，或者在重构或升级到新版本的 Puppet 时提供保护。

让我们继续前两个部分，并使用开发工具包对我们的模块进行单元测试。每当你使用`pdk new class`命令生成一个类时，PDK 会创建一个对应的单元测试文件。这个文件位于模块的`/spec/classes`文件夹中，已经包含了编写单元测试的模板（参见 [`rspec-puppet.com/tutorial`](http://rspec-puppet.com/tutorial)）。你可以使用以下命令运行测试：

```
$ pdk test unit
```

# 注意代码异味

在你的 Puppet 代码库逐渐增大时，要特别注意代码异味！以下链接是一个研究项目，描述了一些 Puppet *代码异味*，这是一种极限编程（XP）术语，指的是代码问题——通常意味着设计或实现不佳：[`www.tusharma.in/wp-content/uploads/2016/03/ConfigurationSmells_preprint.pdf`](http://www.tusharma.in/wp-content/uploads/2016/03/ConfigurationSmells_preprint.pdf)

让我们快速回顾一下如何使用前面研究项目中使用的基于 Python 的`Puppeteer`工具：

1.  确保你已经安装了最新的 Java SDK。

1.  进入你的`workspace`目录`~/workspace`，并克隆以下 Git 仓库：

```
$ git clone https://github.com/tushartushar/Puppeteer
$ cd Puppeteer
```

1.  下载 PMD 工具（[`github.com/pmd/pmd`](https://github.com/pmd/pmd)），并在 shell 脚本中更新路径。PMD 是一个可扩展的静态代码分析器，内置 **复制粘贴检测器**（**CPD**）。

1.  更新所有 Puppet 仓库存放的文件夹路径。

1.  执行`cpdRunner.sh` shell 脚本，使用 PMD-CPD 工具进行克隆检测。

1.  更新`SmellDetector/Constants.py`中的`REPO_ROOT`常量，它表示存放所有 Puppet 仓库的文件夹路径。

1.  执行`Puppeteer.py`。

1.  使用`puppet-lint`分析 Puppet 仓库（可选）。

1.  在设置好仓库根目录后，执行`puppet-lintRunner.py`。

1.  在`Puppet-lint_aggregator/PLConstants.py`中设置仓库根目录。

1.  执行`PuppetLintRules.py`，它将生成所有分析项目的汇总总结。

# 处理死代码

另一个常见问题是，随着 Puppet 代码库的增长，代码中可能存在未使用的代码。但幸运的是，我们可以使用一个工具来解决这个问题。

`puppet-ghostbuster` 实质上将实际使用的内容（存储在 PuppetDB 中）与您认为正在使用的内容（在您的代码库目录中）进行比较。这给了您一个机会，删除和清理那些真正未使用的内容。从软件可维护性的角度来看，这是非常有益的。较小的代码库显然更便宜、更容易维护！

让我们快速了解如何使用这个 Ruby gem。

在您的环境变量中进行以下设置：

+   `HIERA_YAML_PATH`：`hiera.yaml` 文件的位置。默认值为`/etc/puppetlabs/code/hiera.yaml`。

+   `PUPPETDB_URL`：PuppetDB 的 URL。默认值为 `http://puppetdb:8080`。

+   `PUPPETDB_CACERT_FILE`：您站点的 CA 证书。

+   `PUPPETDB_CERT_FILE`：由您站点的 Puppet CA 签名的 SSL 证书。

+   `PUPPETDB_KEY_FILE`：该证书的私钥。

按如下方式运行命令：

```
$ find . -type f -exec puppet-lint --only-checks ghostbuster_classes,ghostbuster_defines,ghostbuster_facts,ghostbuster_files,ghostbuster_functions,ghostbuster_hiera_files,ghostbuster_templates,ghostbuster_types {} \+
```

你可以向逗号分隔的项中添加或删除内容，以检查未使用的类、定义的类型、事实、文件、函数、Hiera 文件、模板和类型。

# 使用 Puppet Forge

也许不言而喻，当你编写 Puppet 模块时，完全没有必要重新发明轮子。在 Puppet Forge 上花几分钟时间（[`forge.puppet.com`](https://forge.puppet.com/)）真的可以为你节省几天的编辑工作。到目前为止，Forge 上已经有超过 5000 个模块，因此充分利用 Puppet 社区所做的所有辛勤工作是非常有意义的。首先在 Forge 上搜索该软件，很可能已经存在类似的模块。

根据我的经验，我发现通常总有一些*几乎*能够完成任务的东西。可能有一个模块（通常是一个不受支持且未经批准的模块），也许它能够执行你所需要的软件管理，但它只适用于 Ubuntu，而你正在使用的是 Red Hat。通常来说，更好的做法是对该模块进行分叉，不管它的状态如何，然后在此基础上进行修改，而不是从头开始。

# 与社区合作

我最好的描述这个最佳实践的方法是用反模式作为例子。

我曾遇到一个 Puppet 开发者，他会从头开始编写模块，然后将 Forge 模块中的代码行复制并粘贴到新模块中。从那时起，该模块完全存在于社区之外！它甚至不是一个分叉，因此将时间积累下来的社区更改集成进来变得非常麻烦。你必须挑选这些更改，以将功能集成到自己的模块中，而且你可能仍然会遇到回归问题。通常，最佳实践是至少要分叉 Forge 模块！这样你就能获得 Git 历史记录，通常这些记录包含了在制作该模块过程中所付出的思考和努力。

你看，如果你曾经读过那本伟大的书《教堂与集市：一位偶然革命者的 Linux 与开源思考》（[`www.amazon.com/Cathedral-Bazaar-Musings-Accidental-Revolutionary/dp/0596001088`](https://www.amazon.com/Cathedral-Bazaar-Musings-Accidental-Revolutionary/dp/0596001088)），你就会明白，Linux 导向的软件开发哲学通过一个*集市*，协作的工作方式，超越了将开发工作分割成一个教堂式的独立工作方式。好吧，这是我对这个开发者工作风格的看法。他是在教堂式工作，而不是集市式工作。实际上，你是在做出决定，把你的教堂团队与集市中的大量人员对抗，在我看来，这在互联网时代，作为项目管理并没有给你带来竞争优势，是不明智的。

有时，Forge 上的模块会有些过时。如果模块的元数据过时了，你可以通过使用 PDK 的`new module`命令 ([`puppet.com/docs/pdk/1.0/pdk_generating_modules.html#create-a-module-with-pdk`](https://puppet.com/docs/pdk/1.0/pdk_generating_modules.html)) 来重新生成，并提交新的元数据。

当然，作为一个优秀的 Puppet 社区成员，做出的更好实践是将你所做的更改提交为 pull 请求，并为社区的工作做出贡献。

# 撰写优秀文档

另一个重要的建议是：简单地写出优秀的文档。作为开发者，我认为没有什么比需要深入代码才能理解一个模块如何工作更糟糕了；这就像必须掀开汽车引擎盖来理解如何驾驶汽车一样！

提高用英语表达技术思想的能力！我真的认为这是每个优秀开发者必须掌握的技能。

# 获取一个 Markdown 编辑器

Puppet 模块使用 markdown 格式来编写文档。因此，使用独立的 Markdown 编辑器或一些 IDE 插件是有意义的，这样你就可以恰当地创建高质量的文档。接下来是我们在本章中考虑过的代码 IDE 的选择，随之而来的对应 markdown 插件如下。

# Vim

如果你是 vim 粉丝，可以使用 vim-instant-markdown 插件 ([`github.com/suan/vim-instant-markdown`](https://github.com/suan/vim-instant-markdown))。

# TextMate

如果你喜欢 TextMate 的苹果风格，可以使用 TextMate markdown 包 ([`github.com/textmate/markdown.tmbundle`](https://github.com/textmate/markdown.tmbundle))。

# Atom

如果像我一样，你喜欢使用 Atom，你可以使用 Markdown Preview Plus 包 ([`atom.io/packages/markdown-preview-plus`](https://atom.io/packages/markdown-preview-plus))。

# Visual Studio

如果你是 Windows 和 .NET 世界的开发者，那么不妨试试 Markdown 编辑器扩展（[`marketplace.visualstudio.com/items?itemName=MadsKristensen.MarkdownEditor`](https://marketplace.visualstudio.com/items?itemName=MadsKristensen.MarkdownEditor)）。

# 独立的 Markdown 编辑器

如果你更喜欢使用独立的 Markdown 编辑器，我个人推荐 macOS X 上的 MacDown。以下是我为各种操作系统列出的独立 Markdown 编辑器的（非常）简短列表。

# Remarkable

如果你使用 Linux，Remarkable 可能是最好的独立编辑器。它也支持 Windows。它的一些功能包括实时预览、导出为 PDF 和 HTML、GitHub Markdown、定制 CSS、语法高亮和快捷键。

# MacDown

如果你更喜欢使用独立的 Markdown 编辑器，我可以推荐 MacDown for macOS X，它是免费的（开源）。它深受 Mou 启发，并且专为 web 开发者设计。它具有可配置的语法高亮、实时预览和自动补全功能。如果你正在寻找一个精简、快速、可配置的独立 Markdown 编辑器，这可能是适合你的选择。

# 添加模块依赖

编辑模块的 `metadata.json` 文件以添加模块依赖。请参见以下示例：

```
"dependencies": [
    { "name":" stankevich/python",
       "version_requirement":">= 1.18.x"
     }
  ]
```

`name` 键是需求的名称，即 `"pe"` 或 `"puppet"`。`version_requirement` 键是 semver（[`semver.org`](http://semver.org)）值或范围。参见以下示例：

+   `1.18.0`

+   `1.18.x`

+   `>= 1.18.x`

+   `>=1.18.x <2.x.x`

这些都是有效的 `version_requirement` 值。

之后，请使用新的 PDK 命令检查 `metadata.json` 文件的有效性，命令如下：

```
$ pdk validate metadata
```

添加模块依赖的好处是，当你运行 `puppet module download` 命令时，Puppet 会相应地下载所有模块依赖。

# 为你的模块添加兼容性数据

本节介绍了如何为适用于你所使用版本的 Puppet 或 Puppet Enterprise 及操作系统的模块添加兼容性数据。首先，编辑模块的 `metadata.json` 文件以添加兼容性数据。

# 操作系统支持

在模块的 `metadata.json` 中表达该模块支持的操作系统，示例如下：

```
"operatingsystem_support": [
       { "operatingsystem": "RedHat", },
       { "operatingsystem": "Ubuntu", },
]
```

预期会有 Facter facts `operatingsystem` 和 `operatingsystemrelease`。以下是一个更完整的示例：

```
"operatingsystem_support": [
       {
           "operatingsystem":"RedHat",
           "operatingsystemrelease":[ "5.0", "6.0" ]
       },
       {
           "operatingsystem": "Ubuntu",
           "operatingsystemrelease": [
               "12.04",
               "10.04"
           ]
       }
   ]
```

之后，请使用新的 `pdk` 命令检查 `metadata.json` 文件的有效性：

```
$ pdk validate metadata
```

# Puppet 和 PE 版本支持

`metadata.json` 文件中的 `requirements` 键是一个外部需求的列表，格式如下：

```
"requirements": [ {“name”: “pe”, “version_requirement”: “5.x”}]
```

`name` 是需求的名称，例如 `"pe"` 或 `"puppet"`。`version_requirement` 可以是一个 semver（[`semver.org`](http://semver.org)）版本范围，类似于依赖项。

同样，你可以使用新的 PDK 命令检查 `metadata.json` 文件的有效性，命令如下：

```
$ pdk validate metadata
```

# 使用新的 Hiera 5 模块级数据

在模块编写过程中，我们已经使用了相当长一段时间的`params.pp`模式。模块中的一个类，按照惯例被称为`<MODULENAME>::params`，为其他任何类设置变量：

```
class zope::params {
  $autoupdate = false,
  $default_service_name = 'ntpd',

  case $facts['os']['family'] {
    'AIX': {
      $service_name = 'xntpd'
    }
    'Debian': {
      $service_name = 'ntp'
    }
    'RedHat': {
      $service_name = $default_service_name
    }
  }
}
```

如你所见，我们在这里使用了一些条件逻辑，这取决于`os::family`事实，从而可以适当地设置`service_name`变量。我们还暴露了`autoupdate`变量，并给它设置了默认值。

这个`params.pp`模式是一个优雅的小技巧，它利用了 Puppet 特有的类继承行为（在 Puppet 中通常不建议使用继承）。然后，模块中的任何其他类都继承自`params`类，以适当地设置它们的参数，如下例所示：

```
class zope (
  $autoupdate   = $zope::params::autoupdate,
  $service_name = $zope::params::service_name,
) inherits zope::params {
 ...
}
```

自从 Hiera 5 发布以来，我们能够大大简化模块的复杂性。通过使用基于 Hiera 的默认值，我们可以简化模块的主类，它们不再需要继承自`params.pp`。此外，你也不再需要在参数声明中显式地使用`=`运算符设置默认值。

让我们来看一下使用 Hiera 5 与`params.pp`模式等效的配置。

首先，为了使用这个新功能，需要在模块的`metadata.json`文件中将`data_provider`键设置为`heira`值：

```
...
"data_provider": "hiera",
...
```

接下来，我们需要将`hiera.yaml`文件添加到模块的根目录：

```
---
version: 5
defaults:
  datadir: data
  data_hash: yaml_data
hierarchy:
  - name: "OS family"
    path: "os/%{facts.os.family}.yaml"

  - name: "common"
    path: "common.yaml"
```

然后，我们可以将三个文件添加到`/data`目录中（请注意`hiera.yaml`文件中的`datadir`设置）。这三者中的第一个文件用于设置 AIX 的`service_name`变量：

```
# zope/data/os/AIX.yaml
---
zope::service_name: xntpd
```

第二个文件用于设置 Debian 的`service_name`变量：

```
# zope/data/os/Debian.yaml
zope::service_name: ntp
```

最后是公共文件，如果 Hiera 在查找`service_name`设置或`autoupdate`值时没有找到对应的操作系统文件，它会退回到此文件来查找其值：

```
# ntp/data/common.yaml
---
ntp::autoupdate: false
ntp::service_name: ntpd
```

我们将在第四章中更加详细地探讨 Hiera 5，*Hiera 5*。

# 总结

在本章中，我们涵盖了很多内容，我介绍了一些最佳实践，你可以利用它们来编写更高质量的组件模块。

在下一章中，我们仍然会讨论 Puppet DSL 的开发，并将注意力转向两个特殊模块：角色和配置文件，它们可以帮助我们构建可重用、可配置、可重构的全站配置代码。

# 第七章：使用 Chef 和 Puppet 测试和编写更好的基础设施代码

在本章中，我们将介绍以下配方：

+   使用 Foodcritic 进行 Chef 代码的 Lint 检查，使用 puppet-lint 进行 Puppet 代码的 Lint 检查

+   使用 ChefSpec 和 rspec-puppet 进行单元测试

+   使用 Test Kitchen 对 Chef 进行基础设施测试，并使用 Beaker 对 Puppet 进行基础设施测试

+   使用 ServerSpec 进行集成测试

# 介绍

在开发领域，良好的软件测试实践广泛应用，例如单元测试和集成测试。Linters（代码检查工具）也是软件开发人员每天使用的工具，几乎适用于大多数语言。幸运的是，这些技术通过我们使用的工具进入了基础设施领域；现在，基础设施基本上就是代码，它可以被分析、测试和报告！结合持续集成（CI）系统，编写经过充分测试的基础设施代码在不同层级上能够极大地帮助实现高质量的可持续代码，并防止出现本来会在之后破坏事物的意外回归。

在本章中，您将发现各种技术，通过使用 linter 和样式工具编写更简洁的代码，以确保我们的代码遵循高标准。您将学习如何对基础设施代码进行单元测试，比如 Chef 资源，并实现尽可能高的代码覆盖率，以确保代码中没有错误，也不会被无意修改。然后，我们将配置测试环境 Test Kitchen，它通过 Vagrant（或其他系统）利用虚拟机（VM）来执行测试套件。这将是我们的基础，以便编写集成测试，确保我们通过多个 cookbook 和代码来源实现预期的目标，并在真实系统上完成任务。

这些工具和技术对编写最佳基础设施代码至关重要，它们不仅强大，而且使用起来非常有趣！

所有配方都基于 Chef。然而，在可能的情况下，我们会尽量展示如何在 Puppet 中类似地工作，它是 Chef 的直接替代品。

# 使用 Foodcritic 进行 Chef 代码的 Lint 检查，使用 puppet-lint 进行 Puppet 代码的 Lint 检查

由于我们主要使用 Ruby 进行编码，我们可以在 Ruby 领域使用像 Rubocop 这样的常用 linter 工具。然而，Rubocop 默认是针对软件开发的，并没有特别针对 Chef cookbook 的开发进行优化。因此，Chef 开发了自己的 Rubocop 版本，命名为 Cookstyle。同时，Foodcritic 工具与规则结合，检查我们的代码是否符合社区公认的良好实践。我们将一起深入了解这些工具，最终让代码变得更加简洁、优雅。

## 准备工作

要执行这个配方，您需要以下工具：

+   在工作站上安装 Chef DK

+   远程主机上的 Chef 客户端配置

+   来自第六章, *使用 Chef 和 Puppet 管理服务器的基础*，或者任何自定义的 Chef 代码。

## 如何操作…

我们将研究并遵循两个互补工具——Cookstyle 和 Foodcritic 的建议。它们都能提供关于代码质量和可移植性的宝贵且互补的建议。让我们从最快最简单的——Cookstyle 开始。

### Cookstyle

导航到 cookbook 根目录并输入以下命令：

```
$ cookstyle

```

这将输出所有关于清理代码的建议。

要获取更多关于建议的信息，包括带有更多信息的网址，请使用以下选项：

```
$ cookstyle -DES

```

如果你对建议感到满意，并希望自动将所有建议直接应用到代码中，请使用以下开关：

```
$ cookstyle -a

```

如果我们对 第六章 *使用 Chef 和 Puppet 管理服务器的基础知识*，我们编写的 Chef cookbook 应用 `cookstyle`，我们将得到两个不错的建议：

+   当不需要插值时，使用单引号表示字符串

+   Ruby 1.9 版本中哈希的语法

由于这些都是有价值且推荐的改动，我们将在所有相关的 `metadata.rb` 文件中更新我们的 cookbook 版本，应用这些建议，并将新的小版本上传到 Chef 服务器。

### Foodcritic

Foodcritic 的检查远远超过 Cookstyle，除了检查 Chef 代码中的不兼容、非幂等、重复或废弃的代码外，还会检查缺失的模板、文件、依赖关系或变量。所有规则都在 Foodcritic 网站上描述，网址是 [`www.foodcritic.io`](http://www.foodcritic.io)，同时提供示例和解释。

执行 `foodcritic`，进入 Chef 仓库并输入以下命令：

```
$ foodcritic <cookbook path>

```

例如，对于测试我们之前的 `mysite` cookbook（排除自动生成的 `test` 目录，因为它本身不是一个 cookbook），我们输入以下命令：

```
$ foodcritic --exclude test cookbooks/mysite
FC003: Check whether you are running with chef server before using server-specific features: cookbooks/mysite/recipes/htaccess.rb:7
FC033: Missing template: cookbooks/mysite/recipes/htaccess.rb:9
FC033: Missing template: cookbooks/mysite/recipes/htaccess.rb:19
FC064: Ensure issues_url is set in metadata: cookbooks/mysite/metadata.rb:1
FC065: Ensure source_url is set in metadata: cookbooks/mysite/metadata.rb:1

```

有趣！让我们从 FC003 开始 ([`www.foodcritic.io/#FC003`](http://www.foodcritic.io/#FC003))。我们的代码确实无法与其他 Chef 模式（如 chef-solo）一起使用，因为我们在代码中直接使用了 Chef 搜索，而 chef-solo 无法与 Chef 服务器交互。这里有两种选择：要么我们不在乎 chef-solo 的兼容性并排除此规则，或者我们关心并相应地修改代码。

要排除 `FC003` 规则，请使用 `-t` 选项：

```
$ foodcritic -t ~FC003 --exclude test cookbooks/mysite/
FC033: Missing template: cookbooks/mysite/recipes/htaccess.rb:9
FC033: Missing template: cookbooks/mysite/recipes/htaccess.rb:19
FC064: Ensure issues_url is set in metadata: cookbooks/mysite/metadata.rb:1
FC065: Ensure source_url is set in metadata: cookbooks/mysite/metadata.rb:1

```

或者，如果我们关心 chef-solo 的兼容性，让我们按照 FC003 规则提出的建议修改代码。更新 `mysite` cookbook 中的 `mysite/metadata.rb` 文件，并编辑 `mysite/recipes/htaccess.rb` 文件中的 `users` 搜索，加入对是否运行 chef-solo 的判断：

```
if Chef::Config[:solo]
  Chef::Log.warn('This recipe uses search. Chef Solo does not support search.')
else
  users = search(:webusers, '*:*')
end
```

使用 Berkshelf 上传 cookbook 的新版本：

```
$ berks upload

```

重新运行 `foodcritic`，警告信息消失了：

```
$ foodcritic --exclude test cookbooks/mysite

```

让我们继续检查建议。`FC033`（[`www.foodcritic.io/#FC033`](http://www.foodcritic.io/#FC033)）是关于缺少模板的。然而，我们的模板是通过`chef`工作流命令放置在`mysite/templates`目录下的。这就是为什么理解建议仅仅是“建议”如此重要的原因。Foodcritic 团队建议在 FC033 中强制要求在`templates/default`目录中存在默认模板。最终由你和你的团队决定是否遵循 Chef 或 Foodcritic 的推荐行为。我们决定遵循 Chef 并忽略此警告：

```
$ foodcritic -t ~FC033 --exclude test cookbooks/mysite/
FC064: Ensure issues_url is set in metadata: cookbooks/mysite/metadata.rb:1
FC065: Ensure source_url is set in metadata: cookbooks/mysite/metadata.rb:1

```

前两个警告（`FC064`和`FC065`）仅与 Chef 超市发布的食谱相关，而这不适用于我们。让我们通过`-t ~supermarket`开关全局排除所有与超市相关的警告：

```
$ foodcritic -t ~FC033 -t ~supermarket --exclude test cookbooks/mysite/

```

现在没有更多的警告了；我们的食谱遵循了 Chef 和 Foodcritic 社区的最佳建议！

强烈建议将这些测试添加到自动化测试过程中。假设我们使用一个全局的`Makefile`来完成这一任务。请在 Chef 仓库的根目录创建它：

```
$ cat Makefile
tests:
 foodcritic -t ~FC033 -t ~supermarket --exclude test cookbooks/mysite

```

现在，你或某个 CI 系统可以自动检查代码的质量或质量回归。

## 还有更多…

使用 Puppet 时，puppet-lint 将帮助我们清理代码。我们需要使用以下命令安装 puppet-lint：

```
$ sudo puppet resource package puppet-lint provider=puppet_gem 

```

如果你已经熟悉 Puppet，你可能会看到我们在上一章中编写的代码不符合标准。让我们基于最新的 Apache 模块食谱，发现一些 puppet-lint 检测到的问题：

```
$ puppet-lint modules/apache/manifests/init.pp
WARNING: class not documented on line 1
ERROR: two-space soft tabs not used on line 3
...
$ puppet-lint modules/apache/manifests/vhost.pp
WARNING: defined type not documented on line 1
WARNING: variable not enclosed in {} on line 6
...
ERROR: trailing whitespace found on line 11
...
ERROR: two-space soft tabs not used on line 2
...
WARNING: indentation of => is not properly aligned
 (expected in column 34, but found it in column 31) on line 12
...
$ puppet-lint modules/apache/manifests/htpasswd.pp
WARNING: defined type not documented on line 1
WARNING: string containing only a variable on line 6
WARNING: variable not enclosed in {} on line 6
ERROR: two-space soft tabs not used on line 2
...
$ puppet-lint modules/apache/manifests/htaccess.pp
WARNING: defined type not documented on line 1
WARNING: variable not enclosed in {} on line 6
ERROR: two-space soft tabs not used on line 2

```

我们可以看到两类错误：

+   Puppet 编码风格警告/错误

+   缺少文档

让我们尝试修复它们！

### Puppet 编码风格

对于我们这里关注的问题，基本规则是：

+   缩进需要使用*两个空格*字符。

+   无尾随空格

+   在字符串插值中，变量应被大括号括起来；例如，`"$docroot/.htaccess"`是错误的，必须是`"${docroot}/.htaccess"`

### 文档

文档应使用 Markdown 进行编写。如果你从未听说过 Markdown，它是一种用于格式化纯文本文档的语言，目的是将其导出为 HTML。使用 Markdown，你可以轻松地添加标题、链接、项目符号和字体效果。可以在[`www.markdowntutorial.com`](http://www.markdowntutorial.com)找到一个简短且互动的教程。

一个带有*实时预览*模式的 Markdown 编辑器可以在[`stackedit.io`](https://stackedit.io)找到。

我们需要在模块的顶层目录创建一个`README.md`文件。该文件应包含简短的描述和一些使用示例。为了提高可读性，我们将重点关注安装和虚拟主机的定义。完整文档可以在代码包中找到。以下是`modules/apache/README.md`的摘录：

```
# Apache module

## Table of Contents

1\. Description
1\. Usage
    * Apache installation
    * Defining a vhost

## Description

Sample module for Apache on Ubuntu systems

## Usage

### installation

To install apache2:

```

class {

'apache':;

}

```

### vhost

To create a vhost:

```

apache::vhost{'mysite':

website    => 'www.example.com',

docroot    => '/var/www/example',

}

```
```

我们还需要记录所有语句及其参数，在每个清单顶部的注释中使用 `@param` 标签。遵循 puppet-lint 推荐的新代码如下：

+   对于 `modules/apache/manifests/init.pp`：

    ```
    # See README
    class apache {
      package {'apache2':
        ensure => present,
      }

      service {'apache2':
        ensure => running,
        enable => true
      }

      file {'/etc/apache2/sites-enabled/000-default.conf':
        ensure => absent,
        notify => Service['apache2'],
      }
    }
    ```

+   对于 `modules/apache/manifests/htpasswd.pp`：

    ```
    # @param filepath Path of the htpasswd database
    # @param users Array of hash containing users
    # See README
    define apache::htpasswd (
      $filepath,
      $users
    ) {
      file { $filepath:
        ensure  => present,
        owner   => 'root',
        group   => 'root',
        mode    => '0644',
        content => template('apache/htpasswd.erb'),
      }
    }
    ```

+   对于 `modules/apache/manifests/htaccess.pp`：

    ```
    # @param filepath Path of the htpasswd database
    # @param docroot DocumentRoot where the .htaccess should be generated
    # See README
    define apache::htaccess (
      $filepath,
      $docroot
    ) {
      file { "${docroot}/.htaccess":
        ensure  => present,
        owner   => 'root',
        group   => 'root',
        mode    => '0644',
        content => template('apache/htaccess.erb'),
      }
    }
    ```

+   对于 `modules/apache/manifests/vhost.pp`：

    ```
    # @param website Site name
    # @param docroot DocumentRoot
    # See README
    define apache::vhost (
      $website,
      $docroot
    ) {
      file { "/etc/apache2/sites-available/${website}.conf":
        ensure  => present,
        owner   => 'root',
        group   => 'root',
        mode    => '0644',
        content => epp('apache/vhost.epp', {
          'website'    => $website,
          'docroot' => $docroot}
        ),
      }

      file { "/etc/apache2/sites-enabled/${website}.conf":
        ensure  => link,
        target  => "/etc/apache2/sites-available/${website}.conf",
        require => File["/etc/apache2/sites-available/${website}.conf"],
        notify  => Service['apache2'],
      }
    }
    ```

文档可以自动生成一组 HTML 页面。为此，我们需要安装 `yard` 和 `puppet-strings` 包：

```
$ sudo puppet resource package yard provider=puppet_gem

$ sudo puppet resource package puppet-strings provider=puppet_gem

```

现在，从我们模块的顶级目录，可以生成文档：

```
$ puppet strings
Files:                    4
Modules:                  0 (    0 undocumented)
Classes:                  0 (    0 undocumented)
Constants:                0 (    0 undocumented)
Attributes:               0 (    0 undocumented)
Methods:                  0 (    0 undocumented)
Puppet Classes:           1 (    0 undocumented)
Puppet Defined Types:     3 (    0 undocumented)
Puppet Types:             0 (    0 undocumented)
Puppet Providers:         0 (    0 undocumented)
Puppet Functions:         0 (    0 undocumented)
 100.00% documented
$ ls -1 doc
_index.html
css/
file.README.html
frames.html
index.html
js/
puppet_class_list.html
puppet_classes/
puppet_defined_type_list.html
puppet_defined_types/
top-level-namespace.html

```

文档位于 `doc` 目录下。我们现在可以通过在浏览器中打开 `index.html` 来查看它。

## 另见

Puppet 语言风格：

+   Cookstyle: [`github.com/chef/cookstyle`](https://github.com/chef/cookstyle)

+   Foodcritic: [`www.foodcritic.io/`](http://www.foodcritic.io/)

# 使用 ChefSpec 和 rspec-puppet 进行单元测试

ChefSpec 是由伟大的 Seth Vargo（Opscode Chef，Hashicorp）编写的 Chef 食谱 RSpec 单元测试框架。ChefSpec 帮助创建快速反馈循环，在本地模拟 Chef 运行（独立或服务器模式），并为每个使用的资源生成代码覆盖率报告。它与 Berkshelf 集成得非常好，因此在测试过程中可以轻松处理食谱依赖项。

我们将为 第六章中创建的食谱编写单元测试，*使用 Chef 和 Puppet 管理服务器的基础*，涵盖最常见的测试，例如收敛问题、软件包安装、服务状态检查、文件和模板创建、访问权限、配方包含、模拟数据包搜索，甚至是拦截预期错误。这些测试非常通用，我们将能够在所有未来的配方中重复使用它们，并开始进行更多的测试。

## 准备工作

要逐步执行此配方，你将需要以下内容：

+   工作站上的有效 Chef 安装

+   远程主机上的有效 Chef 客户端配置

+   来自第六章的 Chef 代码，*使用 Chef 和 Puppet 管理服务器的基础*，或任何自定义的 Chef 代码

## 如何实现……

ChefSpec 单元测试位于每个 Chef 食谱的 `spec/unit/recipes` 文件夹中。根据我们创建食谱的方式，这个文件夹可能已经存在。

为了说明这一点，我们从 第六章中的 `apache` 食谱开始，*使用 Chef 和 Puppet 管理服务器的基础*，但任何类似的自定义食谱同样适用。

如果 `spec/unit/recipes` 目录不存在，请通过执行以下命令来创建它：

```
$ mkdir -p spec/unit/recipes

```

在 `spec/unit` 中的 `recipes` 目录下可以找到 ChefSpec 单元测试，通常：

```
$ tree spec/
spec/
├── spec_helper.rb
└── unit
 └── recipes
 ├── default_spec.rb
 └── virtualhost_spec.rb

```

每个配方都会有其对应的 ChefSpec 文件。在这种情况下，我们的简单食谱包含两个配方，因此我们会有两个测试规范。

### Spec 辅助工具

对于所有相关的食谱测试，拥有一组通用的要求是很有帮助的。默认情况下，它的文件名为`spec_helper.rb`，位于`spec/unit`目录的根目录下。我们建议至少包括以下三项要求：

+   ChefSpec 本身

+   用于依赖管理的 Berkshelf 插件

+   立即开始代码覆盖率

这是我们的示例`spec_helper.rb`文件：

```
require 'chefspec'
require 'chefspec/berkshelf'
ChefSpec::Coverage.start!
```

### 测试成功的 Chef 运行上下文

我们现在将单元测试默认的 apache 食谱配方。我们的第一步是引入在`default_spec.rb`文件中创建的 helper。这将在我们所有的未来测试中被引入：

```
require 'spec_helper'
```

所有单元测试都以描述性块开始，如这里所示：

```
describe 'cookbook::recipe_name' do 
  [...]
end
```

在这个代码块中，我们想要在模拟的 CentOS 7.2 环境中模拟 Chef 运行，使用默认属性。这就是上下文，我们期望这个 Chef 运行不会引发任何错误：

```
describe 'apache::default' do
  context 'Default attributes on CentOS 7.2' do
    let(:chef_run) do
      runner = ChefSpec::ServerRunner.new(platform: 'centos', version: '7.2.1511')
      runner.converge(described_recipe)
    end

    it 'converges successfully' do
      expect { chef_run }.to_not raise_error
    end
  end
end
```

要找到我们可能需要的确切 CentOS 版本（过去或未来），我们可以访问 CentOS 镜像站点，[`mirror.centos.org/centos/`](http://mirror.centos.org/centos/)，或者阅读[`github.com/customink/fauxhai/tree/master/lib/fauxhai/platforms`](https://github.com/customink/fauxhai/tree/master/lib/fauxhai/platforms)上的所有模拟平台的完整列表。

使用`chef exec rspec`执行我们的第一个单元测试（它使用 Chef DK 中捆绑的`rspec`）：

```
$ chef exec rspec --color
.....

Finished in 0.82521 seconds (files took 1.87 seconds to load)
5 examples, 0 failures

ChefSpec Coverage report generated...

 Total Resources:   2
 Touched Resources: 0
 Touch Coverage:    0.0%

Untouched Resources:

 yum_package[httpd]                 apache/recipes/default.rb:7
 service[httpd]                     apache/recipes/default.rb:11

```

我们可以看到模拟的 Chef 运行执行时间，以及覆盖率报告（目前为 0%，因为我们还没有测试任何内容）。ChefSpec 甚至会告诉我们哪些内容尚未进行单元测试！

一个不错的选项是*文档* RSpec 格式化器，这样我们就能看到正在测试的内容描述。在本节的最后，我们将得到类似这样的输出，使用该格式化器：

```
$ chef exec rspec --format documentation --color

apache::default
 Default attributes on CentOS 7.2
 converges successfully
 installs httpd
 enables and starts httpd service

apache::virtualhost
 Default attributes on CentOS 7.2
 converges successfully
 creates a virtualhost directory
 creates and index.html file
 creates a virtualhost configuration file

Finished in 1.14 seconds (files took 2.56 seconds to load)
7 examples, 0 failures

ChefSpec Coverage report generated...

 Total Resources:   5
 Touched Resources: 5
 Touch Coverage:    100.0%

You are awesome and so is your test coverage! Have a fantastic day!

```

### 测试包安装

我们的默认配方首先安装`httpd`包。以下是在我们之前创建的上下文中，使用 ChefSpec 测试它的方法：

```
 it 'installs httpd' do
 expect(chef_run).to install_package('httpd')
 end

```

再次执行`rspec`，查看触摸覆盖率达到 50%，因为默认配方中的两个资源之一现在已被测试。

### 测试服务状态

默认配方启用并启动了`httpd`服务。以下是在之前创建的上下文中，使用 ChefSpec 测试这两个操作是否被代码处理的方式：

```
 it 'enables and starts httpd service' do
 expect(chef_run).to enable_service('httpd')
 expect(chef_run).to start_service('httpd')
 end

```

由于我们测试了两个声明的资源，现在默认配方的测试覆盖率达到了 100%。

### 测试来自同一食谱的另一个配方

由于我们在 apache 食谱中有两个配方，接下来我们将为第二个配方`virtualhost_spec.rb`创建测试。像第一个测试一样，以描述、上下文和初步的有效 Chef 运行测试开始：

```
require 'spec_helper'

describe 'apache::virtualhost' do
  context 'Default attributes on CentOS 7.2' do
    let(:chef_run) do
      runner = ChefSpec::ServerRunner.new(platform: 'centos', version: '7.2.1511')
      runner.converge(described_recipe)
    end

    it 'converges successfully' do
      expect { chef_run }.to_not raise_error
    end
  end
end
```

执行 RSpec，查看覆盖率从 100%降至 40%。现在有三个新的资源未经过测试，来自`apache::virtualhost`配方：

```
$ chef exec rspec --color
[...]
ChefSpec Coverage report generated...

 Total Resources:   5
 Touched Resources: 2
 Touch Coverage:    40.0%

Untouched Resources:

 directory[/var/www/default]        apache/recipes/virtualhost.rb:8
 file[/var/www/default/index.html]   apache/recipes/virtualhost.rb:15
 template[/etc/httpd/conf.d/default.conf]   apache/recipes/virtualhost.rb:22

```

好消息是，ChefSpec 仍然告诉我们哪些资源尚未经过测试！

### 测试目录创建

这个特定的`apache::virtualhost`配方首先创建一个目录。以下是我们如何测试该目录是否存在以及其所有权参数：

```
    it 'creates a virtualhost directory' do
      expect(chef_run).to create_directory('/var/www/default').with(
        user: 'root',
        group: 'root'
      )
    end
```

代码覆盖率现在是 60%！

### 测试文件创建

同一个食谱随后创建了一个索引文件。这是我们如何测试它是否按所需的所有权被创建：

```
    it 'creates and index.html file' do
      expect(chef_run).to create_file('/var/www/default/index.html').with(
        user: 'root',
        group: 'root'
      )
    end
```

代码覆盖率现在为 80%！

### 测试模板创建

该食谱以从模板中创建 Apache VirtualHost 结束。以下是如何测试它是否按默认属性创建：

```
    it 'creates a virtualhost configuration file' do
      expect(chef_run).to create_template('/etc/httpd/conf.d/default.conf').with(
        user: 'root',
        group: 'root'
      )
    end
```

总的来说，我们现在已经覆盖了 100% 的资源！

正如输出所说：

```
You are awesome and so is your test coverage! Have a fantastic day!

```

### 为搜索虚拟数据包进行存根

我们之前创建的 `mysite` cookbook 包含一个搜索虚拟数据包的功能，以便稍后用内容填充文件。问题是，我们正在进行单元测试，没有真实的 Chef 服务器来响应请求。因此，测试失败了：模拟的 Chef 运行没有成功，因为无法执行搜索。幸运的是，ChefSpec 允许我们使用真实内容存根数据包。下面是在 `mysite` cookbook 中的 `spec/unit/recipes/default_spec.rb` 文件中是如何操作的：

```
describe 'mysite::default' do
  context 'Default attributes on CentOS 7.2' do
    let(:chef_run) do
      runner = ChefSpec::ServerRunner.new(platform: 'centos', version: '7.2.1511')
      runner.create_data_bag('webusers', {
 'john' => {
 'id' => 'john',
 'htpasswd' => '$apr1$AUI2Y5pj$0v0PaSlLfc6QxZx1Vx5Se.'
 }
 })
      runner.converge(described_recipe)
    end

    it 'converges successfully' do
      expect { chef_run }.to_not raise_error
    end
  end
end
```

现在模拟的 Chef 运行中有一个 `webusers` 数据包，并且可以使用一些示例数据！

### 测试食谱的包含

在另一个食谱中包含食谱是非常常见的。通常，当使用通知从文件更改重新启动服务时，相关服务必须包含在文件资源所在的食谱中；否则，代码很可能是偶然工作的，因为所需的依赖 cookbook 可能在其他地方被包含！这是如何测试 cookbook 包含的方式：

```
    it 'includes the `apache` recipes' do
      expect(chef_run).to include_recipe('apache::default')
      expect(chef_run).to include_recipe('apache::virtualhost')
    end
```

现在我们确保依赖项始终被包含。

### 在测试中拦截错误

有时我们需要处理第三方 cookbooks，这些 cookbooks 可能会引发错误。这就是官方 MySQL cookbook 的情况，它依赖于 RHEL/CentOS 平台的 SELinux cookbook。由于某种原因，这个 cookbook 无法与 ChefSpec 一起使用，因此在运行时，它会报错 `chefspec not supported!`。ChefSpec 停止在此，并表示 Chef 运行出错。由于我们无法控制这种情况，这里有一个解决方法，用于期望 Chef 运行中的特定错误，之后它将多次派上用场：

```
    it 'converges successfully' do
      # The selinux cookbook raises this error.
      expect { chef_run }.to raise_error(RuntimeError, 'chefspec not supported!')
    end
```

我们已经看到了 Chef cookbook 中最常见和可重用的单元测试！

## 还有更多……

使用 Puppet 时，Puppet Labs 提供了一个包含多个有用工具的仓库，我们将在本章中使用——Puppet Labs Spec Helper。让我们安装它：

```
$ sudo puppet resource package puppetlabs_spec_helper provider=puppet_gem

```

对于单元测试，rspec-puppet 是 Puppet 中相当于 ChefSpec 的工具，并已作为 `puppetlabs_spec_helper` 的依赖安装。现在我们将为 Apache 模块中的每个清单添加单元测试。首先，我们需要一个 `Rakefile` 来创建所需的目标。幸运的是，`puppetlabs_spec_helper` gem 提供了这些目标。让我们在 Apache 模块的顶级目录中创建一个 `Rakefile`，内容如下：

```
require 'puppetlabs_spec_helper/rake_tasks'

```

所有单元测试应保持在 `spec` 目录中。在编写任何测试之前，我们还需要一个适用于所有测试的助手脚本。让我们在 `spec/spec_helper.rb` 中创建它。该文件应包含以下行：

```
require 'puppetlabs_spec_helper/module_spec_helper'

```

现在我们准备好编写单元测试了。我们的模块中有四个清单，我们将为每个清单创建一个单元测试。目标如下：

+   对于`apache/manifests/init.pp`清单：单元测试需要验证清单是否正在编译，`apache2`包是否已安装，`apache2`服务是否正在运行并在启动时激活。

+   对于`apache/manifests/vhost.pp`清单：单元测试应确保虚拟主机已在`/etc/apache2/sites-available`中创建，并在`/etc/apache2/sites-enabled`中激活。

+   对于`apache/manifests/htpasswd.pp`清单：单元测试应确保`htpasswd`文件正确生成。

+   对于`apache/manifests/htaccess.pp`清单：单元测试应确保`.htaccess`文件正确生成。

让我们尝试第一个！由于清单包含类声明，单元测试应放在`spec/classes`中。类名为`apache`，这将是包含测试的文件的基本名称。每个测试文件应以`_spec.rb`为后缀，因此让我们创建`spec/classes/apache_spec.rb`并添加以下内容：

```
require 'spec_helper'

# Description of the "apache" class
describe 'apache' do
  # Assertion list
  it { is_expected.to compile.with_all_deps }
  it { is_expected.to contain_package('apache2').with(
     {
      'ensure' => 'present',
    }
  ) }
  it { is_expected.to contain_service('apache2').with(
     {
      'ensure' => 'running',
      'enable' => 'true',
    }
  ) }
end
```

单元测试位于描述性块中，包含一系列断言。在这里，我们有三个在描述测试目标时提到的断言。

现在，让我们使用`spec` rake 目标运行单元测试：

```
$ rake spec
...

Finished in 2.42 seconds (files took 1.53 seconds to load)
3 examples, 0 failures

```

就是这样！我们的三个断言已成功测试！

另外三个测试应放在`spec/defines`下，因为对应的清单声明了`define`语句。让我们创建：

+   `spec/defines/apache_vhost_spec.rb`，其内容如下：

    ```
    require 'spec_helper'

    # Description of the "apache::vhost" 'define' resource
    describe 'apache::vhost', :type => :define do

      # As a requirement, we should load the apache class
      let :pre_condition do
        'class {"apache":;}'
      end

      # Define a title for the 'define' resource
      let :title do
        'mysite'
      end

      # Parameters list 
      let :params do 
        {
          :website => 'www.sample.com' , 
          :docroot => '/var/www/docroot',
        }
      end

      # Assertions list
      it { is_expected.to compile }
      it { is_expected.to contain_class('apache') }
      it { is_expected.to contain_file('/etc/apache2/sites-available/www.sample.com.conf')
        .with_content(/DocumentRoot \/var\/www\/docroot/) }
      it { is_expected.to contain_file('/etc/apache2/sites-enabled/www.sample.com.conf').with(
        'ensure' => 'link',
        'target' => '/etc/apache2/sites-available/www.sample.com.conf'
      ) }
    end
    ```

+   `spec/defines/apache_htpasswd_spec.rb`，其内容如下：

    ```
    require 'spec_helper'

    # Description of the "apache::htpasswd" 'define' resource
    describe 'apache::htpasswd', :type => :define do

      # As a requirement, we should load the apache class
      let :pre_condition do
        'class {"apache":;}'
      end

      # Define a title for the 'define' resource
      let :title do
        'myhtpasswd'
      end

      # Parameters list
      let :params do 
        {
          :filepath => '/tmp/htpasswd' , 
          :users => [ { "id" => "user1", "htpasswd" => "hash1" } ]
        }
      end

      # Assertion list
      it { is_expected.to compile }
      it { is_expected.to contain_class('apache') }
      it { is_expected.to contain_file('/tmp/htpasswd')
        .with_content(/user1:hash1/) }
    end
    ```

+   `spec/defines/apache_htaccess_spec.rb`，其内容如下：

    ```
    require 'spec_helper'

    # Description of the "apache::htaccess" 'define' resource
    describe 'apache::htaccess', :type => :define do

      # As a requirement, we should load the apache class
      let :pre_condition do
        'class {"apache":;}'
      end

      # Define a title for the 'define' resource
      let :title do
        'myhtaccess'
      end

      # Parameters list
      let :params do 
        {
          :filepath => '/tmp/htpasswd' , 
          :docroot => '/var/www/docroot',
        }
      end

      # Assertion list
      it { is_expected.to compile }
      it { is_expected.to contain_class('apache') }
      it { is_expected.to contain_file('/var/www/docroot/.htaccess')
        .with_content(/AuthUserFile \/tmp\/htpasswd/) }
    end
    ```

现在我们已经完成了所有单元测试，每个测试都验证了我们之前定义的初始目标。总的断言数为`13`，现在我们可以运行完整的测试套件：

```
$ rake spec
.............

Finished in 2.88 seconds (files took 1.52 seconds to load)
13 examples, 0 failures

```

### 注意

提供的 Rake 目标还包含一个`lint`目标，可以使用`rake lint`命令。我们可以直接使用这个目标，而不是像之前那样手动使用 puppet-lint。

## 另请参见

+   ChefSpec：[`sethvargo.github.io/chefspec`](http://sethvargo.github.io/chefspec)

+   ChefSpec GitHub 仓库提供了丰富的高质量示例：[`github.com/se`](https://github.com/se)[thvargo/chefspec/tree/m](http://thvargo/chefspec/tree/m)[aster/examples](http://aster/examples)

+   Puppet RSpec：[`rspec-puppet.com`](http://rspec-puppet.com)

+   Rspec：[`rspec.info`](http://rspec.info)

# 使用 Test Kitchen 对 Chef 进行测试基础设施，使用 Beaker 对 Puppet 进行测试

Test Kitchen 是 Chef 生态系统中的核心工具，它使基础设施代码的彻底测试成为可能，并且与我们已经使用和了解的许多其他工具非常兼容。它将开发世界中的强大测试文化应用于基础设施即代码环境。Test Kitchen 帮助启动一个隔离的系统环境，应用 Chef cookbooks，然后执行测试。支持的测试框架包括 RSpec，ServerSpec 或 Bats（还有更多），支持的环境包括 AWS，Vagrant，Digital Ocean，Docker 和 OpenStack。Test Kitchen 与 Berkshelf 集成得非常好，因此在测试复杂基础设施时，cookbook 的依赖关系不会成为问题。最棒的是，它已经包含在 Chef DK 中，我们只需使用它。

在这一部分，我们将构建所有必要的内容，以便使用 Vagrant 和 CentOS 7.2 正确测试我们的 Chef cookbooks 代码

### 注意

本文撰写时 Chef DK 中使用的 Test Kitchen 版本是 1.13.2。

## 准备就绪

要逐步执行此步骤，您需要以下内容：

+   工作站上安装了可用的 Chef DK

+   工作站上安装了可用的 Vagrant

+   来自第六章的 Chef 代码，*使用 Chef 和 Puppet 管理服务器的基础知识*，或任何自定义 Chef 代码

## 如何操作……

Test Kitchen 由位于 cookbook 根目录的单个`.kitchen.yml`文件进行配置。它包含了大量信息：

+   如何测试系统（默认使用 Vagrant）

+   如何配置系统（chef-solo，chef-zero 或其他模式）

+   测试哪些平台（Ubuntu 16.04，CentOS 7.2 或其他发行版）

+   测试套件（应用什么，在哪里找到信息，在哪种上下文中，类似的信息）

### 配置 Test Kitchen

无论我们是否已经有了`.kitchen.yml`文件，都让我们打开它并填写以下详细信息：

+   我们希望使用**Vagrant**运行测试，以便尽可能模拟生产环境中的虚拟机

+   我们希望使用**Chef Zero**进行配置（通过在本地模拟 Chef 服务器）

+   我们只希望在**CentOS 7.2**上进行测试（我们的代码目前不支持在其他平台上运行）

+   我们希望只有一个测试套件，包含`mysite::default`配方的运行列表，以及**Data Bags**的路径

这是我们的`.kitchen.yml`文件在`mysite` cookbook 中的样子：

```
---
driver:
  name: vagrant

provisioner:
  name: chef_zero

platforms:
  - name: centos-7.2

suites:
  - name: default
    data_bags_path: "../../data_bags"
    run_list:
      - recipe[mysite::default]
    attributes:
```

### 使用 Test Kitchen 进行测试

要简单地启动具有指定配置的 Test Kitchen，请执行以下命令：

```
$ kitchen test
-----> Testing <default-centos-72>
-----> Creating <default-centos-72>...
[...]
 Finished creating <default-centos-72> (1m1.51s).
-----> Converging <default-centos-72>...
[...]
-----> Installing Chef Omnibus (install only if missing)
[...]
 resolving cookbooks for run list: ["mysite::default"]
 Synchronizing Cookbooks:
 - apache (0.5.0)
 - php (0.2.0)
 - selinux (0.9.0)
 - yum-mysql-community (1.0.0)
 - mysite (0.3.1)
 - mysql (8.0.4)
 - yum (4.0.0)
[...]
 Chef Client finished, 41/56 resources updated in 02 minutes 47 seconds
 Finished converging <default-centos-72> (3m18.96s).
-----> Setting up <default-centos-72>...
 Finished setting up <default-centos-72> (0m0.00s).
-----> Verifying <default-centos-72>...
 Preparing files for transfer
 Transferring files to <default-centos-72>
 Finished verifying <default-centos-72> (0m0.00s).
-----> Destroying <default-centos-72>...
 ==> default: Stopping the VMware VM...
 ==> default: Deleting the VM...
 Vagrant instance <default-centos-72> destroyed.
 Finished destroying <default-centos-72> (0m28.38s).
 Finished testing <default-centos-72> (4m48.86s).
-----> Kitchen is finished. (4m50.01s)

```

这里发生的事情如下：

+   Test Kitchen 读取了`.kitchen.yml`文件

+   Test Kitchen 使用指定的镜像创建了 Vagrant 虚拟机

+   Test Kitchen 安装了 Chef，同步了 cookbook，解决了 Berkshelf 中的依赖关系，并应用了`run_list`内容

+   Test Kitchen 启动了测试（目前我们没有测试）

+   Test Kitchen 销毁了虚拟机，因为一切都进展顺利

## 它是如何工作的……

当我们执行简单的`kitchen test`命令时，实际上是在执行五个步骤：

1.  `kitchen create`：这将创建虚拟测试环境（在我们的情况下，通过 Vagrant 和一个 hypervisor），但不进行配置。

1.  `kitchen converge`：这会使用我们创建的 `.kitchen.yml` 中的套件信息配置实例。因为我们正在使用带有 Chef 的 Test Kitchen，它首先安装 Chef，然后为我们解决 cookbook 的依赖关系。然后，它以请求的 Chef 模式（在我们的情况下是 chef-zero）应用 `run_list`。

1.  `kitchen setup`：这将安装我们可能需要的任何额外插件。

1.  `kitchen verify`：这首先安装运行测试所需的所有内容 —— 在我们的情况下，这将是 ServerSpec。

1.  `kitchen destroy`：如果所有测试通过，此步骤将销毁测试环境。

我们强烈建议您按顺序使用每个命令进行调试。

供参考，因为这将在下一节讨论，所有测试都位于 `test/integration/<suite_name>/<plugin_name>` 文件夹中。换句话说，`test/integration/default/serverspec/virtualhost_spec.rb` 文件将匹配名为 `virtualhost` 的 Chef cookbook 配方，从 `default` Kitchen 测试套件执行，并使用 `serverspec` 插件进行测试。

## 还有更多…

Puppet 的对应工具是 Beaker。Beaker 的开发非常活跃，当前版本（6.x）需要至少 Ruby 2.2.5。为了使用 Puppet Collections 提供的嵌入式 Ruby，让我们保持在 5.x 分支上：

```
$ sudo puppet resource package beaker-rspec 
provider=puppet_gem ensure=5.6.0

```

### 注意

安装 Beaker 需要 C/C++ 编译器，因此在尝试安装 `beaker-rspec` 之前，请安装 gcc/g++ 或 clang。还需要 Zlib 库（二进制和头文件）。

我们还需要另一个包含辅助工具的 gem：`beaker-puppet_install_helper`。这个 gem 主要用于在测试期间在虚拟机中安装 Puppet：

```
$ sudo puppet resource package beaker-puppet_install_helper provider=puppet_gem

```

我们首先需要定义一个支持的平台列表，用于运行测试验收。每个平台必须在 `spec/acceptance/nodesets` 中的 YAML 文件中定义。由于我们的代码仅在 Ubuntu 上工作，请在 `spec/acceptance/nodesets/default.yml` 中定义一个平台：

```
HOSTS:
 ubuntu-1604-x64:
 roles:
 - agent
 - default
 platform: ubuntu-16.04-amd64
 hypervisor: vagrant
 box: bento/ubuntu-16.04
CONFIG:
 type: foss

```

正如您所看到的，我们将使用 Vagrant 作为 hypervisor，并使用 Ubuntu Xenial box。

### 注意

`type: foss` 表示将使用 Puppet 的开源版本。

现在我们可以运行 Beaker：

```
$ rake beaker
/opt/puppetlabs/puppet/bin/ruby -I/opt/puppetlabs/puppet/lib/ruby/gems/2.1.0/gems/rspec-support-3.6.0.beta1/lib:/opt/puppetlabs/puppet/lib/ruby/gems/2.1.0/gems/rspec-core-3.6.0.beta1/lib /opt/puppetlabs/puppet/lib/ruby/gems/2.1.0/gems/rspec-core-3.6.0.beta1/exe/rspec --pattern spec/acceptance --color
No examples found.
Finished in 0.00081 seconds (files took 0.14125 seconds to load)
0 examples, 0 failures

```

尚未定义任何验收测试，但我们将在接下来的页面中看到如何编写验收测试。

## 另请参阅

+   Bats 测试框架：[`github.com/sstephenson/bats`](https://github.com/sstephenson/bats)

+   RSpec：[`rspec.info/`](http://rspec.info/)

+   ServerSpec：[`serverspec.org/`](http://serverspec.org/)

+   Test Kitchen 驱动程序：[`rubygems.org/search?query=kitchen-`](https://rubygems.org/search?query=kitchen-)

# 使用 ServerSpec 进行集成测试

集成测试在单元测试之后进行：现在我们正在真实的黑盒系统上测试实际功能。我们可能正在使用许多做着大量事情的菜谱，每个单元在早期阶段已经进行了测试，但它们在实际运行中是如何协同工作的呢？所有组件组装在一起，意图可能一致，但现实可能大相径庭。重写可能会重叠，遗漏的食谱可能会改变行为，服务可能无法启动，接着就会发生变化，回归问题可能会引入，或者更新后的系统或更新可能会导致故障；在真实系统中，事物出错的原因有无数种。这就是我们需要集成测试的原因；测试我们所有菜谱组合应用到真实测试系统后的结果，现在开始。

在 Chef 的情况下，我们有一个很棒的工具，叫做 Test Kitchen，帮助我们处理这个问题，我们之前已经安装并配置好了它来运行和执行测试。现在，让我们开始编写这些测试吧！

我们将编写集成测试，针对第六章中的`mysite`菜谱，*使用 Chef 和 Puppet 管理服务器的基础*，用于演示目的，但这些测试完全通用，可以在任何地方重用。我们将测试服务、文件、目录、yum 仓库、软件包、端口和注入内容。通过这种方式，我们可以确信我们编写的代码实际上在（模拟的）现实世界中做到了预期的功能！

### 注意

我们强烈建议您将这些集成测试添加到自动化 CI 系统中。这样，在代码变更后，测试可以自动启动，随着时间的推移，复杂度会随着更多测试用例的加入而飙升，这样您就不必再考虑它了：所有内容都会被测试，如果您的变更破坏了某些您未注意到的内容，您会在几秒钟内知道。没有人愿意在每次更改时，手动验证在三个版本的四种操作系统上是否没有出错。

## 准备工作

要完成这份食谱，您需要以下内容：

+   工作站上安装好的 Chef DK

+   工作站上安装好的 Vagrant

+   来自第六章的 Chef 代码，或者任何自定义的 Chef 代码

## 如何操作……

根据我们测试的菜谱的创建方式，可以创建一个带有一些示例内容的`test`文件夹。我们不需要它，所以请确保清除`test`文件夹下的所有内容，从头开始。我们将使用第六章中的`mysite`菜谱，作为构建我们的 ServerSpec 测试的基础菜谱，但显然这些测试可以在任何地方使用：

```
$ cd cookbooks/mysite
$ rm -rf test/*

```

Test Kitchen 与*测试套件*一起工作，因此它期望在`integration`文件夹中有一个与套件名称相同的文件夹层次结构。一个`default`测试套件的最终文件夹层次结构将是`mysite/test/integration/default/serverspec`。

```
$ mkdir -p test/integration/default/serverspec

```

### 创建 ServerSpec 辅助脚本

ServerSpec 至少需要两行配置，每个测试中都必须重复这些配置。为了避免重复，我们可以在`test/integration/default/serverspec/spec_helper.rb`中创建一个辅助脚本：

```
require 'serverspec'
# Required by serverspec
set :backend, :exec
```

现在我们所有的测试只需在文件顶部包含以下内容：

```
require 'spec_helper'
```

### 测试包的安装

我们的食谱做了很多事情，其中最重要的之一就是包的安装。这些内容之前已经进行了单元测试，但现在我们进入了集成阶段。这些包真的安装了吗？让我们通过在`apache_spec.rb`中编写测试来查明`httpd`包是否安装：

```
require 'spec_helper'

describe package('httpd') do
  it { should be_installed }
end
```

现在我们可以启动 Test Kitchen，看看这个特定的包是否真的安装了！

### 注意

在编写集成测试时，我们强烈建议使用 Test Kitchen 来创建/汇聚/设置/验证过程，而不是使用简单的`kitchen test`命令来一次性完成所有操作——手动方式要快得多！

类似地，测试`php`包在`php_spec.rb`文件中的内容将完全相同：

```
require 'spec_helper'

describe package('php') do
  it { should be_installed }
end

describe package('php-cli') do
  it { should be_installed }
end

describe package('php-mysql') do
  it { should be_installed }
end
```

### 测试服务状态

ServerSpec 允许我们测试实际的进程状态。在安装 Apache HTTPD 服务器的配方中，我们要求它启用并运行。让我们通过在`apache_spec.rb`文件中添加以下内容来验证它是否真是如此：

```
describe service('httpd') do
  it { should be_enabled }
  it { should be_running }
end
```

对于我们的 MySQL 安装，官方食谱中的文档指出，默认服务名称是`mysql-default`（而不是通常的`mysqld`）。在`mysql_spec.rb`文件中，添加以下内容：

```
describe service('mysql-default') do
  it { should be_enabled }
  it { should be_running }
end
```

### 测试监听端口

ServerSpec 是一个很棒的工具，可以测试监听端口。在我们的案例中，我们期望 Apache 监听`80`端口（HTTP），并且我们将 MySQL 配置为监听`3306`端口。将以下内容添加到`apache_spec.rb`文件中：

```
describe port('80') do
  it { should be_listening }
end
```

类似地，在`mysql_spec.rb`文件中添加以下内容来测试 MySQL：

```
describe port('3306') do
  it { should be_listening }
end
```

### 测试文件的存在性和内容

我们之前已经对在我们的食谱中创建这些文件的意图进行了单元测试，例如具有自定义名称的 VirtualHost，这会影响文件名和内容（这正是第六章的`mysite`食谱所做的，覆盖了自定义 apache 食谱的默认值）。它真的有效吗？让我们通过使用`vhost_spec.rb`测试虚拟主机配置来找出答案：

```
describe file('/etc/httpd/conf.d/mysite.conf') do
  it { should exist }
  it { should be_mode 644 }
  its(:content) { should match /ServerName mysite/ }
  it { should be_owned_by 'root' }
  it { should be_grouped_into 'root' }
end
```

这实际上证明了默认属性确实被`mysite`值覆盖，并且虚拟主机配置文件的内容也与该值匹配。这个食谱确实有效。

目录可以像这样在相同的`vhost_spec.rb`文件中进行测试：

```
describe file('/var/www/mysite') do
  it { should be_directory }
end
```

另一个有趣的测试是检查`htpasswd`文件的内容；在第六章，*使用 Chef 和 Puppet 管理服务器的基础知识*中，我们编写了一个食谱，向 Chef 服务器请求数据包中的授权用户。我们通过模拟数据包来进行单元测试，然后使用 Test Kitchen 配置它以模拟这些数据包的可用性。这个特定于 Chef 服务器的代码是否真的有效，并将`john`用户添加到`htpasswd`文件中，同时限制对该文件的访问？让我们通过在`htaccess_spec.rb`文件中添加以下内容来找出答案：

```
describe file('/etc/httpd/htpasswd') do
  it { should exist }
  it { should be_mode 660 }
  its(:content) { should match /john/ }
  it { should be_owned_by 'root' }
  it { should be_grouped_into 'root' }
end
```

### 测试仓库是否存在

我们的`mysite`食谱示例来自第六章，*使用 Chef 和 Puppet 管理服务器的基础知识*，它使用官方的 Chef 食谱来部署 MySQL，其中包括添加一个 yum 仓库。由于它现在是系统的重要组成部分，我们最好测试它的存在和状态！要测试 yum 仓库，可以将以下内容添加到`mysql_spec.rb`文件中：

```
describe yumrepo('mysql57-community') do
    it { should be_exist   }
    it { should be_enabled }
end
```

系统的许多其他部分也可以使用 ServerSpec 进行测试，特别是在网络方面（路由表、网关和接口）、Unix 用户和组、实际命令、cron 作业等。

## 还有更多内容…

使用 Puppet 和 Beaker，让我们尝试为我们的 Apache 模块编写验收测试。验收测试需要放置在`spec/acceptance`目录中。

我们需要定义一个将由所有验收测试共享的辅助文件。让我们创建一个`spec/spec_helper_acceptance.rb`文件，内容如下：

```
require 'beaker-rspec'
require 'beaker/puppet_install_helper'

# Install puppet
run_puppet_install_helper

RSpec.configure do |c|
  # Project root
  proj_root = File.expand_path(File.join(File.dirname(__FILE__), '..'))

  # Output should contain test descriptions
  c.formatter = :documentation

  # Configure nodes
  c.before :suite do
    # Install module 
    puppet_module_install(:source => proj_root, :module_name => 'apache')
  end
end
```

这个辅助文件将用于在测试机器上安装 Puppet，并将我们的`apache`模块填充到模块目录中。

作为主要`apache`类的第一个基本验收测试，让我们创建`spec/acceptances/classes/apache_spec.rb`，内容如下：

```
require 'spec_helper_acceptance'

describe 'Apache' do
  describe 'Puppet code' do
    it 'should compile and work with no error' do
      pp = <<-EOS
        class { 'apache': }
      EOS

      apply_manifest(pp, :catch_failures => true)
      apply_manifest(pp, :catch_changes => true)
    end
  end
end
```

该测试的目标如下：

+   使用我们的类安装 Apache

+   验证 Puppet 是否正确应用。

+   验证第二次运行 Puppet 时不会改变任何内容：我们想证明代码是幂等的。

让我们进行测试吧！

```
$ rake beaker
...
...
Beaker::Hypervisor, found some vagrant boxes to create
Bringing machine 'ubuntu-1604-x64' up with 'virtualbox' provider...
...
...
Apache
 Puppet code
localhost $ scp /var/folders/k9/7sp85p796qx7c22btk7_tgym0000gn/T/beaker20161101-75828-1of1g5j ubuntu-1604-x64:/tmp/apply_manifest.pp.cZK277 {:ignore => }
localhost $ scp /var/folders/k9/7sp85p796qx7c22btk7_tgym0000gn/T/beaker20161101-75828-1l28bth ubuntu-1604-x64:/tmp/apply_manifest.pp.q2Z81Z {:ignore => }
 should compile and work with no error
Destroying vagrant boxes
==> ubuntu-1604-x64: Forcing shutdown of VM...
==> ubuntu-1604-x64: Destroying VM and associated drives...

Finished in 19.68 seconds (files took 1 minute 20.11 seconds to load)
1 example, 0 failures

```

在这个例子中，Beaker 创建了机器，安装了 Puppet，上传了我们的代码，应用 Puppet 两次以验证我们的测试，并销毁了机器。

若想获得更多关于 Puppet 代理安装和执行的日志，可以在`nodeset`文件中添加一行`log_level: verbose`：

```
HOSTS:
  ubuntu-1604-x64:
    roles:
      - agent
      - default
    platform: ubuntu-16.04-amd64
    hypervisor: vagrant
    box: bento/ubuntu-16.04
CONFIG:
  type: foss
 log_level: verbose

```

现在，让我们扩展我们的测试，使用 Apache 模块中包含的所有代码。我们需要更新文件顶部的清单，以便执行以下操作：

+   安装 Apache

+   定义一个虚拟主机

+   创建虚拟主机的根目录

+   创建一个包含测试用户的`htpasswd`文件

+   在根目录中创建一个`.htaccess`文件，使用之前的`htpasswd`文件

关于测试，我们希望：

+   验证 Puppet 是否应用

+   验证代码是幂等的

+   验证 Apache 是否正在运行并在启动时激活

+   验证 Apache 是否在监听

+   验证虚拟主机是否已部署并且激活，且 `DocumentRoot` 配置正确

+   验证 `htpasswd` 文件是否已部署并且内容正确

+   验证 `.htaccess` 文件是否已部署并且内容正确

更新后的验收测试代码如下：

```
require 'spec_helper_acceptance'

describe 'Apache' do
  describe 'Puppet code' do
    it 'should compile and work with no error' do
      pp = <<-EOS
        class { 'apache': }
 apache::vhost{'mysite':
 website    => 'www.sample.com',
 docroot    => '/var/www/docroot',
 }
 apache::htpasswd{'htpasswd':
 filepath => '/etc/apache2/htpasswd',
 users    => [ { "id" => "user1", "htpasswd" => "hash1" } ],
 }
 file { '/var/www/docroot':
 ensure => directory,
 owner  => 'www-data',
 group  => 'www-data',
 mode   => '0755',
 }
 apache::htaccess{'myhtaccess':
 filepath => '/etc/apache2/htpasswd',
 docroot  => '/var/www/docroot',
 }
      EOS

      apply_manifest(pp, :catch_failures => true)
      apply_manifest(pp, :catch_changes => true)
    end
  end

 # Apache running and enabled at boot ?
 describe service('apache2') do
 it { is_expected.to be_enabled }
 it { is_expected.to be_running }
 end

 # Apache listening ?
 describe port(80) do
 it { is_expected.to be_listening }
 end

 # Vhost deployed ?
 describe file ('/etc/apache2/sites-available/www.sample.com.conf') do
 its(:content) { should match /DocumentRoot \/var\/www\/docroot/ }
 end

 describe file ('/etc/apache2/sites-enabled/www.sample.com.conf') do
 it { is_expected.to be_symlink }
 end

 # htpasswd file deployed ?
 describe file ('/etc/apache2/htpasswd') do
 its(:content) { should match /user1:hash1/ }
 end

 # htaccess file deployed ?
 describe file ('/var/www/docroot/.htaccess') do
 its(:content) { should match /AuthUserFile \/etc\/apache2\/htpasswd/ }
 end

end

```

现在，让我们再次尝试运行 Beaker：

```
$ rake beaker
…
…
Beaker::Hypervisor, found some vagrant boxes to create
Bringing machine 'ubuntu-1604-x64' up with 'virtualbox' provider...
…
…
Apache
 Puppet code
localhost $ scp /var/folders/k9/7sp85p796qx7c22btk7_tgym0000gn/T/beaker20161103-41882-1twwbr2 ubuntu-1604-x64:/tmp/apply_manifest.pp.nWPdZJ {:ignore => }
localhost $ scp /var/folders/k9/7sp85p796qx7c22btk7_tgym0000gn/T/beaker20161103-41882-73vqlb ubuntu-1604-x64:/tmp/apply_manifest.pp.0Jht7j {:ignore => }
 should compile and work with no error
 Service "apache2"
 should be enabled
 should be running
 Port "80"
 should be listening
 File "/etc/apache2/sites-available/www.sample.com.conf"
 content
 should match /DocumentRoot \/var\/www\/docroot/
 File "/etc/apache2/sites-enabled/www.sample.com.conf"
 should be symlink
 File "/etc/apache2/htpasswd"
 content
 should match /user1:hash1/
 File "/var/www/docroot/.htaccess"
 content
 should match /AuthUserFile \/etc\/apache2\/htpasswd/
Destroying vagrant boxes
==> ubuntu-1604-x64: Forcing shutdown of VM...
==> ubuntu-1604-x64: Destroying VM and associated drives...

Finished in 20.22 seconds (files took 1 minute 24.54 seconds to load)
8 examples, 0 failures

```

我们现在有了一个完整的 Apache 模块验收测试套件！

## 另见

+   ServerSpec GitHub: [`github.com/serverspec/`](https://github.com/serverspec/)

+   ServerSpec 主页: [`serverspec.org/`](http://serverspec.org/)

+   Test Kitchen 主页: [`kitchen.ci/`](http://kitchen.ci/)

+   一个启用 Beaker 的 Puppet 模块示例骨架: [`gitlab.com/joshbeard/puppet-module-test`](https://gitlab.com/joshbeard/puppet-module-test)

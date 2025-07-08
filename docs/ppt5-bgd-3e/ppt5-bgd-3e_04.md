# 第四章：理解 Puppet 资源

|   | *困惑是知识的开始。* |   |
| --- | --- | --- |
|   | --*哈里勒·吉布兰* |

我们已经遇到过三种重要的 Puppet 资源类型：`package`、`file` 和 `service`。在本章中，我们将进一步了解这些资源类型，以及用于管理用户、组、SSH 密钥、定时任务和任意命令的其他重要资源类型。

![理解 Puppet 资源](img/B08880_04_01.jpg)

# 文件

我们在第二章 *创建你的第一个清单* 中看到，Puppet 可以使用 `file` 资源管理节点上的文件，并且我们查看了一个示例，该示例使用 `content` 属性将文件内容设置为特定字符串。这里是该示例的再次展示（`file_hello.pp`）：

```
file { '/tmp/hello.txt':
  content => "hello, world\n",
}
```

## path 属性

我们已经看到，每个 Puppet 资源都有一个标题（一个带引号的字符串后跟一个冒号）。在 `file_hello` 示例中，`file` 资源的标题是 `'/tmp/hello.txt'`。很容易猜测，Puppet 将使用此值作为创建文件的路径。事实上，`path` 是你可以为 `file` 指定的属性之一，但如果你没有指定，Puppet 将使用资源的标题作为 `path` 的值。

## 管理整个文件

虽然能够将文件的内容设置为简短的文本字符串很有用，但我们可能希望管理的大多数文件都太大，无法直接包含在 Puppet 清单中。理想情况下，我们会将文件的副本放在 Puppet 仓库中，并让 Puppet 将其复制到文件系统中的指定位置。`source` 属性正是实现这一功能的（`file_source.pp`）：

```
file { '/etc/motd':
  source => '/examples/files/motd.txt',
}
```

要在你的 Vagrant box 上尝试这个示例，请运行以下命令：

```
sudo puppet apply /examples/file_source.pp
cat /etc/motd
The best software in the world only sucks. The worst software is significantly worse than that.
-Luke Kanies
```

（从现在开始，我不会再给出如何运行示例的明确说明；只需按照这里所示的方式使用 `sudo puppet apply` 即可。书中的所有示例都位于 GitHub 仓库的 `examples/` 目录中，我会为每个示例提供相应的文件名，例如 `file_source.pp`。）

### 提示

为什么我们必须运行 `sudo puppet apply` 而不是直接运行 `puppet apply`？Puppet 拥有运行它的用户的权限，因此如果 Puppet 需要修改一个由 `root` 拥有的文件，它必须以 `root` 的权限运行（这正是 `sudo` 的作用）。你通常会以 `root` 用户身份运行 Puppet，因为它需要这些权限来执行诸如安装软件包、修改由 `root` 拥有的配置文件等任务。

`source` 属性的值可以是节点上某个文件的路径，如这里所示，或是一个 HTTP URL，如以下示例所示（`file_http.pp`）：

```
file { '/tmp/README.md':
  source => 'https://raw.githubusercontent.com/puppetlabs/puppet/master/README.md',
}
```

尽管这是一个非常方便的功能，但请记住，每次你将这样的外部依赖添加到 Puppet 清单时，实际上是在添加一个潜在的故障点。

### 提示

在可能的情况下，使用文件的本地副本，而不是每次都让 Puppet 从远程获取文件。这尤其适用于需要从网站下载 tarball 构建的软件。如果可能，下载 tarball 并通过本地 Web 服务器或文件服务器提供它。如果这不可行，使用缓存代理服务器可以在构建大量节点时节省时间和带宽。

## 所有权

在类 Unix 系统中，文件与一个 **所有者**、一个 **组** 和一组 **权限**（用于读取、写入或执行文件）相关联。由于我们通常以 `root` 用户的权限（通过 `sudo`）运行 Puppet，Puppet 管理的文件将由该用户拥有：

```
ls -l /etc/motd
-rw-r--r-- 1 root root 109 Aug 31 04:03 /etc/motd
```

通常，这已经足够了，但如果我们需要文件属于其他用户（例如，如果该用户需要能够写入该文件），我们可以通过设置 `owner` 属性来表达这一点（`file_owner.pp`）：

```
file { '/etc/owned_by_ubuntu':
  ensure => present,
  owner  => 'ubuntu',
}
ls -l /etc/owned_by_ubuntu
-rw-r--r-- 1 ubuntu root 0 Aug 31 04:48 /etc/owned_by_ubuntu
```

您可以看到，Puppet 已创建文件，并且其所有者已设置为 `ubuntu`。您还可以使用 `group` 属性设置文件的组所有权（`file_group.pp`）：

```
file { '/etc/owned_by_ubuntu':
  ensure => present,
  owner  => 'ubuntu',
  group  => 'ubuntu',
}
ls -l /etc/owned_by_ubuntu
-rw-r--r-- 1 ubuntu ubuntu 0 Aug 31 04:48 /etc/owned_by_ubuntu
```

请注意，这次我们没有为文件指定 `content` 或 `source` 属性，而是简单地设置 `ensure => present`。在这种情况下，Puppet 将创建一个大小为零的文件。

## 权限

在类 Unix 系统中，文件具有一个关联的 **模式**，该模式决定了对文件的访问权限。它控制文件所有者、文件所在组的用户以及其他用户的读取、写入和执行权限。Puppet 支持使用 `mode` 属性设置文件的权限。该属性接受一个八进制值（以 0 开头的数字表示），每个数字代表 3 个二进制位字段：分别表示所有者、组和其他用户的权限。在以下示例中，我们使用 `mode` 属性将文件的模式设置为 `0644`（"所有者可读写，组和其他用户只能读取"）（`file_mode.pp`）：

```
file { '/etc/owned_by_ubuntu':
  ensure => present,
  owner  => 'ubuntu',
  mode   => '0644',
}
```

这对于经验丰富的系统管理员来说是非常熟悉的，因为文件权限的八进制值与 Unix 的 `chmod` 命令所理解的完全相同。欲了解更多信息，请运行命令 `man chmod`。

## 目录

创建或管理 **目录** 的权限是一个常见任务，Puppet 也使用 `file` 资源来完成此操作。如果 `ensure` 属性的值为 `directory`，则文件将是一个目录（`file_directory.pp`）：

```
file { '/etc/config_dir':
  ensure => directory,
}
```

与常规文件一样，您可以使用 `owner`、`group` 和 `mode` 属性来控制对目录的访问。

## 文件树

我们已经看到 Puppet 可以将单个文件复制到节点，但如果是一个包含子目录的整个文件夹（称为 **文件树**）呢？`recurse` 属性将处理这个问题（`file_tree.pp`）：

```
file { '/etc/config_dir':
  source  => '/examples/files/config_dir',
  recurse => true,
}
ls /etc/config_dir/
1  2  3
```

当`recurse`为`true`时，Puppet 会将源目录（在本例中为`/examples/files/config_dir/`）中的所有文件和目录（及其子目录）复制到目标目录（`/etc/config_dir/`）。

### 提示

如果目标目录已经存在且其中包含文件，Puppet 将不会干涉它们，但你可以使用`purge`属性来改变此行为。如果`purge`为`true`，Puppet 将删除目标目录中任何未出现在源目录中的文件和目录。使用此属性时请谨慎。

## 符号链接

另一个常见的文件管理需求是创建或修改**符号链接**（简称**symlink**）。你可以通过在`file`资源上设置`ensure => link`并指定`target`属性（`file_symlink.pp`）来让 Puppet 完成此操作：

```
file { '/etc/this_is_a_link':
  ensure => link,
  target => '/etc/motd',
}
ls -l /etc/this_is_a_link
lrwxrwxrwx 1 root root 9 Aug 31 05:05 /etc/this_is_a_link -> /etc/motd
```

# 软件包

我们已经看到如何使用`package`资源安装软件包，这对于大多数软件包来说足够了。然而，`package`资源还有一些额外的功能，可能会很有用。

## 卸载软件包

`ensure`属性通常会取值`installed`来安装软件包，但如果你指定`absent`，Puppet 将**移除**已安装的软件包，否则将不采取任何行动。以下示例将在`apparmor`软件包已安装的情况下将其移除（`package_remove.pp`）：

```
package { 'apparmor':
  ensure => absent,
}
```

默认情况下，当 Puppet 移除软件包时，会保留软件包管理的任何文件。要删除与软件包相关的所有文件，请使用`purged`而不是`absent`。

## 安装特定版本

如果系统的软件包管理器提供了多个版本的包，指定`ensure => installed`将使 Puppet 安装默认版本（通常是最新版本）。但是，如果你需要特定版本，你可以将该版本字符串指定为`ensure`的值，Puppet 将安装该版本（`package_version.pp`）：

```
package { 'openssl':
  ensure => '1.0.2g-1ubuntu4.8',
}
```

### 提示

在使用 Puppet 管理软件包时，最好指定确切的版本，以确保所有节点都安装相同版本的软件包。否则，如果你使用`ensure => installed`，它们将只安装构建时当前的版本，从而导致不同的节点安装了不同版本的软件包。

当软件包发布新版本时，如果你决定升级，你可以更新 Puppet 清单中指定的版本字符串，Puppet 会在所有地方升级该软件包。

## 安装最新版本

另一方面，如果你为软件包指定`ensure => latest`，Puppet 会确保每次应用清单时都安装最新的版本。当软件包有新版本发布时，下一次 Puppet 运行时会自动安装。

### 提示

当使用不在你控制下的包仓库（例如 Ubuntu 主仓库）时，这通常不是你想要的行为。这意味着包会在意外的时间进行升级，可能会导致应用崩溃（或者至少导致计划外的停机）。更好的策略是告诉 Puppet 安装你知道可以正常工作的特定版本，并在受控环境中测试升级，之后再将其应用到生产环境中。

如果你维护自己的包仓库并控制新包的发布，`ensure => latest` 是一个有用的功能：Puppet 会在你向仓库推送新版本时立即更新包。如果你依赖上游仓库，比如 Ubuntu 仓库，最好通过直接指定版本号来管理版本，而不是使用 `ensure`。

## 安装 Ruby gems

虽然 `package` 资源通常用于通过普通的系统包管理器（在 Ubuntu 中是 APT）安装包，它也可以安装其他类型的包。Ruby 编程语言的库包被称为 **gems**。Puppet 可以通过 `provider => gem` 属性为你安装 Ruby gems（`package_gem.pp`）：

```
package { 'ruby':
  ensure => installed,
}

package { 'puppet-lint':
  ensure   => installed,
  provider => gem,
}
```

`puppet-lint` 是一个 Ruby gem，因此我们必须为这个包指定 `provider => gem`，以防 Puppet 把它当作标准的系统包并试图通过 APT 安装。由于 `gem` 提供者只有在安装 Ruby 后才可用，我们首先安装 `ruby` 包，然后安装 `puppet-lint` gem。

顺便说一下，`puppet-lint` 工具是一个很有用的工具。它会检查你的 Puppet 清单文件是否存在常见的样式错误，并确保它们符合官方 Puppet 风格指南。现在就试试吧：

```
puppet-lint /examples/lint_test.pp
WARNING: indentation of => is not properly aligned (expected in column 11, but found it in column 10) on line 2
```

在这个示例中，`puppet-lint` 警告你 `=>` 箭头没有垂直对齐，风格指南规定它们应该是垂直对齐的：

```
file { '/tmp/lint.txt':
  ensure => file,
  content => "puppet-lint is your friend\n",
}
```

当 `puppet-lint` 不产生任何输出时，说明文件没有 lint 错误。

## 在 Puppet 环境中安装 gems

Puppet 本身至少部分是用 Ruby 编写的，并且使用了多个 Ruby gem。为了避免与节点可能需要的 Ruby 版本及其他应用所需 gem 的冲突，Puppet 将自己的 Ruby 版本和相关 gem 打包到 `/opt/puppetlabs/` 目录下。这意味着你可以安装（或移除）任意系统版本的 Ruby，而 Puppet 不会受到影响。

然而，如果你需要安装一个 gem 来扩展 Puppet 的某些功能，使用 `package` 资源和 `provider => gem` 是无法工作的。也就是说，gem 会被安装，但仅在系统的 Ruby 环境中，并且 Puppet 无法看到它。

幸运的是，`puppet_gem` 提供者正是为这个目的而存在的。当你使用这个提供者时，gem 会安装到 Puppet 的环境中（自然，它不会在系统环境中显示）。以下示例展示了如何使用这个提供者（`package_puppet_gem.pp`）：

```
package { 'r10k':
  ensure   => installed,
  provider => puppet_gem,
}
```

### 提示

要查看 Puppet 上下文中安装的 gem，可以使用 Puppet 自己版本的`gem`命令，路径如下：

`/opt/puppetlabs/puppet/bin/gem list`

## 使用`ensure_packages`

为了避免 Puppet 代码的不同部分之间，或者你的代码与第三方模块之间可能发生的包冲突，Puppet 标准库提供了一个有用的`package`资源的包装器，叫做`ensure_packages()`。我们将在第七章中详细介绍，*掌握模块*。

# 服务

尽管操作系统级别的服务实现方式多种多样且复杂，Puppet 通过`service`资源很好地抽象化了大部分内容，并只暴露了你最常用的两个服务属性：服务是否正在运行（`ensure`）以及是否在启动时启动（`enable`）。我们在第二章中介绍了这些内容，*创建你的第一个清单*，而且大多数情况下，你不需要知道更多关于`service`资源的内容。

然而，你偶尔会遇到一些服务由于各种原因与 Puppet 不兼容。有时，Puppet 无法检测到服务已经在运行，并且一直尝试启动它。其他时候，当一个依赖资源发生变化时，Puppet 可能无法正确重启服务。对于`service`资源，有一些有用的属性可以帮助解决这些问题。

## `hasstatus`属性

当`service`资源具有`ensure => running`属性时，Puppet 需要能够检查该服务是否实际上正在运行。它的实现方式取决于底层操作系统。例如，在 Ubuntu 16 及更高版本中，它会运行`systemctl is-active SERVICE`。如果该服务是以`systemd`工作方式打包的，那么这应该没问题，但在许多情况下，尤其是对于旧的软件，它可能无法正常响应。

如果你发现 Puppet 在每次运行时都试图启动服务，即使该服务已经在运行，可能是 Puppet 的默认服务状态检测无法正常工作。在这种情况下，你可以为该服务指定`hasstatus => false`属性（`service_hasstatus.pp`）：

```
service { 'ntp':
  ensure    => running,
  enable    => true,
  hasstatus => false,
}
```

当`hasstatus`为 false 时，Puppet 知道不需要使用默认的系统服务管理命令来检查服务状态，而是会在进程表中查找与服务名称匹配的正在运行的进程。如果找到了，Puppet 将推断该服务正在运行，并且不会采取进一步的行动。

## `pattern`属性

有时，当使用`hasstatus => false`时，Puppet 中定义的服务名称实际上并不会出现在进程表中，因为提供该服务的命令有一个不同的名称。如果是这种情况，你可以通过`pattern`属性告诉 Puppet 具体应该查找什么。

如果`hasstatus`为`false`并且指定了`pattern`，Puppet 将会在进程表中搜索`pattern`的值，以判断服务是否正在运行。为了找到你需要的模式，你可以使用`ps`命令查看正在运行的进程列表：

```
ps ax

```

查找你感兴趣的进程，并选择一个仅与该进程名称匹配的字符串。例如，如果是`ntpd`，你可以将`pattern`属性指定为`ntpd`（`service_pattern.pp`）：

```
service { 'ntp':
  ensure    => running,
  enable    => true,
  hasstatus => false,
  pattern   => 'ntpd',
}
```

## hasrestart 和 restart 属性

当服务被通知时（例如，如果一个`file`资源使用`notify`属性告知服务其配置文件已更改，这是我们在第二章，*创建你的第一个清单*中看到的常见模式），Puppet 的默认行为是先停止服务，然后重新启动它。通常这样做有效，但许多服务在其管理脚本中实现了`restart`命令。如果该命令可用，通常使用它是个好主意：它可能比停止并启动服务更快或更安全。例如，一些服务在停止时需要花费一定时间才能正确关闭，而 Puppet 可能没有等足够的时间就尝试重启它们，这样就可能导致服务根本没有运行。

如果你为一个服务指定了`hasrestart => true`，那么 Puppet 会尝试发送一个`restart`命令，使用当前平台适当的服务管理命令（例如，在 Ubuntu 上是`systemctl`）。以下示例展示了`hasrestart`的使用（`service_hasrestart.pp`）：

```
service { 'ntp':
  ensure     => running,
  enable     => true,
  hasrestart => true,
}
```

更复杂的是，默认的系统服务`restart`命令可能无法工作，或者在重启服务时可能需要执行某些特殊操作（例如禁用监控通知）。你可以使用`restart`属性（`service_custom_restart.pp`）为服务指定任何你喜欢的`restart`命令：

```
service { 'ntp':
  ensure  => running,
  enable  => true,
  restart => '/bin/echo Restarting >>/tmp/debug.log && systemctl restart ntp',
}
```

在这个例子中，`restart`命令会在重新启动服务之前，向日志文件写入一条消息，但它当然可以做你需要的任何事情。注意，`restart`命令仅在 Puppet 重启服务时使用（通常是因为服务收到了某个配置文件更改的通知）。如果 Puppet 发现服务已经停止并需要启动它，它将使用正常的系统服务启动命令，而不是使用`restart`命令。

在极为罕见的情况下，如果无法通过默认的服务管理命令停止或启动服务，Puppet 还提供了`stop`和`start`属性，允许你指定自定义命令来停止和启动服务，和`restart`属性的使用方式一样。不过，如果你需要使用这些命令，可能可以安全地说你今天运气不好。

# 用户

类 Unix 系统中的用户不一定对应于登录并输入命令的具体人，尽管有时确实是。用户只是一个命名实体，可以拥有文件并以特定权限运行命令，可能具有或不具有读取或修改其他用户文件的权限。出于良好的安全考虑，为每个系统服务创建一个独立的用户账户是很常见的做法。这意味着该服务以该用户的身份和权限运行。

例如，web 服务器通常以`www-data`用户身份运行，该用户仅用于拥有 web 服务器需要读取和写入的文件。这限制了通过 web 服务器发生安全漏洞的风险，因为攻击者只能拥有`www-data`用户的权限，该权限非常有限，而不是`root`用户的权限，后者可以修改系统的任何方面。通常，作为`root`用户运行暴露于公共互联网的服务是一个不好的主意。服务用户应该只拥有操作该服务所需的最小权限。

基于此，系统配置的一个重要部分是创建和管理用户，而 Puppet 的`user`资源为此提供了一个模型。正如我们在包和服务中看到的，实施细节和管理用户的命令在不同操作系统之间有很大的差异，但 Puppet 提供了一个抽象，隐藏了这些细节，通过一组通用的用户属性来管理。

## 创建用户

以下示例展示了 Puppet 中典型的`user`和`group`声明（`user.pp`）：

```
group { 'devs':
  ensure => present,
  gid    => 3000,
}

user { 'hsing-hui':
  ensure => present,
  uid    => '3001',
  home   => '/home/hsing-hui',
  shell  => '/bin/bash',
  groups => ['devs'],
}
```

## 用户资源

资源的标题是用户的用户名（登录名）；在这个例子中是`hsing-hui`。`ensure => present`属性表示该用户应当存在于系统中。

`uid`属性需要更多的解释。在类 Unix 系统中，每个用户都有一个唯一的数字标识符，称为**uid**。与用户关联的文本名称仅仅是为了方便那些（比如人类）更喜欢字符串而非数字的情况。访问权限实际上是基于 uid 而不是用户名的。

### 提示

为什么要设置`uid`属性？通常，当手动创建用户时，我们没有指定 uid，系统会自动分配一个。这样做的问题是，如果在三个不同的节点上创建相同的用户（例如`hsing-hui`），可能会得到三个不同的 uid。只要从未在节点之间共享文件或从一个地方复制数据到另一个地方，这不会成为问题。但实际上，这种情况经常发生，因此确保每个用户的 uid 在基础设施中的所有节点上都相同是很重要的。这就是为什么我们在 Puppet 清单中指定`uid`属性的原因。

`home`属性设置用户的主目录（如果用户登录，该目录将是当前工作目录，且为以该用户身份运行的 cron 任务的默认工作目录）。

`shell` 属性指定了用户交互式登录时要运行的命令行 shell。对于人类用户，通常是用户 shell，如 `/bin/bash` 或 `/bin/sh`。对于服务用户，如 `www-data`，shell 应该设置为 `/usr/sbin/nologin`（在 Ubuntu 系统上），这会阻止交互式访问，并显示消息 `此账户当前不可用`。所有不需要交互式登录的用户应该使用 `nologin` shell。

如果用户需要成为某些组的成员，你可以通过 `groups` 属性传递一个包含组名的数组（在此示例中仅为 `devs`）。

尽管 Puppet 为 `user` 资源支持 `password` 属性，我不建议你使用它。服务用户不需要密码，交互式用户应使用 SSH 密钥登录。实际上，你应该配置 SSH 完全禁用密码登录（在 `sshd_config` 中设置 `PasswordAuthentication no`）。

## 组资源

资源的标题是组的名称（`devs`）。你不需要指定 `gid` 属性，但与 `uid` 属性一样，最好指定它。

## 管理 SSH 密钥

我喜欢在生产节点上尽量减少交互式登录，因为这能减少攻击面。幸运的是，使用配置管理工具，通常不需要实际登录到节点。需要交互式登录的最常见原因是系统维护和故障排除，以及部署。在这两种情况下，应该有一个单独的账户，专门用于此目的（例如 `admin` 或 `deploy`），并配置为具有需要登录该账户的任何用户或系统的 SSH 密钥。

Puppet 提供了 `ssh_authorized_key` 资源来控制与用户账户关联的 SSH 密钥。以下示例展示了如何使用 `ssh_authorized_key` 将一个 SSH 密钥（在此示例中是我的密钥）添加到我们 Vagrant 虚拟机上的 `ubuntu` 用户 (`ssh_authorized_key.pp`)：

```
ssh_authorized_key { 'john@bitfieldconsulting.com':
  user => 'ubuntu',
  type => 'ssh-rsa',
  key  => 'AAAAB3NzaC1yc2EAAAABIwAAAIEA3ATqENg+GWACa2BzeqTdGnJhNoBer8x6pfWkzNzeM8Zx7/2Tf2pl7kHdbsiTXEUawqzXZQtZzt/j3Oya+PZjcRpWNRzprSmd2UxEEPTqDw9LqY5S2B8og/NyzWaIYPsKoatcgC7VgYHplcTbzEhGu8BsoEVBGYu3IRy5RkAcZik=',
}
```

资源的标题是 SSH 密钥的注释，提醒我们这个密钥属于谁。`user` 属性指定了此密钥应被授权的用户账户。`type` 属性标识 SSH 密钥的类型，通常是 `ssh-rsa` 或 `ssh-dss`。最后，`key` 属性设置了密钥本身。当应用此清单时，它会将以下内容添加到 `ubuntu` 用户的 `authorized_keys` 文件中：

```
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAIEA3ATqENg+GWACa2BzeqTdGnJhNoBer8x6pfWkzNzeM8Zx7/2Tf2pl7kHdbsiTXEUawqzXZQtZzt/j3Oya+PZjcRpWNRzprSmd2UxEEPTqDw9LqY5S2B8og/NyzWaIYPsKoatcgC7VgYHplcTbzEhGu8BsoEVBGYu3IRy5RkAcZik= john@bitfieldconsulting.com
```

一个用户账户可以关联多个 SSH 密钥，持有其中一个对应私钥及其密码短语的人将能够以该用户身份登录。

## 删除用户

如果你需要 Puppet 移除用户账户（例如，作为员工离职的流程一部分），仅仅从 Puppet 清单中删除 `user` 资源是不够的。Puppet 会忽略系统中它不了解的用户，而且它绝对不会删除任何未在 Puppet 清单中提到的系统内容；那样做会非常不理想（几乎所有内容都会被删除）。因此，我们需要保留 `user` 声明一段时间，但将 `ensure` 属性设置为 `absent`（`user_remove.pp`）：

```
user { 'godot':
  ensure => absent,
}
```

一旦 Puppet 在所有地方运行完成，你可以选择移除 `user` 资源，但完全没有必要，实际上，除非你能手动确认该用户已从所有受影响的系统中删除，否则最好保留该资源。

### 提示

如果你需要防止某个用户登录，但又想保留该用户的账户和所有文件（用于归档或合规性目的），你可以将其 `shell` 设置为 `/usr/sbin/nologin`。你还可以删除与该账户关联的任何 `ssh_authorized_key` 资源，并将 `user` 资源上的 `purge_ssh_keys` 属性设置为 `true`。这将移除任何非 Puppet 管理的授权密钥。

# Cron 资源

Cron 是类 Unix 系统中用于运行定时任务的机制，有时也称为批处理任务，这些任务会在指定的时间或间隔执行。例如，系统维护任务，如日志轮换或检查安全更新，都是通过 cron 来执行的。关于执行内容和时间的详细信息会保存在一个特殊格式的文件中，叫做 `crontab`（即 **cron 表** 的缩写）。

Puppet 提供了 `cron` 资源来管理定时任务，我们在 第三章 *用 Git 管理 Puppet 代码* (`run-puppet.pp`) 的清单中看到过这个例子：

```
cron { 'run-puppet':
  command => '/usr/local/bin/run-puppet',
  hour    => '*',
  minute  => '*/15',
}
```

`run-puppet` 标题标识了该 cron 任务（Puppet 会在 `crontab` 文件中写入一个注释，包含该名称，以区分其他手动配置的 cron 任务）。`command` 属性指定了 cron 执行的命令，`hour` 和 `minute` 指定了时间（`*/15` 是 cron 语法，意思是“每 15 分钟”）。

### 注意

如果需要了解更多关于 cron 及指定定时任务时间的不同方法，可以运行命令 `man 5 crontab`。

## cron 资源的属性

`cron` 资源还有一些其他有用的属性，这些在以下示例（`cron.pp`）中展示：

```
cron { 'cron example':
  command     => '/bin/date +%F',
  user        => 'ubuntu',
  environment => ['MAILTO=admin@example.com', 'PATH=/bin'],
  hour        => '0',
  minute      => '0',
  weekday     => ['Saturday', 'Sunday'],
}
```

`user` 属性指定了谁来运行该 cron 任务（如果没有指定，任务将作为 `root` 用户运行）。如果给定了 `environment` 属性，它会设置 cron 任务可能需要的任何环境变量。一个常见的用途是通过 `MAILTO` 变量将 cron 任务的任何输出邮件发送到指定的邮箱地址。

如前所述，`hour` 和 `minute` 属性设置作业运行的时间，而 `weekday` 属性可用于指定一周的特定日或多个日。(`monthday` 属性也是这样工作的，并且可以取 1-31 之间的任何范围或数组值来指定月份的日期。)

### 提示

关于 cron 调度的一个重要点是，任何调度属性的默认值都是 `*`，这意味着 *所有允许的值*。例如，如果未指定 `hour` 属性，那么 cron 作业将被调度为 `hour` 为 `*`，这意味着它将每小时运行一次。这通常不是您想要的。如果确实希望每小时运行一次，请在您的清单中指定 `hour => '*'`，但否则，请指定应在其运行的特定小时。对于 `minute` 也是如此。意外地省略 `minute` 属性，并导致作业每小时运行六十次，可能会产生有趣的后果，至少可以这么说。

## 随机化 cron 作业

如果在许多节点上运行一个 cron 作业，最好确保作业不会在同一时间同时运行。Puppet 提供了一个内置函数 `fqdn_rand()` 来帮助实现这一点；它提供一个随机数，最大值为指定值，因为随机数生成器是用节点的主机名种子化的，所以在每个节点上都会不同。

如果您有几个这样的作业要运行，还可以向 `fqdn_rand()` 函数提供一个进一步的种子值，这可以是任何字符串，将确保该值对于每个作业都是不同的（`fqdn_rand.pp`）：

```
cron { 'run daily backup':
  command => '/usr/local/bin/backup',
  minute  => '0',
  hour    => fqdn_rand(24, 'run daily backup'),
}

cron { 'run daily backup sync':
  command => '/usr/local/bin/backup_sync',
  minute  => '0',
  hour    => fqdn_rand(24, 'run daily backup sync'),
}
```

因为我们为每个 cron 作业的 `fqdn_rand` 的第二个参数给出了不同的字符串，它将为每个 `hour` 属性返回不同的随机值。

`fqdn_rand()` 返回的值范围包括 0，但不包括您指定的最大值。因此，在前面的示例中，`hour` 的值将在 0 到 23 之间，包括这两个值。

## 删除 cron 作业

就像对待 `user` 资源或任何类型的资源一样，从 Puppet 清单中删除资源声明并不会从节点中删除相应的配置。为了做到这一点，您需要在资源上指定 `ensure => absent`。

# Exec 资源

尽管我们迄今为止看到的其他资源类型（`file`、`package`、`service`、`user`、`ssh_authorized_key` 和 `cron`）都在节点上建模了一些具体的状态，例如文件，但 `exec` 资源有些不同。`exec` 允许您在节点上运行任意命令。这可能会创建或修改状态，也可能不会；您可以从命令行运行的任何内容，也可以通过 `exec` 资源运行。

## 自动化手动交互

`exec`资源最常见的用途是模拟命令行上的手动交互。一些较旧的软件并未为现代操作系统打包，需要从源代码编译和安装，这要求你运行特定的命令。有些软件的作者也没有意识到，或者根本不关心，用户可能正在尝试自动安装他们的产品，而他们的安装脚本会提示用户输入。这可能需要使用`exec`资源来解决这个问题。

## `exec`资源的属性

以下示例展示了一个用于构建和安装虚拟软件的`exec`资源（`exec.pp`）：

```
exec { 'install-cat-picture-generator':
  cwd     => '/tmp/cat-picture-generator',
  command => '/tmp/cat-picture/generator/configure && /usr/bin/make install',
  creates => '/usr/local/bin/cat-picture-generator',
}
```

资源的标题可以是你喜欢的任何内容，不过，像往常一样，Puppet 资源必须是唯一的。我倾向于根据它们尝试解决的问题来命名`exec`资源，就像这个示例中一样。

`cwd`属性设置了执行命令的工作目录（**当前工作目录**）。在安装软件时，这通常是软件源目录。

`command`属性指定要执行的命令。这个必须是命令的完整路径，但你可以使用 shell `&&` 运算符将多个命令链接在一起。只有前一个命令成功执行时，才会执行下一个命令。因此，在这个示例中，如果`configure`命令成功完成，Puppet 将继续执行`make install`，否则它会因错误而停止。

### 注意

如果你应用这个示例，Puppet 会给出类似以下的错误：

```
Error: /Stage[main]/Main/Exec[install-cat-picture-generator]/returns: change from notrun to 0 failed: Could not find command '/tmp/cat-picture/generator/configure'
```

这是预期的，因为指定的命令实际上并不存在。在你自己的清单中，如果你给命令的路径错误，或者提供命令的软件包尚未安装，你可能会看到这个错误。

`creates`属性指定在命令执行后应存在的文件。如果该文件存在，Puppet 将不会再次运行该命令。这非常有用，因为如果没有`creates`属性，`exec`资源每次 Puppet 运行时都会执行，这通常不是你想要的。`creates`属性告诉 Puppet，实际上是“只有在该文件不存在时才运行`exec`”。

让我们来看一下它是如何工作的，假设这个`exec`是第一次运行。我们假设`/tmp/cat-picture/`目录存在，并且包含`cat-picture-generator`应用的源代码。

1.  Puppet 检查`creates`属性，发现`/usr/local/bin/cat-picture-generator`文件不存在；因此，必须运行`exec`资源。

1.  Puppet 执行`/tmp/cat-picture-generator/configure && /usr/bin/make install`命令。这些命令的副作用是创建了`/usr/local/bin/cat-picture-generator`文件。

1.  下次 Puppet 运行时，它再次检查`creates`属性。这次`/usr/local/bin/cat-picture-generator`文件已经存在，因此 Puppet 不做任何操作。

只要 `creates` 属性中指定的文件存在，该 `exec` 资源就永远不会再次应用。你可以通过删除该文件并再次应用 Puppet 来测试此行为。`exec` 资源将被触发并重新创建文件。

### 提示

确保你的 `exec` 资源始终包含 `creates` 属性（或类似的控制属性，如 `onlyif` 或 `unless`，我们将在本章稍后查看）。没有这个属性，`exec` 命令将每次 Puppet 运行时都执行，这几乎肯定不是你想要的。

请注意，从源代码构建和安装软件并不是生产系统中的推荐做法。最好在专用构建服务器上构建软件（可能使用类似本示例的 Puppet 代码），为其创建系统包，然后使用 Puppet 在生产节点上安装该包。

## `user` 属性

如果你没有为 `exec` 资源指定 `user` 属性，Puppet 会以 `root` 用户身份运行该命令。这通常适用于安装系统软件或更改系统配置，但如果你希望命令以特定用户身份运行，可以指定 `user` 属性，如以下示例所示（`exec_user.pp`）：

```
exec { 'say-hello':
  command => '/bin/echo Hello, this is `whoami` >/tmp/hello-ubuntu.txt',
  user    => 'ubuntu',
  creates => '/tmp/hello-ubuntu.txt',
}
```

这将以 `ubuntu` 用户身份运行指定的命令。`whoami` 命令返回运行该命令的用户名，因此当你应用此清单时，文件 `/tmp/hello-ubuntu.txt` 将被创建，内容如下：

```
Hello, this is ubuntu
```

与前面的示例一样，`creates` 属性可以防止 Puppet 多次运行该命令。

## `onlyif` 和 `unless` 属性

假设你只希望在某些条件下应用 `exec` 资源。例如，处理传入数据文件的命令只在有数据文件等待处理时才需要运行。在这种情况下，添加 `creates` 属性没有用；我们希望某个特定文件的存在触发 `exec`，而不是阻止它。

`onlyif` 属性是解决此问题的好方法。它指定了一个 Puppet 要执行的命令，且该命令的退出状态决定是否应用 `exec`。在类 Unix 系统中，命令通常返回零的退出状态表示成功，非零值表示失败。以下示例展示了如何以这种方式使用 `onlyif`（`exec_onlyif.pp`）：

```
exec { 'process-incoming-cat-pictures':
  command => '/usr/local/bin/cat-picture-generator --import /tmp/incoming/*',
  onlyif  => '/bin/ls /tmp/incoming/*',
}
```

精确的命令在这里并不重要，但假设它是我们只希望在 `/tmp/incoming/` 目录中有文件时运行的命令。

`onlyif` 属性指定了 Puppet 首先应运行的检查命令，以确定是否需要应用 `exec` 资源。如果 `/tmp/incoming/` 目录为空，`ls /tmp/incoming/*` 将返回非零退出状态。Puppet 将此视为失败，因此不会应用 `exec` 资源。

另一方面，如果`/tmp/incoming/`目录中有文件，`ls`命令将返回成功。这告诉 Puppet 该`exec`资源必须被应用，所以它继续执行`/usr/local/bin/cat-picture-generator`命令（我们可以假设这个命令在处理后会删除传入的文件）。

你可以将`onlyif`属性看作是告诉 Puppet，“只有在这个命令成功的情况下，才运行`exec`资源”。

`unless`属性与`onlyif`属性完全相同，只是语义相反。如果你为`unless`属性指定一个命令，那么除非该命令返回零退出状态，否则`exec`将始终运行。你可以将`unless`看作是告诉 Puppet，“除非这个命令成功，否则运行`exec`资源”。

当你应用清单时，如果你看到每次都在运行一个`exec`资源，而这个资源不应该这样运行，请检查它是否指定了`creates`、`unless`或`onlyif`属性。如果它指定了`creates`属性，可能是它在寻找错误的文件；如果指定了`unless`或`onlyif`命令，可能它返回的结果与你的预期不符。你可以通过运行带有`-d`（调试）标志的`sudo puppet apply`命令来查看正在执行的命令以及它生成的输出：

```
sudo puppet apply -d exec_onlyif.pp
Debug: Execprocess-incoming-cat-pictures: Executing check '/bin/ls /tmp/incoming/*'
Debug: Executing: '/bin/ls /tmp/incoming/*'
Debug: /Stage[main]/Main/Exec[process-incoming-cat-pictures]/onlyif: /tmp/incoming/foo
```

## `refreshonly`属性

使用`exec`资源执行一次性命令是很常见的，例如重建数据库或设置系统可调节的参数。这些通常只需要在安装软件包时触发一次，或者偶尔在更新配置文件时触发。如果一个`exec`资源只需要在某个其他 Puppet 资源发生变化时运行，我们可以使用`refreshonly`属性来实现这一点。

如果`refreshonly`为`true`，则除非另一个资源通过`notify`触发它，否则`exec`将永远不会被应用。在以下示例中，Puppet 管理`/etc/aliases`文件（该文件将本地用户名映射到电子邮件地址），对该文件的更改会触发执行命令`newaliases`，该命令重建系统别名数据库（`exec_refreshonly.pp`）：

```
file { '/etc/aliases':
  content => 'root: john@bitfieldconsulting.com',
  notify  => Exec['newaliases'],
}

exec { 'newaliases':
  command     => '/usr/bin/newaliases',
  refreshonly => true,
}
```

当第一次应用这个清单时，`/etc/aliases`资源会导致文件内容发生变化，因此 Puppet 会向`exec`资源发送一个`notify`消息。这导致执行`newaliases`命令。如果你再次应用该清单，你会看到`aliases`文件没有变化，因此`exec`没有被执行。

### 提示

虽然`refreshonly`属性偶尔非常有用，但过度使用它会使你的 Puppet 清单难以理解和调试，而且也可能变得相当脆弱。Felix Frank 在一篇博客文章中提出了这个观点，*朋友不要让朋友使用 Refreshonly*：

“考虑到`exec`资源类型是最后的手段，它的`refreshonly`参数应该被视为尤其离谱。为了使`exec`资源更好地融入 Puppet 的模型，你应该使用[`creates`、`onlyif`或`unless`]参数。”请参阅：

[`ffrank.github.io/misc/2015/05/26/friends-don't-let-friends-use-refreshonly/`](http://ffrank.github.io/misc/2015/05/26/friends-don't-let-friends-use-refreshonly/)

请注意，你不需要使用`refreshonly`属性来让`exec`资源被其他资源通知。任何资源都可以通知`exec`资源以使其运行；然而，如果你希望它*仅在*被通知时运行，可以使用`refreshonly`。

### 提示

顺便提一下，如果你确实想在节点上管理电子邮件别名，请使用 Puppet 内置的`mailalias`资源。之前的示例只是为了演示如何使用`refreshonly`。

## logoutput 属性

当 Puppet 通过`exec`资源运行 shell 命令时，输出通常对我们是隐藏的。然而，如果命令似乎没有正确工作，查看它产生的输出通常非常有用，因为这通常能告诉我们为什么它没有工作。

`logoutput`属性决定 Puppet 是否会记录`exec`命令的输出，并与通常的信息性 Puppet 输出一起显示。它可以有三个值：`true`、`false`或`on_failure`。

如果将`logoutput`设置为`on_failure`（这是默认值），Puppet 仅在命令失败时记录命令输出（即返回非零退出状态时）。如果你不希望看到任何命令输出，可以将其设置为`false`。

然而，有时命令会返回成功的退出状态，但似乎没有做任何事情。将`logoutput`设置为`true`将强制 Puppet 记录命令输出，无论退出状态如何，这应该能帮助你找出问题所在。

## 超时属性

有时，命令可能需要很长时间才能运行，或者根本不会终止。默认情况下，Puppet 允许`exec`命令运行 300 秒，在此期间如果命令尚未完成，Puppet 会终止它。如果你需要允许命令稍长时间完成，可以使用`timeout`属性来设置。该值是命令最大执行时间（以秒为单位）。

将`timeout`值设置为`0`会完全禁用自动超时，并允许命令无限期运行。这应当是最后的手段，因为如果没有设置超时，一个阻塞或挂起的命令可能会完全停止 Puppet 的自动运行。为了找到合适的`timeout`值，尝试运行命令几次，并选择一个大约是典型运行时间两倍的值。这应该能避免因网络慢导致的失败，但不会完全阻止 Puppet 运行。

## 如何避免错误使用 exec 资源

`exec` 资源可以对系统做任何你从命令行可以做的事情。正如你可以想象的那样，像这样的强大工具可能会被滥用。理论上，Puppet 是一种声明性语言：清单指定了事物应该是什么样子，Puppet 负责采取必要的措施使其实现。因此，清单是计算机科学家所说的**幂等**的：在应用了 catalog 后，系统始终保持在相同的状态，无论你应用多少次，它都会始终保持在这个状态。

`exec` 资源在一定程度上破坏了这个理论模型，因为它允许 Puppet 清单具有副作用。由于你的 `exec` 命令可以做任何事情，举个例子，它可能会在磁盘上创建一个新的 1 GB 大小的文件，文件名是随机的，而由于每次 Puppet 运行时都会发生这种情况，你可能会很快耗尽磁盘空间。最好避免像这样的具有副作用的命令。通常情况下，无法从 Puppet 内部精确知道 `exec` 资源对系统造成了哪些变化。

通过 `exec` 运行的命令有时也被用来绕过 Puppet 的现有资源。例如，如果 `user` 资源因某些原因没有完全满足你的需求，你可以通过 `exec` 直接运行 `adduser` 命令来创建用户。这也是一个不好的做法，因为这样你就失去了 Puppet 内建资源的声明性和跨平台特性。`exec` 资源可能会以一种 Puppet catalog 看不见的方式改变节点的状态。

### 提示

通常来说，如果你需要管理 Puppet 内建资源类型不支持的系统状态的某个具体方面，你应该考虑创建自定义资源类型和提供程序来完成你想做的事情。这将扩展 Puppet，添加一个新的资源类型，之后你可以在清单中使用它来建模该资源的状态。创建自定义类型和提供程序是一个高级主题，本书不涉及，但如果你想了解更多内容，可以查阅 Puppet 文档：

[`docs.puppet.com/guides/custom_types.html`](https://docs.puppet.com/guides/custom_types.html)

在通过 `exec` 运行复杂命令之前，你也应该三思而后行，尤其是使用了循环或条件语句的命令。一个更好的做法是将任何复杂的逻辑放在 shell 脚本中（或者更好的是放在真正的编程语言中），然后用 Puppet 部署并运行它（避免我们之前提到的，不必要的副作用）。

### 提示

根据良好的 Puppet 风格，每个 `exec` 资源应该至少指定 `creates`、`onlyif`、`unless` 或 `refreshonly` 中的一个，以防止它在每次 Puppet 运行时都被应用。如果你发现自己每次 Puppet 运行时都只是为了执行一个命令而使用 `exec`，不如将其设置为一个 cron 任务。

# 总结

我们详细探索了 Puppet 的`file`资源，涵盖了文件源、所有权、权限、目录、符号链接和文件树。我们学会了如何通过安装特定版本或最新版本来管理软件包，以及如何卸载软件包。我们还了解了 Ruby gems，既包括系统上下文中的，也包括 Puppet 内部上下文中的。在这个过程中，我们接触到了非常有用的`puppet-lint`工具。

我们已经研究了`service`资源，包括`hasstatus`、`pattern`、`hasrestart`、`restart`、`stop`和`start`属性。我们学会了如何创建用户和组，管理主目录、shell、UID 和 SSH 授权密钥。我们还了解了如何调度、管理和删除 cron 作业。

最后，我们已经了解了强大的`exec`资源，包括如何运行任意命令，以及如何在特定条件下或仅在某个文件不存在时运行命令。我们还了解了如何使用`refreshonly`属性，在其他资源更新时触发`exec`资源，并探索了`exec`资源中有用的`logoutput`和`timeout`属性。

在下一章，我们将了解如何在 Puppet 清单中表示数据和变量，包括字符串、数字、布尔值、数组和哈希。我们将学会如何使用变量和条件表达式来确定应用哪些资源，还将学习 Puppet 的`facts`哈希以及如何使用它获取有关系统的信息。

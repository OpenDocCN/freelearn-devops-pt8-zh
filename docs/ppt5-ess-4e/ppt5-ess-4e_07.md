# 第七章：Puppet 4 和 5 的新特性

现在我们已经对 Puppet DSL 及其概念有了全面了解，接下来是时候看看 Puppet 4 版本引入的新特性了。解析器（编译目录的工具）为了更好的性能，几乎是从零开始重写的。这个里程碑版本还增加了一些缺失的功能和编码原则。

Puppet 4 及更高版本不仅提供了新功能，还打破了旧的做法，移除了一些不再被认为是最佳实践的功能。这要求现有的 manifest 代码需要进行适当的测试，并且可能需要进行大量修改以兼容 Puppet 4。

本章将涵盖以下主题：

+   升级到 Puppet 4

+   使用 Puppet 类型系统

+   学习 Lambda 函数和常规函数

+   创建 Puppet 4 函数

+   利用新的模板引擎

+   使用 HEREDOC 处理多行文本

+   Puppet 5 服务器指标

+   打破旧有的做法

# 升级到 Puppet 4

首先让我们看看 Puppet 3 版本的用户如何进行更新。

与其升级 Puppet 主机，不如并行设置一台新服务器，并仔细迁移服务。这有一些优点，例如，如果遇到问题，可以轻松回滚。

新的 Puppet 4 及更高版本可以通过多种方式进行安装：

+   使用 Puppet Labs 的软件仓库，它们将移除旧的 Puppet 包。

+   这种方法意味着没有提前进行测试的强制切换，不推荐使用。在对 Puppet manifest 代码进行深入测试后，才应升级到 Puppet 4 及更高版本。

+   作为 Ruby gem 扩展或从 tarball 安装

+   这种方法需要单独安装 Ruby，但大多数现代 Linux 发行版并没有提供 Ruby。对于 Puppet 4，要求 Ruby 2.1；对于 Puppet 5，需要 Ruby 2.4。

+   更新到 Puppet 3.8，启用并迁移到环境路径设置，并且仅在特定的测试环境中启用未来解析器。

+   后者的解决方案是最智能的，也是最具向后兼容性的。

使用 Puppet 4 及更高版本，以及 Puppet Labs 提供的 **All-in-One** (**AIO**) 打包方式，Puppet 配置、模块、环境和 SSL 证书的路径将会发生变化。

+   Puppet 4 和 5 将其配置文件（`puppet.conf`）存储在 `/etc/puppetlabs/puppet` 目录下。

+   Hiera 配置文件位于 `/etc/puppetlabs/hiera/hiera.yaml`。

+   Puppet CA 和证书可以在 `/etc/puppetlabs/puppet/ssl` 中找到。

+   查找 Puppet 代码（环境和模块）

    安装到 `/etc/puppetlabs/code/environments/` 中

# 使用 Puppet 3.8 和环境目录

新的解析器在 Puppet 3.5 中与旧的解析器一同引入。为了使用新的语言特性，需要显式设置一个特殊的配置项。这使得早期用户和对新技术感兴趣的人可以在早期阶段测试解析器并检查代码的不兼容问题。

在 Puppet 3.x 上，新的解析器可能会在没有进一步通知的情况下发生变化。因此，建议升级到最新的 3.x 版本。

使用目录环境，可以在 `environment.conf` 配置文件中指定特定环境的设置：

```
# /etc/puppet/environments/puppet_update/environment.conf
# environment config file for special puppet_update environment
parser = future
```

接下来，你的所有 Puppet 代码需要放入新创建的环境路径中，包括节点分类。

现在，在每种不同的节点类型上，可以手动运行：

```
puppet agent --test --environment=puppet_update --noop 
```

检查主机和代理的输出及日志文件，查看是否有错误或不需要的更改，并根据需要调整你的 Puppet 代码。

# Puppet 4 和 5 主机

确保你的代理准备好与 Puppet 4 主机一起操作。有关代理的说明，请参见以下部分。

启动一个新的 Puppet 主机是另一种方法。以下过程假设已经使用 `puppet.conf` 文件中的 DNS 替代名称设置创建了 Puppet CA。如果已配置了 DNS 替代名称，则需要完全重新创建 Puppet CA。

Puppet CA 需要了解 Puppet 主机 `fqdn` 的**通用名称**（**CN**）。可以提供 DNS 替代名称，CA 也将对其有效。

通常，Puppet 使用主机的 `fqdn` 作为通用名称。但如果你在生成 CA 之前提供了配置属性 `dns_alt_names`，此配置选项将被添加到 CA 中。

强烈建议配置 `dns_alt_names`。启用此选项后，你可以扩展到多个编译主机，并为迁移过程添加额外的 Puppet 主机。

若要查找是否已添加 DNS 替代名称，可以使用 `puppet cert` 命令：

```
puppet cert list -all  
```

此命令将打印所有证书。检查你 Puppet 主机的证书。例如，考虑以下内容：

```
puppet cert list --all
+ "puppetmaster.example.net" (SHA256) 7D:11:33:00:94:B3:C4:48:D2:10:B3:C7:B0:38:71:28:C5:75:2C:61:3B:3E:63:C6:95:7C:C9:DF:59:F7:C5:BE (alt names: "DNS:puppet", "DNS:puppet.example.net", "DNS:puppetmaster.example.net")
```

以下步骤将指导你完成 Puppet 4 的设置。在基于 Debian 7 的系统上，添加 PC1 Puppet Labs 仓库：

```
curl -O http://apt.puppetlabs.com/puppetlabs-release-pc1-wheezy.deb
dpkg -i puppetlabs-release-pc1-wheezy.deb
apt-get update
apt-get install puppetserver puppet-agent
```

还不要启动 Puppet 服务器进程！需要将新的 Puppet 4 或 5 主机作为 CA 主机运行，这需要将 Puppet 3 主机的 CA 和证书复制到新的 Puppet 4 主机。

截至目前，基于 Java 的 Puppet 5 主机包需要来自 Debian 8 的 Java 8 回溯包。除此之外，PuppetDB 5 现在需要 PostgreSQL 9.6，而 Puppet 4 的 PuppetDB 必须与 PostgreSQL 9.4 一起使用。

在下一步中，所有 Puppet 代理都需要修改 `puppet.conf` 文件。你需要为 `ca_server` 和 `server` 提供不同的设置：

```
ini_setting { 'puppet_ca_server':
  path    => '/etc/puppet/puppet.conf',
  section => 'agent',
  setting => 'ca_server',
  value   => 'puppet4.example.net'
}
```

`ini_setting` 资源类型可通过 Forge 上的 `puppetlabs-inifile` 模块使用。

现在，将你的所有 Puppet 代码放到新的 Puppet 4 主机上的一个环境中（`/etc/puppetlabs/code/environments/development/{manifests,modules}`）。

通过在每个节点上运行以下命令来测试你的 Puppet 4 错误：

```
puppet agent --test --noop --server puppet4.example.net --environment development
```

更改你的 Puppet 代码以修复潜在的错误。一旦在 Puppet 4 主节点和代理上没有错误，也没有不希望的配置更改，你的代码就是与 Puppet 4 兼容的。

另一种验证你的 Puppet 代码在 Puppet 4 和 5 上完全正常工作的方式是比较目录。现在有几种解决方案可用。最常见的有 `puppetlabs-catalog_diff`、`zack-catalog` 和 `octocatalog-diff`。

# 更新 Puppet 代理

确保现有的代理已准备好与已经是版本 4 的主节点一起操作非常重要。请检查以下方面：

+   所有代理应该使用最新版本的 Puppet 3

+   代理配置应指定 `stringify_facts = false`

后续步骤为代理更新做好准备，因为 Puppet 4 始终会表现得像这样，并且避免将所有事实值转换为字符串类型。

请确保你更新到 Puppet Server 2.1 或更高版本。基于 Passenger 的主节点和 Puppet Server 2.0 与 Puppet 3 代理不兼容。

Puppet 在线文档包含了许多关于此更新路径的有用细节：[`docs.puppetlabs.com/puppetserver/latest/compatibility_with_puppet_agent.html`](http://docs.puppetlabs.com/puppetserver/latest/compatibility_with_puppet_agent.html)。

# 测试 Puppet DSL 代码

另一种验证现有 Puppet 代码是否能在 Puppet 4 上运行的方法是使用 `rspec-puppet` 和 `beaker` 进行单元和集成测试。此过程不在本书的范围内。

无论你是全新开始使用 Puppet 4，还是使用上述程序之一迁移 Puppet 3 基础设施，现在是时候发现新版本的好处了。

# 使用类型系统

旧版本的 Puppet 只支持一小部分数据类型：`Bool`、`String`、`Array` 和 `Hash`。Puppet DSL 几乎没有检查一致变量类型的功能。请考虑以下场景。

带参数的类使代码库中的其他用户能够更改类的行为和输出：

```
class ssh (
  $server = true,
){
  if $server {
    include ssh::server
  }
}
```

这个类定义检查是否已将 `server` 参数设置为 true。然而，在这个示例中，类并未防止错误数据的使用：

```
class { 'ssh': 
  server => 'false', 
} 
```

在这个类声明中，`server` 参数被赋予了一个字符串而不是布尔值。由于 `false` 字符串不为空，因此 if `$server` 条件实际上会通过。这不是用户预期的结果。

在 Puppet 3 中，推荐使用 `stdlib` 模块中的多个函数来添加参数验证：

```
class ssh (
  $server = true,
){
  validate_bool($server)
  if $server {
    include ssh::server
  }
}
```

仅使用一个参数时，这似乎是一个不错的方法。但如果你有多个参数呢？我们如何处理像哈希表这样的复杂数据类型？

这就是类型系统发挥作用的地方。类型系统了解许多通用数据类型，并遵循许多现代编程语言中也使用的模式。

Puppet 区分核心数据类型和抽象数据类型。核心数据类型是真正的数据类型，是 Puppet 代码中最常用的类型：

+   字符串

+   整型、浮动型和数字型

+   布尔值

+   数组

+   哈希

+   正则表达式

+   未定义

+   默认

在给定的示例中，`server`参数应始终检查是否包含布尔值。代码可以简化为以下模式：

```
class ssh ( 
  Boolean $server = true, 
){ 
  if $server { 
    include ssh::server 
  } 
} 
```

如果未给定布尔值参数，Puppet 将抛出错误，并说明哪个参数的数据类型不匹配：

```
class { 'ssh': 
  server = 'false', 
} 
```

显示的错误如下：

```
root@puppetmaster# puppet apply ssh.pp
Error: Expected parameter 'server' of 'Class[Ssh]' to have type Boolean, got String at ssh.pp:2 on node puppetmaster.example.net
```

`Numeric`、`Float`和`Integer`数据类型在变量及其类型方面有一些更有趣的方面。

Puppet 会自动识别由数字（无论是否带有负号）组成且没有小数点的`Integers`。

`Floats`通过小数点来识别。当对`Integer`和`Float`的组合进行算术运算时，结果将始终是`Float`。

`Floats`在`-1`和`1`之间的数值必须在小数点前写上`0`数字，否则，Puppet 会抛出错误。

此外，Puppet 还支持十进制、八进制和十六进制表示法，类似于 C 语言等语言的表示法：

+   非零十进制数不能以`0`开头

+   八进制值必须以前导`0`开始

+   十六进制值的前缀是`0x`

Puppet 会在插值过程中自动将数字转换为字符串：`("Value of number: ${number}")`。

Puppet 不会将字符串转换为数字。要实现这一点，你可以简单地在字符串上加上 0 来转换：

```
$ssh_port = '22' 
$ssh_port_integer = 0 + $ssh_port 
```

`Default`数据类型有些特殊。它并不直接指代一个数据类型，但可以在选择器和案例语句中使用：

```
$enable_real = $enable ? { 
  Boolean => $enable, 
  String  => str2bool($enable), 
  default => fail('Unsupported value for ensure. Expected either 
   bool or string.'), 
} 
```

抽象数据类型是有助于更复杂或宽松的类型检查的构造：

+   标量

+   集合

+   变体

+   数据

+   模式

+   枚举

+   元组

+   结构体

+   Optional

+   目录条目

+   类型

+   任意

+   可调用

假设一个参数只接受来自有限集合的字符串。仅检查是否为`String`类型是不足够的。在这种情况下，`Enum`类型很有用；可以为其指定一系列有效值：

```
class ssh (
  Boolean $server = true,
  Enum['des','3des','blowfish'] $cipher = 'des',
){
  if $server {
    include ssh::server
  }
}
```

如果`listen`参数未设置为列出元素之一，Puppet 将抛出错误：

```
class { 'ssh': 
  ciper => 'foo', 
} 
```

显示以下错误：

```
puppet apply ssh.pp
Error: Expected parameter 'ciper' of 'Class[Ssh]' to have type Enum['des','3des','blowfish'] got String at ssh.pp:2 on node puppetmaster.example.net 
```

有时，使用特定数据类型比较困难，因为参数可能被设置为`undef`值。考虑一个可能为空（`undef`）或设置为任意字符串数组的`userlist`参数。

这就是`Optional`类型的作用：

```
class ssh ( 
  Boolean $server = true, 
  Enum['des','3des','blowfish'] $cipher = 'des', 
  Optional[Array[String]] $allowed_users = undef, 
){ 
  if $server { 
    include ssh::server 
  } 
} 
```

再次提醒，使用错误的数据类型会导致 Puppet 错误：

```
class { 'ssh': 
  allowed_users => 'foo', 
} 
```

显示的错误如下：

```
puppet apply ssh.pp
Error: Expected parameter 'userlist' of 'Class[Ssh]' to have type Optional[Array[String]], got String at ssh.pp:2 on node puppetmaster.example.net
```

在前面的示例中，我们使用了数据类型组合。这意味着数据类型可以拥有更多的类型检查信息。

假设我们想在类中设置 `ssh` 服务端口。通常，ssh 应该运行在 1 到 1023 之间的特权端口上。在这种情况下，我们可以通过传递额外信息，将整数数据类型限制为只允许 1 到 1023 之间的数字：

```
class ssh ( 
  Boolean $server = true, 
  Optional[Array[String]] $allowed_users = undef, 
  Integer[1,1023] $sshd_port, 
){ 
  if $server { 
    include ssh::server 
  } 
} 
```

一如既往，提供错误的参数会导致错误：

```
class { 'ssh': 
  sshd_port => 'ssh', 
} 
```

上述代码行给出了以下错误：

```
puppet apply ssh.pp
Error: Expected parameter 'sshd_port' of 'Class[Ssh]' to have type Integer[1, 1023], got String at ssh.pp:2 on node puppetmaster.example.net  
```

使用多种数据类型的复杂哈希，在新类型系统下非常难以描述。

使用 `Hash` 类型时，只能检查一般的哈希，或检查具有特定类型键的哈希。你可以选择性地验证哈希中的最小和最大元素数量。

以下示例提供了一个有效的哈希类型检查：

```
$hash_map = { 
  'ben'     => { 
    'uid'   => 2203, 
    'home'  => '/home/ben', 
  }, 
  'jones'   => { 
    'uid'   => 2204, 
    'home'  => 'home/jones', 
  } 
} 
```

特别地，用户 `jones` 的 `home` 条目缺少前导斜杠：

```
class users ( 
  Hash $users 
){ 
  notify { 'Valid Hash': } 
} 
class { 'users': 
  users => $hash_map, 
} 
```

运行上述代码，将会得到以下输出：

```
puppet apply hash.pp
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.32 seconds
Notice: Valid hash
Notice: /Stage[main]/Users/Notify[Valid hash]/message: defined 'message' as 'Valid hash'
Notice: Applied catalog in 0.03 seconds 
```

使用上述符号，数据类型是有效的，但哈希图内部存在错误。

检查 `Arrays` 或 `Hashes` 的内容需要使用另一种抽象数据类型：`Tuple`（用于 `Arrays`）或 `Struct`（用于 `Hashes`）。

然而，`Struct` 数据类型只会在键名来自已知有限集时工作，这在给定的示例中并不适用。

在这个特殊情况下，我们有两个可能的选择：

+   扩展 `hash` 数据类型以了解哈希的内部结构

+   将 `type` 数据包装成一个定义，使用所有键并利用键函数（来自 `stdlib`）

第一个解决方案如下：

```
class users ( 
  Hash[ 
    String, 
    Struct[ { 'uid' => Integer, 
              'home' => Pattern[ /^\/.*/ ] } ] 
  ] $users 
){ 
  notify { 'Valid hash': } 
} 
```

然而，当数据类型不匹配时，错误信息很难理解：

```
puppet apply hash.pp
Error: Expected parameter 'users' of 'Class[Users]' to have type Hash[String, Struct[{'uid'=>Integer, 'home'=>Pattern[/^\/.*/]}]], got Struct[{'ben'=>Struct[{'uid'=>Integer, 'home'=>String}], 'jones'=>Struct[{'uid'=>Integer, 'home'=>String}]}] at hash.pp:32 on node puppetmaster.example.net
```

第二个解决方案给出了一个更聪明的提示，指示哪些数据可能是错误的：

```
define users::user ( 
  Integer        $uid, 
  Pattern[/^\/.*/] $home, 
){ 
  notify { "User: ${title}, UID: ${uid}, HOME: ${home}": } 
} 
```

该定义类型随后会在用户类中使用：

```
class users ( 
  Hash[String, Hash] $users 
){ 
  $keys = keys($users) 
  each($keys) |String $username| { 
    users::user{ $username: 
      uid  => $users[$username]['uid'], 
      home => $users[$username]['home'], 
    } 
  } 
} 
```

如果在哈希中提交了错误的数据，你将收到以下信息：

错误消息：

```
puppet apply hash.pp
Error: Expected parameter 'home' of 'Users::User[jones]' to have type Pattern[/^\/.*/], got String at hash.pp:23 on node puppetmaster.example.net  
```

错误消息指向用户 `jones` 的 home 参数，该参数在哈希中给出。正确的哈希如下：

```
$hash_map = { 
  'ben'    => { 
    'uid'  => 2203, 
    'home' => '/home/ben', 
  }, 
  'jones'  => { 
    'uid'  => 2204, 
    'home' => '/home/jones', 
  } 
} 
```

上述代码产生了预期的结果，如下所示：

```
puppet apply hash.pp
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.33 seconds
Notice: User: ben, UID: 2203, HOME: /home/ben
Notice: /Stage[main]/Users/Users::User[ben]/Notify[User: ben, UID: 2203, HOME: /home/ben]/message: defined 'message' as 'User: ben, UID: 2203, HOME: /home/ben'
Notice: User: jones, UID: 2204, HOME: /home/jones
Notice: /Stage[main]/Users/Users::User[jones]/Notify[User: jones, UID: 2204, HOME: /home/jones]/message: defined 'message' as 'User: jones, UID: 2204, HOME: /home/jones'
Notice: Applied catalog in 0.03 seconds
```

除了现有的数据类型，Puppet 还提供了基于现有数据类型构建新数据类型的可能性。

在最后一个示例中，我们使用正则表达式来匹配绝对路径，但有时正则表达式可能变得非常复杂且难以理解。这时，类型声明就派上了用场。

类型声明必须是模块的一部分，使用模块命名空间作为前缀，并且必须放置在类型目录中。

```
# stlib/types/absolutepath.pp 
type Stdlib::Absolutepath = Variant[Stdlib::Windowspath, Stdlib::Unixpath] 
```

Windows 和 Unix 路径类型具有适当的正则表达式。

当你想要验证一个非常特定的数据类型时，类型系统非常有用。想想防火墙模块，你可能需要检查 IPv4 或 IPv6 地址。

我们可以使用与 `absolutepath` 数据类型相同的模式：

```
#firewall/types/ipaddress.pp 
type Firewall::Ipaddress = Variant[Firewall::Ipv4, Firewall::Ipv6] 
```

上述清单使用了 `each` 函数，这是 Puppet 4 语言的另一个部分。下一节将更详细地探讨它。

# 学习 lambda 表达式和函数

函数长期以来一直是 Puppet 的一个重要组成部分。由于新的类型系统，基于参数数据类型的不同行为，现已可以实现全新的函数集。

要理解函数，首先我们必须了解 Lambdas，它们是在 Puppet 4 中引入的。Lambdas 代表一段 Puppet 代码片段，可以在函数中使用。从语法上讲，Lambdas 由一个可选类型和至少一个变量组成，可选的默认值，变量被管道符号（`|`）括起来，后跟一段大括号内的 Puppet 代码：

```
$packages = ['htop', 'less', 'vim']
each($packages) |String $package| 
{
      package { $package:
        ensure => latest,
  }
}
```

Lambda 通常用于函数。前面的示例在 `$packages` 变量上使用了 `each` 函数，遍历其内容，在每次迭代中，将 lambda 变量 `$package` 设置为 `htop`、`less` 和 `vim`，分别对应。Puppet 代码块随后在资源类型声明中使用 lambda 变量。

括号中的 Puppet 代码必须确保不会发生重复的资源声明。

由于 Puppet 现在了解数据类型，你可以以更优雅的方式与变量及其内部数据进行交互和操作。

用于数组和哈希的函数：

+   Puppet 4 提供了一整套内建的数组和哈希函数：each

+   slice

+   filter

+   map

+   reduce

+   与

我们已经看到 `each` 函数的实际应用。在 Puppet 4 之前，必须将所需的 Puppet 资源类型包装在 `define` 中，并使用数组声明 `define` 类型：

```
class puppet_symlinks { 
  $symlinks = [ puppet', 'facter', 'hiera' ] 
  puppet_symlinks::symlinks { $symlinks: } 
} 

define puppet_symlinks::symlinks { 
  file { "/usr/local/bin/${title}": 
    ensure => link, 
    target => "/opt/puppetlabs/bin/${title}", 
  } 
} 
```

通过这个概念，动作（创建 `symlink`）被放入一个定义类型中，不再直接出现在清单中。新的迭代方法保持了动作的位置：

```
class puppet_symlinks { 
  $symlinks = [ 'puppet', 'facter', 'hiera' ] 
  $symlinks.each | String $symlink | { 
    file { "/usr/local/bin/${symlink}": 
      ensure => link, 
      target => "/opt/puppetlabs/bin/${symlink}", 
    } 
  } 
} 
```

你是否注意到，这次我们使用了另一种通过函数的方式？在第一个示例中，我们使用了 Puppet 3 风格的函数调用：

```
function($variable) 
```

Puppet 4 还支持后缀表示法，其中函数通过点（`.`）附加到其参数上：

```
$variable.function 
```

Puppet 4 支持使用函数的两种方式。这使你可以继续遵循自己的编码风格，并利用新功能。

让我们回顾一下其他用于数组和哈希的函数：

+   `slice` 函数允许你拆分和分组一个数组或哈希。它需要一个额外的参数（整数），用于定义要分组的对象数量：

```
$array = [ '1', '2', '3', '4'] 
$array.slice(2) |$slice| { 
  notify { "Slice: ${slice}": } 
} 
```

+   这段代码将产生以下输出：

```
    Notice: Slice: [1, 2]
    Notice: Slice: [3, 4]
```

+   在对哈希使用 slice 函数时，会得到键

    （根据分组的键数量）并相应地，

    子哈希：

```
$hash = { 
  'key 1' => {'value11' => '11', 'value12' => '12',}, 
  'key 2' => {'value21' => '21', 'value22' => '22',}, 
  'key 3' => {'value31' => '31', 'value32' => '32',}, 
  'key 4' => {'value41' => '41', 'value42' => '42',}, 
} 

$hash.slice(2) |$hslice| { 
  notify { "HSlice: ${hslice}": } 
} 
```

+   这将返回以下输出：

```
 Notice: HSlice: [[key1, {value11 => 11, value12 => 
  12}], 
  [key2, {value21 => 21, value22 => 22}]]
 Notice: HSlice: [[key3, {value31 => 31, value32 => 
  32}], 
  [key4, {value41 => 41, value42 => 42}]]
```

+   `filter` 函数可以用来过滤数组或哈希中的特定条目。

+   在对数组使用时，所有元素都会传递到代码块中，代码块会判断该项是否匹配。如果你想筛选出数组中的某些项（例如，应该安装的包），这非常有用：

```
$pkg_array = [ 'libjson', 'libjson-devel', 'libfoo', 'libfoo-devel' ] 
$dev_packages = $pkg_array.filter |$element| { 
  $element =~ /devel/ 
} 
notify { "Packages to install: ${dev_packages}": } 
```

+   这将返回以下输出：

```
 Notice: Packages to install: [libjson-devel, libfoo-
 devel]
```

+   哈希的行为有所不同。在使用哈希时，必须提供两个 lambda 变量：`key`和`value`。你可能只想添加具有特定`gid`设置的用户：

```
$hash = { 
  'jones' => { 
    'gid' => 'admin', 
  }, 
  'james' => { 
    'gid' => 'devel', 
  }, 
  'john'  => { 
    'gid' => 'admin', 
  }, 
} 

$user_hash = $hash.filter |$key, $value| { 
  $value['gid'] =~ /admin/ 
} 
$user_list = keys($user_hash) 
notify { "Users to create: ${user_list}": } 
```

+   这将只返回`admin` gid 下的用户：

```
    Notice: Users to create: [jones, john]
```

# 创建 Puppet 4 函数

Puppet 3 函数 API 存在一些限制，并且缺少一些功能。Puppet 4 中的新函数 API 在这一点上有了显著改进。

旧版函数的部分限制如下：

+   这些函数没有自动类型检查。

+   由于平坦的命名空间，这些函数必须具有唯一的名称。

+   这些函数不是私有的，因此可以在任何地方使用。

+   如果不运行 Ruby 代码，无法获取文档。

在 Puppet 3 中，函数必须位于`lib/puppet/parser/functions`目录中的模块中。因此，人们称这些函数为**解析器函数**，但这个名称是误导性的。函数与 Puppet 解析器无关。

在 Puppet 4 中，函数必须放在路径`lib/puppet/functions`中的模块中。

这是创建一个返回 Puppet 主机名的函数的方式：

```
# modules/utils/lib/puppet/functions/resolver.rb 
require 'socket' 
Puppet::Functions.create_function(:resolver) do 
  def resolver() 
    Socket.gethostname 
  end 
end 
```

使用`dispatch`可以对属性进行类型检查。根据所需的功能，可能会有多个`dispatch`块（检查不同的数据类型）。每个`dispatch`可以引用函数内部的另一个已定义的 Ruby 方法。通过为`dispatch`和 Ruby 方法使用相同的名称，可以实现这种引用。

以下示例代码应增加附加功能；根据参数类型，函数应该返回本地系统的主机名，或使用 DNS 根据 IPv4 地址或给定主机名的`ipaddress`获取主机名：

```
require 'resolv' 
require 'socket' 
Puppet::Functions.create_function(:resolver) do 
  dispatch :ip_param do 
     param 'Pattern[/^(?:[0-9]{1,3}\.){3}[0-9]{1,3}$/]', :ip 
  end 
  dispatch :fqdn_param do 
     param 'Pattern[/^([a-z0-9\.].*$/]', :fdqn 
  end 
  dispatch :no_param do 
  end 

  def no_param 
    Socket.gethostname 
  end 
  def ip_param(ip) 
    Resolv.getname(ip) 
  end 
  def fqdn_param(fqdn) 
    Resolv.getaddress(fqdn) 
  end 
end 
```

在文件的开头，我们需要加载一些 Ruby 模块，以允许 DNS 名称解析并找到本地主机名。

前两个`dispatch`部分检查参数值的数据类型并设置唯一的符号。最后一个`dispatch`部分不检查数据类型，这与没有提供参数时相符。

每个已定义的 Ruby 方法使用相应的`dispatch`名称，并根据参数类型执行 Ruby 代码。

现在，解析器函数可以通过三种不同的方式在 Puppet 清单代码中使用：

```
class resolver { 
  $localname = resolver() 
  notify { "Without argument resolver returns local 
  hostname: 
  ${localname}": } 

  $remotename = resolver('puppetlabs.com') 
  notify { "With argument puppetlabs.com: ${remotename}": 
  } 

  $remoteip = resolver('8.8.8.8') 
  notify { "With argument 8.8.8.8: ${remoteip}": } 
  } 
```

声明此类时，以下输出将会显示：

```
puppet apply -e 'include resolver'
Notice: Compiled catalog for puppetmaster.example.net in environment production in 0.35 seconds
...
Notice: Without argument resolver returns local hostname: puppetmaster
Notice: With argument puppetlabs.com: 52.10.10.141
Notice: With argument 8.8.8.8: google-public-dns-a.google.com
Notice: Applied catalog in 0.04 seconds
```

在 Puppet 3 的函数中，无法有两个相同名称的函数。在使用新模块时，必须检查是否出现了重复的函数。

Puppet 4 函数现在提供了与类相同的命名空间功能。

让我们将函数迁移到类命名空间中：

```
# modules/utils/lib/puppet/functions/resolver/resolve.rb 
require 'resolv' 
require 'socket' 
Puppet::Functions.create_function(:'resolver::resolve') do 
  # the rest of the function is identical to the example given 
   # above 
end 
```

在示例中，代码需要位于`resolver/lib/puppet/functions/resolver/resolve.rb`，对应于`function name: 'resolver::resolve'`。

带命名空间的函数按常规方式调用：

```
class resolver { 
  $localname = resolver::resolve() 
  $remotename = resolver::resolve('puppetlabs.com') 
  $remoteip = resolver::resolve('8.8.8.8') 
} 
```

# 利用新的模板引擎

在第六章，*《Puppet 初学者进阶部分》*中，我们介绍了模板和 ERB 模板引擎。在 Puppet 4 中，增加了一个替代方案：EPP 模板引擎。模板引擎之间的主要区别如下：

+   在 ERB 模板中，不能使用 Puppet 语法指定变量（`$variable_name`）

+   ERB 模板不接受参数

+   在 EPP 模板中，你将使用 Puppet DSL 语法，而不是 Ruby 语法

EPP 模板引擎需要模块中的作用域变量：

```
# motd file - managed by Puppet 
This system is running on <%= $::operatingsystem %> 
```

清单定义了以下局部变量：`<%= $motd::local_variable %>`。EPP 模板还具有一个独特的扩展：它们可以接受类型化参数。

为了使用此功能，模板必须以参数声明块开始：

```
<%- | String $local_variable, 
      Array  $local_array 
| -%> 
```

这些参数与 Puppet 清单中的变量不同。相反，必须使用 `epp` 函数传递参数：

```
epp('template/test.epp', {'local_variable' => 'value', 'local_array' => ['value1', 'value2'] }) 
```

没有参数的模板应该仅在模板仅由一个模块使用时使用，这样可以安全地依赖 Puppet 变量来定制内容。

当模板从多个地方使用时，推荐使用带参数的 EPP 模板函数。通过在开始时声明参数，特别清楚模板需要哪些数据。

在遍历数组和哈希时，模板引擎之间有一个具体的区别。ERB 语法使用 Ruby 代码和不受限的局部变量，而 EPP 语法则需要指定 Puppet DSL 代码：

```
# ERB syntax 
<% @array.each do |element| -%> 
<%= element %> 
<% end -%> 

# EPP syntax 
<% $array.each |$element| { -%> 
<%= $element %> 
<% } -%> 
```

内联 ERB 函数也通过内联 EPP 进行了补充。使用内联 EPP，可以指定一小段 EPP 代码进行求值：

```
file {'/etc/motd': 
  ensure  => file, 
  content => inline_epp("Welcome to <%= $::fqdn %>\n") 
} 
```

在 Puppet 4 之前，传递多于一个小代码片段比较不方便。随着 Puppet 4 和 HEREDOC 支持的发布，结合 `inline_epp`，复杂模板变得更加容易且可读性更好。

# 使用 HEREDOC 处理多行

在 Puppet 中编写多行文件片段大多导致代码难以阅读，主要是因为缩进问题。随着 Puppet 4 的发布，增加了 `heredoc` 风格。现在可以指定一个 `heredoc` 标签和标记：

```
$motd_content = @(EOF) 
  This system is managed by Puppet 
  local changes will be overwritten by next Puppet run. 
EOF
```

`heredoc` 标签以 `@` 符号开始，后跟括号中包含的任意字符串。`heredoc` 标记即为标签中给定的字符串。

如果在 `heredoc` 文档中需要变量，可以通过将标签字符串放入双引号来启用变量插值。`heredoc` 中的变量与 Puppet DSL 变量书写方式相同：一个美元符号，后跟作用域和变量名：

```
$motd_content = @("EOF") 
  Welcome to ${::fqdn}. 
  This system is managed by Puppet version ${::puppetversion}. 
  Local changes will be overwritten by the next Puppet run 
EOF
```

通常，`heredoc` 不处理转义序列。转义序列需要显式启用。从 Puppet 4.2 开始，`heredoc` 提供了以下转义序列：

+   `*` `\n` 换行符

+   `*` `\r` 回车符

+   `*` `\t` 制表符

+   `*` `\s` 空格

+   `*` `\$` 字面量美元符号（防止插值）

+   `*` `\u` Unicode 字符

+   `\L` 什么都不做（忽略源代码中的换行符）

启用的转义序列必须放在`heredoc`标签字符串的后面：

```
$modt_content = @("EOF"/tn) 
Welcome to ${::fqdn}.\n\tThis system is managed by Puppet version ${::puppetversion}.\n\tLocal changes will be overwritten on next Puppet run. 
EOF
```

在这个例子中，文本总是从第一列开始，这使得它很难阅读，并且与周围的代码（通常会有一些空白缩进）显得格格不入。

可以通过在`heredoc`标记前放置空格和管道符号来去除缩进。管道符号将指示每行的第一个字符：

```
$motd_content = @("EOF")
    Welcome to ${::fqdn}.
    This system is managed by Puppet version ${::puppetversion}.
    Local changes will be overwritten on next Puppet run.
    | EOF
```

现在可以轻松地将`heredoc`和`inline_epp`结合使用：

```
class my_motd (
  Optional[String] $additional_content = undef
){
  $motd_content = @(EOF)
    Welcome to <%= $::fqdn %>.
    This system is managed by Puppet version 
    <%= $::puppetversion %>.
    Local changes will be overwritten on next Puppet run.
    <% if $additional_content != undef { -%>
    <%= $additional_content %>
    <% } -%>
    | EOF
  file { '/etc/motd':
    ensure  => file,
    content => inline_epp($motd_content, {  
    additional_content => $additional_content } ),
  }
} 
```

声明此类会在`motd`文件中生成以下结果：

```
puppet apply -e 'include my_motd'
Welcome to puppetmaster.example.net.
This system is managed by Puppet version 4.2.1.
Local changes will be overwritten on next Puppet run.  
```

在使用`heredoc`和`inline_epp`组合时，你需要小心不要引用`heredoc`起始标签。否则，变量替换会发生在`inline_epp`函数调用之前。

# 使用 Puppet 5 服务器度量

在 Puppetserver 5 版本中，之前 Puppet Enterprise 的度量系统已被移植到 Puppet Open Source 中。

度量系统允许你从 JMX 控制台读取内部信息，如编译时间、文件服务状态和函数运行时，或者将数据推送到 graphite 系统。

启用度量系统非常简单，只需编辑位于`/etc/puppetlabs/puppetserver/conf.d/metrics.conf`的 puppet 服务器`metrics.conf`文件。

有三个重要的设置。在`metrics.server-id`中，可以指定一个 ID，该 ID 稍后会在 Grafana 仪表板中使用。`metrics.registry.puppetserver.reporters.graphite.enabled`值必须设置为 true，同时`metrics.reporters.graphite`哈希值必须包含 graphite 的主机名和端口，以及更新间隔设置：

```
# settings related to metrics
metrics: {
  # a server id that will be used as part of the namespace 
  for metrics produced
  # by this server
  server-id: "puppet.bi.example42.com"
  registries: {
    puppetserver: {
      # specify metrics to allow in addition to those in 
      the default list
      #metrics-allowed: ["compiler.compile.production"]
      # enable or disable JMX metrics reporter
      jmx: {
        enabled: true
      }
      # enable or disable Graphite metrics reporter
      graphite: {
        enabled: true
      }
    }
  }
  # this section is used to configure settings for 
  reporters that will send
  # the metrics to various destinations for external 
  viewing
  reporters: {
    graphite: {
      host: "10.0.4.2"
      port: "2003"
      update-interval-seconds: 5
    }
  }
  metrics-webservice: {
    jolokia: {
      # Enable or disable the Jolokia-based metrics/v2 
      endpoint.
      # Default is true.
      # enabled: false
      # Configure any of the settings listed at:
      # 
      https://jolokia.org/reference/html/agents.html#war-
      agent-installation
      servlet-init-params: {
        # Specify a custom security policy:
        # https://jolokia.org/reference/html/security.html
        # policyLocation: 
        "file:///etc/puppetlabs/puppetserver/jolokia-
        access.xml"
      }
    }
  }
}
```

在做完更改后，别忘了重新启动`puppetserver`进程。

对于自动化设置，应该使用`puppetlabs-hocon`模块，它能够设置所有所需的值：

```
Hocon_setting {
  path   => 
  '/etc/puppetlabs/puppetserver/conf.d/metrics.conf',
  notify => Service['puppetserver'],
  }
  hocon_setting {'server metrics server-id':
    ensure  => present,
    setting => 'metrics.server-id',
    value   => 'localhost',
  }
  hocon_setting {'server metrics reporters graphite':
    ensure  => present,
    setting => 
 'metrics.registries.puppetserver.reporters.graphite.enabled',
    value   => true,
  }
  hocon_setting {'server metrics graphite host':
    ensure  => present,
    setting => 'metrics.reporters.graphite.host',
    value   => $graphite_server,
  }
  hocon_setting {'server metrics graphite port':
    ensure  => present,
    setting => 'metrics.reporters.graphite.port',
    value   => 2003,
  }
  hocon_setting {'server metrics graphite update 
  interval':
    ensure  => present,
    setting => 'metrics.reporters.graphite.update-
    interval-seconds',
    value   => 5,
  } 
```

本书不涉及 graphite 的设置。可以使用一个 Puppet 模块进行评估目的（[`github.com/tuxmea/puppet-grafanadash`](http://github.com/tuxmea/puppet-grafanadash)）。

该模块会在 CentOS 7 系统上安装并配置 elasticsearch、graphite 和 grafana。

你只需要在节点上包含`grfanadash::dev`类：

```
node 'graphite.example.com' {   include grafanadash::dev }
```

之后，你可以通过`http://graphite.example.com`访问 graphite，通过`http://graphite.example-com:10000`访问 grafana。在 grafana 仪表板中，可以从模块示例文件夹加载`.json`文件。

请注意，可能需要最多 10 分钟，之前的值才能出现在 grafana 中。

# 打破旧有的做法

当 Puppet Labs 决定对解析器和新特性进行改进时，他们也决定移除一些已经在多个版本中弃用的功能。

# 转换节点继承

在旧版本的 Puppet 中，节点继承被视为一种良好的实践。为了避免在节点级别上写太多代码，创建了一个通用的、虚拟的主机（`basenode`），而真实的节点继承自`basenode`：

```
node basenode {
  include security
  include ldap
  include base
}
node 'www01.example.net' inherits 'basenode' {
  class { 'apache': }
  include apache::mod::php
  include webapplication
}
```

Puppet 4 不再支持此节点分类。

从 2012 年开始，角色和配置模式变得越来越流行，带来了新的方法来实现智能节点分类。从技术角度看，角色和配置是 Puppet 类。配置模块封装了技术模块，并通过提供如 `ntp` 服务器和 `ssh` 配置选项等数据，来将它们的使用适应现有的基础设施。角色模块描述了系统的业务用例，并使用已声明的配置：

```
class profile::base { 
  include security 
  include ldap 
  include base 
} 
class profile::webserver { 
  class { 'apache': } 
  include apache_mod_php 
} 

class role::webapplication { 
  include profile::base 
  include profile::webserver 
  include profile::webapplication 
} 

node 'www01.example.net' { 
  include role::webapplication 
} 
```

最后一章将更详细地描述角色和配置模式。

# 处理字符串上的布尔代数

一个影响巨大的小变动是空字符串比较的变化。在 Puppet 4 之前，可以通过检查变量来测试未设置的变量或包含空字符串的变量：

```
class ssh ( 
  $server = true, 
){
  if $server {...} 
} 
```

在以下不同的声明中，`ssh` 类的行为类似（$server 评估为 true）：

```
include ssh 
class { 'ssh': server => 'yes', } 
```

禁用 `ssh` 类中的服务器部分可以通过以下类声明实现：

```
class { 'ssh': server => false, } 
class { 'ssh': server => '', } 
```

最后一个示例（空字符串）的行为在 Puppet 4 中发生了变化。空字符串现在在布尔上下文中等于 true，就像在 Ruby 中一样。如果你的代码使用了这种变量检查方式，你需要添加空字符串检查，以保持与 Puppet 4 一致的行为：

```
class ssh ( 
  $server = true, 
){ 
  if $server and $server != '' {...} 
} 
```

# 使用严格的变量命名

变量有时看起来与常量相同，并展示以下特性：

+   变量不能再次声明

+   在一个节点的作用域内，大多数变量是静态的（`hostname`、`fqdn` 等等）

有时，开发者更喜欢使用大写字母书写变量，这是出于之前提到的原因，使它们看起来与 Ruby 常量相同。

在 Puppet 4 中，变量名不能以大写字母开头：

```
class variables 
{ 
  $Local_var = 'capital variable' 
  notify { "Local capital var: ${Local_var}": } 
} 
```

声明此类将产生以下错误信息：

```
root@puppetmaster:/etc/puppetlabs/code/environments/production/modules# puppet apply -e 'include variables'
Error: Illegal variable name, The given name 'Local_var' does not conform to the naming rule /^((::)?[a-z]\w*)*((::)?[a-z_]\w*)$/ at /etc/puppetlabs/code/environments/production/modules/variables/manifests/init.pp:3:3 on node puppetmaster.example.net 
```

# 学习新的引用语法

由于类型系统以及 Puppet 4 现在将一切视为表达式，因此必须正确命名对其他声明资源的引用。引用现在有一些严格的规定：

+   引用类型和开括号之间不能有空格

+   引用标题（不带引号使用）不得以大写字母拼写

以下内容将在 Puppet 4 中产生错误：

```
User [Root] 
User[Root]
```

从 Puppet 4 开始，引用必须按照以下模式编写：

```
Type['title']
```

我们的示例需要更改为：

```
User['root'] 
```

# 清理名称中的连字符

许多模块（即使是在模块 Forge 上）都使用了连字符

模块名称。连字符现在不再是字符串字符，而是数学运算符（减法）。因此，现在严格禁止在以下描述符中使用连字符：

+   模块名称

+   任何类名

+   已定义类型的名称

使用带有连字符的模块时，需要删除连字符或将其替换为字符串字符（例如下划线）。

这在旧版本中是可行的，如下所示：

```
class syslog-ng {...} 

include syslog-ng 
```

现在，新风格如下：

```
class syslog_ng { 
  ... 
} 

include syslog_ng 
```

# 不再使用 Ruby DSL

一些人利用了将 `.rb` 文件作为模块中的清单文件的可能性。这些 `.rb` 文件包含 Ruby 代码，主要用于处理数据。Puppet 4 现在有了数据类型，使得这一做法变得过时。

Puppet 4 移除了对这些 Ruby 清单的支持。

# 相对类名解析

在 Puppet 3 及更早版本中，如果本地类名与另一个模块相同，必须指定绝对类名：

```
# in module "mysql" 
class mysql { 
  ... 
} 
# in module "application" 
class application::mysql { 
  include mysql 
} 
```

在 `application::` 命名空间的范围内，Puppet 3 会在该命名空间中搜索 `mysql` 类进行包含。实际上，`application::mysql` 类会将自身包含进去。然而，这并不是我们想要的效果。我们其实是想找 `mysql` 模块。作为变通方法，大家都被鼓励指定 `mysql` 模块类的绝对路径：

```
class application::mysql { 
  include ::mysql 
} 
```

这种相对名称解析在 Puppet 4 中不再适用。原始示例现在可以正常工作。

# 处理不同数据类型

因为 Puppet 3 并不知道不同的数据类型

（大多数情况下，一切都被视为字符串类型），所以可以将多个不同的数据类型组合在一起。

Puppet 4 现在对组合不同数据类型非常严格。最简单的例子是处理浮动类型和整数类型；当将浮动数与整数相加时，结果将是浮动数类型。

对不同数据类型（如字符串和布尔值）进行操作将导致错误。以下代码将正常工作：

```
case $::operatingsystemmajrelease { 
  '8': { 
    include base::debian::jessie 
  } 
} 
```

另一方面，以下代码将无法工作：

```
if $::operatingsystemmajrelease > 7 { 
  include base::debian::jessie 
} 
```

您将收到以下错误消息：

```
Error: Evaluation Error: Comparison of: String > Integer, is not possible. Caused by 'A String is not comparable to a non String' 
```

仔细检查不同 Facter 变量的比较。某些 Facter 变量，如 `operatingsystemmajrelease`，返回字符串类型的数据；而 `processorcount` 返回整数值。

# 总结

升级到 Puppet 3 应按逐步程序进行，在此过程中，您的现有代码将使用 Puppet 3.8 和新的解析器进行评估。

多亏了类型系统，现在可以在 Puppet DSL 代码中以更加优雅的方式处理数据。新的函数 API 允许您通过使用命名空间，立即识别一个函数属于哪个模块。通过使用 `dispatch` 方法和数据类型，类似的函数现在可以在一个文件内结合使用，从而实现函数重载。

新的 EPP 模板通过使用 Puppet 语法进行变量引用，提供了更好的变量来源理解。将参数传递给模板将允许您以更灵活的方式使用模块。

结合 EPP 模板和 HEREDOC 语法，将允许您将模板代码和数据直接显示在类内。

在接下来的章节中，您将了解 Hiera 以及它如何帮助您为可扩展的 Puppet 代码库带来秩序。

# 第五章：变量、表达式和事实

|   | *无法开始学习自己认为已经掌握的知识。* |   |
| --- | --- | --- |
|   | --*埃皮克泰图斯* |

在本章中，你将学习 Puppet 变量和数据类型、表达式以及条件语句。你还将学习如何使用 Facter 获取有关节点的数据，了解最重要的标准事实，并查看如何创建你自己的外部事实。最后，你将使用 Puppet 的 `each` 函数迭代数组和哈希，包括 Facter 数据。

![变量、表达式和事实](img/B08880_05_01.jpg)

# 引入变量

在 Puppet 中，**变量**仅仅是给特定值命名的一种方式，这样我们就可以在需要使用字面值的地方使用该变量（`variable_string.pp`）：

```
$php_package = 'php7.0-cli'

package { $php_package:
  ensure => installed,
}
```

美元符号（`$`）告诉 Puppet 后面跟的是变量名。变量名必须以小写字母或下划线开头，尽管变量名的其余部分也可以包含大写字母或数字。

变量可以包含不同类型的数据；其中一种类型是**字符串**（如 `php7.0-cli`），但 Puppet 变量也可以包含**数字**或**布尔**值（`true` 或 `false`）。以下是一些示例（`variable_simple.pp`）：

```
$my_name = 'Zaphod Beeblebrox'
$answer = 42
$scheduled_for_demolition = true
```

## 使用布尔值

字符串和数字是直观的，但 Puppet 还有一种特殊的数据类型来表示真假值，我们称之为**布尔**值，取名自逻辑学家乔治·布尔。我们在 Puppet 资源属性中已经遇到了一些布尔值（`service.pp`）：

```
service { 'sshd':
  ensure => running,
  enable => true,
}
```

布尔变量唯一允许的值是字面值 `true` 和 `false`，但是布尔变量也可以持有条件表达式的值（值为 `true` 或 `false` 的表达式），我们将在本章稍后探讨。

### 提示

你可能会想知道在之前的示例中，`running` 的值是什么类型。实际上，它是一个字符串，但它是一个特殊的、未加引号的字符串，称为**裸字**。虽然如果在这里使用正常的加引号字符串 `'running'` 对 Puppet 来说是完全一样的，但对于只能是少数几个词之一的属性值（例如，服务的 `ensure` 属性只能取值 `running` 或 `stopped`），使用裸字被认为是良好的风格。相比之下，`true` 不是裸字，而是布尔值，并且不能与字符串 `'true'` 互换。布尔值应始终使用未加引号的字面量值 `true` 或 `false`。

## 在字符串中插值变量

如果你无法再将变量中的内容取出来，那么将某些内容存储在变量中也没什么用。最常见的使用变量值的方式之一就是将其**插值**到字符串中。当你这样做时，Puppet 会将变量的当前值插入到字符串内容中，替换掉变量的名称。字符串插值看起来像这样（`string_interpolation.pp`）：

```
$my_name = 'John'
notice("Hello, ${my_name}! It's great to meet you!")
```

当你应用此清单时，以下输出将被打印：

```
Notice: Scope(Class[main]): Hello, John! It's great to meet you!
```

要在字符串中插入（即插值）变量的值，可以在变量名之前加上 `$` 符号，并用大括号（`{}`）将其括起来。这告诉 Puppet 用变量的值替换字符串中的变量名。

### 提示

我们在之前的示例中悄悄添加了一个新的 Puppet 函数`notice()`。它对系统没有影响，但会打印出其参数的值。这在故障排除或查看在清单中的某个特定时刻变量的值时非常有用。

## 创建数组

变量也可以包含多个值。**数组**是值的有序序列，每个值可以是任何类型。以下示例创建了一个 **整数** 类型的数组（`variable_array.pp`）：

```
$heights = [193, 120, 181, 164, 172]

$first_height = $heights[0]
```

你可以通过在方括号中给出索引号来引用数组的任何单个元素，其中第一个元素的索引是`[0]`，第二个是`[1]`，依此类推。（如果你觉得这很混乱，你并不孤单，但可以试着将索引看作是数组起始位置的偏移量。自然地，第一项的偏移量是 0。）

## 声明资源数组

你已经知道，在 Puppet 资源声明中，资源的标题通常是一个字符串，比如文件的路径或软件包的名称。你可能会问：“如果你为资源提供一个字符串数组作为标题，而不是单个字符串，会发生什么呢？Puppet 会为数组中的每个元素创建多个资源吗？”让我们尝试一个实验，正好用一个包含软件包名称的数组来做这个实验，看看会发生什么（`resource_array.pp`）：

```
$dependencies = [
  'php7.0-cgi',
  'php7.0-cli',
  'php7.0-common',
  'php7.0-gd',
  'php7.0-json',
  'php7.0-mcrypt',
  'php7.0-mysql',
  'php7.0-soap',
]

package { $dependencies:
  ensure => installed,
}
```

如果我们的直觉是对的，应用之前的清单应该会为 `$dependencies` 数组中列出的每个软件包创建一个软件包资源，并且每个软件包都会被安装。以下是应用清单时发生的情况：

```
sudo apt-get update
sudo puppet apply /examples/resource_array.pp
Notice: Compiled catalog for ubuntu-xenial in environment production in 0.68 seconds
Notice: /Stage[main]/Main/Package[php7.0-cgi]/ensure: created
Notice: /Stage[main]/Main/Package[php7.0-cli]/ensure: created
Notice: /Stage[main]/Main/Package[php7.0-common]/ensure: created
Notice: /Stage[main]/Main/Package[php7.0-gd]/ensure: created
Notice: /Stage[main]/Main/Package[php7.0-json]/ensure: created
Notice: /Stage[main]/Main/Package[php7.0-mcrypt]/ensure: created
Notice: /Stage[main]/Main/Package[php7.0-mysql]/ensure: created
Notice: /Stage[main]/Main/Package[php7.0-soap]/ensure: created
Notice: Applied catalog in 56.98 seconds
```

将一个字符串数组作为资源的标题会导致 Puppet 创建多个资源，除了标题不同之外，其他都相同。你不仅可以对软件包这么做，还可以对文件、用户，或者任何类型的资源这么做。在第六章，*使用 Hiera 管理数据*中，我们将看到一些更复杂的从数据创建资源的方式。

### 提示

为什么我们在应用清单之前运行了`sudo apt-get update`？这是 Ubuntu 命令，用于从上游服务器更新系统的本地软件包目录。在安装任何软件包之前运行此命令是一个好主意，以确保你安装的是最新版本。当然，在你的生产 Puppet 代码中，你可以通过 `exec` 资源来运行它。

## 理解哈希

**哈希**，在某些编程语言中也被称为字典，类似于数组，但不同于仅仅是值的序列，每个值都有一个名称（`variable_hash.pp`）：

```
$heights = {
  'john'    => 193,
  'rabiah'  => 120,
  'abigail' => 181,
  'melina'  => 164,
  'sumiko'  => 172,
}

notice("John's height is ${heights['john']}cm.")
```

每个值的名称被称为**键**。在前面的示例中，这个哈希的键是`john`、`rabiah`、`abigail`、`melina`和`sumiko`。要查找给定键的值，你可以在哈希名称后面用方括号括住键：`$heights['john']`。

### 提示

**Puppet 风格说明**

你是否注意到前面示例中最后一个哈希键值对和数组最后一个元素后的逗号？尽管逗号不是严格要求的，但添加逗号是一种良好的编码风格。原因是当你需要向数组或哈希中添加新项时，如果最后一个项已经有逗号，就不用记得在扩展列表时再添加一个逗号。

## 从哈希中设置资源属性

你可能已经注意到，哈希看起来非常像资源的属性：它是名称和值之间的一对一映射。如果在声明资源时，我们可以直接指定一个包含所有属性及其值的哈希，那该有多方便？事实证明，你可以这样做（`hash_attributes.pp`）：

```
$attributes = {
  'owner' => 'ubuntu',
  'group' => 'ubuntu',
  'mode'  => '0644',
}

file { '/tmp/test':
  ensure => present,
  *      => $attributes,
}
```

`*`字符，欢快地被称为**属性拆分运算符**，告诉 Puppet 将指定的哈希视为要应用于资源的属性值对列表。这完全等同于直接指定相同的属性，如以下示例所示：

```
file { '/tmp/test':
  ensure => present,
  owner  => 'vagrant',
  group  => 'vagrant',
  mode   => '0644',
}
```

# 引入表达式

变量不是 Puppet 中唯一具有值的事物。表达式也有值。最简单的表达式只是字面值：

```
42
true
'Oh no, not again.'
```

你可以将数字值与算术运算符（如`+`、`-`、`*`和`/`）结合起来，创建**算术** **表达式**，它们有一个数值，并且可以用来让 Puppet 进行计算（`expression_numeric.pp`）：

```
$value = (17 * 8) + (12 / 4) - 1
notice($value)
```

最有用的表达式是那些计算结果为`true`或`false`的表达式，这些被称为**布尔表达式**。以下是一些布尔表达式的例子，它们的结果都为`true`（`expression_boolean.pp`）：

```
notice(9 < 10)
notice(11 > 10)
notice(10 >= 10)
notice(10 <= 10)
notice('foo' == 'foo')
notice('foo' in 'foobar')
notice('foo' in ['foo', 'bar'])
notice('foo' in { 'foo' => 'bar' })
notice('foo' =~ /oo/)
notice('foo' =~ String)
notice(1 != 2)
```

## 认识 Puppet 的比较运算符

在前面的示例中，所有布尔表达式中的运算符被称为**比较运算符**，因为它们用来比较两个值。结果是`true`或`false`。以下是 Puppet 提供的比较运算符：

+   `==` 和 `!=`（等于，不等于）

+   `>`、`>=`、`<`和`<=`（大于、大于或等于、小于、小于或等于）

+   `A in B`（`A`是`B`的子字符串，`A`是数组`B`的元素，或`A`是哈希`B`的键）

+   `A =~ B`（`A`被正则表达式`B`匹配，或`A`是数据类型`B`的值。例如，表达式`'hello' =~ String`为`true`，因为值`'hello'`的类型是 String。）

## 引入正则表达式

`=~`操作符尝试将给定值与**正则表达式**进行匹配。正则表达式（在这里是指构成模式或规则的“常规”表达式）是一种特殊类型的表达式，用于指定一组字符串。例如，正则表达式`/a+/`描述了包含一个或多个连续`a`字符的所有字符串：`a`、`aa`、`aaa`，依此类推，以及包含这样的字符序列的其他字符字符串。正则表达式在 Puppet 中由斜杠字符`//`限定。

当我们说正则表达式**匹配**某个值时，我们指的是该值是正则表达式指定的一组字符串中的一个。例如，正则表达式`/a+/`将匹配字符串`aaa`或字符串`Aaaaargh!`。

以下示例展示了一些正则表达式，它们匹配字符串`foo`（`regex.pp`）：

```
$candidate = 'foo'
notice($candidate =~ /foo/) # literal
notice($candidate =~ /f/)   # substring
notice($candidate =~ /f.*/) # f followed by zero or more characters
notice($candidate =~ /f.o/) # f, any character, o
notice($candidate =~ /fo+/) # f followed by one or more 'o's
notice($candidate =~ /[fgh]oo/) # f, g, or h followed by 'oo'
```

### 提示

正则表达式或多或少是用于表达字符串模式的标准语言。它是一种复杂而强大的语言，确实值得一本书来深入探讨（也确实有几本书），但暂时可以简单了解，Puppet 的正则表达式语法与 Ruby 语言使用的语法相同。你可以在 Ruby 文档中了解更多信息：

[`ruby-doc.org/core/Regexp.html`](http://ruby-doc.org/core/Regexp.html)

## 使用条件表达式

布尔表达式，如前面示例中的那些，十分有用，因为我们可以利用它们在 Puppet 清单中做出选择。只有在满足某个条件时，我们才会应用特定的资源，或者我们可以根据某个表达式是否为真来给某个属性赋予不同的值。以这种方式使用的表达式称为**条件表达式**。

## 使用`if`语句做决策

条件表达式最常见的用法是在`if`语句中。以下示例演示了如何使用`if`来决定是否应用资源（`if.pp`）：

```
$install_perl = true
if $install_perl {
  package { 'perl':
    ensure => installed,
  }
} else {
  package { 'perl':
    ensure => absent,
  }
}
```

你可以看到，布尔变量`$install_perl`的值决定了是否安装`perl`包。如果`$install_perl`为`true`，Puppet 将应用以下资源：

```
  package { 'perl':
    ensure => installed,
  }
```

另一方面，如果`$install_perl`为`false`，应用的资源将是：

```
  package { 'perl':
    ensure => absent,
  }
```

你可以使用`if`语句来控制任何数量资源的应用，甚至控制 Puppet 清单的任何部分。如果你愿意，可以省略`else`子句；在这种情况下，当条件表达式的值为`false`时，Puppet 将不做任何操作。

## 使用`case`语句选择选项

`if`语句允许你根据布尔表达式的值做出是/否决策。但是，如果你需要在多个选项中做选择，可以使用`case`语句代替（`case.pp`）：

```
$webserver = 'nginx'
case $webserver {
  'nginx': {
    notice("Looks like you're using Nginx! Good choice!")
  }
  'apache': {
    notice("Ah, you're an Apache fan, eh?")
  }
  'IIS': {
    notice('Well, somebody has to.')
  }
  default: {
    notice("I'm not sure which webserver you're using!")
  }
}
```

在`case`语句中，Puppet 会将表达式的值与按顺序列出的每个情况进行比较。如果找到匹配项，则会应用相应的资源。名为`default`的特殊情况始终匹配，你可以使用它确保即使其他情况都不匹配，Puppet 也能做出正确的操作。

# 查找事实

Puppet 清单中很常见的需求是需要了解它们运行的系统的一些信息，例如其主机名、IP 地址或操作系统版本。Puppet 获取系统信息的内置机制叫做**Facter**，Facter 提供的每一条信息被称为**事实**。

## 使用事实哈希

你可以在清单中使用**facts 哈希**来访问 Facter 事实。这是一个 Puppet 变量`$facts`，在清单中的任何地方都可以使用，若要获取某个特定的事实，只需提供该事实的键名（`facts_hash.pp`）：

```
notice($facts['kernel'])
```

在 Vagrant 盒子中，或者任何 Linux 系统中，这将返回值`Linux`。

在较旧版本的 Puppet 中，每个事实都是一个独立的全局变量，如下所示：

```
notice($::kernel)
```

你仍然会在一些 Puppet 代码中看到这种事实引用风格，尽管它现在已被弃用，并最终会停止工作，所以你应该始终使用$facts 哈希。

## 运行 facter 命令

你还可以使用`facter`命令查看某些特定事实的值，或仅查看哪些事实是可用的。例如，在命令行中运行`facter os`将显示可用的与操作系统相关的事实的哈希：

```
facter os
{
  architecture => "amd64",
  distro => {
    codename => "xenial",
    description => "Ubuntu 16.04 LTS",
    id => "Ubuntu",
    release => {
      full => "16.04",
      major => "16.04"
    }
  },
  family => "Debian",
  hardware => "x86_64",
  name => "Ubuntu",
  release => {
    full => "16.04",
    major => "16.04"
  },
  selinux => {
    enabled => false
  }
}
```

你还可以使用`puppet facts`命令来查看哪些事实将对 Puppet 清单可用。这还会包括由第三方 Puppet 模块定义的任何自定义事实（有关更多信息，请参阅第七章，*精通模块*）。

## 访问事实的哈希

正如前面的例子所示，许多事实实际上返回的是一个值的哈希，而不是单一的值。`$facts['os']`事实的值是一个哈希，其中包含`architecture`、`distro`、`family`、`hardware`、`name`、`release`和`selinux`等键。其中一些也是哈希；一切都是哈希！

如你所知，要访问哈希中的特定值，你需要在方括号中指定键名。要访问哈希内部的值，你需要在第一个键名后加上另一个键名，如下例所示（`facts_architecture.pp`）：

```
notice($facts['os']['architecture'])
```

你可以继续附加更多的键来获得越来越具体的信息（`facts_distro_codename.pp`）：

```
notice($facts['os']['distro']['codename'])
```

### 注意

**关键事实**

操作系统的主要版本是一个非常有用的事实，你可能会经常用到它：

```
$facts['os']['release']['major']
```

## 在表达式中引用事实

就像普通变量或值一样，你可以在表达式中使用事实，包括条件表达式（`fact_if.pp`）：

```
if $facts['os']['selinux']['enabled'] {
  notice('SELinux is enabled')
} else {
  notice('SELinux is disabled')
}
```

### 提示

虽然基于事实的条件表达式可能很有用，但在清单中基于事实做出决策的更好方法是使用 Hiera，我们将在下一章中介绍。例如，如果发现自己编写了根据操作系统版本选择不同资源的`if`或`case`语句，请考虑改用 Hiera 查询。

## 使用内存事实

另一个有用的事实集是与**系统内存**相关的事实。您可以查看可用的总物理内存，当前使用的内存量，以及交换内存的相同数字。

其一常见用途是根据系统内存量动态配置应用程序。例如，MySQL 参数`innodb_buffer_pool_size`指定为数据库查询缓存和索引分配的内存量，通常应尽可能设置得高（根据文档，"*尽可能大的值，留下足够的内存供节点上的其他进程运行而不过度分页*"）。因此，您可以决定将其设置为总内存的四分之三（例如），使用事实和算术表达式，如以下片段（`fact_memory.pp`）所示：

```
$buffer_pool = $facts['memory']['system']['total_bytes'] * 3/4
notice("innodb_buffer_pool_size=${buffer_pool}")
```

### 注意

**关键事实**

总系统内存事实将帮助您计算随内存分数变化的配置参数：

```
$facts['memory']['system']['total_bytes']
```

## 发现网络事实

大多数应用程序使用网络，因此对于任何涉及网络配置的事情，您会发现 Facter 的网络相关事实非常有用。最常用的事实是系统主机名、完全合格的域名（FQDN）和 IP 地址（`fact_networking.pp`）：

```
notice("My hostname is ${facts['hostname']}")
notice("My FQDN is ${facts['fqdn']}")
notice("My IP is ${facts['networking']['ip']}")
```

### 注意

**关键事实**

系统主机名是您在清单中经常需要引用的内容：

```
$facts['hostname']
```

## 提供外部事实

虽然 Puppet 提供的内置事实提供了大量重要信息，但通过添加自定义事实（称为**外部事实**），您可以使`$facts`哈希表变得更加有用。例如，如果节点位于不同的云提供商中，每个提供商都需要稍有不同的网络设置，您可以创建一个名为`cloud`的自定义事实来记录这一点。然后，您可以在清单中使用此事实来做出决策。

Puppet 在`/opt/puppetlabs/facter/facts.d/`目录中查找外部事实。尝试在该目录中创建一个名为`facts.txt`的文件，并包含以下内容（`fact_external.txt`）：

```
cloud=aws
```

快速实现此目标的方法是运行以下命令：

```
sudo cp /examples/fact_external.txt /opt/puppetlabs/facter/facts.d

```

现在你的清单中包含了`cloud`事实。您可以通过运行以下命令来检查该事实是否有效：

```
sudo facter cloud
aws
```

要在您的清单中使用该事实，请像使用内置事实（`fact_cloud.pp`）一样查询`$facts`哈希表：

```
case $facts['cloud'] {
  'aws': {
    notice('This is an AWS cloud node ')
  }
  'gcp': {
    notice('This is a Google cloud node')
  }
  default: {
    notice("I'm not sure which cloud I'm in!")
  }
}
```

您可以在单个文本文件中放置尽可能多的事实，或者您可以在单独的文件中放置每个事实：这没有任何区别。 Puppet 将读取`facts.d/`目录中的所有文件，并从每个文件中提取所有`key=value`对。

文本文件适用于简单的事实（那些返回单一值的事实）。如果您的外部事实需要返回结构化数据（例如数组或哈希），您可以改用 YAML 或 JSON 文件来实现这一点。我们将在下一章学习更多关于 YAML 的内容，但现在，如果您需要构建结构化的外部事实，请参考 Puppet 文档了解详细信息。

在构建时设置外部事实是很常见的，也许是自动引导脚本的一部分（有关引导过程的更多信息，请参见 第十二章，*综合应用*）。

## 创建可执行事实

外部事实不仅限于静态文本文件。它们也可以是脚本或程序的输出。例如，您可以编写一个脚本，调用 web 服务以获取某些数据，结果将是事实的值。这些被称为**可执行事实**。

可执行事实与其他外部事实位于同一目录（`/opt/puppetlabs/facter/facts.d/`），但是它们通过在文件上设置可执行位来区分（请记住，类 Unix 系统上的文件都有一组位，指示其读、写和执行权限），并且它们不能使用 `.txt`、`.yaml` 或 `.json` 后缀命名。让我们构建一个简单的可执行事实，返回当前日期作为示例：

1.  运行以下命令将可执行事实示例复制到外部事实目录中：

    ```
    sudo cp /examples/date.sh /opt/puppetlabs/facter/facts.d

    ```

1.  使用以下命令为文件设置可执行位：

    ```
    sudo chmod a+x /opt/puppetlabs/facter/facts.d/date.sh

    ```

1.  现在测试该事实：

    ```
    sudo facter date
    2017-04-12
    ```

这是生成该输出的脚本（`date.sh`）：

```
#!/bin/bash
echo "date=`date +%F`"
```

请注意，脚本必须在实际日期值之前输出`date=`。这是因为 Facter 期望可执行的事实输出一个`key=value`对的列表（在这种情况下只有一个对）。`key`是事实的名称（`date`），`value`是通过`` `date +%F` ``返回的内容（ISO 8601 格式的当前日期）。顺便提一下，任何需要表示日期时都应该使用 ISO 8601 格式（`YYYY-MM-DD`），因为它不仅是国际标准日期格式，而且清晰、无歧义，并且按字母顺序排序。

如您所见，可执行事实非常强大，因为它们可以返回程序可以生成的任何信息（例如，程序可以进行网络请求或数据库查询）。然而，您应该小心使用可执行事实，因为 Puppet 必须在每次运行时评估节点上的*所有*外部事实，这意味着它会运行 `/opt/puppetlabs/facter/facts.d` 目录中的每个脚本。

### 提示

如果您不需要每次 Puppet 运行时重新生成可执行事实的信息，考虑从 cron 作业中定期运行脚本，并让它将输出写入事实目录中的静态文本文件。

# 遍历数组

迭代（反复做某事）是 Puppet 清单中一个有用的技巧，可以避免大量重复的代码。例如，考虑以下清单，它创建了多个具有相同属性的文件（`iteration_simple.pp`）：

```
file { '/usr/local/bin/task1':
  content => "echo I am task1\n",
  mode    => '0755',
}

file { '/usr/local/bin/task2':
  content => "echo I am task2\n",
  mode    => '0755',
}

file { '/usr/local/bin/task3':
  content => "echo I am task3\n",
  mode    => '0755',
}
```

你可以看到这些资源每个都是相同的，除了任务编号：`task1`、`task2` 和 `task3`。显然，这样大量的输入是繁琐的，如果你稍后决定修改这些脚本的属性（例如，将它们移动到不同的目录），你需要在清单中找到并修改每一个。对于三个资源来说，这已经很麻烦了，但如果是三十个或者一百个资源，那简直无法忍受。我们需要更好的解决方案。

## 使用 each 函数

Puppet 提供了 `each` 函数来帮助处理这种情况。`each` 函数接收一个数组，并将一段 Puppet 代码应用于数组中的每个元素。这里是我们之前看到的相同示例，只不过这次使用了数组和 `each` 函数（`iteration_each.pp`）：

```
$tasks = ['task1', 'task2', 'task3']
$tasks.each | $task | {
  file { "/usr/local/bin/${task}":
    content => "echo I am ${task}\n",
    mode    => '0755',
  }
}
```

现在这看起来更像是一个计算机程序！我们有了一个由 `each` 函数创建的 **循环**。这个循环一次又一次地运行，为每个 `$tasks` 数组中的元素创建一个新的 `file` 资源。让我们看一下 `each` 循环的示意图：

```
ARRAY.each | ELEMENT | {
  BLOCK
}
```

以下列表描述了 `each` 循环的组成部分：

+   `ARRAY` 可以是任何 Puppet 数组变量或字面量值（甚至可以是返回数组的 Hiera 调用）。在之前的示例中，我们使用了 `$tasks` 作为数组。

+   `ELEMENT` 是一个变量的名称，每次循环时它将保存数组中当前元素的值。在之前的示例中，我们决定将此变量命名为 `$task`，尽管我们本可以取任何名字。

+   `BLOCK` 是一段 Puppet 代码。这可以是一个函数调用、资源声明、包含语句、条件语句：你可以在 Puppet 清单中写入的任何内容，也可以放在循环块内。在之前的示例中，块中唯一的内容是 `file` 资源，它创建了 `/usr/local/bin/$task`。

## 迭代哈希

`each` 函数不仅适用于数组，也适用于哈希。当迭代哈希时，循环需要两个 `ELEMENT` 参数：第一个是哈希的键，第二个是哈希的值。以下示例展示了如何使用 `each` 来迭代一个由 Facter 查询返回的哈希（`iteration_hash.pp`）：

```
$nics = $facts['networking']['interfaces']
$nics.each | String $interface, Hash $attributes | {
  notice("Interface ${interface} has IP ${attributes['ip']}")
}
```

`$facts['networking']['interfaces']` 返回的接口列表是一个哈希，其中键是接口的名称（例如，`lo0` 是本地回环接口的名称），值是接口属性的哈希（包括 IP 地址、子网掩码等）。应用之前示例中的清单会产生如下结果（在我的 Vagrant 虚拟机上）：

```
sudo puppet apply /examples/iteration_hash.pp
Notice: Scope(Class[main]): Interface enp0s3 has IP 10.0.2.15
Notice: Scope(Class[main]): Interface lo has IP 127.0.0.1
```

# 摘要

在本章中，我们了解了 Puppet 的变量和数据类型系统如何工作，包括基本数据类型：字符串、数字、布尔值、数组和哈希。我们学习了如何在字符串中插入变量，以及如何通过资源名称数组快速创建一组相似的资源。我们还学会了如何通过属性-值对的哈希设置资源的常见属性，并使用属性展开操作符。

我们了解了如何在表达式中使用变量和值，包括算术表达式，并探讨了 Puppet 比较操作符的范围，以生成布尔表达式。我们使用条件表达式构建 `if…else` 和 `case` 语句，并简要介绍了正则表达式。

我们学习了 Puppet 的 Facter 子系统如何通过 facts 哈希提供关于节点的信息，以及如何在自己的清单和表达式中使用 facts。我们指出了一些关键的 facts，包括操作系统版本、系统内存容量和系统主机名。我们还了解了如何创建自定义外部 facts，例如 `cloud` fact，以及如何使用可执行的 facts 动态生成 fact 信息。

最后，我们了解了如何使用 Puppet 中的 `each` 函数进行迭代，并基于数组或哈希中的数据（包括 Facter 查询）创建多个资源。

在下一章中，我们将继续讨论数据话题，探索 Puppet 强大的 Hiera 数据库。我们将看到 Hiera 解决了哪些问题，学习如何设置和查询 Hiera，如何编写数据源，如何直接从 Hiera 数据创建 Puppet 资源，以及如何使用 Hiera 加密来管理敏感数据。

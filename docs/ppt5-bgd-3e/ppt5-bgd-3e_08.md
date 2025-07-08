# 第八章 类、角色和配置文件

|   | *我们的生活被琐事浪费了。简化，简化！* |   |
| --- | --- | --- |
|   | --*亨利·戴维·梭罗* |

在本章中，你将探索 Puppet 类的细节，定义类和包含类的区别，如何为类提供参数，如何声明带参数的类并为其指定合适的数据类型。你将学习如何创建定义的资源类型，以及它们与类的不同。你还将了解如何使用节点、角色和配置文件的概念来组织你的 Puppet 代码。

![类、角色和配置文件](img/8880_08_01.jpg)

# 类

在本书中，我们已经多次提到**类**的概念，但并没有真正解释它。现在让我们进一步探索，并看看如何使用这个关键的 Puppet 语言构建块。

## 类关键字

你可能注意到，在我们示例 NTP 模块的代码中（见第七章， *掌握模块*部分中的*编写模块代码*部分），我们使用了`class`关键字：

```
class pbg_ntp {
  ...
}
```

如果你在想`class`关键字的作用，出乎意料的答案是：什么也不做。也就是说，除了通知 Puppet 将其中的资源分组在一起并赋予一个名称（`pbg_ntp`），并且这些资源不应该被应用。

然后你可以在其他地方使用这个名称，告诉 Puppet 一起应用该类中的所有资源。我们通过使用`include`关键字来声明我们的示例模块：

```
include ntp
```

以下示例展示了一个**类定义**，它使得类对 Puppet 可用，但并不会（还没有）应用其中的任何资源：

```
class CLASS_NAME {
  ...
}
```

以下示例展示了`CLASS_NAME`类的**声明**。声明告诉 Puppet 应用该类中的所有资源（并且该类必须已经定义）：

```
include CLASS_NAME
```

你可能还记得在第七章， *掌握模块*中，我们使用了 Hiera 的自动参数查找机制来为类提供参数。我们很快会进一步了解这个，但首先，我们如何编写一个接受参数的类呢？

## 向类声明参数

如果一个类仅仅是将相关资源分组在一起，那也是有用的，但如果我们能使用**参数**，类将变得更强大。参数就像资源属性：它们让你传递数据给类，以改变类的应用方式。

以下示例展示了如何定义一个接受参数的类。这是我们为 NTP 模块开发的`pbg_ntp`类的简化版本（`class_params.pp`）：

```
# Manage NTP
class pbg_ntp_params (
  String $version = 'installed',
) {
  ensure_packages(['ntp'],
    {
      'ensure' => $version,
    }
  )
}
```

需要关注的重要部分是类定义开始后的括号中的内容。这部分指定了类接受的参数：

```
String $version = 'installed',
```

`String` 告诉 Puppet 我们期望这个值是一个字符串，如果我们尝试传递其他类型的值，例如整数，它会抛出错误。`$version` 是参数的名称。最后，`'installed'` 部分指定了该参数的**默认值**。如果有人声明了这个类而没有提供 `pbg_ntp_params::version` 参数，Puppet 会自动使用这个默认值进行填充。

如果你没有为某个参数提供默认值，那么该参数就是**必需的**，因此 Puppet 不允许你声明这个类而不为该参数提供值。

当你声明这个类时，你可以像之前使用 Puppet Forge 模块时那样，使用 `include` 关键字和类名：

```
include pbg_ntp_params
```

这个类没有必需的参数，因此你不必提供任何参数，但如果你提供了参数，可以像下面这样在 Hiera 数据中添加一个值，Puppet 在类被包含时会自动查找它：

```
pbg_ntp_params::version: 'latest'
```

当然，类可以接受多个参数，以下（虚构的）示例展示了如何声明多个不同类型的参数（`class_params2.pp`）：

```
# Manage NTP
class pbg_ntp_params2 (
  Boolean $start_at_boot,
  String[1] $version                        = 'installed',
  Enum['running', 'stopped'] $service_state = 'running',
) {
  ensure_packages(['ntp'],
    {
      'ensure' => $version,
    }
  )

  service { 'ntp':
    ensure => $service_state,
    enable => $start_at_boot,
  }
}
```

要向这个类传递参数，可以添加如下的 Hiera 数据：

```
pbg_ntp_params2::start_at_boot: true
pbg_ntp_params2::version: 'latest'
pbg_ntp_params2::service_state: 'running'
```

让我们仔细看看参数列表：

```
  Boolean $start_at_boot,
  String[1] $version                        = 'installed',
  Enum['running', 'stopped'] $service_state = 'running',
```

第一个参数是 `Boolean` 类型，名为 `$start_at_boot`。没有默认值，因此这个参数是必需的。必需参数必须先声明，在任何可选参数（即有默认值的参数）之前。

我们在前面的例子中看到的 `$version` 参数，但现在它是一个 `String[1]` 类型，而不是一个 `String` 类型。这有什么区别？`String[1]` 是一个至少包含一个字符的字符串。这意味着，你不能将空字符串传递给这样的参数。例如，如果合适的话，为字符串参数指定最小长度是个好主意，以防不小心传递空字符串到类中。

最后的参数 `$service_state` 是一种新类型，`Enum`，我们之前没有遇到过。对于**Enum 参数**，我们可以精确地指定它可以接受的值的列表。

如果你的类期望一个字符串类型的参数，并且该参数只能接受一些有限的值，你可以在 `Enum` 参数声明中列出所有允许的值，Puppet 不会允许传递任何不在该列表中的值。例如，在我们的示例中，如果你尝试声明 `pbg_ntp_params2` 类，并将值 `bogus` 传递给 `$service_state` 参数，你会收到这个错误：

```
Error: Evaluation Error: Error while evaluating a Resource Statement, Class[Pbg_ntp_params2]: parameter 'service_state' expects a match for Enum['running', 'stopped'], got String at /examples/class_params2.pp:22:1 on node ubuntu-xenial
```

就像任何其他参数一样，`Enum` 类型的参数也可以具有默认值，就像我们在示例中所做的那样。

## 从 Hiera 数据中自动查找参数

我们在本章和上一章中看到过，我们可以使用 Hiera 数据将参数传递给类。如果我们包含一个名为 `ntp` 的类，它接受一个名为 `version` 的参数，并且 Hiera 中存在名为 `ntp::version` 的键，则其值将作为 `version` 的值传递给 `ntp` 类。例如，如果 Hiera 数据如下所示：

```
ntp::version: 'latest'
```

Puppet 将自动找到该值，并在声明 `ntp` 类时将其传递给 `ntp` 类。

通常，Puppet 按照以下优先顺序确定参数值，优先级从高到低：

1.  在类声明中指定的文字参数（你可能会看到旧代码使用这种方式）

1.  来自 Hiera 的自动参数查找（键名必须为 `CLASS_NAME::PARAMETER_NAME`）

1.  在类定义中指定的默认值

# 参数数据类型

你应该始终为类参数指定类型，因为这有助于更容易捕捉到错误，比如错误的参数或值被传递到类中。例如，如果你使用的是 String 参数，尽可能将其设为 Enum 参数，并列出类接受的确切值。如果不能限制为一组允许的值，可以指定最小长度，如 `String[x]`。（如果还需要指定最大长度，语法为 `String[min, max]`。）

## 可用数据类型

到目前为止，在本章中，我们遇到了数据类型 String、Enum 和 Boolean。以下是其他类型：

+   Integer（整数）

+   Float（浮动小数，具有可选的小数部分）

+   Numeric（匹配整数或浮动小数）

+   Array

+   Hash

+   Regexp

+   Undef（匹配尚未赋值的变量或参数）

+   Type（表示 Puppet 数据类型的字面值的数据类型，如 String、Integer 和 Array）

还有一些 *抽象* 数据类型，它们更加通用：

+   Optional（匹配一个可能未定义或未提供的值）

+   Pattern（匹配符合指定正则表达式的字符串）

+   Scalar（匹配 Numeric、String、Boolean 或 Regexp 值，但不匹配 Array、Hash 或 Undef）

+   Data（匹配 Scalar 值，但也匹配 Array、Hash 和 Undef）

+   Collection（匹配 Array 或 Hash）

+   Variant（匹配指定数据类型列表中的一种）

+   Any（匹配任何数据类型）

通常，你应该尽可能使用更具体的数据类型。例如，如果你知道某个参数将始终是一个整数，使用 `Integer`。如果它也需要接受浮动小数值，使用 `Numeric`。如果它可以是字符串或数字，使用 `Scalar`。

## 内容类型参数

代表一组值的类型，例如 `Array` 和 `Hash`（或它们的父类型 `Collection`）也可以接受一个参数，指示它们包含的值的类型。例如，`Array[Integer]` 匹配一个整数值的数组。

如果你声明一个内容类型参数给一个集合，那么该集合中的所有值必须与声明的类型匹配。如果没有指定内容类型，默认值为 `Data`，它匹配（几乎）任何类型的值。内容类型参数本身也可以接受参数：`Array[Integer[1]]` 声明了一个正整数数组。

Hash 需要两个内容类型参数，第一个表示键的数据类型，第二个表示值的数据类型。`Hash[String, Integer]` 声明一个哈希，其键是字符串，每个键关联一个整数值（例如，这将匹配哈希 `{'eggs' => 61}`）。

## 范围参数

大多数类型也可以接受方括号中的参数，这些参数使类型声明更加具体。例如，我们已经看到 `String` 可以接受一对参数，表示字符串的最小和最大长度。

大多数类型可以接受**范围****参数**：`Integer[0]` 匹配任何大于或等于零的整数，而 `Float[1.0, 2.0]` 匹配 1.0 到 2.0 之间（包括 1.0 和 2.0）的任何浮点数。

如果任一范围参数为特殊值 `default`，则将使用该类型的默认最小值或最大值。例如，`Integer[default, 100]` 匹配任何小于或等于 100 的整数。

对于数组和哈希，范围参数指定元素或键的最小和最大数量：`Array[Any, 16]` 指定一个包含不少于 16 个 `Any` 类型元素的数组。`Hash[Any, Any, 5, 5]` 指定一个哈希，其中包含恰好五个键值对。

你可以同时指定范围和内容类型参数：`Array[String, 1, 10]` 匹配一个包含 1 到 10 个字符串的数组。`Hash[String, Hash, 1]` 指定一个哈希，其键是字符串，值是哈希，且至少包含一对键值对，键是字符串，值的类型是哈希。

## 灵活的数据类型

如果你不确定值的类型，可以使用 Puppet 更灵活的**抽象类型**，如 `Variant`，它指定一个允许类型的列表。例如，`Variant[String, Integer]` 允许其值为字符串或整数。

类似地，`Array[Variant[Enum['true', 'false'], Boolean]]` 声明一个数组，其中的值可以是字符串值 `'true'` 或 `'false'`，也可以是布尔值 `true` 和 `false`。

`Optional` 类型在值可能未定义时非常有用。例如，`Optional[String]` 指定了一个可以传递也可以不传递的字符串参数。通常，如果一个参数没有声明默认值，Puppet 会在没有提供值时抛出错误。然而，如果声明为 `Optional`，则可以省略，或设置为 `Undef`（意味着该标识符已定义，但没有值）。

`Pattern` 类型允许你指定一个正则表达式。所有符合该正则表达式的字符串都将被视为该参数的有效值。例如，`Pattern[/a/]` 会匹配任何包含小写字母 `a` 的字符串。实际上，你可以指定任意多个正则表达式。`Pattern[/a/, /[0-9]/]` 会匹配任何包含字母 `a` 的字符串，或任何包含数字的字符串。

# 定义的资源类型

与类让你将相关资源组合在一起不同，**定义的资源类型**让你创建新的资源类型，并声明任意多个实例。定义资源类型的定义看起来非常像一个类（`defined_resource_type.pp`）：

```
# Manage user and SSH key together
define user_with_key(
  Enum[
    'ssh-dss',
    'dsa',
    'ssh-rsa',
    'rsa',
    'ecdsa-sha2-nistp256',
    'ecdsa-sha2-nistp384',
    'ecdsa-sha2-nistp521',
    'ssh-ed25519',
    'ed25519'
  ] $key_type,
  String $key,
) {
  user { $title:
    ensure     => present,
    managehome => true,
  }

  file { "/home/${title}/.ssh":
    ensure => directory,
    owner  => $title,
    group  => $title,
    mode   => '0700',
  }

  ssh_authorized_key { $title:
    user => $title,
    type => $key_type,
    key  => $key,
  }
}
```

你可以看到，我们使用的是 `define` 关键字，而不是 `class` 关键字。这告诉 Puppet 我们正在创建一个定义的资源类型，而不是类。这个类型叫做 `user_with_key`，一旦定义，我们就可以像其他 Puppet 资源一样声明它的多个实例：

```
user_with_key { 'john':
  key_type => 'ssh-rsa',
  key      => 'AAAA...AcZik=',
}
```

当我们这样做时，Puppet 会应用 `user_with_key` 中的所有资源：一个用户、该用户的 `.ssh` 目录，以及该用户的 `ssh_authorized_key`，其中包含指定的密钥。

### 提示

等等，我们似乎在示例代码中引用了一个名为 `$title` 的参数。这个参数来自哪里？`$title` 是一个特殊的参数，它在类和定义的资源类型中始终可用，它的值是该类或类型声明的标题。在这个示例中，它是 `john`，因为我们给 `user_with_key` 的声明指定了标题 `john`。

那么，定义的资源类型和类有什么区别呢？它们看起来几乎一样。它们似乎也有相似的作用。为什么你会选择使用其中一个而不是另一个呢？最重要的区别在于，你在一个节点上只能有一个给定类的**声明**，而你可以拥有任意多个不同实例的定义资源类型。唯一的限制是，像所有 Puppet 资源一样，每个定义资源类型实例的标题必须是唯一的。

回想一下我们的 `ntp` 类，它安装并运行 NTP 守护进程。通常，你每个节点只希望有一个 NTP 服务。运行两个几乎没有意义。所以我们只声明一次该类，这就是我们需要的。

将此与定义的 `user_with_key` 资源类型进行对比。很可能你希望在某个节点上拥有多个 `user_with_key`，可能有好几个。在这种情况下，定义的资源类型是正确的选择。

定义的资源类型在模块中非常理想，特别是当你想让模块的用户使用某个资源时。例如，在 `puppetlabs/apache` 模块中，`apache::vhost` 资源是由 `apache` 类提供的定义资源类型。你可以将定义的资源类型看作是多个资源的包装器。

### 提示

在决定是创建类还是定义资源类型时，请记住这个经验法则：如果在给定节点上有多个实例是合理的，那么它应该是一个定义的资源类型；如果只有一个实例，它应该是一个类。

## 类型别名

使用`type`关键字定义新的**类型别名**非常简单（`type_alias.pp`）：

```
type ServiceState = Enum['running', 'stopped']

define myservice(ServiceState $state) {
  service { $name:
    ensure => $state,
  }
}

myservice { 'ntp':
  state => 'running',
}
```

创建类型别名非常有用，例如，当你想确保参数值匹配一个复杂的模式时，手动复制会很麻烦。你可以在一个地方定义这个模式，并声明多个该类型的参数（`type_alias_pattern.pp`）：

```
type IPAddress = Pattern[/\A([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\.([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])){3}\z/]

define socket_server(
  IPAddress $listen_address,
  IPAddress $public_address,
) {
  # ...
}

socket_server { 'myserver':
  listen_address => '0.0.0.0',
  public_address => $facts['networking']['ip'],
}
```

在模块中创建类型别名时，它应该位于模块的`types`子目录下，文件名与类型名称相同。例如，一个名为`IPAddress`的类型应该在`types/ipaddress.pp`文件中定义。

# 使用 Hiera 管理类

在第三章，*使用 Git 管理 Puppet 代码*中，我们看到了如何在多个节点上设置 Puppet 仓库，并使用 cron 作业和`run-puppet`脚本自动应用清单。`run-puppet`脚本运行以下命令：

```
cd /etc/puppetlabs/code/environments/production && git pull/opt/puppetlabs/bin/puppet apply manifests/

```

你可以看到，`manifests/`目录中的所有内容将在每个节点上应用。显然，当我们能够在每个节点上应用不同的清单时，Puppet 的作用就更大了；一些节点将是 Web 服务器，另一些将是数据库服务器，等等。实际上，我们希望在所有节点上包括一些类，用于通用管理，例如管理用户帐户，其他类则仅在特定节点上使用。那么我们该怎么做呢？

## 使用`include`与`lookup()`一起

之前，在我们的清单中包含类时，我们使用了带有字面类名的`include`关键字，如下例所示：

```
include postgresql
include apache
```

然而，`include`也可以作为一个函数使用，它接受一个包含类名的数组：

```
include(['postgresql', 'apache'])
```

我们已经知道，可以使用 Hiera 根据节点名称（或层次结构中定义的任何其他内容）返回不同的查询值，因此我们可以在 Hiera 数据中定义一个合适的数组，如下例所示：

```
classes:
- postgresql
- apache
```

现在我们可以简单地使用`lookup()`来获取这个 Hiera 值，并将结果传递给`include()`函数：

```
include(lookup('classes'), Array[String], 'unique')
```

实际上，这就是你整个 Puppet 清单的内容。每个节点都会应用这个清单，从而包括 Hiera 数据中为其分配的类。由于顶级清单文件通常命名为`site.pp`，因此你可以将这行`include`放入`manifests/site.pp`中，`papply`或`run-puppet`脚本将应用它，因为它们会应用`manifests/`目录中的所有内容。

## 公共类和每节点类

我们可以在`common.yaml`中指定一组类，这些类将应用于所有节点：例如用户帐户、SSH 和`sudoers`配置、时区、NTP 设置等。在第十二章中，*将一切整合*的完整示例仓库定义了一组典型的类，并且这些类都在`common.yaml`中。

然而，某些类只在特定节点上需要。将这些类添加到每节点的 Hiera 数据文件中。例如，我们在 Vagrant 盒子上的`pbg`环境在`hiera.yaml`中包含以下内容：

```
  - name: "Host-specific data"
    path: "nodes/%{facts.hostname}.yaml"
```

所以，名为`node1`的节点的每节点数据将保存在`data/`目录下的`nodes/node1.yaml`文件中。

让我们看一个完整的示例。假设你的`common.yaml`文件包含如下内容：

```
classes:
- postgresql
- apache
```

假设你的每节点文件（`nodes/node1.yaml`）也包含以下内容：

```
classes:
- tomcat
- my_app
```

现在，当你在`node1`上应用以下清单（`manifests/site.pp`）时会发生什么？

```
include(lookup('classes'), Array[String], 'unique')
```

哪些类将会被应用？你可能还记得第六章，*使用 Hiera 管理数据*中提到的，`unique`合并策略会查找整个层次结构中给定键的所有值，将它们合并在一起，并作为一个扁平化的数组返回，重复项被移除。因此，这个`lookup()`调用的结果将是以下数组：

```
[apache, postgresql, tomcat, my_app]
```

这是 Puppet 将应用于节点的完整类列表。当然，如果需要，你可以在层次结构的其他任何级别添加类，但你可能会发现常规层级和每个节点的层级最适合用于包含类。

自然，尽管一些节点可能包含与其他节点相同的类，它们可能需要为这些类提供不同的配置值。你可以像之前在本章的*从 Hiera 数据自动查找参数*部分中描述的那样，使用 Hiera 为包含的类提供不同的参数。

# 角色和配置文件

现在我们知道如何根据节点要执行的任务，在给定节点上包含不同的类集，让我们更深入地思考如何以最有帮助的方式命名这些类。例如，考虑下面某个节点的包含类列表：

```
classes:
- postgresql
- apache
- java
- tomcat
- my_app
```

类名为我们提供了一些关于这个节点可能在做什么的线索。看起来它可能是一个运行 Java 应用程序`my_app`的应用服务器，通过 Tomcat 在 Apache 后端提供服务，并由 PostgreSQL 数据库支持。这是一个很好的开始，但我们可以做得更好，我们将在下一节看到如何做。

## 角色

为了明确节点是一个应用服务器，为什么不创建一个名为`role::app_server`的类，该类仅用于封装节点包含的类呢？该类的定义可能如下所示（`role_app_server.pp`）：

```
# Be an app server
class role::app_server {
  include postgresql
  include apache
  include java
  include tomcat
  include my_app
}
```

我们将这个概念称为**角色类**。角色类可以仅仅是一个独立的模块，或者为了明确表示这是一个角色类，我们可以将其组织成一个特殊的`role`模块。如果你将所有的角色类放在一个模块中，那么它们的名称将都以`role::something`命名，具体取决于它们实现的角色。

### 提示

需要注意的是，角色类在 Puppet 中并不特别。它们只是普通的类；我们之所以称它们为角色类，仅仅是为了提醒自己它们是用来表达分配给特定节点的角色。

Hiera 中 `classes` 的值现在简化为以下内容：

```
classes:
- role::app_server
```

看着 Hiera 数据，现在很容易看出节点的工作是什么——它的*角色*是什么——所有应用服务器现在只需要包括 `role::app_server`。当应用服务器所需的类列表发生变化时，你不需要找到并更新每个应用服务器的 Hiera `classes` 值；只需要编辑 `role::app_server` 类。

## 配置文件

通过采用一个经验法则，我们可以整理清单：除了在 `common.yaml` 中的公共配置外，**节点应该只包含角色类**。这使得 Hiera 数据更具自我文档性，而且我们的角色类都整齐地组织在 `role` 模块中，每个角色类都封装了该角色所需的所有功能。这是一个很大的改进。但我们能做得更好吗？

让我们看看像 `role::app_server` 这样的角色类。它包含很多行，包括模块，像这样：

```
  include tomcat
```

如果你只需要包括一个模块，并且让参数自动从 Hiera 数据中查找，那么就没有问题。这种简单、鼓励性的、不切实际的示例通常会出现在产品文档或会议幻灯片上。

然而，真实的 Puppet 代码通常更复杂，包含逻辑、条件语句、特殊情况以及需要添加的额外资源等。我们不想在将 Tomcat 用作另一个角色的一部分时重复所有这些代码（例如，提供另一个基于 Tomcat 的应用）。我们如何在合适的抽象层次封装它并避免重复？

我们当然可以为每个应用创建一个自定义模块，将所有这些杂乱的支持代码隐藏起来。然而，为了几行代码创建一个新模块是一个很大的开销，因此似乎应该有一个小层代码填补角色与模块之间的空隙。

我们称之为 **配置文件类**。配置文件封装了角色所需的一些特定软件或功能。在我们的例子中，`app_server` 角色需要几种软件：PostgreSQL、Tomcat、Apache 等。现在，每个软件都可以有它自己的配置文件。

让我们重写 `app_server` 角色以包括配置文件，而不是模块（`role_app_server_profiles.pp`）：

```
# Be an app server
class role::app_server {
  include profile::postgresql
  include profile::apache
  include profile::java
  include profile::tomcat
  include profile::my_app
}
```

这些配置文件类中会有什么内容？例如，`profile::tomcat` 类将设置 Tomcat 所需的特定配置，同时还包括任何特定应用或站点所需的资源，如防火墙规则、`logrotate` 配置、文件和目录权限等。配置文件封装了模块，配置它，并提供模块未涵盖的部分，以支持该特定应用或站点。

`profile::tomcat` 类可能看起来像以下示例，改编自一个真实的生产清单（`profile_tomcat.pp`）：

```
# Site-specific Tomcat configuration
class profile::tomcat {
  tomcat::install { '/usr/share/tomcat7':
    install_from_source => false,
    package_ensure      => present,
    package_name        => ['libtomcat7-java','tomcat7-common','tomcat7'],
  }

  exec { 'reload-tomcat':
    command     => '/usr/sbin/service tomcat7 restart',
    refreshonly => true,
  }

  lookup('tomcat_allowed_ips', Array[String[7]]).each |String $source_ip| {
    firewall { "100 Tomcat access from ${source_ip}":
      proto  => 'tcp',
      dport  => '8080',
      source => $source_ip,
      action => 'accept',
    }
  }

  file { '/usr/share/tomcat7/logs':
    ensure  => directory,
    owner   => 'tomcat7',
    require => Tomcat::Install['/usr/share/tomcat7'],
  }

  file { '/etc/logrotate.d/tomcat7':
    source => 'puppet:///site-modules/profile/tomcat/tomcat7.logrotate',
  }
}
```

这类配置文件的具体内容在这里并不重要，但你应该记住的是，这种特定站点的“胶水”代码，用来包装第三方模块并将它们与特定应用程序连接，应该放在配置文件类中。

通常，一个配置文件类应该包含使该特定软件组件或服务正常工作的所有内容，包括必要时的其他配置文件。例如，每个需要特定 Java 配置的配置文件应该包含该 Java 配置文件。你可以从多个其他配置文件中包含配置文件，而不会发生冲突。

以这种方式使用配置文件类，既可以让你的角色类更加整洁、有序且易于维护，还可以让你重用这些配置文件来适应不同的角色。`app_server`角色包含了这些配置文件，其他角色也可以包含它们。这样，我们的代码组织得更加有序，减少了重复，并鼓励重用。第二条经验法则是，**角色应该仅包含配置文件**。

如果你仍然对角色和配置文件之间的确切区别感到困惑，不用担心：你并不孤单。让我们尽可能简洁地定义它们：

+   **角色**标识节点的特定功能，例如作为应用服务器或数据库服务器。角色存在的目的是记录节点的用途。角色应该仅包含配置文件，但可以包含任意数量的配置文件。

+   **配置文件**标识为角色提供特定软件或功能的部分；例如，`tomcat`配置文件是`app_server`角色所需要的。配置文件通常安装和配置特定的软件组件或服务、其相关的业务逻辑以及任何其他所需的 Puppet 资源。配置文件是“胶水层”，位于角色和模块之间。

你的清单可能非常简单，以至于你只需要使用角色或仅使用配置文件来组织它。这没有问题，但当事情变得更加复杂，并且你发现自己在重复代码时，可以考虑按照我们在这里看到的方式重构代码，采用角色和配置文件模式。

# 总结

在本章中，我们讨论了多种组织 Puppet 代码的方式。我们详细介绍了类的定义，说明了如何使用`class`关键字定义一个新类，如何使用`include`关键字声明类，以及如何使用 Hiera 的自动参数查找机制为包含的类提供参数。

声明参数涉及指定参数的允许数据类型，我们简要概述了 Puppet 的数据类型，包括标量、集合、内容类型和范围参数、抽象类型、灵活类型，并介绍了如何创建你自己的类型别名。我们还介绍了定义资源类型，解释了定义资源类型与类之间的区别，以及何时使用其中的一个或另一个。

我们还讨论了如何在 Hiera 中使用 `classes` 数组将公共类包含在所有节点上，而将其他类仅包含在特定节点上。我们介绍了角色类的概念，角色类封装了节点完成特定角色所需的所有内容，比如应用服务器。

最后，我们看到如何使用配置文件类来配置和支持特定的软件包或服务，并且如何将多个配置文件类组合成一个角色类。角色和配置文件类之间架起了 Hiera `classes` 数组（位于顶层）与模块和配置数据（位于最底层）之间的桥梁。我们可以通过总结规则来说，*节点应只包含角色，角色应只包含配置文件*。

在下一章，我们将探讨如何使用 Puppet 利用模板、迭代和 Hiera 数据创建文件。

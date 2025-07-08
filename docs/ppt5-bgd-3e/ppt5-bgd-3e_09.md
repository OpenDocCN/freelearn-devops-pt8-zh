# 第九章. 使用模板管理文件

|   | *简洁并非复杂之前的状态，而是复杂之后的产物。* |   |
| --- | --- | --- |
|   | --*艾伦·珀里斯* |

在本章中，我们将了解 Puppet 的一个重要且强大的功能：**模板**。我们将学习如何使用一个简单的模板将 Puppet 变量、事实和 Hiera 数据的值插入到文件中，还将介绍使用迭代和条件语句的更复杂模板来生成动态配置文件。

![使用模板管理文件](img/8880_09_01.jpg)

# 什么是模板？

在之前的章节中，我们通过多种方式使用 Puppet 管理节点上的 **文件内容**，包括使用 `content` 属性将内容设置为字面字符串，以及使用 `source` 属性从 Puppet 模块复制文件。虽然这些方法非常有用，但它们在一个方面是有限制的：它们只能使用 **静态文本**，而不能基于 Puppet 数据动态生成文件内容。

## 动态数据问题

为了了解为什么这是一个问题，考虑一个常见的 Puppet 文件管理任务，如备份脚本。备份脚本需要知道一些特定于站点和节点的内容：本地备份的目录、将其复制到的目标位置，以及访问备份存储所需的任何凭据。虽然我们可以将这些内容作为字面值插入脚本中，但这并不灵活。我们可能不得不维护多个版本的脚本，每个版本与其他版本相同，除了备份位置不同。这显然是不令人满意的。

考虑一个应用程序的配置文件，其中一些设置依赖于关于节点的特定信息：例如可用的内存。显然，我们不希望维护多个几乎相同的配置文件版本，每个版本都包含适合我们可能遇到的不同内存大小的值。正如我们在第五章 *变量、表达式和事实*中看到的那样，我们有一种方法可以直接在 Puppet 中获取这些信息，我们也有一个灵活且强大的配置数据数据库，正如我们在第六章 *使用 Hiera 管理数据*中看到的那样。问题是，我们如何能将这些数据动态插入到文本文件中。

## Puppet 模板语法

Puppet 的 **模板**机制就是实现这一目标的一种方式。模板只是一个普通的文本文件，包含特殊的占位符标记，Puppet 会用相关的数据值替换这些标记。以下示例展示了这些标记的样子（`aws_credentials.epp`）：

```
aws_access_key_id = <%= $aws_access_key %>
```

`<%=` 和 `%>` 分隔符之外的内容是字面文本，Puppet 会原样渲染这些内容。

然而，分隔符内的文本会被解释为 Puppet 表达式（在这个例子中，仅为变量`$aws_access_key`），在模板编译时会被评估，并将结果插入到文本中。

例如，如果变量`$aws_access_key`的值为`AKIAIAF7V6N2PTOIZVA2`，那么当模板被 Puppet 处理时，生成的输出文本将如下所示：

```
aws_access_key_id = AKIAIAF7V6N2PTOIZVA2
```

你可以在模板中使用任意多的这些分隔符表达式（称为**标签**），它们将在模板使用时被评估并插入。

Puppet 的模板机制称为**EPP**（**嵌入式 Puppet**），模板文件的扩展名为`.epp`。

# 在你的清单中使用模板

由于模板的最终结果是一个文件，你不会惊讶于我们使用 Puppet 的`file`资源来处理模板。实际上，我们使用的是`file`资源的一个属性，你之前见过：`content`属性。

## 引用模板文件

回想一下第二章，*创建第一个清单*，你可以使用`content`属性将文件的内容设置为字面字符串：

```
file { '/tmp/hello.txt':
  content => "hello, world\n",
}
```

当然，你可以将 Puppet 表达式的值插入到该字符串中：

```
file { "/usr/local/bin/${task}":
  content => "echo I am ${task}\n",
  mode    => '0755',
}
```

到此为止，你应该已经很熟悉了，但我们可以再进一步，使用`epp()`函数（`file_epp.pp`）替代字面字符串：

```
file { '/usr/local/bin/backup':
  content => epp('/examples/backup.sh.epp',
    {
      'data_dir' => '/examples',
      }
  ),
  mode    => '0755',
}
```

Puppet 将编译由`backup.sh.epp`引用的模板文件，替换任何标签为其表达式的值，生成的文本将写入文件`/usr/local/bin/backup`。模板文件可能如下所示（`backup.sh.epp`）：

```
<%- | String $data_dir | -%>
#!/bin/bash
mkdir -p /backup
tar cvzf /backup/backup.tar.gz <%= $data_dir %>
```

你可以在任何期望字符串的地方使用`epp()`函数，但最常见的用法是在管理文件时使用它，如示例所示。

要在模块内引用模板文件（例如，在我们的 NTP 模块中，来自第七章，*掌握模块*），将文件放入`modules/pbg_ntp/templates/`目录，并将文件名以`pbg_ntp/`为前缀，如以下示例所示：

```
file { '/etc/ntp.conf':
  content => epp('pbg_ntp/ntp.conf.epp'),
}
```

### 提示

**记住**

不要将`templates/`作为路径的一部分。Puppet 知道这是一个模板，因此它会自动在指定模块的`templates/`目录中查找。

## 内联模板

你的模板文本不必放在单独的文件中：如果是一个简短的模板，你可以将其放在 Puppet 清单中的字面字符串中，并使用`inline_epp()`函数编译它（`file_inline_epp.pp`）：

```
$web_root = '/var/www'
$backup_dir = '/backup/www'

file { '/usr/local/bin/backup':
  content => inline_epp('rsync -a <%= $web_root %>/ <%= $backup_dir %>/'),
  mode    => '0755',
}
```

请注意，我们使用了**单引号字符串**来指定内联模板文本。如果我们使用了双引号字符串，Puppet 会在处理模板之前*插入*`$web_root`和`$backup_dir`的值，这并不是我们想要的结果。

一般来说，最好且更具可读性的是，为除了最简单的模板之外的所有模板使用单独的模板文件。

## 模板标签

我们在本章示例中使用的标签被称为**表达式打印标签**：

```
<%= $aws_access_key %>
```

Puppet 期望此标签的内容具有一个值，该值将被插入模板中，代替标签的位置。

**非打印标签**与之非常相似，但不会生成任何输出。它的开头定界符中没有`=`符号：

```
<% notice("This has no effect on the template output") %>
```

你还可以使用**注释标签**来添加文本，这些文本将在 Puppet 编译模板时被移除：

```
<%# This is a comment, and it will not appear in the output of the template %>
```

## 模板中的计算

到目前为止，我们只是将一个变量的值插入到模板中，但我们可以做得更多。模板标签可以包含任何有效的 Puppet 表达式。

在配置文件中，某些值通常是从其他值**计算**出来的，比如节点的物理内存量。我们在第五章，*变量、表达式和事实*中看到过一个示例，我们基于`$facts['memory']['system']['total_bytes']`的值计算了一个配置值。

自然地，我们在 Puppet 代码中能做的事情，也能在模板中做。所以这是相同的计算逻辑，用模板形式表示（`template_compute.epp`）：

```
innodb_buffer_pool_size=<%= $facts['memory']['system']['total_bytes'] * 3/4 %>
```

生成的输出（在我的 Vagrant 环境中）如下所示：

```
sudo puppet epp render --environment pbg /examples/template_compute.epp
innodb_buffer_pool_size=780257280
```

你不局限于数字计算；你可以做任何 Puppet 表达式能做的事情，包括字符串操作、数组和哈希查找、事实引用、函数调用等等。

## 模板中的条件语句

到目前为止，你可能对模板并不印象深刻，因为你已经可以在字符串中插入 Puppet 表达式的值，从而无需使用模板就可以处理文件。话虽如此，模板允许你将数据插入到比通过 Puppet 清单中的字面字符串更大且更实际的文件中。

模板还允许你做其他非常有用的事情：**根据某些 Puppet 条件表达式的结果包含或排除文本块**。

我们在第五章，*变量、表达式和事实*中已经遇到过清单中的条件语句，我们使用它们来有条件地包含 Puppet 资源集（`if.pp`）：

```
if $install_perl {
  ...
} else {
  ...
}
```

由于模板标签的内容就是 Puppet 代码，你也可以在模板中使用`if`语句。这是与之前类似的示例，但这次控制的是在模板中是否包含配置块（`template_if.epp`）：

```
<% if $ssl_enabled { -%>
  ## SSL directives
  SSLEngine on
  SSLCertificateFile      "<%= $ssl_cert %>"
  SSLCertificateKeyFile   "<%= $ssl_key %>"
  ...
<% } -%>
```

这看起来稍微复杂一些，但实际上它与前面的示例逻辑完全相同。我们有一个`if`语句，用来测试一个布尔变量`$ssl_enabled`的值，根据结果，以下的代码块要么被包含，要么被排除。

你可以看到，`if`语句和结束的`}`被包含在非打印标签中，因此它们本身不会生成任何输出，而当 Puppet 编译模板时，它将执行标签内的 Puppet 代码，这将决定输出内容。如果`$ssl_enabled`为真，模板生成的文件将包含以下内容：

```
  ## SSL directives
  SSLEngine on
  SSLCertificateFile      "<%= $ssl_cert %>"
  SSLCertificateKeyFile   "<%= $ssl_key %>"
  ...
```

否则，这部分模板将被省略。这是一种非常有用的条件包含配置文件中块的方法。

就像在清单文件中的`if`语句一样，你也可以使用`else`来包含一个备用块，如果条件语句为假。

### 提示

请注意，在前面的示例中，关闭标签有一个额外的前导连字符：`-%>`。

当你使用这个语法时，Puppet 会抑制标签后面的任何尾随空格和换行符。通常我们会将此语法与非打印模板标签一起使用，否则你会在输出中看到空行。

# 模板中的迭代

如果我们能够从 Puppet 表达式生成文件的部分内容，并且根据条件包括或排除文件的部分内容，那么我们能否通过 Puppet 循环生成文件的部分内容呢？也就是说，我们能否**对数组或哈希进行迭代**，为每个元素生成模板内容？实际上，我们可以。这是一个非常强大的机制，它使我们能够基于 Puppet 变量、Hiera 和 Facter 数据生成任意大小的文件。

## 遍历 Facter 数据

我们的第一个示例生成了一个应用程序的配置文件的一部分，该应用程序用于捕获网络数据包。为了告诉它监听哪些接口，我们需要生成节点上所有活动网络接口的列表。

我们如何生成这个输出？我们知道 Facter 可以通过`$facts['networking']['interfaces']`给我们一个所有可用网络接口的列表。实际上，这是一个哈希，键是接口的名称，值是接口属性的哈希，例如 IP 地址和子网掩码。

你可能还记得在第五章，*变量、表达式与事实*中提到过，为了遍历哈希，我们使用类似以下的语法：

```
HASH.each | KEY, VALUE | {
  BLOCK
}
```

那么我们来将这个模式应用到 Facter 数据中，看看输出结果是什么样的（`template_iterate.epp`）：

```
<% $facts['networking']['interfaces'].each |String $interface, Hash $attrs| { -%>
interface <%= $interface %>;
<% } -%>
```

每次循环时，`$interface`和`$attrs`的值将设置为由`$facts['networking']['interfaces']`返回的哈希中的下一个键和值。实际上，我们不会使用`$attrs`的值，但我们仍然需要将其声明为循环语法的一部分。

每次循环时，`$interface`的值将设置为列表中下一个接口的名称，并生成像下面这样的新输出行：

```
interface em1;
```

在循环结束时，我们生成了与接口数量相同的输出行，这是我们期望的结果。以下是最终输出，在一个有很多网络接口的节点上：

```
interface em1;
interface em2;
interface em3;
interface em4;
interface em5;
interface lo;
```

## 遍历结构化事实

我们应用程序所需的下一个配置数据是与节点相关的 IP 地址列表，这可以通过类似前一个示例的方法生成。

我们可以使用与前一个示例中几乎相同的 Puppet 代码，不过这一次我们将使用每个接口的 `$attrs` 哈希表来获取相关接口的 IP 地址。

以下示例展示了这一过程是如何工作的（`template_iterate2.epp`）：

```
<% $facts['networking']['interfaces'].each |String $interface, Hash $attrs| { -%>
local_address <%= $attrs['bindings'][0]['address'] %>;
<% } -%>
```

循环与前一个示例相同，但这次每行输出的不是 `$interface` 的值，而是 `$attrs['bindings'][0]['address']` 的值，其中包含每个接口的 IP 地址。

这是最终输出：

```
local_address 10.170.81.11;
local_address 75.76.222.21;
local_address 204.152.248.213;
local_address 66.32.100.81;
local_address 189.183.255.6;
local_address 127.0.0.1;
```

## 遍历 Hiera 数据

在第六章，*使用 Hiera 管理数据*中，我们使用了一个 Hiera 用户数组来为每个用户生成 Puppet 资源。现在我们使用相同的 Hiera 数据，通过在模板中进行迭代来构建一个动态配置文件。

SSH 守护进程 `sshd` 可以配置为仅允许通过一个用户列表进行 SSH 访问（使用 `AllowUsers` 指令），而且，实际上，最好这样做。

### 提示

**安全提示**

大多数可以从公共互联网访问的服务器会定期收到针对随机用户名的暴力登录尝试，处理这些登录尝试可能会消耗大量资源。如果 `sshd` 配置为仅允许指定的用户，它可以迅速拒绝任何不在此列表中的用户，而无需进一步处理该请求。

如果我们的用户列在 Hiera 中，那么使用模板生成 `sshd_config` 文件的 `AllowUsers` 列表就变得简单了。

就像我们在生成 Puppet `user` 资源时所做的那样，我们将调用 `lookup()` 来获取用户数组，并使用 `each` 对其进行迭代。以下示例展示了在模板中（`template_hiera.epp`）是如何实现的：

```
AllowUsers<% lookup('users').each | $user | { -%>
 <%= $user -%>
<% } %>
```

注意第二行前面的空格，这会导致输出中的用户名之间以空格分隔。还要注意前导连字符的使用（`-%>`），正如我们在本章前面所看到的，它将抑制该行末尾的任何空白字符。

这是结果：

```
AllowUsers katy lark bridget hsing-hui charles
```

## 使用模板

模板的一个潜在问题（因为它们可以包含 Puppet 代码、变量和 Hiera 数据）是，从 Puppet 清单中并不总是能清楚地知道模板将使用哪些**变量**。相反，从模板代码中也不容易看出任何引用的变量来自哪里。这可能使得维护或更新模板变得困难，也增加了调试因错误数据传入模板而导致问题的难度。

理想情况下，我们希望能够在 Puppet 代码中精确地指定模板将接收哪些变量，并且这些变量的列表也会出现在模板中。作为附加内容，我们希望能够指定输入变量的**数据类型**，就像我们为类和定义的资源类型所做的那样（有关更多信息，请参见第八章，*类、角色和配置文件*）。

好消息是，EPP 模板允许你像为类那样声明传递给模板的参数，并指定所需的数据类型。虽然不强制要求为 EPP 模板声明参数，但这么做是一个非常好的主意。通过声明并类型化的参数，你将能够在模板编译阶段捕捉到大多数数据错误，从而使故障排除变得更加容易。

## 向模板传递参数

要声明模板的参数，请将它们列在管道符号（`|`）之间，放在一个非打印标签内，如以下示例所示（`template_params.epp`）：

```
<% | String[1] $aws_access_key,
     String[1] $aws_secret_key,
| -%>
aws_access_key_id = <%= $aws_access_key %>
aws_secret_access_key = <%= $aws_secret_key %>
```

在模板中声明参数时，必须将这些参数以哈希形式显式地传递作为`epp()`函数调用的第二个参数。以下示例展示了如何做到这一点（`epp_params.pp`）：

```
file { '/root/aws_credentials':
  content => epp('/examples/template_params.epp',
    {
      'aws_access_key' => 'AKIAIAF7V6N2PTOIZVA2',
      'aws_secret_key' => '7IBpXjoYRVbJ/rCTVLaAMyud+i4co11lVt1Df1vt',
    }
  ),
}
```

这种形式的`epp()`函数调用接受两个参数：模板文件的路径和一个包含所有必需模板参数的哈希。哈希的键是参数名，值是对应的值。（这些值不一定是字面量值；例如，它们可以是 Hiera 查找的结果。）

很可能你会在模板中使用 Hiera 数据，虽然在我们之前的`AllowUsers`示例中我们直接从模板中调用`lookup()`来查找数据，但这并不是最好的做法。现在我们知道如何声明并传递参数到模板，我们应该用相同的方法来处理 Hiera 数据。

这是更新后的`AllowUsers`示例，我们在清单中执行了 Hiera 查找，作为`epp()`调用的一部分。首先，我们需要在模板中声明一个`$users`参数（`template_hiera_params.epp`）：

```
<% | Array[String] $users | -%>
AllowUsers<% $users.each | $user | { -%>
 <%= $user -%>
<% } %>
```

然后，当我们使用`epp()`编译模板时，我们通过在参数哈希中调用`lookup()`来传递 Hiera 数据（`epp_hiera.pp`）：

```
file { '/tmp/sshd_config_example':
  content => epp('/examples/template_hiera_params.epp',
    {
      'users' => lookup('users'),
    }
  ),
}
```

如果你在模板中声明了参数列表，必须在`epp()`调用中准确地传递这些参数，而不能传递其他任何参数。EPP 模板声明参数的方式与类相同：参数可以具有默认值，任何没有默认值的参数都是必需的。

从前面的示例中可以清楚地看出，声明参数使得查看模板将从调用代码中使用哪些信息变得更加容易，现在我们还可以享受到自动检查参数及其类型的好处。

但请注意，即使是包含参数列表的模板，仍然可以访问模板主体中的任何 Puppet 变量或事实；Puppet 不会阻止模板使用未声明为参数的变量，或者直接从 Hiera 获取数据。不过，应该清楚的是，通过这种方式绕过参数检查机制是一个不好的做法。

### 小贴士

**最佳实践**

使用 EPP 模板来生成动态文件，在模板中声明类型化参数，并将这些参数作为哈希传递给 `epp()` 函数。为了使模板代码更易于理解和维护，始终显式地将数据传递给模板。如果模板需要查找 Hiera 数据，应在 Puppet 清单中执行查找，并让模板声明一个参数来接收数据。

## 验证模板语法

我们在本章中看到，模板可以包含复杂的逻辑和迭代，几乎可以生成任何所需的输出。这种强大灵活性的缺点是，模板代码可能很难阅读和调试。

幸运的是，Puppet 提供了一个工具，可以在命令行中检查和验证你的模板：`puppet epp validate`。要使用它，请运行以下命令检查你的模板文件：

```
puppet epp validate /examples/template_params.epp

```

如果没有输出，模板是有效的。如果模板包含错误，你将看到错误信息，类似如下：

```
Error: Syntax error at '%' at /examples/template_params.epp:3:4
Error: Errors while validating epp
Error: Try 'puppet help epp validate' for usage
```

## 在命令行渲染模板

正如任何程序员所知道的，即使程序具有有效的语法，也不一定会产生正确的结果。查看模板将要生成的输出非常有用，Puppet 也提供了一个工具来实现这一点：`puppet epp render`。

使用该工具时，运行以下命令：

```
puppet epp render --values "{ 'aws_access_key' => 'foo', 'aws_secret_key' => 'bar' }" /examples/template_params.epp
aws_access_key_id = foo
aws_secret_access_key = bar
```

`--values` 参数允许你传入一对参数值的哈希，就像你在 Puppet 清单中调用 `epp()` 函数时一样。

你也可以使用 `--values_file` 参数来引用一个包含参数哈希的 Puppet 清单文件：

```
echo "{ 'aws_access_key' => 'foo', 'aws_secret_key' => 'bar' }" >params.pp
puppet epp render --values_file params.pp /examples/template_params.epp
aws_access_key_id = foo
aws_secret_access_key = bar
```

你可以通过命令行使用 `--values` 传递参数，也可以通过 `--values_file` 从文件中传递参数，二者可以同时使用。命令行传递的参数将优先于文件中的参数：

```
puppet epp render --values_file params.pp --values "{ 'aws_access_key' => 'override' }" /examples/template_params.epp
aws_access_key_id = override
aws_secret_access_key = bar
```

你还可以使用 `puppet epp render` 来测试内联模板代码，通过 `-e` 选项传入一个字面模板字符串：

```
puppet epp render --values "{ 'name' => 'Dave' }" -e 'Hello, <%= $name %>'
Hello, Dave
```

就像测试你的清单一样，你也可以直接使用 `puppet apply` 来测试模板，使用类似以下的命令：

```
sudo puppet apply -e "file { '/tmp/result': content => epp('/examples/template_iterate.epp')}"

```

这种方法的一个优势是，所有 Puppet 变量、事实和 Hiera 数据都将可供模板使用。

## 旧版 ERB 模板

你可能会在旧代码和文档中遇到另一种类型的 Puppet 模板：**ERB 模板**。ERB（嵌入式 Ruby）是 Puppet 在版本 3.5 之前提供的唯一模板机制，直到增加了对 EPP 的支持，现在 EPP 已经取代 ERB 成为 Puppet 的默认模板格式。

ERB 模板的语法与 EPP 相似。以下是一个 ERB 模板的代码片段：

```
AllowUsers <%= @users.join(' ') %><%= scope['::ubuntu'] == 'yes' ? ',ubuntu' : '' %>
```

区别在于，标签内部的模板语言是 Ruby，而不是 Puppet。早期版本的 Puppet 在语言特性上相对有限（例如，缺少`each`函数来遍历变量），因此通常在模板中嵌入 Ruby 代码来解决这一问题。

这需要一些复杂的操作来管理 Puppet 和 Ruby 之间的接口；例如，在 ERB 模板中访问非本地作用域的变量需要使用`scope`哈希，如前面的示例所示。同样，为了访问 Puppet 函数，如`strftime()`，你必须调用：

```
scope.call_function('strftime', ...)
```

ERB 模板也不支持声明参数或类型检查。我建议你在自己的代码中仅使用 EPP 模板。

# 总结

在本章中，我们讨论了 Puppet 工具箱中最强大的工具之一——模板文件。我们研究了 EPP 标签语法，并了解了不同种类的标签，包括打印标签和非打印标签。

我们已经了解到，你不仅可以简单地将变量中的值插入模板中，还可以根据 Puppet 表达式的值包含或排除整个文本块，或者通过遍历数组和哈希生成任意大小的模板。

我们看了一些从 Facter 和 Hiera 数据动态生成配置文件的实际示例，并且了解了如何在模板文件中声明类型化参数，并在调用`epp()`函数时为这些参数传入值。

我们已经了解了如何使用`puppet epp validate`检查模板的语法，如何使用`puppet epp render`渲染模板的输出，并使用`--values`和`--values_file`传递模板参数的预设值，或者直接使用`puppet apply`渲染模板。

最后，我们讨论了遗留的 ERB 模板，它们的来源，如何与 EPP 模板进行比较，以及为什么尽管你仍可能在实际使用中遇到 ERB 模板，但你应该仅在自己的代码中使用 EPP。

在下一章，我们将探索一个热门话题——容器，了解如何使用 Puppet 管理 Docker 引擎和 Docker 容器，以及如何处理如何在容器中管理配置这一棘手问题。

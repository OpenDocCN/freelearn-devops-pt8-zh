# 第四章：处理文件和软件包

|   | *"作家的职责是做好，不是做得差；真实，不是虚假；生动，不是乏味；准确，不是充满错误的。"* |   |
| --- | --- | --- |
|   | --*E.B. White* |

在本章中，我们将覆盖以下方案：

+   快速编辑配置文件

+   使用 puppetlabs-inifile 编辑 INI 风格的文件

+   使用 Augeas 可靠地编辑配置文件

+   使用代码片段构建配置文件

+   使用 ERB 模板

+   在模板中使用数组迭代

+   使用 EPP 模板

+   使用 GnuPG 加密机密数据

+   从第三方仓库安装软件包

+   比较软件包版本

# 介绍

在本章中，我们将学习如何对文件进行小范围编辑，如何使用**Augeas**工具以结构化的方式进行更大范围的更改，如何从拼接的代码片段构建文件，以及如何从模板生成文件。我们还将学习如何从附加的仓库安装软件包，并管理这些仓库。此外，我们将学习如何使用 Puppet 存储和解密机密数据。

# 快速编辑配置文件

当你需要让 Puppet 修改配置文件中的某个特定设置时，通常的做法是直接通过 Puppet 部署整个文件。然而，这并不总是可行，尤其是当这是一个 Puppet 清单的多个部分可能需要修改的文件时。

一个有用的做法是提供一个简单的方案，向配置文件中添加一行（如果该行尚不存在）。例如，向`/etc/modules`添加模块名称，以便在启动时让内核加载该模块。有几种方法可以做到这一点，最简单的是使用`puppetlabs-stdlib`模块提供的`file_line`类型。在这个例子中，我们安装了`stdlib`模块并使用该类型向文本文件追加一行。

## 准备工作

使用 Puppet 安装`puppetlabs-stdlib`模块：

```
t@mylaptop ~ $ puppet module install puppetlabs-stdlib
Notice: Preparing to install into /home/thomas/.puppet/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/home/thomas/.puppet/modules
└── puppetlabs-stdlib (v4.5.1)

```

这会将模块从 forge 安装到我的用户 Puppet 目录；如果要安装到系统目录中，可以以 root 身份运行命令或使用`sudo`。为了方便本示例，我们将继续作为当前用户操作。

## 如何做...

使用`file_line`资源类型，我们可以确保某一行在配置文件中存在或不存在。使用`file_line`我们可以快速对文件进行编辑，而无需控制整个文件。

1.  创建一个名为`oneline.pp`的清单文件，该文件将使用`file_line`对`/tmp`中的文件进行操作：

    ```
      file {'/tmp/cookbook':
        ensure => 'file',
      }
      file_line {'cookbook-hello':
        path    => '/tmp/cookbook',
        line    => 'Hello World!',
        require => File['/tmp/cookbook'],
      }
    ```

1.  在`oneline.pp`清单上运行`puppet apply`：

    ```
    t@mylaptop ~/.puppet/manifests $ puppet apply oneline.pp 
    Notice: Compiled catalog for mylaptop in environment production in 0.39 seconds
    Notice: /Stage[main]/Main/File[/tmp/cookbook]/ensure: created
    Notice: /Stage[main]/Main/File_line[cookbook-hello]/ensure: created
    Notice: Finished catalog run in 0.02 seconds

    ```

1.  现在验证`/tmp/cookbook`是否包含我们定义的那一行：

    ```
    t@mylaptop ~/.puppet/manifests $ cat /tmp/cookbook
    Hello World!

    ```

## 它是如何工作的...

我们将`puppetlabs-stdlib`模块安装到了 Puppet 的默认模块路径中，因此当我们运行`puppet apply`时，Puppet 知道去哪里找到`file_line`类型的定义。然后，Puppet 会在`/tmp/cookbook`文件不存在时创建该文件。由于文件中没有找到`Hello World!`这一行，Puppet 将这行添加到文件中。

## 还有更多…

我们可以定义更多的`file_line`实例，并向文件中添加更多行；我们可以让多个资源修改同一个文件。

修改`oneline.pp`文件并添加另一个`file_line`资源：

```
  file {'/tmp/cookbook':
    ensure => 'file',
  }
  file_line {'cookbook-hello':
    path    => '/tmp/cookbook',
    line    => 'Hello World!',
    require => File['/tmp/cookbook'],
  }
  file_line {'cookbook-goodbye':
    path    => '/tmp/cookbook',
    line    => 'So long, and thanks for all the fish.',
    require => File['/tmp/cookbook'],
  }
```

现在再次应用清单并验证新行是否已追加到文件中：

```
t@mylaptop ~/.puppet/manifests $ puppet apply oneline.pp 
Notice: Compiled catalog for mylaptop in environment production in 0.36 seconds
Notice: /Stage[main]/Main/File_line[cookbook-goodbye]/ensure: created
Notice: Finished catalog run in 0.02 seconds
t@mylaptop ~/.puppet/manifests $ cat /tmp/cookbook 
Hello World!
So long, and thanks for all the fish.

```

`file_line`类型也支持模式匹配和行删除，正如我们将在下面的示例中展示的：

```
  file {'/tmp/cookbook':
    ensure => 'file',
  }
  file_line {'cookbook-remove':
    ensure  => 'absent',
    path    => '/tmp/cookbook',
    line    => 'Hello World!',
    require => File['/tmp/cookbook'],
  }
  file_line {'cookbook-match':
    path    => '/tmp/cookbook',
    line    => 'Oh freddled gruntbuggly, thanks for all the fish.',
    match   => 'fish.$',
    require => File['/tmp/cookbook'],
  }
```

在运行 Puppet 之前，验证`/tmp/cookbook`的内容：

```
t@mylaptop ~/.puppet/manifests $ cat /tmp/cookbook 
Hello World!
So long, and thanks for all the fish.

```

应用更新后的清单：

```
t@mylaptop ~/.puppet/manifests $ puppet apply oneline.pp 
Notice: Compiled catalog for mylaptop in environment production in 0.30 seconds
Notice: /Stage[main]/Main/File_line[cookbook-match]/ensure: created
Notice: /Stage[main]/Main/File_line[cookbook-remove]/ensure: removed
Notice: Finished catalog run in 0.02 seconds

```

验证该行已被删除，且“goodbye”行已被替换：

```
t@mylaptop ~/.puppet/manifests $ cat /tmp/cookbook 
Oh freddled gruntbuggly, thanks for all the fish.

```

使用`file_line`编辑文件对于结构简单的文件效果很好。结构化文件可能在不同部分有相似的行，但其含义不同。在接下来的部分中，我们将向你展示如何处理一种特定类型的结构化文件——使用**INI 语法**的文件。

# 使用 puppetlabs-inifile 编辑 INI 风格的文件

INI 文件在许多系统中都有使用，Puppet 在`puppet.conf`文件中使用 INI 语法。`puppetlabs-inifile`模块创建了两种类型，`ini_setting`和`ini_subsetting`，它们可以用来编辑 INI 风格的文件。

## 准备就绪

按照以下方式从 Forge 安装模块：

```
t@mylaptop ~ $ puppet module install puppetlabs-inifile
Notice: Preparing to install into /home/tuphill/.puppet/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/home/tuphill/.puppet/modules
└── puppetlabs-inifile (v1.1.3)

```

## 如何操作……

在这个示例中，我们将创建一个`/tmp/server.conf`文件，并确保该文件中设置了`server_true`：

1.  创建一个`initest.pp`清单，内容如下：

    ```
      ini_setting {'server_true':
        path    => '/tmp/server.conf',
        section => 'main',
        setting => 'server',
        value   => 'true',
      }
    ```

1.  应用清单：

    ```
    t@mylaptop ~/.puppet/manifests $ puppet apply initest.pp 
    Notice: Compiled catalog for burnaby in environment production in 0.14 seconds
    Notice: /Stage[main]/Main/Ini_setting[server_true]/ensure: created
    Notice: Finished catalog run in 0.02 seconds

    ```

1.  验证`/tmp/server.conf`文件的内容：

    ```
    t@mylaptop ~/.puppet/manifests $ cat /tmp/server.conf 

    [main]
    server = true

    ```

## 它是如何工作的……

`inifile`模块定义了两种类型，`ini_setting`和`ini_subsetting`。我们的清单定义了一个`ini_setting`资源，它在`ini`文件的主部分中创建了一个`server = true`设置。在我们的例子中，文件不存在，所以 Puppet 创建了该文件，然后创建了`main`部分，最后将设置添加到`main`部分。

## 还有更多……

使用`ini_subsetting`，你可以将多个资源添加到一个设置中。例如，我们的`server.conf`文件中有一行 server，我们可以让每个节点将其主机名追加到这行 server 后面。将以下内容添加到`initest.pp`文件的末尾：

```
  ini_subsetting {'server_name':
    path    => '/tmp/server.conf',
    section => 'main',
    setting => 'server_host',
    subsetting => "$hostname",
  }
```

应用清单：

```
t@mylaptop ~/.puppet/manifests $ puppet apply initest.pp 
Notice: Compiled catalog for mylaptop in environment production in 0.34 seconds
Notice: /Stage[main]/Main/Ini_subsetting[server_name]/ensure: created
Notice: Finished catalog run in 0.02 seconds
t@mylaptop ~/.puppet/manifests $ cat /tmp/server.conf 
[main]
server = true
server_host = mylaptop

```

现在临时更改你的主机名并重新运行 Puppet：

```
t@mylaptop ~/.puppet/manifests $ sudo hostname inihost
t@mylaptop ~/.puppet/manifests $ puppet apply initest.pp 
Notice: Compiled catalog for inihost in environment production in 0.43 seconds
Notice: /Stage[main]/Main/Ini_subsetting[server_name]/ensure: created
Notice: Finished catalog run in 0.02 seconds
t@mylaptop ~/.puppet/manifests $ cat /tmp/server.conf 
[main]
server = true
server_host = mylaptop inihost

```

### 提示

在处理 INI 语法文件时，使用`inifile`模块是一个极好的选择。

如果你的配置文件不是 INI 语法格式，可以使用另一个工具 Augeas。在接下来的部分中，我们将使用`augeas`来修改文件。

# 使用 Augeas 可靠地编辑配置文件

有时候似乎每个应用程序都有自己微妙不同的配置文件格式，而编写正则表达式来解析和修改这些文件可能是件乏味的事。

幸好有 Augeas 来帮忙。Augeas 是一个旨在简化不同配置文件格式处理的系统，它将所有文件呈现为一个简单的值树。Puppet 对 Augeas 的支持允许你创建`augeas`资源，这些资源能够智能且自动地进行所需的配置更改。

## 如何操作…

按照以下步骤创建一个示例`augeas`资源：

1.  按如下方式修改你的`base`模块：

    ```
      class base {
        augeas { 'enable-ip-forwarding':
          incl    => '/etc/sysctl.conf',
          lens    => 'Sysctl.lns',
          changes => ['set net.ipv4.ip_forward 1'],
        }
      }
    ```

1.  运行 Puppet：

    ```
    [root@cookbook ~]# puppet agent -t
    Info: Applying configuration version '1412130479'
    Notice: Augeasenable-ip-forwarding: 
    --- /etc/sysctl.conf	2014-09-04 03:41:09.000000000 -0400
    +++ /etc/sysctl.conf.augnew	2014-09-30 22:28:03.503000039 -0400
    @@ -4,7 +4,7 @@
     # sysctl.conf(5) for more details.

     # Controls IP packet forwarding
    -net.ipv4.ip_forward = 0
    +net.ipv4.ip_forward = 1

     # Controls source route verification
     net.ipv4.conf.default.rp_filter = 1
    Notice: /Stage[main]/Base/Augeas[enable-ip-forwarding]/returns: executed successfully
    Notice: Finished catalog run in 2.27 seconds

    ```

1.  检查设置是否已正确应用：

    ```
    [root@cookbook ~]# sysctl -p |grep ip_forward
    net.ipv4.ip_forward = 1

    ```

## 它是如何工作的……

我们声明一个名为`enable-ip-forwarding`的`augeas`资源：

```
augeas { 'enable-ip-forwarding':
```

我们指定希望在文件`/etc/sysctl.conf`中进行更改：

```
incl => '/etc/sysctl.conf',
```

接下来，我们指定要在该文件上使用的镜头。Augeas 使用名为“镜头”的文件，将配置文件转换为对象表示。Augeas 默认附带多个镜头，位于`/usr/share/augeas/lenses`目录。指定`augeas`资源中的镜头时，镜头名称会被大写并带有`.lns`后缀。在本例中，我们将指定`Sysctl`镜头，具体如下：

```
lens => 'Sysctl.lns',
```

`changes`参数指定我们希望进行的更改。它的值是一个数组，因为我们可以一次提供多个更改。在这个示例中，只有一个更改，因此其值是一个包含一个元素的数组：

```
changes => ['set net.ipv4.ip_forward 1'],
```

一般来说，Augeas 的更改形式如下：

```
set <parameter> <value>
```

在这种情况下，设置将在`/etc/sysctl.conf`中被翻译为如下所示的一行：

```
net.ipv4.ip_forward=1
```

## 还有更多…

我选择`/etc/sysctl.conf`作为示例，因为它可以包含多种内核设置，你可能希望出于各种不同的目的以及在不同的 Puppet 类中更改这些设置。比如在示例中，你可能希望为路由器类启用 IP 转发，但你也可能希望为负载均衡器类调整`net.core.somaxconn`的值。

这意味着仅仅将`/etc/sysctl.conf`文件以文本文件形式传递并分发是行不通的，因为根据你要修改的设置，可能会有多个不同且冲突的版本。在这里，Augeas 是一个正确的解决方案，因为你可以在不同的位置定义`augeas`资源，这些资源会修改同一个文件，且它们不会发生冲突。

关于如何使用 Puppet 和 Augeas 的更多信息，请参阅 Puppet Labs 网站上的页面[`projects.puppetlabs.com/projects/1/wiki/Puppet_Augeas`](http://projects.puppetlabs.com/projects/1/wiki/Puppet_Augeas)。

另一个使用 Augeas 的项目是**Augeasproviders**。Augeasproviders 使用 Augeas 定义了几种类型。其一是`sysctl`类型，使用此类型可以在不需要了解如何在 Augeas 中编写更改的情况下进行 sysctl 更改。有关更多信息，请访问[`forge.puppetlabs.com/domcleal/augeasproviders`](https://forge.puppetlabs.com/domcleal/augeasproviders)。

一开始学习如何使用 Augeas 可能有点令人困惑。Augeas 提供了一个命令行工具`augtool`，可以用来熟悉在 Augeas 中进行更改。

# 使用片段构建配置文件

有时你不能将整个配置文件一次性部署，而逐行编辑又不够。通常，你需要从由不同类管理的各种配置片段中构建配置文件。你可能还会遇到需要将本地信息导入到文件中的情况。在这个示例中，我们将使用本地文件以及我们在清单中定义的片段来构建配置文件。

## 准备工作

虽然我们可以创建自己的系统从各个部分构建文件，但我们将使用 Puppetlabs 支持的 `concat` 模块。我们将从安装 `concat` 模块开始，在之前的示例中我们将模块安装到了本地机器中。在这个示例中，我们将修改 Puppet 服务器配置，并将模块下载到 Puppet 服务器。

在你的 Git 仓库中创建一个 `environment.conf` 文件，内容如下：

```
modulepath = public:modules
manifest = manifests/site.pp
```

创建公共目录并将模块下载到该目录，方法如下：

```
t@mylaptop ~/puppet $ mkdir public && cd public
t@mylaptop ~/puppet/public $ puppet module install puppetlabs-concat --modulepath=.
Notice: Preparing to install into /home/thomas/puppet/public ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/home/thomas/puppet/public
└─┬ puppetlabs-concat (v1.1.1)
 └── puppetlabs-stdlib (v4.3.2)

```

现在将新模块添加到我们的 Git 仓库中：

```
t@mylaptop ~/puppet/public $ git add .
t@mylaptop ~/puppet/public $ git commit -m "adding concat"
[production 50c6fca] adding concat
 407 files changed, 20089 insertions(+)

```

然后推送到我们的 Git 服务器：

```
t@mylaptop ~/puppet/public $ git push origin production

```

## 如何操作……

现在我们已经在服务器上可用 `concat` 模块，我们可以在 `base` 模块中创建一个 `concat` 容器资源：

```
  concat {'hosts.allow':
    path => '/etc/hosts.allow',
    mode => 0644
  }
```

为新文件的头部创建一个`concat::fragment`模块：

```
  concat::fragment {'hosts.allow header':
    target  => 'hosts.allow',
    content => "# File managed by puppet\n",
    order   => '01'
  }
```

创建一个包含本地文件的`concat::fragment`：

```
  concat::fragment {'hosts.allow local':
    target => 'hosts.allow',
    source => '/etc/hosts.allow.local',
    order  => '10',
  }
```

创建一个将在文件末尾的 `concat::fragment` 模块：

```
  concat::fragment {'hosts.allow tftp':
    target  => 'hosts.allow',
    content => "in.ftpd: .example.com\n",
    order   => '50',
  }
```

在节点上，创建 `/etc/hosts.allow.local`，内容如下：

```
  in.tftpd: .example.com
```

运行 Puppet 以创建文件：

```
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1412138600'
Notice: /Stage[main]/Base/Concat[hosts.allow]/File[hosts.allow]/ensure: defined content as '{md5}b151c8bbc32c505f1c4a98b487f7d249'
Notice: Finished catalog run in 0.29 seconds

```

验证新文件的内容如下：

```
[root@cookbook ~]# cat /etc/hosts.allow
# File managed by puppet
in.tftpd: .example.com
in.ftpd: .example.com

```

## 它是如何工作的……

`concat` 资源定义了一个容器，将包含所有随后的 `concat::fragment` 资源。每个 `concat::fragment` 资源都将 `concat` 资源作为目标。每个 `concat::fragment` 还包括一个 `order` 属性。`order` 属性用于指定将片段添加到最终文件的顺序。我们的 `/etc/hosts.allow` 文件由头行、本地文件的内容以及我们定义的 `in.tftpd` 行组成。

# 使用 ERB 模板

尽管你可以像简单的文本文件一样使用 Puppet 部署配置文件，但模板更为强大。模板文件可以进行计算、执行 Ruby 代码，或引用 Puppet 清单中的变量值。你可以在任何使用 Puppet 部署文本文件的地方，使用模板代替。

在最简单的情况下，模板可以只是一个静态文本文件。更有用的是，你可以使用 ERB（嵌入式 Ruby）语法将变量插入其中。例如：

```
  <%= @name %>, this is a very large drink.
```

如果模板在一个变量 `$name` 包含 `Zaphod Beeblebrox` 的上下文中使用，那么模板将评估为：

```
  Zaphod Beeblebrox, this is a very large drink.
```

这种简单的技术对于生成大量仅在一个或两个变量值上有所不同的文件非常有用，例如虚拟主机，并且可以将值插入到脚本中，比如数据库名称和密码。

## 如何操作……

在这个例子中，我们将使用一个 ERB 模板将密码插入到备份脚本中：

1.  创建文件 `modules/admin/templates/backup-mysql.sh.erb`，内容如下：

    ```
    #!/bin/sh
    /usr/bin/mysqldump -uroot \ -p<%= @mysql_password %> \ --all-databases | \ /bin/gzip > /backup/mysql/all-databases.sql.gz
    ```

1.  按照如下方式修改你的 `site.pp` 文件：

    ```
    node 'cookbook' {
      $mysql_password = 'secret'
      file { '/usr/local/bin/backup-mysql':
        content => template('admin/backup-mysql.sh.erb'),
        mode    => '0755',
      }
    }
    ```

1.  运行 Puppet：

    ```
    [root@cookbook ~]# puppet agent -t
    Info: Caching catalog for cookbook.example.com
    Info: Applying configuration version '1412140971'
    Notice: /Stage[main]/Main/Node[cookbook]/File[/usr/local/bin/backup-mysql]/ensure: defined content as '{md5}c12af56559ef36529975d568ff52dca5'
    Notice: Finished catalog run in 0.31 seconds

    ```

1.  检查 Puppet 是否正确地将密码插入模板：

    ```
    [root@cookbook ~]# cat /usr/local/bin/backup-mysql 
    #!/bin/sh
    /usr/bin/mysqldump -uroot \
     -psecret \
     --all-databases | \
     /bin/gzip > /backup/mysql/all-databases.sql.gz

    ```

## 它是如何工作的……

在模板中引用的每个变量，例如 `<%= @mysql_password %>`，Puppet 将用相应的值 `secret` 替换它。

## 还有更多……

在示例中，我们只使用了一个变量，但你可以根据需要使用多个变量。它们也可以是事实：

```
ServerName <%= @fqdn %>
```

或者是 Ruby 表达式：

```
MAILTO=<%= @emails.join(',') %>
```

或者是你想要的任何 Ruby 代码：

```
ServerAdmin <%= @sitedomain == 'coldcomfort.com' ? 'seth@coldcomfort.com' : 'flora@poste.com' %>
```

## 另请参阅

+   本章中的 *使用 GnuPG 加密机密信息* 配方

+   [`docs.puppetlabs.com/guides/templating.html`](https://docs.puppetlabs.com/guides/templating.html)

# 在模板中使用数组迭代

在前面的示例中，我们看到你可以使用 Ruby 根据表达式的结果在模板中插入不同的值。但是，你并不局限于一次插入一个值。你可以将多个值放入 Puppet 数组中，然后通过循环让模板为数组中的每个元素生成内容。

## 如何实现…

按照以下步骤构建一个数组迭代的示例：

1.  按照以下方式修改你的 `site.pp` 文件：

    ```
      node 'cookbook' {
        $ipaddresses = ['192.168.0.1', '158.43.128.1', '10.0.75.207' ]
        file { '/tmp/addresslist.txt':
          content => template('base/addresslist.erb')
        }
      }
    ```

1.  创建文件 `modules/base/templates/addresslist.erb`，并写入以下内容：

    ```
    <% @ipaddresses.each do |ip| -%>
    IP address <%= ip %> is present
    <% end -%>
    ```

1.  运行 Puppet：

    ```
    [root@cookbook ~]# puppet agent -t
    Info: Caching catalog for cookbook.example.com
    Info: Applying configuration version '1412141917'
    Notice: /Stage[main]/Main/Node[cookbook]/File[/tmp/addresslist.txt]/ensure: defined content as '{md5}073851229d7b2843830024afb2b3902d'
    Notice: Finished catalog run in 0.30 seconds

    ```

1.  检查生成文件的内容：

    ```
    [root@cookbook ~]# cat /tmp/addresslist.txt 
     IP address 192.168.0.1 is present.
     IP address 158.43.128.1 is present.
     IP address 10.0.75.207 is present.

    ```

## 它是如何工作的…

在模板的第一行，我们引用了数组 `ipaddresses`，并调用了它的 `each` 方法：

```
<% @ipaddresses.each do |ip| -%>
```

在 Ruby 中，这会创建一个循环，每次迭代都会执行一次，并且每次循环时，变量 `ip` 会被设置为当前元素的值。

在我们的示例中，`ipaddresses` 数组包含三个元素，因此接下来的行将执行三次，每次针对一个元素：

```
IP address <%= ip %> is present.
```

这将生成三行输出：

```
IP address 192.168.0.1 is present.
IP address 158.43.128.1 is present.
IP address 10.0.75.207 is present.
```

最后一行结束循环：

```
<% end -%>
```

### 注意

请注意，第一行和最后一行以 `-%>` 结束，而不是我们之前看到的 `%>`。`-` 的作用是抑制每次循环时生成的新行，这样就不会在文件中产生不必要的空行。

## 还有更多内容…

模板还可以对哈希或哈希数组进行迭代：

```
$interfaces = [ {name => 'eth0', ip => '192.168.0.1'},
  {name => 'eth1', ip => '158.43.128.1'},
  {name => 'eth2', ip => '10.0.75.207'} ]

<% @interfaces.each do |interface| -%>
Interface <%= interface['name'] %> has the address <%= interface['ip'] %>.
<% end -%>

Interface eth0 has the address 192.168.0.1.
Interface eth1 has the address 158.43.128.1.
Interface eth2 has the address 10.0.75.207.
```

## 另请参阅

+   本章中的 *使用 ERB 模板* 配方

# 使用 EPP 模板

EPP 模板是 Puppet 3.5 及更新版本中的新特性。EPP 模板使用类似于 ERB 模板的语法，但不通过 Ruby 编译。定义了两个新函数来调用 EPP 模板，`epp` 和 `inline_epp`。这两个函数分别是 ERB 函数 `template` 和 `inline_template` 的 EPP 等效函数。EPP 模板的主要区别是，变量是使用 Puppet 语法引用的，`$variable` 而不是 `@variable`。

## 如何实现…

1.  在 `~/puppet/epp-test.epp` 中创建一个 EPP 模板，内容如下：

    ```
    This is <%= $message %>.
    ```

1.  创建一个 `epp.pp` 清单，使用 `epp` 和 `inline_epp` 函数：

    ```
    $message = "the message"
    file {'/tmp/epp-test':
      content => epp('/home/thomas/puppet/epp-test.epp')
    }
    notify {inline_epp('Also prints <%= $message %>'):}
    ```

1.  应用清单时，请确保使用未来解析器（未来解析器是定义 `epp` 和 `inline_epp` 函数所必需的）：

    ```
    t@mylaptop ~/puppet $ puppet apply epp.pp --parser=future
    Notice: Compiled catalog for mylaptop in environment production in 1.03 seconds
    Notice: /Stage[main]/Main/File[/tmp/epp-test]/ensure: defined content as '{md5}999ccc2507d79d50fae0775d69b63b8c'
    Notice: Also prints the message

    ```

1.  验证模板是否按预期工作：

    ```
    t@mylaptop ~/puppet $ cat /tmp/epp-test 
    This is the message.

    ```

## 它是如何工作的...

使用未来解析器，定义了 `epp` 和 `inline_epp` 函数。EPP 模板与 ERB 模板的主要区别在于，变量是以与 Puppet 清单中相同的方式进行引用的。

## 还有更多内容…

`epp`和`inline_epp`都允许在函数调用中重写变量。函数调用的第二个参数可以用于为函数作用域内使用的变量指定值。例如，我们可以用以下代码重写`$message`的值：

```
file {'/tmp/epp-test':
  content => epp('/home/tuphill/puppet/epp-test.epp',
    { 'message' => "override $message"} )
}
notify {inline_epp('Also prints <%= $message %>',
  { 'message' => "inline override $message"}):}
```

现在，当我们运行 Puppet 并验证输出时，我们看到`$message`的值已被重写：

```
t@mylaptop ~/puppet $ puppet apply epp.pp --parser=future
Notice: Compiled catalog for mylaptop.pan.costco.com in environment production in 0.85 seconds
Notice: Also prints inline override the message
Notice: Finished catalog run in 0.05 seconds
t@mylaptop ~/puppet $ cat /tmp/epp-test 
This is override the message.

```

# 使用 GnuPG 加密机密信息

我们通常需要 Puppet 访问机密信息，例如密码或加密密钥，以便它能够正确配置系统。但如何避免将这些机密信息直接放入 Puppet 代码中呢？这样会导致任何具有读取权限的人都能看到这些信息。

第三方开发人员和承包商通常需要通过 Puppet 进行更改，但他们绝对不应看到任何机密信息。类似地，如果您使用的是如第二章所描述的分布式 Puppet 设置，*Puppet 基础设施*，那么每台机器都有整个仓库的副本，其中包括它不需要且不应拥有的其他机器的机密信息。我们如何防止这种情况发生？

一个方法是使用**GnuPG**工具加密机密信息，以便在没有适当密钥的情况下，Puppet 仓库中的任何机密信息都无法被解密（在实际操作中）。然后，我们将密钥安全地分发给需要它的人或机器。

## 准备工作

首先，您需要一个加密密钥，请按照以下步骤生成一个。如果您已经有一个想要使用的 GnuPG 密钥，可以跳过此部分，进入下一节。要完成此部分，您需要安装 gpg 命令：

1.  使用`puppet`资源安装 gpg：

    ```
    # puppet resource package gnupg ensure=installed

    ```

    ### 提示

    根据目标操作系统的不同，您可能需要使用 gnupg2 作为包名称。

1.  运行以下命令。按照提示回答问题，除了将我的名字和电子邮件地址替换为您的信息。当系统提示输入密码短语时，直接按*Enter*：

    ```
    t@mylaptop ~/puppet $ gpg --gen-key
    gpg (GnuPG) 1.4.18; Copyright (C) 2014 Free Software Foundation, Inc.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    Please select what kind of key you want:
     (1) RSA and RSA (default)
     (2) DSA and Elgamal
     (3) DSA (sign only)
     (4) RSA (sign only)
    Your selection? 1
    RSA keys may be between 1024 and 4096 bits long.
    What keysize do you want? (2048) 2048
    Requested keysize is 2048 bits
    Please specify how long the key should be valid.
     0 = key does not expire
     <n>  = key expires in n days
     <n>w = key expires in n weeks
     <n>m = key expires in n months
     <n>y = key expires in n years
    Key is valid for? (0) 0
    Key does not expire at all
    Is this correct? (y/N) y
    You need a user ID to identify your key; the software constructs the user ID
    from the Real Name, Comment and Email Address in this form:
     "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

    Real name: Thomas Uphill
    Email address: thomas@narrabilis.com
    Comment: <enter>
    You selected this USER-ID:
     "Thomas Uphill <thomas@narrabilis.com>"

    Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
    You need a Passphrase to protect your secret key.

    ```

    在这里按两次回车以设置空的密码短语

    ```
    You don't want a passphrase - this is probably a *bad* idea!
    I will do it anyway.  You can change your passphrase at any time,
    using this program with the option "--edit-key".

    gpg: key F1C1EE49 marked as ultimately trusted
    public and secret key created and signed.

    gpg: checking the trustdb
    gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
    gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
    pub   2048R/F1C1EE49 2014-10-01
     Key fingerprint = 461A CB4C 397F 06A7 FB82  3BAD 63CF 50D8 F1C1 EE49
    uid                  Thomas Uphill <thomas@narrabilis.com>
    sub   2048R/E2440023 2014-10-01

    ```

1.  如果您的系统没有配置随机源，您可能会看到类似这样的消息：

    ```
    We need to generate a lot of random bytes. It is a good idea to perform
    some other action (type on the keyboard, move the mouse, utilize the
    disks) during the prime generation; this gives the random number
    generator a better chance to gain enough entropy.

    ```

1.  在这种情况下，安装并启动一个随机数生成守护进程，如`haveged`或`rng-tools`。将刚刚创建的 gpg 密钥复制到 Puppet 主机上`puppet`用户的账户中：

    ```
    t@mylaptop ~ $ scp -r .gnupg puppet@puppet.example.com:
    gpg.conf                                      100% 7680     7.5KB/s   00:00 
    random_seed                                   100%  600     0.6KB/s   00:00 
    pubring.gpg                                   100% 1196     1.2KB/s   00:00 
    secring.gpg                                   100% 2498     2.4KB/s   00:00 
    trustdb.gpg                                   100% 1280     1.3KB/s   00:00

    ```

## 如何操作...

当您的加密密钥安装在`puppet`用户的密钥环中（上一节描述的密钥生成过程会为您完成这项工作）时，您已经准备好配置 Puppet 来解密机密信息。

1.  创建以下目录：

    ```
    t@cookbook:~/puppet$ mkdir -p modules/admin/lib/puppet/parser/functions

    ```

1.  创建文件`modules/admin/lib/puppet/parser/functions/secret.rb`，并添加以下内容：

    ```
    module Puppet::Parser::Functions
      newfunction(:secret, :type => :rvalue) do |args|
        'gpg --no-tty -d #{args[0]}'
      end
    end
    ```

1.  创建文件`secret_message`，并添加以下内容：

    ```
    For a moment, nothing happened.
    Then, after a second or so, nothing continued to happen.
    ```

1.  使用以下命令加密此文件（使用您在创建 GnuPG 密钥时提供的电子邮件地址）：

    ```
    t@mylaptop ~/puppet $ gpg -e -r thomas@narrabilis.com secret_message

    ```

1.  将生成的加密文件移动到 Puppet 仓库中：

    ```
    t@mylaptop:~/puppet$ mv secret_message.gpg modules/admin/files/

    ```

1.  删除原始的（明文）文件：

    ```
    t@mylaptop:~/puppet$ rm secret_message

    ```

1.  按如下方式修改您的`site.pp`文件：

    ```
    node 'cookbook' {
      $message = secret('/etc/puppet/environments/production/ modules/admin/files/secret_message.gpg')
      notify { "The secret message is: ${message}": }
    }
    ```

1.  运行 Puppet：

    ```
    [root@cookbook ~]# puppet agent -t
    Info: Caching catalog for cookbook.example.com
    Info: Applying configuration version '1412145910'
    Notice: The secret message is: For a moment, nothing happened. 
    Then, after a second or so, nothing continued to happen.
    Notice: Finished catalog run in 0.27 seconds

    ```

## 它是如何工作的...

首先，我们创建了一个自定义函数，允许 Puppet 使用 GnuPG 解密秘密文件：

```
module Puppet::Parser::Functions
  newfunction(:secret, :type => :rvalue) do |args|
    'gpg --no-tty -d #{args[0]}'
  end
end
```

上述代码创建了一个名为`secret`的函数，该函数接受一个文件路径作为参数并返回解密后的文本。它不管理加密密钥，因此您需要确保`puppet`用户已安装必要的密钥。您可以通过以下命令检查：

```
puppet@puppet:~ $ gpg --list-secret-keys
/var/lib/puppet/.gnupg/secring.gpg
----------------------------------
sec   2048R/F1C1EE49 2014-10-01
uid                  Thomas Uphill <thomas@narrabilis.com>
ssb   2048R/E2440023 2014-10-01

```

设置好`secret`函数和所需的密钥后，我们现在为该密钥加密一条消息：

```
tuphill@mylaptop ~/puppet $ gpg -e -r thomas@narrabilis.com secret_message

```

这会创建一个加密文件，只有拥有密钥访问权限的人（或者在已安装密钥的机器上运行 Puppet 的人）才能读取。

然后我们调用`secret`函数解密此文件并获取内容：

```
$message = secret(' /etc/puppet/environments/production/modules/admin/files/secret_message.gpg')
```

## 还有更多内容…

您应该使用`secret`函数，或类似的功能，来保护您 Puppet 仓库中的任何机密数据：密码、AWS 凭证、许可证密钥，甚至其他秘密密钥如 SSL 主机密钥。

您可以决定使用一个单一的密钥，并在构建机器时将其推送到机器上，或许可以作为引导过程的一部分，像第二章中的*使用 Bash 引导 Puppet*食谱所描述的那样。为了更高的安全性，您可能会为每台机器或机器组创建一个新的密钥，并仅针对需要它的机器加密给定的秘密。

例如，您的 web 服务器可能需要某个秘密信息，而您不希望该信息在其他任何机器上可以访问。您可以为 web 服务器创建一个密钥，并仅针对该密钥加密数据。

如果您想使用加密数据与 Hiera 配合使用，可以使用一个 Hiera 的 GnuPG 后端，详情见 [`www.craigdunn.org/2011/10/secret-variables-in-puppet-with-hiera-and-gpg/`](http://www.craigdunn.org/2011/10/secret-variables-in-puppet-with-hiera-and-gpg/)。

## 另见

+   第二章中的*配置 Hiera*食谱，*Puppet 基础设施*

+   第二章中的*使用 hiera-gpg 存储秘密数据*食谱，*Puppet 基础设施*

# 从第三方仓库安装软件包

最常见的是，您会希望从主分发仓库安装软件包，因此一个简单的软件包资源就足够了：

```
package { 'exim4': ensure => installed }
```

有时，您可能需要一个仅在第三方仓库中找到的软件包（例如 Ubuntu PPA），或者您可能需要比分发版提供的更近期的包版本，而这些版本可以从第三方获取。

在手动管理的机器上，通常通过将仓库源配置添加到`/etc/apt/sources.list.d`（如有必要，还需添加仓库的 gpg 密钥）来进行操作，然后再安装软件包。我们可以通过 Puppet 很容易地自动化此过程。

## 如何操作…

在这个示例中，我们将使用流行的 Percona APT 仓库（Percona 是一家 MySQL 咨询公司，他们维护并发布自己专门的 MySQL 版本，更多信息请访问 [`www.percona.com/software/repositories`](http://www.percona.com/software/repositories)）：

1.  创建文件 `modules/admin/manifests/percona_repo.pp`，并添加以下内容：

    ```
    # Install Percona APT repo
    class admin::percona_repo {
      exec { 'add-percona-apt-key':
        unless  => '/usr/bin/apt-key list |grep percona',
        command => '/usr/bin/gpg --keyserver hkp://keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A && /usr/bin/gpg -a --export CD2EFD2A | apt-key add -',
        notify  => Exec['percona-apt-update'],
      }

      exec { 'percona-apt-update':
        command     => '/usr/bin/apt-get update',
        require     => [File['/etc/apt/sources.list.d/percona.list'],
    File['/etc/apt/preferences.d/00percona.pref']],
        refreshonly => true,
      }

      file { '/etc/apt/sources.list.d/percona.list':
        content => 'deb http://repo.percona.com/apt wheezy main',
        notify  => Exec['percona-apt-update'],
      }

      file { '/etc/apt/preferences.d/00percona.pref':
        content => "Package: *\nPin: release o=Percona
        Development Team\nPin-Priority: 1001",
        notify  => Exec['percona-apt-update'],
      }
    }
    ```

1.  按照以下方式修改你的 `site.pp` 文件：

    ```
    node 'cookbook' {
      include admin::percona_repo

      package { 'percona-server-server-5.5':
        ensure  => installed,
        require => Class['admin::percona_repo'],
      }
    }
    ```

1.  运行 Puppet：

    ```
    root@cookbook-deb:~# puppet agent -t
    Info: Caching catalog for cookbook-deb
    Notice: /Stage[main]/Admin::Percona_repo/Exec[add-percona-apt-key]/returns: executed successfully
    Info: /Stage[main]/Admin::Percona_repo/Exec[add-percona-apt-key]: Scheduling refresh of Exec[percona-apt-update]
    Notice: /Stage[main]/Admin::Percona_repo/File[/etc/apt/sources.list.d/percona.list]/ensure: defined content as '{md5}b8d479374497255804ffbf0a7bcdf6c2'
    Info: /Stage[main]/Admin::Percona_repo/File[/etc/apt/sources.list.d/percona.list]: Scheduling refresh of Exec[percona-apt-update]
    Notice: /Stage[main]/Admin::Percona_repo/File[/etc/apt/preferences.d/00percona.pref]/ensure: defined content as '{md5}1d8ca6c1e752308a9bd3018713e2d1ad'
    Info: /Stage[main]/Admin::Percona_repo/File[/etc/apt/preferences.d/00percona.pref]: Scheduling refresh of Exec[percona-apt-update]
    Notice: /Stage[main]/Admin::Percona_repo/Exec[percona-apt-update]: Triggered 'refresh' from 3 events

    ```

## 它是如何工作的……

要安装任何 Percona 包，我们首先需要在机器上安装仓库配置。这就是为什么 `percona-server-server-5.5` 包（Percona 版本的标准 MySQL 服务器）需要 `admin::percona_repo` 类的原因：

```
package { 'percona-server-server-5.5':
  ensure  => installed,
  require => Class['admin::percona_repo'],
}
```

那么，`admin::percona_repo` 类做了什么呢？它：

+   安装 Percona APT 密钥，安装包将使用该密钥进行签名

+   将 Percona 仓库 URL 配置为 `/etc/apt/sources.list.d` 中的文件

+   运行 `apt-get update` 来检索仓库的元数据

+   在 `/etc/apt/preferences.d` 中添加 APT pin 配置

首先，我们安装 APT 密钥：

```
exec { 'add-percona-apt-key':
  unless  => '/usr/bin/apt-key list |grep percona',
  command => '/usr/bin/gpg --keyserver  hkp://keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A && /usr/bin/gpg -a --export CD2EFD2A | apt-key add -',
  notify  => Exec['percona-apt-update'],
}
```

`unless` 参数检查 `apt-key list` 的输出，以确保 Percona 密钥尚未安装，如果已经安装，就不需要执行任何操作。如果没有安装，`command` 命令将执行：

```
/usr/bin/gpg --keyserver  hkp://keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A && /usr/bin/gpg -a --export CD2EFD2A | apt-key add -
```

此命令从 GnuPG 密钥服务器获取密钥，以 ASCII 格式导出并将其传递给 `apt-key add` 命令，从而将其添加到系统密钥链中。对于大多数需要 APT 签名密钥的第三方仓库，可以使用类似的模式。

安装密钥后，我们添加仓库配置：

```
file { '/etc/apt/sources.list.d/percona.list':
  content => 'deb http://repo.percona.com/apt wheezy main',
  notify  => Exec['percona-apt-update'],
}
```

然后运行 `apt-get update`，用新仓库的元数据更新系统的 APT 缓存：

```
exec { 'percona-apt-update':
  command     => '/usr/bin/apt-get update',
  require     => [File['/etc/apt/sources.list.d/percona.list'], File['/etc/apt/preferences.d/00percona.pref']],
  refreshonly => true,
}
```

最后，我们为仓库配置 APT pin 优先级：

```
file { '/etc/apt/preferences.d/00percona.pref':
  content => "Package: *\nPin: release o=Percona Development Team\nPin-Priority: 1001",
  notify  => Exec['percona-apt-update'],
}
```

这样可以确保从 Percona 仓库安装的包永远不会被其他地方（例如主 Ubuntu 发行版）安装的包所替代。否则，你可能会遇到依赖关系损坏的问题，无法自动安装 Percona 包。

## 还有更多内容……

APT 包框架特定于 Debian 和 Ubuntu 系统。对于管理 apt 仓库，有一个 forge 模块，[`forge.puppetlabs.com/puppetlabs/apt`](https://forge.puppetlabs.com/puppetlabs/apt)。如果你使用的是 Red Hat 或 CentOS 系统，可以直接使用 `yumrepo` 资源来管理 RPM 仓库：

[`docs.puppetlabs.com/references/latest/type.html#yumrepo`](http://docs.puppetlabs.com/references/latest/type.html#yumrepo)

# 比较包版本

包版本号是一个奇怪的东西。它们看起来像十进制数，但实际上并不是：版本号通常是类似 `2.6.4` 的形式。例如，如果你需要比较两个版本号，你不能直接进行字符串比较：`2.6.4` 会被解读为大于 `2.6.12`。而且数值比较也行不通，因为它们并不是有效的数字。

Puppet 的 `versioncmp` 函数来解救了。如果你传递两个看起来像版本号的东西，它会比较它们并返回一个值，指示哪个更大：

```
versioncmp( A, B )
```

返回：

+   如果 A 和 B 相等，则返回 0

+   如果 A 大于 B，则返回大于 1

+   如果 A 小于 B，则返回小于 0

## 如何做…

这是使用 `versioncmp` 函数的示例：

1.  按如下方式修改你的 `site.pp` 文件：

    ```
    node 'cookbook' {
      $app_version = '1.2.2'
      $min_version = '1.2.10'

      if versioncmp($app_version, $min_version) >= 0 {
        notify { 'Version OK': }
      } else {
        notify { 'Upgrade needed': }
      }
    }
    ```

1.  运行 Puppet：

    ```
    [root@cookbook ~]# puppet agent -t
    Info: Caching catalog for cookbook.example.com
    Notice: Upgrade needed

    ```

1.  现在更改 `$app_version` 的值：

    ```
    $app_version = '1.2.14'
    ```

1.  再次运行 Puppet：

    ```
    [root@cookbook ~]# puppet agent -t
    Info: Caching catalog for cookbook.example.com
    Notice: Version OK

    ```

## 它是如何工作的…

我们指定了最小可接受版本（`$min_version`）是 `1.2.10`。因此，在第一个示例中，我们要将其与 `$app_version` 的 `1.2.2` 进行比较。简单的字母顺序比较这两个字符串（例如在 Ruby 中）会得出错误的结果，但 `versioncmp` 正确地判断 `1.2.2` 小于 `1.2.10`，并提醒我们需要升级。

在第二个示例中，`$app_version` 现在是 `1.2.14`，`versioncmp` 正确地识别它大于 `$min_version`，因此我们得到了消息 **版本正常**。

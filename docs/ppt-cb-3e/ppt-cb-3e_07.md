# 第七章 管理应用程序

|   | *每个人都知道，调试比编写程序要困难两倍。那么，如果你在编写时尽可能聪明，你又怎么调试它呢？* |   |
| --- | --- | --- |
|   | --*Brian W. Kernighan.* |

在本章中，我们将介绍以下食谱：

+   使用公共模块

+   管理 Apache 服务器

+   创建 Apache 虚拟主机

+   创建 nginx 虚拟主机

+   管理 MySQL

+   创建数据库和用户

# 引言

没有应用程序，服务器只是一个非常昂贵的取暖器。在本章中，我将展示一些使用 Puppet 管理特定软件的食谱：MySQL、Apache、**nginx** 和 Ruby。我希望这些食谱本身对你有所帮助。然而，它们所使用的模式和技术适用于几乎任何软件，因此你可以轻松地将其调整为适合自己的目的。这些应用程序的一个共同点是它们很常见。大多数 Puppet 安装都需要处理一个 Web 服务器，Apache 或 nginx。大多数情况下，甚至所有情况下，都会有数据库，其中一些将使用 MySQL。当每个人都需要处理一个问题时，社区解决方案通常比自家开发的解决方案经过更好的测试和更彻底的完善。我们将在本章中使用来自 Puppet Forge 的模块来管理这些应用程序。

当你从头开始编写自己的 Apache 或 nginx 模块时，你需要关注你所支持的发行版的细微差别。一些发行版将 Apache 包称为 `httpd`，而其他发行版则使用 `apache2`；MySQL 也是如此。此外，基于 Debian 的发行版使用启用文件夹方法来启用 Apache 中的自定义站点，这些站点是虚拟站点，而基于 RPM 的发行版则没有这种方法。有关虚拟站点的更多信息，请访问 [`httpd.apache.org/docs/2.2/vhosts/`](http://httpd.apache.org/docs/2.2/vhosts/)。

# 使用公共模块

当你编写一个 Puppet 模块来管理某些软件或服务时，你不必从头开始。许多流行应用程序的社区贡献模块可以在 Puppet Forge 网站上找到。有时，社区模块正是你所需要的，你可以直接下载并开始使用。在大多数情况下，你需要做一些修改以适应你特定的需求和环境。

和所有社区项目一样，Forge 上有一些优秀的模块，也有一些不太优秀的模块。你应该阅读模块的 README 部分，决定该模块是否适用于你的安装。至少要确保你的发行版是受支持的。Puppetlabs 引入了一些受支持的模块，也就是说，如果你是企业客户，他们会支持你在安装中使用这些模块。此外，大多数 Forge 模块都涉及多个操作系统、发行版和大量用例。在很多情况下，不使用 Forge 模块就像是重新发明轮子。不过，有一点需要注意的是，Forge 模块可能比你本地的模块更复杂。你应该阅读代码并了解模块的作用。了解模块如何工作将帮助你在之后进行调试。

## 如何操作…

在这个例子中，我们将使用`puppet module`命令来查找并安装有用的`stdlib`模块，该模块包含许多帮助你开发 Puppet 代码的工具函数。它是 puppetlabs 提供的上述受支持模块之一。我将把模块下载到我的用户主目录，并手动将其安装到 Git 仓库中。要安装 puppetlabs stdlib 模块，请按照以下步骤操作：

1.  执行以下命令：

    ```
    t@mylaptop ~ $ puppet module search puppetlabs-stdlib
    Notice: Searching https://forgeapi.puppetlabs.com ...
    NAME               DESCRIPTION                         AUTHOR        KEYWORDS 
    puppetlabs-stdlib  Puppet Module Standard Library      @puppetlabs   stdlib stages 

    ```

1.  我们已经验证了我们所需的模块，因此现在将使用 `module install` 命令安装它：

    ```
    t@mylaptop ~ $ puppet module install puppetlabs-stdlib
    Notice: Preparing to install into /home/thomas/.puppet/modules ...
    Notice: Downloading from https://forgeapi.puppetlabs.com ...
    Notice: Installing -- do not interrupt ...
    /home/thomas/.puppet/modules
    └── puppetlabs-stdlib (v4.3.2)

    ```

1.  该模块现在已准备好在你的清单中使用；大多数优秀模块都会附带一个`README`文件，告诉你如何操作。

## 它是如何工作的…

你可以使用 `puppet module search` 命令搜索与你感兴趣的软件包或软件匹配的模块。要安装特定的模块，使用 `puppet module install`。你可以添加 `-i` 选项来告诉 Puppet 在哪里找到你的模块目录。

你可以浏览 Forge，查看可用的模块：[`forge.puppetlabs.com/`](http://forge.puppetlabs.com/)。

有关受支持模块的更多信息，请访问 [`forge.puppetlabs.com/supported`](https://forge.puppetlabs.com/supported)。

当前支持的模块列表可以在 [`forge.puppetlabs.com/modules?endorsements=supported`](https://forge.puppetlabs.com/modules?endorsements=supported) 查看。

## 还有更多内容…

Forge 上的模块包括一个 `metadata.json` 文件，描述该模块以及模块支持的操作系统。此文件还包含该模块所需的其他模块列表。

### 注意

该文件之前被命名为 Modulefile，且不采用 JSON 格式；旧的 Modulefile 格式在 3.6 版本中已被弃用。

正如我们在下一节中将看到的，当从 Forge 安装一个模块时，所需的依赖项也会自动安装。

并非所有公开可用的模块都在 Puppet Forge 上。一些很棒的模块可以在 GitHub 上找到：

+   [`github.com/camptocamp`](https://github.com/camptocamp)

+   [`github.com/example42`](https://github.com/example42)

尽管 Puppet Cookbook 网站本身不是一个模块集合，但它包含了许多有用且富有启发性的代码示例、模式和提示，由令人钦佩的 Dean Wilson 维护：

[`www.puppetcookbook.com/`](http://www.puppetcookbook.com/)

# 管理 Apache 服务器

Apache 是世界上最受欢迎的 Web 服务器，因此您的 Puppet 工作职责中很可能包括安装和管理 Apache。

## 如何操作...

我们将安装并使用 `puppetlabs-apache` 模块来安装并启动 Apache。这一次，当我们运行 `puppet module install` 时，我们将使用 `-i` 选项告诉 Puppet 将模块安装到我们 Git 仓库的模块目录中。

1.  使用 `puppet modules install` 安装模块：

    ```
    t@mylaptop ~/puppet $ puppet module install -i modules puppetlabs-apache
    Notice: Preparing to install into /home/thomas/puppet/modules ...
    Notice: Downloading from https://forgeapi.puppetlabs.com ...
    Notice: Installing -- do not interrupt ...
    /home/thomas/puppet/modules
    └─┬ puppetlabs-apache (v1.1.1)
     ├── puppetlabs-concat (v1.1.1)
     └── puppetlabs-stdlib (v4.3.2)

    ```

1.  将模块添加到您的 Git 仓库并推送：

    ```
    t@mylaptop ~/puppet $ git add modules/apache modules/concat modules/stdlib
    t@mylaptop ~/puppet $ git commit -m "adding puppetlabs-apache module"
    [production 395b079] adding puppetlabs-apache module
     647 files changed, 35017 insertions(+), 13 deletions(-)
     rename modules/{apache => apache.cookbook}/manifests/init.pp (100%)
     create mode 100644 modules/apache/CHANGELOG.md
     create mode 100644 modules/apache/CONTRIBUTING.md
    ...
    t@mylaptop ~/puppet $ git push origin production
    Counting objects: 277, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (248/248), done.
    Writing objects: 100% (266/266), 136.25 KiB | 0 bytes/s, done.
    Total 266 (delta 48), reused 0 (delta 0)
    remote: To puppet@puppet.example.com:/etc/puppet/environments/puppet.git
    remote:    9faaa16..395b079  production -> production

    ```

1.  在 `site.pp` 中创建一个 Web 服务器节点定义：

    ```
    node webserver {
     class {'apache': }
    }

    ```

1.  运行 Puppet 来应用默认的 Apache 模块配置：

    ```
    [root@webserver ~]# puppet agent -t
    Info: Caching certificate for webserver.example.com
    Notice: /File[/var/lib/puppet/lib/puppet/provider/a2mod]/ensure: created
    ...
    Info: Caching catalog for webserver.example.com
    ...
    Info: Class[Apache::Service]: Scheduling refresh of Service[httpd]
    Notice: /Stage[main]/Apache::Service/Service[httpd]: Triggered 'refresh' from 51 events
    Notice: Finished catalog run in 11.73 seconds

    ```

1.  验证您是否可以访问 `webserver.example.com`：

    ```
    [root@webserver ~]# curl http://webserver.example.com
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
    <html>
     <head>
     <title>Index of /</title>
     </head>
     <body>
    <h1>Index of /</h1>
    <table><tr><th><img src="img/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr><tr><th colspan="5"><hr></th></tr>
    <tr><th colspan="5"><hr></th></tr>
    </table>
    </body></html>

    ```

## 它是如何工作的...

从 Forge 安装 puppetlabs-Apache 模块会同时安装 puppetlabs-concat 和 puppetlabs-stdlib 到我们的模块目录中。concat 模块用于将文件片段按特定顺序拼接在一起。它被 Apache 模块用于创建主要的 Apache 配置文件。

然后我们定义了一个 Web 服务器节点，并将 Apache 类应用到该节点上。我们使用了所有默认值，并让 Apache 模块将我们的服务器配置为 Apache Web 服务器。

然后，Apache 模块会重写我们所有的 Apache 配置文件。默认情况下，该模块会清除 Apache 目录中的所有配置文件（`/etc/apache2` 或 `/etc/httpd`，具体取决于发行版）。该模块可以配置多种不同的发行版，并处理每个发行版的细微差别。作为该模块的用户，您无需了解您的发行版如何处理 Apache 模块配置。

在清除并重写配置文件后，该模块会确保 apache2 服务正在运行（在企业级 Linux（EL）系统上是 `httpd`）。

然后我们使用 curl 测试了 Web 服务器。返回的只有一个空的索引页面。这是预期的行为。通常，当我们在服务器上安装 Apache 时，有一些文件会显示默认页面（在 EL 系统上是 `welcome.conf`），由于模块清除了这些配置，我们只看到了一个空页面。

在生产环境中，您将修改 Apache 模块应用的默认设置；README 中建议的配置如下：

```
class { 'apache':
  default_mods        => false,
  default_confd_files => false,
}
```

# 创建 Apache 虚拟主机

Apache 虚拟主机是通过 `apache` 模块和定义的类型 `apache::vhost` 创建的。我们将在 Apache Web 服务器上创建一个新的虚拟主机，命名为 **navajo**，这是 Apache 部落之一。

## 如何操作...

按照以下步骤创建 Apache 虚拟主机：

1.  创建一个新的 navajo `apache::vhost` 定义，如下所示：

    ```
    apache::vhost { 'navajo.example.com':
        port          => '80',
        docroot => '/var/www/navajo',
      }
    ```

1.  为新的虚拟主机创建一个索引文件：

    ```
    file {'/var/www/navajo/index.html':
        content => "<html>\nnavajo.example.com\nhttp://en.wikipedia.org/wiki/Navajo_people\n</html>\n",
        mode    => '0644',
        require => Apache::Vhost['navajo.example.com']
      }
    ```

1.  运行 Puppet 来创建新的虚拟主机：

    ```
    [root@webserver ~]# puppet agent -t
    Info: Caching catalog for webserver.example.com
    Info: Applying configuration version '1414475598'
    Notice: /Stage[main]/Main/Node[webserver]/Apache::Vhost[navajo.example.com]/File[/var/www/navajo]/ensure: created
    Notice: /Stage[main]/Main/Node[webserver]/Apache::Vhost[navajo.example.com]/File[25-navajo.example.com.conf]/ensure: created
    Info: /Stage[main]/Main/Node[webserver]/Apache::Vhost[navajo.example.com]/File[25-navajo.example.com.conf]: Scheduling refresh of Service[httpd]
    Notice: /Stage[main]/Main/Node[webserver]/File[/var/www/navajo/index.html]/ensure: defined content as '{md5}5212fe215f4c0223fb86102a34319cc6'
    Notice: /Stage[main]/Apache::Service/Service[httpd]: Triggered 'refresh' from 1 events
    Notice: Finished catalog run in 2.73 seconds

    ```

1.  验证您是否可以访问新的虚拟主机：

    ```
    [root@webserver ~]# curl http://navajo.example.com
    <html>
    navajo.example.com
    http://en.wikipedia.org/wiki/Navajo_people
    </html>

    ```

## 它是如何工作的...

`apache::vhost`定义的类型为 Apache 创建了一个虚拟主机配置文件，文件名为`25-navajo.example.com.conf`。该文件通过模板创建；文件名开头的`25`是该虚拟主机的“优先级”级别。当 Apache 首次启动时，它会遍历其配置目录，并按照字母顺序执行文件。以数字开头的文件会先于以字母开头的文件被读取。通过这种方式，Apache 模块确保虚拟主机按照特定顺序读取，而这个顺序可以在定义虚拟主机时进行指定。该文件的内容如下：

```
# ************************************
# Vhost template in module puppetlabs-apache
# Managed by Puppet
# ************************************

<VirtualHost *:80>
  ServerName navajo.example.com

  ## Vhost docroot
  DocumentRoot "/var/www/navajo"

  ## Directories, there should at least be a declaration for /var/www/navajo

  <Directory "/var/www/navajo">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Order allow,deny
    Allow from all
  </Directory>

  ## Load additional static includes

  ## Logging
  ErrorLog "/var/log/httpd/navajo.example.com_error.log"
  ServerSignature Off
  CustomLog "/var/log/httpd/navajo.example.com_access.log" combined

</VirtualHost>
```

如你所见，默认文件已经创建了日志文件，设置了目录访问权限和选项，此外还指定了监听端口和`DocumentRoot`。

vhost 定义创建了`DocumentRoot`目录，该目录作为'root'传递给`apache::virtual`定义。目录在虚拟主机配置文件之前创建；文件创建之后，通知触发器会发送给 Apache 进程以重新启动。

我们的清单文件包含了一个需要`Apache::Vhost['navajo.example.com']`资源的文件；在目录和虚拟主机配置文件创建之后，我们的文件也随之创建。

当我们在新网站上运行 curl 时（如果你没有在 DNS 中创建主机名别名，你需要在本地`/etc/hosts`文件中为`navajo.example.com`创建一个，或者将主机指定为`curl -H 'Host: navajo.example.com' <navajo.example.com 的 ip 地址>`），我们会看到我们创建的索引文件内容：

```
file {'/var/www/navajo/index.html':
    content => "<html>\nnavajo.example.com\nhttp://en.wikipedia.org/wiki/Navajo_people\n</html>\n",
    mode    => '0644',
    require => Apache::Vhost['navajo.example.com']
  }	
[root@webserver ~]# curl http://navajo.example.com
<html>
navajo.example.com
http://en.wikipedia.org/wiki/Navajo_people
<\html>
```

## 还有更多...

定义的类型和模板都考虑了虚拟主机的多种配置场景。几乎不可能找到这个模块没有涵盖的设置。你应该查看`apache::virtual`的定义，以及可能的各种参数。

该模块还为你处理了几个设置。例如，如果我们将`navajo`虚拟主机的监听端口从`80`更改为`8080`，该模块将在`/etc/httpd/conf.d/ports.conf`中进行如下更改：

```
Listen 80
+Listen 8080
 NameVirtualHost *:80
+NameVirtualHost *:8080
```

在我们的虚拟主机文件中：

```
-<VirtualHost *:80>
+<VirtualHost *:8080>
```

这样我们就可以在端口`8080`上使用 curl，并看到相同的结果：

```
[root@webserver ~]# curl http://navajo.example.com:8080
<html>
navajo.example.com
http://en.wikipedia.org/wiki/Navajo_people
</html>

```

当我们尝试使用端口`80`时：

```
[root@webserver ~]# curl http://navajo.example.com 
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
 <title>Index of /</title>
 </head>
 <body>
<h1>Index of /</h1>
<table><tr><th><img src="img/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr><tr><th colspan="5"><hr></th></tr>
<tr><th colspan="5"><hr></th></tr>
</table>
</body>
</html>

```

如我们所见，虚拟主机不再监听端口`80`，我们收到的是之前示例中看到的默认空目录列表。

# 创建 nginx 虚拟主机

Nginx 是一个快速、轻量级的 Web 服务器，在许多场景中，比 Apache 更受欢迎，特别是在性能要求较高的环境中。Nginx 的配置方式与 Apache 略有不同；不过，像 Apache 一样，有一个 Forge 模块可以帮助我们配置 nginx。与 Apache 不同的是，推荐使用的这个模块并不是由 puppetlabs 提供的，而是由 James Fryman 提供的。这个模块使用了一些有趣的技巧来进行自我配置。该模块的早期版本使用了 R.I. Pienaar 的 `module_data` 包。这个包用于在模块内配置 hieradata，为 nginx 模块提供默认值。现在我不建议从这个模块开始使用，但它是一个很好的例子，展示了模块配置未来可能的发展方向。赋予模块修改 hieradata 的能力可能会变得非常有用。

## 如何操作...

在这个示例中，我们将使用一个 Forge 模块来配置 nginx。我们将下载该模块，并使用它来配置虚拟主机。

1.  从 Forge 下载 `jfryman-nginx` 模块：

    ```
    t@mylaptop ~ $ cd ~/puppet
    t@mylaptop ~/puppet $ puppet module install -i modules jfryman-nginx
    Notice: Preparing to install into /home/thomas/puppet/modules ...
    Notice: Downloading from https://forgeapi.puppetlabs.com ...
    Notice: Installing -- do not interrupt ...
    /home/thomas/puppet/modules
    └─┬ jfryman-nginx (v0.2.1)
     ├── puppetlabs-apt (v1.7.0)
     ├── puppetlabs-concat (v1.1.1)
     └── puppetlabs-stdlib (v4.3.2)

    ```

1.  将 webserver 的定义替换为 nginx 配置：

    ```
    node webserver {
      class {'nginx':}
      nginx::resource::vhost { 'mescalero.example.com':
          www_root => '/var/www/mescalero',
      }
      file {'/var/www/mescalero':
        ensure  => 'directory''directory',
        mode    => '0755',
        require => Nginx::Resource::Vhost['mescalero.example.com'],
      }
      file {'/var/www/mescalero/index.html':
        content => "<html>\nmescalero.example.com\nhttp://en.wikipedia.org/wiki/Mescalero\n</html>\n",
        mode    => 0644,
        require => File['/var/www/mescalero'],
      }
    }
    ```

1.  如果 Apache 仍在 Web 服务器上运行，请停止它：

    ```
    [root@webserver ~]# puppet resource service httpd ensure=false
    Notice: /Service[httpd]/ensure: ensure changed 'running' to 'stopped'
    service { 'httpd':
     ensure => 'stopped',
    }
    Run puppet agent on your webserver node:
    [root@webserver ~]# puppet agent -t
    Info: Caching catalog for webserver.example.com
    Info: Applying configuration version '1414561483'
    Notice: /Stage[main]/Main/Node[webserver]/Nginx::Resource::Vhost[mescalero.example.com]/Concat[/etc/nginx/sites-available/mescalero.example.com.conf]/File[/etc/nginx/sites-available/mescalero.example.com.conf]/ensure: defined content as '{md5}35bb59bfcd0cf5a549d152aaec284357'
    Info: /Stage[main]/Main/Node[webserver]/Nginx::Resource::Vhost[mescalero.example.com]/Concat[/etc/nginx/sites-available/mescalero.example.com.conf]/File[/etc/nginx/sites-available/mescalero.example.com.conf]: Scheduling refresh of Class[Nginx::Service]
    Info: Concat[/etc/nginx/sites-available/mescalero.example.com.conf]: Scheduling refresh of Class[Nginx::Service]
    Notice: /Stage[main]/Main/Node[webserver]/Nginx::Resource::Vhost[mescalero.example.com]/File[mescalero.example.com.conf symlink]/ensure: created
    Info: /Stage[main]/Main/Node[webserver]/Nginx::Resource::Vhost[mescalero.example.com]/File[mescalero.example.com.conf symlink]: Scheduling refresh of Service[nginx]
    Notice: /Stage[main]/Main/Node[webserver]/File[/var/www/mescalero]/ensure: created
    Notice: /Stage[main]/Main/Node[webserver]/File[/var/www/mescalero/index.html]/ensure: defined content as '{md5}2bd618c7dc3a3addc9e27c2f3cfde294'
    Notice: /Stage[main]/Nginx::Config/File[/etc/nginx/conf.d/proxy.conf]/ensure: defined content as '{md5}1919fd65635d49653273e14028888617'
    Info: Computing checksum on file /etc/nginx/conf.d/example_ssl.conf
    Info: /Stage[main]/Nginx::Config/File[/etc/nginx/conf.d/example_ssl.conf]: Filebucketed /etc/nginx/conf.d/example_ssl.conf to puppet with sum 84724f296c7056157d531d6b1215b507
    Notice: /Stage[main]/Nginx::Config/File[/etc/nginx/conf.d/example_ssl.conf]/ensure: removed
    Info: Computing checksum on file /etc/nginx/conf.d/default.conf
    Info: /Stage[main]/Nginx::Config/File[/etc/nginx/conf.d/default.conf]: Filebucketed /etc/nginx/conf.d/default.conf to puppet with sum 4dce452bf8dbb01f278ec0ea9ba6cf40
    Notice: /Stage[main]/Nginx::Config/File[/etc/nginx/conf.d/default.conf]/ensure: removed
    Info: Class[Nginx::Config]: Scheduling refresh of Class[Nginx::Service]
    Info: Class[Nginx::Service]: Scheduling refresh of Service[nginx]
    Notice: /Stage[main]/Nginx::Service/Service[nginx]: Triggered 'refresh' from 2 events
    Notice: Finished catalog run in 28.98 seconds

    ```

1.  验证是否能访问新的虚拟主机：

    ```
    [root@webserver ~]# curl mescalero.example.com
    <html>
    mescalero.example.com
    http://en.wikipedia.org/wiki/Mescalero
    </html>

    ```

## 它是如何工作的...

安装 `jfryman-nginx` 模块会导致 concat、stdlib 和 APT 模块被安装。我们在主节点上运行 Puppet，让这些模块创建的插件被添加到正在运行的主节点中。stdlib 和 concat 模块包含了需要安装的 facter 和 Puppet 插件，以确保 nginx 模块能够正常工作。

插件同步后，我们可以在我们的 Web 服务器上运行 puppet agent。作为预防措施，如果 Apache 之前已经启动，我们先停止 Apache（因为我们不能让 nginx 和 Apache 都监听端口 `80`）。运行 puppet agent 后，我们确认 nginx 正在运行，且虚拟主机已配置好。

## 还有更多内容...

这个 nginx 模块正在积极开发中。模块中采用了几种有趣的解决方案。之前的版本使用了 `ripienaar-module_data` 模块，该模块通过 hiera 插件允许模块为其各种属性包含默认值。尽管仍处于开发的早期阶段，但该系统已经可以使用，并且代表了 Forge 上最前沿的模块之一。

在接下来的章节中，我们将使用一个支持的模块来配置和管理 MySQL 安装。

# 管理 MySQL

MySQL 是一个非常广泛使用的数据库服务器，几乎可以确定你在某个时刻需要安装并配置一个 MySQL 服务器。`puppetlabs-mysql` 模块可以简化 MySQL 部署。

## 如何操作...

按照以下步骤创建示例：

1.  安装 `puppetlabs-mysql` 模块：

    ```
    t@mylaptop ~/puppet $ puppet module install -i modules puppetlabs-mysql
    Notice: Preparing to install into /home/thomas/puppet/modules ...
    Notice: Downloading from https://forgeapi.puppetlabs.com ...
    Notice: Installing -- do not interrupt ...
    /home/thomas/puppet/modules
    └─┬ puppetlabs-mysql (v2.3.1)
     └── puppetlabs-stdlib (v4.3.2)

    ```

1.  为您的 MySQL 服务器创建一个新的节点定义：

    ```
    node dbserver {
      class { '::mysql::server':
        root_password    => 'PacktPub',
        override_options => { 
          'mysqld' => { 'max_connections' => '1024' } 
        }
      }
    }
    ```

1.  运行 Puppet 安装数据库服务器并应用新的 root 密码：

    ```
    [root@dbserver ~]# puppet agent -t
    Info: Caching catalog for dbserver.example.com
    Info: Applying configuration version '1414566216'
    Notice: /Stage[main]/Mysql::Server::Install/Package[mysql-server]/ensure: created
    Notice: /Stage[main]/Mysql::Server::Service/Service[mysqld]/ensure: ensure changed 'stopped' to 'running'
    Info: /Stage[main]/Mysql::Server::Service/Service[mysqld]: Unscheduling refresh on Service[mysqld]
    Notice: /Stage[main]/Mysql::Server::Root_password/Mysql_user[root@localhost]/password_hash: defined 'password_hash' as '*6ABB0D4A7D1381BAEE4D078354557D495ACFC059'
    Notice: /Stage[main]/Mysql::Server::Root_password/File[/root/.my.cnf]/ensure: defined content as '{md5}87bc129b137c9d613e9f31c80ea5426c'
    Notice: Finished catalog run in 35.50 seconds

    ```

1.  验证是否能连接到数据库：

    ```
    [root@dbserver ~]# mysql
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 11
    Server version: 5.1.73 Source distribution

    Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql>

    ```

## 它是如何工作的...

MySQL 模块安装 MySQL 服务器并确保服务器正在运行。然后，它配置 MySQL 的 root 密码。这个模块还为你做了很多其他事情。它创建了一个`.my.cnf`文件，其中包含 root 用户密码。当我们运行`mysql`客户端时，`.my.cnf`文件会设置所有默认值，因此我们无需提供任何参数。

## 还有更多...

在下一部分中，我们将展示如何创建数据库和用户。

# 创建数据库和用户

管理数据库意味着不仅要确保服务正在运行；没有数据库，数据库服务器什么都不是。数据库需要用户和权限。权限是通过`GRANT`语句来处理的。我们将使用`puppetlabs-mysql`包来创建一个数据库，并为该数据库创建一个用户。我们将创建一个名为 Drupal 的 MySQL 用户，以及一个名为 Drupal 的数据库。接着，我们将创建一个名为 nodes 的表，并将数据插入该表。

## 如何操作...

按照以下步骤创建数据库和用户：

1.  在`dbserver`类中创建一个数据库定义：

    ```
    mysql::db { 'drupal':
        host    => 'localhost',
        user    => 'drupal',
        password    => 'Cookbook',
        sql     => '/root/drupal.sql',
        require => File['/root/drupal.sql']
      }

      file { '/root/drupal.sql':
        ensure => present,
        source => 'puppet:///modules/mysql/drupal.sql',
      }
    ```

1.  允许 Drupal 用户修改 nodes 表：

    ```
    mysql_grant { 'drupal@localhost/drupal.nodes':
        ensure     => 'present',
        options    => ['GRANT'],
        privileges => ['ALL'],
        table      => 'drupal.nodes'nodes',
        user       => 'drupal@localhost',
      }
    ```

1.  创建一个包含以下内容的`drupal.sql`文件：

    ```
    CREATE TABLE users (id INT PRIMARY KEY AUTO_INCREMENT, title VARCHAR(255), body TEXT);
    INSERT INTO users (id, title, body) VALUES (1,'First Node','Contents of first Node');
    INSERT INTO users (id, title, body) VALUES (2,'Second Node','Contents of second Node');
    ```

1.  运行 Puppet 以创建用户、数据库和`GRANT`：

    ```
    [root@dbserver ~]# puppet agent -t
    Info: Caching catalog for dbserver.example.com
    Info: Applying configuration version '1414648818'
    Notice: /Stage[main]/Main/Node[dbserver]/File[/root/drupal.sql]/ensure: defined content as '{md5}780f3946cfc0f373c6d4146394650f6b'
    Notice: /Stage[main]/Main/Node[dbserver]/Mysql_grant[drupal@localhost/drupal.nodes]/ensure: created
    Notice: /Stage[main]/Main/Node[dbserver]/Mysql::Db[drupal]/Mysql_user[drupal@localhost]/ensure: created
    Notice: /Stage[main]/Main/Node[dbserver]/Mysql::Db[drupal]/Mysql_database[drupal]/ensure: created
    Info: /Stage[main]/Main/Node[dbserver]/Mysql::Db[drupal]/Mysql_database[drupal]: Scheduling refresh of Exec[drupal-import]
    Notice: /Stage[main]/Main/Node[dbserver]/Mysql::Db[drupal]/Mysql_grant[drupal@localhost/drupal.*]/ensure: created
    Notice: /Stage[main]/Main/Node[dbserver]/Mysql::Db[drupal]/Exec[drupal-import]: Triggered 'refresh' from 1 events
    Notice: Finished catalog run in 10.06 seconds

    ```

1.  验证数据库和表是否已创建：

    ```
    [root@dbserver ~]# mysql drupal
    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A

    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 34
    Server version: 5.1.73 Source distribution

    Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql> show tables;
    +------------------+
    | Tables_in_drupal |
    +------------------+
    | users            |
    +------------------+
    1 row in set (0.00 sec)

    ```

1.  现在，验证我们的默认数据是否已加载到表中：

    ```
    mysql> select * from users;
    +----+-------------+-------------------------+
    | id | title       | body                    |
    +----+-------------+-------------------------+
    |  1 | First Node  | Contents of first Node  |
    |  2 | Second Node | Contents of second Node |
    +----+-------------+-------------------------+
    2 rows in set (0.00 sec)

    ```

## 它是如何工作的...

我们从新 Drupal 数据库的定义开始：

```
  mysql::db { 'drupal':
    host    => 'localhost',
    user    => 'drupal',
    password    => 'Cookbook',
    sql     => '/root/drupal.sql',
    require => File['/root/drupal.sql']
  }
```

我们指定将从 localhost 连接（也可以从另一台服务器连接到数据库），使用 drupal 用户。我们为该用户提供密码，并指定一个将在数据库创建后应用的 SQL 文件。我们要求这个文件已经存在，并接下来定义该文件：

```
file { '/root/drupal.sql':
    ensure => present,
    source => 'puppet:///modules/mysql/drupal.sql',
  }
```

然后，我们确保用户拥有适当的权限，通过`mysql_grant`语句：

```
  mysql_grant { 'drupal@localhost/drupal.nodes':
    ensure     => 'present',
    options    => ['GRANT'],
    privileges => ['ALL'],
    table      => 'drupal.nodes',
    user       => 'drupal@localhost',
  }
```

## 还有更多...

使用 puppetlabs-MySQL 和 puppetlabs-Apache 模块，我们可以创建一个完整的功能性 Web 服务器。puppetlabs-Apache 模块将安装 Apache，我们还可以包括 PHP 模块和 MySQL 模块。然后，我们可以使用 puppetlabs-MySQL 模块来安装 MySQL 服务器，并创建所需的 Drupal 数据库，再将数据填充到数据库中。

部署一个新的 Drupal 安装就像在节点上包含一个类一样简单。

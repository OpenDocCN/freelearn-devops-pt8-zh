# 第二章：Puppet 基础设施

|   | *“未来的计算机可能只有 1,000 个真空管，重量仅为 1.5 吨。”* |   |
| --- | --- | --- |
|   | --*《Popular Mechanics》，1949 年* |

在本章中，我们将讨论：

+   安装 Puppet

+   使用 Git 管理你的清单

+   创建分散式 Puppet 架构

+   编写 papply 脚本

+   从 cron 运行 Puppet

+   使用 bash 引导 Puppet

+   创建集中式 Puppet 基础设施

+   创建具有多个 DNS 名称的证书

+   从 passenger 运行 Puppet

+   设置环境

+   配置 PuppetDB

+   配置 Hiera

+   使用 Hiera 设置节点特定数据

+   使用 hiera-gpg 存储机密数据

+   使用 MessagePack 序列化

+   使用 Git 钩子进行自动语法检查

+   使用 Git 推送代码

+   使用 Git 管理环境

# 介绍

在本章中，我们将介绍如何以集中式和分散式方式部署 Puppet。在每种方法中，我们将看到最佳实践、我的个人经验和社区解决方案的结合。

我们将配置并使用 PuppetDB 和 Hiera。PuppetDB 与导出的资源一起使用，我们将在第五章，*用户与虚拟资源*中进行详细讲解。Hiera 用于将变量数据与 Puppet 代码分离。

最后，我将介绍 Git，并展示如何使用 Git 来组织我们的代码和基础设施。

由于 Linux 发行版（如 Ubuntu、Red Hat 和 CentOS）在软件包名称、配置文件路径以及许多其他细节上有所不同，我决定出于篇幅和清晰度考虑，本书选择一个发行版（*Debian 7*，代号 *Wheezy*）并坚持使用它。然而，Puppet 可以在大多数流行的操作系统上运行，因此你应该能够轻松地将这些示例适配到你自己喜爱的操作系统和发行版上。

在本书写作时，Puppet 3.7.2 是最新的稳定版本，本书中使用的就是这一版本的 Puppet。Puppet 命令的语法变化较为频繁，因此请注意，虽然较旧版本的 Puppet 仍然可以正常使用，但它们可能不支持本书中描述的所有功能和语法。正如我们在第一章，*Puppet 语言与风格*中所见，未来的解析器展示了计划在 Puppet 4 版本中成为默认功能的语言特性。

# 安装 Puppet

在第一章，*Puppet 语言与风格*中，我们通过 gem 安装安装了 Puppet 作为 rubygem。在部署到多个节点时，这可能不是最佳方法。使用你所选择的发行版的包管理器，是保持所有节点上 Puppet 版本一致的最佳方法。Puppet Labs 为基于 APT 和 YUM 的发行版维护了软件仓库。

## 准备工作

如果你的 Linux 发行版使用 APT 作为包管理，访问 [`apt.puppetlabs.com/`](http://apt.puppetlabs.com/) 下载适合你发行版的 Puppet Labs 发布包。对于我们的 wheezy cookbook 节点，我们将使用 [`apt.puppetlabs.com/puppetlabs-release-wheezy.deb`](http://apt.puppetlabs.com/puppetlabs-release-wheezy.deb)。

如果你使用的是一个使用 YUM 作为包管理的 Linux 发行版，访问 [`yum.puppetlabs.com/`](http://yum.puppetlabs.com/) 下载适合你发行版的 Puppet Labs 发布包。

## 如何操作...

1.  一旦找到适合你发行版的 Puppet Labs 发布包，安装 Puppet 的步骤对于 APT 和 YUM 都是相同的：

    +   安装 Puppet Labs 发布包

    +   安装 Puppet 包

1.  一旦安装了 Puppet，请按照下面的示例验证 Puppet 的版本：

    ```
    t@ckbk:~ puppet --version 3.7.2

    ```

现在我们已经有了在节点上安装 Puppet 的方法，接下来需要关注的是如何保持 Puppet 清单的有序管理。在接下来的章节中，我们将展示如何使用 Git 来保持代码的组织性和一致性。

# 使用 Git 管理你的清单

将 Puppet 清单放入版本控制系统（如 Git 或 Subversion）是一个非常好的主意（Git 是 Puppet 的事实标准）。这样做有几个优势：

+   你可以撤销更改，并恢复到任何先前的清单版本

+   你可以使用分支来尝试新特性

+   如果多人需要修改清单，他们可以在各自的工作副本中独立进行更改，然后再合并更改。

+   你可以使用 `git log` 功能查看更改的内容，以及更改发生的时间（和更改者）

## 准备工作

在这一部分中，我们将把你现有的清单文件导入到 Git 中。如果你在之前的部分中创建了 Puppet 目录，请使用该目录，否则使用你现有的清单目录。

在本示例中，我们将在一个所有节点都能访问的服务器上创建一个新的 Git 仓库。为了将代码托管在 Git 仓库中，我们需要执行几个步骤：

1.  在中央服务器上安装 Git。

1.  创建一个用户来运行 Git 并拥有仓库。

1.  创建一个仓库来存放代码。

1.  创建 **SSH** 密钥以允许基于密钥的访问仓库。

1.  在节点上安装 Git 并从我们的 Git 仓库下载最新版本。

## 如何操作...

按照以下步骤操作：

1.  首先，在你的 Git 服务器（在我们的示例中是 `git.example.com`）上安装 Git。最简单的方式是使用 Puppet。创建以下清单，命名为 `git.pp`：

    ```
      package {'git':
        ensure => installed }
    ```

1.  使用 `puppet apply git.pp` 应用此清单，这将安装 Git。

1.  接下来，创建一个 Git 用户，供节点登录并获取最新代码。我们仍然使用 Puppet 来完成这个操作。我们还将创建一个目录来存放我们的仓库（`/home/git/repos`），如下面的代码片段所示：

    ```
    group { 'git': gid => 1111, }
    user {'git': uid => 1111, gid => 1111, comment => 'Git User', home => '/home/git', require => Group['git'], }
    file {'/home/git': ensure => 'directory', owner => 1111, group => 1111, require => User['git'], }
    file {'/home/git/repos': ensure => 'directory', owner => 1111, group => 1111, require => File['/home/git'] }
    ```

1.  应用该清单后，作为 Git 用户登录，并使用以下命令创建一个空的 Git 仓库：

    ```
    # sudo -iu git git@git $ cd repos git@git $ git init --bare puppet.git Initialized empty Git repository in /home/git/repos/puppet.git/

    ```

1.  为 Git 用户设置密码，我们将在下一步后远程登录：

    ```
    [root@git ~]# passwd git
    Changing password for user git.
    New password: 
    Retype new password: 
    passwd: all authentication tokens updated successfully.

    ```

1.  现在回到本地机器，创建一个`ssh`密钥供我们的节点用来更新仓库：

    ```
    t@mylaptop ~ $ cd .ssh
    t@mylaptop ~/.ssh $ ssh-keygen -b 4096 -f git_rsa
    Generating public/private rsa key pair.
    Enter passphrase (empty for no passphrase): 
    Enter same passphrase again: 
    Your identification has been saved in git_rsa.
    Your public key has been saved in git_rsa.pub.
    The key fingerprint is:
    87:35:0e:4e:d2:96:5f:e4:ce:64:4a:d5:76:c8:2b:e4 thomas@mylaptop
    ```

1.  现在，将新创建的公钥复制到`authorized_keys`文件中。这将允许我们使用这个新密钥连接到 Git 服务器：

    ```
    t@mylaptop ~/.ssh $ ssh-copy-id -i git_rsa git@git.example.com
    git@git.example.com's password: 
    Number of key(s) added: 1
    ```

1.  现在尝试使用以下命令登录到机器："ssh 'git@git.example.com'"，并检查确保只有你希望添加的密钥被加入。

1.  接下来，配置`ssh`在访问 Git 服务器时使用你的密钥，并将以下内容添加到你的`~/.ssh/config`文件中：

    ```
    Host git git.example.com
     User git
     IdentityFile /home/thomas/.ssh/git_rsa

    ```

1.  将仓库克隆到你的机器上，放入一个名为 Puppet 的目录中（如果你没有使用`git.example.com`，请替换为你的服务器名称）：

    ```
    t@mylaptop ~$ git clone git@git.example.com:repos/puppet.git
    Cloning into 'puppet'...
    warning: You appear to have cloned an empty repository.
    Checking connectivity... done.
    ```

    我们已经创建了一个 Git 仓库；在我们向仓库提交任何更改之前，最好在 Git 中设置你的名字和电子邮件。你的名字和电子邮件将附加到你做的每个提交上。

1.  当你在一个大型团队中工作时，知道是谁做了哪些更改非常重要；为此，请使用以下代码片段：

    ```
    t@mylaptop puppet$ git config --global user.email"thomas@narrabilis.com"
    t@mylaptop puppet$ git config --global user.name "ThomasUphill"

    ```

1.  你可以使用以下代码片段验证你的 Git 设置：

    ```
    t@mylaptop ~$ git config --global --list
    user.name=Thomas Uphill
    user.email=thomas@narrabilis.com
    core.editor=vim
    merge.tool=vimdiff
    color.ui=true
    push.default=simple

    ```

1.  现在我们已经正确配置了 Git，切换到你的仓库目录，并按照以下代码片段创建一个新的站点清单：

    ```
    t@mylaptop ~$ cd puppet
    t@mylaptop puppet$ mkdir manifests
    t@mylaptop puppet$ vim manifests/site.pp
    node default {
     include base
    }

    ```

1.  这个站点清单将在每个节点上安装我们的基础类；我们将像在第一章中一样使用 Puppet 模块创建基础类，*Puppet 语言与风格*：

    ```
    t@mylaptop puppet$ mkdir modules
    t@mylaptop puppet$ cd modules
    t@mylaptop modules$ puppet module generate thomas-base
    Notice: Generating module at /home/tuphill/puppet/modules/thomas-base
    thomas-base
    thomas-base/Modulefile
    thomas-base/README
    thomas-base/manifests
    thomas-base/manifests/init.pp
    thomas-base/spec
    thomas-base/spec/spec_helper.rb
    thomas-base/tests
    thomas-base/tests/init.pp
    t@mylaptop modules$ ln -s thomas-base base

    ```

1.  最后一步，我们在`thomas-base`目录和`base`之间创建一个符号链接。现在，为了确保我们的模块能够执行有用的操作，请在`thomas-base/manifests/init.pp`中定义的`base`类的主体中添加以下内容：

    ```
    class base {
     file {'/etc/motd':
     content => "${::fqdn}\nManaged by puppet ${::puppetversion}\n"
     }
    }

    ```

1.  现在使用`git add`和`git commit`将新的基础模块和站点清单添加到 Git，如下所示：

    ```
    t@mylaptop modules$ cd ..
    t@mylaptop puppet$ git add modules manifests
    t@mylaptop puppet$ git status
    On branch master
    Initial commit
    Changes to be committed:
     (use "git rm --cached <file>..." to unstage)
    new file:   manifests/site.pp
    new file:   modules/base
    new file:   modules/thomas-base/Modulefile
    new file:   modules/thomas-base/README
    new file:   modules/thomas-base/manifests/init.pp
    new file:   modules/thomas-base/spec/spec_helper.rb
    new file:   modules/thomas-base/tests/init.pp
    t@mylaptop puppet$ git commit -m "Initial commit with simple base module"
    [master (root-commit) 3e1f837] Initial commit with simple base module
     7 files changed, 102 insertions(+)
     create mode 100644 manifests/site.pp
     create mode 120000 modules/base
     create mode 100644 modules/thomas-base/Modulefile
     create mode 100644 modules/thomas-base/README
     create mode 100644 modules/thomas-base/manifests/init.pp
     create mode 100644 modules/thomas-base/spec/spec_helper.rb
     create mode 100644 modules/thomas-base/tests/init.pp

    ```

1.  目前，你对 Git 仓库的更改已经在本地提交；你现在需要将这些更改推送到`git.example.com`，以便其他节点可以获取更新的文件：

    ```
    t@mylaptop puppet$ git push origin master
    Counting objects: 15, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (9/9), done.
    Writing objects: 100% (15/15), 2.15 KiB | 0 bytes/s, done.
    Total 15 (delta 0), reused 0 (delta 0)
    To git@git.example.com:repos/puppet.git
     * [new branch]      master -> master

    ```

## 它是如何工作的...

Git 跟踪文件更改，并存储所有更改的完整历史记录。仓库的历史由提交（commits）构成。一个提交代表某个特定时间点仓库的状态，你可以通过`git commit`命令创建并附加一条信息来标注提交。

你现在已经将 Puppet 清单文件添加到仓库并创建了第一个提交。这更新了仓库的历史记录，但仅在你本地的工作副本中。要将更改与`git.example.com`副本同步，`git push`命令会推送自上次同步以来所做的所有更改。

## 还有更多内容...

既然你已经有了一个中央的 Git 仓库来存储 Puppet 清单，你就可以在不同的位置检出多个副本，并在提交更改之前进行工作。例如，如果你在一个团队中工作，每个成员可以有自己本地的仓库副本，并通过中央服务器与其他成员同步更改。你也可以选择使用 GitHub 作为你的中央 Git 仓库服务器。GitHub 为公开仓库提供免费的 Git 仓库托管，如果你不希望 Puppet 代码公开，可以支付 GitHub 的高级服务费用。

在接下来的章节中，我们将使用我们的 Git 仓库来配置集中式和去中心化的 Puppet 配置。

# 创建一个去中心化的 Puppet 架构

Puppet 是一个配置管理工具。你可以使用 Puppet 配置并防止大量客户端计算机出现配置漂移。如果你的所有客户端计算机都可以通过一个中心位置轻松访问，你可以选择让中央 Puppet 服务器控制所有客户端计算机。在集中式模型中，Puppet 服务器被称为 Puppet 主控。我们将在接下来的几节中介绍如何配置一个中央 Puppet 主控。

如果你的客户端计算机分布广泛，或者无法保证客户端计算机与中心位置之间的通信，那么去中心化架构可能适合你的部署。在接下来的几节中，我们将看到如何配置一个去中心化的 Puppet 架构。

正如我们所见，我们可以直接在清单文件上运行 `puppet apply` 命令来让 Puppet 应用它。这个方法的问题是我们需要将清单传输到客户端计算机上。

我们可以使用上一节中创建的 Git 仓库，将我们的清单传输到我们创建的每个新节点。

## 准备工作

创建一个新的测试节点，给这个新节点命名，你可以随意命名，我这里用 `testnode`。按照之前的方法在机器上安装 Puppet。

## 如何操作...

创建一个 `bootstrap.pp` 清单，该清单将在我们的新节点上执行以下配置步骤：

1.  安装 Git：

    ```
    package {'git':
      ensure => 'installed'
    }
    ```

1.  在 Puppet 用户的主目录（`/var/lib/puppet/.ssh/id_rsa`）中安装 `ssh` 密钥，以访问 `git.example.com`：

    ```
    File {
      owner => 'puppet',
      group => 'puppet',
    }
    file {'/var/lib/puppet/.ssh':
      ensure => 'directory',
    }
    file {'/var/lib/puppet/.ssh/id_rsa':
      content => "
    -----BEGIN RSA PRIVATE KEY-----
    …
    NIjTXmZUlOKefh4MBilqUU3KQG8GBHjzYl2TkFVGLNYGNA0U8VG8SUJq
    -----END RSA PRIVATE KEY-----
    ",
      mode    => 0600,
      require => File['/var/lib/puppet/.ssh']
    }
    ```

1.  从 `git.example.com` 下载 `ssh` 主机密钥（`/var/lib/puppet/.ssh/known_hosts`）：

    ```
    exec {'download git.example.com host key': 
      command => 'sudo -u puppet ssh-keyscan git.example.com >> /var/lib/puppet/.ssh/known_hosts',
      path    => '/usr/bin:/usr/sbin:/bin:/sbin',
      unless  => 'grep git.example.com /var/lib/puppet/.ssh/known_hosts',
      require => File['/var/lib/puppet/.ssh'],
    }
    ```

1.  创建一个目录来存放 Git 仓库（`/etc/puppet/cookbook`）：

    ```
    file {'/etc/puppet/cookbook':
      ensure => 'directory',
    }
    ```

1.  将 Puppet 仓库克隆到新机器上：

    ```
    exec {'create cookbook':
      command => 'sudo -u puppet git clone git@git.example.com:repos/puppet.git /etc/puppet/cookbook',
      path    => '/usr/bin:/usr/sbin:/bin:/sbin',
      require => [Package['git'],File['/var/lib/puppet/.ssh/id_rsa'],Exec['download git.example.com host key']],
      unless  => 'test -f /etc/puppet/cookbook/.git/config',
    }
    ```

1.  现在，当我们在新机器上运行 Puppet apply 时，`ssh` 密钥将为 Puppet 用户安装。然后，Puppet 用户会将 Git 仓库克隆到 `/etc/puppet/cookbook`：

    ```
    root@testnode /tmp# puppet apply bootstrap.pp 
    Notice: Compiled catalog for testnode.example.com in environment production in 0.40 seconds
    Notice: /Stage[main]/Main/File[/etc/puppet/cookbook]/ensure: created
    Notice: /Stage[main]/Main/File[/var/lib/puppet/.ssh]/ensure: created
    Notice: /Stage[main]/Main/Exec[download git.example.com host key]/returns: executed successfully
    Notice: /Stage[main]/Main/File[/var/lib/puppet/.ssh/id_rsa]/ensure: defined content as '{md5}da61ce6ccc79bc6937bd98c798bc9fd3'
    Notice: /Stage[main]/Main/Exec[create cookbook]/returns: executed successfully
    Notice: Finished catalog run in 0.82 seconds

    ```

    ### 注意

    你可能需要禁用 `sudo` 的 `tty` 要求。如果 `/etc/sudoers` 文件中有 `Defaults requiretty` 这一行，请将其注释掉。

    或者，你可以在 `'create cookbook' exec` 类型中设置 `user => Puppet`。请注意，使用 user 属性会导致命令的任何错误信息丢失。

1.  现在，你的 Puppet 代码已经可以在新节点上使用，你可以通过 `puppet apply` 来应用它，并指定 `/etc/puppet/cookbook/modules` 包含这些模块：

    ```
    root@testnode ~# puppet apply --modulepath=/etc/puppet/cookbook/modules /etc/puppet/cookbook/manifests/site.pp 
    Notice: Compiled catalog for testnode.example.com in environment production in 0.12 seconds
    Notice: /Stage[main]/Base/File[/etc/motd]/content: content changed '{md5}86d28ff83a8d49d349ba56b5c64b79ee' to '{md5}4c4c3ab7591d940318279d78b9c51d4f'
    Notice: Finished catalog run in 0.11 seconds
    root@testnode /tmp# cat /etc/motd
    testnode.example.com
    Managed by puppet 3.6.2

    ```

## 它是如何工作的...

首先，我们的 `bootstrap.pp` 清单确保 Git 已安装。该清单接着确保 Git 用户在 `git.example.com` 上的 `ssh` 密钥已安装到 Puppet 用户的主目录（默认是 `/var/lib/puppet`）。接下来，清单确保 `git.example.com` 的主机密钥被 Puppet 用户信任。配置好 `ssh` 后，启动脚本确保 `/etc/puppet/cookbook` 存在并且是一个目录。

然后我们使用 `exec` 命令让 Git 克隆仓库到 `/etc/puppet/cookbook`。所有代码就绪后，我们再次调用 `puppet apply` 来部署仓库中的代码。在生产环境中，你可以将 `bootstrap.pp` 清单分发到所有节点，可能通过内部 web 服务器，使用类似 curl [`puppet/bootstrap.pp >bootstrap.pp && puppet apply bootstrap.pp`](http://puppet/bootstrap.pp >bootstrap.pp && puppet apply bootstrap.pp) 的方式。

# 编写一个 papply 脚本

我们希望尽可能快速和简单地在机器上应用 Puppet；为此，我们将编写一个小脚本，将 `puppet apply` 命令与它所需的参数封装在一起。我们将使用 Puppet 本身在需要的地方部署这个脚本。

## 如何操作...

按照以下步骤操作：

1.  在你的 Puppet 仓库中，创建 Puppet 模块所需的目录：

    ```
    t@mylaptop ~$ cd puppet/modules
    t@mylaptop modules$ mkdir -p puppet/{manifests,files}

    ```

1.  创建 `modules/puppet/files/papply.sh` 文件，内容如下：

    ```
    #!/bin/sh sudo puppet apply /etc/puppet/cookbook/manifests/site.pp \--modulepath=/etc/puppet/cookbook/modules $*
    ```

1.  创建 `modules/puppet/manifests/init.pp` 文件，内容如下：

    ```
    class puppet {
      file { '/usr/local/bin/papply':
        source => 'puppet:///modules/puppet/papply.sh',
        mode   => '0755',
      }
    }
    ```

1.  按如下方式修改你的 `manifests/site.pp` 文件：

    ```
    node default {
      include base
      include puppet
    }
    ```

1.  将 Puppet 模块添加到 Git 仓库并提交更改，如下所示：

    ```
    t@mylaptop puppet$ git add manifests/site.pp modules/puppet
    t@mylaptop puppet$ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes to be committed:
     (use "git reset HEAD <file>..." to unstage)
    modified:   manifests/site.pp
    new file:   modules/puppet/files/papply.sh
    new file:   modules/puppet/manifests/init.pp
    t@mylaptop puppet$ git commit -m "adding puppet module to include papply"
    [master 7c2e3d5] adding puppet module to include papply
     3 files changed, 11 insertions(+)
     create mode 100644 modules/puppet/files/papply.sh
     create mode 100644 modules/puppet/manifests/init.pp

    ```

1.  现在记得将更改推送到 `git.example.com` 上的 Git 仓库：

    ```
    t@mylaptop puppet$ git push origin master Counting objects: 14, done. Delta compression using up to 4 threads. Compressing objects: 100% (7/7), done. Writing objects: 100% (10/10), 894 bytes | 0 bytes/s, done. Total 10 (delta 0), reused 0 (delta 0) To git@git.example.com:repos/puppet.git 23e887c..7c2e3d5  master -> master

    ```

1.  如下所示，拉取 Git 仓库的最新版本到你的新节点（对我来说是 `testnode`）：

    ```
    root@testnode ~# sudo -iu puppet
    puppet@testnode ~$ cd /etc/puppet/cookbook/puppet@testnode /etc/puppet/cookbook$ git pull origin master remote: Counting objects: 14, done. remote: Compressing objects: 100% (7/7), done. remote: Total 10 (delta 0), reused 0 (delta 0) Unpacking objects: 100% (10/10), done. From git.example.com:repos/puppet * branch            master     -> FETCH_HEAD Updating 23e887c..7c2e3d5 Fast-forward manifests/site.pp                |    1 + modules/puppet/files/papply.sh   |    4 ++++ modules/puppet/manifests/init.pp |    6 ++++++ 3 files changed, 11 insertions(+), 0 deletions(-) create mode 100644 modules/puppet/files/papply.sh create mode 100644 modules/puppet/manifests/init.pp

    ```

1.  手动应用清单一次以安装 `papply` 脚本：

    ```
    root@testnode ~# puppet apply /etc/puppet/cookbook/manifests/site.pp --modulepath /etc/puppet/cookbook/modules
    Notice: Compiled catalog for testnode.example.com in environment production in 0.13 seconds
    Notice: /Stage[main]/Puppet/File[/usr/local/bin/papply]/ensure: defined content as '{md5}d5c2cdd359306dd6e6441e6fb96e5ef7'
    Notice: Finished catalog run in 0.13 seconds

    ```

1.  最后，测试脚本：

    ```
    root@testnode ~# papply
    Notice: Compiled catalog for testnode.example.com in environment production in 0.13 seconds
    Notice: Finished catalog run in 0.09 seconds

    ```

现在，每当你需要运行 Puppet 时，只需运行 `papply`。将来，当我们应用 Puppet 更改时，我会让你运行 `papply` 而不是完整的 `puppet apply` 命令。

## 它是如何工作的...

正如你所见，为了在机器上运行 Puppet 并应用指定的清单文件，我们使用 `puppet apply` 命令：

```
puppet apply manifests/site.pp

```

当你使用模块时（例如我们刚才创建的 Puppet 模块），你还需要告诉 Puppet 在哪里查找模块，使用 `modulepath` 参数：

```
puppet apply manifests/nodes.pp \--modulepath=/home/ubuntu/puppet/modules
```

为了以所需的根权限运行 Puppet，我们必须在所有命令前加上 `sudo`：

```
sudo puppet apply manifests/nodes.pp \--modulepath=/home/ubuntu/puppet/modules
```

最后，传递给 `papply` 的任何附加参数将通过 `$*` 参数传递给 Puppet 本身：

```
sudo puppet apply manifests/nodes.pp \--modulepath=/home/ubuntu/puppet/modules $*
```

这需要大量的输入，因此将其放入脚本中是合理的。我们添加了一个 Puppet 文件资源，将脚本部署到 `/usr/local/bin` 并使其可执行：

```
file { '/usr/local/bin/papply': source => 'puppet:///modules/puppet/papply.sh', mode   => '0755',}
```

最后，我们在默认节点声明中包含了 Puppet 模块：

```
node default {
  include base
  include puppet
}
```

你可以对任何其他由 Puppet 管理的节点做同样的操作。

# 从 cron 运行 Puppet

你可以利用现有的设置做很多事情：作为团队共同处理 Puppet 清单，通过中央 Git 仓库通信更改，并使用 `papply` 脚本手动在机器上应用它们。

然而，你仍然需要登录到每台机器上以更新 Git 仓库并重新运行 Puppet。拥有每台机器自动更新并应用更改的功能会很有帮助。然后你只需推送更改到仓库，它会在一定时间内自动传送到所有机器。

最简单的方法是使用 **cron** 任务定期从仓库拉取更新，并在有任何更改时运行 Puppet。

## 准备工作

你将需要我们在*使用 Git 管理清单*和*创建去中心化的 Puppet 架构*食谱中设置的 Git 仓库，以及来自*编写 papply 脚本*食谱的 `papply` 脚本。你需要应用我们创建的 `bootstrap.pp` 清单，以安装 `ssh` 密钥并下载最新的仓库。

## 如何操作...

按照以下步骤操作：

1.  将 `bootstrap.pp` 脚本复制到任何你希望注册的节点。`bootstrap.pp` 清单包括用于访问 Git 仓库的私钥，生产环境中应该保护该密钥。

1.  创建 `modules/puppet/files/pull-updates.sh` 文件，内容如下：

    ```
    #!/bin/sh
    cd /etc/puppet/cookbook
    sudo –u puppet git pull && /usr/local/bin/papply
    ```

1.  修改 `modules/puppet/manifests/init.pp` 文件，在 `papply` 文件定义后添加以下代码片段：

    ```
    file { '/usr/local/bin/pull-updates':
      source => 'puppet:///modules/puppet/pull-updates.sh',
      mode   => '0755',
    }
    cron { 'run-puppet':
      ensure  => 'present',
      user    => 'puppet',
      command => '/usr/local/bin/pull-updates',
      minute  => '*/10',
      hour    => '*',
    }
    ```

1.  如之前所示，提交更改并推送到 Git 服务器，命令行如下：

    ```
    t@mylaptop puppet$ git add modules/puppet
    t@mylaptop puppet$ git commit -m "adding pull-updates"
    [master 7e9bac3] adding pull-updates
     2 files changed, 14 insertions(+)
     create mode 100644 modules/puppet/files/pull-updates.sh
    t@mylaptop puppet$ git push
    Counting objects: 14, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (7/7), done.
    Writing objects: 100% (8/8), 839 bytes | 0 bytes/s, done.
    Total 8 (delta 0), reused 0 (delta 0)
    To git@git.example.com:repos/puppet.git
       7c2e3d5..7e9bac3  master -> master

    ```

1.  在测试节点上发出 Git pull 命令：

    ```
    root@testnode ~# cd /etc/puppet/cookbook/
    root@testnode /etc/puppet/cookbook# sudo –u puppet git pull
    remote: Counting objects: 14, done.
    remote: Compressing objects: 100% (7/7), done.
    remote: Total 8 (delta 0), reused 0 (delta 0)
    Unpacking objects: 100% (8/8), done.
    From git.example.com:repos/puppet
     23e887c..7e9bac3  master     -> origin/master
    Updating 7c2e3d5..7e9bac3
    Fast-forward
     modules/puppet/files/pull-updates.sh |    3 +++
     modules/puppet/manifests/init.pp     |   11 +++++++++++
     2 files changed, 14 insertions(+), 0 deletions(-)
     create mode 100644 modules/puppet/files/pull-updates.sh

    ```

1.  在测试节点上运行 Puppet：

    ```
    root@testnode /etc/puppet/cookbook# papply
    Notice: Compiled catalog for testnode.example.com in environment production in 0.17 seconds
    Notice: /Stage[main]/Puppet/Cron[run-puppet]/ensure: created
    Notice: /Stage[main]/Puppet/File[/usr/local/bin/pull-updates]/ensure: defined content as '{md5}04c023feb5d566a417b519ea51586398'
    Notice: Finished catalog run in 0.16 seconds

    ```

1.  检查 `pull-updates` 脚本是否正常工作：

    ```
    root@testnode /etc/puppet/cookbook# pull-updates
    Already up-to-date.
    Notice: Compiled catalog for testnode.example.com in environment production in 0.15 seconds
    Notice: Finished catalog run in 0.14 seconds

    ```

1.  验证 `cron` 任务是否成功创建：

    ```
    root@testnode /etc/puppet/cookbook# crontab -l -u puppet
    # HEADER: This file was autogenerated at Tue Sep 09 02:31:00 -0400 2014 by puppet.
    # HEADER: While it can still be managed manually, it is definitely not recommended.
    # HEADER: Note particularly that the comments starting with 'Puppet Name' should
    # HEADER: not be deleted, as doing so could cause duplicate cron jobs.
    # Puppet Name: run-puppet
    */10 * * * * /usr/local/bin/pull-updates

    ```

## 它是如何工作的...

当我们创建 `bootstrap.pp` 清单时，我们确保 Puppet 用户可以使用 `ssh` 密钥签出 Git 仓库。这使得 Puppet 用户可以在食谱目录中无人值守地运行 Git pull。我们还添加了 `pull-updates` 脚本，它会执行此操作，并在拉取到任何更改时运行 Puppet：

```
#!/bin/sh
cd /etc/puppet/cookbook
sudo –u puppet git pull && papply
```

我们通过 Puppet 将此脚本部署到节点上：

```
file { '/usr/local/bin/pull-updates':
  source => 'puppet:///modules/puppet/pull-updates.sh',
  mode   => '0755',
}
```

最后，我们创建了一个 `cron` 任务，它定期（每 10 分钟，但如果需要可以更改）运行 `pull-updates`：

```
cron { 'run-puppet':
  ensure  => 'present',
  command => '/usr/local/bin/pull-updates',
  minute  => '*/10',
  hour    => '*',
}
```

## 还有更多...

恭喜，你现在拥有一个完全自动化的 Puppet 基础架构！一旦你应用了 `bootstrap.pp` 清单，运行 Puppet 在仓库中；机器将设置为拉取任何新的更改并自动应用它们。

比如，如果你想为所有机器添加一个新的用户账户，你需要做的就是在工作副本的清单中添加该账户，然后提交并推送到中央 Git 仓库。在 10 分钟内，它会自动应用到所有运行 Puppet 的机器上。

# 使用 bash 启动 Puppet

本书的早期版本使用 Rakefiles 来启动 Puppet。使用 Rake 配置节点的问题在于，您是从笔记本电脑上运行命令；您假设已经有 `ssh` 访问该机器的权限。大多数启动过程通过在节点已被配置后执行一个容易记住的命令来完成。在本节中，我们将展示如何使用 bash 和 web 服务器以及启动脚本来启动 Puppet。

## 准备工作

在一个中心可访问的服务器上安装 httpd，并创建一个受密码保护的区域来存储启动脚本。在我的示例中，我将使用之前设置的 Git 服务器 `git.example.com`。首先在 web 服务器的根目录下创建一个目录：

```
# cd /var/www/html
# mkdir bootstrap

```

现在执行以下步骤：

1.  将以下位置定义添加到您的 apache 配置中：

    ```
    <Location /bootstrap>
    AuthType basic
    AuthName "Bootstrap"
    AuthBasicProvider file
    AuthUserFile /var/www/puppet.passwd
    Require valid-user
    </Location>
    ```

1.  重新加载您的 web 服务器以确保位置配置生效。使用 curl 验证您无法在没有身份验证的情况下从 bootstrap 目录下载：

    ```
    [root@bootstrap-test tmp]# curl http://git.example.com/bootstrap/
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>401 Authorization Required</title>
    </head><body>
    <h1>Authorization Required</h1>

    ```

1.  创建您在 apache 配置中引用的密码文件（`/var/www/puppet.passwd`）：

    ```
    root@git# cd /var/www
    root@git# htpasswd –cb puppet.passwd bootstrap cookbook
    Adding password for user bootstrap 

    ```

1.  验证用户名和密码是否允许访问 bootstrap 目录，如下所示：

    ```
    [root@node1 tmp]# curl --user bootstrap:cookbook http://git.example.com/bootstrap/
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
    <html>
     <head>
     <title>Index of /bootstrap</title>

    ```

## 如何操作...

现在，您有了一个安全的位置来存储启动脚本，在 bootstrap 目录中为您支持的每个操作系统创建一个启动脚本。在这个例子中，我将展示如何为基于 Red Hat Enterprise Linux 6 的发行版创建启动脚本。

### 提示

尽管 bootstrap 位置需要密码保护，但由于我们没有在服务器上配置 SSL，因此没有加密。没有加密的话，这个位置并不太安全。

在 bootstrap 目录中创建一个名为`el6.sh`的脚本，内容如下：

```
#!/bin/bash

# bootstrap for EL6 distributions
SERVER=git.example.com
LOCATION=/bootstrap
BOOTSTRAP=bootstrap.pp
USER=bootstrap
PASS=cookbook

# install puppet
curl http://yum.puppetlabs.com/RPM-GPG-KEY-puppetlabs >/etc/pki/rpm-gpg/RPM-GPG-KEY-puppetlabs
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-puppetlabs
yum -y install http://yum.puppetlabs.com/puppetlabs-release-el-6.noarch.rpm
yum -y install puppet
# download bootstrap
curl --user $USER:$PASS http://$SERVER/$LOCATION/$BOOTSTRAP >/tmp/$BOOTSTRAP
# apply bootstrap
cd /tmp
puppet apply /tmp/$BOOTSTRAP
# apply puppet
puppet apply --modulepath /etc/puppet/cookbook/modules /etc/puppet/cookbook/manifests/site.pp
```

## 工作原理...

Apache 配置仅允许使用用户名和密码组合访问 bootstrap 目录。我们通过将`--user`参数传递给 curl 来提供这些信息，从而获得对文件的访问权限。我们使用管道符号（`|`）将 curl 的输出重定向到 bash，这样 bash 就会执行该脚本。我们像编写任何其他 bash 脚本一样编写我们的 bash 脚本。该脚本下载我们的`bootstrap.pp`清单并应用它。最后，我们从 Git 仓库应用 Puppet 清单，并将机器配置为我们去中心化基础设施的一部分。

## 更多内容...

要支持另一个操作系统，我们只需要创建一个新的 bash 脚本。所有的 Linux 发行版都支持 bash 脚本，Mac OS X 也支持。由于我们将大部分逻辑放入了`bootstrap.pp`清单中，因此启动脚本非常简洁，且易于移植到新的操作系统。

# 创建一个集中式 Puppet 基础设施

像 Puppet 这样的配置管理工具在管理大量机器时最为有效。如果所有机器都能连接到一个中心位置，使用集中式的 Puppet 基础设施可能是一个好方案。不幸的是，Puppet 在节点数量较多时扩展性较差。如果你的部署有不到 800 台服务器，假设你的目录不复杂（每个目录编译时间少于 10 秒），单一的 Puppet 主服务应该能够处理这个负载。如果你的节点数量更大，我建议参考 *Mastering Puppet* 中描述的负载均衡配置，*Thomas Uphill* 著，*Packt Publishing* 出版。

Puppet 主服务是一个 Puppet 服务器，充当 Puppet 的 X509 证书颁发机构，并将编译后的清单（catalogs）分发给客户端节点。Puppet 配带了一个内置的 Web 服务器 **WEBrick**，可以处理非常少量的节点。在本节中，我们将看到如何使用该内置服务器来控制少量（少于 10）节点。

## 准备就绪

Puppet 主服务进程通过运行 `puppet master` 启动；大多数 Linux 发行版将 Puppet 主服务的启动和停止脚本放在一个单独的包中。为了开始，我们将创建一台名为 `puppet.example.com` 的新的 Debian 服务器。

## 如何操作...

1.  在新服务器上安装 Puppet，然后使用 Puppet 来安装 Puppet 主服务包：

    ```
    # puppet resource package puppetmaster ensure='installed' Notice: /Package[puppetmaster]/ensure: created package { 'puppetmaster': ensure => '3.7.0-1puppetlabs1', }

    ```

1.  现在启动 Puppet 主服务，并确保它在启动时自动启动：

    ```
    # puppet resource service puppetmaster ensure=true enable=true service { 'puppetmaster': ensure => 'running', enable => 'true', }

    ```

## 它是如何工作的...

Puppet 主服务包包括启动和停止 Puppet 主服务的脚本。我们使用 Puppet 来安装这个包并启动服务。一旦服务启动，我们可以将另一个节点指向 Puppet 主服务（你可能需要禁用机器上的基于主机的防火墙）。

1.  从另一个节点运行 `puppet agent` 启动一个 `puppet agent`，它将联系服务器并请求新的证书：

    ```
    t@ckbk:~$ sudo puppet agent -t
    Info: Creating a new SSL key for cookbook.example.com
    Info: Caching certificate for ca
    Info: Creating a new SSL certificate request for cookbook.example.com
    Info: Certificate Request fingerprint (SHA256): 06:C6:2B:C4:97:5D:16:F2:73:82:C4:A9:A7:B1:D0:95:AC:69:7B:27:13:A9:1A:4C:98:20:21:C2:50:48:66:A2
    Info: Caching certificate for ca
    Exiting; no certificate found and waitforcert is disabled

    ```

1.  现在在 Puppet 服务器上，签署新的密钥：

    ```
    root@puppet:~# puppet cert list
    pu  "cookbook.example.com" (SHA256) 06:C6:2B:C4:97:5D:16:F2:73:82:C4:A9:A7:B1:D0:95:AC:69:7B:27:13:A9:1A:4C:98:20:21:C2:50:48:66:A2
    root@puppet:~# puppet cert sign cookbook.example.com
    Notice: Signed certificate request for cookbook.example.com
    Notice: Removing file Puppet::SSL::CertificateRequestcookbook.example.com at'/var/lib/puppet/ssl/ca/requests/cookbook.example.com.pem'

    ```

1.  返回到烹饪书节点，并再次运行 Puppet：

    ```
    t@ckbk:~$ sudo puppet agent –vt
    Info: Caching certificate for cookbook.example.com
    Info: Caching certificate_revocation_list for ca
    Info: Caching certificate for cookbook.example.comInfo: Retrieving pluginfacts Info: Retrieving plugin Info: Caching catalog for cookbook Info: Applying configuration version '1410401823' Notice: Finished catalog run in 0.04 seconds

    ```

## 还有更多...

当我们运行 `puppet agent` 时，Puppet 会查找名为 `puppet.example.com` 的主机（因为我们的测试节点位于 `example.com` 域中）；如果找不到该主机，它会继续查找名为 Puppet 的主机。我们可以通过在 `puppet agent` 中使用 `--server` 选项来指定要联系的服务器。当我们安装 Puppet 主服务包并启动 Puppet 主服务时，Puppet 会根据我们的主机名创建默认的 SSL 证书。在接下来的部分中，我们将看到如何为我们的 Puppet 服务器创建一个包含多个 DNS 名称的 SSL 证书。

# 创建包含多个 DNS 名称的证书

默认情况下，Puppet 会为你的 Puppet 主服务器创建一个只包含服务器完全限定域名的 SSL 证书。根据你的网络配置，服务器被其他名称识别可能很有用。在本食谱中，我们将为 Puppet 主服务器创建一个包含多个 DNS 名称的新证书。

## 准备就绪

如果你还没有安装 Puppet 主包，请安装它。然后，你需要至少启动一次 Puppet 主服务以创建**证书颁发机构**（**CA**）。

## 如何操作...

步骤如下：

1.  使用以下命令停止正在运行的 Puppet 主进程：

    ```
    # service puppetmaster stop
    [ ok ] Stopping puppet master.

    ```

1.  删除（`clean`）当前的服务器证书：

    ```
    # puppet cert clean puppet
    Notice: Revoked certificate with serial 6
    Notice: Removing file Puppet::SSL::Certificate puppet at '/var/lib/puppet/ssl/ca/signed/puppet.pem'
    Notice: Removing file Puppet::SSL::Certificate puppet at '/var/lib/puppet/ssl/certs/puppet.pem'
    Notice: Removing file Puppet::SSL::Key puppet at '/var/lib/puppet/ssl/private_keys/puppet.pem'

    ```

1.  使用`--dns-alt-names`选项通过 Puppet 证书生成命令创建新的 Puppet 证书：

    ```
    root@puppet:~# puppet certificate generate puppet --dns-alt-names puppet.example.com,puppet.example.org,puppet.example.net --ca-location local
    Notice: puppet has a waiting certificate request
    true

    ```

1.  签署新证书：

    ```
    root@puppet:~# puppet cert --allow-dns-alt-names sign puppet
    Notice: Signed certificate request for puppet
    Notice: Removing file Puppet::SSL::CertificateRequest puppet at '/var/lib/puppet/ssl/ca/requests/puppet.pem'

    ```

1.  重启 Puppet 主进程：

    ```
    root@puppet:~# service puppetmaster restart
    [ ok ] Restarting puppet master.

    ```

## 它是如何工作的...

当你的 Puppet 代理连接到 Puppet 服务器时，它们会查找名为`Puppet`的主机，然后查找名为`Puppet.[你的域名]`的主机。如果你的客户端位于不同的域中，你需要让 Puppet 主服务器对所有正确的名称做出回应。通过删除现有证书并生成新证书，你可以让 Puppet 主服务器对多个 DNS 名称做出响应。

# 从 Passenger 运行 Puppet

我们在前一部分配置的 WEBrick 服务器无法处理大量节点。为了处理大量节点，需要一个可扩展的 Web 服务器。Puppet 是一个 Ruby 进程，因此我们需要一种在 Web 服务器中运行 Ruby 进程的方法。**Passenger**是解决此问题的方案。它允许我们在 Web 服务器中运行 Puppet 主进程（默认使用 Apache）。许多发行版都提供了一个 puppetmaster-passenger 软件包，可以为你配置这个功能。在本节中，我们将使用这个软件包来配置 Puppet 在 Passenger 中运行。

## 准备工作

安装 puppetmaster-passenger 软件包：

```
# puppet resource package puppetmaster-passenger ensure=installed
Notice: /Package[puppetmaster-passenger]/ensure: ensure changed 'purged'
 to 'present'
package { 'puppetmaster-passenger':
 ensure => '3.7.0-1puppetlabs1',
}

```

### 注意

使用`puppet resource`安装软件包可确保相同的命令在多个发行版上有效（前提是软件包名称相同）。

## 如何操作...

步骤如下：

1.  确保 Puppet 主站点在你的 Apache 配置中已启用。根据你的发行版，它可能位于`/etc/httpd/conf.d`或`/etc/apache2/sites-enabled`。配置文件应该已经为你创建，并包含以下信息：

    ```
    PassengerHighPerformance on
    PassengerMaxPoolSize 12
    PassengerPoolIdleTime 1500
    # PassengerMaxRequests 1000
    PassengerStatThrottleRate 120
    RackAutoDetect Off
    RailsAutoDetect Off
    Listen 8140

    ```

1.  这些行是 Passenger 的调优设置。然后，文件指示 Apache 在端口 8140 上监听，这是 Puppet 主端口。接着，创建一个`VirtualHost`定义，加载 Puppet CA 证书和 Puppet 主证书：

    ```
    <VirtualHost *:8140>
     SSLEngine on
     SSLProtocol             ALL -SSLv2 -SSLv3
     SSLCertificateFile      /var/lib/puppet/ssl/certs/puppet.pem
     SSLCertificateKeyFile   /var/lib/puppet/ssl/private_keys/puppet.pem
     SSLCertificateChainFile /var/lib/puppet/ssl/certs/ca.pem
     SSLCACertificateFile    /var/lib/puppet/ssl/certs/ca.pem
     SSLCARevocationFile     /var/lib/puppet/ssl/ca/ca_crl.pem
     SSLVerifyClient optional
     SSLVerifyDepth  1
     SSLOptions +StdEnvVars +ExportCertData

    ```

    ### 提示

    根据你所使用的 puppetmaster-passenger 软件包的版本，这里可能会有更多或更少的 SSL 配置行。

1.  接下来，设置几个重要的头信息，以便 Passenger 进程可以访问客户端节点发送的 SSL 信息：

    ```
    RequestHeader unset X-Forwarded-For
    RequestHeader set X-SSL-Subject %{SSL_CLIENT_S_DN}e
    RequestHeader set X-Client-DN %{SSL_CLIENT_S_DN}e
    RequestHeader set X-Client-Verify %{SSL_CLIENT_VERIFY}e

    ```

1.  最后，给出了 Passenger 配置文件`config.ru`的位置，以及`DocumentRoot`位置，具体如下：

    ```
     DocumentRoot /usr/share/puppet/rack/puppetmasterd/public/
     RackBaseURI /

    ```

1.  `config.ru`文件应位于`/usr/share/puppet/rack/puppetmasterd/`，并应包含以下内容：

    ```
    $0 = "master"
    ARGV << "--rack"
    ARGV << "--confdir" << "/etc/puppet"
    ARGV << "--vardir"  << "/var/lib/puppet"
    require 'puppet/util/command_line'
    run Puppet::Util::CommandLine.new.execute

    ```

1.  配置好 passenger apache 配置文件，并正确配置 `config.ru` 文件后，启动 apache 服务器，并验证 apache 是否在 Puppet master 端口上监听（如果你之前配置了独立的 Puppet master，必须现在停止该进程，使用命令 `service puppetmaster stop`）：

    ```
    root@puppet:~ # service apache2 start
    [ ok ] Starting web server: apache2
    root@puppet:~ # lsof -i :8140
    COMMAND  PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
    apache2 9048     root    8u  IPv6  16842      0t0  TCP *:8140 (LISTEN)
    apache2 9069 www-data    8u  IPv6  16842      0t0  TCP *:8140 (LISTEN)
    apache2 9070 www-data    8u  IPv6  16842      0t0  TCP *:8140 (LISTEN)

    ```

## 它是如何工作的...

passenger 配置文件使用现有的 Puppet master 证书监听 8140 端口，并处理服务器与客户端之间的所有 SSL 通信。一旦证书信息处理完毕，连接就会被交给一个由 passenger 启动的 ruby 进程，该进程使用来自 `config.ru` 文件的命令行参数。

在这种情况下，`$0`变量被设置为`master`，而参数变量被设置为`--rack --confdir /etc/puppet --vardir /var/lib/puppet`；这相当于从命令行运行以下内容：

```
puppet master --rack --confdir /etc/puppet --vardir /var/lib/puppet

```

## 还有更多...

你可以向 `config.ru` 文件添加额外的配置参数，以进一步改变 Puppet 在通过 passenger 运行时的行为。例如，要在 passenger Puppet master 上启用调试，请在 `config.ru` 文件中的 `Puppet::Util::CommandLine.new.execute` 之前添加以下行：

```
ARGV << "--debug"
```

# 设置环境

Puppet 中的环境是包含不同版本 Puppet 清单的目录。在 Puppet 3.6 版本之前，环境并不是 Puppet 的默认配置。在较新版本的 Puppet 中，环境是默认配置的。

每当一个节点连接到 Puppet master 时，它会通知 Puppet master 它的环境。默认情况下，所有节点都报告到 `production` 环境。这会导致 Puppet master 在生产环境中查找清单。你可以在运行 puppet agent 时通过 `--environment` 设置指定一个备用环境，或者在 `/etc/puppet/puppet.conf` 的 [agent] 部分中设置 `environment = newenvironment`。

## 准备工作

通过在 `/etc/puppet/puppet.conf` 的 `[main]` 部分添加以下行，设置你的安装的 `environmentpath` 功能：

```
[main]
...
environmentpath=/etc/puppet/environments
```

## 如何操作...

步骤如下：

1.  在 `/etc/puppet/environments` 下创建一个 `production` 目录，包含 `modules` 和 `manifests` 目录。然后创建一个 `site.pp` 文件，该文件在 `/tmp` 中创建一个文件，如下所示：

    ```
    root@puppet:~# cd /etc/puppet/environments/
    root@puppet:/etc/puppet/environments# mkdir -p production/{manifests,modules}
    root@puppet:/etc/puppet/environments# vim production/manifests/site.pp
    node default {
     file {'/tmp/production':
     content => "Hello World!\nThis is production\n",
     }
    }

    ```

1.  在主节点上运行 puppet agent 以连接到主节点，并验证生产代码是否已交付：

    ```
    root@puppet:~# puppet agent -vt
    Info: Retrieving pluginfacts
    Info: Retrieving plugin
    Info: Caching catalog for puppet
    Info: Applying configuration version '1410415538'
    Notice: /Stage[main]/Main/Node[default]/File[/tmp/production]/ensure: defined content as '{md5}f7ad9261670b9da33a67a5126933044c'
    Notice: Finished catalog run in 0.04 seconds
    # cat /tmp/production
    Hello World!
    This is production

    ```

1.  配置另一个环境 `devel`。在 `devel` 环境中创建一个新的清单：

    ```
    root@puppet:/etc/puppet/environments# mkdir -p devel/{manifests,modules}
    root@puppet:/etc/puppet/environments# vim devel/manifests/site.pp
    node default {
     file {'/tmp/devel':
     content => "Good-bye! Development\n",
     }
    }

    ```

1.  通过运行以下命令，使用 `--environment devel` 选项来应用新环境：

    ```
    root@puppet:/etc/puppet/environments# puppet agent -vt --environment devel
    Info: Retrieving pluginfacts
    Info: Retrieving plugin
    Info: Caching catalog for puppet
    Info: Applying configuration version '1410415890'
    Notice: /Stage[main]/Main/Node[default]/File[/tmp/devel]/ensure: defined content as '{md5}b6313bb89bc1b7d97eae5aa94588eb68'
    Notice: Finished catalog run in 0.04 seconds
    root@puppet:/etc/puppet/environments# cat /tmp/devel
    Good-bye! Development

    ```

### 提示

你可能需要重新启动 apache2 才能启用新环境，这取决于你的 Puppet 版本和 `puppet.conf` 中的 `environment_timeout` 参数。

## 还有更多...

每个环境可以有自己的 `modulepath`，只要在环境目录中创建一个 `environment.conf` 文件。关于环境的更多信息可以在 Puppet Labs 网站上找到，链接是 [`docs.puppetlabs.com/puppet/latest/reference/environments.html`](https://docs.puppetlabs.com/puppet/latest/reference/environments.html)。

# 配置 PuppetDB

PuppetDB 是一个用于存储与 Puppet 主服务器连接的节点信息的数据库。PuppetDB 还是导出资源的存储区域。导出资源是指在节点上定义但应用于其他节点的资源。安装 PuppetDB 的最简单方法是使用 Puppet Labs 提供的 PuppetDB 模块。从这一点开始，我们假设你正在使用 `puppet.example.com` 机器，并且拥有基于 Passenger 的 Puppet 配置。

## 准备工作

在之前步骤中创建的生产环境中安装 PuppetDB 模块。如果你没有创建目录环境，也不用担心，使用 `puppet module install` 将会把模块安装到你的安装目录的正确位置，执行以下命令：

```
root@puppet:~# puppet module install puppetlabs-puppetdb
Notice: Preparing to install into /etc/puppet/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/environments/production/modules
└─┬ puppetlabs-puppetdb (v3.0.1)
 ├── puppetlabs-firewall (v1.1.3)
 ├── puppetlabs-inifile (v1.1.3)
 └─┬ puppetlabs-postgresql (v3.4.2)
 ├─┬ puppetlabs-apt (v1.6.0)
 │ └── puppetlabs-stdlib (v4.3.2)
 └── puppetlabs-concat (v1.1.0)

```

## 如何操作...

现在我们的 Puppet 主服务器已经安装了 PuppetDB 模块，我们需要将 PuppetDB 模块应用到 Puppet 主服务器。我们可以在站点清单中完成此操作。在你的（生产）`site.pp` 文件中添加以下内容：

```
node puppet {
  class { 'puppetdb': }
  class { 'puppetdb::master::config': 
    puppet_service_name => 'apache2',
  }
}
```

运行 `puppet agent` 来应用 `puppetdb` 类和 `puppetdb::master::config` 类：

```
root@puppet:~# puppet agent -t
Info: Caching catalog for puppet
Info: Applying configuration version '1410416952'
...
Info: Class[Puppetdb::Server::Jetty_ini]: Scheduling refresh of Service[puppetdb]
Notice: Finished catalog run in 160.78 seconds

```

## 它是如何工作的...

PuppetDB 模块是一个很好的例子，展示了如何将复杂的配置任务自动化。通过将 `puppetdb` 类添加到 Puppet 主节点，Puppet 安装并配置了 `postgresql` 和 `puppetdb`。

当我们调用 `puppetdb::master::config` 类时，我们将 `puppet_service_name` 变量设置为 `apache2`，这是因为我们通过 Passenger 运行 Puppet。如果没有这一行，我们的代理会尝试启动 puppetmaster 进程，而不是 `apache2`。

代理接着为 PuppetDB 设置了配置文件，并配置了 Puppet 使用 PuppetDB。如果你查看`/etc/puppet/puppet.conf`，你会看到以下两行新的配置：

```
storeconfigs = true
storeconfigs_backend = puppetdb
```

## 还有更多内容...

现在 PuppetDB 已经配置完毕，并且我们成功运行了代理，PuppetDB 将会有我们可以查询的数据：

```
root@puppet:~# puppet node status puppet
puppet
Currently active
Last catalog: 2014-09-11T06:45:25.267Z
Last facts: 2014-09-11T06:45:22.351Z

```

# 配置 Hiera

**Hiera** 是 Puppet 的信息仓库。使用 Hiera，你可以对关于节点的数据进行分层分类，这些数据保存在清单之外。这对于共享代码和处理任何 Puppet 部署中不可避免的例外情况非常有用。

## 准备工作

Hiera 应该已经作为 Puppet 主服务器的依赖项安装。如果还没有安装，可以通过 Puppet 来安装它：

```
root@puppet:~# puppet resource package hiera ensure=installed
package { 'hiera':
 ensure => '1.3.4-1puppetlabs1',
}

```

## 如何操作...

1.  Hiera 是通过一个 yaml 文件 `/etc/puppet/hiera.yaml` 进行配置的。创建该文件，并添加以下内容作为最小配置：

    ```
    ---
    :hierarchy:
     - common
    :backends:
     - yaml
    :yaml:
     :datadir: '/etc/puppet/hieradata'

    ```

1.  创建在层次结构中引用的 `common.yaml` 文件：

    ```
    root@puppet:/etc/puppet# mkdir hieradata
    root@puppet:/etc/puppet# vim hieradata/common.yaml
    ---
    message: 'Default Message'

    ```

1.  编辑 `site.pp` 文件并基于 Hiera 值添加一个通知资源：

    ```
    node default {
     $message = hiera('message','unknown')
     notify {"Message is $message":}
    }

    ```

1.  将清单应用于测试节点：

    ```
    t@ckbk:~$ sudo puppet agent -t
    Info: Retrieving pluginfacts
    Info: Retrieving plugin
    ...
    Info: Caching catalog for cookbook-test
    Info: Applying configuration version '1410504848'
    Notice: Message is Default Message
    Notice: /Stage[main]/Main/Node[default]/Notify[Message is Default Message]/message: defined 'message' as 'Message is Default Message'
    Notice: Finished catalog run in 0.06 seconds

    ```

## 它是如何工作的...

Hiera 使用层次结构搜索一组 yaml 文件，以找到合适的值。我们在 `hiera.yaml` 中定义了这个层次结构，并且只包含了 `common.yaml` 的条目。我们在 `site.pp` 中使用了 `hiera` 函数来查找消息的值并将其存储在变量 `$message` 中。用于定义层次结构的值可以是系统上定义的任何 facter 数据。常见的层次结构示例如下：

```
:hierarchy:
  - hosts/%{hostname}
  - os/%{operatingsystem}
  - network/%{network_eth0}
  - common
```

## 还有更多...

Hiera 可以用于带有参数化类的自动参数查找。例如，如果你有一个名为 `cookbook::example` 的类，其中有一个名为 `publisher` 的参数，你可以在 Hiera 的 yaml 文件中加入以下内容来自动设置这个参数：

```
cookbook::example::publisher: 'PacktPub'
```

另一个常用的 fact 是 `environment`，你可以通过 `%{environment}` 引用客户端节点的 `environment`，如下所示的层次结构：

```
:hierarchy:
hosts/%{hostname}
os/%{operatingsystem}
environment/%{environment}
common
```

### 提示

一个好的经验法则是将层次结构限制为 8 层或更少。请记住，每次使用 Hiera 搜索参数时，所有层次都会被搜索，直到找到匹配项。

默认的 Hiera 函数返回与搜索键匹配的第一个结果，你还可以使用 `hiera_array` 和 `hiera_hash` 来搜索并返回 Hiera 中存储的所有值。

Hiera 也可以从命令行进行搜索，如以下命令所示（请注意，目前命令行的 Hiera 工具使用 `/etc/hiera.yaml` 作为其配置文件，而 Puppet 主控使用 `/etc/puppet/hiera.yaml`）：

```
root@puppet:/etc/puppet# rm /etc/hiera.yaml 
root@puppet:/etc/puppet# ln -s /etc/puppet/hiera.yaml /etc/
root@puppet:/etc/puppet# hiera message
Default Message

```

### 注意

欲了解更多信息，请参阅 Puppet labs 网站 [`docs.puppetlabs.com/hiera/1/`](https://docs.puppetlabs.com/hiera/1/)。

# 使用 Hiera 设置特定节点数据

在我们在 `hiera.yaml` 中定义的层次结构中，我们基于主机名 fact 创建了一个条目；在本节中，我们将在 Hiera 数据的 `hosts` 子目录中创建 yaml 文件，其中包含特定主机的信息。

## 准备工作

按照上一节的步骤安装并配置 Hiera，并使用前面配方中定义的层次结构，该层次结构包括一个 `hosts/%{hostname}` 条目。

## 如何操作...

以下是步骤：

1.  在 `/etc/puppet/hieradata/hosts` 创建一个文件，该文件的名称与测试节点的主机名相同。例如，如果主机名为 `cookbook-test`，则文件应命名为 `cookbook-test.yaml`。

1.  在此文件中插入特定消息：

    ```
    message: 'This is the test node for the cookbook'
    ```

1.  在两个不同的测试节点上运行 Puppet，以注意其中的差异：

    ```
    t@ckbk:~$ sudo puppet agent -t
    Info: Caching catalog for cookbook-test
    Notice: Message is This is the test node for the cookbook
    [root@hiera-test ~]# puppet agent -t
    Info: Caching catalog for hiera-test.example.com
    Notice: Message is Default Message

    ```

## 它是如何工作的...

Hiera 会在层次结构中搜索与 facter 返回的值匹配的文件。在此情况下，通过将节点的主机名替换到搜索路径`/etc/puppet/hieradata/hosts/%{hostname}.yaml`中，找到了 `cookbook-test.yaml` 文件。

使用 Hiera，可以大大减少 Puppet 代码的复杂性。我们将使用 `yaml` 文件来存储分开的值，而不再需要像以前那样写大型的 `case` 语句或嵌套的 `if` 语句。

# 使用 hiera-gpg 存储机密数据

如果您使用 Hiera 存储配置数据，可以使用名为**hiera-gpg**的 gem，它为 Hiera 添加了一个加密后端，允许您保护存储在 Hiera 中的值。

## 准备就绪

设置 hiera-gpg，请按照以下步骤操作：

1.  安装`ruby-dev`软件包；它将用于构建`hiera-gpg` gem，如下所示：

    ```
    root@puppet:~# puppet resource package ruby-dev ensure=installed
    Notice: /Package[ruby-dev]/ensure: ensure changed 'purged' to 'present'
    package { 'ruby-dev':
     ensure => '1:1.9.3',
    }

    ```

1.  使用 gem 提供程序安装`hiera-gpg` gem：

    ```
    root@puppet:~# puppet resource package hiera-gpg ensure=installed provider=gem
    Notice: /Package[hiera-gpg]/ensure: created
    package { 'hiera-gpg':
     ensure => ['1.1.0'],
    }

    ```

1.  按如下方式修改`hiera.yaml`文件：

    ```
        :hierarchy:
            - secret
            - common
        :backends:
            - yaml
            - gpg
        :yaml:
            :datadir: '/etc/puppet/hieradata'
        :gpg:
            :datadir: '/etc/puppet/secret'
    ```

## 如何操作...

在此示例中，我们将创建一段加密数据，并使用`hiera-gpg`按如下方式检索它：

1.  在`/etc/puppet/secret`处创建`secret.yaml`文件，内容如下：

    ```
    top_secret: 'Val Kilmer'

    ```

1.  如果您还没有 GnuPG 加密密钥，请按照第四章中*使用 GnuPG 加密机密*的步骤进行操作，*与文件和软件包一起工作*。

1.  使用以下命令将`secret.yaml`文件加密为此密钥（将`puppet@puppet.example.com`替换为您在创建密钥时指定的电子邮件地址）。这将生成`secret.gpg`文件：

    ```
    root@puppet:/etc/puppet/secret# gpg -e -o secret.gpg -r puppet@puppet.example.com secret.yaml 
    root@puppet:/etc/puppet/secret# file secret.gpg
    secret.gpg: GPG encrypted data

    ```

1.  删除明文的`secret.yaml`文件：

    ```
    root@puppet:/etc/puppet/secret# rm secret.yaml

    ```

1.  按如下方式修改`site.pp`文件中的默认节点：

    ```
    node default {
     $message = hiera('top_secret','Deja Vu')
     notify { "Message is $message": }
    }

    ```

1.  现在在节点上运行 Puppet：

    ```
    [root@hiera-test ~]# puppet agent -t
    Info: Caching catalog for hiera-test.example.com
    Info: Applying configuration version '1410508276'
    Notice: Message is Deja Vu
    Notice: /Stage[main]/Main/Node[default]/Notify[Message is Deja Vu]/message: defined 'message' as 'Message is Deja Vu'
    Notice: Finished catalog run in 0.08 seconds

    ```

## 它是如何工作的...

安装`hiera-gpg`时，它为 Hiera 添加了解密`.gpg`文件的功能。因此，您可以将任何机密数据放入`.yaml`文件中，然后使用 GnuPG 将其加密到相应的密钥。只有拥有正确密钥的机器才能访问这些数据。

例如，您可以使用`hiera-gpg`加密 MySQL root 密码，并仅在数据库服务器上安装相应的密钥。尽管其他机器可能也有`secret.gpg`文件的副本，但除非它们拥有解密密钥，否则无法读取此文件。

## 还有更多内容...

您可能还想了解`hiera-eyaml`，这是另一个 Hiera 的秘密数据后端，它支持加密 Hiera 数据文件中的单个值。如果您需要在单个文件中混合加密和未加密的事实数据，这将非常有用。了解更多关于 hiera-eyaml 的信息，请访问[`github.com/TomPoulton/hiera-eyaml`](https://github.com/TomPoulton/hiera-eyaml)。

## 另见

+   第四章中*使用 GnuPG 加密机密*的步骤，*与文件和软件包一起工作*。

# 使用 MessagePack 序列化

在集中式架构中运行 Puppet 会在节点之间产生大量流量。大部分流量是 JSON 和 yaml 数据。Puppet 最新版本的实验性功能允许使用**MessagePack**（**msgpack**）对这些数据进行序列化。

## 准备就绪

将 msgpack gem 安装到您的 Puppet 主节点和节点上。使用 Puppet 资源让 Puppet 为您完成这项工作。此时，您可能需要在节点/服务器上安装`ruby-dev`或`ruby-devel`软件包：

```
t@ckbk:~$ sudo puppet resource package msgpack ensure=installedprovider=gem
Notice: /Package[msgpack]/ensure: created
package { 'msgpack':
 ensure => ['0.5.8'],
}

```

## 如何操作...

在节点`puppet.conf`文件的`[agent]`部分，将`preferred_serialization_format`设置为`msgpack`：

```
[agent]
preferred_serialization_format=msgpack
```

## 它是如何工作的...

当节点开始与主机通信时，主机会收到此选项。任何支持与 `msgpack` 进行序列化的类将通过节点与主机之间的 `msgpack.Serialization` 数据传输。理论上，这会通过优化传输的数据，提高节点之间的通信速度。此功能仍在实验阶段。

# 使用 Git 钩子进行自动语法检查

如果我们能够在提交之前就知道清单中是否有语法错误，那该多好。你可以使用 `puppet parser validate` 命令让 Puppet 检查清单的语法：

```
t@ckbk:~$ puppet parser validate bootstrap.pp
Error: Could not parse for environment production: Syntax error at
 'File'; expected '}' at /home/thomas/bootstrap.pp:3 

```

这尤其有用，因为清单中的任何错误都会导致 Puppet 在任何节点上停止运行，即使是那些没有使用该部分清单的节点也是如此。因此，提交一个有问题的清单可能会导致 Puppet 停止向生产环境应用更新，直到问题被发现，而这可能会带来严重后果。避免这种情况的最佳方法是通过在版本控制仓库中使用预提交钩子（precommit hook）来自动化语法检查。

## 如何操作…

按照以下步骤操作：

1.  在 Puppet 仓库中创建一个新的 `hooks` 目录：

    ```
    t@mylaptop:~/puppet$ mkdir hooks

    ```

1.  创建文件`hooks/check_syntax.sh`，内容如下（基于 Puppet Labs 的脚本）：

    ```
    #!/bin/sh

    syntax_errors=0
    error_msg=$(mktemp /tmp/error_msg.XXXXXX)

    if git rev-parse --quiet --verify HEAD > /dev/null
    then
        against=HEAD
    else
        # Initial commit: diff against an empty tree object
        against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
    fi

    # Get list of new/modified manifest and template files
      to check (in git index)
    for indexfile in 'git diff-index --diff-filter=AM --
      name-only --cached $against | egrep '\.(pp|erb)''
    do
        # Don't check empty files
        if [ 'git cat-file -s :0:$indexfile' -gt 0 ]
        then
            case $indexfile in
                *.pp )
                    # Check puppet manifest syntax
                    git cat-file blob :0:$indexfile | 
                      puppet parser validate > $error_msg ;;
                *.erb )
                    # Check ERB template syntax
                    git cat-file blob :0:$indexfile | 
                      erb -x -T - | ruby -c 2> $error_msg >
                        /dev/null ;;
            esac
            if [ "$?" -ne 0 ]
            then
                echo -n "$indexfile: "
                cat $error_msg
                syntax_errors='expr $syntax_errors + 1'
            fi
        fi
    done

    rm -f $error_msg

    if [ "$syntax_errors" -ne 0 ]
    then
        echo "Error: $syntax_errors syntax errors found,
          aborting commit."
        exit 1
    fi
    ```

1.  使用以下命令为`hook`脚本设置执行权限：

    ```
    t@mylaptop:~/puppet$ chmod a+x hooks/check_syntax.sh

    ```

1.  现在，将脚本通过符号链接或复制到 hooks 目录中的预提交钩子（precommit hook）中。如果你的 Git 仓库在 `~/puppet` 目录下，则按以下方式在 `~/puppet/hooks/pre-commit` 创建符号链接：

    ```
    t@mylaptop:~/puppet$ ln -s ~/puppet/hooks/check_syntax.sh.git/hooks/pre-commit

    ```

## 它是如何工作的…

当 `check_syntax.sh` 脚本作为 Git 的预提交钩子使用时，它将防止你提交任何包含语法错误的文件：

```
t@mylaptop:~/puppet$ git commit -m "test commit"
Error: Could not parse for environment production: Syntax error at
 '}' at line 3
Error: Try 'puppet help parser validate' for usage
manifests/nodes.pp: Error: 1 syntax errors found, aborting commit.

```

如果将 `hooks` 目录添加到 Git 仓库中，任何有仓库检出的用户都可以将脚本复制到本地的 `hooks` 目录，以获得此语法检查功能。

# 使用 Git 推送代码

正如我们在去中心化模型中看到的，Git 可以通过结合使用 `ssh` 和 `ssh` 密钥在机器之间传输文件。让 Git 钩子在每次成功提交到仓库时执行相同的操作也是有用的。

存在一个叫做 post-commit 的钩子，可以在成功提交到仓库后运行。在本配方中，我们将创建一个钩子，用于将代码从 Git 服务器上的 Git 仓库更新到 Puppet 主机上的代码。

## 准备工作

按照以下步骤开始：

1.  创建一个 `ssh` 密钥，使其能够访问 Puppet 主机上的 Puppet 用户，并将该密钥安装到 `git.example.com` 上 Git 用户的帐户中：

    ```
    [git@git ~]$ ssh-keygen -f ~/.ssh/puppet_rsa
    Generating public/private rsa key pair.
    Your identification has been saved in /home/git/.ssh/puppet_rsa.
    Your public key has been saved in /home/git/.ssh/puppet_rsa.pub.
    Copy the public key into the authorized_keys file of the puppet user on your puppetmaster
    puppet@puppet:~/.ssh$ cat puppet_rsa.pub >>authorized_keys

    ```

1.  修改 Puppet 帐号以允许 Git 用户按以下方式登录：

    ```
    root@puppet:~# chsh puppet -s /bin/bash

    ```

## 如何操作…

执行以下步骤：

1.  现在 Git 用户可以作为 Puppet 用户登录到 Puppet 主机，修改 Git 用户的 `ssh` 配置，使其默认使用新创建的 `ssh` 密钥：

    ```
    [git@git ~]$ vim .ssh/config
    Host puppet.example.com
     IdentityFile ~/.ssh/puppet_rsa

    ```

1.  使用以下命令将 Puppet 主机添加为 Git 服务器上的 Puppet 仓库的远程位置：

    ```
    [git@git puppet.git]$ git remote add puppetmaster puppet@puppet.example.com:/etc/puppet/environments/puppet.git

    ```

1.  在 Puppet 主服务器上，将 `production` 目录移开，并检出你的 Puppet 仓库：

    ```
    root@puppet:~# chown -R puppet:puppet /etc/puppet/environments
    root@puppet:~# sudo -iu puppet
    puppet@puppet:~$ cd /etc/puppet/environments/
    puppet@puppet:/etc/puppet/environments$ mv production production.orig
    puppet@puppet:/etc/puppet/environments$ git clone git@git.example.com:repos/puppet.git
    Cloning into 'puppet.git'...
    remote: Counting objects: 63, done.
    remote: Compressing objects: 100% (52/52), done.
    remote: Total 63 (delta 10), reused 0 (delta 0)
    Receiving objects: 100% (63/63), 9.51 KiB, done.
    Resolving deltas: 100% (10/10), done.

    ```

1.  现在我们在 Puppet 服务器上有一个本地裸仓库，可以将其推送，并将其远程克隆到 `production` 目录：

    ```
    puppet@puppet:/etc/puppet/environments$ git clone puppet.git production
    Cloning into 'production'...
    done.

    ```

1.  现在从 Git 服务器执行 Git 推送到 Puppet 主服务器：

    ```
    [git@git ~]$ cd repos/puppet.git/
    [git@git puppet.git]$ git push puppetmaster
    Everything up-to-date

    ```

1.  在 Git 服务器上的仓库的 `hooks` 目录中创建一个 post-commit 文件，内容如下：

    ```
    [git@git puppet.git]$ vim hooks/post-commit
    #!/bin/sh
    git push puppetmaster
    ssh puppet@puppet.example.com "cd /etc/puppet/environments/production && git pull"
    [git@git puppet.git]$ chmod 755 hooks/post-commit

    ```

1.  从你的笔记本提交一个更改到仓库，并验证该更改是否传播到 Puppet 主服务器，步骤如下：

    ```
    t@mylaptop puppet$ vim README
    t@mylaptop puppet$ git add README
    t@mylaptop puppet$ git commit -m "Adding README"
    [master 8148902] Adding README
     1 file changed, 4 deletions(-)
    t@mylaptop puppet$ git push
    X11 forwarding request failed on channel 0
    Counting objects: 5, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (3/3), 371 bytes | 0 bytes/s, done.
    Total 3 (delta 1), reused 0 (delta 0)
    remote: To puppet@puppet.example.com:/etc/puppet/environments/puppet.git
    remote:    377ed44..8148902  master -> master
    remote: From /etc/puppet/environments/puppet
    remote:    377ed44..8148902  master     -> origin/master
    remote: Updating 377ed44..8148902
    remote: Fast-forward
    remote:  README |    4 ----
    remote:  1 file changed, 4 deletions(-)
    To git@git.example.com:repos/puppet.git
     377ed44..8148902  master -> master

    ```

## 它是如何工作的...

我们在 Puppet 主服务器上创建了一个裸仓库，然后将其作为 `git.example.com` 上仓库的远程。然后我们将该裸仓库克隆到 `production` 目录中。我们将 `puppet.example.com` 上的裸仓库添加为 `git.example.com` 上裸仓库的远程。接着，我们在 `git.example.com` 上的仓库中创建一个 post-receive 钩子。

钩子对 Puppet 主服务器上的裸仓库执行 Git 推送。然后，我们从 Puppet 主服务器上的更新的裸仓库更新 `production` 目录。在下一节中，我们将修改钩子以使用分支。

# 使用 Git 管理环境

分支是一种在单个源代码仓库中保持多个开发轨迹的方法。Puppet 环境与 Git 分支非常相似。你可以在不同的分支之间拥有相同的代码，并且稍有不同，就像你可以为不同的环境创建不同的模块一样。在本节中，我们将展示如何使用 Git 分支在 Puppet 主服务器上定义环境。

## 准备工作

在上一节中，我们创建了一个基于主分支的 `production` 目录；现在我们将删除该目录：

```
puppet@puppet:/etc/puppet/environments$ mv production production.master

```

## 如何操作...

修改 `post-receive` 钩子以接受分支变量。该钩子将使用此变量在 Puppet 主服务器上创建一个目录，步骤如下：

```
#!/bin/sh

read oldrev newrev refname
branch=${refname#*\/*\/}

git push puppetmaster $branch
ssh puppet@puppet.example.com "if [ ! -d 
/etc/puppet/environments/$branch ]; then git clone
 /etc/puppet/environments/puppet.git
 /etc/puppet/environments/$branch; fi; cd
 /etc/puppet/environments/$branch; git checkout $branch; git pull"

```

再次修改你的 `README` 文件，并推送到 `git.example.com` 上的仓库：

```
t@mylaptop puppet$ git add README
t@mylaptop puppet$ git commit -m "Adding README"
[master 539d9f8] Adding README
 1 file changed, 1 insertion(+)
t@mylaptop puppet$ git push
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 374 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: To puppet@puppet.example.com:/etc/puppet/environments/puppet.git
remote:    0d6b49f..539d9f8  master -> master
remote: Cloning into '/etc/puppet/environments/master'...
remote: done.
remote: Already on 'master'
remote: Already up-to-date.
To git@git.example.com:repos/puppet.git
 0d6b49f..539d9f8  master -> master

```

## 它是如何工作的...

这个钩子现在读取 `refname` 并解析出正在更新的分支。我们使用这个分支变量将仓库克隆到一个新的目录，并检出该分支。

## 还有更多...

现在，当我们想创建一个新环境时，可以在 Git 仓库中创建一个新分支。这个分支将在 Puppet 主服务器上创建一个目录。Git 仓库的每个分支都代表 Puppet 主服务器上的一个环境：

1.  按照以下命令行创建 `production` 分支：

    ```
    t@mylaptop puppet$ git branch production
    t@mylaptop puppet$ git checkout production
    Switched to branch 'production'

    ```

1.  更新 `production` 分支并推送到 Git 服务器，步骤如下：

    ```
    t@mylaptop puppet$ vim README
    t@mylaptop puppet$ git add README
    t@mylaptop puppet$ git commit -m "Production Branch"
    t@mylaptop puppet$ git push origin production
    Counting objects: 7, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (3/3), 372 bytes | 0 bytes/s, done.
    Total 3 (delta 1), reused 0 (delta 0)
    remote: To puppet@puppet.example.com:/etc/puppet/environments/puppet.git
    remote:    11db6e5..832f6a9  production -> production
    remote: Cloning into '/etc/puppet/environments/production'...
    remote: done.
    remote: Switched to a new branch 'production'
    remote: Branch production set up to track remote branch production from origin.
    remote: Already up-to-date.
    To git@git.example.com:repos/puppet.git
     11db6e5..832f6a9  production -> production

    ```

现在，每当我们创建一个新分支时，环境目录中将创建一个对应的目录。环境和分支之间建立了一对一的映射关系。

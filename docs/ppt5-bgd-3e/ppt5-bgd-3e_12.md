# 第十二章：将一切汇集在一起

|   | *男人的气度是耐心。掌握之道是九分耐心。* |   |
| --- | --- | --- |
|   | --*厄休拉·K·勒古恩，《地海巫师》* |

在本章中，我们将应用前面所有章节中的思想，看看完整的、工作的 Puppet 基础设施是什么样子，使用一个展示所有本书中解释的原则的示例仓库。你可以将其作为自己 Puppet 代码库的基础，并根据需要进行调整和扩展。

![将一切汇集在一起](img/8880_12_01.jpg)

# 获取示例仓库

示例仓库可以在 GitHub 上找到，你可以像克隆本书示例仓库一样，通过运行以下命令来克隆它：

```
git clone -b production https://github.com/bitfield/control-repo-3

```

它包含了你管理节点所需的一切内容：

+   用户账户和 SSH 密钥

+   SSH 和 `sudoers` 配置

+   时区和 NTP 设置

+   Hiera 数据

+   自动 Puppet 更新和应用脚本

+   新节点的引导脚本

它还包含一个 Vagrantfile，以便你可以在 Vagrant 虚拟机上尝试这个仓库。

## 复制仓库

如果你打算将示例仓库作为自己 Puppet 仓库的基础，你需要将其复制一份，以便自己编辑和维护。

你可以通过两种方式做到这一点。一种是将仓库 *fork* 到自己的 GitHub 账户中。为此，请登录 GitHub 并浏览到示例仓库 URL：

[`github.com/bitfield/control-repo-3.git`](https://github.com/bitfield/control-repo-3.git)

在页面右上角找到 **Fork** 按钮并点击它。这将会在你的账户下创建一个新仓库，包含示例仓库中的所有代码和历史。

或者，你也可以按照以下步骤操作：

1.  在你的 GitHub 账户中创建一个新仓库（命名为 `puppet`、`control-repo`，或其他你喜欢的名字）。

1.  记下仓库的 URL。

1.  将示例仓库克隆到你的个人机器上：

    ```
    git clone -b production https://github.com/bitfield/control-repo-3
    cd control-repo-3

    ```

1.  重命名原始仓库的远程（以便将来能获得更新）：

    ```
    git remote rename origin upstream

    ```

1.  将你的新仓库添加为 `origin` 远程仓库（使用你之前记下的仓库 URL）：

    ```
    git remote add origin YOUR_GIT_URL

    ```

1.  推送到新的远程仓库：

    ```
    git push origin production

    ```

现在，你的仓库包含了完整的示例仓库副本，你可以根据需要编辑和定制它。

随着原始仓库未来的更新，你将能够将这些更改拉取到你自己的版本中。要从上游获取更改，请运行以下命令：

```
git fetch upstream
git rebase upstream/production

```

# 理解示例仓库

现在是时候看看前面章节中的所有思想如何结合起来了。了解完整的 Puppet 基础设施是如何工作的对你应该很有帮助，你也可以将这个仓库作为自己项目的基础。我们将在本章稍后介绍如何做到这一点，但首先，我们来简单了解一下仓库的整体结构。

## 控制仓库

**控制仓库**是一个 Puppet 代码库，它不包含模块，或者只包含特定站点的模块，是组织 Puppet 代码的好方法。

在第七章，*精通模块*中，我们学习了如何使用 `r10k` 工具通过 Puppetfile 管理模块。Puppetfile 指定我们使用哪些模块，及其确切版本和来源（通常是 Puppet Forge，也可以来自远程 Git 仓库）。

因此，我们的 Puppet 仓库只需要包含一个 Puppetfile，以及我们的 Hiera 数据和 `role`、`profile` 模块。

## 模块管理

因为 `r10k` 期望通过 Puppetfile 管理 `modules/` 目录中的所有内容，我们的 **站点特定模块** 保存在控制仓库中一个名为 `site-modules/` 的单独目录下。

为了启用此功能，我们需要将以下设置添加到 `environment.conf` 文件中：

```
modulepath = "modules:site-modules:$basemodulepath"
```

这将 `site-modules/` 添加到 Puppet 查找模块的路径列表中。

如第七章，*精通模块*中详细说明的那样，我们将使用 `r10k` 和 Puppetfile 来管理所有第三方模块。因此，演示仓库中没有 `modules/` 目录：`r10k` 会在安装所需模块时创建此目录。

这是我们初始仓库所需模块列表的 Puppetfile。当然，随着你根据自己的需求调整仓库，你会将更多的模块添加到这个列表中（`Puppetfile`）：

```
forge "http://forge.puppetlabs.com"

# Modules from the Puppet Forge
mod 'puppetlabs/accounts', '1.1.0'
mod 'puppetlabs/ntp', '6.2.0'
mod 'puppetlabs/stdlib', '4.19.0'
mod 'saz/sudo', '4.2.0'
mod 'saz/timezone', '3.5.0'
mod 'stm/debconf', '2.0.0'
```

我们将在接下来的章节中看到这些模块的使用方式。

定期使用 `generate-puppetfile` 工具自动更新你的模块版本和依赖项（更多内容请参见第七章，*精通模块*）。在仓库目录中运行以下命令：

```
generate-puppetfile -p Puppetfile

```

将输出复制并粘贴回你的 Puppetfile，替换现有的 `mod` 语句。

## 类

正如你可能还记得的，第八章，*类、角色和配置文件*中，我们使用 Hiera 数据来确定哪些类和资源应该应用到节点。常见的类列在 `common.yaml` 中，`demo` 节点有一个专门的数据文件，其中包括 `role::demo` 类。这些类通过 `manifests/site.pp` 中的以下行包含：

```
include(lookup('classes', Array[String], 'unique'))
```

## 角色

**角色类**通过名称标识节点的功能，并定义应包含哪些配置文件类（更多内容请参见第八章，*类、角色和配置文件*）。

常见做法是将角色类保存在一个 `role` 模块中，由于这是一个站点特定的模块，它被归档在 `site-modules/` 下。

这是 `role::demo` 角色清单（`site-modules/role/manifests/demo.pp`）：

```
# Be the demo node
class role::demo {
  include profile::common
}
```

## 配置文件

**配置文件类**通过名称标识角色所需的某个特定软件或功能，并声明管理该功能所需的资源（有关配置文件的更详细说明，请参见第八章，*类、角色和配置文件*）。

通常，所有节点都会有一些公共的配置文件：例如我们的用户账户，还有一些其他的文件。将这些配置文件保存在`common.yaml`的 Hiera 数据文件中是合理的，这样所有节点都会包含这些配置文件。

这里是`common.yaml`中包含的类：

```
classes:
- profile::ntp
- profile::puppet
- profile::ssh
- profile::sudoers
- profile::timezone
- profile::users
```

我们将在接下来的章节中查看这些配置文件的作用。

### 提示

在 Hiera 数据中，类按字母顺序列出：当你包含许多类时，这非常有帮助，能够让你更容易查看某个类是否已经在列表中。当你添加新的类时，请确保保持列表的字母顺序。

## 用户和访问控制

`puppetlabs/accounts`模块提供了一种标准方法，通过`accounts::user`类来处理用户账户。因此，我们将使用这个模块在`profile::users`类中管理我们的用户。

### 注意

如果你更倾向于直接在 Puppet 中使用`user`和`ssh_authorized_key`资源来管理用户账户，请参阅第四章，*理解 Puppet 资源*，了解更多信息。

当然，你也可以直接在 Puppet 清单中列出所需的用户作为字面量资源。但我们将采用第六章，*使用 Hiera 管理数据*中描述的数据驱动方法，通过 Hiera 数据定义用户。

这是数据结构的样子（`data/common.yaml`）：

```
users:
  'john':
    comment: 'John Arundel'
    uid: '1010'
    sshkeys:
      - 'ssh-rsa AAAA ...'
  'bridget':
    comment: 'Bridget X. Zample'
    uid: '1011'
    sshkeys:
      - 'ssh-rsa AAAA ...'
```

这是`users`配置文件中的代码，用于读取数据并创建相应的`accounts::user`资源（`site-modules/profile/manifests/users.pp`）：

```
# Set up users
class profile::users {
  lookup('users', Hash, 'hash').each | String $username, Hash $attrs | {
    accounts::user { $username:
      * => $attrs,
    }
  }
}
```

正如你所见，我们通过调用`lookup()`将所有用户数据提取到一个`$users`哈希表中。然后我们遍历这个哈希表，为每个用户声明一个`accounts::user`资源，其属性从哈希表数据中加载。

请注意，使用`accounts::user`资源时，`sshkeys`属性必须包含一个用户的授权 SSH 公钥数组。

## SSH 配置

限制 SSH 登录仅允许特定用户使用是一个良好的安全实践，使用`/etc/ssh/sshd_config`中的`AllowUsers`指令。我们在第九章，*使用模板管理文件*中使用了一个 Puppet 模板来构建此配置文件。在那个例子中，我们从 Hiera 获取了允许的用户列表，并且在这里我们也将使用相同的方法。

这是`sshd_config`文件的模板（`site-modules/profile/templates/ssh/sshd_config.epp`）：

```
<%- | Array[String] $allow_users | -%>
# File is managed by Puppet

AcceptEnv LANG LC_*
ChallengeResponseAuthentication no
GSSAPIAuthentication no
PermitRootLogin no
PrintMotd no
Subsystem sftp internal-sftp
AllowUsers <%= join($allow_users, ' ') %>
UseDNS no
UsePAM yes
X11Forwarding yes
```

我们声明模板接受一个`$allow_users`参数，该参数是一个字符串数组。由于`sshd_config`中的`AllowUsers`参数需要一个用空格分隔的用户列表，我们使用标准库中的`join()`函数从 Puppet 数组中创建这个列表（参见第七章，*掌握模块*，了解更多有关此及其他标准库函数的信息）。

这里是相关的 Hiera 数据（`data/common.yaml`）：

```
allow_users:
  - 'john'
  - 'bridget'
  - 'ubuntu'
```

### 提示

我们本可以仅从`$users`哈希构建这个列表，`$users`哈希包含了所有已知的用户，但我们不一定希望列表中的每个人都能够登录到每个节点。相反，我们可能需要允许一些由 Puppet 未管理的帐户登录。例如，`ubuntu`账户是 Vagrant 所需的，以便正确管理虚拟机。如果你不使用 Vagrant 盒子，可以从这个列表中移除`ubuntu`用户。

以下是读取 Hiera 数据并填充模板的代码（`site-modules/profile/manifests/ssh.pp`）：

```
# Manage sshd config
class profile::ssh {
  ensure_packages(['openssh-server'])

  file { '/etc/ssh/sshd_config':
    content => epp('profile/ssh/sshd_config.epp', {
      'allow_users' => lookup('allow_users', Array[String], 
        'unique'),
    }),
    notify  => Service['ssh'],
  }

  service { 'ssh':
    ensure => running,
    enable => true,
  }
}
```

这是一个包-文件-服务模式，你可能还记得来自第二章，*创建你的第一个清单*。

首先，我们安装`openssh-server`包（通常它已经安装，但声明这个包仍然是好的做法，因为我们依赖它来完成接下来的操作）。

接下来，我们使用模板管理`/etc/ssh/sshd_config`文件，并通过调用`lookup('allow_users', Array[String], 'unique')`从 Hiera 数据中填充该文件。每当文件发生更改时，它都会通知`ssh`服务。

最后，我们声明`ssh`服务，并指定它应该在启动时运行并启用。

## Sudoers 配置

`sudo`命令是控制**用户权限**的标准 Unix 机制。它通常用于允许普通用户以`root`用户的权限运行命令。

### 提示

使用`sudo`优于允许用户以`root`身份登录并运行 shell，而且`sudo`还会审计并记录哪个用户运行了哪些命令。你还可以指定非常细粒度的权限，例如允许用户以`root`身份运行某个命令，但不允许运行其他命令。

管理`sudo`权限的最流行 Forge 模块是`saz/sudo`，我们将在这里使用它。以下是列出具有`sudo`访问权限的用户的 Hiera 数据（`data/common.yaml`）：

```
sudoers:
  - 'john'
  - 'bridget'
  - 'ubuntu'
```

### 提示

如果你不使用 Vagrant，可以从这个列表中移除`ubuntu`用户。

以下是读取数据的`profile`类（`site-modules/profile/manifests/sudoers.pp`）：

```
# Manage user privileges
class profile::sudoers {
  sudo::conf { 'secure_path':
    content  => 'Defaults      secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/puppetlabs/puppet/bin:/opt/puppetlabs/bin"',
    priority => 0,
  }
  $sudoers = lookup('sudoers', Array[String], 'unique', [])
  $sudoers.each | String $user | {
    sudo::conf { $user:
      content  => "${user} ALL=(ALL) NOPASSWD: ALL",
      priority => 10,
    }
  }
}
```

这允许我们以普通用户身份运行像`sudo puppet`这样的命令。这部分清单就是这么做的：

```
  sudo::conf { 'secure_path':
    content  => 'Defaults      secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/puppetlabs/puppet/bin:/opt/puppetlabs/bin"',
    priority => 0,
  }
```

`sudo::conf`资源由`saz/sudo`模块提供，它允许我们将任意的`sudoers`配置作为字符串编写：在这种情况下，设置`secure_path`变量。

配置文件的其余部分负责为每个在 Hiera 数组`sudoers`中列出的用户配置无密码`sudo`权限。像往常一样，我们从 Hiera 获取数组，然后使用`each`遍历它，为每个命名的用户创建`sudo::conf`资源。

## 时区和时钟同步

有一个方便的 Forge 模块可以用来管理服务器时区：`saz/timezone`。以下是我们的`timezone`配置文件，它使用该模块将所有节点设置为 UTC（`site-modules/profile/manifests/timezone.pp`）：

```
# Set the time zone for all nodes
class profile::timezone {
  class { 'timezone':
    timezone => 'Etc/UTC',
  }
}
```

### 提示

可能会让人很想将节点的时区设置为你自己的本地时区，而不是 UTC。然而，这种做法无法扩展。当你的节点分布在多个时区，或者分布在世界各地时，它们会处于不同的时区，这将导致你在尝试比较来自不同日志文件的时间戳时产生非常混乱的结果。始终将节点的时区设置为 UTC，这样你就不会感到困惑了（至少，不是关于这个问题）。

同样，我们希望确保所有节点的时钟不仅相互同步，而且与全球时间标准同步。我们将使用`puppetlabs/ntp`模块来实现这一点，这里是相关的配置文件（`site-modules/profile/manifests/ntp.pp`）：

```
# Synchronize with NTP
class profile::ntp {
  include ::ntp
}
```

实际上，NTP 不需要进行特殊配置（尽管如果你愿意，你可以指定一个时间服务器的列表，例如）。

## Puppet 配置

我们需要配置一个定期的 cron 任务，从 Git 仓库拉取更新并运行 Puppet 以应用更新后的清单。

`profile::puppet`类设置了这个配置（`site-modules/profile/manifests/puppet.pp`）：

```
# Set up Puppet config and cron run
class profile::puppet {
  service { ['puppet', 'mcollective', 'pxp-agent']:
    ensure => stopped, # Puppet runs from cron
    enable => false,
  }

  cron { 'run-puppet':
    ensure  => present,
    command => '/usr/local/bin/run-puppet',
    minute  => '*/10',
    hour    => '*',
  }

  file { '/usr/local/bin/run-puppet':
    source => 'puppet:///modules/profile/puppet/run-puppet.sh',
    mode   => '0755',
  }

  file { '/usr/local/bin/papply':
    source => 'puppet:///modules/profile/puppet/papply.sh',
    mode   => '0755',
  }
}
```

这个配置文件中有相当多的资源，让我们逐一查看它们。

首先，我们停止并禁用 Puppet 包启动的一些不需要的服务：

```
  service { ['puppet', 'mcollective', 'pxp-agent']:
    ensure => stopped, # Puppet runs from cron
    enable => false,
  }
```

接下来是定期执行 Git 更新和 Puppet 运行的 cron 任务。`run-puppet`脚本如下所示（`site-modules/profile/files/run-puppet.sh`）：

```
#!/bin/bash
cd /etc/puppetlabs/code/environments/production && git pull
/opt/puppetlabs/puppet/bin/r10k puppetfile install
/opt/puppetlabs/bin/puppet apply --environment production manifests/
```

这是运行该脚本的`cron`资源：

```
  cron { 'run-puppet':
    ensure  => present,
    command => '/usr/local/bin/run-puppet',
    minute  => '*/10',
    hour    => '*',
  }
```

这个任务被设置为每 10 分钟运行一次，但你可以根据需要进行调整。

这看起来很像你可能记得的`run-puppet`脚本，来自于第三章，*使用 Git 管理 Puppet 代码*。唯一的区别是额外的步骤来运行`r10k puppetfile install`（如果你在 Puppetfile 中添加了新的外部模块），以及向`puppet apply`添加了`--environment`开关。

`profile::puppet`中的下一个资源部署了一个名为`papply`的便利脚本，它能帮助你避免手动输入整个`puppet apply`命令（`site-modules/profile/files/papply.sh`）：

```
#!/bin/bash
environment=${PUPPET_ENV:-production}
/opt/puppetlabs/puppet/bin/r10k puppetfile install
/opt/puppetlabs/bin/puppet apply --environment ${environment} --strict=warning /etc/puppetlabs/code/environments/${environment}/manifests/ $*
```

只需从命令行运行`papply`就会立即应用 Puppet，而不需要拉取任何 Git 更改。

如果你想从不同的环境中测试 Puppet 的更改（例如，如果你在`/etc/puppetlabs/code/environments/staging`中检出了一个 staging 分支），你可以通过以下方式使用`PUPPET_ENV`变量来控制这一点：

```
PUPPET_ENV=staging papply

```

请注意，`papply`会将其命令行参数传递给 Puppet（使用`$*`），因此你可以添加任何`puppet apply`命令支持的参数：

```
papply --noop --show_diff

```

### 小贴士

我们还为 `puppet apply` 命令提供了 `--strict=warning` 标志，这将导致 Puppet 在遇到潜在问题代码时发出警告（例如引用了尚未定义的变量）。如果你希望 Puppet 更加严格，可以改为设置 `--strict=error`，这将阻止在所有问题解决之前应用清单。

# 引导过程

为了使用示例仓库为 Puppet 管理准备新节点，我们需要做一些事情：

+   安装 Puppet

+   克隆 Git 仓库

+   第一次运行 Puppet

在第三章，*使用 Git 管理你的 Puppet 代码*，我们手动执行了这些步骤，但示例仓库自动化了这个过程（通常称为**引导**）。这里是引导脚本（`scripts/bootstrap.sh`）：

```
#!/bin/bash
PUPPET_REPO=$1
HOSTNAME=$2
BRANCH=$3
if [ "$#" -ne 3 ]; then
  echo "Usage: $0 PUPPET_REPO HOSTNAME BRANCH"
  exit 1
fi
hostname ${HOSTNAME}
echo ${HOSTNAME} >/etc/hostname
source /etc/lsb-release
apt-key adv --fetch-keys http://apt.puppetlabs.com/DEB-GPG-KEY-puppet
wget http://apt.puppetlabs.com/puppetlabs-release-${DISTRIB_CODENAME}.deb
dpkg -i puppetlabs-release-${DISTRIB_CODENAME}.deb
apt-get update
apt-get -y install git puppet-agent
cd /etc/puppetlabs/code/environments
mv production production.orig
git clone ${PUPPET_REPO} production
cd production
git checkout ${BRANCH}
/opt/puppetlabs/puppet/bin/gem install r10k --no-rdoc --no-ri
/opt/puppetlabs/puppet/bin/r10k puppetfile install --verbose
/opt/puppetlabs/bin/puppet apply --environment=production /etc/puppetlabs/code/environments/production/manifests/
```

它期望通过三个参数运行（稍后我们将看到如何做到这一点）：`PUPPET_REPO`，要克隆的 Puppet 仓库的 Git URL，`HOSTNAME`，目标节点的主机名，和 `BRANCH`，使用的 Puppet 仓库的分支。

首先，脚本设置指定的主机名：

```
hostname ${HOSTNAME}
echo ${HOSTNAME} >/etc/hostname
```

接下来，它查看 `/etc/lsb-release` 文件以找出已安装的 Ubuntu 版本。

### 提示

这个脚本是专门针对 Ubuntu 的，但如果需要，你可以轻松修改它以适配其他 Linux 发行版。

使用 `wget` 下载适当的 Puppet Labs APT 仓库软件包并安装。然后安装 `puppet-agent` 包以及 `git`：

```
source /etc/lsb-release
apt-key adv --fetch-keys http://apt.puppetlabs.com/DEB-GPG-KEY-puppet
wget http://apt.puppetlabs.com/puppetlabs-release-${DISTRIB_CODENAME}.deb
dpkg -i puppetlabs-release-${DISTRIB_CODENAME}.deb
apt-get update && apt-get -y install git puppet-agent
```

引导过程中的下一步是将 Git 仓库克隆到 Puppet 期望找到其清单的位置：

```
cd /etc/puppetlabs/code/environments
mv production production.orig
git clone ${PUPPET_REPO} production
cd production
git checkout ${BRANCH}
```

接下来，我们安装 `r10k`（在 Puppet 的 gem 环境中，使用 Puppet 特定的 `gem` 命令）并运行 `r10k puppetfile install`，以安装 `Puppetfile` 中列出的所有必要模块：

```
/opt/puppetlabs/puppet/bin/gem install r10k --no-rdoc --no-ri
/opt/puppetlabs/puppet/bin/r10k puppetfile install --verbose
```

现在我们可以第一次运行 Puppet，它将配置我们所需的其他所有内容：

```
/opt/puppetlabs/bin/puppet apply --environment=production /etc/puppetlabs/code/environments/production/manifests/
```

当然，为了在目标节点上运行这个脚本，我们必须先将它复制到目标节点。这一步通过 `puppify` 脚本（`scripts/puppify`）完成：

```
#!/bin/bash
PUPPET_REPO=https://github.com/bitfield/control-repo-3.git
IDENTITY="-i /Users/john/.ssh/pbg.pem"
if [ "$#" -lt 2 ]; then
  cat <<USAGE
Usage: $0 TARGET HOSTNAME [BRANCH]
Install Puppet on the node TARGET (IP address or DNS name) and run
the bootstrap process. Set the hostname to HOSTNAME, and optionally use
the control repo branch BRANCH.
USAGE
  exit 1
fi
TARGET=$1
HOSTNAME=${2}
BRANCH=${3:-production}
OPTIONS="-oStrictHostKeyChecking=no"
echo -n "Copying bootstrap script... "
scp ${IDENTITY} ${OPTIONS} $(dirname $0)/bootstrap.sh ubuntu@${TARGET}:/tmp
echo "done."
echo -n "Bootstrapping... "
ssh ${IDENTITY} ${OPTIONS} ubuntu@${TARGET} "sudo bash /tmp/bootstrap.sh ${PUPPET_REPO} ${HOSTNAME} ${BRANCH}"
echo "done."
```

首先，脚本设置要克隆的 Git 仓库的 URL（在调整示例仓库以供自己使用时，你需要更改为自己的 URL）：

```
PUPPET_REPO=https://github.com/bitfield/control-repo-3.git
```

接下来，我们指定用于通过 SSH 连接目标节点的密钥文件（再次，修改为你自己的密钥）：

```
IDENTITY="-i /Users/john/.ssh/pbg.pem"
```

在显示使用信息并处理命令行参数之后，脚本继续将 `bootstrap.sh` 文件复制到目标节点：

```
scp ${IDENTITY} ${OPTIONS} $(dirname $0)/bootstrap.sh ubuntu@${TARGET}:/tmp
```

最后的步骤是，在节点上运行引导脚本，并传递所需的命令行参数：

```
ssh ${IDENTITY} ${OPTIONS} ubuntu@${TARGET} "sudo bash /tmp/bootstrap.sh ${PUPPET_REPO} ${HOSTNAME} ${BRANCH}"
```

# 将仓库调整为适合自己的使用

你需要更改示例仓库中的一些数据和设置，以便能自己使用。为了帮助你入门，这里有一个表格，列出了需要更改的文件以及需要提供的信息，详细解释将在后续章节中提供：

| 文件 | 需要更改的内容 |
| --- | --- |
| `data/common.yaml` | `users`: 所有节点通用的用户和 SSH 密钥`allow_users`: 允许登录所有节点的用户`sudoers`: 允许在所有节点上使用`sudo`的用户`classes`: 所有节点包含的类 |
| `data/nodes/[NODE NAME].yaml` | `users`: 仅在此节点上存在的用户和 SSH 密钥`allow_users`: 仅允许登录此节点的用户`sudoers`: 仅允许在此节点上使用`sudo`的用户`classes`: 仅此节点包含的类 |
| `site-modules/role/manifests/` | 节点的角色类（在每个类中包括`profile::common`） |
| `scripts/puppify` | `PUPPET_REPO`: 你的 Puppet 仓库的 Git URL`IDENTITY`: 初始引导节点时使用的 SSH 密钥路径（如果需要的话） |

## 配置用户

正如我们在本章前面所看到的，Puppet 管理的用户账户是通过 Hiera 数据进行配置的。编辑`data/common.yaml`文件，文件内容如下所示：

```
users:
  'john':
    comment: 'John Arundel'
    uid: '1010'
    sshkeys:
      - 'ssh-rsa AAAA... john@susie'
...
```

将现有的用户替换为你希望在节点上创建的用户账户（最开始可能只有一个账户，用于你自己）。将你希望与这些账户一起使用的任何 SSH 密钥添加到`sshkeys`数组中。

每个节点上允许的用户列表由`allow_users`数组控制。将列出的用户替换为你自己的用户。

拥有`sudo`权限的用户列表由`sudoers`数组控制。将那里列出的用户替换为你希望拥有 root 权限的用户。

## 添加每个节点的数据文件和角色类

每个节点的 Hiera 数据，包括类，保存在`data/nodes/`目录下。当你添加一个新节点时，为其添加一个名为`data/nodes/NODE_NAME.yaml`的数据文件，将`NODE_NAME`替换为该节点的主机名。

包括适合该节点的角色类（详见第八章，*类、角色和配置文件*，了解更多信息）。如果你在每个节点的文件中没有指定任何类，节点将仅包括`common.yaml`中列出的类。这将足以通过你的 SSH 账户和密钥设置节点，并验证引导过程是否正常工作。之后，你可以开始将角色类添加到每个节点的文件中，以完成实际的工作。

将你的角色类添加到`site-modules/role/manifests/`目录中，类似`role::demo`。

如果某些用户只需要在特定节点上存在，而不希望它们出现在所有节点上，请将它们列在每个节点的数据文件中的`users`下。如果这些用户需要通过 SSH 登录，也请将它们添加到`allow_users`中。同样，如果你只希望某个用户在此节点上拥有`sudo`权限，请将他们列在每个节点数据文件中的`sudoers`下。

## 修改引导凭据

在`scripts/puppify`文件中，编辑`PUPPET_REPO`设置为你自己 Git 仓库的 URL。如果你需要 SSH 密钥来连接目标节点（例如，如果你使用的是 Amazon EC2，此时你会有一个`.pem`文件，其中包含你从 AWS 控制台下载的密钥），请将其位置添加到`IDENTITY`变量中。

# 启动新节点

如果你希望在 Vagrant 盒子中尝试演示仓库，可以在仓库目录中找到适用的 Vagrantfile。

### 注意

如果你没有安装 Vagrant，请先按照第一章中*安装 VirtualBox 和 Vagrant*部分的说明进行操作，完成 Puppet 的初步设置。

## 启动 Vagrant 虚拟机

在仓库目录中运行以下命令以启动你的 Vagrant 虚拟机：

```
scripts/start_vagrant.sh

```

## 启动物理或云节点

或者，你也可以使用仓库来引导物理或云节点。你只需要目标节点的 IP 地址或 DNS 名称。

在 Puppet 仓库中运行以下命令，将`TARGET_SERVER`替换为目标节点的地址或名称，将`HOSTNAME`替换为你希望设置的主机名（例如`demo`）：

```
scripts/puppify TARGET_SERVER HOSTNAME

```

你将看到一些与复制引导脚本、安装 Puppet 包、克隆仓库、安装 Forge 模块以及首次运行 Puppet 相关的输出。完成后，节点应该已经准备好，你可以尝试使用自己的 SSH 账户登录。

## 使用其他发行版和提供者

演示仓库中包含的`puppify`和`bootstrap`脚本适用于 Amazon EC2 上的 Ubuntu 节点，但你可以修改它们以适用于任何 Linux 发行版或服务器提供者。

例如，如果你使用的是**Google 计算引擎**（**GCE**）实例，可以编辑`puppify`脚本，将`ssh`命令替换为`gcloud compute ssh`。如果你使用的是 Digital Ocean 的 Droplet，可以在通过 Web 界面配置时将你的 SSH 密钥添加到 Droplet，并修改`puppify`脚本以使用`root`用户而不是`ubuntu`用户登录。

如果你在多个不同平台上管理节点，可能会发现为每个平台使用定制的`puppify`脚本更为方便，命名方式可以是（例如）`puppify_ec2`、`puppify_linode`等。

如果你不是使用 Ubuntu 或 Debian，可能需要对`bootstrap.sh`脚本做一些修改。例如，如果你使用的是 Red Hat Linux 或 CentOS，你需要让脚本通过`yum`而不是`apt`安装 Puppet。同样，如果你管理多个操作系统节点，可能需要为每个操作系统维护一个自定义的引导脚本。

# 总结

在本章中，我们介绍了示例控制仓库，并演示了如何下载它。我们解释了控制仓库模式，以及它如何与`r10k`和 Puppetfile 一起使用来管理第三方和本地模块。我们还学习了如何分叉仓库并从上游拉取更新。

我们查看了示例角色和配置文件类，了解了 Puppet 如何使用 Hiera 数据来配置用户帐户、SSH 密钥、允许的用户和`sudoers`权限。我们还介绍了使用 Forge 模块来管理时区设置和 NTP 同步。此外，我们探索了控制自动 Puppet 更新和运行所需的资源和脚本。

演示仓库包含了启动脚本，帮助你将新配置的节点纳入 Puppet 控制范围，我们已经详细研究了这些脚本是如何工作的。

最后，我们已经学会了如何调整演示仓库以适应你自己的网站，概述了如何添加自己的用户和访问设置，如何添加自己的常用配置文件和每个节点的角色类。我们还看到如何将自己的信息插入到启动脚本中，以及如何使用它们启动一个新节点。

# 开始

希望你喜欢这本书，并从中学到了一些有用的东西；我在写这本书时也学到了很多。然而，从书本中你能学到的东西是有限的。正如普鲁斯特所写，“我们并非接受智慧，而是必须在一场无人能代替的旅程中自己发现它。”

有一个朋友指引我们走上正确的道路并在旁边给予一些精神支持是好的，但最终我们还需要自己独立前行。我希望这本书能成为你旅程的开始，而不是结束。

世界著名的古典吉他演奏家约翰·威廉姆斯曾被问到学会弹吉他花了多长时间。他回答：“我还在学习。”

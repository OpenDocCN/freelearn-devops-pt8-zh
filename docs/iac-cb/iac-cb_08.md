# 第八章：使用 Chef 和 Puppet 维护系统

本章我们将介绍以下几种配方：

+   使用计划的收敛性保持一致的系统

+   创建环境

+   使用 Chef 加密数据包和 Puppet 中的 Hiera-eyaml

+   使用 Chef Vault 加密

+   使用 Ohai 访问和操作系统信息

+   自动化应用部署（以 WordPress 为例）

+   使用 TDD 工作流

+   为最坏情况做好准备——训练重建工作系统

# 引言

我们之前已经看到如何用代码自动化系统，以及如何正确测试这些代码。现在我们已经准备好迎接正式应用；有一整套特性、约束和目标需要正确设置。我们将希望将开发、测试和生产环境隔离开。我们需要确保基础设施代码在没有干预的情况下保持一致。安全性和机密性将开始成为一个问题，也许那些密码和密钥根本不应该以明文存储。几个月后，我们的自动化基础设施将扩展到大量的管理节点，且必须确保系统按照配置文件一致地运行——我们将需要收集和处理系统信息。最终，我们将直接从 Chef 部署 Web 应用。为了保持代码质量，即便我们的代码库越来越复杂，我们将采用**测试驱动开发**（**TDD**）方法来优化工作流。最后，我们将确保随时准备重新部署任何基础设施部分（想象一下灾难发生时）。

所有的配方都是基于 Chef 的。然而，在可能的情况下，我们将尝试展示如何使用 Chef 的直接替代品——Puppet 来实现类似的功能。

# 使用计划的收敛性保持一致的系统

一旦初步部署并配置好系统，几乎无法想象仍然通过登录每台主机并启动`chef-client`命令来手动更新系统。使用 Chef 维护的系统可以在预定时间通过`chef-client`守护进程或 cron 作业进行收敛。我们将详细讲解这两种选项。

## 准备工作

要逐步执行此配方，你需要：

+   在工作站上安装一个可用的 Chef DK

+   在工作站上安装一个可用的 Vagrant

+   Chef 代码（可选）来自第六章，*使用 Chef 和 Puppet 管理服务器的基础知识*，第七章，*使用 Chef 和 Puppet 编写更好的基础设施代码并进行测试*，或任何自定义的 Chef 代码

## 如何操作…

我们建议你创建一个与其他 cookbook 不同的 cookbook，专门用于配置底层主机。我们称这个 cookbook 为`common`：

```
$ cd chef-repo/cookbooks
$ chef generate cookbook common
$ cd common

```

要配置 Chef 客户端，有一个官方食谱名为`chef-client`。让我们在`Berksfile`中声明对它的`cookbook`依赖：

```
cookbook 'chef-client', '~> 7.0.0'
```

在`common/metadata.rb`文件中，添加依赖项：

```
depends 'chef-client'
```

### 将 Chef 客户端作为守护进程使用

文档告诉我们，包含默认食谱将自动检测主机平台，并相应配置`chef-client`以作为守护进程运行。以下是启用`chef-client`的步骤：

1.  将以下内容添加到`recipes/default.rb`中：

    ```
    include_recipe 'chef-client'
    ```

1.  使用 Berkshelf 安装依赖项：

    ```
    $ berks install

    ```

1.  现在上传`common`食谱及其所有依赖项：

    ```
    $ berks upload

    ```

1.  将`common`食谱添加到主机的`run-list`中：

    ```
    $ knife node run_list add vagrant common

    ```

1.  在目标主机上，最后一次启动 Chef 客户端以使其作为服务部署：

    ```
    $ chef-client 
    Recipe: chef-client::systemd_service
    [...]
     * service[chef-client] action enable
     - enable service service[chef-client]
     * service[chef-client] action start
     - start service service[chef-client]
     * service[chef-client] action restart
     - restart service service[chef-client]

    ```

1.  日志看起来相当乐观，但我们还是再检查一下守护进程是否真的在主机上运行：

    ```
    $ systemctl status chef-client
    
     chef-client.service - Chef Client daemon
     Loaded: loaded (/etc/systemd/system/chef-client.service; enabled; vendor preset: disabled)
     Active: active (running) since Mon 2016-11-07 01:35:05 UTC; 57s ago
     Main PID: 12943 (chef-client)
     CGroup: /system.slice/chef-client.service
     └─12943 /opt/chef/embedded/bin/ruby /usr/bin/chef-client -c /etc/chef/client.rb -i 1800 -s 300

    ```

`chef-client`服务已经启用并在运行！

### 调整收敛间隔时间

足够有趣的是，我们看到间隔是每 1800 秒（30 分钟）引入的。如果我们想要不同的收敛间隔，比如每 900 秒（15 分钟）呢？让我们来修改`default.rb`食谱：

```
node.override['chef_client']['interval'] = '900'
include_recipe 'chef-client'
```

在`metadata.rb`中提升版本，上传新版本，等待新的`chef-client`执行，或者自己启动它以节省一些时间。`systemd`单元已更新：

```
$ systemctl status chef-client

 chef-client.service - Chef Client daemon
[...]
 └─13316 /opt/chef/embedded/bin/ruby /usr/bin/chef-client -c /etc/chef/client.rb -i 900 -s 300

```

现在我们的系统已经配置为每 15 分钟收敛一次，变动控制在 300 秒以内。

### 注意

我们强烈建议在每次新的主机部署过程中都包含这个`common`食谱，这样它们都能被自动配置为按照预定的间隔进行收敛。

### 将 Chef 客户端作为 cron 作业运行

在某些情况下，我们可能不希望将 Chef 客户端作为守护进程运行（例如内存或安全需求）。幸运的是，我们可以简单地退回到基于 cron 的简单方法。让我们修改默认的`recipe.rb`食谱以匹配这一点：

```
node.override['chef_client']['init_style'] = 'none'
include_recipe 'chef-client::cron'
```

使用 Berkshelf 上传这个食谱版本，并在目标主机上执行`chef-client`。查看 root 的`crontab`文件：

```
$ sudo crontab -l
# Chef Name: chef-client
0 0,4,8,12,16,20 * * * /bin/sleep 69;  /usr/bin/chef-client > /dev/null 2>&1

```

默认情况下，它每四小时执行一次`chef-client`，并在此情况下延迟`69`秒，以避免每个节点同时向 Chef 服务器发送请求。

### 调整 Chef 的 cron 作业

如果每四小时收敛一次还不够，你希望像我们在`default.rb`食谱中做的那样每 15 分钟收敛一次，以下是你需要做的：

```
node.override['chef_client']['init_style'] = 'none'
node.override['chef_client']['cron']['minute'] = '*/15'
node.override['chef_client']['cron']['hour'] = '*'

include_recipe 'chef-client::cron'
```

上传食谱并运行`chef-client`（或等待下一次计划的运行）。间隔现在已设置为每 15 分钟运行一次：

```
$ sudo crontab -l
# Chef Name: chef-client
*/15 * * * * /bin/sleep 69;  /usr/bin/chef-client > /dev/null 2>&1

```

## 还有更多…

在 Puppet 中，代理也可以作为服务或 cron 作业运行。

以下命令用于启用服务模式：

```
# puppet resource service puppet ensure=running enable=true

```

在此模式下，Puppet 代理将默认每 30 分钟应用一次配置。这个延迟可以在`/etc/puppetlabs/puppet.conf`中更改。以下是将延迟减少到五分钟的示例：

```
[agent]
  runinterval = 5m
```

要将 Puppet 代理作为 cron 作业运行，我们需要声明一个 Puppet cron 资源，如下所示：

```
# puppet resource cron puppet-agent ensure=present user=root minute=0 command='/opt/puppet/bin/puppet agent --onetime --no-daemonize --splay --splaylimit 60' 

```

生成的`crontab`文件是：

```
$ sudo crontab -l
0 * * * * /opt/puppet/bin/puppet agent --onetime --no-daemonize --splay --splaylimit 60

```

在本示例中，Puppet 代理将每小时运行一次。`splay`选项用于在运行之前引入一个随机延迟，该延迟不能超过 60 分钟（即`splaylimit`选项的值）。这对于有很多节点连接到同一个 Puppet 服务器的情况特别有用，可以将 Puppet 代理的请求分散到不同的时间点。

当然，如果您的基础设施中有许多节点，您应该创建一个包含这些 Puppet 资源的模块，并为每个节点包含该模块。基于我们之前的 Vagrant LAMP 环境设置，让我们创建一个本地模块，并创建一个文件`module/baseconfig/manifests/init.pp`，其内容为：

```
# @param agentmode Agent type: service or cron. If anything else, agent will be disabled. Default value: service
class baseconfig (
  $agentmode='service'
) {
  case $agentmode {
    'service': {
      $ensureservice='running';
      $enableservice=true;
      $ensurecron='absent'
    }
    'cron': {
      $ensureservice='stopped';
      $enableservice=false;
      $ensurecron='present'
    }
    default: {
      $ensureservice='stopped';
      $enableservice=false;
      $ensurecron='absent'
    }
  }
  service {'puppet':
    ensure => $ensureservice,
    enable => $enableservice,
  }
  cron {'puppet-agent':
    ensure  => $ensurecron,
    user    => root,
    minute  => 0,
    command => '/opt/puppet/bin/puppet agent --onetime --no-daemonize --splay --splaylimit 60',
  }
}
```

现在我们可以从主清单中定义所需的模式：

```
node 'web.pomes.pro' {
...
  class { 'baseconfig':
    agentmode => 'cron';
  }
...
}
```

如果将来在`service`与`cron`之间发生任何更改，我们的`baseconfig`模块将删除之前模式的配置。

## 另见

+   `chef-client`菜谱源，参考[`github.com/chef-coo`](https://github.com/chef-coo)[kbooks/chef-client](http://kbooks/chef-client)

+   Puppet 的 cron 资源，参考[`docs.puppet.com/puppet/4.8/types/cron.html`](https://docs.puppet.com/puppet/4.8/types/cron.html)

# 创建环境

一个经典的组织至少有两个运行基础设施的环境：开发环境和生产环境。通常会看到许多不同的环境，如暂存环境、测试环境、Alpha 或 Beta 环境。组织可以根据自身需求来建模基础设施，复杂性也可能迅速增长。好消息是，Chef 在将这一模型映射到基础设施时提供了很大帮助。有一组信息会在两个不同的环境中有所不同，例如菜谱版本或属性，而 Chef 尽可能简化了管理这些环境的过程。默认情况下，未设置环境的节点将运行在`_default`环境中。

在本节中，我们将介绍如何创建不同的环境，如何将节点（包括现有节点和新节点）设置到专用环境中，如何设置菜谱约束，最后如何在每个环境中覆盖属性。

## 准备工作

要逐步执行此配方，您需要：

+   工作站上已安装的 Chef DK

+   工作站上已安装的 Vagrant

+   Chef 代码（可选）来自第六章, *使用 Chef 和 Puppet 管理服务器的基础*，第七章, *用 Chef 和 Puppet 编写和测试更好的基础设施代码*，或任何自定义 Chef 代码

## 如何操作……

Chef 环境存储在名为`environments`的文件夹中，该文件夹位于`chef-repo`的根目录。如果该文件夹不存在，请创建它：

```
$ mkdir environments

```

### 创建生产环境

要创建生产环境，请按照以下步骤操作：

1.  让我们从创建一个名为`production.rb`的`production`环境开始：

    ```
    name 'production'
    description 'The production environment'
    ```

1.  这是最简单的环境，它什么也不做。将其上传到 Chef 服务器：

    ```
    $ knife environment from file environments/production.rb
    Updated Environment production

    ```

1.  列出可用的远程环境：

    ```
    $ knife environment list
    _default
    production

    ```

我们看到有两个可用环境：`production` 和 `_default`。

### 将环境设置为节点

要将已存在的节点设置为新的 `production` 环境，请执行以下命令：

```
$ knife node environment set my_node_name production
my_node_name:
 chef_environment: production

```

### 使用环境引导节点

如果我们正在使用 `knife bootstrap` 命令引导一个节点，我们可以从一开始就使用所需的环境（使用名为 `vagrant` 的用户，如前面的示例所示）：

```
$ knife bootstrap a.b.c.d -N vagrant -x vagrant --sudo --environment production --run-list 'recipe[mysite]'

```

### 为环境修正配方版本

假设我们的生产系统运行的是版本为 0.3.1 的稳定 `mysite` 配方，但我们希望在开发基础设施中尝试同一配方的 0.4.0 版本新特性。由于每个配方版本可以共存，因此每个环境可以调用其自己的版本。`production.rb` 文件将包含以下内容，用于 `production` 环境：

```
cookbook_versions  'mysite' => '= 0.3.1'
```

`development.rb` 文件将包含以下内容，用于开发环境：

```
cookbook_versions  'mysite' => '= 0.4.0'
```

一个 Chef 环境文件可能包含多个配方约束，如下所示：

```
cookbook_versions: {
    'mysite': '= 0.4.0',
    'apache': '= 0.6.0'
}
```

### 为环境覆盖属性

每个环境可以覆盖任何值，在 Chef 中，这是覆盖的最高级别。没有其他内容可以覆盖为环境设置的值。所以，如果我们仅想将 `sitename` 属性的值覆盖为 `production.rb`，它将像这样：

```
override_attributes 'sitename' => 'mysite_production'
```

### 从配方中访问环境

节点的环境可以通过 `node.chef_environment` 属性从任何配方中访问。

所以，如果我们的目标是创建一个显示节点正在运行的环境的文件，我们需要创建一个像这样的模板：

```
Running in <%= @node.chef_environment %> mode.
```

## 还有更多内容...

使用 Puppet 时，环境位于 Puppet 服务器上的不同目录中。你可能在 第六章，*管理 Chef 和 Puppet 服务器的基础* 中注意到这一点；我们在 `/etc/puppetlabs/code/environments/production` 目录中创建了代码。

这是因为默认的 Puppet 环境是 `production`。其他环境，例如 `test`，应该在 `/etc/puppetlabs/code/environments/` 下创建。

### 在 Puppet 服务器上手动创建环境

我们从 第六章，*管理 Chef 和 Puppet 服务器的基础*，开始，尝试创建一个新的环境，命名为 `test`。在 Puppet 服务器上，我们只需要执行如下操作：

```
$ sudo -s
# cd /etc/puppetlabs/code/environments/
# cp -a production test

```

### 节点环境选择

在节点端，使用的环境可以通过 `--environment` 控制：

```
# puppet agent --test --environment test

```

若要将此环境设置为默认环境，而不使用 `--environment`，我们可以在 `/etc/puppetlabs/puppet/puppet.conf` 中配置如下：

```
[agent]
environment = test
```

### 从清单中获取环境

对于 Chef，我们可以从任何清单中获取正在运行的环境名称。通过使用由 Puppet 服务器设置的`$environment`变量来实现。

为了说明这一点，让我们修改 `index.php` 文件（`manifests/site.pp`），分别在`production`和`test`中进行修改：

```
file {"${docroot}/index.php":
    ensure  => present,
    owner   => 'www-data',
    group   => 'www-data',
    mode    => '0644',
    content => "<?php echo \"Running from ${environment}\" ?>"
}
```

我们现在可以在`test`和`production`之间切换并查看更改：

```
# puppet agent --test
Info: Using configured environment 'production' 
…
@@ -1 +1 @@
-<?php echo "Running from test" ?>
+<?php echo "Running from production" ?>
…
# puppet agent --test --environment test
Info: Using configured environment 'test'
@@ -1 +1 @@
-<?php echo "Running from production" ?>
+<?php echo "Running from test" ?>
```

### 动态方式 – r10k

我们直接在 Puppet 主机中编辑了环境和代码，这是不推荐的。幸运的是，r10k（我们在第六章中已经使用过，用于安装模块）可以用来从 Git 仓库创建环境。每个 Git 仓库中的分支将被检出到一个独立的目录，并作为一个环境提供。这一特性是动态的：每次向 Git 仓库添加新分支，r10k 都会部署它。

让我们从工作站尝试一下。到目前为止，我们的 Vagrant 设置中共享文件夹的工作是将相对目录`puppetcode`映射到`/etc/puppetlabs/code/environments/production`路径上，位于`puppet.pomes.pro`箱中。我们即将使用多个环境，因此需要将映射更改为`/etc/puppetlabs/code/`。

我们需要一个包含两个分支（`production`和`test`）的 Git 仓库，里面包含所有先前的代码。一个示例可以在 [`github.com/ppomes/r10k_sample.git`](https://github.com/ppomes/r10k_sample.git) 找到。

r10k 工具需要一个全局配置文件，必须在与 Vagrantfile 同一级别创建，文件内容如下：

```
:sources:
  :my-repos:
    remote: 'https://github.com/ppomes/r10k_sample.git'
    basedir: 'puppetcode/environments'
```

现在让我们使用 r10k：

```
$ r10k -c ./r10k.yaml deploy environment -p
$ ls -l puppetcode/environments/
total 0
drwxr-xr-x  8 ppomes  staff   272B 26 Nov 16:40 production/
drwxr-xr-x  8 ppomes  staff   272B 26 Nov 16:40 test/

```

Git 仓库中的两个分支都已部署，我们现在可以启动 Vagrant，操作我们的盒子和分支。

### 注意

r10k 工具还会处理每个分支中的`Puppetfile`文件，正如我们在第六章中所看到的，*使用 Chef 和 Puppet 管理服务器的基础知识*，并部署外部模块（如果有的话）。

## 另请参见

+   Chef 环境文档：[`docs.chef.io/environments.html`](http://://docs.chef.io/environments.html)

+   Puppet 的 r10k：[`github.com/puppetlabs/r10k`](https://github.com/puppetlabs/r10k)

# 使用 Chef 加密数据包和 Hiera-eyaml 与 Puppet

数据包中的一些信息可以安全地以明文存储在 Chef 服务器中，但在某些情况下，敏感信息加密后可能更为安全。公司可能不希望生产环境的 API 密钥、私钥或类似的敏感内容以明文形式存储在 Chef 服务器或第三方服务（如 GitHub）中。我们将看到如何在命令行中以及在 Chef 配方中对数据进行加密和解密。

## 准备工作

要执行此配方，你将需要：

+   工作站上安装好的 Chef DK

+   工作站上安装好的 Vagrant

+   来自第六章的 Chef 代码（可选），*使用 Chef 和 Puppet 管理服务器的基础知识*，第七章，*使用 Chef 和 Puppet 测试和编写更好的基础架构代码*，或者任何自定义 Chef 代码

## 如何操作……

我们的目标是创建一个包含`us-east-1`区域 AWS 凭证的配置文件，且不允许将凭证以明文存储在 Chef 服务器上。我们希望使用数据包，因为它可以加密：

1.  创建一个名为`aws`的数据包文件夹来存储`us-east-1`区域的凭证：

    ```
    $ mkdir data_bags/aws

    ```

1.  在 Chef 服务器上创建数据包，顺便说一下：

    ```
    $ knife data bag create aws
    Created data_bag[aws]

    ```

1.  在`aws`数据包文件夹内，创建一个示例文件`us-east-1.json`，其中包含凭证：

    ```
    {
      "id": "us-east-1",
      "aws_access_key": "AKIAJWTIBGE3NFDB4HOB",
      "aws_secret_key": "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"
    }
    ```

这是正常数据包的标准过程。如果我们现在按原样上传，它将不会被加密。

### 使用共享密钥加密数据包

使用加密数据包的解决方案是从我们的工作站发送加密数据。加密通过共享密钥完成，密钥可以是文件或字符串。我们使用字符串`s3cr3t`作为加密密钥（弱密钥）。为了简单地发送加密版本的数据包，我们可以使用`knife`命令的加密功能：

```
$ knife data bag from file --encrypt --secret s3cr3t aws us-east-1.json
Updated data_bag_item[aws::us-east-1]

```

如果我们请求数据但没有提供解密密钥，我们将从 Chef 服务器获取加密数据：

```
$ knife data bag show aws us-east-1
WARNING: Encrypted data bag detected, but no secret provided for decoding. Displaying encrypted data.
aws_access_key:
 cipher:         aes-256-cbc
 encrypted_data: RwbfsWgKk16sSCkMD38tXKGHmT1AHFGHRm/7fyzppye7wSS0kk19Zml0VuhQ
 XxxI

 iv:             iRRgrKfz6Ou2qdpYLkUA+w==

 version:        1
aws_secret_key:
 cipher:         aes-256-cbc
 encrypted_data: uSppKMYrRbEYn/njDYo3CIGC5tY+pptN1Z7LiARtNIU/zsllBNdSVENC1XwX
 QksifE6g00sdcHTGlHlVU0WJ0Q==

 iv:             ppjeAJcegZ9Yyn9rXgHRBQ==

 version:        1
id:             us-east-1

```

看起来我们得到了我们想要的：数据已经加密存储在 Chef 服务器上！

由于将未加密的数据包存储在版本控制系统（如 Git）中可能不是一个安全的做法，我们可以请求一个 JSON 格式的加密版本，如下所示，并将输出重定向到 JSON 文件以便存储：

```
$ knife data bag show aws us-east-1 -Fj
{
 "id": "us-east-1",
 "aws_access_key": {
 "encrypted_data": "RwbfsWgKk16sSCkMD38tXKGHmT1AHFGHRm/7fyzppye7wSS0kk19Zml0VuhQ\nXxxI\n",
 "iv": "iRRgrKfz6Ou2qdpYLkUA+w==\n",
 "version": 1,
 "cipher": "aes-256-cbc"
 },
 "aws_secret_key": {
 "encrypted_data": "uSppKMYrRbEYn/njDYo3CIGC5tY+pptN1Z7LiARtNIU/zsllBNdSVENC1XwX\nQksifE6g00sdcHTGlHlVU0WJ0Q==\n",
 "iv": "ppjeAJcegZ9Yyn9rXgHRBQ==\n",
 "version": 1,
 "cipher": "aes-256-cbc"
 }
}

```

这可能是你想存储到 Git 中的内容！

### 在 CLI 中访问加密数据包

要从 knife CLI 访问未加密的数据，过程与加密数据一样简单——将共享密钥作为参数传递：

```
$ knife data bag show aws us-east-1 --secret s3cr3t
Encrypted data bag detected, decrypting with provided secret.
aws_access_key: AKIAJWTIBGE3NFDB4HOB
aws_secret_key: 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
id:             us-east-1

```

现在我们可以访问我们的数据，但它是未加密的。

### 从食谱中使用加密数据包

现在数据已安全存储在 Chef 服务器上，我们如何从 Chef 食谱中访问它？假设我们的目标是创建一个名为`/etc/aws/credentials`的文件，里面包含 Chef 服务器上加密版本的未加密值。最终的文件应如下所示：

```
[region_name]
aws_access_key_id = the_access_key
aws_secret_access_key = the_secret_key
```

1.  为此，在`mysite`食谱中创建一个名为`aws`的新食谱：

    ```
    $ chef generate recipe aws

    ```

    ### 注

    不要忘记相应地增加食谱版本和环境限制。

1.  首先使用`directory`资源创建`/etc/aws`文件夹：

    ```
    directory "/etc/aws" do
      owner 'root'
      group 'root'
      mode '0755'
      action :create
    end
    ```

1.  这是我们目标文件`/etc/aws/credentials`的`templates/aws.erb` ERB 模板文件：

    ```
    [<%= @aws_region %>]
    aws_access_key_id = <%= @aws_access_key %>
    aws_secret_access_key = <%= @aws_secret_key %>
    ```

    我们看到模板期望`aws_region`、`aws_access_key`和`aws_secret_key`变量。让我们编写代码将这些值注入到`aws.rb`食谱中。首先，我们通过内联共享密钥`s3cr3t`访问加密的数据包项`us-east-1`，该数据包项位于`aws`数据包中：

    ```
    aws = Chef::EncryptedDataBagItem.load("aws", "us-east-1", 's3cr3t')
    ```

    ### 注意

    如果需要，我们可以将所有这些信息设置为属性。如果选择文件方法来共享密钥，最后一个参数将是解密数据的密钥文件路径。

1.  现在让我们创建模板，将解密后的凭据写入`/etc/aws/credentials`文件：

    ```
    template "/etc/aws/credentials" do
      source 'aws.erb'
      owner 'root'
      group 'root'
      mode '0600'
      variables(
        aws_region: aws['id'],
        aws_access_key: aws['aws_access_key'],
        aws_secret_key: aws['aws_secret_key']
      )
    end
    ```

我们完成了！现在，Chef 服务器已经安全地存储了加密数据。为了提高安全性，最好不要硬编码共享密钥——使用单独发送的密钥文件（但这会在部署系统中增加复杂性）。

## 还有更多…

在使用 Puppet 时，最好将凭据和站点信息存储在 Hiera 中，正如我们在第六章，*使用 Chef 和 Puppet 管理服务器的基础知识*中看到的那样。使用`hiera-eyaml`，可以加密敏感数据。使用我们之前的 LAMP 设置和 Vagrant，让我们尝试加密 MySQL 的 root 密码。

### 准备 Puppet 服务器。

我们需要为 Hiera 安装一个新的后端。我们还没有深入讨论 Hiera，现在是时候这样做了。Hiera 用于将数据存储在清单之外，并基于层级结构查找数据。安装 Puppet 服务器时会提供默认配置，位于`/etc/puppetlabs/puppet/hiera.yaml`：

```
---
:backends:
  - yaml
:hierarchy:
  - "nodes/%{::trusted.certname}"
  - common

:yaml:
:datadir:
```

在这里，定义了一个`yaml`后端，允许我们在环境的`hieradata`目录中使用`yaml`文件。然后，定义了一个层级结构。Puppet 将首先尝试查找与客户端证书名称（即 FQDN 节点）匹配的`yaml`文件，并将其放置在`nodes`子目录下。如果没有找到数据，Puppet 将尝试查找`common.yaml`文件。

使用`hiera-eyaml`时，我们需要声明一个新的后端来查找加密文件中的数据。这个后端是`eyaml`，默认情况下，我们将查找扩展名为`.eyaml`的文件。这个后端依赖于一对密钥来读取数据，因此我们需要生成这些密钥。

幸运的是，存在一个名为`puppet/hiera`的 Puppet 模块来处理这一切。因此，我们只需将其与依赖项添加到`Puppetfile`中（不要忘记运行`r10k puppetfile install`）：

```
mod 'puppetlabs/inifile'
mod 'puppet/hiera'
mod 'puppetlabs/puppetserver_gem' 
```

使用这个模块，现在可以非常轻松地使用以下命令准备 Puppet 服务器：

```
node 'puppet.pomes.pro' {

  # Create a service resource for the puppetserver
  # This is needed by the hiera module, in order
  # to restart the server once hiera-eyaml is installed
  service {'puppetserver':
    ensure => running,
  }
  # Configure hiera
  class { 'hiera':
    hierarchy => [
      'nodes/%{::trusted.certname}',
    ],
    eyaml          => true,
    manage_package => true,
    provider       => 'puppetserver_gem',
    master_service => 'puppetserver',
  }
}
```

这段代码将会：

+   为 Puppet 服务器声明一个服务资源。这是`puppet/hiera`模块所需的（请参见参数`master_service`）。

+   在 Puppet 服务器中安装`eyaml`后端。

+   更新 Hiera 配置以便使用这个后端。

+   生成私钥和公钥。

+   重启 Puppet 服务器。

私钥和公钥将分别放置在 `/etc/puppetlabs/puppet/keys/private_key.pkcs7.pem` 和 `/etc/puppetlabs/puppet/keys/public_key.pkcs7.pem` 中。

### 准备工作站

为了准备工作站，按照以下步骤操作：

1.  要创建和编辑加密数据，我们需要`eyaml`。让我们通过以下命令来安装它：

    ```
    $ sudo puppet resource package hiera-eyaml provider=puppet_gem

    ```

1.  让我们从 Puppet 服务器复制密钥，并将它们存储在 `$HOME` 下的 `keys` 文件夹中：

    ```
    $ ls ~/keys/
    private_key.pkcs7.pem  public_key.pkcs7.pem

    ```

1.  出于安全原因，限制对私钥的访问是个好主意：

    ```
    $ chmod 500 keys
    $ chmod 400 keys/private_key.pkcs7.pem

    ```

1.  我们还需要一个 `eyaml` 配置文件，位于 `~/.eyaml/config.yaml`，内容如下（不要忘记调整你的 `$HOME` 目录路径）：

    ```
    ---
    pkcs7_public_key: "/Users/me/keys/public_key.pkcs7.pem"
    pkcs7_private_key: "/Users/me/keys/private_key.pkcs7.pem"
    ```

我们现在准备加密敏感数据。

### 安全地设置 MySQL 根密码

从命令行，`eyaml` 可以加密值。以下是一个会话示例：

```
$ eyaml encrypt -s 'super_secure_password'
[hiera-eyaml-core] Loaded config from /Users/me/.eyaml/config.yaml
string: ENC[PKCS7,MIIBiQYJKoZIhvcNAQcDoIIBejCCAXYCAQAxggEhMIIBHQIBADAFMAACAQEwDQYJKoZIhvcNAQEBBQAEggEALjJ2a9uZ04lk2V5xKqEd0n3BtA4OLe1B6rA2iVruJRKxWJdevuGvJ55DDedRwBMZmqbvSMO1cgMUyPbfEy54i3SXw4x3LEuxc1R31ILoOspBgzU4OLuepCotuhBASA/pI/xu40y66AZAcCQ4CtD9SZJYjiWNtUA91rcARy/xYQGK39QievxT2eq5De89qIn2w/5fIRIkJBRyNqnwyYCWKcKSRwaiLbimpwmarOP+dxGHEFRrD/FiM4NfoV1WNNVr1UkPEFuNrWBzwBpvyZUnMbGHN676Rg5vq9sS6aWI6zPxTrJyLtssZm1f4GsfhmE+anFmuxrcWtEH6C82wKMOoTBMBgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBC3MhSP09yUw8XTj0XdlG1VgCCDCGhqIFdUmORYKlq0Pn5CE/cDZKTO+bhHxdBw5amAGQ==]

OR

block: >
 ENC[PKCS7,MIIBiQYJKoZIhvcNAQcDoIIBejCCAXYCAQAxggEhMIIBHQIBADAFMAACAQEw
 DQYJKoZIhvcNAQEBBQAEggEALjJ2a9uZ04lk2V5xKqEd0n3BtA4OLe1B6rA2
 iVruJRKxWJdevuGvJ55DDedRwBMZmqbvSMO1cgMUyPbfEy54i3SXw4x3LEux
 c1R31ILoOspBgzU4OLuepCotuhBASA/pI/xu40y66AZAcCQ4CtD9SZJYjiWN
 tUA91rcARy/xYQGK39QievxT2eq5De89qIn2w/5fIRIkJBRyNqnwyYCWKcKS
 RwaiLbimpwmarOP+dxGHEFRrD/FiM4NfoV1WNNVr1UkPEFuNrWBzwBpvyZUn
 MbGHN676Rg5vq9sS6aWI6zPxTrJyLtssZm1f4GsfhmE+anFmuxrcWtEH6C82
 wKMOoTBMBgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBC3MhSP09yUw8XTj0Xd
 lG1VgCCDCGhqIFdUmORYKlq0Pn5CE/cDZKTO+bhHxdBw5amAGQ==]

```

由于 `eyaml` 后端正在寻找扩展名为 `.eyaml` 的文件，我们只需创建一个 `hieradata/nodes/web.pomes.pro.eyaml` 文件，内容如下：

```
---
root_password: >
 ENC[PKCS7,MIIBiQYJKoZIhvcNAQcDoIIBejCCAXYCAQAxggEhMIIBHQIBADAFMAACAQEw
 DQYJKoZIhvcNAQEBBQAEggEALjJ2a9uZ04lk2V5xKqEd0n3BtA4OLe1B6rA2
 iVruJRKxWJdevuGvJ55DDedRwBMZmqbvSMO1cgMUyPbfEy54i3SXw4x3LEux
 c1R31ILoOspBgzU4OLuepCotuhBASA/pI/xu40y66AZAcCQ4CtD9SZJYjiWN
 tUA91rcARy/xYQGK39QievxT2eq5De89qIn2w/5fIRIkJBRyNqnwyYCWKcKS
 RwaiLbimpwmarOP+dxGHEFRrD/FiM4NfoV1WNNVr1UkPEFuNrWBzwBpvyZUn
 MbGHN676Rg5vq9sS6aWI6zPxTrJyLtssZm1f4GsfhmE+anFmuxrcWtEH6C82
 wKMOoTBMBgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBC3MhSP09yUw8XTj0Xd
 lG1VgCCDCGhqIFdUmORYKlq0Pn5CE/cDZKTO+bhHxdBw5amAGQ==]

```

然而，`eyaml` 具有一个非常方便的功能——`edit` 模式——允许我们基于存储在 `$HOME` 目录中的密钥，创建和编辑加密的明文值：

```
$ eyaml edit hieradata/nodes/web.pomes.pro.eyaml

```

这个命令将启动一个编辑器，我们只需输入以下内容：

```
---
root_password: >
 DEC::PKCS7[super_secure_password]!

```

在保存时，`eyaml` 将写入包含加密内容的文件用于 `root_password`。如果需要，我们可以再次编辑文件，所有加密值将被自动解密。

### 注意

使用 `eyaml edit` 编辑时，所有新的值应包含在 `DEC::PKCS7[value]!` 块中。对于现有值，`eyaml` 将为 `DEC(<num>)::PKCS7[value]!` 块添加一个名为 `num` 的索引。此索引必须保持不变。

作为最后一步，我们需要修改主清单文件，进行 Hiera 查找，以便获取密码：

```
node 'web.pomes.pro' {
...
    $pass=hiera('root_password');
...
    class { 'mysql::server':
      root_password => $pass;
    }
...
}
```

现在，根密码已经在 Hiera 中加密，只有持有密钥的人才能恢复它。

## 另见

+   `hiera-eyaml` GitHub 仓库及其文档在 [`github.com/TomPoulton/hiera-eyaml`](https://github.com/TomPoulton/hiera-eyaml)

+   `puppet-hiera` GitHub 仓库及其文档在 [`github.com/voxpupuli/puppet-hiera`](https://github.com/voxpupuli/puppet-hiera)

# 使用 Chef Vault 加密

另一种加密数据的方法是通过 Chef Vault 提供的，它不要求你将密钥包含在代码中。这个概念既优雅又简单：共享密钥加密针对每个现有的 Chef 节点进行，并通过它们已经存在的客户端密钥完成。这样，只有被授权访问数据的节点才能解密数据——每个节点使用自己的私钥——确保没有明文共享的密钥像经典的加密数据包方案那样发送。

## 准备就绪

要按照这个步骤操作，你需要：

+   工作站上的 Chef DK 安装已就绪

+   工作站上的 Vagrant 安装已就绪

+   来自第六章, *使用 Chef 和 Puppet 管理服务器的基础*， 第七章, *使用 Chef 和 Puppet 测试与编写更好的基础设施代码*，或者任何自定义的 Chef 代码

## 如何操作……

我们将在之前已有的`mysite`食谱基础上进行扩展；不过，任何其他情况也可以类似处理。我们将创建一个新的`eu-west-1`条目，类似于`data_bags/aws/eu-west-1.json`中`us-east-1`的条目：

```
{
  "id": "eu-west-1",
  "aws_access_key": "an_access_key",
  "aws_secret_key": "a_secret_key"
}
```

如我们所知，数据将为每个运行节点的公钥进行加密。这意味着我们必须根据搜索来过滤主机。我建议你使用`search(*:*)`来搜索每个节点；然而，你可以根据自己的需要，限制搜索条件，以提高安全性或适应实际情况，比如使用标签或角色，例如`search(tags:aws)`或`search(role:mysite)`：

```
$ knife vault create aws eu-west-1 --json data_bags/aws/eu-west-1.json --search "*:*" --mode "client"

```

### 注意

执行时别忘了`--mode "client"`选项，尤其是当使用 Chef 服务器时，就像我们一样！

### 从一个食谱中访问加密的 Vault

`knife vault`的配套工具是`chef-vault`食谱。我们将用它来轻松地在我们的食谱中访问加密的数据。如果你使用 Berkshelf 管理依赖项，请不要忘记在需要的地方添加食谱（无论是`metadata.rb`还是`Berksfile`）。在`aws.rb`文件中，包含`chef-vault`食谱，并将`aws`设置为`chef_vault_item`帮助器搜索的结果：

```
include_recipe 'chef-vault'
aws = chef_vault_item('aws', 'eu-west-1')
```

如果发起请求的节点没有权限用其私钥解密数据，则会出现错误。如果节点能够解密数据，就像我们之前使用传统数据包那样，数据就会可供使用：

```
template "/etc/aws/credentials" do
  source 'aws.erb'
  owner 'root'
  group 'root'
  mode '0600'
  variables(
    aws_region: aws['id'],
    aws_access_key: aws['aws_access_key'],
    aws_secret_key: aws['aws_secret_key']
  )
end
```

最终，`/etc/aws/credentials`文件会填充有效的未加密数据：

```
$ sudo cat /etc/aws/credentials
[eu-west-1]
aws_access_key_id = an_access_key
aws_secret_access_key = a_secret_key

```

使用 Chef Vault 时，从未有任何共享密钥以明文形式传输，只有过滤后的和现有的节点才能解密专门为它们加密的数据。这个工具能做的远不止这些！

## 另见

+   Chef Vault 宝石在[`github.com/chef/chef-vault`](https://github.com/chef/chef-vault)

+   Chef Vault 食谱在[`github.com/chef-cookbooks/chef-vault`](https://github.com/chef-cookbooks/chef-vault)

# 使用 Ohai 访问和操作系统信息

Chef 通过 Ohai 可以获取大量来自系统的信息。这个程序在每次 Chef 运行时执行，并将收集的所有信息存储在 Chef 数据库中，使其能够直接在食谱中使用。默认情况下收集的信息量非常庞大。

它涵盖了从网络详细信息——比如链路速度、MTU 或地址——到你在像`top`这样的工具上看到的所有内存使用详情，再到文件系统或虚拟化系统的所有可想象的数据，甚至包括每个已安装的包和登录用户的列表。

除此之外，Ohai 是一个模块化系统，拥有许多社区插件，可以将 Dell DRAC 信息与与 KVM、LXC 或 XenServer 相关的支持信息集成。

它甚至可以用来检索与**Windows 管理工具**（**WMI**）相关的特定数据。显然，我们可以编写自己的插件，但这已经超出了本书的范围。

## 准备工作

要逐步完成这个食谱，你需要：

+   工作站上安装的 Chef DK

+   工作站上安装的 Vagrant

+   来自第六章, *通过 Chef 和 Puppet 管理服务器的基础*, 第七章, *使用 Chef 和 Puppet 测试和编写更好的基础设施代码*，或任何自定义的 Chef 代码

## 如何操作…

在一个全新且精简的 CentOS 7.2 虚拟机安装中，`ohai`输出包含 5,292 行信息，内容丰富。要逐步查看，请参考以下内容：

```
$ ohai | more
{
 "cpu": {
 "0": {
 "vendor_id": "GenuineIntel",
 "family": "6",
 "model": "69",
 "model_name": "Intel(R) Core(TM) i7-4578U CPU @ 3.00GHz",
 "stepping": "1",
 "mhz": "2999.991",
 "cache_size": "4096 KB",
 "physical_id": "0",
 "core_id": "0",
 "cores": "1",

```

或者，另一种解决方案是将其内容重定向到一个文件中，以便使用专用工具更方便地处理：

```
$ ohai > ohai.json

```

所有这些信息在 Chef 界面中也可以图形化显示，当你选择节点并进入**属性**选项卡时：

![如何操作…](img/B05671_08_01.jpg)

### 从 Chef 食谱中访问 Ohai 信息

现在让我们从食谱中访问这些信息。我们希望有一个`index.html`页面来展示其中的一些信息，因此让我们编辑已经存在的`apache`食谱中的页面；不过，你也可以从头开始。我们希望这个页面动态地展示类似这样的内容：

```
This centos 7.2.1511 linux system version 3.10.0-327.el7.x86_64 listening on 192.168.146.129 is up since 25 minutes 55 seconds

```

我们需要的所有信息都存储在 ohai 的某个地方：`platform`、`platform_version`、`os`、`os_version`、`ipaddress`或`uptime`都是有效的值。让我们使用它们。

在`apache/templates/index.html.erb`中，添加以下内容：

```
This <%= node['platform'] %> <%= node['platform_version'] %> <%= node['os'] %> system version <%= node['os_version'] %> listening on <%= node['ipaddress'] %> is up since <%= node['uptime'] %>
```

为了构建更有趣的东西，因为平台名称可用，我们可以让我们的`apache`食谱在不同的 Linux 发行版之间更具可移植性。当运行在 Ubuntu 上时，安装`apache2`包；否则，安装`httpd`包。（这需要更精确，以处理所有实际情况。）在`apache::default`食谱中，做如下更改，以便在运行 Ubuntu 时将`httpd`变量设置为`apache2`，在其他地方设置为默认的`httpd`：

```
if node['platform'] == 'ubuntu'
  httpd = 'apache2'
else
  httpd = 'httpd'
end

package httpd do
  action :install
end

service httpd do
  action [:enable, :start]
end
```

这就是我们如何开始在 Chef 基础设施中利用强大的`ohai`命令。

## 还有更多…

Puppet 的对应工具是`facter`，它与 Puppet 代理一起安装。像`ohai`一样，`facter`是一个命令行工具：

```
$ facter | more
aio_agent_version => 1.8.0
augeas => {
 version => "1.4.0"
}
disks => {
 sda => {
 model => "VBOX HARDDISK",
 size => "40.00 GiB",
 size_bytes => 42949672960,
 vendor => "ATA"
 }
}
dmi => {
 bios => {
 release_date => "12/01/2006",
 vendor => "innotek GmbH",
 version => "VirtualBox"
 },
 board => {
 manufacturer => "Oracle Corporation",
 product => "VirtualBox"
 },
...

```

至于 Chef，`facter`信息可以从 Puppet 清单中访问。在 Puppet 中，这种信息被称为**事实**。

从 Puppet 4.x 开始，可以通过`$facts`哈希从清单中访问事实。让我们尝试为`apache`模块创建更具可移植性的代码行：

```
if $facts['os']['family'] == 'debian' {
  $packagename='apache2'
} else {
  $packagename='httpd'
}

package{'apache2':
  ensure => present,
  name   => $packagename,
}  

service{'apache2':
  ensure => running,
  enable => true,
  name   => $packagename,
}
```

### 注意

你可能会看到一些访问事实的代码片段，使用变量，如 `$osfamily` 而不是 `$facts['os']['family']`。这种方法适用于旧版本的 Puppet，但这里没有明显显示正在使用一个事实。

## 另见

+   Ohai 插件在 [`docs.chef.io/plugin_com`](http://ttps://docs.chef.io/plugin_com)[munity.html#ohai](http://munity.html#ohai)

+   [`docs.puppet.com/puppet/4.8/lang_facts_and_builtin_vars.html`](https://docs.puppet.com/puppet/4.8/lang_facts_and_builtin_vars.html)

# 自动化应用程序部署（一个 WordPress 示例）

Chef 也可以用来从代码仓库部署应用程序。它结合了最完整、功能最丰富、最复杂的 Chef 资源之一——`deploy` 资源——以及各种强大而流行的食谱，如 `database` 食谱。我们将展示如何直接从 GitHub 仓库部署一个简单的 WordPress 应用程序，创建专用的数据库和用户，以及所有必要的依赖项。这在先前的基础上进行了扩展，但这里展示的资源和食谱可以在任何地方重用。

## 准备工作

要逐步执行此食谱，您将需要：

+   工作站上有一个可用的 Chef DK 安装

+   工作站上有一个可用的 Vagrant 安装

+   来自 第六章，*使用 Chef 和 Puppet 管理服务器的基础知识*，第七章，*测试和编写更好的基础设施代码*，或任何自定义的 Chef 代码

## 如何操作…

由于我们将为 MySite 部署一个应用程序（可能是 MySite 公司的工程博客），让我们将此食谱命名为 `mysite::deploy`。从 `chef-repo` 中创建该食谱：

```
$ chef generate recipe cookbooks/mysite deploy

```

我们接下来的步骤将包括 Apache 和 MySQL 依赖项，配置 MySQL 以便安装 WordPress，并最终从 GitHub 部署 WordPress 代码。

### 包括依赖项

一个 WordPress 安装至少需要一个 HTTP 服务器和一个数据库。首先，通过包括我们已经拥有的服务的已知依赖项来启动：一个 Apache 虚拟主机和 MySQL。在 `deploy.rb` 中包含它们：

```
include_recipe 'apache::virtualhost'
include_recipe 'mysite::mysql'
```

### 创建应用程序的数据库

在我们部署任何内容之前，我们需要在已经运行的 MySQL 服务器上创建一个数据库，并为 WordPress 创建一个专用用户。这里有一个专门用于此目的的精彩食谱：`database` 食谱。我们将频繁重用这个食谱。它提供了许多帮助函数，适用于大多数用例和各种类型的数据库。根据文档，我们需要部署一个名为 `mysql2_chef_gem` 的 gem，幸运的是，它也带有一个专门的食谱。最后，由于我们使用的是 MySQL，让我们确保依赖于其官方食谱。我们将所有这些信息包括在我们的 `mysite` 食谱的 `metadata.rb` 中：

```
depends 'database'
depends 'mysql2_chef_gem', '~> 1.1'
depends 'mysql', '~> 8.1'
```

要使用烹饪手册中的新 `mysql2_chef_gem` 资源构建 `mysql2` gem，我们需要安装名为 `mysql-community-devel` 的 MySQL 开发包。我们需要将以下内容添加到我们的 `deploy.rb` 配方中：

```
package 'mysql-community-devel'

mysql2_chef_gem 'default' do
  action :install
end
```

数据库烹饪手册创建了包括 `mysql_database` 和 `mysql_database_user` 在内的两个对我们非常有用的资源。顾名思义，它们分别帮助创建 MySQL 数据库和 MySQL 用户。现在我们来创建 MySQL 连接信息变量，以便在这两个资源中重复使用：

```
mysql_connection_info = {
  host: '127.0.0.1',
  username: 'root',
  password: 'super_secure_password'
}
```

### 注意

在正式的 `production` 环境中，我们应该使用加密的数据包来处理这个问题，正如本章所展示的那样。这里我们试图保持代码简单。

现在我们可以使用 `mysql_database` 资源创建名为 `wordpress` 的数据库：

```
mysql_database 'wordpress' do
  connection  mysql_connection_info
  action      :create
end
```

此外，创建一个 `wordpress_user` MySQL 用户，密码为 `changeme`。这将创建该用户并授予其所有权限：

```
mysql_database_user 'wordpress_user' do
  connection    mysql_connection_info
  password      'changeme'
  database_name 'wordpress'
  host          '%'
  privileges    [:all]
  action        [:create, :grant]
end
```

到此为止，我们应该已经具备与数据库相关的所有内容。

### 从 git 或 GitHub 部署应用程序

现在进入应用程序部署！我们知道我们想从 `git` 部署。让我们确保 `git` 已安装：

```
package 'git'
```

`deploy_revision` 资源是所有资源中最复杂的。它有很多选项，甚至一个完整的章节也不足以讲解它。我们在这里保持简单，并建议你参考完整的在线文档以了解更复杂的用法。我们保持简单，请参考非常完整的在线文档以获取更复杂的用法——因为这个资源非常强大，正确使用时能产生惊人的效果。我们知道以下几点：

+   我们的代码可通过 [`github.com/WordPress/WordPress`](https://github.com/WordPress/WordPress) 获取

+   我们想尝试最新的修订版本（`HEAD`）并保留最后五个修订版本以便回滚

+   我们的 HTTP Web 服务器以 `apache` 用户身份运行

+   虚拟主机文件夹继承自之前设置的属性（`/var/www/#{node['sitename']}`）

+   在 WordPress 中无需执行数据库迁移

`deploy_revision` 资源是模仿 Capistrano 的，因此它源自 Ruby on Rails 的世界。但这些概念仍然适用于大多数编程语言，并且在生产环境中创建共享文件夹和符号链接以配置长期存在的文件和配置是一个很好的实践。它包括证书、数据库配置文件、本地资产等。然而，为了保持当前部署的简单性，我们现在不使用这些内容，尽管你可能会在需要时开始考虑它们。我们将包含符号链接的配置，并将其初始化为空，这样在需要时代码已经准备好。下面是所有这些如何联系起来：

```
deploy_revision 'wordpress' do
  repo 'https://github.com/WordPress/WordPress'
  revision 'HEAD'
  user 'apache'
  deploy_to "/var/www/#{node['sitename']}"
  keep_releases 5
  symlinks({})
  symlink_before_migrate({})
  migrate false
  action :deploy
end
```

一旦代码应用，`/var/www/mysite`（或你可能已覆盖的任何名称）结构将稍有变化：

```
$ ls /var/www/mysite/
current  index.html  releases  shared

```

这里有 `current`、`releases` 和 `shared` 文件夹。`shared` 文件夹包含所有在发布版本之间保留的内容，包括当前代码的缓存副本。`releases` 文件夹包含所有已存储的发布版本。`current` 文件夹本身是一个指向特定发布版本的符号链接，该版本是 GitHub 上的 git 提交 SHA（`72606bed348e61b6f98318cf920684765aa08b37`）。每个随后的发布版本将通过其 SHA 标识，指示其唯一标识，并在部署过程结束时符号链接到 `current`。保留的发布版本数量由 `keep_releases` 整数设置：

```
$ ls -ld /var/www/mysite/current
lrwxrwxrwx. 1 apache root 65 Nov 17 02:18 /var/www/mysite/current -> /var/www/mysite/releases/72606bed348e61b6f98318cf920684765aa08b37

```

一旦这个代码应用到我们的节点，如果我们访问 `http://<node_ip>/current/`，就会看到 WordPress 安装页面：

![从 git 或 GitHub 部署应用程序](img/B05671_08_02.jpg)

为了检查与数据库的连接是否正常，输入我们 Chef 代码中的所有信息：

![从 git 或 GitHub 部署应用程序](img/B05671_08_03.jpg)

干得漂亮！WordPress 安装程序说，**好样的，sparky！你已经完成了安装的这一部分。WordPress 现在可以与数据库通信。如果你准备好了，现在是... 运行安装程序的时候了！**

现在，我们基本上可以随时随地、反复无常地从零开始部署任何 WordPress 安装。

### 注意

**让我们再坚持一遍**：

一旦你熟悉了这个，参考 `deploy` 资源文档，发现这个资源能提供的一切功能。它效果非凡。

## 还有更多…

使用 Puppet 时，没有 `deploy` 资源。然而，Puppet Labs 提供了一个有用的模块——`vcsrepo`。通过这个模块，我们将能够从 git 部署一个 WordPress 网站。

让我们在 第六章中重用我们的 Vagrant LAMP 示例，*使用 Chef 和 Puppet 管理服务器的基础*。我们只需将 `vcsrepo` 模块添加到 `Puppetfile` 中（不要忘记运行 `r10k puppetfile install`）：

```
mod 'puppetlabs/vcsrepo', '1.4.0'
```

现在，我们准备修改箱子的主清单，即 `web.pomes.pro`，以包括 WordPress 部署。首先，安装 `git` 包：

```
  package {'git':
    ensure => installed,
  }
```

然后，为 WordPress 创建一个数据库：

```
  mysql::db {'wordpress':
    user     => 'wordpress_user',
    password => 'changeme',
    host     => '%',
    grant    => 'ALL',
  }
```

另外，更新我们的虚拟定义以更改 `DocumentRoot`：

```
  apache::vhost { 'web.pomes.pro':
    website => 'web.pomes.pro',
    docroot => '/var/www/wordpress',
  }
```

最后，从 `git` 安装 WordPress，并赋予 Apache 权限：

```
  vcsrepo {'/var/www/wordpress':
    ensure   => latest,
    provider => git,
    source   => 'https://github.com/WordPress/WordPress',
    revision => 'master',
  }

  file {'/var/www/wordpress':
    ensure    => directory,
    owner     => 'www-data',
    group => 'www-data',
	recurse => true,
}	

```

## 另请参见

+   `deploy` 资源文档，网址为 [`docs.chef.io/resource_`](https://docs.chef.io/resource_)[deploy.html](http://deploy.html)

+   PuppetLabs 的 `vcsrepo` 模块，网址为 [`forge.puppet.com/puppetlabs/vcsrepo`](https://forge.puppet.com/puppetlabs/vcsrepo)

+   WordPress 强化指南，网址为 [`codex.wordpress.org/Hardening_WordPress`](https://codex.wordpress.org/Hardening_WordPress)

# 使用 TDD 工作流

TDD（测试驱动开发）是开发团队中一种流行的技术，其包含以下步骤：首先编写测试，这些测试会失败，因为实际上没有编写代码，然后编写代码使得这些测试通过。通过这种方式，我们确保编写的代码已经过测试，并且真正覆盖了测试的范围；如果有某天出现回归问题，它会立即被发现。这里，我们将展示一个完整的工作流程，从开发到生产，我们将在 CentOS 7 和 Ubuntu 16.04 上使用 TDD 技术部署 Docker。通过使用 Git 分支、Chef 工具、Test Kitchen、linting 和 ServerSpec，我们将按照 TDD 原则的每个步骤进行一个小项目的开发。我们这样做是为了在团队中实现最高的代码质量，从最初的开发阶段一直到最终的生产环境。

## 准备工作

要执行这个食谱，你需要：

+   在工作站上安装一个可用的 Chef DK。

+   在工作站上安装一个可用的 Vagrant。

+   Chef 代码（可选）来自 第六章，*使用 Chef 和 Puppet 管理服务器的基础知识*，第七章，*使用 Chef 和 Puppet 编写更好的基础设施代码*，或任何自定义的 Chef 代码。

## 如何执行……

我们的目标是基于 Docker 启动一个新平台。为此，请按照以下步骤操作：

1.  从创建 `platform` cookbook 开始：

    ```
    $ cd chef-repo
    $ chef generate cookbook cookbooks/platform

    ```

1.  现在创建一个空的 `platform::docker` 食谱：

    ```
    $ chef generate recipe cookbooks/platform docker

    ```

1.  如果还未初始化 `git` 仓库，请执行初始化操作：

    ```
    $ git init

    ```

1.  如果当前有任何工作内容，请添加并提交到仓库：

    ```
    $ git add .
    $ git commit -m "initial chef repo state" 

    ```

1.  为我们即将处理的 `docker` 支持创建一个功能分支：

    ```
    $ git checkout -b docker_support

    ```

### 基础设施 TDD —— 先编写测试

让我们先编写测试，以便它们肯定会失败，这样我们就知道我们是从正确的地方开始构建的。

在平台 cookbook 中创建 ServerSpec 集成文件夹：

```
$ mkdir -p test/integration/default/serverspec

```

在 `platform` cookbook 文件的根目录创建 `.kitchen.yml` 文件，内容如下。我们将使用 Vagrant 并通过 `chef_zero` 提供程序模拟一个 Chef 服务器。我们希望我们的平台能够在 Ubuntu 16.04 和 CentOS 7.2 上运行，并希望将 cookbook 的默认食谱作为其入口点：

```
---
driver:
  name: vagrant

provisioner:
  name: chef_zero
  always_update_cookbooks: true

platforms:
  - name: centos-7.2

suites:
  - name: default
    run_list:
      - recipe[platform::default]
    attributes:
```

如本书前面所述，在 `serverspec` 文件夹中创建一个名为 `spec_helper.rb` 的助手脚本：

```
require 'serverspec'
# Required by serverspec
set :backend, :exec
```

让我们开始测试，看看根据我们的需求，想要做什么：

+   我们希望安装 `docker-engine` 包。

+   我们希望 `docker` 服务能够启用并启动。

+   我们希望拉取一个特定的 `docker` 镜像（即 `sjourdan/terraform:0.7.10`）。

让我们在 `serverspec` 文件夹中的 `docker_spec.rb` 中编写这些测试：

```
require 'spec_helper'

describe package('docker-engine') do
  it { should be_installed }
end

describe service('docker') do
  it { should be_enabled }
  it { should be_running }
end

describe command('docker images') do
  its(:exit_status) { should eq 0 }
  its(:stdout) { should match(%r{^sjourdan/terraform\s.*0.7.10}) }
end
```

这已经足够满足我们的需求！让我们通过启动 `kitchen` 来启动我们的测试环境：

```
$ kitchen create
$ kitchen converge
$ kitchen verify
[...]
       Package "docker-engine"
         should be installed (FAILED - 1)

       Service "docker"
         should be enabled (FAILED - 2)
         should be running (FAILED - 3)

       Command "docker images"
         exit_status
           should eq 0 (FAILED - 4)
         stdout
           should match /^sjourdan\/terraform\s.*0.7.10/ (FAILED - 5)
[...]
```

我们成功地失败了！Docker 既未安装，也未启用，也未启动，且没有 `docker` 镜像。

让我们开始工作。

### 使用 Chef 部署 Docker。

有一本非常棒的食谱，文档写得极其详尽，做了我们需要的所有事情（[`github.com/chef-cookbooks/docker`](https://github.com/chef-cookbooks/docker)）。我们将它添加到 `metadata.rb` 中，以便让它成为我们的依赖：

```
depends 'docker', '~> 2.0'
```

如果你计划在 `Berksfile` 中使用它，也可以将它添加到 Berkshelf 中：

```
cookbook 'docker', '~> 2.0'
```

由于我们将在 `platform::docker` 食谱中编写 Docker 代码，首先让我们将其包含在 `default.rb` 食谱中：

```
include_recipe 'platform::docker'
```

`docker` 食谱为我们提供了一个新的资源，名为 `docker_installation`，它正是做这件事：安装 `docker`。有许多安装选项可以让你玩弄。让我们保持简单，从 Docker 仓库安装当前稳定版本的 Docker（而不是从我们的 Linux 发行版安装）。将以下内容添加到 `docker.rb` 食谱中：

```
docker_installation 'default' do
  repo 'main'
  action :create
end
```

再次执行 `kitchen` 以应用我们的代码，并查看测试是通过还是失败：

```
$ kitchen converge
$ kitchen verify
[...]
 Package "docker-engine"
 should be installed
[...]
 Finished in 0.18797 seconds (files took 0.43908 seconds to load)
 5 examples, 4 failures

```

很好！几分钟前失败的测试现在通过了。这证明我们的行动解决了问题，我们走在正确的道路上。然而，其他测试仍然失败。

让我们创建 Docker 服务并使用 `docker_service` 资源启动它，这是食谱为我们提供的：

```
docker_service 'default' do
  action [:create, :start]
end
```

再次执行 `kitchen` 以应用我们的代码，并查看测试结果：

```
$ kitchen converge
$ kitchen verify
[...]
 Package "docker-engine"
 should be installed

 Service "docker"
 should be enabled
 should be running
[...]
 Finished in 0.12301 seconds (files took 0.28237 seconds to load)
 5 examples, 1 failure

```

很好！服务现在已通过测试并启用，并且正在运行。让我们添加一个小要求，即从一开始就拉取一个镜像，我们选择了 Docker 镜像 `sjourdan/terraform`，版本为 0.7.10：

```
docker_image 'sjourdan/terraform' do
  tag '0.7.10'
  action :pull
end
```

再次执行 `kitchen` 以应用我们新的代码，并检查测试是否通过：

```
$ kitchen converge
$ kitchen verify
[...]
 Finished in 0.23526 seconds (files took 0.44015 seconds to load)
 5 examples, 0 failures

```

我们的代码似乎正如我们预期的那样运行！现在让我们销毁测试环境：

```
$ kitchen destroy

```

### 代码检查

别忘了使用 `cookstyle` 来检查我们的代码是否干净，运行在 `platform` 食谱内部：

```
$ cookstyle
Inspecting 6 files
......

6 files inspected, no offenses detected

```

没有错误！我们的代码很干净。我们继续进行下一步。

### 支持其他平台

让我们检查一下这段代码是否也能在 Ubuntu 16.04 上运行。能在当前支持长期维护的两个平台上都运行岂不是太棒了吗？只需将平台添加到食谱的 `kitchen.yml` 文件中：

```
  - name: ubuntu-16.04
```

再次启动 `kitchen` 并检查它是否也能在 Ubuntu 16.04 上工作：

```
$ kitchen test
[...]

 Package "docker-engine"
 should be installed

 Service "docker"
 should be enabled
 should be running

 Command "docker images"
 exit_status
 should eq 0
 stdout
 should match /^sjourdan\/terraform\s.*0.7.10/

 Finished in 0.27516 seconds (files took 0.43079 seconds to load)
 5 examples, 0 failures

 Finished verifying <default-ubuntu-1604> (5m9.12s).

```

我们现在确认我们的代码也支持 Ubuntu 16.04 了！

### 团队使用 Chef 和 git 协作

现在我们的 `platform` 食谱在 `docker_support` git 分支上运行得相当好，让我们提交这项工作。首先检查哪些内容尚未被跟踪：

```
$ git status
On branch docker_support
Untracked files:
 (use "git add <file>..." to include in what will be committed)

 cookbooks/platform/ 

```

提交这项工作：

```
$ git add cookbooks/platform
$ git commit -m "added docker support to the platform"

```

我们的 `git` 树干净吗？可以将它提交给我们的团队了吗？使用以下代码检查：

```
$ git status
On branch docker_support
nothing to commit, working tree clean

```

然后，让我们将其推送到我们的 `git` 仓库（可能是 GitHub，也可能是其他任何平台）：

```
$ git push

```

现在我们的一位同事可以进行代码审查，并最终将 `docker_support` 合并到 `master` 分支：

```
$ git merge docker_support master 

```

我们的新食谱现在已经准备好投入实际使用，可以部署到预发布环境了。

### 注意

在更复杂的设置中，强烈建议你在持续集成系统中运行这些集成测试，如 Jenkins。这些系统可以与 GitHub 或 GitLab 等服务很好地集成，并在推送或拉取请求后自动启动测试。这是附加的价值，确保在发布前的过程质量。

### 部署到暂存环境

现在，让我们将这个新平台的 cookbook 部署到我们的暂存环境。首先，确保我们有所有必需的 cookbook 依赖项：

```
$ cd cookbooks/platform
$ berks

```

然后，上传所有必需的 cookbook：

```
$ berks upload

```

使用已存在的环境，如`staging`，并将我们新的`platform` cookbook 版本约束添加到`environments/staging.rb`或你正在使用的任何类似环境中：

```
name 'staging'
description 'The staging environment'
cookbook_versions  'platform' => '= 0.1.0'
```

使用`knife`命令更新该环境：

```
$ knife environment from file environments/staging.rb
Updated Environment staging

```

将此代码提交到`git`：

```
$ git add .
$ git commit -m "added platform::docker to staging"

```

将平台 cookbook 添加到目标节点的 run_list 中：

```
$ knife node run_list add my_node_name 'recipe[platform]'

```

等待下次 Chef 运行或自行运行，Docker 将在任何节点上可用，包括此配方。

### 部署到生产环境

在此阶段，部署到生产环境与部署到暂存环境完全相同；没有区别。`environments/production.rb`文件现在应如下所示：

```
name 'production'
description 'The production environment'
cookbook_versions  'platform' => '= 0.1.0'
```

别忘了将其上传到 Chef 服务器：

```
$ knife environment from file environments/production.rb
Updated Environment production

```

将更改提交到`git`：

```
$ git add .
$ git commit -m "updated production env with platform::docker"

```

等待下次 Chef 运行或自行执行，从现在开始，我们将拥有一个简单的四步工作流程：

1.  本地存储 TDD 基础设施代码

1.  同行评审和合并

1.  部署到暂存环境

1.  部署到生产环境

现在，每当我们想要测试或暂存通过了步骤 1 和 2 的新版本 cookbook 时，我们只需增加 cookbook 版本号的约束，在不影响生产环境的情况下验证暂存环境中的结果，最后尽可能将其部署到生产环境。

## 还有更多内容…

对于 Puppet，相同的逻辑适用。在第七章，*使用 Chef 和 Puppet 测试和编写更好的基础设施代码*中，我们介绍了 Beaker 作为验收测试工具。在 TDD 工作流中，我们可以首先在任何模块的`specs/acceptance`子目录中编写验收测试，然后再编写代码本身。

使用多个`nodesets`，我们还可以确保测试可以在多个平台上进行验证。在第七章，*使用 Chef 和 Puppet 测试和编写更好的基础设施代码*中，我们仅在`spec/acceptance/nodesets/default.yml`中使用了一个平台（Ubuntu）。但是，我们可以根据需要创建任意多个。下面是一个应该在`spec/acceptance/nodesets/centos-7-x64.yml`中定义的 CentOS 节点的示例：

```
HOSTS:
  centos-7-x64:
    roles:
      - agent
      - default
    platform: redhat-7-x86_64
    hypervisor: vagrant
    box: puppetlabs/centos-7.2-64-nocm
CONFIG:
  type: foss
```

使用环境变量`BEAKER_set`，然后可以指定需要在哪个平台上运行测试：

```
$ BEAKER_set=centos-7-x64 rake beaker

```

使用 git 和 r10k 时，团队的工作流是相同的。我们首先在`test`分支上进行开发。当所有测试都成功通过时，我们将它们合并到`production`分支，并使用 r10k 部署代码。

## 另见

+   Docker 的 Chef 食谱，位于[`github.com/chef-cookbooks/docker`](https://github.com/chef-cookbooks/docker)

# 为最坏的情况做准备——训练重建工作系统

最终通过 Chef 管理整个基础设施是件不容易的事——一块一块地搭建，经过数周，经过不断的修改——确保 Chef 运行始终顺利且正常工作。然而，能够从零开始重新引导一个正常工作的系统是另一回事。假设当前的设置之所以能正常工作，其实是因为某个脚本或二进制文件仍然留在去年某个地方，正是它做了使系统正常运行的事情？如果应用服务器今晚发生了损坏呢？如果发生这种情况，我们能从头开始重建它吗？如果明天我们的 IaaS 云服务提供商崩溃了，在什么时间框架内我们能在其他地方重建系统（假设备份正常工作；嗯，那是另一回事）？

现在我们的系统尽可能地实现了自动化，希望能达到百分之百。了解在灾难发生时我们是否能够完全重新引导这些系统非常重要；如果可以，花多长时间。你可能会感到惊讶，当你收集一些数据后会发现，许多系统可以在几分钟内恢复。与之相比，找出过时的文档，应用未经测试的手动流程，最后在紧急情况下采取一切手段让系统运行起来，可能需要花费的时间要更长。如果我们知道所有的系统配置文件都在持续重新引导且成功地运行，那么我们会度过更轻松的夜晚和周末；实际上，为什么不每天晚上使用 CI 系统，这样每天早晨我们就能知道前一天的更改是否影响了任何事情。作为一个团队，我们始终知道，如果需要，我们随时准备重新部署系统。

## 做好准备

要按此步骤操作，您需要：

+   工作中的 Chef DK 安装

+   工作中的 Vagrant 安装

+   第六章中的 Chef 代码（可选），*使用 Chef 和 Puppet 管理服务器的基础知识*， 第七章中的 Chef 代码，*使用 Chef 和 Puppet 测试与编写更好的基础设施代码*，或任何自定义的 Chef 代码

## 如何操作……

没有单一的方式来实现我们的目标。我们已经介绍了 Test Kitchen，它可能是一个不错的解决方案，特别是如果我们编写了大量的测试。将其集成到公司的**持续集成**（**CI**）系统中，这将完成任务。

一个更简单和更快速的解决方案也可以是仅使用正确的 Chef-provisioning 配置文件启动 Vagrant 盒子，以满足每种用例：`docker`，`webserver`，数据库服务器或完整部署。

### 注意

有关 Vagrant 工具的更多信息，请参考本书的 Vagrant 章节！

我们的生产服务器通过一些 Chef 代码进行配置，并且目前这项工作做得很好。我们能够轻松地重新引导类似的 CentOS 7.2 服务器到一个类似的安装点，而没有任何 Chef 或系统错误吗？让我们通过在基础设施存储库的根目录包含`Vagrantfile`，使用以前用于部署 Docker 的项目代码来找出答案（但是对于任何类型的 Chef 存储库，这个想法是一样的）。我们最少可以做的是引导一个新鲜的 CentOS 7.2：

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
end
```

我们希望自动在我们的临时节点上安装 Chef，因此让我们使用`vagrant-omnibus`插件（记住，安装它很容易：`vagrant plugin install vagrant-omnibus`）。以下是执行此操作的代码：

```
  config.omnibus.chef_version = :latest
```

让我们配置 Vagrant provisioning 系统以使用 Chef Zero 来模拟 Chef 服务器。我们也可以直接使用真实的 Chef 服务器；如果我们在防火墙后面有一个，那将非常方便。我们必须指定所有内容的位置（cookbooks，environments，roles 等），还要添加一个空的`nodes`文件夹的微妙之处，这在我们的情况下将被留空。我们的虚拟机将在`production`环境中运行，并应用`docker`角色：

```
  config.vm.provision "chef_zero" do |chef|
    chef.cookbooks_path = "cookbooks"
    chef.environments_path = "environments"
    chef.roles_path = "roles"
    chef.nodes_path = "nodes"
    chef.environment = "production"
    chef.add_role "docker"
  end
```

我们快完成了！我们需要告诉 Vagrant Berkshelf 插件在哪里查找`Berksfile`以及是否启用它（安装 Berkshelf 插件很容易：`vagrant plugin install vagrant-berkshelf`）。以下是执行此操作的代码：

```
  config.berkshelf.berksfile_path = "cookbooks/platform/Berksfile"
  config.berkshelf.enabled = true
```

此时启动 Vagrant 只会从头部署所有内容：

```
$ vagrant up 
[...]
# Chef Client finished, 17/45 resources updated in 03 minutes 30 seconds

```

如果运行成功，表示已应用来自 Docker 角色的代码，我们是安全的。让我们销毁 VM：

```
$ vagrant destroy -f

```

在我们的 CI 系统中包含此 Vagrant 命令将确保此特定角色在此特定环境和此特定系统中无缺陷地运行，并且可能仅需三分半钟即可从无到有的工作状态恢复。

### 多机恢复

让我们转向更复杂的设置。Vagrant 支持多机设置，让我们为每个定义配置文件。在本章的前一个示例中，我们使用数据包和模板配置了 WordPress 安装，包括配置了 Apache web 服务器，所有数据都是加密的。我们将实现相同的想法，但`Vagrantfile`将包括多个机器配置文件：一个只启动带有`webserver`角色的虚拟机，另一个仅部署数据库部分，第三个启动所有内容，包括 Web 应用程序。所以我们要确保最终产品的所有部分都可以从头开始重新部署（这是主要目标）。

所有 VM 定义都将存储在主 Vagrant 配置内：

```
Vagrant.configure('2') do |config|
  config.vm.define 'whatever_vm', autostart: false do |node|
    [...]
  end
end
```

### 注意

我们建议禁用 VM 的自动启动，以免因错误而启动数十个 VM。

为了确保我们的代码能够从头开始仅部署`webserver`角色，我们将需要进行以下操作——为所有内容设置路径，包括作业的特定`Berksfile`：

```
  config.vm.define 'webserver', autostart: false do |ws|
    ws.vm.box = 'bento/centos-7.2'

    ws.vm.provision :chef_zero do |chef|
      chef.cookbooks_path = 'cookbooks'
      chef.environments_path = 'environments'
      chef.roles_path = 'roles'
      chef.nodes_path = 'nodes'
      chef.environment = 'production'
      chef.add_role 'webserver'
    end

    ws.berkshelf.berksfile_path = 'cookbooks/apache/Berksfile'
    ws.berkshelf.enabled = true
  end
```

为了仅启动此框，以确保能够从头开始部署`webserver`角色，请使用以下命令：

```
$ vagrant up webserver

```

为了确保我们的代码能够从头开始仅启动此平台的数据库部分，只需在类似的环境中执行`mysite::mysql`配方：

```
  config.vm.define 'db', autostart: false do |db|
    db.vm.box = 'bento/centos-7.2'

    db.vm.provision :chef_zero do |chef|
      chef.cookbooks_path = 'cookbooks'
      chef.environments_path = 'environments'
      chef.roles_path = 'roles'
      chef.nodes_path = 'nodes'
      chef.environment = 'production'
      chef.add_recipe 'mysite::mysql'
    end

    db.berkshelf.berksfile_path = 'cookbooks/mysite/Berksfile'
    db.berkshelf.enabled = true
  end
```

为了仅启动此框，以确保能够从头开始部署数据库配方，请使用以下命令：

```
$ vagrant up db
[...]
Chef Client finished, 29/43 resources updated in 01 minutes 28 seconds

```

为了确保我们的代码能够从头开始启动整个平台，我们只需执行整个`mysite::default`配方再加一步。其中一个包含的配方使用了加密数据包。它在 Chef 服务器上以加密形式存储，但在本地，我们的`./data_bags/`目录当前仅包含未加密的 JSON 版本。我们必须确保另一个文件夹存储加密版本（例如，您可能已经有一个文件夹将它们存储在 GitHub 上）。否则，从 Chef 服务器导入加密版本到一个新目录中，例如，使用`-Fj`：

```
$ mkdir data_bags_encrypted 
$ knife data bag show aws us-east-1 -Fj > data_bags_encrypted/us-east-1.json 

```

现在我们可以像其他 VM 一样定义完整的 VM，只需修改数据包路径以适应加密版本：

```
  config.vm.define 'mysite', autostart: false do |mysite|
    mysite.vm.box = 'bento/centos-7.2'

    mysite.vm.provision :chef_zero do |chef|
      chef.cookbooks_path = 'cookbooks'
      chef.environments_path = 'environments'
      chef.data_bags_path = 'data_bags_encrypted'
      chef.roles_path = 'roles'
      chef.nodes_path = 'nodes'
      chef.environment = 'production'
      chef.add_recipe 'mysite::default'
    end

    mysite.berkshelf.berksfile_path = 'cookbooks/mysite/Berksfile'
    mysite.berkshelf.enabled = true
  end
```

为了确保我们的代码能够从头开始部署整个配方，请使用以下命令：

```
$ vagrant up mysite

```

将这些命令（及其销毁对应命令）放入 CI 或任何您喜欢的系统中，定期执行，如每天或每周，用于基础设施的每个自动化部分。通过这样做，您将始终确保在灾难来临时可以重新部署系统。

## 还有更多……

使用 Puppet，我们使用的所有示例都基于 Vagrant，并且可以轻松地从头重建节点。但在现实中，您可能不会在您的工作站上部署和维护运行 Vagrant 的生产系统。

然而，这些示例表明，通过简单的`vagrant up`命令可以模拟完整的基础设施，因此，可以轻松地将其放入任何 CI 系统中，以确保能够轻松重建生产系统。

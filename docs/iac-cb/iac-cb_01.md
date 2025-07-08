# 第一章：Vagrant 开发环境

本章将介绍以下内容：

+   添加一个 Ubuntu Xenial (16.04 LTS) 的 Vagrant 盒子

+   使用临时的 Ubuntu Xenial (16.04) 环境，几秒钟内完成

+   在 Vagrant 中启用 VirtualBox 客户端附加功能

+   使用 VMware 快速启动一个临时的 CentOS 7.x 环境

+   扩展 VMware 虚拟机的功能

+   启用多提供商的 Vagrant 环境

+   自定义 Vagrant 虚拟机

+   在 Vagrant 中使用 Docker

+   在 Vagrant 中使用 Docker 来运行 Ghost 博客并通过 NGINX 代理

+   使用 Vagrant 远程连接 AWS EC2 和 Docker

+   模拟动态多主机网络环境

+   使用 Vagrant 模拟一个三层架构的网络化应用

+   在使用 Laravel 时，将你的工作展示到局域网中

+   与全球共享你 Vagrant 环境的访问权限

+   使用 Vagrant 模拟 Chef 升级

+   使用 Ansible 和 Vagrant 创建一个 Docker 主机

+   在 CoreOS 上使用 Vagrant 运行 Docker 容器

# 介绍

Vagrant 是由 Hashicorp 开发的一个免费开源工具，旨在通过简单的 Ruby 代码在虚拟机内部构建可重复的开发环境。你可以将这个简单的文件与其他人、团队成员和外部贡献者共享，只要他们的笔记本电脑支持虚拟化，就能立即拥有一个可用的运行环境。这也意味着你可以使用一台 Mac 笔记本，通过一个简单的命令启动一个完全配置的 Linux 环境以供本地使用。无论每个人的本地机器是什么配置，大家都可以在相同的环境下工作。Vagrant 还非常适用于模拟完整的生产环境，涵盖多个机器和特定操作系统版本。Vagrant 兼容大多数虚拟机监控器，如 VMware、VirtualBox 或 Parallels，并且可以通过插件大幅扩展功能。

Vagrant 使用 *盒子* 运行。这些盒子实际上是已经打包好的虚拟机镜像，可以从 [`atlas.hashicorp.com/boxes/search`](https://atlas.hashicorp.com/boxes/search) 等网站获取，或者你也可以使用各种工具构建自己的盒子。

Vagrant 可以通过插件进行极大的扩展。几乎任何你能想到的功能都有相应的插件，而且大多数插件都由社区支持。从特定的操作系统到远程 IaaS 提供商，再到共享、缓存或快照、网络、测试功能，甚至是 Chef/Puppet 相关的特性，都可以通过 Vagrant 的插件实现。

所有可用插件的列表，包括所有 Vagrant 提供商，已在 Vagrant wiki 上提供，链接如下：[`github.com/mitchellh/vagrant/wiki/Available-Vagrant-Plugins`](https://github.com/mitchellh/vagrant/wiki/Available-Vagrant-Plugins)。

关于所有集成提供商的更多信息，可以在 Vagrant 官网找到：[`www.vagrantup.com/docs/providers/`](https://www.vagrantup.com/docs/providers/)。

你可以从 [`www.vagrantup.com/downloads.html`](https://www.vagrantup.com/downloads.html) 下载适用于你平台的 Vagrant 安装程序。

### 注意

本书使用的 Vagrant 版本是 Vagrant 1.8.4。

# 添加一个 Ubuntu Xenial (16.04 LTS) Vagrant 盒子

Vagrant 盒子通过名称进行引用，通常遵循 *用户名/盒子名* 的命名规则。由 *Ubuntu* 发布的 64 位 *Precise* 盒子将命名为 *ubuntu/precise64*，而 *centos/7* 盒子将始终是最新的 CentOS 7 官方盒子。

## 准备工作

要执行此步骤，你将需要以下内容：

+   使用免费开源的 Virtualbox 虚拟机管理程序的 Vagrant 安装实例

+   一个互联网连接

## 如何操作…

打开终端并输入以下代码：

```
$ vagrant box add ubuntu/xenial64
==> box: Loading metadata for box 'ubuntu/xenial64'
 box: URL: https://atlas.hashicorp.com/ubuntu/xenial64
==> box: Adding box 'ubuntu/xenial64' (v20160815.0.0) for provider: virtualbox
 box: Downloading: https://atlas.hashicorp.com/ubuntu/boxes/xenial64/versions/20160815.0.0/providers/virtualbox.box
==> box: Successfully added box 'ubuntu/xenial64' (v20160815.0.0) for 'virtualbox'!

```

## 它是如何工作的…

Vagrant 知道在哪里查找请求的盒子在 Atlas 服务中的最新版本，并会自动通过互联网下载它。所有的盒子默认存储在`~/.vagrant.d/boxes`目录下。

## 还有更多内容…

如果你有兴趣创建自己的基础 Vagrant 盒子，请参考 Packer ([`www.packer.io/`](https://www.packer.io/)) 和 Chef Bento 项目 ([`chef.github.io/bento/`](http://chef.github.io/bento/))。

# 几秒钟内使用一次性 Ubuntu Xenial (16.04) 系统

我们希望尽可能快速地访问并使用 Ubuntu Xenial 系统（16.04 LTS）。

为了做到这一点，Vagrant 使用名为 `Vagrantfile` 的文件来描述 Vagrant 基础设施。这个文件实际上是纯 Ruby 代码，Vagrant 读取它来管理你的环境。所有与 Vagrant 相关的操作都在一个类似下面的代码块内完成：

```
Vagrant.configure("2") do |config|
  # all your Vagrant configuration here
end
```

## 准备工作

要执行此步骤，你将需要以下内容：

+   一个工作的 Vagrant 安装实例

+   一个工作的 VirtualBox 安装实例

+   一个互联网连接

## 如何操作…

1.  创建一个项目文件夹：

    ```
    $ mkdir vagrant_ubuntu_xenial_1 && cd $_
    ```

1.  使用你最喜欢的编辑器，创建这个非常简单的 Vagrantfile 来启动 ubuntu/`xenial64` 盒子：

    ```
    Vagrant.configure("2") do |config|
      config.vm.box = "ubuntu/xenial64"
    end
    ```

1.  现在你可以显式使用 Virtualbox 虚拟机管理程序执行 Vagrant：

    ```
    $ vagrant up --provider=virtualbox
    ```

1.  几秒钟内，你将在主机上运行一个 Ubuntu 16.04 Vagrant 盒子，你可以对它做任何你想做的事。例如，可以通过发出以下 `vagrant` 命令使用 **安全外壳** (**SSH**) 登录并正常使用系统：

    ```
    $ vagrant ssh
    Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-34-generic x86_64)
    […]
    ubuntu@ubuntu-xenial:~$ hostname
    ubuntu-xenial
    ubuntu@ubuntu-xenial:~$ free -m
    ubuntu@ubuntu-xenial:~$ cat /proc/cpuinfo

    ```

1.  当你完成使用 Vagrant 虚拟机时，你可以直接销毁它：

    ```
    $ vagrant destroy
    ==> default: Forcing shutdown of VM...
    ==> default: Destroying VM and associated drives...

    ```

    另外，我们可以通过使用 vagrant halt 停止 Vagrant 虚拟机，目的是稍后在当前状态下重新启动它：

    **$ vagrant halt**

## 它是如何工作的…

当你启动 Vagrant 时，它读取了 Vagrantfile，要求运行特定的盒子（Ubuntu Xenial）。如果你之前已添加过，它会通过默认的虚拟机管理程序（在这种情况下是 VirtualBox）立即启动；如果是新的盒子，它会自动为你下载。它创建了所需的虚拟网络接口，然后 Ubuntu 虚拟机获得了一个私有 IP 地址。Vagrant 负责配置 **SSH**，通过暴露一个可用的端口并插入默认密钥，这样你就可以通过 SSH 无问题地登录。

# 在 Vagrant 中启用 VirtualBox 客户端附加组件

VirtualBox Guest Additions 是一组驱动程序和应用程序，旨在部署到虚拟机中，以提高性能并启用诸如文件夹共享等功能。虽然可以将 **Guest Additions** 直接包含在 box 中，但并不是所有找到的 box 都包含它，即使有，也可能会很快过时。

解决方案是通过插件按需自动部署 VirtualBox Guest Additions。

### 注意

使用这个插件的缺点是，Vagrant box 可能需要更长的时间来启动，因为它可能需要下载并安装正确的 Guest Additions。

## 准备就绪

要执行此食谱，你将需要以下内容：

+   一个正常工作的 Vagrant 安装

+   一个正常工作的 VirtualBox 安装

+   一个互联网连接

+   来自上一食谱的 Vagrantfile

## 如何操作……

按照以下步骤在 Vagrant 中启用 VirtualBox Guest Additions：

1.  安装 `vagrant-vbguest` 插件：

    ```
    $ vagrant plugin install vagrant-vbguest
    Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
    Installed the plugin 'vagrant-vbguest (0.13.0)'!

    ```

1.  确认插件已安装：

    ```
    $ vagrant plugin list
    vagrant-vbguest (0.13.0)

    ```

1.  启动 Vagrant 并查看 VirtualBox Guest Additions 是否已安装：

    ```
    $ vagrant up
    […]
    Installing Virtualbox Guest Additions 5.0.26
    […]
    Building the VirtualBox Guest Additions kernel modules
     ...done.
    Doing non-kernel setup of the Guest Additions …done.

    ```

1.  现在，可能你不想每次启动 Vagrant box 时都做这个，因为它需要时间和带宽，或者因为你的宿主机 VirtualBox 版本和 Vagrant box 中已经安装的版本之间的小差异对你来说不是问题。在这种情况下，你可以直接从 Vagrantfile 中告诉 Vagrant 禁用自动更新功能：

    ```
    config.vbguest.auto_update = false
    ```

1.  保持代码与没有安装此插件的人兼容的更好方法是，仅在 Vagrant 自身找到插件时才使用此插件配置：

    ```
    if Vagrant.has_plugin?("vagrant-vbguest") then
        config.vbguest.auto_update = false
    end
    ```

1.  完整的 Vagrantfile 现在看起来是这样的：

    ```
    Vagrant.configure("2") do |config|
        config.vm.box = "ubuntu/xenial64"
        if Vagrant.has_plugin?("vagrant-vbguest") then
              config.vbguest.auto_update = false
        end
    end
    ```

## 它是如何工作的……

Vagrant 插件会自动从供应商的网站安装，并在你的系统上全局可用，供你运行的所有其他 Vagrant 环境使用。一旦虚拟机准备就绪，插件会检测操作系统，决定是否需要安装 Guest Additions，如果需要，它将安装必要的工具（编译器、内核头文件和库），最后下载并安装相应的 Guest Additions。

## 还有更多……

使用 Vagrant 插件还扩展了你可以通过 Vagrant CLI 做的事情。在 VirtualBox Guest Additions 插件的情况下，你可以做很多事情，如状态检查、管理安装等：

```
$ vagrant vbguest --status
[default] GuestAdditions 5.0.26 running --- OK.

```

该插件可以通过 Vagrant 直接调用；这里它触发了虚拟机中 Guest Additions 的安装：

```
$ vagrant vbguest --do install

```

# 使用一个一次性的 CentOS 7.x 版本和 VMware，几秒钟内即可完成

Vagrant 支持通过 Vagrant 商店中的官方插件支持 VMware Workstation 和 VMware Fusion（[`www.vagrantup.com/vmware`](https://www.vagrantup.com/vmware)）。按照官网的说明安装插件。

Vagrant 镜像依赖于虚拟化程序——VirtualBox 镜像不能在 VMware 上运行。你需要为每个你选择使用的虚拟化程序使用专门的镜像。例如，Ubuntu 官方发布的镜像仅提供 VirtualBox 镜像。如果你尝试使用一个为另一虚拟化程序构建的镜像来创建 Vagrant 镜像，你将遇到错误。

## 准备工作

要逐步执行此食谱，你将需要以下内容：

+   一个有效的 Vagrant 安装

+   一个有效的 VMware Workstation（PC）或 Fusion（Mac）安装

+   一个有效的 Vagrant VMware 插件安装

+   一个互联网连接

## 如何操作…

Chef Bento 项目提供了多种多提供商镜像，我们可以使用。例如，使用这个最简单的 Vagrantfile 来运行 CentOS 7.2 与 Vagrant（bento/centos-7.2）：

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
end
```

启动你的 CentOS 7.2 虚拟环境，并指定你要运行的虚拟化程序：

```
$ vagrant up --provider=vmware_fusion
$ vagrant ssh

```

你现在正在使用 VMware 运行 CentOS 7.2 Vagrant 镜像！

## 它是如何工作的…

Vagrant 通过插件扩展其使用和功能。在这种情况下，Vagrant 的 VMware 插件将所有虚拟化功能委托给 VMware 安装，从而不再需要 VirtualBox。

## 还有更多…

如果 VMware 是你主要的虚拟化程序，你很快会厌倦每次都在命令行中指定提供商。通过将 `VAGRANT_DEFAULT_PROVIDER` 环境变量设置为对应的插件，你将再也无需指定提供商，VMware 将成为默认：

```
$ export VAGRANT_DEFAULT_PROVIDER=vmware_fusion
$ vagrant up

```

## 另见

+   Chef Bento 项目，地址为 [`chef.github.io/bento/`](http://chef.github.io/bento/)

+   一个社区版的 VMware vSphere 插件，地址为 [`github.com/nsidc/vagrant-vsphere`](https://github.com/nsidc/vagrant-vsphere)

+   一个社区版的 VMware vCloud Director 插件，地址为 [`github.com/frapposelli/vagrant-vcloud`](https://github.com/frapposelli/vagrant-vcloud)

+   一个社区版的 VMware vCenter 插件，地址为 [`github.com/frapposelli/vagrant-vcenter`](https://github.com/frapposelli/vagrant-vcenter)

+   一个社区版的 VMware vCloud Air 插件，地址为 [`github.com/frapposelli/vagrant-vcloudair`](https://github.com/frapposelli/vagrant-vcloudair)

# 扩展 VMware 虚拟机功能

Vagrant 镜像的硬件规格因镜像而异，因为它们在创建时指定。然而，它并非永远固定：这只是默认行为。你可以在 Vagrantfile 中设置需求，以便保持一个小的 Vagrant 镜像，并按需调整。

## 准备工作

要逐步执行此食谱，你将需要以下内容：

+   一个有效的 Vagrant 安装

+   一个有效的 VMware Workstation（PC）或 Fusion（Mac）安装

+   一个有效的 Vagrant VMware 插件安装

+   一个互联网连接

+   使用 bento/centos72 镜像的 Vagrantfile，参见前面的食谱

## 如何操作…

VMware 提供商可以在以下配置块中进行配置：

```
# VMware Fusion configuration
config.vm.provider "vmware_fusion" do |vmware|
  # enter all the vmware configuration here
end

# VMware Workstation configuration
config.vm.provider "vmware_workstation" do |vmware|
  # enter all the vmware configuration here
end
```

如果配置相同，你将会有很多重复的代码。利用 Vagrantfile 的 Ruby 特性，使用一个简单的循环来迭代这两个值：

```
["vmware_fusion", "vmware_workstation"].each do |vmware|
  config.vm.provider vmware do |v|
    # enter all the vmware configuration here
  end
end
```

我们默认的 Bento CentOS 7.2 镜像只有 512 MB 内存和一个 CPU。为了更好的性能，我们将通过 `vmx["numvcpus"]` 和 `vmx["memsize"]` 键来将其翻倍：

```
  ["vmware_fusion", "vmware_workstation"].each do |vmware|
    config.vm.provider vmware do |v|
      v.vmx["numvcpus"] = "2"
      v.vmx["memsize"] = "1024"
    end
  end
```

启动或重启你的 Vagrant 虚拟机以应用更改：

```
$ vagrant up
[…]

```

你的 box 现在使用了两个 CPU 和 1 GB 内存。

## 它是如何工作的…

虚拟机配置是 Vagrant 启动前的最后一步。这里，它仅仅告诉 VMware 为它启动的虚拟机分配两个 CPU 和 1 GB 的内存，就像你在软件内部手动操作时所做的一样。

## 还有更多…

Vagrant 的作者可能会在未来某个时刻将这两个插件合并成一个。目前，插件的 4.x 版本仍然是分开的。

VMware 对 VMX 格式的文档并不完善。有关 VMX 配置的可能键和值，可以在 VMware Inc. 的大多数文档中找到。

# 启用多提供商 Vagrant 环境

你可能在笔记本上运行 VMware，但你的同事可能没有。或者，你希望大家可以选择，或者你只是希望两种环境都能工作！我们将看到如何构建一个单一的 Vagrantfile 来支持它们所有。

## 准备工作

要按照这个步骤进行，你需要以下内容：

+   一个有效的 Vagrant 安装

+   一个有效的 VirtualBox 安装

+   一个有效的 VMware Workstation（PC）或 Fusion（Mac）安装

+   一个有效的 Vagrant VMware 插件安装

+   一条互联网连接

+   使用 bento/centos72 box 的上一个配方中的 Vagrantfile

## 如何操作…

一些 Vagrant box 可以用于多个虚拟化平台，例如我们之前使用的 CentOS 7 Bento box。这样，我们可以简单地选择使用哪个。

让我们从我们之前的 Vagrantfile 开始，其中包含了对 VMware 的自定义配置：

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
  ["vmware_fusion", "vmware_workstation"].each do |vmware|
    config.vm.provider vmware do |v|
      v.vmx["numvcpus"] = "2"
      v.vmx["memsize"] = "1024"
    end
  end
end
```

我们如何在 VirtualBox 上添加与 VMware 相同的配置？以下是在 Vagrantfile 中类似地自定义 VirtualBox 的方法：

```
  config.vm.provider :virtualbox do |vb|
    vb.memory = "1024"
    vb.cpus = "2"
  end
```

将此添加到你当前的 Vagrantfile，重新加载后，你将从你的虚拟化平台（无论是 VMware 还是 VirtualBox）获取所请求的资源。

这很好，但我们仍然在重复使用这些值，未来可能会导致错误、遗漏或问题。让我们再次利用 Vagrantfile 的 Ruby 特性，在文件的顶部声明一些有意义的变量：

```
vm_memory = 1024
vm_cpus = 2
```

现在，将四个值替换为它们的变量名，你就完成了：你正在集中管理你所使用和分发的 Vagrant 环境的特性，无论你使用的是哪个虚拟化平台。

## 它是如何工作的…

Vagrantfile 作为纯 Ruby 文件这一事实，有助于通过简单地设置变量来创建强大而动态的配置，后续可以在所有提供商中使用这些变量。

# 自定义 Vagrant 虚拟机

Vagrant 通过 Vagrantfile 支持许多配置选项。以下是日常使用中最有用的一些。

## 准备工作

要按照这个步骤进行，你需要以下内容：

+   一个有效的 Vagrant 安装（带有虚拟化平台）

+   一条互联网连接

+   使用 bento/centos72 box 的上一个配方中的 Vagrantfile

## 如何做到…

以下是一些可能的 Vagrant 虚拟机自定义选项。

### 设置主机名

如果你想直接从 Vagrant 指定虚拟机名称，只需添加以下内容：

```
config.vm.hostname = "vagrant-lab-1"
```

这还将向 `/etc/host` 文件中添加一个包含主机名的条目。

### 禁用启动时的新盒子版本检查

你可能正在使用较慢的互联网连接，或者你知道你确实想使用当前安装的盒子，或者你可能很急，只想完成工作；你可以通过添加以下内容，去除启动时检查新版本盒子的选项：

```
config.vm.box_check_update = false
```

### 使用特定的盒子版本

如果你知道自己想使用某个特定版本的盒子（可能出于调试目的或合规性需求），而不是最新版本，你可以简单地按如下方式声明：

```
config.vm.box_version = "2.2.9"
```

### 向用户显示信息性消息

一个有用的功能是向启动 Vagrant 盒子的用户显示一些基本但相关的信息，比如使用说明或连接信息。别忘了转义特殊字符。由于这是 Ruby，你可以访问所有可用的变量，因此信息可以变得更加动态并对用户更有用：

```
config.vm.post_up_message = "Use \"vagrant ssh\" to log into the box. This VM uses #{vm_cpus} CPUs and #{vm_memory}MB of RAM."
```

### 指定最低 Vagrant 版本

Vagrant 经常更新，并定期添加新功能。如果你使用了一个已知只有在特定版本之后才能正常工作的功能，最好在 Vagrantfile 中声明，以便使用旧版本的人知道他们需要更新：

```
Vagrant.require_version ">= 1.8.0"
```

# 在 Vagrant 中使用 Docker

开发环境通常会混合使用虚拟机和 Docker 容器。虚拟机包含运行完整操作系统所需的所有内容，如内存、CPU、内核和所有必需的库，而容器则更加轻量，可以与宿主机共享所有这些资源，同时通过名为 cgroups 的特殊内核功能保持良好的隔离。Docker 容器帮助开发者使用、共享和发布一个包含运行应用程序所需的一切的*包*。在这里，我们将展示如何使用 Vagrant 启动容器。由于 Docker 在 Linux 主机和其他平台上的使用略有不同，本文所参考的是原生 Docker 平台——Linux。

## 准备工作

要逐步执行这个过程，你将需要以下内容：

+   一个工作正常的 Vagrant 安装（无需虚拟化管理程序）

+   一个工作正常的 Docker 安装和基本的 Docker 知识

+   一个互联网连接

## 如何做到…

我们将看到如何使用 Docker 作为提供者，在 Vagrant 中使用、访问和操作一个 NGINX 容器。

### 通过 Vagrant 使用 NGINX Docker 容器

从最简单的 Vagrantfile 开始，使用 `nginx:stable` 容器与 Docker Vagrant 提供者：

```
Vagrant.configure("2") do |config|
  config.vm.hostname = "vagrant-docker-1"
  config.vm.post_up_message = "HTTP access: http://localhost/"
  config.vm.provider "docker" do |docker|
      docker.image = "nginx:stable"
  end
end
```

只需使用以下代码启动它：

```
$ vagrant up --provider=docker
Bringing machine 'default' up with 'docker' provider...
==> default: Creating the container...
[…]
==> default: HTTP access: http://localhost/

```

通过在 Vagrantfile 顶部设置一个简单的 Ruby 环境访问代码，来消除在命令行中指定提供者的需求：

```
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'docker'
```

现在你可以分发你的 Vagrantfile，而不必担心别人忘记明确指定 Docker 提供者。

### 在 Vagrant 中暴露 Docker 端口

好吧，之前的示例并不太有用，因为我们没有暴露任何端口。现在让我们告诉 Vagrant 暴露 Docker 容器的 HTTP（TCP/80）端口到主机的 HTTP（TCP/80）端口：

```
  config.vm.provider "docker" do |docker|
      docker.image = "nginx:stable"
      docker.ports = ['80:80']
  end
```

重启 Vagrant 并验证你是否能访问到你的 NGINX 容器：

```
$ curl http://localhost/

```

### 通过 Vagrant 共享 Docker 文件夹

那么，如果共享一个本地文件夹，让你可以在笔记本上编码并查看 Vagrant 环境处理后的结果呢？默认的 NGINX 配置会从`/usr/share/nginx/html`读取文件。我们把自己的`index.html`放在这里。

创建一个简单的`src/index.html`文件，包含一些文本：

```
$ mkdir src; echo "<h1>Hello from Docker via Vagrant<h1>" > src/index.html

```

将 Docker 卷配置添加到 Vagrant 中的 Docker 提供者块：

```
  config.vm.provider "docker" do |docker|
      docker.image = "nginx:stable"
      docker.ports = ['80:80']
      docker.volumes = ["#{Dir.pwd}/src:/usr/share/nginx/html"]
  end
```

### 注意

`#{Dir.pwd}`是 Ruby 中用于查找当前目录的命令，使用它可以避免硬编码路径，使得代码具有很好的可分发性。

重启 Vagrant 环境并查看结果：

```
$ curl http://localhost
<h1>Hello from Docker via Vagrant<h1>

```

### 注意

在启用了 SELinux 的系统上，你可能需要做一些配置，这超出了本书的范围。我们鼓励你使用 SELinux 来确保 Docker 系统的安全，但要禁用 SELinux，只需键入以下命令：

```
$ sudo setenforce 0

```

## 还有更多内容…

你可以选择不使用本地或默认的 Docker 安装，而是使用专用的虚拟机，也许是为了模拟生产环境或特定的操作系统（例如 CoreOS）。在这种情况下，你可以像下面这样指定一个专用的 Vagrantfile：

```
config.vm.provider "docker" do |docker|
docker.vagrant_vagrantfile = "docker_host/Vagrantfile"
end
```

# 在 Vagrant 中使用 Docker 为 NGINX 后面的 Ghost 博客提供服务

在 Docker 中使用 Vagrant 可以更有效地模拟传统的设置，比如应用程序后面有负载均衡器或反向代理。我们已经设置了 NGINX，那么可以用它作为前端反向代理，并将像 Ghost 这样的博客引擎放在后面吗？我们最后将展示如何使用 docker-compose 做类似的事情。

## 准备工作

要执行这个操作，你需要以下条件：

+   一个有效的 Vagrant 安装（不需要虚拟化管理程序）

+   一个有效的 Docker 安装和基本的 Docker 知识

+   一个互联网连接

## 如何操作…

之前的示例只允许启动一个容器，这有点遗憾，因为 Docker 的强大之处就在于可以运行多个容器。让我们定义多个容器，并从创建一个`front`容器（我们之前的 NGINX）开始：

```
  config.vm.define "front" do |front|
    front.vm.provider "docker" do |docker|
      docker.image = "nginx:stable"
      docker.ports = ['80:80']
      docker.volumes = ["#{Dir.pwd}/src:/usr/share/nginx/html"]
    end
  end
```

现在，如何创建一个应用程序容器呢，也许是像 Ghost 这样的博客引擎？Ghost 在 Docker Hub 上发布了一个现成的容器，所以我们就使用它（本文写作时为版本 0.9.0），并将 TCP/2368 上监听的应用程序容器暴露到 TCP/8080 上：

```
  config.vm.define "app" do |app|
    app.vm.provider "docker" do |docker|
      docker.image = "ghost:0.9.0"
      docker.ports = ['8080:2368']
    end
  end
```

检查是否可以在`http://localhost:8080`访问博客，在`http://localhost`访问 NGINX：

```
$ curl -IL http://localhost:8080
HTTP/1.1 200 OK
X-Powered-By: Express
[…]

$ curl -IL http://localhost
HTTP/1.1 200 OK
Server: nginx/1.10.1

```

现在让我们使用 NGINX 来实现它的目的——为应用程序提供服务。配置 NGINX 作为反向代理超出了本书的范围，所以只需使用以下简单配置，将其保存为工作文件夹根目录下的`nginx.conf`文件：

```
server {
  listen 80;
  location / {
    proxy_set_header   X-Real-IP $remote_addr;
    proxy_set_header   Host      $http_host;
    proxy_pass         http://app:2368;
  }
}
```

更改 Vagrant 中`front`容器的配置，使用这个配置，删除旧的`index.html`，因为我们不再使用它，并将此容器与`app`容器连接：

```
  config.vm.define "front" do |front|
    front.vm.provider "docker" do |docker|
      docker.image = "nginx:stable"
      docker.ports = ['80:80']
      docker.volumes = ["#{Dir.pwd}/nginx.conf:/etc/nginx/conf.d/default.conf"]
      docker.link("app:app")
    end
  end
```

链接`app`容器使其可以被`front`容器访问，因此现在无需直接暴露 Ghost 博客容器，让我们通过反向代理将其变得更加简洁和安全：

```
  config.vm.define "app" do |app|
    app.vm.provider "docker" do |docker|
      docker.name = "app"
      docker.image = "ghost:0.9.0"
    end
  end
```

我们快完成了！但是这个设置最终会失败，原因很简单：我们的系统太快了，Vagrant 默认会并行启动虚拟机，也会并行启动容器。容器启动得太快，以至于`app`容器可能在 NGINX 启动时还没有准备好。为了确保顺序启动，请在 Vagrantfile 顶部使用`VAGRANT_NO_PARALLEL`环境变量：

```
ENV['VAGRANT_NO_PARALLEL'] = 'true'
```

现在你可以浏览到`http://localhost/admin`并开始使用你在容器中的 Ghost 博客，它位于 NGINX 反向代理容器后面，整个过程由 Vagrant 管理！

## 还有更多……

你可以通过 Vagrant 直接访问容器日志：

```
$ vagrant docker-logs --follow
==> app: > ghost@0.9.0 start /usr/src/ghost
==> app: > node index
==> app: Migrations: Creating tables...
[…]
==> front: 172.17.0.1 - - [21/Aug/2016:10:55:08 +0000] "GET / HTTP/1.1" 200 1547 "-" "Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:48.0) Gecko/20100101 Firefox/48.0" "-"
==> app: GET / 200 113.120 ms - -
[…]

```

### Docker Compose 等效方式

Docker Compose 是一个工具，用于从单一的 YAML 文件编排多个容器并管理 Docker 功能。所以如果你更熟悉 Docker Compose，或者你希望用这个工具做类似的事情，以下是`docker-compose.yml`文件中的代码：

```
version: '2'
services:
  front:
    image: nginx:stable
    volumes:
      - "./nginx.conf:/etc/nginx/conf.d/default.conf"
    restart: always
    ports:
      - "80:80"
    depends_on:
      - app
    links:
      - app
  app:
    image: ghost:0.9.0
    restart: always
```

### 注意

记住，使用 Vagrant，你可以混合虚拟机和 Docker 容器，而使用 docker-compose 则不行。

# 使用 Vagrant 远程连接 AWS EC2 和 Docker

Vagrant 的另一个强大用法是与远程 IaaS 资源结合使用，如 Amazon EC2。Amazon Web Services 弹性计算云（EC2）和类似的基础设施即服务提供商，如 Google Cloud、Azure 或 Digital Ocean 等，出售具有不同计算能力和网络带宽的虚拟机。你不一定总是能在你的笔记本上获得所需的全部 CPU 和内存，或者你需要为某个任务提供特定的计算能力，或者你只是想复制现有生产环境的一部分：这是你如何利用 Vagrant 与 Amazon EC2 结合使用的方式。

在这里，我们将在 AWS EC2 上使用 Ubuntu Xenial 16.04 和 Docker 部署一个带有 NGINX 反向代理的 Ghost 博客！这是为了模拟应用程序的实际部署，这样你就可以看到它在真实条件下是否能正常工作。

## 准备工作

要按照这个教程操作，你需要以下内容：

+   一个正常工作的 Vagrant 安装（不需要虚拟化管理程序）

+   一个 Amazon EC2 账户（如果你还没有，可以在[`aws.amazon.com/`](https://aws.amazon.com/)免费创建一个），需要有效的访问密钥、名为*iac-lab*的密钥对、一个名为*iac-lab*的安全组，允许至少 HTTP 端口和 SSH 访问。

+   需要一个互联网连接

## 如何操作…

首先安装插件：

```
$ vagrant plugin install vagrant-aws

```

这个插件的一个要求是必须有一个什么都不做的虚拟 Vagrant box：

```
$ vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box

```

记得我们在前面的教程中如何配置 Docker 提供者吗？这没什么不同：

```
config.vm.provider :aws do |aws, override|
  # AWS Configuration
  override.vm.box = "dummy"
end
```

然后，定义一个应用程序虚拟机，将包括指定其使用的提供商（在我们这里是 AWS）、**Amazon 机器映像**（**AMI**）（在我们这里是 Ubuntu 16.04 LTS），以及一个我们创意命名为 `script.sh` 的配置脚本。

您可以在 [`cloud-images.ubuntu.com/locator/ec2/`](http://cloud-images.ubuntu.com/locator/ec2/) 找到其他 AMI ID：

```
config.vm.define "srv-1" do |config|
    config.vm.provider :aws do |aws|
      aws.ami = "ami-c06b1eb3"
    end
    config.vm.provision :shell, :path => "script.sh"
end
```

那么，我们需要填写哪些与 AWS 相关的信息，以便 Vagrant 能够在 AWS 上启动服务器呢？

我们需要 AWS 访问密钥，最好从环境变量中获取，这样就不需要在 Vagrantfile 中硬编码它们：

```
aws.access_key_id = ENV['AWS_ACCESS_KEY_ID']
aws.secret_access_key = ENV['AWS_SECRET_ACCESS_KEY']
```

指定您希望实例启动的区域和可用区：

```
aws.region = "eu-west-1"
aws.availability_zone = "eu-west-1a"
```

包括实例类型；在这里，我们选择了 AWS 免费套餐中包含的类型，因此在新帐户下不会花费您一分钱：

```
aws.instance_type = "t2.micro"
```

指定此实例将所在的安全组（您可以根据需要调整要求）：

```
aws.security_groups = ['iac-lab']
```

指定 AWS 密钥对名称，并覆盖默认的 SSH 用户名和密钥：

```
aws.keypair_name = "iac-lab"
override.ssh.username = "ubuntu"
override.ssh.private_key_path = "./keys/iac-lab.pem"
```

在某些情况下，使用 Vagrant 和 AWS EC2 时，可能会遇到 NFS 的 bug，因此我选择禁用此功能：

```
override.nfs.functional = false
```

最后，为实例打标签是一个好习惯，这样您以后就能查找它们的来源：

```
aws.tags = {
  'Name'   => 'Vagrant'
}
```

添加一个简单的 shell 脚本来安装 Docker 和 `docker-compose`，然后执行 docker-compose 文件：

```
#!/bin/sh
# install Docker
curl -sSL https://get.docker.com/ | sh
# add ubuntu user to docker group
sudo usermod -aG docker ubuntu
# install docker-compose
curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# execute the docker compose file
cd /vagrant
docker-compose up -d

```

包含前面配方中的 NGINX 配置和 `docker-compose.yml` 文件，您就可以开始了：

```
$ vagrant up
Bringing machine 'srv-1' up with 'aws' provider...
[…]
==> srv-1: Launching an instance with the following settings...
==> srv-1:  -- Type: t2.micro
==> srv-1:  -- AMI: ami-c06b1eb3
==> srv-1:  -- Region: eu-west-1
[…]
==> srv-1: Waiting for SSH to become available...
==> srv-1: Machine is booted and ready for use!
[…]
==> srv-1:  docker version
[…]
==> srv-1: Server:
==> srv-1:  Version:      1.12.1
[…]
==> srv-1: Creating vagrant_app_1
==> srv-1: Creating vagrant_front_1

```

打开浏览器并访问 `http://a.b.c.d/`（使用 EC2 实例的公网 IP），您将看到 Ghost 博客通过 NGINX 反向代理在 Docker 容器中运行，使用 Vagrant 在 Amazon EC2 上部署。

这种设置的常见用途是让开发人员在接近真实生产环境的条件下测试应用程序，可能是向远程产品经理展示新功能，复制仅在此设置中出现的 bug，或者在某个 CI 阶段进行测试。一旦构建了 Docker 容器，可以在 EC2 上进行烟雾测试，然后再继续其他操作。

# 模拟动态的多个主机网络

当用于模拟网络中的多个主机时，Vagrant 也非常有用。这样，您可以在同一个私有网络中拥有能够相互通信的完整系统，轻松测试系统间的连接性。

## 准备开始

要逐步完成此配方，您需要以下内容：

+   一个正常工作的 Vagrant 安装

+   一个正常工作的 VirtualBox 安装

+   一个互联网连接

## 如何做到这一点……

下面是如何创建一个 CentOS 7.2 虚拟机，配置 512 MB 内存和一个 CPU，位于私有网络并使用固定 IP 地址 192.168.50.11，输出一个简单的 shell：

```
vm_memory = 512
vm_cpus = 1

Vagrant.configure("2") do |config|

  config.vm.box = "bento/centos-7.2"

  config.vm.provider :virtualbox do |vb|
    vb.memory = vm_memory
    vb.cpus = vm_cpus
  end

   config.vm.define "srv-1" do |config|
     config.vm.provision :shell, :inline => "ip addr | grep \"inet\" | awk '{print $2}'"
     config.vm.network "private_network", ip: "192.168.50.11", virtualbox__intnet: "true"
   end
end
```

要向该网络添加新机器，我们可以简单地复制 `srv-1` 机器定义，如下所示：

```
config.vm.define "srv-2" do |config|
     config.vm.provision :shell, :inline => "ip addr | grep \"inet\" | awk '{print $2}'"
     config.vm.network "private_network", ip: "192.168.50.12", virtualbox__intnet: "true"
end
```

这不是很符合 DRY 原则，所以让我们利用 Vagrantfile 的 Ruby 特性创建一个循环，动态且简洁地创建我们需要的虚拟机数量。

首先，声明一个变量来指定我们想要的虚拟机数量（`2`）：

```
vm_num = 2
```

然后遍历该值，以生成 IP 和主机名的值：

```
(1..vm_num).each do |n|
    # a lan lab in the 192.168.50.0/24 range
    lan_ip = "192.168.50.#{n+10}"
    config.vm.define "srv-#{n}" do |config|
      config.vm.provision :shell, :inline => "ip addr | grep \"inet\" | awk '{print $2}'"
      config.vm.network "private_network", ip: lan_ip, virtualbox__intnet: "true"
    end
  end
```

这将创建两个虚拟机（`srv-1`的 IP 为`192.168.50.11`，`srv-2`的 IP 为`192.168.50.12`）在同一内部网络中，这样它们就可以互相通信。

现在你只需简单地更改`vm_num`的值，就能轻松在几秒钟内启动新的虚拟机。

## 还有更多内容…

我们还可以选择进一步扩展，使用以下克隆和网络功能。

### 通过链接克隆加速部署

链接克隆是一个特性，允许基于现有的磁盘映像创建新的虚拟机，而不需要重复所有内容。每个虚拟机只存储其增量状态，从而实现非常快速的虚拟机启动时间。

由于我们启动了多个虚拟机，你可以选择启用链接克隆以加速过程：

```
config.vm.provider :virtualbox do |vb|
    vb.memory = vm_memory
    vb.cpus = vm_cpus
 vb.linked_clone = true
end
```

### 使用命名的 NAT 网络

VirtualBox 提供了一个选项，让你定义自己的网络以便进一步参考或重用。你可以在**首选项** | **网络** | **NAT 网络**下进行配置。幸运的是，Vagrant 也可以与这些命名的 NAT 网络一起使用。为了测试此功能，你可以在 VirtualBox 中创建一个网络（如 iac-lab），并将其分配给网络`192.168.50.0/24`。

只需更改前面 Vagrantfile 中的网络配置，即可在此特定网络中启动虚拟机：

```
config.vm.network "private_network", ip: lan_ip, virtualbox__intnet: "iac-lab"

```

# 使用 Vagrant 模拟一个网络化的三层架构应用

Vagrant 是一个非常棒的工具，可以帮助模拟孤立网络中的系统，使我们能够轻松地模拟生产环境中的架构。多层架构的核心理念是将应用程序的不同元素的逻辑和执行分开，而不是将所有内容集中在一个地方。常见的模式是首先获取一个处理常见用户请求的层，第二层执行应用程序的任务，第三层存储和检索数据，通常是从数据库中获取。

在此模拟中，我们将使用传统的三层架构，每一层都在自己的独立网络上运行 CentOS 7 虚拟机：

+   **前端**：NGINX 反向代理

+   **应用**：一个在两个节点上运行的 Node.js 应用

+   **数据库**：Redis

| 虚拟机名称 | front_lan IP | app_lan IP | db_lan IP |
| --- | --- | --- | --- |
| front-1 | 10.10.0.11/24 | 10.20.0.101/24 | N/A |
| app-1 | N/A | 10.20.0.11/24 | 10.30.0.101/24 |
| app-2 | N/A | 10.20.0.12/24 | 10/30.0.102/24 |
| db-1 | N/A | N/A | 10.30.0.11/24 |

你将访问反向代理（NGINX），只有它能够与应用服务器（Node.js）进行通信，而应用服务器则是唯一能够连接到数据库的。

## 准备工作

要完成这个步骤，你需要以下内容：

+   一个正常工作的 Vagrant 安装

+   一个正常工作的 VirtualBox 安装

+   一条互联网连接

## 如何实现…

按照以下步骤使用 Vagrant 模拟一个网络化的三层架构应用。

### 第三层 – 数据库

数据库位于一个名为 db_lan 的私有网络中，IP 地址为 10.30.0.11/24。

该应用程序将使用一个简单的 Redis 安装。安装和配置 Redis 超出了本书的范围，因此我们将尽量简化（安装它，配置它监听局域网端口而不是 127.0.0.1，并启动它）：

```
  config.vm.define "db-1" do |config|
    config.vm.hostname = "db-1"
    config.vm.network "private_network", ip: "10.30.0.11", virtualbox__intnet: "db_lan"
    config.vm.provision :shell, :inline => "sudo yum install -q -y epel-release"
    config.vm.provision :shell, :inline => "sudo yum install -q -y redis"
    config.vm.provision :shell, :inline => "sudo sed -i 's/bind 127.0.0.1/bind 127.0.0.1 10.30.0.11/' /etc/redis.conf"
    config.vm.provision :shell, :inline => "sudo systemctl enable redis"
    config.vm.provision :shell, :inline => "sudo systemctl start redis"
  end
```

### 第二级：应用程序服务器

这一层是我们应用程序所在的地方，背后是一个应用程序（Web）服务器。该应用程序可以连接到数据库层，并通过第一级代理服务器提供给最终用户。通常这就是应用程序完成所有逻辑处理的地方。

#### Node.js 应用程序

这将通过我能够编写的最简单的 Node.js 代码来模拟，展示服务器主机名（文件名为`app.js`）。

首先，它将在`db_lan`网络上创建与 Redis 服务器的连接：

```
#!/usr/bin/env node
var os = require("os");
var redis = require('redis');
var client = redis.createClient(6379, '10.30.0.11');
client.on('connect', function() {
    console.log('connected to redis on '+os.hostname()+' 10.30.0.11:6379');
});
```

如果一切顺利，它将创建一个在`:8080`上监听的 HTTP 服务器，显示服务器的主机名：

```
var http = require('http');
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Running on '+os.hostname()+'\n');
}).listen(8080);
console.log('HTTP server listening on :8080');
```

启动应用程序，这是最简单的`systemd`服务文件（`systemd`单元文件不在本书讨论范围内）：

```
[Unit]
Description=Node App
After=network.target

[Service]
ExecStart=/srv/nodeapp/app.js
Restart=always
User=vagrant
Group=vagrant
Environment=PATH=/usr/bin
Environment=NODE_ENV=production
WorkingDirectory=/srv/nodeapp
[Install]
WantedBy=multi-user.target
```

让我们逐步部署若干个应用程序服务器（在本例中为两个），以提供应用程序服务。再次强调，部署 Node.js 应用程序不在本书的讨论范围内，因此我尽量简化了——仅创建简单的目录和权限，并部署 systemd 单元。在生产环境中，这通常会通过像 Chef 或 Ansible 这样的配置管理工具来完成，并可能结合适当的部署工具：

```
# Tier 2: a scalable number of application servers
vm_app_num = 2
  (1..vm_app_num).each do |n|
    app_lan_ip = "10.20.0.#{n+10}"
    db_lan_ip = "10.30.0.#{n+100}"
    config.vm.define "app-#{n}" do |config|
      config.vm.hostname = "app-#{n}"
      config.vm.network "private_network", ip: app_lan_ip, virtualbox__intnet: "app_lan"
      config.vm.network "private_network", ip: db_lan_ip, virtualbox__intnet: "db_lan"
      config.vm.provision :shell, :inline => "sudo yum install -q -y epel-release"
      config.vm.provision :shell, :inline => "sudo yum install -q -y nodejs npm"
      config.vm.provision :shell, :inline => "sudo mkdir /srv/nodeapp"
      config.vm.provision :shell, :inline => "sudo cp /vagrant/app.js /src/nodeapp"
      config.vm.provision :shell, :inline => "sudo chown -R vagrant.vagrant /srv/"
      config.vm.provision :shell, :inline => "sudo chmod +x /srv/nodeapp/app.js"
      config.vm.provision :shell, :inline => "cd /srv/nodeapp; npm install redis"
      config.vm.provision :shell, :inline => "sudo cp /vagrant/nodeapp.service /etc/systemd/system"
      config.vm.provision :shell, :inline => "sudo systemctl daemon-reload"
      config.vm.provision :shell, :inline => "sudo systemctl start nodeapp"
    end
  end
```

### 第一级：NGINX 反向代理

第一级在这里由 CentOS 7 上的 NGINX 反向代理配置表示，尽可能简单，以适应本次演示。配置一个带有服务器池的 NGINX 反向代理超出了本书的讨论范围：

```
events {
  worker_connections 1024;
}
http {
  upstream app {
    server 10.20.0.11:8080 max_fails=1 fail_timeout=1s;
    server 10.20.0.12:8080 max_fails=1 fail_timeout=1s;
  }
  server {
    listen 80;
    server_name  _;
    location / {
      proxy_set_header   X-Real-IP $remote_addr;
      proxy_set_header   Host      $http_host;
      proxy_pass         http://app;
    }
  }
}
```

现在，让我们创建一个反向代理虚拟机，通过应用程序服务器池提供`http://localhost:8080`。这台虚拟机在自己的局域网（`front_lan`）上监听`10.10.0.11/24`，在应用程序服务器的局域网（`app_lan`）上监听`10.20.0.101/24`：

```
  # Tier 1: an NGINX reverse proxy VM, available on http://localhost:8080
  config.vm.define "front-1" do |config|
    config.vm.hostname = "front-1"
    config.vm.network "private_network", ip: "10.10.0.11", virtualbox__intnet: "front_lan"
    config.vm.network "private_network", ip: "10.20.0.101", virtualbox__intnet: "app_lan"
    config.vm.network "forwarded_port", guest: 80, host: 8080
    config.vm.provision :shell, :inline => "sudo yum install -q -y epel-release"
    config.vm.provision :shell, :inline => "sudo yum install -q -y nginx"
    config.vm.provision :shell, :inline => "sudo cp /vagrant/nginx.conf /etc/nginx/nginx.conf"
    config.vm.provision :shell, :inline => "sudo systemctl enable nginx"
    config.vm.provision :shell, :inline => "sudo systemctl start nginx"
  end
```

启动它（`vagrant up`），并导航到`http://localhost:8080`，在该页面上，应用程序显示应用服务器主机名，你可以确认跨网络的负载均衡是否正常工作（同时应用服务器能够与 Redis 后端通信）。

# 在使用 Laravel 时展示你的工作

你正在使用 Laravel 框架（一个免费开源的 PHP 框架，https://laravel.com/）开发应用程序，并且想向你的同事展示你的工作。使用 Vagrant 开发环境可以帮助你保持工作机的整洁，并允许你使用常用工具和编辑器，同时使用接近生产环境的基础设施。

在这个示例中，我们将部署一台 CentOS 7 服务器，安装 NGINX、PHP-FPM 和 MariaDB，所有 PHP 依赖项，并安装 Composer。你可以从这个示例和本书中的其他示例中构建一个模拟生产环境的环境（三层架构、多台机器和其他特征）。

这个环境将对您网络中的所有同事开放，并且代码对您本地可访问。

## 准备就绪

要按步骤完成此教程，您需要以下内容：

+   一个正常工作的 Vagrant 安装

+   一个正常工作的 VirtualBox 或 VMware 安装

+   一个互联网连接

## 如何操作……

让我们从最简单的 Vagrant 环境开始：

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
  config.vm.define "srv-1" do |config|
    config.vm.hostname = "srv-1"
  end
end
```

### 一个适用于 Laravel 的示例 NGINX 配置

配置 NGINX 以支持 Laravel 超出了本书的范围，但作为参考，这里有一个简单的 NGINX 配置，它能很好地支持我们，监听 HTTP，服务文件位于`/srv/app/public`，并使用 PHP-FPM（文件名为`nginx.conf`）：

```
events {
  worker_connections 1024;
}
http {
  sendfile off;
  server {
    listen 80;
    server_name  _;
    root /srv/app/public ;
    try_files $uri $uri/ /index.php?q=$uri&$args;
    index index.php;
    location / {
      try_files $uri $uri/ /index.php?$query_string;
    }
    location ~ \.php$ {
      try_files $uri /index.php =404;
      fastcgi_split_path_info ^(.+\.php)(/.+)$;
      fastcgi_pass 127.0.0.1:9000;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_param PATH_INFO $fastcgi_script_name;
      include fastcgi_params;
    }
  }
}
```

### 简单的 Shell 配置

我们将创建一个名为`provision.sh`的配置脚本，里面包含我们需要的所有步骤，以便拥有一个完整工作的 Laravel 环境。具体细节超出了本书的范围，但以下是步骤：

1.  我们需要**企业 Linux 的额外软件包**（**EPEL**）：

    ```
    sudo yum install -q -y epel-release

    ```

1.  我们需要 PHP-FPM：

    ```
    sudo yum install -q -y php-fpm

    ```

1.  我们希望 PHP-FPM 以 Vagrant 用户身份运行，以便拥有适当的权限：

    ```
    sudo sed -i 's/user = apache/user = vagrant/' /etc/php-fpm.d/www.conf

    ```

1.  安装一堆 PHP 依赖：

    ```
    sudo yum install -q -y php-pdo php-mcrypt php-mysql php-cli php-mbstring php-dom

    ```

1.  安装 Composer：

    ```
    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer
    sudo chmod +x /usr/local/bin/composer

    ```

1.  安装并配置一个足够好的 NGINX 配置：

    ```
    sudo yum install -q -y nginx
    sudo cp /vagrant/nginx.conf /etc/nginx/nginx.conf

    ```

1.  安装 MariaDB Server：

    ```
    sudo yum install -q -y mariadb-server

    ```

1.  启动所有服务：

    ```
    sudo systemctl enable php-fpm
    sudo systemctl start php-fpm
    sudo systemctl enable nginx
    sudo systemctl start nginx
    sudo systemctl enable mariadb
    sudo systemctl start mariadb

    ```

### 启用配置

要启用使用我们的脚本进行配置，请在虚拟机定义块中添加以下代码：

```
config.vm.provision :shell, :path => "provision.sh"

```

### 共享文件夹

要在主机和 Vagrant 虚拟机之间共享`src`文件夹，并将其挂载到`/srv/app`，您可以添加以下代码：

```
config.vm.synced_folder "src/", "/srv/app"

```

### 公共 LAN 网络

我们现在需要做的最后一件事是向我们的 Vagrant 虚拟机添加一个网络接口，它将连接到真实的局域网，这样我们的同事就可以通过网络轻松访问它：

```
config.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)"

```

根据需要调整您的网络适配器名称（这在 Mac 上，如您所见）。另一种解决方案是不指定适配器名称，这样会显示一个可供桥接的适配器列表：

```
==> srv-1: Available bridged network interfaces:
1) en0: Wi-Fi (AirPort)
[...]

```

启动 Vagrant 环境（`vagrant up`），当它可用时，您可以执行诸如获取网络信息等命令：`vagrant ssh -c "ip addr"`。您的情况可能不同，但在这个网络中，这个 Vagrant 盒子的公共 IP 地址是`192.168.1.106`，所以我们的工作是可访问的。

现在，您可以开始在`./src/`文件夹中编码。虽然这不是一本 Laravel 书籍，但创建新项目的方式如下：

```
cd /srv/app
composer create-project --prefer-dist laravel/laravel.

```

别忘了提前清空文件夹中的所有文件。导航到`http://local-ip/`，您将看到默认的 Laravel 欢迎页面。

要验证文件共享同步是否正常工作，请编辑`./resources/views/welcome.blade.php`文件，并重新加载浏览器查看更改是否已反映。

## 还有更多……

如果将 Vagrantfile 直接与项目代码一起包含，您的同事或贡献者只需运行`vagrant up`，即可看到它在运行。

其他 Vagrantfile 共享选项包括 Windows 共享（smb）、rsync（对于远程虚拟机，如 AWS EC2 非常有用），甚至 NFS。

使用 VirtualBox 共享功能时，一个明显的 bug 会导致文件损坏或无法更新。解决方法是禁用 Web 服务器配置中的 `sendfile`，使用 NGINX：

```
sendfile off;

```

使用 Apache，如下所示：

```
EnableSendfile Off

```

# 将你的 Vagrant 环境共享给全世界

你正在本地 Vagrant 环境中进行项目开发，并且你想向位于另一个城市的客户展示工作的状态。也许你在配置某些内容时遇到问题，需要来自地球另一端同事的远程帮助。或者，你可能希望从家里、酒店或共享办公空间访问你的工作 Vagrant 盒子？这里有一个很棒的 Vagrant 共享功能，我们将用于在 CentOS 7.2 上运行的 Ghost 博客。

## 准备就绪

要完成这个教程，你需要以下内容：

+   一个正常工作的 Vagrant 安装

+   一个正常工作的 VirtualBox 安装

+   一个免费的 HashiCorp Atlas 账户 ([`atlas.hashicorp.com/account/new`](https://atlas.hashicorp.com/account/new))

+   一个互联网连接

## 如何操作……

我们从这个简单的 Vagrantfile 开始：

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
  config.vm.define "blog" do |config|
    config.vm.hostname = "blog"
  end
end
```

我们知道需要安装一些包，所以让我们添加一个配置脚本来执行：

```
    config.vm.provision :shell, :path => "provision.sh"
```

我们想要在本地对 Ghost 博客进行修改，比如添加主题等，因此我们将 `src/` 文件夹同步到远程 `/srv/blog` 文件夹：

```
    config.vm.synced_folder "src/", "/srv/blog"
```

我们需要一个本地私有网络，以便可以访问虚拟机，并将 `2368` TCP 端口（Ghost 默认端口）重定向到我们主机的 `8080` HTTP 端口：

```
    config.vm.network "private_network", type: "dhcp"
    config.vm.network "forwarded_port", guest: 2368, host: 8080
```

### 配置

1.  为了配置我们的新盒子，首先需要启用 EPEL：

    ```
    sudo yum install -q -y epel-release

    ```

1.  然后安装要求的工具，`node`、`npm` 和 `unzip`：

    ```
    sudo yum install -q -y node npm unzip 

    ```

1.  下载最新版本的 Ghost：

    ```
    curl -L https://ghost.org/zip/ghost-latest.zip -o ghost.zip

    ```

1.  将其解压到 `/srv/blog` 文件夹中：

    ```
    sudo unzip -uo ghost.zip -d /srv/blog/

    ```

1.  安装 Ghost 依赖项：

    ```
    cd /srv/blog && sudo npm install --production

    ```

将所有这些命令放入 `provisioning.sh` 脚本中，之后我们就可以开始了：`vagrant up`。

### 启动 Ghost 引擎

按照通常的方式，登录到你的 Vagrant 盒子，启动节点服务器：

```
vagrant ssh
cd /srv/blog && sudo npm start --production
[…]
Ghost is running in production...
Your blog is now available on http://my-ghost-blog.com
Ctrl+C to shut down

```

将生成的 `config.js` 文件中的主机 IP 从 `127.0.0.1` 更改为 `0.0.0.0`，这样服务器就能监听所有接口：

```
server: {
 host: '0.0.0.0',
 port: '2368'
 }

```

重启节点服务器：

```
cd /srv/blog && sudo npm start --production

```

你现在可以通过你的盒子局域网 IP 直接访问博客（根据你的情况调整 IP）：`http://172.28.128.3:2368/`。

### 共享访问

现在，你可以通过你的 Vagrant 盒子本地访问你的应用，让我们通过互联网使用 `vagrant share` 给它提供访问权限：

#### HTTP

默认情况下，通过 HTTP 共享，因此你可以通过 Web 浏览器访问你的工作：

```
$ vagrant share
==> srv-1: Detecting network information for machine...
[...]
==> srv-1: Your Vagrant Share is running! Name: anxious-cougar-6317
==> srv-1: URL: http://anxious-cougar-6317.vagrantshare.com

```

这个 URL 是你可以分享给任何人以公开访问你工作的链接：Vagrant 服务器作为代理。

### SSH

另一种可能的共享选项是通过 SSH（默认情况下禁用）。该程序会要求你输入密码，用以远程连接到盒子：

```
$ vagrant share --ssh
==> srv-1: Detecting network information for machine...
[...]
srv-1: Please enter a password to encrypt the key:
 srv-1: Repeat the password to confirm:
[...]
==> srv-1: You're sharing with SSH access. This means that another user
==> srv-1: simply has to run `vagrant connect --ssh subtle-platypus-4976`
==> srv-1: to SSH to your Vagrant machine.
[...]

```

现在，在家里或共享办公空间，你只需连接到你的工作 Vagrant 盒子（如果需要，默认的 Vagrant 密码是 vagrant）：

```
$ vagrant connect --ssh subtle-platypus-4976
Loading share 'subtle-platypus-4976'...
[...]
[vagrant@srv-1 ~]$ head -n1 /srv/blog/config.js
// # Ghost Configuration

```

你或你的同事现在可以通过互联网远程登录到你自己的 Vagrant 盒子了！

# 使用 Vagrant 模拟 Chef 升级

如果能够快速模拟生产环境的变更，那不是很棒吗？很可能你在生产环境中使用的是 Chef。我们将学习如何在 Vagrant 中使用 Chef cookbook，并且如何在不同环境之间模拟 Chef 版本升级。这种配置是将基础设施即代码（Infrastructure as Code）结合得非常好的起点。

## 准备工作

要执行这个步骤，你需要以下资源：

+   一个正常工作的 Vagrant 安装

+   一个正常工作的 VirtualBox 安装

+   一个互联网连接

## 如何做到这一点…

让我们从一个最小的虚拟机 `prod` 开始，它仅启动一个 CentOS 7.2，就像我们在生产环境中的设置：

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
  config.vm.define "prod" do |config|
    config.vm.hostname = "prod"
    config.vm.network "private_network", type: "dhcp"
  end

end
```

### Vagrant Omnibus Chef 插件

现在，如果我们想使用 Chef 代码（Chef 代码是以目录组织的 Ruby 文件，这些文件形成一个名为“cookbook”的单元，用于配置和维护系统的特定区域），我们首先需要在 Vagrant 虚拟机上安装 Chef。实现这一点有很多方法，从配置 Shell 脚本到使用已经安装 Chef 的盒子。一个干净、可靠且可重复的方法是使用 Vagrant 插件来实现——vagrant-omnibus。Omnibus 是一个打包好的 Chef。像其他 Vagrant 插件一样安装它：

```
$ vagrant plugin install vagrant-omnibus
Installing the 'vagrant-omnibus' plugin. This can take a few minutes...
Installed the plugin 'vagrant-omnibus (1.4.1)'!

```

然后，只需在 Vagrantfile 的虚拟机定义中添加以下配置，你就能确保始终安装最新版本的 Chef：

```
config.omnibus.chef_version = :latest
```

然而，我们的目标是模拟生产环境，也许我们仍然在使用 Chef v11.x 系列的最新版本，而不是最新的 12.x 版本，因此我们将具体指定我们需要的版本：

```
config.omnibus.chef_version = "11.18.12"
```

现在，我们使用了一个新插件，Vagrantfile 对每个人可能无法直接工作。用户必须安装 vagrant-omnibus 插件。如果你关心一致性和可重复性，可以选择在 Vagrantfile 的开头添加以下 Ruby 检查：

```
%w(vagrant-vbguest vagrant-omnibus).each do |plugin|
  unless Vagrant.has_plugin?(plugin)
    raise "#{plugin} plugin is not installed! Please install it using `vagrant plugin install #{plugin}`"
  end
end
```

这段代码会遍历每个插件名称，验证 Vagrant 是否将它们标记为 *已安装*。如果没有，就停止并返回一个帮助退出信息，指导如何安装所需的插件。

### 一个示例的 Chef 配方

本书的这一部分并不是关于编写 Chef 配方（后面章节会详细讲解！），因此我们会简化这一部分。我们的目标是在 CentOS 7 上安装 Apache 2 Web 服务器（`httpd` 包），并启动它。下面是我们的示例配方（`cookbooks/apache2/recipes/default.rb`）；它做的正是用通俗的语言描述的内容：

```
package "httpd"

service "httpd" do
  action [ :enable, :start ]
end
```

### Vagrant 与 Chef 集成

在我们的虚拟机定义块中，我们将告诉 Vagrant 使用 Chef Solo（一个无需 Chef 服务器即可单独运行 Chef 的方式）来配置我们的虚拟机：

```
    config.vm.provision :chef_solo do |chef|
      chef.add_recipe 'apache2'
    end
```

就这么简单。启动 Vagrant (`vagrant up`)，你将得到一个完全配置好的虚拟机，使用的是旧版本 11.18.12，并且 Apache 2 Web 服务器正在运行。

我们的手动测试可以包括检查 chef-solo 版本是否是我们要求的版本：

```
$ chef-solo --version
Chef: 11.18.12

```

它们还可以检查我们是否已经安装了 `httpd`：

```
$ httpd -v
Server version: Apache/2.4.6 (CentOS)

```

另外，我们可以检查 `httpd` 是否正在运行：

```
$ pidof httpd
13029 13028 13027 13026 13025 13024

```

### 注意

除 chef-solo 外，还存在其他各种选项，比如 chef-client 和 chef-zero。

### 测试 Chef 版本更新

因此，我们在本地模拟了我们的生产环境，使用相同的 CentOS 版本、生产环境中使用的 apache2 cookbook 和旧的 Chef 版本 11。我们的下一个任务是测试在升级到新版本 12 后，一切是否仍然运行顺利。让我们创建第二个“暂存”虚拟机，设置与生产环境非常相似，唯一不同的是我们想安装当前最新的 Chef 版本（截至本文写作时是 12.13.37，随时可以使用 `:latest`）：

```
  config.vm.define "staging" do |config|
    config.vm.hostname = "staging"
    config.omnibus.chef_version = "12.13.37"
    config.vm.network "private_network", type: "dhcp"
    config.vm.provision :chef_solo do |chef|
      chef.add_recipe 'apache2'
    end
  end
```

启动这个新机器（`vagrant up staging`），看看我们的设置在新版本的 Chef 中是否仍然有效：

```
$ vagrant ssh staging
$ chef-solo --version
Chef: 12.13.37
$ httpd -v
Server version: Apache/2.4.6 (CentOS)
$ pidof httpd
13029 13028 13027 13026 13025 13024

```

因此，基于我们的测试结果，我们可以安全地假设，最新版本的 Chef 仍然能够与我们的生产 Chef 代码正常工作。

## 还有更多内容…

这里有更多控制 Vagrant 环境的方法，并可以在其中使用更好的 Chef 工具。

### 控制默认的 Vagrant 虚拟机

你可能不总是希望启动生产和暂存的 Vagrant 虚拟机，尤其是在你只想处理默认的生产环境设置时。要指定默认的虚拟机：

```
config.vm.define "prod", primary: true do |config|
  […]
end
```

要在执行 `vagrant up` 命令时不自动启动虚拟机：

```
config.vm.define "staging", autostart: false do |config|
  […]
end
```

### Berkshelf 和 Vagrant

如果你的生产环境使用 Chef，那么你很可能也在使用 Berkshelf 来进行依赖管理，而不是完全使用本地 cookbooks（如果你没有这样做，你应该尝试！）。

Vagrant 在启用了 Berkshelf 的 Chef 环境中工作得很好，使用 `vagrant-berkshelf` 插件。

### 注意

你的工作站需要 Chef 开发工具包（Chef DK：[`downloads.chef.io/chef-dk/`](https://downloads.chef.io/chef-dk/)）才能正确运行。

### 使用 Test Kitchen 进行测试

这个设置实际上非常接近用于基础架构代码测试的方式，你会在本书的专门章节中看到许多相似之处。

# 使用 Ansible 与 Vagrant 创建 Docker 主机

Ansible ([`www.ansible.com/`](https://www.ansible.com/)) 是一个非常简单且强大的开源自动化工具。虽然使用和创建 Ansible *playbooks* 超出了本书的主题，但我们将使用一个非常简单的 *playbook* 来安装和配置 CentOS 7 上的 Docker。从这里开始，你将能够逐步编写更复杂的 Ansible playbooks。

## 准备工作

要执行此配方，你需要以下内容：

+   一个正常工作的 Vagrant 安装

+   一个正常工作的虚拟化管理程序

+   你机器上的 Ansible 安装（一个简单的方法是 `$ pip install ansible`，或者使用你习惯的包管理器，如 APT 或 YUM/DNF）

+   一个互联网连接

## 如何操作…

因为编写复杂的 Ansible playbook 超出了本书的范围，我们将使用一个非常简单的 playbook，这样你可以稍后深入了解 Ansible，并仍然能够重用这个示例。

### 一个简单的 Ansible Docker playbook 用于 Vagrant

我们的 playbook 文件（`playbook.yml`）是一个普通的 YAML 文件，我们将按照以下顺序进行操作：

1.  安装 EPEL。

1.  创建一个 Docker Unix 组。

1.  将默认的 Vagrant 用户添加到新的 Docker 组中。

1.  从 CentOS 仓库安装 Docker。

1.  启用并启动 Docker 引擎。

这是 `playbook.yml` 文件的样子：

```
---
- hosts: all
 become: yes
 tasks:
 - name: Enable EPEL
 yum: name=epel-release state=present
 - name: Create a Docker group
 group: name=docker state=present
 - name: Add the vagrant user to Docker group
 user: name=vagrant groups=docker append=yes
 - name: Install Docker
 yum: name=docker state=present
 - name: Enable and Start Docker Daemon
 service: name=docker state=started enabled=yes

```

### 从 Vagrant 应用 Ansible

为了使用我们的 Ansible playbook，我们从一个简单的 Vagrantfile 开始，它启动一个 CentOS 7 虚拟机：

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
  config.vm.define "srv-1" do |config|
    config.vm.hostname = "srv-1"
    config.vm.network "private_network", type: "dhcp"
  end
end
```

只需将 Ansible 配置添加到虚拟机定义中，这样它将加载并应用你的 `playbook.yml` 文件：

```
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbook.yml"
    end
```

你现在可以运行 `vagrant up` 并立即使用 CentOS 7 Docker 引擎版本：

```
$ vagrant ssh
[vagrant@srv-1 ~]$ systemctl status docker
[vagrant@srv-1 ~]$ docker --version
Docker version 1.10.3, build d381c64-unsupported
[vagrant@srv-1 ~]$ docker run -it --rm alpine /bin/hostname
0f44a4d7afcd

```

## 还有更多内容…

如果由于某些原因你不能在主机上安装 Ansible，或者你需要在 Vagrant box 中安装特定版本的 Ansible 来模拟生产环境，而又不想影响本地的 Ansible 安装，你可以使用一个有趣的 Ansible 提供者变体：它将直接使用来自客机 VM 的 Ansible，如果没有安装，它会从官方仓库或 PIP 安装。你可以使用这个非常简单的默认配置：

```
    config.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "playbook.yml"
    end
```

你也可以使用以下命令：

```
$ vagrant up
[…]
==> srv-1: Running provisioner: ansible_local...
 srv-1: Installing Ansible...
 srv-1: Running ansible-playbook...
[…]

```

通过 SSH 登录到 box 并检查是否已本地安装最新版本的 Ansible：

```
$ vagrant ssh
$ ansible --version
ansible 2.1.1.0

```

如果你的使用场景不同，你可以使用更精确的部署选项，通过 PIP 固定 Ansible 版本号（例如，使用 1.9.6 版本，而不是最新的 2.x 系列）：

### 注意

启动过程会明显变慢，因为它需要在客机系统上安装许多软件包。

```
    config.vm.provision "ansible_local" do |ansible|
 ansible.version = "1.9.6"
 ansible.install_mode = :pip
      ansible.playbook = "playbook.yml"
    end
```

你也可以使用以下命令：

```
$ vagrant up
[…]
==> srv-1: Running provisioner: ansible_local...
 srv-1: Installing Ansible...
 srv-1: Installing pip... (for Ansible installation)
 srv-1: Running ansible-playbook...

```

在 Vagrant 客机中，你现在可以检查 PIP 和 Ansible 的版本：

```
$ pip --version
pip 8.1.2 from /usr/lib/python2.7/site-packages (python 2.7)
$ ansible --version
ansible 1.9.6

```

你也可以检查我们的 playbook 是否在旧版本的 1.x Ansible 上正确安装：

```
$ docker version

```

还要检查 Docker 是否已安装，并且现在可以作为 Vagrant 用户正常工作：

```
$ docker run -it --rm alpine ping -c2 google.com
PING google.com (216.58.211.78): 56 data bytes
64 bytes from 216.58.211.78: seq=0 ttl=61 time=22.078 ms
64 bytes from 216.58.211.78: seq=1 ttl=61 time=21.061 ms

```

# 在 CoreOS 上使用 Docker 容器与 Vagrant

Vagrant 可以帮助模拟环境，Docker 容器在 Vagrant 中也得到了支持。我们将使用一个最佳平台来运行容器——免费且开源的轻量级操作系统 CoreOS。它基于 Linux，旨在简化容器和集群的部署，并提供官方的 Vagrant box。我们将使用 Vagrant Docker 提供者（而不是 Vagrant Docker 提供者）部署官方的 WordPress 容器与 MariaDB 容器。

## 准备工作

为了执行这个步骤，你需要以下内容：

+   一个正常工作的 Vagrant 安装

+   一个正常工作的虚拟化管理程序

+   一个互联网连接

## 如何操作…

CoreOS 并没有在 Atlas 的默认位置托管官方镜像，而是自行托管。所以，我们必须在 Vagrantfile 中指定 Vagrant box 的完整 URL：

```
Vagrant.configure("2") do |config|
  config.vm.box = https://stable.release.core-os.net/amd64-usr/current/coreos_production_vagrant.box
end
```

由于 CoreOS 是一个极简操作系统，它不支持任何 VirtualBox 客户机附加工具，因此我们将禁用它们。如果我们（很可能）安装了 `vagrant-vbguest` 插件，也不要尝试使用这些工具：

```
  config.vm.provider :virtualbox do |vb|
      vb.check_guest_additions = false
      vb.functional_vboxsf     = false
  end

  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end
```

让我们创建一个新的虚拟机定义，使用 CoreOS Vagrant box：

```
  config.vm.define "core-1" do |config|
    config.vm.hostname = "core-1"
    config.vm.network "private_network", type: "dhcp" 
  end
```

我们现在需要运行 Docker Hub 上的 `mariadb` 和 `wordpress` 官方容器。直接使用 Docker，我们将运行以下命令：

```
$ docker run -d --name mariadb -e MYSQL_ROOT_PASSWORD=h4ckm3 mariadb
$ docker run -d -e WORDPRESS_DB_HOST=mariadb -e 'WORDPRESS_DB_PASSWORD=h4ckm3 --link mariadb:mariadb -p 80:80 wordpress

```

让我们将其转换到我们的 Vagrantfile 中：

```
db_root_password = "h4ckm3"
config.vm.provision "docker" do |docker|
 docker.run "mariadb",
 args: "--name 'mariadb' -e 'MYSQL_ROOT_PASSWORD=#{db_root_password}'"
 docker.run "wordpress",
 args: "-e 'WORDPRESS_DB_HOST=mariadb' -e 'WORDPRESS_DB_PASSWORD=#{db_root_password}' --link 'mariadb:mariadb' -p '80:80'"
 end

```

启动 Vagrant (`$ vagrant up`)，你将访问一个已经安装好并可以使用的 WordPress 环境，运行在 CoreOS 上：

```
$ curl -IL http://172.28.128.3/wp-admin/install.php
HTTP/1.1 200 OK
Date: Thu, 25 Aug 2016 10:54:17 GMT
Server: Apache/2.4.10 (Debian)
X-Powered-By: PHP/5.6.25
Expires: Wed, 11 Jan 1984 05:00:00 GMT
Cache-Control: no-cache, must-revalidate, max-age=0
Content-Type: text/html; charset=utf-8

```

## 还有更多……

CoreOS 团队提供了一个完整的 Vagrant 环境，用于尝试和操作一个 CoreOS 集群 [`github.com/coreos/coreos-vagrant`](https://github.com/coreos/coreos-vagrant)。你将能够尝试所有 CoreOS 的功能和配置选项，涵盖所有发布渠道（alpha、beta 或 stable）。

其他操作系统如 Ubuntu 或 CentOS 完全支持为 Docker 容器提供配置，即使基础镜像中一开始没有安装 Docker。Vagrant 会为你安装 Docker，因此它将透明地工作，并在安装完成后立即运行容器。

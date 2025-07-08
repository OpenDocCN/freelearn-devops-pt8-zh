# 第十章：控制容器

|   | *计算机的内部运作是如此愚笨，但它的速度却快得惊人！* |   |
| --- | --- | --- |
|   | --*理查德·费曼* |

在本章中，我们将探讨新兴的**容器**话题，并看看它与配置管理的关系。我们将看到如何使用 Puppet 管理 Docker 守护进程，以及镜像和容器，另外还会探索在容器中管理配置的不同策略。

![控制容器](img/8880_10_01.jpg)

# 了解容器

尽管容器背后的技术至少已有三十年的历史，但容器真正大规模应用还是近几年的事（为了更形象的比喻，可以说它们终于起飞了）。这主要得益于 Docker 的崛起，Docker 是一个软件平台，使得创建和管理容器变得更加简便。

## 部署问题

Docker 解决的问题主要是**软件部署**：也就是说，Docker 使得在各种环境中安装和运行软件变得轻而易举。举个例子，假设有一个典型的 PHP web 应用。为了运行这个应用，至少需要以下内容在节点上：

+   PHP 源代码

+   PHP 解释器

+   相关的依赖项和库

+   应用程序所需的 PHP 模块

+   用于构建 PHP 模块原生二进制文件的编译器和构建工具

+   Web 服务器（例如 Apache）

+   用于提供 PHP 应用的模块（例如 `mod_php`）

+   应用程序的配置文件

+   用于运行应用程序的用户

+   用于存放日志文件、图片和上传数据等内容的目录

那么，如何管理这些东西呢？你可以使用系统软件包格式，比如 RPM 或 DEB，这些格式使用元数据来描述它们之间的依赖关系，并且包含能够完成大部分系统配置的脚本。

然而，这种打包方式是特定于某个操作系统版本的。例如，为 Ubuntu 18.04 打包的程序包无法在 Ubuntu 16.04 或 Red Hat Enterprise Linux 上安装。为多个流行操作系统维护多个程序包，将给维护应用程序本身增加很大的工作量。

## 部署选项

解决这个问题的一种方式是作者为软件提供**配置管理****清单**，比如 Puppet 模块或 Chef 配方来安装软件。然而，如果软件的预期用户没有使用 CM 工具，或者使用了不同的工具，那么这种方式就无济于事。即使他们使用的是完全相同版本的相同工具，并且操作系统也是一样的，他们在集成第三方模块时可能仍然会遇到问题，而该模块本身又依赖于其他模块，等等。这显然不是一个即插即用的解决方案。

另一个选项是**综合包**，它包含了软件运行所需的一切。举例来说，针对我们 PHP 应用的综合包可能包含 PHP 二进制文件及其所有依赖，还有应用程序需要的其他任何内容。然而，这些包通常非常庞大，而且综合包依然是针对特定操作系统和版本的，并且需要进行大量的维护工作。

大多数包管理器并没有提供高效的二进制更新功能，因此即使是最小的更新也需要重新下载整个包。一些大型包甚至还包括它们自己的配置管理工具！

另一种解决方案是提供整个**虚拟机镜像**，例如 Vagrant box（我们在本书中使用的 Vagrant box 就是一个很好的例子）。这个镜像不仅包含应用程序及其依赖和配置，还包括整个操作系统。这是一个相对便捷的解决方案，因为任何能够运行虚拟机宿主软件的平台（例如 VirtualBox 或 VMware）都能够运行这个虚拟机。

然而，虚拟机存在性能损失，并且它们也消耗大量资源，如内存和磁盘空间，虚拟机镜像本身体积庞大，且在网络中移动时十分笨重。

虽然理论上你可以通过构建一个虚拟机镜像并将其推送到生产环境的虚拟机主机来部署你的应用程序，一些人也确实这么做，但这远不是一种高效的分发方法。

## 引入容器

近年来，许多操作系统增加了自包含执行环境的功能，更简洁地称之为**容器**，在这些环境中，程序可以直接在 CPU 上运行，但对机器其他部分的访问受到严格限制。容器就像一个安全沙箱，里面运行的任何程序只能访问容器内的文件和程序，而无法访问外部资源。

这在原理上类似于虚拟机，只是底层技术截然不同。容器中的程序不是通过虚拟处理器和软件仿真层运行，而是直接在底层物理硬件上运行。这使得容器比虚拟机更高效。换句话说，运行容器所需的硬件性能远低于运行相同性能虚拟机所需的硬件性能。

单个虚拟机会消耗大量宿主机的资源，这意味着在同一主机上运行多个虚拟机可能非常消耗资源。相比之下，在容器中运行一个进程所消耗的资源与直接在宿主机上运行相同的进程所消耗的资源没有区别。

因此，您可以在单个主机上运行非常多的容器，并且每个容器都是完全自包含的，无法访问主机或任何其他容器（除非您明确允许）。容器在内核层面上其实只是一个命名空间。运行在该命名空间中的进程无法访问外部的任何内容，反之亦然。机器上的所有容器都使用主机操作系统的内核，因此，尽管容器可以在不同的 Linux 发行版之间移植，但例如 Linux 容器不能直接在 Windows 主机上运行。不过，Linux 容器可以通过 Docker for Windows 提供的虚拟化层在 Windows 上运行。

## Docker 对容器的作用

那么，如果容器本身是由内核提供的，Docker 是做什么的呢？事实证明，拥有一个引擎并不等于拥有一辆车。操作系统内核可能提供了容器化的基本设施，但您还需要：

+   构建容器的规范

+   容器镜像的标准文件格式

+   用于存储、版本控制、组织和检索容器镜像的协议

+   启动、运行和管理容器的软件

+   允许容器之间网络流量传输的驱动程序

+   容器之间的通信方式

+   将数据传入容器的功能

这些需要通过额外的软件提供。事实上，有许多软件前端可以帮助您管理容器：Docker、OCID、CoreOS/rkt、Apache Mesos、LXD、VMware Photon、Windows Server 容器等。然而，Docker 无疑是市场的领导者，目前生产环境中大多数容器都运行在 Docker 下（最近的一项调查显示，比例超过 90%）。

# 使用 Docker 部署

使用容器部署软件的原理非常简单：软件及其运行所需的一切都包含在容器 **镜像** 中，镜像类似于一个包文件，但可以直接由容器运行时执行。

要运行软件，您只需执行类似下面的命令（如果您已经安装了 Docker，可以尝试一下！）：

```
docker run bitfield/hello
Hello, world
```

Docker 将从您配置的 **仓库** 中下载指定的镜像（这可以是公共仓库，称为 Docker Hub，或者是您自己的私有 Docker 仓库），并执行它。您可以使用数以千计的 Docker 镜像，许多软件公司也越来越多地使用 Docker 镜像作为部署产品的主要方式。

## 构建 Docker 容器

那么，这些 Docker 镜像来自哪里呢？Docker 镜像就像一个存档或包文件，包含容器内部所有文件的文件和目录结构，包括可执行二进制文件、共享库和配置文件。要创建这个镜像文件，您可以使用 `docker build` 命令。

`docker build` 使用一个叫做 **Dockerfile** 的特殊文本文件，该文件指定容器中应该包含什么。通常，一个新的 Docker 镜像是基于现有镜像，并做了一些修改。例如，有一个 Ubuntu Linux 的 Docker 镜像，它包含了一个已经完全安装的操作系统，可以直接运行。

你的 Dockerfile 可能会指定使用 Ubuntu Docker 镜像作为起点，然后安装 `nginx` 包。最终的 Docker 容器包含了 Ubuntu 镜像中的所有内容，以及 `nginx` 包。你现在可以将这个镜像上传到注册中心，并使用 `docker run` 在任何地方运行它。

如果你想用 Docker 打包自己的软件，你可以选择一个合适的基础镜像（如 Ubuntu），然后编写一个 Dockerfile，将你的软件安装到该基础镜像上。当你使用 `docker build` 构建容器镜像时，结果将是一个包含你软件的容器，任何人都可以使用 `docker run` 运行。唯一需要安装的就是 Docker。

这使得 Docker 成为一个非常适合软件供应商将产品打包成易于安装的格式的方式，同时也方便用户快速尝试不同的软件，以查看它是否满足他们的需求。

## 分层文件系统

Docker 文件系统有一个叫做 **分层** 的特性。容器是通过分层构建的，所以如果某些内容发生变化，只有受影响的层和它之上的层需要重新构建。这使得一旦容器镜像被构建并部署后，更新镜像变得更加高效。

例如，如果你在应用程序中更改了一行代码并重建容器，那么只有包含你应用程序的层需要重新构建，以及任何其上方的层。基础镜像和受影响层下方的其他层保持不变，并且可以被新容器重复使用。

## 使用 Puppet 管理容器

使用 Docker 打包和运行软件时，你需要能够做到以下几件事：

+   安装、配置和管理 Docker 服务本身

+   构建你的镜像

+   当 Dockerfile 更改或依赖项更新时，重新构建镜像

+   管理运行中的镜像、它们的数据存储和配置

除非你希望将镜像公开，否则你还需要为自己的镜像托管一个镜像注册中心。

这些听起来像是配置管理工具可以解决的问题，幸运的是，我们有一个很棒的配置管理工具可用。有趣的是，虽然大多数人都认识到传统的服务器需要由像 Puppet 这样的工具自动构建和管理，但容器似乎还没有（至少目前）面临同样的需求。

问题在于，制作并运行一个简单的容器太容易了，以至于许多人认为容器的配置管理是多余的。虽然在你首次尝试 Docker 并实验简单容器时可能如此，但当你在生产环境中运行复杂的多容器服务时，规模一大，问题就变得更加复杂。

首先，将非简单应用程序容器化并非简单。这些应用程序需要依赖项、配置设置和数据，还需要与其他应用程序和服务通信的方式。虽然 Docker 为你提供了相关工具，但它并不会为你做所有的工作。

其次，你需要一个基础设施来构建容器、更新容器、存储和检索结果镜像，并在生产环境中部署和管理这些容器。容器的配置管理与传统服务器应用的配置管理非常相似，只是它发生在稍高的层次上。

容器非常棒，但它们并不能完全取代配置管理工具的需求（还记得第一章中提到的*痛苦守恒定律*吗？）：

*“如果你在某个地方避免了痛苦，它会在另一个地方再次出现。无论什么新技术出现，都无法解决我们所有的问题；顶多，它会用不同的、令人耳目一新的问题来取代它们。”*

# 使用 Puppet 管理 Docker

Puppet 当然可以像管理其他软件一样为你安装和管理 Docker 服务，但它还能做更多的事情。它可以下载并运行 Docker 镜像、从 Dockerfile 构建镜像、挂载容器中的文件和目录，并管理 Docker 卷和网络。在本章中，我们将学习如何完成这些操作。

## 安装 Docker

在进行其他操作之前，我们需要在节点上安装 Docker（当然是使用 Puppet）。`puppetlabs/docker_platform`模块非常适合这个目的。

1.  如果你已经安装并运行了`r10k`模块管理工具，如第七章中*使用 r10k*部分所示，所需的模块将已经安装。如果没有，请运行以下命令来安装它：

    ```
    cd /etc/puppetlabs/code/environments/pbg
    sudo r10k puppetfile install

    ```

1.  一旦模块安装完成，你只需应用如下清单（`docker_install.pp`）即可在节点上安装 Docker：

    ```
    include docker
    ```

1.  运行以下命令以应用清单：

    ```
    sudo puppet apply --environment pbg /examples/docker_install.pp

    ```

1.  要检查 Docker 是否已安装，请运行以下命令（你可能会看到不同的版本号，但没关系）：

    ```
    docker --version
    Docker version 17.05.0-ce, build 89658be
    ```

## 运行 Docker 容器

要运行一个 Docker 容器，我们首先需要从 Docker 注册表中下载它，Docker 注册表是一个存储容器镜像的服务器。默认的注册表是 Docker Hub，官方的公共 Docker 注册表。

要通过 Puppet 实现这一点，可以使用`docker::image`资源（`docker_image.pp`）：

```
docker::image { 'bitfield/hello':
  ensure => 'latest',
}
```

与`package`资源一样，如果你指定`ensure => latest`，Puppet 每次运行时会检查注册表，并确保你拥有最新版本的镜像。

要运行刚刚下载的镜像，向清单中添加一个`docker::run`资源（`docker_run.pp`）：

```
docker::run { 'hello':
  image   => 'bitfield/hello',
  command => '/bin/sh -c "while true; do echo Hello, world; sleep 1; done"',
}
```

使用以下命令应用此清单：

```
sudo puppet apply /examples/docker_run.pp

```

`docker::run` 资源告诉 Docker 从本地镜像缓存中获取 `bitfield/hello` 镜像并运行指定的命令，在本例中，该命令会无限循环并打印 `Hello, world`。（我告诉过你，容器非常有用。）

容器现在在你的节点上运行，你可以通过以下命令来检查：

```
sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED
STATUS              PORTS               NAMES
ba1f4aced778        bitfield/hello      „/bin/sh -c ‚while tr"   4 minutes ago       Up 4 minutes                            hello
```

`docker ps` 命令显示所有当前正在运行的容器（`docker ps -a` 也会显示停止的容器），并提供以下信息：

+   容器 ID，Docker 对容器的内部标识符

+   镜像名称（在我们的示例中是 `bitfield/hello`）

+   当前正在容器中执行的命令

+   创建时间

+   当前状态

+   容器映射的所有端口

+   容器的可读名称（即我们在清单中为 `docker::run` 资源指定的标题）

容器正在作为服务运行，我们可以通过以下命令来检查：

```
systemctl status docker-hello
* docker-hello.service - Daemon for hello
   Loaded: loaded (/etc/systemd/system/docker-hello.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2017-05-16 04:07:23 PDT; 1min 4s ago
 Main PID: 24385 (docker)
   CGroup: /system.slice/docker-hello.service
           `-24385 /usr/bin/docker run --net bridge -m 0b --name hello bitfield/hello...
...
```

## 停止容器

根据 Docker 文档，你可以通过运行 `sudo docker stop NAME` 来停止容器。但是，如果你尝试这样做，然后再运行 `sudo docker ps`，你会看到容器仍在运行。这是怎么回事？

Puppet 模块默认假设你希望将所有容器作为服务运行；即配置 `systemd` 以保持容器运行，并在启动时启动它。

因此，如果你想停止作为服务运行的容器，你需要通过 Puppet 来实现，将 `docker::run` 资源的 `ensure` 参数设置为 `absent`，如下示例所示（`docker_absent.pp`）：

```
docker::run { 'hello':
  ensure => absent,
  image  => 'bitfield/hello',
}
```

或者，你可以在命令行中使用 `systemctl` 命令来停止服务：

```
sudo systemctl stop docker-hello

```

### 提示

如果你不希望容器由 `systemd` 作为服务管理，可以在 `docker::run` 资源中指定参数 `restart => always`。这会告诉 Docker 在容器退出时自动重启容器；因此，Puppet 不需要创建 `systemd` 服务来管理它。

## 运行多个容器实例

当然，自动化的真正强大之处在于可扩展性。我们不局限于运行单个容器实例；Puppet 会非常乐意启动任意多个容器实例。

每个 `docker::run` 资源必须具有唯一名称，和其他 Puppet 资源一样，所以你可以在 `each` 循环中创建它们，如下示例所示（`docker_run_many.pp`）：

```
range(1,20).each | $instance | {
  docker::run { "hello-${instance}":
    image   => 'bitfield/hello',
    command => '/bin/sh -c "while true; do echo Hello, world; sleep 1; done"',
  }
}
```

`range()` 函数来自 `stdlib` 模块，正如你可能预期的那样，`range(1,20)` 返回的是从 1 到 20（包括 20）之间的整数序列。我们使用 `each` 函数遍历这个序列，每次循环时，`$instance` 被设置为下一个整数。

`docker::run`资源标题在每次迭代时包含`$instance`的值，因此每个容器都会有唯一的名称：`hello-1`、`hello-2`、… `hello-20`。我随机选择了数字 20，仅作为示例；你可以根据可用资源（例如系统的 CPU 数量或可用内存）计算要运行的实例数量。

别忘了在之后停止这些容器（编辑示例清单，向`docker::run`资源添加`ensure => absent`并重新应用）。

# 管理 Docker 镜像

当然，从 Docker Hub 或其他注册中心下载和运行公共镜像是非常有用的，但要解锁 Docker 的真正威力，我们还需要能够构建和管理我们自己的镜像。

## 从 Dockerfiles 构建镜像

正如我们在之前的示例中所看到的，如果你的系统中没有指定的容器镜像，Puppet 的`docker::image`资源会从 Docker Hub 拉取该镜像并将其保存在本地。

然而，`docker::image`资源最常用的功能是**构建**Docker 镜像。这通常是通过 Dockerfile 来完成的，下面是一个我们可以用来构建镜像的示例 Dockerfile（`Dockerfile.hello`）：

```
FROM library/alpine:3.6
CMD /bin/sh -c "while true; do echo Hello, world; sleep 1; done"

LABEL org.label-schema.vendor="Bitfield Consulting" \
  org.label-schema.url="http://bitfieldconsulting.com" \
  org.label-schema.name="Hello World" \
  org.label-schema.version="1.0.0" \
  org.label-schema.vcs-url="github.com:bitfield/puppet-beginners-guide.git" \
  org.label-schema.docker.schema-version="1.0"
```

`FROM`语句告诉 Docker 从许多可用的公共镜像中选择一个作为基础镜像。`FROM scratch`会从一个完全空的容器开始。`FROM library/ubuntu`则会使用官方的 Ubuntu Docker 镜像。

当然，容器的一个主要优势是它们可以根据需要小到多小，大到多大，因此如果你只需要运行`/bin/echo`，下载包含整个 Ubuntu 的 188MB 镜像是没有必要的。

Alpine 是另一种 Linux 发行版，旨在尽可能小巧和轻量化，非常适合用于容器。`library/alpine`镜像仅有 4MB，比`ubuntu`小四十倍；这节省了大量空间。此外，如果你从相同的基础镜像构建所有容器，Docker 的分层系统意味着它只需要下载和存储基础镜像一次。

### 提示

Dockerfile 可以非常简单，如本示例，或者非常复杂。你可以通过 Docker 文档了解更多关于 Dockerfile 格式和命令的内容：

[`docs.docker.com/engine/reference/builder/`](https://docs.docker.com/engine/reference/builder/)

以下代码展示了如何从这个文件创建一个 Docker 镜像（`docker_build_hello.pp`）：

```
docker::image { 'pbg-hello':
  docker_file => '/examples/Dockerfile.hello',
  ensure      => latest,
}
```

一旦`docker::image`资源被应用，生成的`pbg-hello`镜像将可以用于以容器形式运行（`docker_run_hello.pp`）：

```
docker::run { 'pbg-hello':
  image => 'pbg-hello',
}
```

## 管理 Dockerfiles

当你在容器中运行自己的应用程序，或者在自己的容器中运行第三方应用程序时，你可以通过 Puppet 管理相关的 Dockerfile。下面是一个简单的 Dockerfile 示例，它构建一个使用 Nginx 来提供带有友好问候信息的网页的容器（`Dockerfile.nginx`）：

```
FROM nginx:1.13.3-alpine
RUN echo "Hello, world" >/usr/share/nginx/html/index.html

LABEL org.label-schema.vendor="Bitfield Consulting" \
  org.label-schema.url="http://bitfieldconsulting.com" \
  org.label-schema.name="Nginx Hello World" \
  org.label-schema.version="1.0.0" \
  org.label-schema.vcs-url="github.com:bitfield/puppet-beginners-guide.git" \
  org.label-schema.docker.schema-version="1.0"
```

这是管理此 Dockerfile 并从中构建镜像的 Puppet 清单（`docker_build_nginx.pp`）：

```
file { '/tmp/Dockerfile.nginx':
  source => '/examples/Dockerfile.nginx',
  notify => Docker::Image['pbg-nginx'],
}

docker::image { 'pbg-nginx':
  docker_file => '/tmp/Dockerfile.nginx',
  ensure      => latest,
}
```

运行以下命令以应用此清单：

```
sudo puppet apply /examples/docker_build_nginx.pp

```

每当 Dockerfile 的内容发生更改时，应用此清单将导致镜像被重建。

### 提示

出于本示例的目的，我们在同一节点上构建并运行容器。然而，在实际应用中，你应该在专用的构建节点上构建容器，并将生成的镜像上传到注册表，这样生产节点就可以下载并运行它们。

这是运行我们刚刚构建的容器的清单（`docker_run_nginx.pp`）：

```
docker::run { 'pbg-nginx':
  image         => 'pbg-nginx:latest',
  ports         => ['80:80'],
  pull_on_start => true,
}
```

### 提示

请注意`pull_on_start`属性，它告诉 Puppet 在启动或重新启动容器时始终下载最新版本的容器。

如果你完成了第七章，*掌握模块*，Apache Web 服务器将运行并监听端口`80`，因此你需要运行以下命令将其移除，然后再应用此清单：

```
sudo apt-get -y --purge remove apache2
sudo service docker restart
sudo puppet apply --environment pbg /examples/docker_run_nginx.pp

```

你可以通过在本地机器上访问以下 URL 来检查容器是否正常工作：

`http://localhost:8080`

你应该看到文本`Hello, world`。

### 提示

如果你使用的是 Vagrant box，本地机器上的端口`8080`将自动映射到虚拟机的端口`80`，然后 Docker 将其映射到`pbg-nginx`容器的端口`80`。如果由于某种原因你需要更改此端口映射，请编辑你的 Vagrantfile（在 Puppet 初学者指南仓库中）并查找以下行：

```
  config.vm.network "forwarded_port", guest: 80, host: 8080
```

根据需要更改这些设置，并在本地机器的 PBG 仓库目录中运行以下命令：

```
vagrant reload

```

如果你没有使用 Vagrant box，容器的端口`80`将暴露在本地端口`80`上，所以 URL 将简单地显示如下：

`http://localhost`

# 构建动态容器

尽管 Dockerfile 是一种相当强大和灵活的构建容器的方式，但它们只是静态文本文件，通常你需要将信息传递到容器中，告诉它该做什么。我们可以称这些容器为——其配置灵活并基于构建时可用的数据——**动态容器**。

## 使用模板配置容器

配置容器的一个方法是使用 Puppet 管理 Dockerfile 作为 EPP 模板（参见第九章，*管理文件与模板*），并插入所需的数据（这些数据可能来自 Hiera、Facter 或直接来自 Puppet 代码）。

让我们升级之前的`Hello, world`示例，让 Nginx 在构建时由 Puppet 提供任何任意的文本字符串。

这是从模板生成 Dockerfile 并运行生成的镜像的清单（`docker_template.pp`）：

```
file { '/tmp/Dockerfile.nginx':
  content => epp('/examples/Dockerfile.nginx.epp',
    {
      'message' => 'Containers rule!'
    }
  ),
  notify => Docker::Image['pbg-nginx'],
}

docker::image { 'pbg-nginx':
  docker_file => '/tmp/Dockerfile.nginx',
  ensure      => latest,
  notify      => Docker::Run['pbg-nginx'],
}

docker::run { 'pbg-nginx':
  image         => 'pbg-nginx:latest',
  ports         => ['80:80'],
  pull_on_start => true,
}
```

使用以下命令应用此清单：

```
sudo puppet apply --environment pbg /examples/docker_template.pp

```

当你应用了清单并构建了容器时，你会发现如果你更改 `message` 的值并重新应用，容器将会使用更新后的文本重新构建。`docker::image` 资源使用 `notify` 告诉 `docker::run` 资源在镜像变化时重启容器。

### 提示

像这样模板化 Dockerfile 是一种强大的技术。因为你可以让 Puppet 将任何任意数据放入 Dockerfile 中，你可以配置容器及其构建过程的任何内容：基础镜像、安装的软件包列表、应添加到容器中的文件和数据，甚至是容器的命令入口点。

## 自配置容器

让我们进一步拓展这个思路，使用 Puppet 动态配置一个可以从 Git 获取数据的容器。我们不再像构建时那样提供静态文本，而是让容器本身从 Git 仓库中检出网站内容。

上一个示例中的大部分代码保持不变，唯一不同的是 Dockerfile 资源（`docker_website.pp`）：

```
file { '/tmp/Dockerfile.nginx':
  content => epp('/examples/Dockerfile.website.epp',
    {
      'git_url' => 'https://github.com/bitfield/pbg-website.git'
    }
  ),
  notify  => Docker::Image['pbg-nginx'],
}

docker::image { 'pbg-nginx':
  docker_file => '/tmp/Dockerfile.nginx',
  ensure      => latest,
  notify      => Docker::Run['pbg-nginx'],
}

docker::run { 'pbg-nginx':
  image         => 'pbg-nginx:latest',
  ports         => ['80:80'],
  pull_on_start => true,
}
```

Dockerfile 本身稍微复杂一些，因为我们需要在容器中安装 Git，并使用它来检出提供的 Git 仓库（`Dockerfile.website.epp`）：

```
<% | String $git_url | -%>
FROM nginx:1.13.3-alpine
RUN apk update \
  && apk add git \
  && cd /usr/share/nginx \
  && mv html html.orig \
  && git clone <%= $git_url %> html

LABEL org.label-schema.vendor="Bitfield Consulting" \
  org.label-schema.url="http://bitfieldconsulting.com" \
  org.label-schema.name="Nginx Git Website" \
  org.label-schema.version="1.0.0" \
  org.label-schema.vcs-url="github.com:bitfield/puppet-beginners-guide.git" \
  org.label-schema.docker.schema-version="1.0"
```

当你应用这个清单并访问`http://localhost:8080`时，你应该会看到如下文本：

```
Hello, world!
This is the demo website served by the examples in Chapter 10, 'Controlling containers', from the Puppet Beginner's Guide.
```

尽管我们直接将 `git_url` 参数传递给 Dockerfile 模板，但这些数据当然可以来自任何地方，包括 Hiera。使用这种技术，你只需更改传递给它的 Git URL 就可以构建一个容器来服务任何网站。

使用我们在本章前面看到的 `docker_run_many` 示例中的迭代模式，你可以从一组 `git_url` 值中构建一组容器，每个容器服务一个不同的网站。现在我们真正开始发挥 Docker 和 Puppet 的强大功能了。

在继续下一个示例之前，运行以下命令停止容器：

```
sudo docker stop pbg-nginx

```

这个想法有一个小问题。尽管让容器能够在构建时确定的 Git 仓库中提供内容是很好的，但每次容器启动或重启时，它都必须再次执行 `git clone` 过程。这需要时间，并且如果由于某些原因仓库或网络不可用，可能会导致容器无法正常工作。

更好的解决方案是从持久化存储中提供内容，我们将在下一节中看到如何做到这一点。

# 容器的持久化存储

容器被设计为临时的；它们运行一段时间后就消失。容器内的任何东西都会随着容器的消失而消失，包括容器运行期间创建的文件和数据。当然，这并不是我们总是希望的结果。例如，如果你在容器中运行数据库，通常你希望容器消失后数据能够持久保存。

有两种方法可以在容器中持久化数据：第一种是从主机机器挂载一个目录到容器内，这种方式被称为 **主机挂载卷**；第二种是使用所谓的 **Docker 卷**。我们将在以下章节中讨论这两种方法。

## 主机挂载卷

如果你希望容器能够访问主机机器的文件系统上的文件（例如，你正在工作的应用程序代码，且希望进行测试），最简单的方法是将主机上的目录挂载到容器中。以下示例演示了如何做到这一点（`docker_mount.pp`）：

```
docker::run { 'mount_test':
  image   => 'library/alpine:3.6',
  volumes => ['/tmp/container_data:/mnt/data'],
  command => '/bin/sh -c "echo Hello, world >/mnt/data/hello.txt"',
}
```

`volumes` 属性指定了要附加到容器的卷的数组。如果卷的形式是 `HOST_PATH:CONTAINER_PATH`，Docker 会假定你想要将主机上的目录 `HOST_PATH` 挂载到容器中，容器中的路径将是 `CONTAINER_PATH`。挂载目录中已经存在的任何文件将对容器可访问，容器写入该目录的任何内容，在容器停止后仍然可用。

如果应用此示例清单，容器将挂载主机机器的 `/tmp/container_data/` 目录（如果该目录不存在，将会创建）到容器中的 `/mnt/data/`。

`command` 属性告诉容器将字符串 `Hello, world` 写入文件 `/mnt/data/hello.txt`。

运行以下命令以应用此清单：

```
sudo puppet apply /examples/docker_mount.pp

```

容器将启动，写入数据，然后退出。如果一切顺利，你会看到文件 `/tmp/container_data/hello.txt` 现在已经存在，并包含容器写入的数据：

```
cat /tmp/container_data/hello.txt
Hello, world
```

主机挂载的卷在容器需要访问或与主机机器上运行的应用程序共享数据时非常有用。例如，你可以使用一个主机挂载的卷，与一个容器一起运行语法检查、代码检查或持续集成测试，针对你的源代码目录进行操作。

然而，使用主机挂载卷的容器不可移植，它们依赖于主机机器上存在特定的目录。你无法在 Dockerfile 中指定主机挂载卷，因此你不能发布一个依赖于主机挂载卷的容器。尽管主机挂载卷在测试和开发中可能很有用，但在生产环境中，更好的解决方案是使用 Docker 卷。

## Docker 卷

一种为容器添加持久存储的更具移植性的方法是使用 **Docker 卷**。这是一种持久的数据对象，存储在 Docker 的存储区域内，并可以附加到一个或多个容器。

以下示例演示了如何使用 `docker::run` 启动一个带有 Docker 卷（`docker_volume.pp`）的容器：

```
docker::run { 'volume_test':
  image   => 'library/alpine:3.6',
  volumes => ['pbg-volume:/mnt/volume'],
  command => '/bin/sh -c "echo Hello from inside a Docker volume >/mnt/volume/index.html"',
}
```

### 提示

`volumes`属性与前面的例子稍有不同。它的形式为`VOLUME_NAME:CONTAINER_PATH`，这告诉 Docker 这不是一个主机挂载卷，而是一个名为`VOLUME_NAME`的 Docker 卷。如果冒号前的值是路径，Docker 假定你想从主机机器挂载该路径，否则，它假定你想挂载一个具有指定名称的 Docker 卷。

如前面的例子所示，容器的`command`参数将消息写入挂载卷上的文件。

如果你应用此清单，一旦容器退出，你可以运行以下命令查看卷是否仍然存在：

```
sudo docker volume ls
DRIVER              VOLUME NAME
local               pbg-volume
```

Docker 卷是一种存储数据的好方式，即使容器不运行时也需要保留这些数据（例如数据库）。它也是一种让数据对容器可用的好方式，无需每次容器启动时都加载数据。

在本章早些时候的 Web 网站示例中，与你让每个容器检出自己的 Git 仓库副本不同，你可以将仓库检出到一个 Docker 卷中，然后让每个容器在启动时挂载该卷。

让我们通过以下清单（`docker_volume2.pp`）测试这个想法：

```
docker::run { 'volume_test2':
  image   => 'nginx:alpine',
  volumes => ['pbg-volume:/usr/share/nginx/html'],
  ports   => ['80:80'],
}
```

这就是我们在本章早些时候使用的相同`nginx`容器，它将其`/usr/share/nginx/html`目录中的内容作为网站提供。

`volumes`属性告诉容器将`pbg-volume`卷挂载到`/usr/share/nginx/html`。

运行以下命令以应用此清单：

```
sudo docker stop pbg-nginx
sudo puppet apply /examples/docker_volume2.pp

```

如果一切按预期工作，我们应该能够在本地机器上浏览以下 URL：`http://localhost:8080/`

我们应该看到以下文本：

```
Hello from inside a Docker volume
```

这是容器的一个非常强大的功能。它们可以读取、写入和修改由其他容器创建的数据，保持自己的持久化存储，并通过卷与其他正在运行的容器共享数据。

在 Docker 中运行应用程序的常见模式是使用多个相互通信的容器，每个容器提供一个特定的服务。例如，一个 Web 应用程序可能使用 Nginx 容器向用户提供应用程序，同时将会话数据存储在一个 MySQL 容器中，该容器挂载了一个持久化卷。它还可以使用链接的 Redis 容器作为内存中的键值存储。

除了通过卷共享数据之外，这些容器如何通过网络进行通信呢？我们将在下一节看到答案。

# 网络与调度

我们在本章开始时说过，容器是完全自包含的，彼此之间没有访问权限，即使它们在同一主机上运行。但为了运行真正的应用程序，我们需要容器之间进行通信。幸运的是，有一种方法可以做到这一点：**Docker 网络**。

## 连接容器

Docker 网络就像是容器的私人聊天室：网络内的所有容器可以相互通信，但它们不能与网络外或其他网络中的容器通信，反之亦然。你只需要让 Docker 创建一个网络，给它命名，然后就可以在这个网络内启动容器，容器们就能相互通信。

让我们开发一个示例来试验一下。假设我们想在容器中运行 Redis 数据库，并从另一个容器向其发送数据。这是许多应用程序的常见模式。

在我们的示例中，我们将创建一个 Docker 网络，并在其中启动两个容器。第一个容器是一个公共的 Docker Hub 镜像，将运行 Redis 数据库服务器。第二个容器将安装 Redis 客户端工具，并将一些数据写入 Redis 服务器容器。然后，为了检查是否成功，我们可以尝试从服务器读取数据。

运行以下命令应用 Docker 网络示例清单：

```
sudo puppet apply /examples/docker_network.pp

```

如果一切按预期工作，我们的 Redis 数据库现在应该包含一个名为`message`的字段，里面有一条友好的问候信息，证明我们已经通过 Docker 网络将数据从一个容器传输到另一个容器。

运行以下命令连接到客户端容器并检查情况：

```
sudo docker exec -it pbg-redis redis-cli get message
"Hello, world"
```

那么它是如何工作的呢？让我们看看示例清单。首先，我们使用 Puppet 中的`docker_network`资源（`docker_network.pp`）为两个容器创建网络：

```
docker_network { 'pbg-net':
  ensure => present,
}
```

现在，我们运行 Redis 服务器容器，使用公开的`redis:4.0.1-alpine`镜像。

```
docker::run { 'pbg-redis':
  image => 'redis:4.0.1-alpine',
  net   => 'pbg-net',
}
```

### 注意

你注意到我们为`docker::run`资源提供了`net`属性吗？这指定了容器应该运行的 Docker 网络。

接下来，我们构建一个安装了 Redis 客户端（`redis-cli`）的容器，这样我们就可以用它向 Redis 容器写入一些数据。

这是客户端容器的 Dockerfile（`Dockerfile.pbg-demo`）：

```
FROM nginx:1.13.3-alpine
RUN apk update \
  && apk add redis

LABEL org.label-schema.vendor="Bitfield Consulting" \
  org.label-schema.url="http://bitfieldconsulting.com" \
  org.label-schema.name="Redis Demo" \
  org.label-schema.version="1.0.0" \
  org.label-schema.vcs-url="github.com:bitfield/puppet-beginners-guide.git" \
  org.label-schema.docker.schema-version="1.0"
```

我们通过常规方式使用`docker::image`构建这个容器：

```
docker::image { 'pbg-demo':
  docker_file => '/examples/Dockerfile.pbg-demo',
  ensure      => latest,
}
```

最后，我们使用`docker::run`运行客户端容器的实例，并传入一个命令给`redis-cli`，将一些数据写入另一个容器。

```
docker::run { 'pbg-demo':
  image   => 'pbg-demo',
  net     => 'pbg-net',
  command => '/bin/sh -c "redis-cli -h pbg-redis set message \"Hello, world\""',
}
```

如你所见，这个容器也有`net => 'pbg-net'`属性。因此，它将在与`pbg-redis`容器相同的 Docker 网络中运行，两个容器将能够互相通信。

当容器启动时，`command`属性调用`redis-cli`并执行以下命令：

```
redis-cli -h pbg-redis set message "Hello, world"

```

`-h pbg-redis`参数告诉 Redis 连接到主机`pbg-redis`。

### 注意

使用`pbg-redis`名称是如何连接到正确的容器的？当你在网络中启动一个容器时，Docker 会自动配置容器内的 DNS 查找，以便通过名称找到网络中的其他容器。当你引用一个容器名称（即容器的`docker::run`资源的标题，在我们的例子中是`pbg-redis`）时，Docker 会将网络连接路由到正确的位置。

命令`set message "Hello, world"`创建了一个名为`message`的 Redis 键，并赋值为`"Hello, world"`。

我们现在拥有了容器化一个真实应用所需的所有技术：使用 Puppet 管理多个容器，这些容器由动态数据构建，推送到注册表，按需更新，通过网络通信，监听外部端口，并通过卷持久化和共享数据。

## 容器编排

在本章中，我们已经看到了一些管理单个容器的方法，但如何在大规模和跨多个主机上配置和管理容器——我们称之为容器**编排**——的问题仍然存在。

例如，如果你的应用运行在容器中，你可能不会仅仅运行一个容器实例：你需要运行多个实例，并将流量路由和负载均衡到它们。你还需要能够将容器分布到多个主机上，以便应用能够抵御任何单个容器主机的故障。

## 什么是编排？

当在分布式集群中运行容器时，你还需要能够处理诸如容器和主机之间的网络连接、故障转移、健康监控、滚动更新、服务发现以及通过键值数据库在容器间共享配置数据等问题。

尽管容器编排是一个广泛的任务，不同的工具和框架侧重于它的不同方面，但编排的核心要求包括：

+   **调度**：在集群中运行容器，并决定将哪些容器运行在哪些主机上，以提供给定的服务。

+   **集群管理**：监控和调度集群中容器和主机的活动，并添加或删除主机。

+   **服务发现**：赋予容器找到并连接它们所需的服务和数据的能力。

## 有哪些编排工具可用？

谷歌的 Kubernetes 和 Docker 的 Swarm 都是为容器编排而设计的。另一个产品，Apache Mesos，是一个集群管理框架，可以在不同类型的资源上操作，包括容器。

今天大多数生产环境中的容器都在这三种编排系统之一下运行。Kubernetes 存在时间最久，且拥有最大的用户基础，而 Swarm 虽然是一个相对较新的工具，但它是 Docker 官方堆栈的一部分，因此正迅速被采纳。

由于所有这些产品都必然需要相对复杂的设置和操作，因此也有 **平台即服务**（**PaaS**）编排的选项：本质上，就是在托管的云平台上运行你的容器。**Google 容器引擎**（**GKE**）是作为服务的 Kubernetes；亚马逊的 **EC2 容器服务**（**ECS**）是一个类似于 Kubernetes 的专有系统。

到目前为止，Puppet 与容器编排器的集成仍然相对有限，处于初期阶段。不过，考虑到容器的流行，这一领域很可能会迅速发展。目前，Puppet 支持从 Puppet 资源生成 Kubernetes 配置，以及管理 Amazon ECS 资源，但可以公平地说，使用 Puppet 在规模化的容器编排自动化方面仍处于起步阶段。不过，值得关注这一领域的动态。

# 在容器中运行 Puppet。

如果一个容器可以包含整个操作系统，例如 Ubuntu，你可能会想：“我能不能直接在容器内部运行 Puppet？”

你可以这样做，而且有些人确实采用了这种管理容器的方法。这种方法也有不少优点：

+   你可以使用现有的 Puppet 清单或 Forge 模块；无需编写复杂的 Dockerfile。

+   Puppet 会持续保持容器更新；当某些内容发生变化时，无需重新构建。

当然，也有一些缺点：

+   安装 Puppet 会显著增加镜像大小，并且会拉入各种依赖项。

+   在容器中运行 Puppet 会减慢构建过程，并消耗容器中的资源。

还有一些混合选项，例如在构建阶段在容器中运行 Puppet，然后在保存最终镜像之前，移除 Puppet 及其依赖项，以及任何中间构建产物。

Puppet 的 `image_build` 模块是一个有前景的新方法，可以直接从 Puppet 清单构建容器，我预计在不久的将来这一领域会有快速进展。

## 容器是迷你虚拟机还是单一进程？

你倾向于哪种选择，可能取决于你对容器的基本看法。你是否将它们视为迷你虚拟机，和你当前管理的服务器没有太大区别？还是你认为它们是临时的、轻量级的、单进程的包装？

如果你把容器当作迷你虚拟机来对待，你可能会想在容器内运行 Puppet，就像在物理和虚拟服务器上运行 Puppet 一样。另一方面，如果你认为容器只应该运行单一的进程，那么在容器中运行 Puppet 似乎不太合适。对于单进程的容器，几乎没有什么可配置的内容。

我可以理解支持迷你虚拟机方法的观点。首先，这使得将现有的应用程序和服务迁移到容器变得更加容易；你无需将它们运行在虚拟机中，而是直接将整个应用程序、支持服务和数据库连同当前的管理和监控工具一起移入容器。

然而，虽然这是一个有效的方法，但它并没有真正发挥容器固有优势的最大作用：小的镜像大小、快速部署、高效重建和可移植性。

## 使用 Puppet 配置容器

就个人而言，我是一个容器极简主义者：我认为容器应该只包含完成工作所需的内容。因此，我更喜欢从外部使用 Puppet 来管理、配置和构建我的容器，而不是从内部管理，这也是我在本章中采用这种方法的原因。

这意味着生成来自模板和 Hiera 数据的 Dockerfile，如我们在示例中所看到的，以及模板化容器所需的配置文件。你可以让 Dockerfile 在构建过程中将这些文件复制到容器中，或者将主机上的单个文件和目录挂载到容器中。

正如我们所看到的，处理共享数据的一个好方法是让 Puppet 将其写入 Docker 卷或主机上的文件，然后由所有正在运行的容器挂载（通常是只读的）。

这样做的好处是，在配置更改后，你不需要重新构建所有容器。你只需让 Puppet 将更改写入配置卷，并使用 `docker::exec` 资源触发每个容器重新加载其配置，该资源在正在运行的容器上执行指定命令。

## 容器也需要 Puppet

以免重复强调，容器化并不是使用像 Puppet 这样的配置管理工具的替代方案。事实上，对配置管理的需求更大，因为你不仅需要构建和配置容器本身，还需要存储、部署和运行它们：所有这些都需要基础设施。

和往常一样，Puppet 让这类任务变得更容易、更愉快，而且——最重要的是——更具可扩展性。

# 总结

在本章中，我们探讨了与软件部署相关的一些问题，解决这些问题的部分方案，以及容器解决方案的优势。我们简要介绍了容器技术的基础，特别是 Docker，并了解到容器是另一种配置管理问题，而 Puppet 可以帮助解决这些问题。

我们安装了 `docker_platform` 模块，并使用它在我们的虚拟机上设置了 Docker，构建并运行了简单的 Docker 容器。我们看到，当底层 Dockerfile 更改时，如何自动重建容器镜像，并了解如何在构建时使用 Puppet 动态配置 Dockerfile。

我们介绍了容器的持久化存储话题，包括主机挂载卷和 Docker 卷，并讨论了如何使用 Puppet 来管理这些存储。我们设置了一个 Docker 网络，包含两个通过网络端口交换数据的通信容器。

我们分析了在容器内部运行 Puppet 相比于使用 Puppet 从外部配置和构建容器的优缺点，还提出了一种混合策略，即 Puppet 管理附加到运行中容器的卷上的配置数据。

最后，我们已经讨论了容器编排中涉及的一些问题，并介绍了当前最受欢迎的一些平台和框架。

在下一章，我们将学习如何使用 Puppet 管理云计算资源，并通过一个深入的示例来开发一个软件定义的 Amazon EC2 基础设施。

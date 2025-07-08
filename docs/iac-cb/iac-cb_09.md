# 第九章：与 Docker 一起工作

在本章中，我们将涵盖以下内容：

+   Docker 使用概览

+   选择合适的 Docker 基础镜像

+   优化 Docker 镜像大小

+   使用标签版本化 Docker 镜像

+   在 Docker 中部署 Ruby-on-Rails Web 应用

+   使用 Docker 构建和使用 Golang 应用

+   使用 Docker 进行网络配置

+   创建更动态的容器

+   自动配置动态容器

+   使用非特权用户提高安全性

+   使用 Docker Compose 进行编排

+   对 Dockerfile 进行代码检查

+   使用 S3 存储部署私有 Docker 注册表

# 介绍

在本章中，我们将探索在开发环境中使用 Docker 的最佳实践：从 Docker 镜像优化到版本控制、安全性和网络配置，选择合适的基础 Docker 镜像的技巧，以及如何使其动态和自配置；如何利用 Docker 进行 Go 程序的交叉编译或部署 Ruby-on-Rails Web 应用。依然以开发者为中心，旨在实现尽可能高的代码质量，我们将花一些时间进行代码检查，最后部署我们自己的 Docker 注册表来存储内部镜像——既在本地存储，也在 AWS S3 上实现无限存储空间。

# Docker 使用概览

本节是 Docker 初学者的入门介绍，也可以作为其他人复习的资料。我们将学习如何快速使用 Docker 完成一些任务，例如执行 Ubuntu 容器或网络 Web 服务器、与容器共享数据、构建镜像以及访问不同于默认注册表的注册表。

## 准备工作

为了执行本食谱，您需要一个工作正常的 Docker 安装。

## 如何操作…

我们将快速操作 Docker，确保基本使用上手。

### 在 Ubuntu 16.04 容器中运行 Bash

要在 Ubuntu 容器中执行 `/bin/bash`，请使用标签 16.04（`ubuntu:16.04`）。我们的环境将是交互式的（使用 `-i`），并且希望分配一个伪终端（使用 `-t`）。我们希望在之后销毁该容器（使用 `--rm`）：

```
$ docker run -it --rm ubuntu:16.04 /bin/bash
root@d372dba0ab90:/# hostname
d372dba0ab90

```

我们已经运行了第一个容器！现在可以随意操作它。退出容器将销毁它，并且由于我们指定了 `--rm` 选项，它的内容将永久丢失。

### 在容器中运行 Nginx

Nginx 已正式打包为 Docker 容器。我们希望通过 `-p` 选项，从容器的 `80` 端口访问宿主机的 `80` 端口，且使用最新的 Nginx 版本：

```
$ docker run --rm -p 80:80 nginx

```

发出一些 HTTP 请求，例如 `curl`：

```
$ curl -IL http://localhost
HTTP/1.1 200 OK
Server: nginx/1.11.5
[…]

```

Docker 的标准输出日志显示日志如下：

```
172.17.0.1 - - [21/Nov/2016:21:21:15 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.43.0" "-"

```

也许由于某些原因，我们需要启动特定版本的 Nginx，例如：

```
$ docker run --rm -p 80:80 nginx:1.10

```

HTTP 头部将反映我们现在运行的是当前稳定版本：

```
$ curl -IL http://localhost
HTTP/1.1 200 OK
Server: nginx/1.10.2

```

### 与容器共享数据

我们希望显示我们自己的内容，而不是默认的 Nginx 页面。让我们在 `www` 目录中创建一个 `index.html` 文件，包含一些自定义内容，例如：

```
<html>
  <h1>Hello from Docker!</h1>
</html>
```

默认情况下，Nginx 在 `/usr/share/nginx/html` 提供内容；让我们使用 `-v` 选项将我们自己的目录与容器共享：

```
$ docker run --rm -p 80:80 -v ${PWD}/www:/usr/share/nginx/html nginx:1.10

```

让我们来看一下新内容的服务：

```
$ curl -L http://localhost
<html>
 <h1>Hello from Docker!</h1>
</html>

```

### 构建带有工具的容器

让我们创建自己的 Ubuntu 16.04 镜像，并在其中包含一些工具，如`curl`、`dig`和`netcat`，这样无论我们使用什么机器，都能随时使用这些工具。为了构建我们的容器，我们需要一个名为`Dockerfile`的文件，它像脚本一样按行执行，来构建最终的容器。我们知道我们想从 Ubuntu 16.04 开始，然后更新 APT 基础，最后安装我们需要的工具。让我们使用`FROM`和`RUN`指令来完成这个任务：

```
FROM ubuntu:16.04
RUN apt-get -yq update
RUN apt-get install -yq dnsutils curl netcat
```

现在使用`docker build`命令进行构建，传入容器名称，并使用`-t`选项：

```
$ docker build -t utils .
Step 1 : FROM ubuntu:16.04
 ---> 2fa927b5cdd3
Step 2 : RUN apt-get update -yq
 ---> Running in 0d8f8e01bde8
[...]
Step 3 : RUN apt-get install ruby -yq
 ---> Running in 425bfb1e8ee1
[...]
Removing intermediate container 425bfb1e8ee1
Successfully built c86310e48731

```

我们可以看到`Dockerfile`的每一行都是构建过程中的*一步*，每一步都是一个容器（因此每次都会有不同的 ID）。

现在让我们执行容器，使用`dig`进行 DNS 请求：

```
$ docker run -it --rm utils dig +short google.com
172.217.5.14

```

或者，我们可以使用`curl`，如下所示：

```
$ docker run -it --rm utils curl -I google.com
HTTP/1.1 302 Found
Cache-Control: private
Content-Type: text/html; charset=UTF-8
Location: http://www.google.ca/?gfe_rd=cr&ei=UgA1VMLPRUvF9gfJ_riACg
Content-Length: 258
Date: Wed, 23 Nov 2016 02:34:58 GMT

```

### 使用私有注册表

当没有指定其他内容时，Docker 会在本地查找该容器，然后再查找 Docker Hub（[`hub.docker.com`](https://hub.docker.com)）。不过，我们可以运行自己的注册表或使用其他注册表，例如[`quay.io/`](https://quay.io/)。其工作方式如下：我们不仅指定容器名称或`用户名/容器名称`组合，还要在它们前面加上注册表的 DNS 名称，例如[`quay.io/`](https://quay.io/)。在这里，我们将启动在 CoreOS 账户中托管的 HTTP/2 Caddy Web 服务器，该服务器位于 Quay.io 注册表：

```
$ docker run -it --rm -p 80:2015 quay.io/coreos/caddy
Activating privacy features... done.
http://0.0.0.0:2015

```

这是一个关于如何使用 Docker 的简短介绍。

## 另请参见

+   Docker 运行参考：[`docs.docker.com/engine/reference/run/`](https://docs.docker.com/engine/reference/run/)

+   Dockerfile 参考：[`docs.docker.com/engine/reference/builder/`](https://docs.docker.com/engine/reference/builder/)

+   Quay.io 替代注册表：[`quay.io/`](https://quay.io/)

+   Docker Hub：[`hub.docker.com/`](https://hub.docker.com/)

+   Docker Store：[`store.docker.com/`](https://store.docker.com/)

# 选择正确的 Docker 基础镜像

根据我们的最终目标，使用我们最喜欢的 Linux 发行版的镜像可能是也可能不是最佳方案。从一个完整的 CentOS 容器镜像开始可能会浪费资源，而 Alpine Linux 镜像可能没有我们所需的最完整的 libc。在其他情况下，使用我们最喜欢的编程语言的镜像可能是个好主意，也可能不是。让我们深入了解并学习在何时选择什么来源。

## 准备工作

要执行此操作，您需要一个有效的 Docker 安装。

## 如何做……

大多数常见的发行版都可以以容器的形式使用。

### 从 Ubuntu 镜像开始

Ubuntu 提供了官方镜像，并且每个镜像都带有版本号和名称标签：`ubuntu:16.04`等同于`ubuntu:xenial`。截至目前，支持的 Ubuntu 版本有 12.04（precise）、14.04（trusty）、16.04（xenial）和 16.10（yakkety）。

要在 Dockerfile 中使用 Ubuntu 镜像，执行以下命令：

```
FROM ubuntu:16.04
ENTRYPOINT ["/bin/bash"]
```

### 从 CentOS 镜像开始

CentOS 团队发布了官方容器镜像，所有镜像都带有版本标签。强烈建议使用持续更新的*滚动版本*，因为这些版本仅通过主要版本号标记，如 `centos:7`。在撰写本文时，受支持的 CentOS 版本为 CentOS 7、6 和 5。如果由于某些合规原因需要使用特定的 CentOS 7 版本，可以使用如 `centos:7.3.1611`、`centos:7.2.1511`、`centos:7.1.1503` 和 `centos:7.0.1406` 等特定标签。

要从最新的 CentOS 7 开始，在 Dockerfile 中执行以下命令：

```
FROM centos:7
ENTRYPOINT ["/bin/bash"]
```

### 从 Red Hat Enterprise Linux（RHEL）镜像开始

Red Hat 也发布了 RHEL 的容器镜像。撰写时，这些镜像托管在 Red Hat 的 Docker 注册服务器上（[`access.redhat.com/containers/`](https://access.redhat.com/containers/)）。这些镜像没有使用发行版版本标记，而是直接用镜像的名称标记：`rhel7` 代表 RHEL 7，`rhel6` 代表 RHEL 6。类似地，子版本也直接体现在镜像名称中：RHEL 7.3 的镜像名为 `rhel7.3`。

要从最新的 RHEL 7 开始，在 Dockerfile 中执行以下命令：

```
FROM registry.access.redhat.com/rhel7
ENTRYPOINT ["/bin/bash"]
```

### 从 Fedora 镜像开始

Fedora 官方为 Docker 构建，每个版本都会简单地标记为其版本号。Fedora 25 的标签是 `fedora:25`，撰写时版本号最早可以追溯到 `fedora:20`。

要从最新的 Fedora 版本开始，在 Dockerfile 中使用以下命令：

```
FROM fedora:latest
ENTRYPOINT ["/bin/bash"]
```

### 从 Alpine Linux 镜像开始

Alpine Linux 是容器世界中非常流行且安全的轻量级 Linux 发行版。它的体积比其他主流发行版小数十倍：不到 5 MB。它变得如此流行，以至于 Docker（公司）现在将其作为所有官方镜像的基础——而 Alpine 的创始人现在也在 Docker 工作。Alpine 的版本可以在镜像标签中找到：Alpine 3.1 为 `alpine:3.1`，类似地，Alpine 3.4 为 `alpine:3.4`。

要从 Alpine Linux 的 3.4 版本开始，在 Dockerfile 中使用以下命令：

```
FROM alpine:3.4
ENTRYPOINT ["/bin/sh"]
```

### 从 Debian 镜像开始

Debian 发行版也有多个不同的标签：我们可以找到常见的 `debian:stable`、`debian:unstable` 和 `debian:sid`，以及一些其他标签，如 `debian:oldstable`。发行版的名称会像相应的版本一样标记，因此镜像 `debian:8` 与 `debian:jessie` 相同。Debian 为每个发行版发布*精简版*镜像：debian:jessie-slim 比主版本小 30%（撰写时为 80 MB，相较于 126 MB）。

要从 Debian 8（Jessie）版本开始，在 Dockerfile 中使用以下命令：

```
FROM debian:jessie
ENTRYPOINT ["/bin/bash"]
```

### Linux 发行版容器镜像大小表

下面是当前各个引用镜像的大小表：

| Linux 发行版镜像 | 大小 |
| --- | --- |
| Alpine 3.4 | 4.799 MB |
| Debian 8 (slim) | 80 MB |
| Debian 8 | 123 MB |
| Ubuntu 16.04 | 126.6 MB |
| RHEL 7.3 | 192.5 MB |
| CentOS 7.3 | 191.8 MB |
| Fedora 25 | 199.9 MB |

有了这些信息，我们现在可以决定选择其中任何一个。

尽管如此，许多流行的编程语言（Go、Node、Java、Python、Ruby、PHP 等）也在发布自己的容器镜像。它们通常基于前面表格中的操作系统容器镜像。如果我们的产品确定要使用相应的语言，使用这些镜像将会很有趣，因为它们通常提供定制的版本和功能。

### 从 Node JS 镜像开始

Node Docker 镜像的官方仓库包含多个标签版本和多个基础镜像：`node:7` 基于 Debian Jessie，而 `node:7-alpine` 基于 Alpine 3.4。`node:7-slim` 将基于精简版的 Debian Jessie，如果我们想在 Debian Wheezy 上运行 Node 7，还有 `node:7-wheezy`。另外，Node 6、4 及更低版本也有提供。

要从最新的 Node 7 镜像版本开始，可以在 Dockerfile 中使用以下内容：

```
FROM node:7
ENTRYPOINT ["/bin/bash"]
```

需要说明的是，`node:7` 镜像大约为 650 MB，而 `node:4-slim` 镜像大约为 205 MB。

### 从 Golang 镜像开始

Go 作为 Docker 镜像被广泛分发。它的版本通过版本号进行标签（例如 `golang:1.7`），还有基于 Alpine 的替代版本（`golang:1.7-alpine`）或甚至适用于 Windows Server 的版本（`golang:1.7-windowsservercore` 和 `golang:1.7-nanoserver`）。

要从 Go 镜像开始，可以在 Dockerfile 中使用以下内容：

```
FROM golang:1.7
ENTRYPOINT ["/bin/bash"]
```

主 Go `1.7` 镜像为 672 MB。

### 从 Ruby 镜像开始

Ruby 也作为官方 Docker 镜像发布：所有最新的版本都以 `ruby:2.3` 的标签形式存在。此外，还有来自 Alpine Linux 和 Debian Jessie 精简版镜像的替代构建版本。

### 注意

曾经有一个独立的 Ruby-on-Rails Docker 镜像，但现在已被弃用，转而使用主 Ruby Docker 镜像。

要从 Ruby `2.3` 镜像开始，可以使用以下内容启动 Dockerfile：

```
FROM ruby:2.3
ENTRYPOINT ["/bin/bash"]
```

主 Ruby `2.3` 镜像为 725 MB。

### 从 Python 镜像开始

Python 官方发布并支持多个版本的 Docker 镜像。我们可以找到版本 2.7、3.3、3.4、3.5 和当前的 beta 版本，这些版本基于 Debian Jessie 或 Wheezy、Alpine 和 Windows Server。

要使用 Python `3.5` 镜像启动我们的项目，可以在 Dockerfile 中添加以下内容：

```
FROM python:3.5
ENTRYPOINT ["/bin/bash"]
```

主 `python:3.5` 镜像大约为 683 MB。

### 从 Java 镜像开始

Java 用户也可以在 Docker 上获取官方发布版本。OpenJDK 和 JRE 都可以使用，适用于 6、7、8 和 9 版本，基于 Debian Jessie 或 Alpine。

要使用 OpenJDK 9 镜像，可以在 Dockerfile 中使用以下内容：

```
FROM openjdk:9
ENTRYPOINT ["/bin/bash"]
```

主 `openjdk:9` 镜像为 548 MB——是可用的最小的编程语言镜像之一。

### 从 PHP 镜像开始

PHP Docker 镜像非常流行，并且有很多不同的版本。它是轻松测试新旧 PHP 版本的一种非常简便的方式。PHP 5.6 和 7.0（以及所有的 beta 版本）都有提供，而且每个版本也有不同的变种，例如基于 Alpine 的`php:7-alpine`，基于 Debian Jessie 的 Apache 版本`php:7-apache`，或者基于 Debian Jessie 的 FPM 版本`php:7-fpm`，但如果我们仍然喜欢 Alpine 版本的 FPM，也可以使用`php:7-fpm-alpine`。

要开始使用经典的 PHP 7 Docker 镜像，在 Dockerfile 中使用以下内容：

```
FROM php:7
ENTRYPOINT ["/bin/bash"]
```

主要的`php:7`镜像为 363 MB——这是目前最小的编程语言镜像。

## 另见

+   Docker Hub 上的镜像: [`hub.docker.com/explore/`](https://hub.docker.com/explore/)

+   Red Hat 容器目录: [`access.redhat.com/containers`](https://access.redhat.com/containers)

# 优化 Docker 镜像大小

Docker 镜像是按照 Dockerfile 中的指令逐步生成的。尽管完全正确，但在谈到镜像大小时，很多镜像在优化上存在不足。让我们通过在 Ubuntu 16.04 上构建 Apache Docker 容器来看看能做些什么。

## 准备工作

要执行这个操作，你需要一个正常工作的 Docker 安装。

## 如何实现……

以下是更新 Ubuntu 镜像、安装`apache2`包，并删除`/var/lib/apt`缓存文件夹的`Dockerfile`。它完全正确，如果你构建它，镜像大小大约是 260 MB：

```
FROM ubuntu:16.04
RUN apt-get update -y
RUN apt-get install -y apache2
RUN rm -rf /var/lib/apt
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

现在，每一层都是在上一层的基础上添加的。所以，在`apt-get update`这层中所写的内容会一直保留，即使我们在最后的`RUN`命令中删除它。

让我们用一行代码重写这个`Dockerfile`，以节省一些空间：

```
FROM ubuntu:16.04
RUN apt-get update -y && \
    apt-get install -y apache2 && \
    rm -rf /var/lib/apt/
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

这个镜像与原镜像完全相同，但大小仅为 220 MB。节省了 15%的空间！

用`debian:stable-slim`镜像替代`ubuntu:16.04`镜像可以得到相同的结果，但大小为 135 MB（大小减少了 48%！）：

```
FROM debian:stable-slim
RUN apt-get update -y && \
    apt-get install -y apache2 && \
    rm -rf /var/lib/apt/
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

## 它是如何工作的……

每一层都在其前一层之上添加。通过将下载到删除的所有相关命令合并在一起，我们在这个特定的层上保持了一个干净的状态。另一个很好的例子是，当 Dockerfile 下载一个压缩包时；分别下载、解压和删除这个压缩包会使用大量的额外层空间。如果把这些操作写成一行代码，它们会一次性完成。所以，代替从压缩包及其解压后的内容中累积的空间，所占空间仅来自解压后的内容。通常，这样做会带来非常可观的空间节省！

# 使用标签对 Docker 镜像进行版本控制

一个非常常见的需求是快速识别一个 Docker 镜像正在运行的软件版本，并选择是否固定版本，或者确保始终运行一个稳定版本。这是 Docker 标签的完美应用场景。我们将构建一个 Terraform 容器，使用稳定和不稳定标签，这样多个版本可以并存——一个用于生产，另一个用于测试。

### 注意

Docker *标签*与 Docker *标签*不同。标签是纯粹的信息性，而标签则可以直接请求，以便从操作角度区分镜像。

## 准备工作

要按照此配方操作，您需要一个正常工作的 Docker 安装。

## 如何操作…

这是一个简单的 Dockerfile，用于创建 Terraform 容器（Terraform 在本书前面已经介绍过）：

```
FROM alpine:latest
ENV TERRAFORM_VERSION=0.7.12
VOLUME ["/data"]
WORKDIR /data
RUN apk --update --no-cache add ca-certificates openssl && \
  wget -O terraform.zip "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip" && \
  unzip terraform.zip -d /bin && \
  rm -rf terraform.zip /var/cache/apk/*
ENTRYPOINT ["/bin/terraform"]
CMD [ "--help" ]
```

这是当前的稳定和最新版本，也是 0.7.12。我们希望我们的用户能够请求以下版本之一：

+   `terraform:latest`（对于那些始终需要最新版本的用户）

+   `terraform:stable`（对于那些始终需要稳定版本的用户，而不是测试版）

+   `terraform:0.7.12`（对于那些始终需要特定版本的用户，例如由于兼容性问题）

通过直接构建所有这些不同的标签，轻松实现这一目标：

```
$ docker build -t terraform:latest -t terraform:stable -t terraform:0.7.12 .

```

现在，当请求哪些镜像可用时，我们可以看到它们都有相同的镜像 ID，但具有不同的标签。这正是我们想要的，因为它是相同的镜像，分享所有这些标签：

```
$ docker images terraform
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
terraform           0.7.12              9d53a0811d63        About a minute ago   83.61 MB
terraform           latest              9d53a0811d63        About a minute ago   83.61 MB
terraform           stable              9d53a0811d63        About a minute ago   83.61 MB

```

几天后，我们发布了软件的新版本作为 Docker 容器供我们的团队测试。这一次是一个不稳定的 0.8.0-rc1 版本。我们希望我们的用户能够请求以下镜像之一：

+   `terraform:latest`（它仍然是可用的最新版本，即使是不稳定的）

+   `terraform:unstable`（这是一个发布候选版本，而不是稳定版本）

+   `terraform:0.8.0-rc1`（这是这个特定版本）

在`Dockerfile`中更改`TERRAFORM_VERSION`变量，并使用以下标签构建镜像：

```
$ docker build -t terraform:latest -t terraform:unstable -t terraform:0.8.0-rc1 .

```

现在，如果我们查看可用的 Terraform 镜像，我们可以确认它是相同的镜像 ID，`latest`、`unstable`和`0.8.0-rc1`标签共享该镜像 ID，而我们的用户如果更倾向于稳定版本，则不会受到我们更改的影响：

```
$ docker images terraform
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
terraform           0.8.0-rc1           44609fa7c016        18 seconds ago      86.77 MB
terraform           latest              44609fa7c016        18 seconds ago      86.77 MB
terraform           unstable            44609fa7c016        18 seconds ago      86.77 MB
terraform           0.7.12              9d53a0811d63        9 minutes ago       83.61 MB
terraform           stable              9d53a0811d63        9 minutes ago       83.61 MB

```

### 注意

这引出了一个非常重要的问题：由于默认情况下未指定任何标签时使用的是最新标签，是否也应该将其用于不稳定的版本？这需要根据您的需求和环境来回答。

# 在 Docker 中部署 Ruby-on-Rails Web 应用程序

Docker 的好处在于，作为开发人员，我们可以在某个环境（如开发或预发布）上将所有工作正常的内容打包到该容器中，并确信它将在另一个环境（如生产）中类似运行。部署压力较小，回滚也更容易。然而，要实现这种安心感，我们需要的不仅仅是一个 Ruby-on-Rails 应用程序，例如，我们需要打包一个包含所有内容的 Dockerfile，以便任何人都可以运行它。以下是操作方法。

## 准备工作

要按照此配方操作，您需要以下内容：

+   一个正常工作的 Docker 安装

+   一个 Rails 应用程序

## 如何操作…

这是我们的标准要求：

+   这个 Rails 应用程序需要 Ruby 2.3

+   所有依赖项都由 Bundler 管理，需要在容器中安装

+   还需要 Node 5

+   我们希望在镜像中预编译资产（将它们放到其他地方超出了此范围）。

我们将按以下步骤进行操作。为了满足我们的主要需求，我们将从 `ruby:2.3` 镜像开始：

```
FROM ruby:2.3
```

启用官方 Node 5 仓库的一种方法是下载并执行一个安装脚本。我们来做一下：

```
RUN curl -sL https://deb.nodesource.com/setup_5.x | bash -
```

现在我们需要安装 Node 5 (`apt-get install nodejs`) 并删除所有缓存文件：

```
RUN apt-get install -qy nodejs && \
  rm -rf /var/lib/apt/* && \
  rm -rf /var/lib/cache/* && \
  rm -rf /var/lib/log/* && \
  rm -rf /tmp/*
```

Ruby 镜像文档建议将 `/usr/src/app` 用作我们代码的目标文件夹。我们确保它已创建并切换到该目录，直到完成其余过程：

```
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
```

为了安装所有声明的依赖项，我们需要将 `Gemfile` 和 `Gemfile.lock` 发送到目标文件夹 `/usr/src/app`。我们将其作为一个独立的步骤，以便以后可以根据需要定制此步骤。然后我们执行 Bundler（如果有的话，跳过测试和开发部分）。如果您是 Ruby 开发人员，请根据需要进行定制！

```
COPY Gemfile /usr/src/app/
COPY Gemfile.lock /usr/src/app/
RUN bundle install --without test development --jobs 20 --retry 5
```

现在是时候将应用程序代码本身复制到目标文件夹 `/usr/src/app`（在本例中，它是当前文件夹）：

```
COPY . /usr/src/app
```

下一步是预编译资产，设置 `RAILS_ENV` 为生产环境，但您可以根据需要调整，包括编译命令：

```
RUN RAILS_ENV=production rake assets:precompile
```

最后，通过 Bundler 在所有接口上运行 Rails 服务器（默认情况下，它监听 TCP/`3000`）：

```
CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
```

我们现在可以构建这个 Dockerfile，并使我们的完整、独立且完全可用的 Ruby-on-Rails 应用程序准备好运行在 Docker 上。

### 注意

在 CI 中插入构建过程并执行测试是一个好习惯，运行这个新镜像时一定要进行测试！

# 使用 Docker 构建和使用 Golang 应用程序

Golang 是一种伟大的编程语言，能够为不同平台创建静态链接的二进制文件，如 Linux（ELF 二进制文件）或 Mac OS（Mach-O 二进制文件）。这些二进制文件通常非常小，且该语言在微服务领域日益流行，因为它的可移植性和部署速度：在几十台服务器上部署一个自给自足的 10 MB Docker 镜像比部署一个满是库的 1.5 GB 镜像要方便且快速得多。Golang 和容器是两种非常匹配的技术，使用 Go 程序来运输或管理基础设施轻松便捷。

## 准备工作

要逐步进行此配方，您需要以下内容：

+   一个有效的 Docker 安装

+   一个 Golang 应用程序源代码

## 如何操作……

假设我们的应用程序代码存放在 `src/hello` 中。我们希望至少编译程序，无论是为 Linux 平台还是为 Mac 操作系统。

### 使用 golang Docker 镜像进行 Go 程序的交叉编译

我们可以通过共享代码文件夹并将工作目录设置为该文件夹来编译我们的程序：

```
$ docker run --rm -v "${PWD}/src/hello":/usr/src/hello -w /usr/src/hello golang:1.7 go build -v

```

这样，即使在 Mac OS 系统上，我们也可以生成一个正确的 ELF 二进制文件：

```
$ file src/hello/hello
src/hello/hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped

```

也就是说，如果我们明确想要一个 Mac 二进制文件，我们可以传递标准的 Go 环境变量 `GOOS` 和 `GOARCH`，这样即使是 Linux 机器也能构建 Mac 二进制文件：

```
$ docker run --rm -v "${PWD}/src/hello":/usr/src/hello -w 
/usr/src/hello -e GOOS=darwin -e GOARCH=amd64 golang:1.7 go build -v

```

确认我们有一个 Mach-O 可执行文件，而不是 ELF 二进制文件：

```
$ file src/hello/hello
src/hello/hello: Mach-O 64-bit executable x86_64

```

### 使用 golang Docker 镜像构建并部署 Go 程序

如果我们想直接从 Dockerfile 构建程序并生成 Docker 镜像，那么可以按如下方式进行：

```
FROM golang:1.7
COPY src/hello /go/src/hello
RUN go install hello
ENTRYPOINT ["/go/bin/hello"]
```

只需构建该镜像并执行即可：

```
$ docker build -t hello .
$ docker run -it --rm hello

```

### 使用 scratch Docker 镜像

现在，对于经常只有几 MB 的小型 Golang 应用程序，使用 675 MB 以上的镜像是有些浪费空间的，而且在服务器上部署也需要时间。这里就用到了 scratch 镜像：它实际上不存在。我们从零开始，复制二进制文件并执行它。我们的构建过程（Makefile、构建过程和 CI）使用 `golang` 镜像来构建应用程序，但不会将编译后的应用程序一同打包，从而节省了通常 95-99% 的空间，具体取决于二进制文件的大小：

```
FROM scratch
COPY src/hello/hello /hello
ENTRYPOINT ["/hello"]
```

这将生成一个最小的镜像。想象一下，只有几兆字节。

### 使用 Alpine Linux 作为 Go 程序的替代方案

使用 scratch 镜像的主要问题是无法轻松地从容器内部进行调试，以及无法依赖外部库或依赖项（如 SSL 和证书）。Alpine Linux 是一个非常小的镜像（约 5 MB），如果我们希望访问 shell（`/bin/sh` 可用）和包管理器来调试应用程序，它将非常有帮助。我们可以这样做：

```
FROM alpine:latest
RUN apk --update --no-cache add ca-certificates openssl && \
    rm -rf /var/cache/apk/*
COPY src/hello/hello /bin/hello
ENTRYPOINT ["/bin/hello"]
```

这样的镜像通常比应用程序二进制文件大几个兆字节，但在调试时非常有帮助。

# 使用 Docker 进行网络连接

Docker 提供了一些非常不错的网络选项，从选择暴露哪些端口到同时运行隔离或桥接的网络。它非常有用，可以快速、轻松地模拟生产环境、创建更好的架构，并增加容器在网络前端的曝光度。我们将看到不同的端口暴露方式、如何创建新网络、如何在其中执行 Docker 容器，甚至让每个容器拥有多个网络。

## 准备工作

要按照此教程操作，您需要以下内容：

+   一个可用的 Docker 安装

+   一个示例 HTTP 服务器二进制文件（示例代码包括）

## 如何操作……

要使容器网络端口对其他人可用，首先需要将其 *暴露*。考虑任何监听端口的服务，除非正确暴露在 3 中，否则无法访问：

```
FROM debian:jessie-slim
COPY src/hello/hello /hello
EXPOSE 8000
ENTRYPOINT ["/hello"]
```

该服务正在 `8000` 端口监听，默认情况下，运行在主机上的任何其他 Docker 容器都可以访问它，且都在相同的网络上：

```
# curl -I http://172.17.0.2:8000/
HTTP/1.1 200 OK

```

然而，这个服务对主机系统不可用：

```
$ curl http://localhost:8000
curl: (7) Failed to connect to localhost port 8000: Connection refused

```

为了让其对主机系统可用，容器必须通过显式的端口重定向运行。可以使用选项 `-P` 随机映射暴露的端口（例如，端口 `8000` 可以映射到本地机器的 32768 端口），或者使用另一个选项 `-p 8000:8000` 来固定端口映射：

```
$ docker run -ti --rm -P --name hello hello 

```

在另一个终端中，查找端口重定向：

```
$ docker port hello
8000/tcp -> 0.0.0.0:32771

```

同时，尝试连接到该端口：

```
$ curl -I http://localhost:32771/
HTTP/1.1 200 OK

```

这些是与 Docker 容器进行网络连接的基础知识。

### Docker 网络

容器也可以存在于专用网络中，以增加安全性和隔离性。要创建一个新的 Docker 网络，只需给它命名即可：

```
$ docker network create hello_network
d01a3784dec1ade72b813d87c1e6fff14dc1b55fdf6067d6ed8dbe42a3af96c2

```

使用`docker network inspect`命令获取有关此网络的一些信息：

```
$ docker network inspect hello_network -f '{{json .IPAM.Config }}'
[{"Subnet":"172.18.0.0/16","Gateway":"172.18.0.1/16"}]

```

这是一个新的子网：`172.18.0.0/16`（在此情况下）。

要在这个特定的 Docker 网络中执行容器，请像这样使用`--network <docker_network_name>`选项：

```
$ docker run -it --rm --name hello --network hello_network hello

```

确认该容器位于`hello_network`网络的`172.18.0.0/16`网络空间中：

```
$ docker inspect --format '{{json .NetworkSettings.Networks.hello_network.IPAddress }}' hello
"172.18.0.2"

```

这个容器将被保护，避免任何未在正确网络上运行的容器访问。这里有一个例子，来自一个在默认网络上运行的容器：

```
# curl -I --connect-timeout 5 http://172.18.0.2:8000/
curl: (28) Connection timed out after 5003 milliseconds 

```

然而，从同一网络中的容器进行连接是允许的，并且按预期工作：

```
# curl -I http://hello:8000/
HTTP/1.1 200 OK

```

### 为一个容器连接多个网络

在多个网络上拥有一些特定容器可能会很有用；代理、内部服务和其他类似服务可能面临不同的网络配置。一个 Docker 容器可以连接多个 Docker 网络。以这个简单的 HTTP 服务为例，它监听 8000 端口并在默认的桥接网络上启动：

```
$ docker run -ti --rm --name hello hello

```

现在，这个服务可以供任何其他容器在默认网络上使用：

```
# curl -I http://172.17.0.2:8000/
HTTP/1.1 200 OK

```

然而，我们希望它也可以在*hello_network* Docker 网络上使用。让我们将它们连接到主机：

```
$ docker network connect hello_network hello

```

这个容器现在在`hello_network`子网中有了一个新的网络接口：

```
$ docker exec -it hello ip addr
[...]
116: eth0@if117: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
 link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
 inet 172.17.0.2/16 scope global eth0
[...]
118: eth1@if119: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
 link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
 inet 172.18.0.2/16 scope global eth1
[...]

```

这意味着它也可以响应来自该网络中容器的请求！

```
$ curl http://hello:8000
Hello world

```

在完成后，我们最终会移除与原始网络的链接：

```
$ docker network disconnect bridge hello

```

# 创建更动态的容器

我们可以创建比仅提前固定其用法并执行它们更好的容器。也许某些命令部分需要保留（比如我们始终希望执行 OpenVPN 二进制文件及其选项），也许所有的命令都需要被覆盖（这就是工具箱容器模型，例如默认情况下是`/bin/bash`命令，但可以执行其他传入的命令），或者两者结合，以实现一个更动态的容器。

## 准备工作

要执行此步骤，您需要一个正常工作的 Docker 安装。

## 如何操作……

要让容器执行固定命令，请使用`ENTRYPOINT`指令。如果命令后面有需要强制执行的参数，请使用数组：

```
FROM debian:stable-slim
RUN apt-get update -y && \
    apt-get install -y apache2 && \
    rm -rf /var/lib/apt/
EXPOSE 80
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

要在运行时覆盖整个命令，请使用`--entrypoint`选项：

```
$ docker run -it --rm --entrypoint /bin/sh httpd
# hostname
585dff032d21

```

要使用可以简单覆盖的命令，请使用`CMD`指令，而不是`ENTRYPOINT`：

```
FROM debian:stable-slim
RUN apt-get update -y && \
    apt-get install -y apache2 && \
    rm -rf /var/lib/apt/
EXPOSE 80
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

要覆盖命令，只需在运行时提供另一个命令作为参数：

```
$ docker run -it --rm httpd /bin/sh
# hostname
cb1c6a7083ad

```

我们可以结合这两条指令，创建一个更动态的容器。在这种情况下，我们希望获取一个始终执行`/usr/sbin/apache2ctl`的容器，并默认在前台启动守护进程，除非在容器启动时通过任何参数进行覆盖：

```
FROM debian:stable-slim
RUN apt-get update -y && \
    apt-get install -y apache2 && \
    rm -rf /var/lib/apt/
EXPOSE 80
CMD ["-D", "FOREGROUND"]
ENTRYPOINT ["/usr/sbin/apache2ctl"]
```

如果这个容器按原样执行，什么也不会改变；`apache2ctl`会使用`-D FOREGROUND`选项执行。

然而，给它传递参数后，它变成了一个更有用的容器，因为它会动态地将这些参数添加到`apache2ctl`命令中，替换掉`CMD`指令中指定的原始命令：

```
$ docker run -it --rm httpd -v
Server version: Apache/2.4.10 (Debian)
Server built:   Sep 15 2016 20:44:43

```

我们可以交互式地传递`/usr/sbin/apache2ctl`参数，而无需覆盖入口点，例如，提出备选的 Apache 配置文件或选项。

# 自动配置动态容器

我们不能总是执行二进制文件来获取我们想要的内容。动态配置是一种非常常见的情况；系统路径可以是动态的，用户和密码可以是自动生成的，网络端口可以是上下文相关的，第三方凭证在开发和生产环境中可能不同，工作节点会加入其主节点，集群成员会找到其他节点，其他类似的变化元素都需要在运行时进行适应。这里的关键是将环境变量与作为入口点执行的脚本结合起来，无论如何都会执行，并根据环境变量进行行为调整，可以选择性地与 Dockerfile 中的命令结合使用。

## 准备就绪

要按照这个配方进行操作，您需要有一个正常工作的 Docker 安装。

## 如何做……

我们的目标是创建一个临时、动态的 SSH 服务器，容器中的凭证我们无法事先知道。所以，为了按预期工作，我们需要像这样执行该容器：

```
$ docker run -e USER=john -e PASSWORD=s3cur3 sshd 

```

以这个简单的`Dockerfile`为例，它在 Alpine Docker 镜像上创建了运行 Dropbear SSH 服务器所需的环境：

```
FROM alpine:latest
RUN apk add --update openssh-sftp-server openssh-client dropbear &&\
    rm -rf /var/cache/apk/*
RUN mkdir /etc/dropbear && touch /var/log/lastlog
COPY entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]
CMD ["dropbear", "-RFEmwg", "-p", "22"]
```

构建完成后，该容器将通过执行`entrypoint.sh`脚本启动，然后是`dropbear`二进制文件。以下是一个示例`entrypoint.sh`，它仅对`USER`和`PASSWORD`环境变量进行简单检查，在容器上创建所需的用户，设置一些权限，最后执行原始 Dockerfile 中的`CMD`指令：

```
#!/bin/sh

# Checks for USER variable
if [ -z "$USER" ]; then
  echo >&2 'Please set an USER variable (ie.: -e USER=john).'
  exit 1
fi

# Checks for PASSWORD variable
if [ -z "$PASSWORD" ]; then
  echo >&2 'Please set a PASSWORD variable (ie.: -e PASSWORD=hackme).'
  exit 1
fi

echo "Creating user ${USER}"
adduser -D ${USER} && echo "${USER}:${PASSWORD}" | chpasswd
echo "Fixing permissions for user ${USER}"
chown -R ${USER}:${USER} /home/${USER}
exec "$@"
```

如果在没有任何参数的情况下执行此容器，它会报错，这得益于`entrypoint.sh`脚本中的检查：

```
$ docker run --rm ssh
Please set an USER variable (ie.: -e USER=john).

```

要正确使用这个动态配置的容器，请根据需要使用环境变量：

```
$ docker run --rm -h ssh-container -e USER=john -e PASSWORD=s3cur3 -p 22:22 ssh
Creating user john
Password for 'john' changed
Fixing permissions for user john
[1] Nov 29 23:02:02 Not backgrounding

```

现在，尝试从另一个终端或容器连接到此容器，并提供正确的凭证：

```
$ ssh john@localhost
[...]
john@localhost's password:
ssh-container:~$ hostname
ssh-container

```

我们已经登录到 SSH 容器了！

这样的动态系统可以用来为需要的人提供临时、受控且安全的 SSH 访问权限，例如共享卷存储访问或类似用途。关闭容器会撤销所有操作，我们就完成了。

# 通过无特权用户提高安全性

默认情况下，容器会以 `root` 用户身份执行所有操作。虽然容器是在隔离环境中运行的，但仍然，公开面向外部的守护进程是以 root 身份在系统上运行的，如果发生安全漏洞，攻击者可能会获得对该容器的访问权限，甚至是 root shell 访问权限，从而至少获得容器的 Docker 覆盖网络访问权限。我们是否愿意看到这个问题与一个 0-day 本地内核安全漏洞结合，进而让攻击者获得对 Docker 主机的访问权限？大概不会。那么，也许我们应该遵循一些古老的最佳实践，从一开始就以非 root 用户身份执行我们的守护进程。

## 准备工作

要完成这个配方，您需要以下内容：

+   一个工作中的 Docker 安装

+   一个示例 HTTP 服务器二进制文件（包括示例代码）

## 如何实现……

让我们以一个简单的 HTTP 服务器为例，它在容器的 `8000` 端口上提供响应。通过容器执行时，它的样子将是这样的，如本书前面所示：

```
FROM debian:jessie-slim
COPY src/hello/hello /usr/bin/hello
RUN chmod +x /usr/bin/hello
EXPOSE 8000
ENTRYPOINT ["/usr/bin/hello"]
```

这样做是有效的，但从安全角度来看情况并不理想；我们的守护进程实际上是以 `root` 用户身份运行的，尽管它是在一个非特权端口上运行：

```
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.6  0.2  36316  4180 ?        Ssl+ 23:30   0:00 /usr/bin/hello

```

从安全角度来看，这是不理想的。容器是真正的系统，因此它们也可以有用户。结合 Dockerfile 中的 `USER` 指令，我们将能够以非特权用户身份执行命令！这是一个优化过的 Dockerfile，增加了一个普通的用户和组为 `hello` 用户，然后作为这个新的非特权用户执行 `/usr/bin/hello` HTTP 服务器：

```
FROM debian:jessie-slim
COPY src/hello/hello /usr/bin/hello
RUN chmod +x /usr/bin/hello
RUN groupadd -r hello && useradd -r -g hello hello
USER hello
EXPOSE 8000
ENTRYPOINT ["/usr/bin/hello"]
```

一旦构建并运行，守护进程仍然能够正常运行，但作为一个非特权用户：

```
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
hello        1  0.0  0.2  36316  4768 ?        Ssl+ 23:33   0:00 /usr/bin/hello

```

我们现在正在构建更强大的容器！

# 使用 Docker Compose 进行编排

手动启动多个容器可能会很麻烦，尤其是在基础设施日益复杂的情况下。依赖关系、共享变量和公共网络可以通过名为 Docker Compose 的编排工具轻松处理。在一个简单的 YAML 文件中，我们可以描述运行应用程序所需的服务（代理、应用程序、数据库等）。在本节中，我们将展示如何创建一个简单的 LAMP docker-compose 文件，然后我们将展示如何从中进行迭代，构建一些适用于暂存和生产的特定更改。

## 准备工作

要完成这个配方，您需要以下内容：

+   一个工作中的 Docker 安装

+   一个工作中的 Docker Compose 安装

## 如何实现……

要使用 Docker Compose 编排多个容器，我们从一个简单的 WordPress 示例开始。WordPress 团队构建了一个容器，该容器通过类似本章前面所见的环境变量自动配置到一定程度。如果我们仅仅应用随 WordPress Docker 容器一起发布的文档，我们将得到以下 `docker-compose.yml` 文件，位于某个新目录的根目录中（如果需要，可以是 Git 仓库）：

```
version: '2'

services:
  wordpress:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_PASSWORD: example
  mysql:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: example
```

这有一个很大的优点，即开箱即用；最新的 WordPress 和 MariaDB 镜像会被下载，本地 HTTP 端口 80 会被重定向到主机的端口 8080，MySQL 保持隔离。WordPress 容器在这种情况下只需要一个环境变量——MySQL 的 root 密码，它应与 MySQL 的环境变量匹配。我们会看到，实际上还有更多的环境变量可以配置。

执行 Docker Compose 将自动创建一个 Docker 网络并启动容器：

```
$ docker-compose up
[...]
mysql_1      | 2016-12-01 20:51:14 139820361766848 [Note] mysqld (mysqld 10.1.19-MariaDB-1~jessie) starting as process 1 ...
[...]
mysql_1      | 2016-12-01 20:51:15 139820361766848 [Note] mysqld: ready for connections.
[...]
wordpress_1  | [Thu Dec 01 20:51:17.865932 2016] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.10 (Debian) PHP/5.6.28 configured -- resuming normal operations
wordpress_1  | [Thu Dec 01 20:51:17.865980 2016] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'

```

让我们验证是否能通过本地重定向端口 `8080` 连接到 WordPress HTTP 服务器：

```
$ curl -IL http://localhost:8080
HTTP/1.1 302 Found
[...]

HTTP/1.1 200 OK
[...]

```

使用 `ps` 命令可以查看更多信息：

```
$ docker-compose ps
 Name                      Command               State          Ports
-----------------------------------------------------------------------------------
1basics_mysql_1       docker-entrypoint.sh mysqld      Up      3306/tcp
1basics_wordpress_1   docker-entrypoint.sh apach ...   Up      0.0.0.0:8080->80/tcp

```

让我们通过使用 `docker-compose exec` 命令来确保 MySQL root 密码使用的确实是 `docker-compose.yml` 文件中提供的密码，这个命令非常类似于 `docker run` 命令（它接受 `docker-compose.yml` 文件中的名称）：

```
$ docker-compose exec mysql /usr/bin/mysql -uroot -pexample
[...]

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| wordpress          |
+--------------------+
4 rows in set (0.00 sec)

```

当我们完成初始的 Docker Compose 环境后，让我们销毁它；容器和网络将被移除：

```
$ docker-compose down

```

### 扩展 Docker Compose

现在我们已经掌握了基本知识，接下来我们稍微扩展一下使用。我们不满意默认密码，想使用更好的密码，以便模拟一个临时环境。为此，我们将利用 Docker Compose 的覆盖特性，创建一个 `docker-compose.staging.yml` 文件，简单地覆盖相关的值：

```
version: '2'
services:
  wordpress:
    image: wordpress:4.6
    environment:
      WORDPRESS_DB_PASSWORD: s3cur3
  mysql:
    environment:
      MYSQL_ROOT_PASSWORD: s3cur3
```

当使用多个配置文件执行 `docker-compose` 时，`WORDPRESS_DB_PASSWORD` 和 `MYSQL_ROOT_PASSWORD` 这两个环境变量会被覆盖：

```
$ docker-compose -f docker-compose.yml -f docker-compose.staging.yml up

```

验证新密码是否在 MySQL 中正常工作：

```
$ docker exec -it 1basics_mysql_1 mysql -uroot -ps3cur3
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4

```

我们非常容易地通过简单的 YAML 文件来覆盖值！

假设我们现在想要在配置中加入反向代理，使用一个稍早版本的 Docker 镜像并更换 MySQL 密码，以模拟我们在生产环境中的特定情况。我们可以使用 `jwilder/nginx-proxy` 提供的优秀动态 Nginx 镜像来完成这项工作，并在 `docker-compose.production.yml` 文件中添加一个新的 *proxy* 服务，共享端口 `80` 以及本地 Docker 套接字作为只读（以动态访问运行中的容器）：

```
  proxy:
    image: jwilder/nginx-proxy
    ports:
      - "80:80"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
```

这个 `nginx-proxy` 容器需要一个名为 `VIRTUAL_HOST` 的变量，以便在有多个虚拟主机的情况下知道该回答什么。我们将其设置为 localhost（或者根据你的本地主机名进行调整），并添加更好的密码以及 WordPress 镜像版本：

```
  wordpress:
    image: wordpress:4.5
    environment:
      WORDPRESS_DB_PASSWORD: sup3rs3cur3
      VIRTUAL_HOST: localhost
```

让 MySQL 部分的密码也匹配，我们就完成了生产环境的模拟：

```
$ docker-compose -f docker-compose.yml -f docker-compose.production.yml up

```

确认 `nginx-proxy` 在 HTTP/`80` 上正常响应，并转发来自 WordPress 容器的正确 HTTP 响应：

```
$ curl -IL http://localhost/
HTTP/1.1 302 Found
Server: nginx/1.11.3
[...]
HTTP/1.1 200 OK
Server: nginx/1.11.3

```

我们已经看到，仅用几行 YAML 配置就能轻松编排容器，如何处理不同的情况和环境，并且它也能成功扩展。不过，这只是 Docker Compose 可以实现的功能的一个小介绍——它是一个非常强大的工具！

## 另见

+   Nginx-proxy: [`github.com/jwilder/nginx-proxy`](https://github.com/jwilder/nginx-proxy)

+   WordPress Docker 镜像：[`hub.docker.com/_/wordpress/`](https://hub.docker.com/_/wordpress/)

+   Docker Compose 文档：[`docs.docker.com/compose/`](https://docs.docker.com/compose/)

# Lint 检查 Dockerfile

就像其他任何语言一样，Dockerfile 也可以并且应该进行 lint 检查，以确保最佳实践和代码质量。Docker 也不例外，好的实践总是在不断发展，更新，并且在不同社区之间可能略有不同。在本节中，我们将从之前找到的一个基本 Dockerfile 开始，最后得到一个完全经过双重检查的 lint 检查文件。

## 准备工作

为了按此方法进行操作，您需要以下内容：

+   一个正常工作的 Docker 安装

+   一个 AWS 账户

## 如何操作……

有许多不同的 linters 可以用于 lint 检查 Dockerfile：Hadolint（[`hadolint.lukasmartinelli.ch/`](http://hadolint.lukasmartinelli.ch/)）可能是使用最广泛的 linters，而 Project Atomic 的 `dockerfile_lint` 项目则可能是最完整的一个（[`github.com/projectatomic/dockerfile_lint`](https://github.com/projectatomic/dockerfile_lint)）。

这是本书之前提到的工作中的 Dockerfile：

```
FROM debian:stable-slim
RUN apt-get update -y \
    && apt-get install -y apache2 \
    && rm -rf /var/lib/apt
ENTRYPOINT ["/usr/sbin/apache2ctl"]
CMD ["-D", "FOREGROUND"]
```

### Hadolint

让我们开始使用 Hadolint，因为它易于安装（提供预构建的二进制文件和 Docker 镜像）和使用。所有规则都可以在 Hadolint 的 Wiki 中找到解释（[`github.com/lukasmartinelli/hadolint/wiki`](https://github.com/lukasmartinelli/hadolint/wiki)），而且使用方法非常简单：

```
$ hadolint Dockerfile

```

另外，可以使用 Docker 容器化版本；它可能在 CI 脚本中非常有用。请注意镜像的大小；在撰写时，该镜像为 1.7 GB，而 Hadolint 二进制文件小于 20 MB：

```
$ docker run --rm -i lukasmartinelli/hadolint < Dockerfile

```

从本章开始对 Dockerfile 进行 lint 检查，我们会注意到不同的警告。也许有些是误报，或者有些规则还没有更新到最新的废弃通知，例如以下内容：

```
$ hadolint Dockerfile
Dockerfile DL4000 Specify a maintainer of the Dockerfile

```

事实上，这个 Dockerfile 遵循了 Docker 1.13 的推荐做法，其中包括不再包含 `maintainer` 指令。然而，Hadolint 还没有更新以适应这一废弃的变化，因此执行以下命令以忽略一个或多个 ID，仍然可以保持正常：

```
$ hadolint --ignore DL4000 --ignore <another_ID> Dockerfile

```

### Dockerfile_lint

由 Project Atomic 团队（[`www.projectatomic.io/`](http://www.projectatomic.io/)）主导的这个项目也提出了不同的检查和关于如何编写 Dockerfile 的强烈意见。这些建议通常都是很好的建议。

执行此命令以从官方 Docker 镜像启动 `dockerfile_lint`：

```
$ docker run -it --rm -v $PWD:/root/ projectatomic/dockerfile-lint dockerfile_lint

```

会出现一些建议（错误、警告和信息），每个建议都带有相关的参考 URL 供您参考。

当有疑问时，通常跟随建议并相应地修复代码是一个不错的选择。

在这个双重 lint 检查过程结束时，我们的 Dockerfile 改动很大，如下所示：

```
FROM debian:stable-slim
LABEL name="apache"
LABEL maintainer="John Doe <john@doe.com>"
LABEL version=1.0
RUN apt-get update -y \
    && apt-get install -y --no-install-recommends apache2=2.4.10-10+deb8u7 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
EXPOSE 80
ENTRYPOINT ["/usr/sbin/apache2ctl"]
CMD ["-D", "FOREGROUND"]
```

我们添加了标签以识别镜像、版本和维护者，并且我们固定了 apache2 包的正确版本。这样就不会因为未经测试的更新出现意外（更新包需要重新构建镜像），我们更精确地清理了 apt 缓存，并明确地从容器中暴露了一个端口。

总体来说，linters 提出的这些更改帮助我们构建了一个更好、更强大的容器。它们在 CI 中的作用至关重要；在你的 Jenkins、Circle 或 Travis CI 作业中加入 linters！

# 部署一个带 S3 存储的私有 Docker 注册表

Docker 注册表是一个中央镜像分发服务。当我们*拉取*或*推送*镜像时，它来自 Docker 注册表。它可以是商业托管的（例如 CoreOS Quay [`quay.io/`](https://quay.io/) 和 Docker 自己的 [`hub.docker.com/`](https://hub.docker.com/)），也可以是自托管的（出于隐私、速度、带宽问题或公司政策的考虑）。Docker 公司让我们轻松部署它；它有详细的文档和打包好的版本。在众多可部署的功能中，我们首先简单地部署一个准备好负载均衡的单一注册表，然后将其后端存储切换为 AWS S3，这样磁盘空间再也不是问题。

## 准备就绪

要按照这个配方进行操作，你将需要以下内容：

+   一个正常工作的 Docker 安装

+   一个具有完全 S3 访问权限的 AWS 账户

## 如何操作…

我们将使用 Docker Compose 来完成这个配方。我们的目标是托管自己的私有 Docker 注册表，最初使用本地存储，然后使用 S3 桶来提供无限的空间。该注册表将通过 `http://localhost:5000` 访问，但你也可以使用任何其他可解析的名称或具有本地可用名称的专用服务器。

首先，我们需要 Docker 注册表 v2 镜像：`registry:2`。根据文档，我们知道注册表服务器会暴露端口 `5000`，所以我们需要将其转发到主机以便本地使用。如果我们在负载均衡器后面运行多个注册表，分享一个通用密钥是安全的，我们将其设置为 `s3cr3t`。

这是我们初始的 `docker-compose.yml` 文件：

```
version: '2'

services:
  registry:
    image: registry:2
    ports:
      - 5000:5000
    environment:
      REGISTRY_HTTP_SECRET: s3cr3t
```

通过这个简单的设置，我们已经能够运行我们自己的本地 Docker 注册服务器：

```
$ docker-compose up

```

要将镜像上传到我们的私有注册表，过程就是简单地用本地注册表 URL 标记镜像并推送。执行以下命令将 `ubuntu:16.04` 镜像标记为 `localhost:5000/ubuntu`：

```
$ docker tag ubuntu:16.04 localhost:5000/ubuntu

```

然后，要将镜像推送到本地注册表，执行以下操作：

```
$ docker push localhost:5000/ubuntu

```

这个 Docker 镜像现在已存储在本地，可以在不访问公共网络、Docker Hub 或类似服务的情况下重复使用。

### 使用 S3 后端

高使用率的本地 Docker 注册表面临的一个问题是磁盘空间管理——它是有限的。好消息是，Docker 注册表可以轻松处理 S3 后端（如果我们有内部的 OpenStack，也可以使用 Swift）。需要说明的是，Google Cloud 和 Azure 存储也得到支持。要启用 S3 后端，只需在 `docker-compose.yml` 文件中设置少数几个变量：要联系的 AWS 区域、密钥和桶名称。

```
      REGISTRY_STORAGE: s3
      REGISTRY_STORAGE_S3_REGION: us-east-1
      REGISTRY_STORAGE_S3_BUCKET: registry-iacbook
      REGISTRY_STORAGE_S3_ACCESSKEY: AKIAXXXXXXXXX
      REGISTRY_STORAGE_S3_SECRETKEY: 1234abcde#
```

如果你尝试过之前的示例，请先销毁（`docker-compose down`）它，然后启动这个更新后的示例：

```
$ docker-compose up

```

现在重新在本地标记一个镜像：

```
$ docker tag ubuntu:16.04 localhost:5000/ubuntu

```

然后，将镜像推送到本地注册表：

```
$ docker push localhost:5000/ubuntu

```

根据你的上传速度，注册表同步我们推送的层到 AWS S3 后端所需的时间长短会有所不同。

现在我们拥有了自己的本地注册表，具备了无限存储空间！

## 另见

+   Docker 注册表文档：[`docs.docker.com/registry/configuration/`](https://docs.docker.com/registry/configuration/)

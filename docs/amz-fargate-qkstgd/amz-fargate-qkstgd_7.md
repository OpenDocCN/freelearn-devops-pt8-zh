# 使用亚马逊 ECS CLI

亚马逊 ECS **命令行界面**（**CLI**）是一个命令行工具，用于创建、更新和监控 ECS 集群和任务。亚马逊 ECS CLI 支持 Fargate 启动类型。要使用 ECS CLI 创建容器应用程序，需要 Docker Compose（v1 或 v2）。在本章中，我们将使用 ECS CLI 在 Windows PowerShell 中启动 Fargate 启动类型的 ECS 集群。

在本章中，我们将学习以下内容：

+   设置环境

+   安装 ECS CLI

+   安装 AWS 工具 for Windows PowerShell

+   配置 ECS CLI

+   设置 Fargate 的前置条件

+   注册任务执行策略

+   创建 ECS 集群

+   创建安全组

+   授予安全组入站权限

+   创建 Compose 文件

+   配置 ECS 特定参数

+   部署 Compose 文件到集群

+   列出集群中正在运行的容器

+   列出容器日志

+   扩展集群中的任务

+   访问 WordPress 服务

+   删除服务和集群

# 设置环境

如果尚未安装 PowerShell，请下载并安装 PowerShell 3.0 或更高版本。Windows 管理框架 3.0 包含 PowerShell 3.0：

1.  以管理员身份登录 PowerShell，右键单击 PowerShell 应用程序，选择“以管理员身份运行”

1.  使用 `get-host` 命令验证版本是否为 3.0

1.  使用以下命令启用脚本执行，它允许所有用户创建的脚本运行，且所有从互联网上下载的脚本都需要签名：

```
 Set-ExecutionPolicy RemoteSigned
```

我们还需要获取 AWS 安全凭证，表示为访问密钥，由访问密钥 ID 和秘密访问密钥组成：

1.  要获取安全凭证，请从帐户下拉菜单中选择“我的安全凭证”。

1.  在确认对话框中，单击“继续到安全凭证”。

1.  显示“您的安全凭证”页面；选择“访问密钥”。安全凭证只显示访问密钥 ID，而不显示秘密访问密钥。除非之前已记录访问密钥 ID 和秘密访问密钥，否则请通过“创建新访问密钥”创建一个新的访问密钥。

1.  “创建访问密钥”对话框指示访问密钥已创建。

1.  单击“显示访问密钥”链接。

1.  复制访问密钥 ID 和秘密访问密钥。设置环境变量 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_KEY`。

# 安装 ECS CLI

要在 PowerShell 上安装 ECS CLI，首先以管理员身份运行 PowerShell，方法是右键单击 Windows PowerShell，选择“以管理员身份运行”：

1.  随后，运行以下命令以在指定路径创建一个新的目录类型项：

```
New-Item -ItemType "directory" -Path "C:\Program Files\Amazon\ECSCLI"
```

1.  下载并安装 ecs-cli `ecs-cli-windows-amd64-latest.exe` 到目录 `C:\Program Files\Amazon\ECSCLI\` 下的应用程序 `ecs-cli.exe`。

```
Invoke-WebRequest -OutFile ‘C:\Program Files\Amazon\ECSCLI\ecs-cli.exe’ https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-windows-amd64-latest.exe
```

1.  将 `C:\Program Files\Amazon\ECSCLI` 添加到 `PATH` 环境变量中。输出 `ecs-cli` 版本：

```
ecs-cli --version
```

上述命令在 PowerShell 中运行，如下所示：

![](img/c56eaf03-de53-4f44-8bb4-67a778b0f6c8.png)

# 安装 AWS Tools for Windows PowerShell

接下来，安装 AWS Tools for Windows PowerShell，用于管理 AWS 服务，包括通过 PowerShell 管理 ECS：

1.  从 [`aws.amazon.com/powershell/`](https://aws.amazon.com/powershell/) 下载 AWS Tools for Windows PowerShell 的 MSI 安装文件（`AWSToolsAndSDKForNet_sdk-3.3.231.0_ps-3.3.225.1_tk-1.14.0.1`）。

1.  双击 MSI 安装程序启动 AWS Tools for Windows 安装向导。在欢迎屏幕中点击“下一步”。

1.  在最终用户许可协议中，接受协议并点击“下一步”。

1.  在“自定义设置”中，选择默认安装位置并点击“下一步”。

1.  在“准备安装 AWS Tools for Windows”中，点击“安装”。AWS Tools for Windows 开始安装。

1.  当 AWS Tools for Windows 安装完成后，点击“完成”。

# 配置 ECS CLI

配置 ECS CLI 包括以下两个配置：

+   配置具有 Fargate 启动类型的 ECS 集群

+   配置 ECS CLI 配置文件

接下来，我们将讨论这些配置中的每一个。配置集群的语法如下：

```
ecs-cli configure --cluster cluster_name --default-launch-type launch_type --region region_name --config-name configuration_name
```

`ecs-cli` 配置命令中的不同命令参数在下表中讨论：

| **命令参数** | **描述** | **设置的值** |
| --- | --- | --- |
| `--cluster` | 现有 ECS 集群的名称或要创建的新集群名称 | `hello-world` |
| `--region` | AWS 区域 | `us-east-1` |
| `--default-launch-type` | 启动类型 | `FARGATE` |
| `--config-name` | 配置名称 | `hello-world` |

1.  在 PowerShell 中运行以下命令以配置 ECS 集群：

```
 ecs-cli configure --cluster hello-world --region us-east-1 --default-
      launch-type FARGATE --config-name hello-world
```

1.  创建一个目录，`C:\PowerShell`，用于 PowerShell，并从该目录运行前面的命令。正如命令输出所示，ECS CLI 集群配置 `hello-world` 被保存：

```
 PS C:\PowerShell>
 PS C:\PowerShell> ecs-cli configure --cluster hello-world --region
      us-east-1 --default-launch-type FARGATE --config-name hello-world
      time="2018-01-09T15:26:52-08:00" level=info msg="Saved ECS CLI 
      cluster configuration hello-world."
 PS C:\PowerShell>
```

# 设置 CLI 配置文件

配置 ECS CLI 配置文件的命令语法如下：

```
ecs-cli configure profile --profile-name profile_name --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY
```

`ecs-cli` 配置文件命令中的不同命令参数在下表中讨论：

| **命令参数** | **描述** | **设置的值** |
| --- | --- | --- |
| `--profile` | 现有 ECS 集群的名称或要创建的新集群名称 | `hello-world` |
| `--access-key` | AWS 访问密钥 ID | 不同用户的值不同 |
| `--secret-key` | 密钥访问密钥 | 不同用户的值不同 |

1.  创建 ECS CLI 配置文件 `hello-world`，其中需要用 `AWS 访问密钥 ID` 和 `密钥访问密钥` 替换变量，这些密钥对于不同用户是不同的：

```
 ecs-cli configure profile --access-key $AWS_ACCESS_KEY_ID --
      secret-key $AWS_SECRET_ACCESS_KEY --profile-name hello-world
```

1.  正如命令输出所示，ECS CLI 配置文件被保存：

```
 PS C:\PowerShell> ecs-cli configure profile --access-key Abcdef --
      secret-key D+d+KidQel/myUXJi/hJjtk --profile-name hello-world 
      time="2018-01-09T15:46:15-08:00" level=info msg="Saved ECS CLI 
      profile configuration hello-world."
```

1.  作为创建 `hello-world` 配置文件（或其他配置文件名）的替代，可以使用 `-StoreAs` 默认参数创建一个默认的 ECS 配置文件：

```
 PS C:\PowerShell> Set-AWSCredential -AccessKey AKIAJ4DECNW6BFYN5PNA -
      SecretKey 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 -StoreAs default
```

# 设置 Fargate 的前置条件

除了配置 ECS CLI 配置文件外，我们还需要设置以下前置条件，以便使用 FARGATE 启动类型创建 ECS 集群：

+   创建任务执行角色

+   注册任务执行策略

+   创建任务执行角色

为了让 Fargate 任务能够下载 Docker 镜像并在 CloudWatch 中发送和保存容器日志，需要创建一个任务执行角色。请在 `C:\PowerShell\` 目录中创建一个 IAM 策略文件 `execution-assume-role.json`，并将以下 JSON 内容复制并保存到策略文件中：

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

使用以下命令在 PowerShell 中创建任务执行角色：

```
New-IAMRole -RoleName ecsExecutionRole -AssumeRolePolicyDocument (Get-Content -Raw C:\PowerShell\execution-assume-role.json)
```

如命令输出所示，创建了一个任务执行角色：

```
PS C:\PowerShell> New-IAMRole -RoleName ecsExecutionRole -AssumeRolePolicyDocument (Get-Content -Raw C:\PowerShell\execution-assume-role.json) Path    RoleName         RoleId                 CreateDate   Description ----    --------         ------                 ----------   ----------- /      ecsExecutionRole  AROAIRRZ5LMXLR6QPZQJC  1/9/2018 3:07:16 PM
```

# 注册任务执行策略

接下来，使用以下命令在 PowerShell 中注册任务执行策略：

```
Register-IAMRolePolicy -RoleName ecsExecutionRole -PolicyArn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

前面的命令不会产生任何输出，任务执行策略会被注册。

# 创建 ECS 集群

配置好前提条件后，接下来我们将创建一个 ECS Fargate 集群。在 PowerShell 中运行以下命令以创建集群：

```
esc-cli up
```

如命令输出所示，使用先前设置的集群配置创建了集群，并创建了一个 VPC 和两个子网：

```
PS C:\PowerShell> ecs-cli up
←[36mINFO←[0m[0002] Created cluster ←[36mcluster←[0m=hello-world ←[36mregion←[0m=us-east-1 ←[36mINFO←[0m[0003] Waiting for your cluster resources to be created...
←[36mINFO←[0m[0004] Cloudformation stack status ←[36mstackStatus←[0m=CREATE_IN_PROGRESS
←[36mINFO←[0m[0065] Cloudformation stack status ←36mstackStatus←[0m=CREATE_IN_PROGRESS
 VPC created: vpc-6e021915
 Subnet created: subnet-2c02dd4b
 Subnet created: subnet-f2d50bdc
 Cluster creation succeeded.
 PS C:\PowerShell>
```

在网页浏览器中访问 ECS 控制台，`hello-world` 集群会列出，如下所示：

![# 创建安全组接下来，使用在前一节创建集群时创建的 VPC 来创建一个安全组。如果没有在持久化/外壳默认设置中配置 AWS 区域，则需要在以下命令中指定区域：```$groupid = New-EC2SecurityGroup -VpcId "vpc-6e021915" -Region "us-east-1" -GroupName "hello-worldPSSecurityGroup" -GroupDescription "EC2-VPC from PowerShell"```使用以下 `Get-EC2SecurityGroup` 命令列出 EC2 安全组：```Get-EC2SecurityGroup -Region "us-east-1" -GroupId $groupid```EC2 安全组列出，如下所示：![](img/15a18143-b901-4d47-ad26-8b38ae6bce87.png)

如有需要，可以使用以下命令删除 EC2 安全组：

```
Remove-EC2SecurityGroup -Region "us-east-1" $groupid
```

# 授予安全组 Ingress 权限

接下来，使用 `Grant-EC2SecurityGroupIngress` cmdlet 添加一个安全组规则，允许端口 `80` 上的入站访问，按照列出的顺序运行以下命令：

```
$ip1 = new-object Amazon.EC2.Model.IpPermission
$ip1.IpProtocol = "tcp"
$ip1.FromPort = 80
$ip1.ToPort = 80
$ip1.IpRanges.Add("0.0.0.0/0") Grant-EC2SecurityGroupIngress -GroupId $groupid -Region "us-east-1" -IpPermissions @($ip1)
```

随后，再次运行 `Get-EC2SecurityGroup` cmdlet，如此处的输出所示，Ingress 权限已设置到 EC2 安全组：

![](img/497f57e0-9984-419c-a153-97895e1aef83.png)

我们将仅为 WordPress 创建一个 ECS 任务，但如果同一任务中还需要配置 MySQL 数据库，则还需要为 MySQL 数据库暴露的端口 `3306` 添加一个 Ingress 规则。按以下顺序运行命令，以为端口 `3306` 添加 Ingress：

```
$ip2 = new-object Amazon.EC2.Model.IpPermission
$ip2.IpProtocol = "tcp"
$ip2.FromPort = 3306
$ip2.ToPort = 3306
$ip2.IpRanges.Add("0.0.0.0/0")
Grant-EC2SecurityGroupIngress -GroupId $groupid -Region "us-east-1" -IpPermissions @($ip2)
```

随后，再次运行 `Get-EC2SecurityGroup` 命令。如输出所示，端口 `3306` 的 Ingress 也已添加：

![](img/4369591a-a470-4e9a-9bd0-9376ba10c7ce.png)

# 创建 Compose 文件

ECS CLI 支持 Docker Compose 版本 1 和 2，在这个 WordPress 示例 Compose 文件中我们使用的是版本 2。在 `C:/PowerShell` 目录中创建一个 `docker-compose.yml` 文件，并将以下内容复制到该文件中：

```
version: '2'
services:wordpress:
    image: wordpress
    ports:
      - "80:80"
    logging:
      driver: awslogs
      options:
        awslogs-group: hello-world
        awslogs-region: us-east-1
        awslogs-stream-prefix: wordpress
```

Docker Compose 文件指定了一个基于 `wordpress` Docker 镜像的 `wordpress` 服务，并暴露端口 `80`。CloudWatch 日志使用 `awslogs` 驱动程序进行配置，这是 Fargate 启动类型唯一支持的驱动程序。

# 配置 ECS 特定参数

我们还需要创建一个 **ECS 特定参数** 文件 `ecs-params.yml`，它定义了任务创建所需的资源（CPU 和内存）设置，并设置网络模式为 `awsvpc`，这是 Fargate 启动类型唯一支持的网络模式。运行参数包括子网、安全组的网络配置，并将任务的 `Assign public IP` 选项设置为 `ENABLED`。子网从 `ecs-cli up` 命令的输出中获取。安全组从 `Get-EC2SecurityGroup -Region "us-east-1" -GroupId $groupid` 命令的输出中获取，如前所述：

```
version: 1
task_definition:
  task_execution_role: ecsExecutionRole
  ecs_network_mode: awsvpc
  task_size:
    mem_limit: 0.5GB
    cpu_limit: 256
run_params:
  network_configuration:
    awsvpc_configuration:
      subnets:
        - "subnet-2c02dd4b"
        - "subnet-f2d50bdc"
      security_groups:
        - "sg-8c7dafc7"
      assign_public_ip: ENABLED
```

# 将 Compose 文件部署到集群

`ecs-cli compose service up` 命令将 Docker Compose 文件部署到集群。该命令可以通过本表中讨论的命令选项进行参数化：

| **参数** | **描述** | **默认值** |
| --- | --- | --- |
| `--project-name` | 项目名称 | 当前目录 |
| `--ecs-params` | 使用的 ECS 参数文件 | 当前目录中的 `ecs-params.yml` 文件 |
| `--create-log-groups` | 为容器日志创建 CloudWatch 日志组 |  |
| `--file` | Docker Compose 文件 | 当前目录中的 `docker-compose.yml` 文件 |

运行以下命令，其中使用了 `--project-name` 和 `--create-log-groups` 选项。我们不需要使用其他选项，因为所需的文件都在当前目录中：

```
ecs-cli compose --project-name hello-world service up --create-log-groups
```

Docker Compose 文件已部署到 `hello-world` 集群。命令的输出如下所示：

![](img/b6142489-39f8-4469-b257-0c1d7c2bb37d.png)

`hello-world` 集群列出了一个正在运行的任务，如下所示：

![](img/aaac7d42-ef41-4d3c-bec3-9a5f8e52e863.png)

点击 `hello-world` 集群链接以显示其详细信息。创建的服务会列在“服务”选项卡中。点击服务链接，如下所示，以列出其详细信息：

![](img/ea9cdc91-87ff-4676-82ce-7ea9e107cffc.png)

服务详情包括集群名称、状态、任务定义、启动类型、平台版本、期望数量、待处理数量和运行数量，如下所示：

![](img/3ad86abb-a2f9-429b-9d48-23faa595e7c9.png)

要查找任务的详细信息，选择“任务”选项卡并点击任务链接，如下所示：

![](img/3bb4f32d-30a1-4b46-aac2-eb383fc490a9.png)

任务的详细信息会显示，如下图所示。还列出了任务可访问的公共 IP：

![](img/b41d6f20-e96c-4347-9e62-0bd36b9c7b69.png)

向下滚动以查看容器。`wordpress` 容器会列出，如下图所示：

![](img/fe723d13-e502-4e92-9a03-31f0609c7c65.png)

由于我们配置了日志，容器详情中显示了日志配置，如下所示：

![](img/bad4a640-9e11-4f1f-be41-dea8bcd09db0.png)

点击查看 CloudWatch 日志的链接，以显示 CloudWatch 日志，如下截图所示：

![](img/4b585343-785d-431b-b66d-257ebf0a51ae.png)

# 列出集群中正在运行的容器

要列出集群中正在运行的容器，请运行以下命令：

```
ecs-cli compose --project-name hello-world service ps
```

如下截图所示，列出了一个 Docker 容器。容器的状态、端口和任务定义也被列出：

![](img/ce4c5985-0408-426b-9a13-0f78997e057a.png)

# 列出容器日志

要列出容器日志，请从容器名称中复制任务 ID（任务 ID 是容器名称中 `/` 前的子字符串），然后运行以下命令：

```
ecs-cli logs --task-id 0c23d765-88c5-46cd-a317-9db243590c89 –follow
```

容器日志将显示，如下截图所示：

![](img/a8a7e459-fc38-4ccf-aaf1-c2cbdb472190.png)

# 扩展集群中的任务

要将集群扩展到两个任务，例如，运行以下命令：

```
ecs-cli compose --project-name hello-world service scale 2
```

如输出所示，`desiredCount` 和 `runningCount` 被设置为 `2`：

![](img/1f876b9b-f14c-4466-9394-b4f9842bf202.png)

随后，列出正在运行的任务：

```
ecs-cli compose --project-name hello-world service ps
```

如下所示，列出了两个任务：

![](img/c87aaa2e-d6c8-4468-8315-a5d0874bf336.png)

如下截图所示，`hello-world` 集群中的运行任务也列出了 2 个。点击 `hello-world` 集群链接：

![](img/425c71b8-0787-4812-8a79-33e591f5aec8.png)

选择任务标签，列出了两个任务，如下截图所示：

![](img/f0940f4a-a263-464b-bf74-3d66958b76e6.png)

# 访问 WordPress 服务

要访问 WordPress 服务，请在浏览器中打开其中一个任务的公共 IP。选择语言并点击继续。WordPress 应用的欢迎页面将如截图所示显示。点击“Let’s go!”按钮：

![](img/9946534d-7ee8-4480-b98d-526478eae154.png)

指定数据库连接详情并点击提交，访问 WordPress 仪表盘。下面显示的连接详情可能需要从默认设置进行修改。

# 删除服务和集群

要删除服务和相关任务，请运行以下命令：

```
ecs-cli compose --project-name hello-world service down
```

如输出所示，任务的运行数量和期望数量被设置为 0，`hello-world` 服务被删除。服务必须先缩减至 0，然后才能删除服务：

![](img/f0ae1964-d83d-4ec7-977d-ed11bcd95371.png)

使用以下命令删除集群：

```
ecs-cli down –force
```

在命令提示符下选择 y 确认是否删除集群。`hello-world` 集群及所有相关资源将被删除。如果由于超时或某些资源未被删除，导致集群未能成功删除，请再次运行前述命令：

![](img/c36c429e-3a31-466f-adef-ac604a1973bf.png)

或者，可以通过在 AWS CloudFormation 控制台中选择“操作”|“删除堆栈”来直接删除与集群关联的 CloudFormation 堆栈，如下图所示：

![](img/8e6777d6-0dc1-484f-bae7-6d0a2fab2c97.png)

在删除堆栈确认对话框中，选择尚未删除的堆栈资源，然后点击“是，删除”，如图所示：

![](img/3782e02a-e3e1-4ba9-8a30-c8cb7a2fef1b.png)

在直接删除 CloudFormation 堆栈后，重新运行`ecs-cli down -force`命令，集群应当会被删除，如图所示：

![](img/49983bd6-8709-4165-aad0-a251b9561884.png)

# 概述

本章中，我们讨论了如何使用 ECS CLI 创建 Fargate 启动类型的集群。随后，我们在集群上部署了一个 Docker Compose 文件，运行了一个用于 Docker `wordpress`镜像的任务。我们讨论了日志列出和容器运行，并演示了如何扩展服务。

本章总结了 Amazon Fargate 快速入门指南。Amazon Fargate 是 Amazon **弹性容器服务**（**ECS**）的一种启动类型。在写作时，Amazon **弹性 Kubernetes 服务**（**EKS**）和 AWS Fargate 之间没有原生的 AWS 集成。

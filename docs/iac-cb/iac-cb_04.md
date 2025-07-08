# 第四章：使用 Terraform 自动化完整基础设施  

在本章中，我们将介绍以下配方：  

+   在 Digital Ocean 上使用 Terraform 部署完整的 CoreOS 基础设施  

+   在 Google Compute Engine 上部署三层基础设施  

+   在 OpenStack 上部署 GitLab CE + CI 运行器  

+   使用 Terraform 管理 Heroku 应用和插件  

+   在裸金属上使用 Packet 创建可扩展的 Docker Swarm 集群  

# 介绍  

在本章中，我们将描述使用 Terraform 的完整基础设施，展示当一切都结合在一起时的样子，并以实际项目为例。前几章中大多数 Terraform 示例都使用了亚马逊 Web 服务（AWS），因此为了更加多样化和完整，本章将专注于其他基础设施服务，即 Digital Ocean、Google Cloud、Heroku 和 Packet。在 Digital Ocean 上，我们将构建一个完全可用并且实时监控的 CoreOS 集群，DNS 动态更新。在 Google Cloud 上，我们将构建一个三层基础设施，包含两个 HTTP 节点在负载均衡器后面以及一个独立的 MySQL 管理数据库。使用 OpenStack，我们将部署 GitLab CE 和两个 GitLab CI 运行器，使用不同的存储解决方案。我们将看到如何整合和自动化 Heroku 环境。最后，我们将使用 Packet 在裸金属上构建一个强大且可扩展的 Docker Swarm 集群，能够扩展至数百个容器。  

### 注意  

本书使用的 Terraform 版本是 0.7.4。  

# 在 Digital Ocean 上使用 Terraform 部署完整的 CoreOS 基础设施  

在此配方中，我们将从零开始构建一个完全可用的 CoreOS 集群，部署在 Digital Ocean 的纽约数据中心，使用 Terraform 和 cloud-init。我们还将使用 StatusCake 添加一些延迟监控，因此我们为在 Digital Ocean 上使用 Terraform 打下了一个良好的基础。  

## 准备工作  

要执行此配方，你将需要以下内容：  

+   一个可用的 Terraform 安装  

+   一个 Digital Ocean 账户  

+   一个 StatusCake 账户  

+   一个互联网连接  

## 如何执行此操作…  

首先，我们从在名为 `providers.tf` 的文件中创建 `digitalocean` 提供者（它只需要一个 API 密钥）开始：

```
provider "digitalocean" {
  token = "${var.do_token}"
}
```

在名为 `variables.tf` 的文件中声明 `do_token` 变量：  

```
variable "do_token" {
  description = "Digital Ocean Token"
}
```

同时，别忘了在私有的 `terraform.tfvars` 文件中设置它：  

```
do_token = "a1b2c3d4e5f6"
```

### 处理 SSH 密钥  

我们知道，我们将需要一个 SSH 密钥来登录集群成员。对于 Digital Ocean，该资源名为 `digitalocean_ssh_key`。我建议我们将 SSH 密钥文件命名为 `iac_admin_sshkey`，并将其放置在 `keys` 目录中，但由于你可能喜欢其他名称，我们也可以为此使用一个变量。让我们在 `keys.tf` 文件中编写此内容：  

```
resource "digitalocean_ssh_key" "default" {
  name       = "Digital Ocean SSH Key"
  public_key = "${file("${var.ssh_key_file}.pub")}"
}
```

在 `variables.tf` 中创建相关变量，并使用我们建议的默认值：  

```
variable "ssh_key_file" {
  default     = "keys/iac_admin_sshkey"
  description = "Default SSH Key file"
}
```

如果你愿意，现在是有效覆盖 `terraform.tfvars` 文件中的值的时候了：  

```
ssh_key_file = "./keys/my_own_key"
```

### 创建 CoreOS 集群成员  

这是我们基础设施的核心：三个节点运行在纽约市数据中心 NYC1，启用了私有网络，未激活备份（如果需要，可以设置为 `true`！），我们之前创建的 SSH 密钥，以及一个用于初始化配置的 cloud-init 文件。在 Digital Ocean 中，虚拟机被称为 *droplet*，所以启动 droplet 的资源是 `digitalocean_droplet`。所有变量名都与我们刚才列举的内容相关：

```
resource "digitalocean_droplet" "coreos" {
  image              = "${var.coreos_channel}"
  count              = "${var.cluster_nodes}"
  name               = "coreos-${count.index+1}"
  region             = "${var.do_region}"
  size               = "${var.do_droplet_size}"
  ssh_keys           = ["${digitalocean_ssh_key.default.id}"]
  private_networking = true
  backups            = false
  user_data          = "${file("cloud-config.yml")}"
}
```

在 `variables.tf` 文件中声明所有变量，并设置一些良好的默认值（最小 512 MB droplet，三节点集群），以及我们想要覆盖的一些默认值（AMS3 数据中心或稳定版 CoreOS 渠道）：

```
variable "do_region" {
  default     = "ams3"
  description = "Digital Ocean Region"
}

variable "do_droplet_size" {
  default     = "512mb"
  description = "Droplet Size"
}

variable "coreos_channel" {
  default     = "coreos-stable"
  description = "CoreOS Channel"
}

variable "cluster_nodes" {
  default     = "3"
  description = "Number of nodes in the cluster"
}
```

这是我们在 `terraform.tfvars` 中的覆盖值（但你可以随意使用自己的值，比如使用不同的数据中心或 CoreOS 版本）：

```
do_region = "nyc1"
coreos_channel = "coreos-beta"
```

### 添加有用的输出

如果能自动生成一些关于如何连接到我们的 CoreOS 集群的文档行，那就太棒了。由于我们可以通过 Terraform 输出做到这一点，让我们从 `outputs.tf` 中这个例子开始。它构造了一个带有动态信息的 SSH 命令行，我们可以轻松使用（它只是遍历每个可用的 `digitalocean_droplet.coreos.*`）：

```
output "CoreOS Cluster Members" {
 value = "${formatlist("ssh core@%v -i ${var.ssh_key_file}", digitalocean_droplet.coreos.*.ipv4_address)}"
}

```

输出将如下所示：

```
CoreOS Cluster Members = [
 ssh core@192.241.128.44 -i ./keys/iac_admin_sshkey,
 ssh core@192.241.130.33 -i ./keys/iac_admin_sshkey,
 ssh core@198.199.120.212 -i ./keys/iac_admin_sshkey
]

```

### 动态 DNS 集成

Digital Ocean 的一个吸引人的特点是简单的 DNS 集成。例如，如果我们的域名是 `infrastructure-as-code.org`，而我们启动了一个 *blog* droplet，我们将自动将其注册为公有 DNS 名称 `blog.infrastructure-as-code.org`。非常简单和动态！为了让 Digital Ocean 管理我们的域名，我们需要访问我们的域名注册商（即购买域名的地方），并配置我们的域名由 Digital Ocean 管理，使用他们的自有 DNS 服务器，具体如下：

+   `ns1.digitalocean.com`

+   `ns2.digitalocean.com`

+   `ns3.digitalocean.com`

完成此先决条件后，我们在 `dns.tf` 文件中声明我们的域名，使用 `digitalocean_domain` 资源，自动使用 `cluster_domainname` 变量作为域名，并进行初始 IP 地址匹配，这个 IP 地址我们可以设置为已知值或任意一个 droplet：

```
resource "digitalocean_domain" "cluster_domainname" {
  name       = "${var.cluster_domainname}"
  ip_address = "${digitalocean_droplet.coreos.0.ipv4_address}"
}
```

在 `variables.tf` 中添加新变量：

```
variable "cluster_domainname" {
  default     = "infrastructure-as-code.org"
  description = "Domain to use"
}
```

别忘了在 `terraform.tfvars` 中根据需要覆盖它。

下一步是自动将每个 droplet 注册到 DNS。通过遍历每个 droplet，提取它们的 `name` 和 `ipv4_address` 属性，我们将把这个 `digitalocean_record` 资源加入到配置中：

```
resource "digitalocean_record" "ipv4" {
  count  = "${var.cluster_nodes}"
  domain = "${digitalocean_domain.cluster_domainname.name}"
  type   = "A"
  name   = "${element(digitalocean_droplet.coreos.*.name, count.index)}"
  value  = "${element(digitalocean_droplet.coreos.*.ipv4_address, count.index)}"
}
```

这将自动将每个 droplet 注册为名称为 core-[1,2,3].mydomain.com，以便更方便地访问和引用。

如果你愿意，你可以直接在输出文件（`outputs.tf`）中访问该资源的 `fqdn` 属性：

```
output "CoreOS Cluster Members DNS" {
  value = "${formatlist("ssh core@%v -i ${var.ssh_key_file}", digitalocean_record.ipv4.*.fqdn)}"
}
```

### 集成 cloud-init

我们需要为 CoreOS 集群构建一个完全可用的 `cloud-config.yml` 文件。有关 `cloud-config.yml` 文件的更多信息，特别是如何配置 CoreOS，参考本书的 第五章，*使用 Cloud-Init 配置最后一公里*。

我们需要的完全可用的 CoreOS 集群包括以下内容：

+   在本地网络接口（`$private_ipv4`）上的工作 etcd 集群

+   在本地网络接口（`$private_ipv4`）上的工作 fleet 集群

    Fleet 是一个分布式初始化系统。你可以把它看作是整个集群的 systemd。

要配置 etcd，首先需要获取一个新的令牌。这个令牌是唯一的，可以通过不同的渠道分发。可以通过 [`coreos.com/os/docs/latest/cluster-discovery.html`](https://coreos.com/os/docs/latest/cluster-discovery.html) 等等服务轻松获取。然后我们将启动两个单元——etcd 和 fleet。

```
$ curl -w "\n" 'https://discovery.etcd.io/new?size=3'
https://discovery.etcd.io/b04ddb7ff454503a66ead486b448afb7

```

仔细注意这个 URL，并将其复制粘贴到以下 `cloud-config.yml` 文件中：

```
#cloud-config
# https://coreos.com/validate/
coreos:
  etcd2:
    discovery: "https://discovery.etcd.io/b04ddb7ff454503a66ead486b448afb7"
    advertise-client-urls: "http://$private_ipv4:2379"
    initial-advertise-peer-urls: "http://$private_ipv4:2380"
    listen-client-urls: http://0.0.0.0:2379
    listen-peer-urls: http://$private_ipv4:2380
  units:
    - name: etcd2.service
      command: start
    - name: fleet.service
      command: start
  fleet:
    public-ip: "$public_ipv4"
    metadata: "region=ams,provider=digitalocean"
```

这就足够在 CoreOS 上启动一个 etcd + fleet 集群。第五章，*配置最后一公里*

*与 Cloud-Init 配置最后一公里*，深入了解 cloud-init。

### 集成动态 StatusCake 监控

我们可以复用之前章节中的知识，通过使用免费的 StatusCake 账户 ([`statuscake.com`](https://statuscake.com))，轻松地为 CoreOS 集群的主机集成完整的延迟监控。

首先在 `providers.tf` 中配置提供商：

```
provider "statuscake" {
  username = "${var.statuscake_username}"
  apikey   = "${var.statuscake_apikey}"
}
```

在 `variables.tf` 中声明所需的变量：

```
variable "statuscake_username" {
  default     = "changeme"
  description = "StatusCake Account Username"
}

variable "statuscake_apikey" {
  default     = "hackme"
  description = "StatusCake Account API Key"
}
```

此外，在 `terraform.tfvars` 中用你自己的值进行覆盖。

现在，我们可以使用 `statuscake_test` 资源通过遍历每个 `digitalocean_droplet.coreos.*` 资源值，在每个 Droplet 上启用即时延迟（ping）监控：

```
resource "statuscake_test" "coreos_cluster" {
  count        = "${var.cluster_nodes}"
  website_name = "${element(digitalocean_droplet.coreos.*.name, count.index)}.${var.cluster_domainname}"
  website_url  = "${element(digitalocean_droplet.coreos.*.ipv4_address, count.index)}"
  test_type    = "PING"
  check_rate   = 300
  paused       = false
}
```

是时候运行 `terraform apply` 了：

```
$ terraform apply
[...]

CoreOS Cluster Members = [
 ssh core@159.203.189.142 -i ./keys/iac_admin_sshkey,
 ssh core@159.203.189.146 -i ./keys/iac_admin_sshkey,
 ssh core@159.203.189.131 -i ./keys/iac_admin_sshkey
]
CoreOS Cluster Members DNS = [
 ssh core@coreos-1.mydomain.com -i ./keys/iac_admin_sshkey,
 ssh core@coreos-2.mydomain.com -i ./keys/iac_admin_sshkey,
 ssh core@coreos-3.mydomain.com -i ./keys/iac_admin_sshkey
]

```

确认我们可以通过命令行连接到一个成员，查看输出：

```
$ ssh core@159.203.189.142 -i ./keys/iac_admin_sshkey

```

验证 etcd 集群的健康状况：

```
$ core@coreos-1 ~ $ etcdctl cluster-health
member 668f889d5f96b578 is healthy: got healthy result from http://10.136.24.178:2379
member c8e8906e0f3f63be is healthy: got healthy result from http://10.136.24.176:2379
member f3b53735aca3062e is healthy: got healthy result from http://10.136.24.177:2379
cluster is healthy

```

检查所有 fleet 成员是否正常：

```
core@coreos-1 ~ $ fleetctl list-machines
MACHINE         IP              METADATA
24762c02...     159.203.189.146 provider=digitalocean,region=ams
3b4b0792...     159.203.189.142 provider=digitalocean,region=ams
59e15b88...     159.203.189.131 provider=digitalocean,region=ams

```

享受吧，在不到一分钟的时间里，你就可以通过完全自动化的 Terraform 代码，使用带有基本监控的 CoreOS 集群！

# 在 Google Compute Engine 上配置三层基础设施

我们将在 Google Compute Engine 上配置一个三层负载均衡的 web 基础设施，使用两个 CentOS 7.2 服务器作为 web 服务器，以及一个主 Google MySQL 实例。MySQL 实例只允许来自这两个 web 服务器的连接（并且需要有效的凭证），所有三个实例（SQL 和 HTTP）都将通过一个单一的 *公司* 网络（我们公司的网络）进行访问。拓扑结构如下：

![在 Google Compute Engine 上配置三层基础设施](img/B05671_04_01.jpg)

## 准备工作

要按步骤执行这个配方，你需要以下内容：

+   一个可用的 Terraform 安装

+   一个 Google Compute Engine 项目账号

+   一条互联网连接

## 如何执行...

我们需要首先从控制台获取我们的凭据。

### 为 Google 项目生成 API 凭据。

转到您的 Google Cloud 项目，在 *API 管理器* 中选择 **凭据** | **创建凭据** | **服务帐号密钥**。现在从下拉列表中选择 **Compute Engine 默认服务帐号**，格式选择 JSON。将此文件保存为 `account.json`，放在基础设施存储库的根目录。

在 `variables.tf` 文件中创建变量来定义我们的凭据文件，存储我们正在运行的区域和 Google Compute 项目名称：

```
variable "credentials_file" {
  default     = "account.json"
  description = "API credentials JSON file"
}
variable "region" {
  default     = "europe-west"
  description = "Region name"
}
variable "project_name" {
  default     = "default-project"
  description = "Project ID to use"
}
```

不要忘记在 `terraform.tfvars` 中覆盖这些值（如果需要）：

```
project_name = "iac-book-infra"
region = "us-east1"
```

现在，在 `providers.tf` 文件中，添加 `google` 供应商：

```
provider "google" {
  credentials = "${file("${var.credentials_file}")}"
  project     = "${var.project_name}"
  region      = "${var.region}"
}
```

我们的 `google` 供应商现在已配置完成！

### 创建 Google Compute HTTP 实例。

这是我们为这些 HTTP 主机的需求清单：

+   我们需要两个实例。

+   它们的类型是 `n1-standard-1`（3.75 GB RAM，1 vCPU）。

+   它们的区域和区域是：us-east1-d。

+   它们正在运行 CentOS 7.2（官方镜像为：centos-cloud/centos 7）。

+   默认的 SSH 用户名是 `centos`。

+   我们已知的 SSH 密钥是 (`keys/admin_key`)。

+   我们需要一个完全更新的系统，并安装并运行 Docker。

让我们在 `variables.tf` 文件中为所有这些要求定义通用变量：

```
variable "machine_type" {
  default     = "f1-micro"
  description = "Machine type"
}

variable "zone" {
  default     = "c"
  description = "Region Zone"
}

variable "disk_image" {
  default     = "centos-cloud/centos-7"
  description = "Disk image"
}

variable "ssh_key" {
  default     = "keys/admin_key"
  description = "SSH key"
}

variable "ssh_username" {
  default     = "root"
  description = "The SSH username to use"
}

variable "www_servers" {
  default = "2"
  description = "Amount of www servers"
}
```

现在让我们在 `terraform.tfvars` 中覆盖刚设置的通用值：

```
machine_type = "n1-standard-1"
zone = "d"
ssh_username = "centos"
```

Google Cloud 实例通过 `google_compute_instance` 资源从 Terraform 调用：

让我们在此资源中添加我们已知的内容：

```
resource "google_compute_instance" "www" {
  count        = "${var.www_servers}"
  name         = "www-${count.index+1}"
  machine_type = "${var.machine_type}"
  zone         = "${var.region}-${var.zone}"

  disk {
    image = "${var.disk_image}"
  }

  metadata {
    ssh-keys = "${var.ssh_username}:${file("${var.ssh_key}.pub")}"
  }
}
```

这可能已经足够了，但我们希望走得更远。

例如，稍后我们将添加一个防火墙，其规则将应用于由其标签定义的目标。现在让我们立即添加一个标签，以便稍后使用它：

```
tags         = ["www"]
```

我们必须配置网络。在我们的情况下需要一个公共 IPv4 地址，因为我们需要从外部通过 SSH 访问服务器。我们也可以选择不公开暴露服务器而使用堡垒主机。要在默认网络中创建一个网络接口，映射到公共 IPv4 后面，请将以下内容添加到 `google_compute_instance` 资源中：

```
  network_interface {
    network = "default"

    access_config {
      nat_ip = ""
    }
  }
```

让我们最后通过 `remote-exec` 配置项自动连接到每个实例并完全更新它，然后安装、启用和启动 Docker。我们将正确配置 `remote-exec` 与正确的 SSH 用户名和私钥：

```
provisioner "remote-exec" {
    connection {
      user        = "${var.ssh_username}"
      private_key = "${file("${var.ssh_key}")}"
    }

    inline = [
      "sudo yum update -y",
      "sudo yum install -y docker",
      "sudo systemctl enable docker",
      "sudo systemctl start docker",
    ]
  }
```

我们终于完成了，我们的两个实例已自动配置完成！

### 创建一个 Google Compute 防火墙规则。

我们的目标很简单：我们希望允许任何人（0.0.0.0/0）通过 HTTP（TCP 端口 `80`）访问任何标记为 `www` 的默认网络中的实例。为此，让我们使用 `google_compute_firewall` 资源：

```
resource "google_compute_firewall" "fw" {
  name    = "www-firewall"
  network = "default"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["www"]
}
```

### 负载均衡 Google Compute 实例。

为了在我们的两个实例之间负载均衡请求，我们需要创建一个*主机池*，其成员资格将通过简单的健康检查来管理：每秒对 / 进行一次 HTTP `GET` 请求，立即超时（`1` 秒），并在发生 `3` 次错误后移除。我们可以在名为 `pool.tf` 的文件中使用 `google_compute_http_health_check` 资源来实现：

```
resource "google_compute_http_health_check" "www" {
  name                = "http"
  request_path        = "/"
  check_interval_sec  = 1
  healthy_threshold   = 1
  unhealthy_threshold = 3
  timeout_sec         = 1
}
```

随意将这些值转换为变量，以便在你的环境中进行更好的调优！

现在，让我们定义池，池是由健康检查的结果和实例的包含情况定义的。这可以通过 `google_compute_target_pool` 资源来实现：

```
resource "google_compute_target_pool" "www" {
  name          = "www-pool"
  instances     = ["${google_compute_instance.www.*.self_link}"]
  health_checks = ["${google_compute_http_health_check.www.name}"]
}
```

### 注意

`self_link` 属性返回资源的 URI。

现在我们已经有了带健康检查的主机池，接下来创建负载均衡器本身。我们可以使用 `google_compute_forwarding_rule` 资源来实现，只需指向我们之前创建的主机池即可。将以下内容添加到 `loadbalancer.tf` 文件中：

```
resource "google_compute_forwarding_rule" "http" {
  name       = "http-lb"
  target     = "${google_compute_target_pool.www.self_link}"
  port_range = "80"
}
```

### 创建一个 Google MySQL 数据库实例

我们的典型目标应用需要一个数据库来存储和访问数据。这里我们不深入讨论数据库复制，但它也可以通过 Terraform 在 Google Cloud 上轻松实现。

### 注意

仔细检查你是否在 Google Cloud 控制台中激活了 SQL API：[`console.cloud.google.com/apis/library`](https://console.cloud.google.com/apis/library)。默认情况下，它是没有激活的。

这是我们关于 MySQL 数据库的检查清单：

+   它运行在 us-east1 区域

+   它运行 MySQL 5.6

+   它的类型是*D2*（1 GB 内存）

+   我们自己的网络和两个 HTTP 服务器都可以访问它

+   我们希望创建一个名为 `app_db` 的数据库

+   我们希望一个具有密码的用户能够从 HTTP 服务器连接

让我们把所有这些变量放在 `variables.tf` 文件中：

```
variable "db_type" {
  default     = "D0"
  description = "Google SQL DB type"
}

variable "db_authorized_network" {
  default     = "0.0.0.0/0"
  description = "A corporate network authorized to access the DB"
}

variable "db_username" {
  default     = "dbadmin"
  description = "A MySQL username"
}

variable "db_password" {
  default     = "changeme"
  description = "A MySQL password"
}

variable "db_name" {
  default     = "db_name"
  description = "MySQL database name"
}
```

别忘了在 `terraform.tfvars` 中覆盖每个通用值：

```
db_authorized_network = "163.172.161.158/32"
db_username = "sqladmin"
db_password = "pwd1970"
db_name = "app_db"
db_type = "D2"
```

现在我们可以使用 `google_sql_database_instance` 资源在 `db.tf` 文件中构建数据库：

```
resource "google_sql_database_instance" "master" {
  name             = "mysql-mastr-1"
  region           = "${var.region}"
  database_version = "MYSQL_5_6"

  settings = {
    tier              = "${var.db_type}"
    activation_policy = "ALWAYS"         // vs "ON_DEMAND"
    pricing_plan      = "PER_USE"        // vs "PACKAGE"

    ip_configuration {
      ipv4_enabled = true

      authorized_networks {
        name  = "authorized_network"
        value = "${var.db_authorized_network}"
      }

      authorized_networks {
        name  = "${google_compute_instance.www.0.name}"
        value = "${google_compute_instance.www.0.network_interface.0.access_config.0.assigned_nat_ip}"
      }

      authorized_networks {
        name  = "${google_compute_instance.www.1.name}"
        value = "${google_compute_instance.www.1.network_interface.0.access_config.0.assigned_nat_ip}"
      }
    }
  }
}
```

### 注意

`pricing_plan` `"PACKAGE"` 更适合长期使用的数据库。同时，`authorized_network` 块当前不支持 `count` 值，因此我们无法动态迭代每个 HTTP 主机。目前，我们必须复制该块，但在未来的 Terraform 版本中，这种情况可能会改变。

现在，让我们使用 `google_sql_database` 资源创建一个数据库：

```
resource "google_sql_database" "db" {
  name     = "${var.db_name}"
  instance = "${google_sql_database_instance.master.name}"
}
```

最后，通过创建具有主机限制的 SQL 用户来完成。与 `authorized_network` 块类似，`google_sql_user` 资源当前还不支持 `count` 值，因此我们暂时需要为每个 HTTP 服务器复制代码：

```
resource "google_sql_user" "user_www_1" {
  name     = "${var.db_username}"
  password = "${var.db_password}"
  instance = "${google_sql_database_instance.master.name}"
  host     = "${google_compute_instance.www.0.network_interface.0.access_config.0.assigned_nat_ip}"
}

resource "google_sql_user" "user_www_2" {
  name     = "${var.db_username}"
  password = "${var.db_password}"
  instance = "${google_sql_database_instance.master.name}"
  host     = "${google_compute_instance.www.1.network_interface.0.access_config.0.assigned_nat_ip}"
}
```

### 添加一些有用的输出

拥有一些有用的信息，如所有实例和服务的 IP、用户名和密码，会非常棒。让我们在 `outputs.tf` 中添加一些输出：

```
output "HTTP Servers" {
  value = "${join(" ", google_compute_instance.www.*.network_interface.0.access_config.0.assigned_nat_ip)}"
}

output "MySQL DB IP" {
  value = "${google_sql_database_instance.master.ip_address.0.ip_address}"
}

output "Load Balancer Public IPv4" {
  value = "${google_compute_forwarding_rule.http.ip_address}"
}

output "DB Credentials" {
  value = "Username=${var.db_username} Password=${var.db_password}"
}
```

到这里了！

```
$ terraform apply
[...]
Outputs:

DB Credentials = Username=sqladmin Password=pwd1970
HTTP Servers = 104.196.180.192 104.196.157.246
Load Balancer Public IPv4 = 104.196.45.46
MySQL DB IP = 173.194.111.120

```

只需在 HTTP 服务器上部署我们的应用程序，任务就完成了！为了测试负载均衡器和 HTTP 实例，您可以简单地在每个服务器上部署 NGINX 容器并查看流量：

```
$ sudo docker run -it --rm -p 80:80 --name web nginx

```

# 在 OpenStack 上部署 GitLab CE 和 CI 运行器

OpenStack 是一个非常流行的开源云计算解决方案。许多提供商都基于它，你也可以在自己的数据中心中构建它。在这个示例中，我们将使用 OVH 提供的公共 OpenStack，位于加拿大蒙特利尔，但你也可以使用任何其他 OpenStack。每个自定义部署在实现上有所不同，但我们将坚持使用非常稳定的功能。

我们将启动一个运行 Ubuntu LTS 16.04 的计算实例来托管 GitLab，使用一个专用的块设备来存储 Docker，并且还会启动两个计算实例来作为 GitLab CI 运行器。安全组将允许所有人访问 HTTP，但仅允许来自公司网络已知 IP 的 SSH 访问。为了存储我们的构建或发布，我们将创建一个 *容器*，在 OpenStack 术语中，这就是对象存储。AWS S3 的等效物是 *存储桶*。

## 准备工作

要执行这个操作步骤，你将需要以下资源：

+   一个有效的 Terraform 安装。

+   一个 OpenStack 账户，使用任何 OpenStack 提供商（公有或私有）。这个配方使用的是 OVH 公共 OpenStack 的账户（[`www.ovh.com/us/`](https://www.ovh.com/us/)）。

+   一个互联网连接。

## 如何操作…

我们将创建：

+   三个计算实例（虚拟机）

+   一个密钥对

+   一个块存储设备

+   一个安全组

+   一个对象存储桶

### 配置 OpenStack 提供商

让我们先配置 OpenStack 提供商。我们需要四个信息：用户名、密码、OpenStack 租户名称和 OpenStack 认证端点 URL。为了使代码更加动态，我们将在 `variables.tf` 文件中为这些信息创建变量：

```
variable "user_name" {
  default     = "changeme"
  description = "OpenStack username"
}

variable "password" {
  default     = "hackme"
  description = "OpenStack password"
}

variable "tenant_name" {
  default     = "123456"
  description = "OpenStack Tenant name"
}

variable "auth_url" {
  default     = "https://openstack.url/v2.0"
  description = "OpenStack Authentication Endpoint"
}
```

别忘了在 `terraform.tfvars` 文件中用你自己的值覆盖默认值！

```
user_name   = "***"
tenant_name = "***"
password    = "***"
auth_url    = "https://auth.cloud.ovh.net/v2.0/"
```

现在我们准备好开始了。

### 在 OpenStack 上创建一个密钥对

为了在实例上进行身份验证，我们需要将密钥对的公钥部分提供给 OpenStack。可以使用 `openstack_compute_keypair_v2` 资源来完成此操作，指定我们希望在哪个区域生成密钥，以及密钥存放的位置。让我们在 `variables.tf` 中添加这两个变量：

```
variable "region" {
  default     = "GRA1"
  description = "OpenStack Region"
}

variable "ssh_key_file" {
  default     = "keys/admin_key"
  description = "Default SSH key"
}
```

接下来，在 `terraform.tfvars` 文件中覆盖它们：

```
region      = "BHS1"
```

现在我们可以在 `keys.tf` 文件中构建我们的资源：

```
resource "openstack_compute_keypair_v2" "ssh" {
  name       = "Admin SSH Public Key"
  region     = "${var.region}"
  public_key = "${file("${var.ssh_key_file}.pub")}"
}
```

### 在 OpenStack 上创建安全组

我们知道我们的要求是允许来自任何地方的 HTTP（TCP/80）访问，但仅允许来自一个公司网络的 SSH（TCP/22）访问。现在就将其添加到 `variables.tf` 中，以便我们可以使用：

```
variable "allowed_network" {
  default = "1.2.3.4/32"
  description = "The Whitelisted Corporate Network"
}
```

别忘了在 `terraform.tfvars` 中覆盖为你自己的网络配置。

让我们创建一个安全组，允许我们区域中的所有人访问 HTTP，使用 `openstack_compute_secgroup_v2` 资源，并将其放在 `security.tf` 文件中：

```
 resource "openstack_compute_secgroup_v2" "http-sg" {
  name        = "http-sg"
  description = "HTTP Security Group"
  region      = "${var.region}"

  rule {
    from_port   = 80
    to_port     = 80
    ip_protocol = "tcp"
    cidr        = "0.0.0.0/0"
  }
}
```

按照相同的模式，再创建一个安全组，仅允许来自我们公司网络的 SSH 访问：

```
resource "openstack_compute_secgroup_v2" "base-sg" {
  name        = "base-sg"
  description = "Base Security Group"
  region      = "${var.region}"

  rule {
    from_port   = 22
    to_port     = 22
    ip_protocol = "tcp"
    cidr        = "${var.allowed_network}"
  }
}
```

### 在 OpenStack 上创建块存储卷

根据我们的需求，我们希望为 GitLab 实例提供一个专用的卷，用于 Docker。我们决定这个卷的大小为 `10` GB。该卷将由计算实例挂载为一个专用设备（可能是 `/dev/vdb`）。整个过程将使用 `openstack_blockstorage_volume_v2` 资源完成：

```
resource "openstack_blockstorage_volume_v2" "docker" {
  region      = "${var.region}"
  name        = "docker-vol"
  description = "Docker volume"
  size        = 10
}
```

在 `outputs.tf` 中添加一个简单的输出，以便我们知道存储卷的描述、名称和大小：

```
output "Block Storage" {
  value = "${openstack_blockstorage_volume_v2.docker.description}: ${openstack_blockstorage_volume_v2.docker.name}, ${openstack_blockstorage_volume_v2.docker.size}GB"
}
```

我们现在已经具备启动计算实例的所有要求。

### 在 OpenStack 上创建计算实例

现在是创建实例的时候了。我们知道它们必须是 Ubuntu 16.04，并且我们决定了一个 flavor 名称：flavor 是机器的类型，每个 OpenStack 安装的类型可能不同。在我们的情况下，它被命名为 `vps-ssd-1`。我们来在 `variables.tf` 文件中定义一些默认值：

```
variable "image_name" {
  default     = "CentOS"
  description = "Default OpenStack image to boot"
}

variable "flavor_name" {
  default     = "some_flavor"
  description = "OpenStack instance flavor"
}
```

同时，在 `terraform.tfvars` 中使用合适的值覆盖它们：

```
image_name  = "Ubuntu 16.04"
flavor_name = "vps-ssd-1"
```

要创建一个计算实例，我们使用名为 `openstack_compute_instance_v2` 的资源。此资源需要我们之前声明的所有参数（名称、镜像、flavor、SSH 密钥和安全组）。让我们在 `instances.tf` 中尝试一下：

```
resource "openstack_compute_instance_v2" "gitlab" {
  name            = "gitlab"
  region          = "${var.region}"
  image_name      = "${var.image_name}"
  flavor_name     = "${var.flavor_name}"
  key_pair        = "${openstack_compute_keypair_v2.ssh.name}"
  security_groups = ["${openstack_compute_secgroup_v2.base-sg.name}", "${openstack_compute_secgroup_v2.http-sg.name}"]
}
```

要附加我们创建的块存储卷，我们需要在资源中添加一个 `volume {}` 块：

```
  volume {
    volume_id = "${openstack_blockstorage_volume_v2.docker.id}"
    device    = "/dev/vdb"
  }
```

现在，稍显有趣但可选的部分是需要的命令，用于格式化卷、将其挂载到正确的位置、全面更新系统、安装 Docker，并运行 GitLab CE 容器。这是通过 `remote-exec` 提供器完成的，并且需要 SSH 用户名。我们将其设置为 `variables.tf`：

```
variable "ssh_username" {
  default     = "ubuntu"
  description = "SSH username"
}
```

现在我们可以输入所有命令，当实例准备好时它们将被执行：

```
  provisioner "remote-exec" {
    connection {
      user        = "${var.ssh_username}"
      private_key = "${file("${var.ssh_key_file}")}"
    }

    inline = [
      "sudo mkfs.ext4 /dev/vdb",
      "sudo mkdir /var/lib/docker",
      "sudo su -c \"echo '/dev/vdb /var/lib/docker ext4 defaults 0 0' >> /etc/fstab\"",
      "sudo mount -a",
      "sudo apt update -y",
      "sudo apt upgrade -y",
      "sudo apt install -y docker.io",
      "sudo systemctl enable docker",
      "sudo systemctl start docker",
      "sudo docker run -d -p 80:80 --name gitlab gitlab/gitlab-ce:latest",
    ]
  }
```

在 `outputs.tf` 文件中添加一个简单的输出，以便我们可以轻松查看 GitLab 实例的公网 IP：

```
output "GitLab Instance" {
  value = "gitlab: http://${openstack_compute_instance_v2.gitlab.access_ip_v4}"
}
```

这些 runner 实例是相同的，但稍微简单一些，因为它们不需要本地存储卷。不过，我们需要在 `variables.tf` 中设置所需的 runner 数量：

```
variable "num_runners" {
  default     = "1"
  description = "Number of GitLab CI runners"
}
```

在 `terraform.tfvars` 中覆盖值，以便有更多的 runners：

```
num_runners = "2"
```

现在我们可以使用 `openstack_compute_instance_v2` 资源来创建我们的 runner 实例：

```
resource "openstack_compute_instance_v2" "runner" {
  count           = "${var.num_runners}"
  name            = "gitlab-runner-${count.index+1}"
  region          = "${var.region}"
  image_name      = "${var.image_name}"
  flavor_name     = "${var.flavor_name}"
  key_pair        = "${openstack_compute_keypair_v2.ssh.name}"
  security_groups = ["${openstack_compute_secgroup_v2.base-sg.name}", "${openstack_compute_secgroup_v2.http-sg.name}"]

  provisioner "remote-exec" {
    connection {
      user        = "${var.ssh_username}"
      private_key = "${file("${var.ssh_key_file}")}"
    }

    inline = [
      "sudo apt update -y",
      "sudo apt upgrade -y",
      "sudo apt install -y docker.io",
      "sudo systemctl enable docker",
      "sudo systemctl start docker",
      "sudo docker run -d --name gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest",
    ]
  }
}
```

这将启动一个 GitLab CI runner，因此可以通过 GitLab 触发构建！(不过还有最后一步配置，这超出了本书的范围，但我们需要通过执行 `docker exec -it gitlab-runner gitlab-runner register` 来将每个 runner 注册到主 GitLab 实例，并回答相应问题)。

在 `outputs.tf` 中添加以下输出，以便我们知道所有 runner 的 IP 地址：

```
output "GitLab Runner Instances" {
  value = "${join(" ", openstack_compute_instance_v2.runner.*.access_ip_v4)}"
}
```

### 在 OpenStack 上创建对象存储容器

这个非常简单：它只需要一个名称和一个区域。由于它是用来存储发布的，我们将其命名为 `releases`，使用 `openstack_objectstorage_container_v1` 资源，在 `objectstorage.tf` 文件中：

```
resource "openstack_objectstorage_container_v1" "releases" {
  region = "${var.region}"
  name   = "releases"
}
```

在 `outputs.tf` 中添加一个简单的输出，以便我们记住 `Object Storage` 容器的名称：

```
output "Object Storage" {
  value = "Container name: ${openstack_objectstorage_container_v1.releases.name}"
}
```

### 应用

最后，执行 `terraform apply`：

```
$ terraform apply
[...]

Outputs:

Block Storage = Docker volume: docker-vol, 10GB
GitLab Instance = gitlab: http://158.69.95.202
GitLab Runner Instances = 158.69.95.200 158.69.95.201
Object Storage = Container name: releases

```

连接到 GitLab 实例，享受运行的过程（在 GitLab token 注册之后）！

# 使用 Terraform 管理 Heroku 应用和附加组件

Heroku 是一个流行的 **平台即服务** (**PaaS**)，在这里你对基础设施没有任何控制权。但即便是这样的平台，Terraform 也能为你自动化和管理许多工作，剩下的交给 Heroku 来处理。我们将创建一个应用程序（一个简单的 GitHub Hubot：[`hubot.github.com/`](http://hubot.github.com/)），但你也可以使用自己的应用程序。在这个应用程序的基础上，我们将自动添加一个 Heroku 插件（redis）并部署所有内容。

## 准备工作

要完成此教程，你需要以下资源：

+   一个工作正常的 Terraform 安装

+   一个 Heroku 账户 ([`www.heroku.com/`](https://www.heroku.com/))

+   一个可选的 Slack 令牌

+   需要互联网连接

## 如何操作…

首先：我们需要定义 Heroku 提供者。它由一个电子邮件地址和 API 密钥组成。让我们在 `variables.tf` 中为此创建通用变量：

```
variable "heroku_email" {
  default     = "user@mail.com"
  description = "Heroku account email"
}

variable "heroku_api_key" {
  default     = "12345"
  description = "Heroku account API key"
}
```

别忘了在 `terraform.tfvars` 中重写这些变量：

```
heroku_email = "me@gmail.com"
heroku_api_key = "52eef461-5e34-47d8-8191-ede7ef6cf9bg"
```

现在我们可以用已有的信息创建 Heroku 提供者：

```
provider "heroku" {
  email   = "${var.heroku_email}"
  api_key = "${var.heroku_api_key}"
}
```

### 使用 Terraform 创建 Heroku 应用程序

我们不通过点击 Heroku 来创建应用程序，而是直接从 Terraform 中完成。我们希望在欧洲运行我们的应用程序，并且希望 Hubot 连接到 Slack，因此我们还需要提供一个 Slack 令牌。让我们从在 `variables.tf` 中创建默认值开始：

```
variable "heroku_region" {
  default = "us"
  description = "Heroku region"
}

variable "slack_token" {
  default = "xoxb-1234-5678-1234-5678"
  description = "Slack Token"
}
```

现在我们可以使用 `heroku_app` 资源，在 `heroku.tf` 中创建我们的第一个 Heroku 应用程序及其变量：

```
resource "heroku_app" "hubot" {
  name   = "iac-book-hubot"
  region = "${var.heroku_region}"

  config_vars {
    HUBOT_SLACK_TOKEN = "${var.slack_token}"
  }
}
```

就这样！看起来很简单。

在 `outputs.tf` 中添加一些输出，以便我们能获取更多关于应用程序的信息，比如 Heroku 应用程序 URL 和环境变量：

```
output "heroku URL" {
  value = "${heroku_app.hubot.web_url}"
}

output "heroku_vars" {
  value = "${heroku_app.hubot.all_config_vars}"
}

output "heroku Git URL" {
  value = "${heroku_app.hubot.git_url}"
}
```

### 使用 Terraform 添加 Heroku 插件

一些插件需要 Redis 来存储数据。我们不通过 Web 应用程序启用插件，而是使用 `heroku_addon` 资源。它需要一个应用程序的引用来链接插件，并且还需要一个计划（`hobby-dev` 是免费的，所以我们使用这个计划）：

```
resource "heroku_addon" "redis" {
  app  = "${heroku_app.hubot.name}"
  plan = "heroku-redis:hobby-dev"
}
```

### 使用 Heroku 和 Terraform

本书不涉及 Heroku 的使用，但让我们应用这段 terraform 代码：

```
$ terraform apply
[...]
Outputs:

heroku Git URL = https://git.heroku.com/iac-book-hubot.git
heroku URL = https://iac-book-hubot.herokuapp.com/
heroku_vars = {
  HUBOT_SLACK_TOKEN = xoxb-1234-5678-91011-00e4dd
}
```

如果你还没有准备好要在 Heroku 上部署的应用程序，那就尝试部署 GitHub 的聊天机器人 *Hubot* 吧。它是一个可以在 Heroku 上直接使用的简单应用程序。快速浏览 Hubot 文档后，让我们安装 Hubot 生成器：

```
$ npm install -g yo generator-hubot

```

创建一个新的 `hubot`：

```
$ mkdir src; cd src
$ yo hubot

```

回答问题，完成后，使用常规的 `heroku` 命令，为我们的 Heroku 应用程序添加 Heroku git 远程连接：

```
$ heroku git:remote --app iac-book-hubot

```

现在你可以使用 `git push heroku` 来看到应用程序被部署，整个过程都在 Terraform 的控制下。

# 在裸金属服务器上通过 Packet 创建可扩展的 Docker Swarm 集群

IaaS 云服务通过广泛使用虚拟机已经变得非常流行。最近的举措针对裸金属服务器提供 API 服务，使得我们可以同时享受两者的优势——通过 API 按需获取服务器，并通过直接访问硬件获得卓越的性能。[`www.packet.net/`](https://www.packet.net/)是一个裸金属 IaaS 提供商（[`www.scaleway.com/`](https://www.scaleway.com/)是另一个），并且 Terraform 对此提供了极好的支持，拥有强大的全球网络。几分钟内，我们就能获得新的硬件并将其连接到网络中。

我们将构建一个完全自动化且可扩展的 Docker Swarm 集群，以便在裸金属上运行高度可扩展和高性能的工作负载：这个设置可以在几分钟内扩展数千个容器。该集群由*类型 0*机器组成（4 核和 8GB RAM），包括 1 个管理节点和 2 个工作节点，总共 12 核和 24GB RAM，但如果需要，我们可以使用更高性能的机器：同样的集群如果使用*类型 2*机器，将拥有 72 核和 768GB RAM（不过价格会相应调整）。

## 准备工作

要逐步完成这个步骤，你将需要以下内容：

+   一个可用的 Terraform 安装环境

+   一个 Packet.net 账户和 API 密钥

+   需要一个互联网连接

## 如何操作……

首先，我们通过 API 密钥（身份验证 token）创建`packet`提供者。在`variables.tf`文件中创建变量：

```
variable "auth_token" {
  default     = "1234"
  description = "API Key Auth Token"
}
```

同时，确保在`terraform.tfvars`中覆盖真实的 token 值：

```
auth_token = "JnN7e6tPMpWNtGcyPGT93AkLuguKw2eN"
```

### 使用 Terraform 创建 Packet 项目

Packet 和一些其他 IaaS 提供商一样，使用*项目*的概念来组织机器。让我们在`projects.tf`文件中创建一个名为`Docker Swarm Bare Metal Infrastructure`的项目，因为这正是我们想做的：

```
resource "packet_project" "swarm" {
  name = "Docker Swarm Bare Metal Infrastructure"
}
```

这样，如果你需要管理多个项目或客户，你可以将它们分配到各自的项目中。

### 使用 Terraform 管理 Packet SSH 密钥

要通过 SSH 连接到机器，我们需要至少上传一个公钥到 Packet 账户。我们在`variables.tf`文件中创建一个变量来存储它：

```
variable "ssh_key" {
  default     = "keys/admin_key"
  description = "Path to SSH key"
}
```

如果你使用了不同的密钥名称，别忘了在`terraform.tfvars`中覆盖其值。

我们将使用`packet_ssh_key`资源在 Packet 账户中创建 SSH 密钥：

```
resource "packet_ssh_key" "admin" {
  name       = "admin_key"
  public_key = "${file("${var.ssh_key}.pub")}"
}
```

### 使用 Terraform 在 Packet 上启动 Docker Swarm 管理节点

我们将为这个 Docker Swarm 集群创建两种类型的服务器：管理节点和工作节点。管理节点控制在工作节点上执行的任务。我们将从引导 Docker Swarm 管理节点服务器开始，使用 Packet 服务（Packet API 提供了更多的选择）：

+   我们希望选择最便宜的服务器（`baremetal_0`）

+   我们希望服务器位于阿姆斯特丹（`ams1`）

+   我们希望服务器运行 Ubuntu 16.04（`ubuntu_16_04_image`）

+   默认 SSH 用户是`root`

+   计费将按`小时`计算，但也可以选择`月度`计费

我们将把一些通用信息放入`variables.tf`，以便之后进行操作：

```
variable "facility" {
  default     = "ewr1"
  description = "Packet facility (us-east=ewr1, us-west=sjc1, eu-west=ams1)"
}

variable "plan" {
  default     = "baremetal_0"
  description = "Packet machine type"
}

variable "operating_system" {
  default     = "coreos_stable"
  description = "Packet operating_system"
}

variable "ssh_username" {
  default     = "root"
  description = "Default host username"
}
```

同样，在`terraform.tfvars`中覆盖它们，以确保与我们的值匹配：

```
facility = "ams1"
operating_system = "ubuntu_16_04_image"
```

要使用 Packet 创建服务器，我们可以使用`packet_device`资源，指定所选计划、设施、操作系统、计费方式以及运行项目：

```
resource "packet_device" "swarm_master" {
  hostname         = "swarm-master"
  plan             = "${var.plan}"
  facility         = "${var.facility}"
  operating_system = "${var.operating_system}"
  billing_cycle    = "hourly"
  project_id       = "${packet_project.swarm.id}"
}
```

现在，让我们创建两个脚本，当服务器准备好后执行。第一个脚本将更新 Ubuntu（`update_os.sh`），而第二个脚本将安装 Docker（`install_docker.sh`）。

```
#!/usr/bin/env bash
# file: ./scripts/update_os.sh
sudo apt update -yqq
sudo apt upgrade -yqq

```

该脚本将安装并启动 Docker：

```
#!/usr/bin/env bash
# file: ./scripts/install_docker.sh
curl -sSL https://get.docker.com/ | sh
sudo systemctl enable docker
sudo systemctl start docker

```

现在，我们可以将这些脚本作为`remote-exec`配置器，在`packet_device`资源内调用：

```
  provisioner "remote-exec" {
    connection {
      user        = "${var.ssh_username}"
      private_key = "${file("${var.ssh_key}")}"
    }

    scripts = [
      "scripts/update_os.sh",
      "scripts/install_docker.sh",
    ]
  }
```

此时，系统已完全配置并正常运行，Docker 也正在运行。

要初始化 Docker Swarm 集群，从 Docker 1.12 开始，我们只需要执行以下命令：

```
$ docker swarm init --advertise-addr docker.manager.local.ip

```

Packet 上的服务器有一个接口，既共享公共 IP 地址，也共享私有 IP 地址。私有 IP 是第二个，且可以通过以下导出的属性访问：`${packet_device.swarm_master.network.2.address}`。我们来创建另一个`remote-exec`配置器，这样 Swarm 管理器将在启动后自动初始化：

```
  provisioner "remote-exec" {
    connection {
      user        = "${var.ssh_username}"
      private_key = "${file("${var.ssh_key}")}"
    }

    inline = [
      "docker swarm init --advertise-addr ${packet_device.swarm_master.network.2.address}",
    ]
  }
```

到此为止，我们已经有一个正在运行的 Docker 集群，只有一个节点——即管理节点本身。

最后一步是存储 Swarm 令牌，以便节点能够加入。可以使用以下命令获取令牌：

```
$ docker swarm join-token worker -q

```

我们将把此令牌存储在基础设施存储库中的一个简单文件（`worker.token`）中，这样我们可以访问并版本化它。我们来创建一个变量，将令牌存储在`variables.tf`中的文件中：

```
variable "worker_token_file" {
  default     = "worker.token"
  description = "Worker token file"
}
```

当其他操作完成后，我们将通过 SSH 执行先前的`docker swarm`命令，使用`local-exec`配置器。由于我们无法与进程进行交互，所以跳过主机密钥检查和其他初始的 SSH 检查：

```
  provisioner "local-exec" {
    command = "ssh -t -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -i ${var.ssh_key} ${var.ssh_username}@${packet_device.swarm_master.network.0.address} \"docker swarm join-token worker -q\" > ${var.worker_token_file}"
  }
```

我们现在完成了 Docker Swarm 管理器的配置！

### 使用 Terraform 在 Packet 上启动 Docker Swarm 节点

我们需要节点加入 Swarm 集群，以便工作负载能够分散。为了方便起见，节点的机器规格将与主节点相同。以下是将要发生的情况：

+   创建了两个节点

+   令牌文件已发送到每个节点

+   操作系统已更新，Docker 已安装

+   节点加入 Swarm 集群

让我们从在`variables.tf`中创建一个用于指定节点数量的变量开始：

```
variable "num_nodes" {
  default     = "1"
  description = "Number of Docker Swarm nodes"
}
```

随着集群的增长，在`terraform.tfvars`中重写该值：

```
num_nodes = "2"
```

使用与主节点相同的`packet_device`资源来创建节点：

```
resource "packet_device" "swarm_node" {
  count            = "${var.num_nodes}"
  hostname         = "swarm-node-${count.index+1}"
  plan             = "${var.plan}"
  facility         = "${var.facility}"
  operating_system = "${var.operating_system}"
  billing_cycle    = "hourly"
  project_id       = "${packet_project.swarm.id}"
}
```

添加一个`file`配置器来复制令牌文件：

```
  provisioner "file" {
    source      = "${var.worker_token_file}"
    destination = "${var.worker_token_file}"
  }
```

使用与主节点相同的更新和 Docker 安装脚本，创建相同的`remote-exec`配置器：

```
  provisioner "remote-exec" {
    connection {
      user        = "${var.ssh_username}"
      private_key = "${file("${var.ssh_key}")}"
    }

    scripts = [
      "scripts/update_os.sh",
      "scripts/install_docker.sh",
    ]
  }
```

操作系统现已完全更新，Docker 正在运行。

现在我们想加入 Docker Swarm 集群。为此，我们需要两项信息：token 和主节点的本地 IP 地址。我们已经在本地文件中有了 token，Terraform 也知道 Swarm 管理节点的本地 IP 地址。所以一个技巧是创建一个简单的脚本（建议你写一个更健壮的脚本！），它读取本地的 token，并将本地管理节点的 IP 地址作为参数传递。在名为 `scripts/join_swarm.sh` 的文件中，输入以下内容：

```
#!/usr/bin/env bash
# file: scripts/join_swarm.sh
MASTER=$1
SWARM_TOKEN=$(cat worker.token)
docker swarm join --token ${SWARM_TOKEN} ${MASTER}:2377
```

现在我们只需要使用 `file` 提供者将这个文件发送到各个节点：

```
  provisioner "file" {
    source      = "scripts/join_swarm.sh"
    destination = "join_swarm.sh"
  }
```

作为最后一步，通过 `remote-exec` 提供者使用它，将本地 Docker 主节点 IP (`${packet_device.swarm_master.network.2.address}"`) 作为参数传递给脚本：

```
  provisioner "remote-exec" {
    connection {
      user        = "${var.ssh_username}"
      private_key = "${file("${var.ssh_key}")}"
    }

    inline = [
      "chmod +x join_swarm.sh",
      "./join_swarm.sh ${packet_device.swarm_master.network.2.address}",
    ]
  }//.
```

启动整个基础设施：

```
$ terraform apply
Outputs:

Swarm Master Private IP = 10.80.86.129
Swarm Master Public IP = 147.75.100.19
Swarm Nodes = Public: 147.75.100.23,147.75.100.3, Private: 10.80.86.135,10.80.86.133

```

我们的集群正在运行。

### 使用 Docker Swarm 集群

使用我们的 Docker Swarm 集群超出了本书的范围，但现在我们已经有了它，让我们快速看看如何将容器扩展到数千个！

验证我们是否有 3 个节点：

```
# docker node ls
ID                           HOSTNAME                STATUS  AVAILABILITY  MANAGER STATUS
9sxqi2f1pywmofgf63l84n7ps *  swarm-master.local.lan  Ready   Active        Leader
ag07nh1wzsbsvnef98sqf5agy    swarm-node-1.local.lan  Ready   Active
cppk5ja4spysu6opdov9f3x8h    swarm-node-2.local.lan  Ready   Active
```

我们需要为我们的容器创建一个共享网络，并且希望能够扩展到数千个容器。所以一个典型的 /24 网络是不够的（那是 `docker network` 的默认设置）。我们来创建一个 /16 的覆盖网络，这样就有足够的扩展空间了！

```
# docker network create -d overlay --subnet 172.16.0.0/16 nginx-network

```

创建一个 Docker 服务，它将在这个新的覆盖网络上启动一个 nginx 容器，并且有 3 个副本（容器的 3 个实例同时运行）：

```
# docker service create --name nginx --network nginx-network --replicas 3 -p 80:80/tcp nginx

```

验证它是否正常工作：

```
# docker service ls
ID            NAME   REPLICAS  IMAGE  COMMAND
aeq9lspl0mpg  nginx  3/3       nginx

```

现在，通过 HTTP 访问集群的任何公共 IP 地址，任何节点上的容器都可以响应：我们可以向节点-1 发起 HTTP 请求，而响应可能来自节点-2 上的一个容器。很棒！

现在我们来扩展我们的服务，从 3 个副本扩展到 100 个：

```
# docker service scale nginx=100
nginx scaled to 100
# docker service ls
ID            NAME   REPLICAS  IMAGE  COMMAND
aeq9lspl0mpg  nginx  100/100   nginx

```

我们在几秒钟内将容器扩展到一百个，并将它们分布到所有 3 台裸金属机器上。

现在，你知道可以进行扩展，并且通过这样的配置，你可以将 `nginx` 服务扩展到 500 个、1000 个，甚至更多！

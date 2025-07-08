# 第八章：在 OpenStack 中使用 LXC

在前面的章节中，我们探讨了利用 Jenkins、自定义 REST API 和监控工具等工具帮助自动扩展 LXC 容器中运行的服务的常见设计模式示例。在本章中，我们将探讨一种通过使用 OpenStack 在一组服务器上完全自动化地配置 LXC 容器的方法。

OpenStack 是一个云操作系统，允许以集中但模块化和可扩展的方式配置虚拟机、LXC 容器、负载均衡器、数据库以及存储和网络资源。它非常适合管理一组计算资源（服务器），并根据 CPU 负载、内存使用率、虚拟机/容器密度等标准，选择最佳的目标来配置服务。

在本章中，我们将涵盖以下 OpenStack 组件和服务：

+   部署 Keystone 身份服务，该服务将提供一个集中式的用户和服务目录，并提供一种使用令牌进行身份验证的简单方法

+   安装 Nova 计算控制器，它将管理一池服务器并在其上配置 LXC 容器

+   配置 Glance 镜像仓库，该仓库将存储 LXC 镜像

+   配置 Neutron 网络服务，它将管理计算主机上的 DHCP、DNS 和网络桥接

+   最后，我们将使用 libvirt OpenStack 驱动程序配置一个 LXC 容器

# 在 Ubuntu 上部署支持 LXC 的 OpenStack

OpenStack 部署可能由多个组件组成，这些组件通过暴露的 API 或像 RabbitMQ 这样的消息总线相互作用，如下图所示：

![在 Ubuntu 上部署支持 LXC 的 OpenStack](img/image_08_001.jpg)

在本章中，我们将部署这些组件的最小集——Keystone、Glance、Nova 和 Neutron——这些组件足以配置 LXC 容器，并且仍然能够利用 OpenStack 提供的调度逻辑和可扩展的网络。

在本教程中，我们将使用 Ubuntu Xenial，并且截至本写作时间，使用最新的 Newton OpenStack 版本。那个 OpenStack 版本的名称是什么？

## 准备主机

为了简化操作，我们将使用单个服务器托管所有服务，最小内存为 16 GB。在生产环境中，常见的做法是将每个服务分离到自己的服务器组中，以便于扩展和高可用性。按照本章中的步骤，您可以通过根据需要更换 IP 地址和主机名，轻松地在多个主机上进行部署。

### 提示

如果使用多个服务器，您需要确保所有主机的时间已同步，可以使用 `ntpd` 等服务。

让我们首先确保我们拥有最新的包，并安装包含 Newton OpenStack 版本的仓库：

```
root@controller:~# apt install -y software-properties-common 
root@controller:~# add-apt-repository cloud-archive:newton 
. . . 
Press [ENTER] to continue or ctrl-c to cancel adding it 
. . . 
OK 
root@controller:~# apt update && apt dist-upgrade -y 
root@controller:~# reboot 
root@controller:~# apt install -y python-openstackclient

```

确保将服务器名称（在此示例中为`controller`）添加到`/etc/hosts`。

## 安装数据库服务

我们将要部署的服务使用 MariaDB 数据库作为其后端存储。通过运行以下命令来安装它：

```
root@controller:~# apt install -y mariadb-server python-pymysql

```

最小的配置文件应类似于以下内容：

```
root@controller:~# cat /etc/mysql/mariadb.conf.d/99-openstack.cnf 
[mysqld] 
bind-address = 10.208.130.36 
default-storage-engine = innodb 
innodb_file_per_table 
max_connections = 4096 
collation-server = utf8_general_ci 
character-set-server = utf8 
root@controller:~#

```

将服务绑定的 IP 地址替换为您服务器上的地址，然后启动服务并运行将保护安装的脚本：

```
root@controller:~# service mysql restart 
root@controller:~# mysql_secure_installation

```

上述命令将提示输入新 root 密码。为简便起见，我们将在本章其余部分使用 `lxcpassword` 作为所有服务的密码。

验证 MySQL 是否正确设置并能够连接：

```
root@controller:~# mysql -u root -h 10.208.130.36 -p 
Enter password: [lxcpassword] 
Welcome to the MariaDB monitor. Commands end with; or \g. 
Your MariaDB connection id is 47 
Server version: 10.0.28-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04 
Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others. 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement. 
MariaDB [(none)]> [quit]

```

## 安装消息队列服务

OpenStack 支持以下消息队列——RabbitMQ、Qpid 和 ZeroMQ——它们促进了服务之间的进程间通信。我们将使用 RabbitMQ：

```
root@controller:~# apt install -y rabbitmq-server

```

添加新用户和密码：

```
root@controller:~# rabbitmqctl add_user openstack lxcpassword 
Creating user "openstack" ... 
root@controller:~# 

```

同时，授予该用户权限：

```
root@controller:~# rabbitmqctl set_permissions openstack ".*" ".*" ".*" 
Setting permissions for user "openstack" in vhost "/" ... 
root@controller:~#

```

## 安装缓存服务

Keystone 身份服务使用 Memcached 来缓存身份验证令牌。要安装它，请执行以下命令：

```
root@controller:~# apt install -y memcached python-memcache

```

将本地主机地址替换为您服务器的 IP 地址：

```
root@controller:~# sed -i 's/127.0.0.1/10.208.130.36/g' /etc/memcached.conf

```

配置文件应类似于以下内容：

```
root@controller:~# cat /etc/memcached.conf | grep -vi "#" | sed '/^$/d' 
-d 
logfile /var/log/memcached.log 
-m 64 
-p 11211 
-u memcache 
-l 10.208.130.36 
root@controller:~# service memcached restart

```

## 安装和配置身份服务

Keystone 身份服务为管理其余 OpenStack 组件的身份验证和授权提供了集中式的管理点。Keystone 还维护着一个服务目录，记录了各个服务及其提供的端点，用户可以通过查询该目录来定位这些服务。

要部署 Keystone，首先创建一个数据库并授予 `keystone` 用户权限：

```
root@controller:~# mysql -u root -plxcpassword 
MariaDB [(none)]> CREATE DATABASE keystone; 
Query OK, 1 row affected (0.01 sec) 
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'lxcpassword'; 
Query OK, 0 rows affected (0.00 sec) 
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'lxcpassword'; 
Query OK, 0 rows affected (0.01 sec) 
MariaDB [(none)]> exit 
Bye 
root@controller:~#

```

接下来，安装身份服务组件：

```
root@controller:~# apt install -y keystone

```

以下是 Keystone 的最小工作配置：

```
root@controller:~# cat /etc/keystone/keystone.conf 
[DEFAULT] 
log_dir = /var/log/keystone 
[assignment] 
[auth] 
[cache] 
[catalog] 
[cors] 
[cors.subdomain] 
[credential] 
[database] 
connection = mysql+pymysql://keystone:lxcpassword@controller/keystone 
[domain_config] 
[endpoint_filter] 
[endpoint_policy] 
[eventlet_server] 
[federation] 
[fernet_tokens] 
[identity] 
[identity_mapping] 
[kvs] 
[ldap] 
[matchmaker_redis] 
[memcache] 
[oauth1] 
[os_inherit] 
[oslo_messaging_amqp] 
[oslo_messaging_notifications] 
[oslo_messaging_rabbit] 
[oslo_messaging_zmq] 
[oslo_middleware] 
[oslo_policy] 
[paste_deploy] 
[policy] 
[profiler] 
[resource] 
[revoke] 
[role] 
[saml] 
[security_compliance] 
[shadow_users] 
[signing] 
[token] 
provider = fernet 
[tokenless_auth] 
[trust] 
[extra_headers] 
Distribution = Ubuntu 
root@controller:~#

```

如果您使用的是与本教程相同的主机名和密码，则无需更改。

接下来，通过运行以下命令填充 `keystone` 数据库：

```
root@controller:~# su -s /bin/sh -c "keystone-manage db_sync" keystone 
... 
root@controller:~#

```

Keystone 使用令牌来验证和授权用户及服务。提供了不同的令牌格式，例如 UUID、PKI 和 Fernet 令牌。在这个示例部署中，我们将使用 Fernet 令牌，它与其他类型不同，不需要在后端持久化。要初始化 Fernet 密钥库，请运行以下命令：

```
root@controller:~# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone 
root@controller:~# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone 
root@controller:~#

```

### 注意

欲了解有关可用身份令牌的更多信息，请参阅 [`docs.openstack.org/admin-guide/identity-tokens.html`](http://docs.openstack.org/admin-guide/identity-tokens.html)。

执行以下命令进行基本的引导过程：

```
root@controller:~# keystone-manage bootstrap --bootstrap-password lxcpassword --bootstrap-admin-url http://controller:35357/v3/ --bootstrap-internal-url http://controller:35357/v3/ --bootstrap-public-url http://controller:5000/v3/ --bootstrap-region-id RegionOne 
root@controller:~#

```

我们将使用 Apache 和 WSGI 模块来驱动 Keystone。在 Apache 配置文件中添加以下配置段并重新启动：

```
root@controller:~# cat /etc/apache2/apache2.conf 
... 
ServerName controller 
... 
root@controller:~# service apache2 restart 

```

删除 Keystone 默认随附的 SQLite 数据库：

```
root@controller:~# rm -f /var/lib/keystone/keystone.db

```

让我们通过定义以下环境变量来创建管理员账户：

```
root@controller:~# export OS_USERNAME=admin 
root@controller:~# export OS_PASSWORD=lxcpassword 
root@controller:~# export OS_PROJECT_NAME=admin 
root@controller:~# export OS_USER_DOMAIN_NAME=default 
root@controller:~# export OS_PROJECT_DOMAIN_NAME=default 
root@controller:~# export OS_AUTH_URL=http://controller:35357/v3 
root@controller:~# export OS_IDENTITY_API_VERSION=3

```

现在是时候在 Keystone 中创建我们的第一个项目了。项目代表一个所有权单位，其中的所有资源都归项目所有。我们接下来要创建的 `service` 项目将用于我们在本章中将要部署的所有服务：

```
root@controller:~# openstack project create --domain default --description "LXC Project" service 
+-------------+----------------------------------+ 
| Field | Value | 
+-------------+----------------------------------+ 
| description | LXC Project | 
| domain_id | default | 
| enabled | True | 
| id | 9a1a863fe41b42b2955b313f2cca0ef0 | 
| is_domain | False | 
| name | service | 
| parent_id | default | 
+-------------+----------------------------------+ 
root@controller:~#

```

要列出可用的项目，请运行以下命令：

```
root@controller:~# openstack project list 
+----------------------------------+---------+ 
| ID | Name | 
+----------------------------------+---------+ 
| 06f4e2d7e384474781803395b24b3af2 | admin | 
| 9a1a863fe41b42b2955b313f2cca0ef0 | service | 
+----------------------------------+---------+ 
root@controller:~#

```

让我们创建一个普通的项目和用户，普通用户可以使用该项目，而不是 OpenStack 服务：

```
root@controller:~# openstack project create --domain default --description "LXC User Project" lxc 
+-------------+----------------------------------+ 
| Field | Value | 
+-------------+----------------------------------+ 
| description | LXC User Project | 
| domain_id | default | 
| enabled | True | 
| id | eb9cdc2c2b4e4f098f2d104752970d52 | 
| is_domain | False | 
| name | lxc | 
| parent_id | default | 
+-------------+----------------------------------+ 
root@controller:~# 
root@controller:~# openstack user create --domain default --password-prompt lxc 
User Password: 
Repeat User Password: 
+---------------------+----------------------------------+ 
| Field | Value | 
+---------------------+----------------------------------+ 
| domain_id | default | 
| enabled | True | 
| id | 1e83e0c8ca194f2e9d8161eb61d21030 | 
| name | lxc | 
| password_expires_at | None | 
+---------------------+----------------------------------+ 
root@controller:~#

```

接下来，创建一个 `user` 角色，并将其与我们在前两步中创建的 `lxc` 项目和用户关联：

```
root@controller:~# openstack role create user 
+-----------+----------------------------------+ 
| Field | Value | 
+-----------+----------------------------------+ 
| domain_id | None | 
| id | 331c0b61e9784112874627264f03a058 | 
| name | user | 
+-----------+----------------------------------+ 
root@controller:~# openstack role add --project lxc --user lxc user 
root@controller:~#

```

使用以下文件来配置 Keystone 的 **Web 服务网关接口** (**WSGI**) 中间件管道：

```
root@controller:~# cat /etc/keystone/keystone-paste.ini 
# Keystone PasteDeploy configuration file. 
[filter:debug] 
use = egg:oslo.middleware#debug 
[filter:request_id] 
use = egg:oslo.middleware#request_id 
[filter:build_auth_context] 
use = egg:keystone#build_auth_context 
[filter:token_auth] 
use = egg:keystone#token_auth 
[filter:admin_token_auth] 
use = egg:keystone#admin_token_auth 
[filter:json_body] 
use = egg:keystone#json_body 
[filter:cors] 
use = egg:oslo.middleware#cors 
oslo_config_project = keystone 
[filter:http_proxy_to_wsgi] 
use = egg:oslo.middleware#http_proxy_to_wsgi 
[filter:ec2_extension] 
use = egg:keystone#ec2_extension 
[filter:ec2_extension_v3] 
use = egg:keystone#ec2_extension_v3 
[filter:s3_extension] 
use = egg:keystone#s3_extension 
[filter:url_normalize] 
use = egg:keystone#url_normalize 
[filter:sizelimit] 
use = egg:oslo.middleware#sizelimit 
[filter:osprofiler] 
use = egg:osprofiler#osprofiler 
[app:public_service] 
use = egg:keystone#public_service 
[app:service_v3] 
use = egg:keystone#service_v3 
[app:admin_service] 
use = egg:keystone#admin_service 
[pipeline:public_api] 
pipeline = cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension public_service 
[pipeline:admin_api] 
pipeline = cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension s3_extension admin_service 
[pipeline:api_v3] 
pipeline = cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension_v3 s3_extension service_v3 
[app:public_version_service] 
use = egg:keystone#public_version_service 
[app:admin_version_service] 
use = egg:keystone#admin_version_service 
[pipeline:public_version_api] 
pipeline = cors sizelimit osprofiler url_normalize public_version_service 
[pipeline:admin_version_api] 
pipeline = cors sizelimit osprofiler url_normalize admin_version_service 
[composite:main] 
use = egg:Paste#urlmap 
/v2.0 = public_api 
/v3 = api_v3 
/ = public_version_api 
[composite:admin] 
use = egg:Paste#urlmap 
/v2.0 = admin_api 
/v3 = api_v3 
/ = admin_version_api 
root@controller:~#

```

让我们测试到目前为止的配置，通过请求 `admin` 用户和 `lxc` 用户的令牌：

```
root@controller:~# openstack --os-auth-url http://controller:35357/v3 --os-project-domain-name default --os-user-domain-name default --os-project-name admin --os-username admin token issue 
Password: 
... 
root@controller:~# openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name default --os-user-domain-name default --os-project-name lxc --os-username lxc token issue 
Password: 
... 
root@controller:~#

```

我们可以创建两个文件，分别包含我们之前配置的 `admin` 和 `user` 凭证：

```
root@controller:~# cat rc.admin 
export OS_PROJECT_DOMAIN_NAME=default 
export OS_USER_DOMAIN_NAME=default 
export OS_PROJECT_NAME=admin 
export OS_USERNAME=admin 
export OS_PASSWORD=lxcpassword 
export OS_AUTH_URL=http://controller:35357/v3 
export OS_IDENTITY_API_VERSION=3 
export OS_IMAGE_API_VERSION=2 
root@controller:~# 
root@controller:~# cat rc.lxc 
export OS_PROJECT_DOMAIN_NAME=default 
export OS_USER_DOMAIN_NAME=default 
export OS_PROJECT_NAME=lxc 
export OS_USERNAME=lxc 
export OS_PASSWORD=lxcpassword 
export OS_AUTH_URL=http://controller:5000/v3 
export OS_IDENTITY_API_VERSION=3 
export OS_IMAGE_API_VERSION=2 
root@controller:~#

```

例如，要使用 `admin` 用户，请按如下方式加载文件：

```
root@controller:~# . rc.admin

```

注意新的环境变量：

```
root@controller:~# env | grep ^OS 
OS_USER_DOMAIN_NAME=default 
OS_IMAGE_API_VERSION=2 
OS_PROJECT_NAME=admin 
OS_IDENTITY_API_VERSION=3 
OS_PASSWORD=lxcpassword 
OS_AUTH_URL=http://controller:35357/v3 
OS_USERNAME=admin 
OS_PROJECT_DOMAIN_NAME=default 
root@controller:~#

```

在加载了管理员凭证后，让我们请求一个认证令牌，之后可以与其他 OpenStack 服务一起使用：

```
root@controller:~# openstack token issue 
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ 
| Field | Value | 
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ 
| expires | 2016-12-02 19:49:07+00:00 | 
| id | gAAAAABYQcIj7eKEfCMTWY43EXZbqZ8UdeZ8CZIb2l2sqIFHFBV_bv6LHO4CFbFLdh7kUEw_Zk-MzQrl5mbq7g8RXPAZ31iBDpDie2-xIAMgRqsxkAh7PJ2kdhcHAxkBj-Uq65rHmjYmPZTUlUTONOP3_dId0_8DsdLkFWoardxG0FAotrlH-2s | 
| project_id | 06f4e2d7e384474781803395b24b3af2 | 
| user_id | 5c331f397597439faef5a1199cdf354f | 
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ 
root@controller:~#

```

## 安装和配置镜像服务

镜像服务为用户提供一个 API，用于发现、注册和获取虚拟机镜像，或可作为 LXC 容器根文件系统的镜像。Glance 支持多种存储后端，但为了简便起见，我们将使用文件存储，它将 LXC 镜像直接保存在文件系统中。

要部署 Glance，首先创建一个数据库和一个用户，就像我们为 Keystone 所做的那样：

```
root@controller:~# mysql -u root -plxcpassword 
MariaDB [(none)]> CREATE DATABASE glance; 
Query OK, 1 row affected (0.00 sec) 
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'lxcpassword'; 
Query OK, 0 rows affected (0.00 sec) 
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'lxcpassword'; 
Query OK, 0 rows affected (0.00 sec) 
MariaDB [(none)]> exit 
Bye 
root@controller:~#

```

接下来，创建 `glance` 用户并将其添加到 `admin` 角色中：

```
root@controller:~# openstack user create --domain default --password-prompt glance 
User Password: 
Repeat User Password: 
+---------------------+----------------------------------+ 
| Field | Value | 
+---------------------+----------------------------------+ 
| domain_id | default | 
| enabled | True | 
| id | ce29b972845d4d77978b7e7803275d53 | 
| name | glance | 
| password_expires_at | None | 
+---------------------+----------------------------------+ 
root@controller:~# openstack role add --project service --user glance admin 
root@controller:~#

```

现在是创建 `glance` 服务记录的时候了：

```
root@controller:~# openstack service create --name glance --description "OpenStack Image" image 
+-------------+----------------------------------+ 
| Field | Value | 
+-------------+----------------------------------+ 
| description | OpenStack Image | 
| enabled | True | 
| id | 2aa82fc0a0224baab8d259e4f5279907 | 
| name | glance | 
| type | image | 
+-------------+----------------------------------+ 
root@controller:~#

```

在 Keystone 中创建 Glance API 端点：

```
root@controller:~# openstack endpoint create --region RegionOne image public http://controller:9292 
+--------------+----------------------------------+ 
| Field | Value | 
+--------------+----------------------------------+ 
| enabled | True | 
| id | aa26c33d456d421ca3555e6523c7814f | 
| interface | public | 
| region | RegionOne | 
| region_id | RegionOne | 
| service_id | 2aa82fc0a0224baab8d259e4f5279907 | 
| service_name | glance | 
| service_type | image | 
| url | http://controller:9292 | 
+--------------+----------------------------------+ 
root@controller:~#

```

OpenStack 支持多区域部署以实现高可用性；然而，为了简单起见，我们将所有服务部署在同一区域：

```
root@controller:~# openstack endpoint create --region RegionOne image internal http://controller:9292 
... 
root@controller:~# openstack endpoint create --region RegionOne image admin http://controller:9292 
... 
root@controller:~#

```

既然 Keystone 已经知道了 `glance` 服务，让我们安装它：

```
root@controller:~# apt install -y glance

```

使用以下两个最小配置文件，根据需要替换密码和主机名：

```
root@controller:~# cat /etc/glance/glance-api.conf 
[DEFAULT] 
[cors] 
[cors.subdomain] 
[database] 
connection = mysql+pymysql://glance:lxcpassword@controller/glance 
[glance_store] 
stores = file,http 
default_store = file 
filesystem_store_datadir = /var/lib/glance/images/ 
[image_format] 
disk_formats = ami,ari,aki,vhd,vhdx,vmdk,raw,qcow2,vdi,iso,root-tar 
[keystone_authtoken] 
auth_uri = http://controller:5000 
auth_url = http://controller:35357 
memcached_servers = controller:11211 
auth_type = password 
project_domain_name = default 
user_domain_name = default 
project_name = service 
username = glance 
password = lxcpassword 
[matchmaker_redis] 
[oslo_concurrency] 
[oslo_messaging_amqp] 
[oslo_messaging_notifications] 
[oslo_messaging_rabbit] 
[oslo_messaging_zmq] 
[oslo_middleware] 
[oslo_policy] 
[paste_deploy] 
flavor = keystone 
[profiler] 
[store_type_location_strategy] 
[task] 
[taskflow_executor] 
root@controller:~# 
root@controller:~# cat /etc/glance/glance-registry.conf 
[DEFAULT] 
[database] 
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance 
[keystone_authtoken] 
auth_uri = http://controller:5000 
auth_url = http://controller:35357 
memcached_servers = controller:11211 
auth_type = password 
project_domain_name = default 
user_domain_name = default 
project_name = service 
username = glance 
password = lxcpassword 
[matchmaker_redis] 
[oslo_messaging_amqp] 
[oslo_messaging_notifications] 
[oslo_messaging_rabbit] 
[oslo_messaging_zmq] 
[oslo_policy] 
[paste_deploy] 
flavor = keystone 
[profiler] 
root@controller:~#

```

通过运行以下命令填充 `glance` 数据库：

```
root@controller:~# su -s /bin/sh -c "glance-manage db_sync" glance 
... 
root@controller:~#

```

启动 Glance 服务：

```
root@controller:~# service glance-registry restart 
root@controller:~# service glance-api restart 
root@controller:~#

```

如我们在第二章 *安装和运行 LXC 在 Linux 系统上* 中看到的，我们可以手动为 LXC 容器构建镜像，或者从 Ubuntu 仓库下载预构建的镜像。让我们下载一个镜像并提取它：

```
root@controller:~# wget http://uec-images.ubuntu.com/releases/precise/release/ubuntu-12.04-server-cloudimg-amd64.tar.gz 
root@controller:~# tar zxfv ubuntu-12.04-server-cloudimg-amd64.tar.gz 
precise-server-cloudimg-amd64.img 
precise-server-cloudimg-amd64-vmlinuz-virtual 
precise-server-cloudimg-amd64-loader 
precise-server-cloudimg-amd64-floppy 
README.files 
root@controller:~#

```

包含根文件系统的文件具有 `.img` 扩展名。让我们将其添加到镜像服务中：

```
root@controller:~# openstack image create "lxc_ubuntu_12.04" --file precise-server-cloudimg-amd64.img --disk-format raw --container-format bare --public 
+------------------+------------------------------------------------------+ 
| Field | Value | 
+------------------+------------------------------------------------------+ 
| checksum | b5e5895e85127d9cebbd2de32d9b193c | 
| container_format | bare | 
| created_at | 2016-12-02T19:01:28Z | 
| disk_format | raw | 
| file | /v2/images/f344646d-d293-4638-bab4-86d461f38233/file | 
| id | f344646d-d293-4638-bab4-86d461f38233 | 
| min_disk | 0 | 
| min_ram | 0 | 
| name | lxc_ubuntu_12.04 | 
| owner | 06f4e2d7e384474781803395b24b3af2 | 
| protected | False | 
| schema | /v2/schemas/image | 
| size | 1476395008 | 
| status | active | 
| tags | | 
| updated_at | 2016-12-02T19:01:41Z | 
| virtual_size | None | 
| visibility | public | 
+------------------+------------------------------------------------------+ 
root@controller:~#

```

### 注

请注意，LXC 使用 `raw` 磁盘和 `bare` 容器格式。

镜像现在存储在 `glance-api.conf` 中定义的 `filesystem_store_datadir` 位置，如我们在前面的配置示例中看到的：

```
root@controller:~# ls -la /var/lib/glance/images/ 
total 1441804 
drwxr-xr-x 2 glance glance 4096 Dec 2 19:01 . 
drwxr-xr-x 4 glance glance 4096 Dec 2 18:53 .. 
-rw-r----- 1 glance glance 1476395008 Dec 2 19:01 f344646d-d293-4638-bab4-86d461f38233 
root@controller:~#

```

让我们列出 Glance 中可用的镜像：

```
root@controller:~# openstack image list 
+--------------------------------------+------------------+--------+ 
| ID | Name | Status | 
+--------------------------------------+------------------+--------+ 
| f344646d-d293-4638-bab4-86d461f38233 | lxc_ubuntu_12.04 | active | 
+--------------------------------------+------------------+--------+ 
root@controller:~#

```

## 安装和配置计算服务

OpenStack 计算服务管理一个计算资源池（服务器）和在这些资源上运行的各种虚拟机或容器。它提供一个调度服务，接收来自队列的创建新虚拟机或容器的请求，并决定在哪个计算主机上创建并启动它。

### 注

有关各种 Nova 服务的更多信息，请参考：[`docs.openstack.org/developer/nova/`](http://docs.openstack.org/developer/nova/)。

让我们从创建`nova`数据库和用户开始：

```
root@controller:~# mysql -u root -plxcpassword 
MariaDB [(none)]> CREATE DATABASE nova_api; 
Query OK, 1 row affected (0.00 sec) 
MariaDB [(none)]> CREATE DATABASE nova; 
Query OK, 1 row affected (0.00 sec) 
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'lxcpassword'; 
Query OK, 0 rows affected (0.03 sec) 
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'lxcpassword'; 
Query OK, 0 rows affected (0.00 sec) 
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'lxcpassword'; 
Query OK, 0 rows affected (0.00 sec) 
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'lxcpassword'; 
Query OK, 0 rows affected (0.00 sec) 
MariaDB [(none)]> exit 
Bye 
root@controller:~#

```

一旦数据库创建并授权了用户权限，创建`nova`用户并将其添加到身份服务中的`admin`角色：

```
root@controller:~# openstack user create --domain default --password-prompt nova 
User Password: 
Repeat User Password: 
+---------------------+----------------------------------+ 
| Field | Value | 
+---------------------+----------------------------------+ 
| domain_id | default | 
| enabled | True | 
| id | a1305c903548431a80b608fadf78f287 | 
| name | nova | 
| password_expires_at | None | 
+---------------------+----------------------------------+ 
root@controller:~# openstack role add --project service --user nova admin 
root@controller:~#

```

接下来，创建`nova`服务和端点：

```
root@controller:~# openstack service create --name nova --description "OpenStack Compute" compute 
+-------------+----------------------------------+ 
| Field | Value | 
+-------------+----------------------------------+ 
| description | OpenStack Compute | 
| enabled | True | 
| id | 779d2feb591545cf9d2acc18765a0ca5 | 
| name | nova | 
| type | compute | 
+-------------+----------------------------------+ 
root@controller:~# openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1/%\(tenant_id\)s 
+--------------+-------------------------------------------+ 
| Field | Value | 
+--------------+-------------------------------------------+ 
| enabled | True | 
| id | cccfd4d817a24f9ba58128901cbbb473 | 
| interface | public | 
| region | RegionOne | 
| region_id | RegionOne | 
| service_id | 779d2feb591545cf9d2acc18765a0ca5 | 
| service_name | nova | 
| service_type | compute | 
| url | http://controller:8774/v2.1/%(tenant_id)s | 
+--------------+-------------------------------------------+ 
root@controller:~# openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1/%\(tenant_id\)s 
... 
root@controller:~# openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1/%\(tenant_id\)s 
... 
root@controller:~#

```

现在是时候安装提供 API、调度器、控制台和调度服务的 Nova 软件包了：

```
root@controller:~# apt install -y nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler

```

我们刚刚安装的 Nova 软件包提供以下服务：

+   `nova-api`：该服务通过 RESTful API 接受并响应用户请求。我们使用它来创建、运行和停止实例等等。

+   `nova-conductor`：该服务位于我们之前创建的`nova`数据库与运行在计算节点上的`nova-compute`服务之间，后者负责创建虚拟机和容器。我们将在本章后续部分安装该服务。

+   `nova-consoleauth`：该服务为希望通过各种控制台连接到虚拟机或容器的用户授权令牌。

+   `nova-novncproxy`：该服务提供对运行 VNC 的实例的访问。

+   `nova-scheduler`：如前所述，该服务负责决定在哪里部署虚拟机或 LXC 容器。

以下是一个最小功能的 Nova 配置：

```
root@controller:~# cat /etc/nova/nova.conf 
[DEFAULT] 
dhcpbridge_flagfile=/etc/nova/nova.conf 
dhcpbridge=/usr/bin/nova-dhcpbridge 
log-dir=/var/log/nova 
state_path=/var/lib/nova 
force_dhcp_release=True 
verbose=True 
ec2_private_dns_show_ip=True 
enabled_apis=osapi_compute,metadata 
transport_url = rabbit://openstack:lxcpassword@controller 
auth_strategy = keystone 
my_ip = 10.208.132.45 
use_neutron = True 
firewall_driver = nova.virt.firewall.NoopFirewallDriver 
[database] 
connection = mysql+pymysql://nova:lxcpassword@controller/nova 
[api_database] 
connection = mysql+pymysql://nova:lxcpassword@controller/nova_api 
[oslo_concurrency] 
lock_path = /var/lib/nova/tmp 
[libvirt] 
use_virtio_for_bridges=True 
[wsgi] 
api_paste_config=/etc/nova/api-paste.ini 
[keystone_authtoken] 
auth_uri = http://controller:5000 
auth_url = http://controller:35357 
memcached_servers = controller:11211 
auth_type = password 
project_domain_name = default 
user_domain_name = default 
project_name = service 
username = nova 
password = lxcpassword 
[vnc] 
vncserver_listen = $my_ip 
vncserver_proxyclient_address = $my_ip 
[glance] 
api_servers = http://controller:9292 
root@controller:~#

```

配置文件就绪后，我们现在可以填充`nova`数据库：

```
root@controller:~# su -s /bin/sh -c "nova-manage api_db sync" nova 
... 
root@controller:~# su -s /bin/sh -c "nova-manage db sync" nova 
... 
root@controller:~#

```

最后，启动计算服务：

```
root@controller:~# service nova-api restart 
root@controller:~# service nova-consoleauth restart 
root@controller:~# service nova-scheduler restart 
root@controller:~# service nova-conductor restart 
root@controller:~# service nova-novncproxy restart 
root@controller:~#

```

由于我们将使用单节点进行此 OpenStack 部署，因此需要安装`nova-compute`服务。在生产环境中，我们通常有一个计算服务器池，只运行该服务。

```
root@controller:~# apt install -y nova-compute

```

使用以下最小配置文件，该文件允许在同一服务器上运行`nova-compute`及其余 Nova 服务：

```
root@controller:~# cat /etc/nova/nova.conf 
[DEFAULT] 
dhcpbridge_flagfile=/etc/nova/nova.conf 
dhcpbridge=/usr/bin/nova-dhcpbridge 
log-dir=/var/log/nova 
state_path=/var/lib/nova 
force_dhcp_release=True 
verbose=True 
ec2_private_dns_show_ip=True 
enabled_apis=osapi_compute,metadata 
transport_url = rabbit://openstack:lxcpassword@controller 
auth_strategy = keystone 
my_ip = 10.208.132.45 
use_neutron = True 
firewall_driver = nova.virt.firewall.NoopFirewallDriver 
compute_driver = libvirt.LibvirtDriver 
[database] 
connection = mysql+pymysql://nova:lxcpassword@controller/nova 
[api_database] 
connection = mysql+pymysql://nova:lxcpassword@controller/nova_api 
[oslo_concurrency] 
lock_path = /var/lib/nova/tmp 
[libvirt] 
use_virtio_for_bridges=True 
[wsgi] 
api_paste_config=/etc/nova/api-paste.ini 
[keystone_authtoken] 
auth_uri = http://controller:5000 
auth_url = http://controller:35357 
memcached_servers = controller:11211 
auth_type = password 
project_domain_name = default 
user_domain_name = default 
project_name = service 
username = nova 
password = lxcpassword 
[vnc] 
enabled = True 
vncserver_listen = $my_ip 
vncserver_proxyclient_address = $my_ip 
novncproxy_base_url = http://controller:6080/vnc_auto.html 
[glance] 
api_servers = http://controller:9292 
[libvirt] 
virt_type = lxc 
root@controller:~#

```

请注意，在`libvirt`部分下，我们如何指定 LXC 作为默认的虚拟化类型。要在 Nova 中启用 LXC 支持，安装以下软件包：

```
root@controller:~# apt install -y nova-compute-lxc

```

该软件包提供以下配置文件：

```
root@controller:~# cat /etc/nova/nova-compute.conf 
[DEFAULT] 
compute_driver=libvirt.LibvirtDriver 
[libvirt] 
virt_type=lxc 
root@controller:~#

```

重启`nova-compute`服务并列出所有可用的 Nova 服务：

```
root@controller:~# service nova-compute restart 
root@controller:~# openstack compute service list 
+----+------------------+------------+----------+---------+-------+----------------------------+ 
| ID | Binary | Host | Zone | Status | State | Updated At | 
+----+------------------+------------+----------+---------+-------+----------------------------+ 
| 4 | nova-consoleauth | controller | internal | enabled | up | 2016-12-02T20:01:19.000000 | 
| 5 | nova-scheduler | controller | internal | enabled | up | 2016-12-02T20:01:25.000000 | 
| 6 | nova-conductor | controller | internal | enabled | up | 2016-12-02T20:01:26.000000 | 
| 8 | nova-compute | controller | nova | enabled | up | 2016-12-02T20:01:22.000000 | 
+----+------------------+------------+----------+---------+-------+----------------------------+ 
root@controller:~#

```

确保验证所有四个服务都已`启用`并处于运行状态。配置并运行所有 Nova 服务后，现在是时候进入部署的网络部分了。

## 安装和配置网络服务

OpenStack 的网络组件，代号 Neutron，管理网络、IP 地址、软件桥接和路由。在前面的章节中，我们需要创建 Linux 桥接，向其添加端口，配置 DHCP 为容器分配 IP 地址等等。Neutron 通过方便的 API 和库暴露了所有这些功能，我们可以利用它们。

让我们从创建数据库、用户和权限开始：

```
root@controller:~# mysql -u root -plxcpassword 
MariaDB [(none)]> CREATE DATABASE neutron; 
Query OK, 1 row affected (0.00 sec) 
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'lxcpassword'; 
Query OK, 0 rows affected (0.00 sec) 
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'lxcpassword'; 
Query OK, 0 rows affected (0.00 sec) 
MariaDB [(none)]> exit 
Bye 
root@controller:~#

```

接下来，创建`neutron`用户并将其添加到 Keystone 中的`admin`角色：

```
root@controller:~# openstack user create --domain default --password-prompt neutron 
User Password: 
Repeat User Password: 
+---------------------+----------------------------------+ 
| Field | Value | 
+---------------------+----------------------------------+ 
| domain_id | default | 
| enabled | True | 
| id | 68867f6864574592b1a29ec293defb5d | 
| name | neutron | 
| password_expires_at | None | 
+---------------------+----------------------------------+ 
root@controller:~# openstack role add --project service --user neutron admin 
root@controller:~#

```

创建`neutron`服务和端点：

```
root@controller:~# openstack service create --name neutron --description "OpenStack Networking" network 
+-------------+----------------------------------+ 
| Field | Value | 
+-------------+----------------------------------+ 
| description | OpenStack Networking | 
| enabled | True | 
| id | 8bd98d58bfb5410694cbf7b6163a71a5 | 
| name | neutron | 
| type | network | 
+-------------+----------------------------------+ 
root@controller:~# openstack endpoint create --region RegionOne network public http://controller:9696 
+--------------+----------------------------------+ 
| Field | Value | 
+--------------+----------------------------------+ 
| enabled | True | 
| id | 4e2c1a8689b146a7b3b4207c63a778da | 
| interface | public | 
| region | RegionOne | 
| region_id | RegionOne | 
| service_id | 8bd98d58bfb5410694cbf7b6163a71a5 | 
| service_name | neutron | 
| service_type | network | 
| url | http://controller:9696 | 
+--------------+----------------------------------+ 
root@controller:~# openstack endpoint create --region RegionOne network internal http://controller:9696 
... 
root@controller:~# openstack endpoint create --region RegionOne network admin http://controller:9696 
... 
root@controller:~#

```

在身份服务中定义了所有服务和端点后，安装以下软件包：

```
root@controller:~# apt install -y neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent

```

我们之前安装的 Neutron 包提供以下服务：

+   `neutron-server`：该包提供 API，用于动态请求和配置虚拟网络

+   `neutron-plugin-ml2`：这是一个框架，允许使用各种网络技术，如 Linux 桥接、Open vSwitch、GRE 和 VXLAN，这些我们在前面的章节中已看到

+   `neutron-linuxbridge-agent`：该服务提供 Linux 桥接插件代理

+   `neutron-l3-agent`：通过创建虚拟路由器，该服务在软件定义的网络之间执行转发和 NAT 功能

+   `neutron-dhcp-agent`：该服务控制分配 IP 地址给运行在计算节点上的实例的 DHCP 服务

+   `neutron-metadata-agent`：这是一个将实例元数据传递给 Neutron 的服务

以下是 Neutron 的最小工作配置文件：

```
root@controller:~# cat /etc/neutron/neutron.conf 
[DEFAULT] 
core_plugin = ml2 
service_plugins = router 
allow_overlapping_ips = True 
transport_url = rabbit://openstack:lxcpassword@controller 
auth_strategy = keystone 
notify_nova_on_port_status_changes = True 
notify_nova_on_port_data_changes = True 
[agent] 
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf 
[cors] 
[cors.subdomain] 
[database] 
connection = mysql+pymysql://neutron:lxcpassword@controller/neutron 
[keystone_authtoken] 
auth_uri = http://controller:5000 
auth_url = http://controller:35357 
memcached_servers = controller:11211 
auth_type = password 
project_domain_name = default 
user_domain_name = default 
project_name = service 
username = neutron 
password = lxcpassword 
[matchmaker_redis] 
[nova] 
auth_url = http://controller:35357 
auth_type = password 
project_domain_name = default 
user_domain_name = default 
region_name = RegionOne 
project_name = service 
username = nova 
password = lxcpassword 
[oslo_concurrency] 
[oslo_messaging_amqp] 
[oslo_messaging_notifications] 
[oslo_messaging_rabbit] 
[oslo_messaging_zmq] 
[oslo_policy] 
[qos] 
[quotas] 
[ssl] 
root@controller:~#

```

我们需要定义将要支持的网络扩展和网络类型。所有这些信息将在创建 LXC 容器及其配置文件时使用，稍后我们将看到：

```
root@controller:~# cat /etc/neutron/plugins/ml2/ml2_conf.ini 
[DEFAULT] 
[ml2] 
type_drivers = flat,vlan,vxlan 
tenant_network_types = vxlan 
mechanism_drivers = linuxbridge,l2population 
extension_drivers = port_security 
[ml2_type_flat] 
flat_networks = provider 
[ml2_type_geneve] 
[ml2_type_gre] 
[ml2_type_vlan] 
[ml2_type_vxlan] 
vni_ranges = 1:1000 
[securitygroup] 
enable_ipset = True 
root@controller:~#

```

定义将添加到软件桥接中的接口，以及该桥接将绑定的 IP 地址。在本例中，我们使用 `eth1` 接口及其 IP 地址：

```
root@controller:~# cat /etc/neutron/plugins/ml2/linuxbridge_agent.ini 
[DEFAULT] 
[agent] 
[linux_bridge] 
physical_interface_mappings = provider:eth1 
[securitygroup] 
enable_security_group = True 
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver 
[vxlan] 
enable_vxlan = True 
local_ip = 10.208.132.45 
l2_population = True 
root@controller:~#

```

我们为 L3 代理指定桥接驱动程序如下：

```
root@controller:~# cat /etc/neutron/l3_agent.ini 
[DEFAULT] 
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver 
[AGENT] 
root@controller:~#

```

DHCP 代理的配置文件应类似于以下内容：

```
root@controller:~# cat /etc/neutron/dhcp_agent.ini 
[DEFAULT] 
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver 
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq 
enable_isolated_metadata = True 
[AGENT] 
root@controller:~#

```

最后，元数据代理的配置如下：

```
root@controller:~# cat /etc/neutron/metadata_agent.ini 
[DEFAULT] 
nova_metadata_ip = controller 
metadata_proxy_shared_secret = lxcpassword 
[AGENT] 
[cache] 
root@controller:~#

```

我们需要更新 Nova 服务的配置文件。新的完整文件应如下所示；根据需要替换 IP 地址：

```
root@controller:~# cat /etc/nova/nova.conf 
[DEFAULT] 
dhcpbridge_flagfile=/etc/nova/nova.conf 
dhcpbridge=/usr/bin/nova-dhcpbridge 
log-dir=/var/log/nova 
state_path=/var/lib/nova 
force_dhcp_release=True 
verbose=True 
ec2_private_dns_show_ip=True 
enabled_apis=osapi_compute,metadata 
transport_url = rabbit://openstack:lxcpassword@controller 
auth_strategy = keystone 
my_ip = 10.208.132.45 
use_neutron = True 
firewall_driver = nova.virt.firewall.NoopFirewallDriver 
compute_driver = libvirt.LibvirtDriver 
scheduler_default_filters = RetryFilter, AvailabilityZoneFilter, RamFilter, ComputeFilter, ComputeCapabilitiesFilter, ImagePropertiesFilter, ServerGroupAntiAffinityFilter, ServerGroupAffinityFilter 
[database] 
connection = mysql+pymysql://nova:lxcpassword@controller/nova 
[api_database] 
connection = mysql+pymysql://nova:lxcpassword@controller/nova_api 
[oslo_concurrency] 
lock_path = /var/lib/nova/tmp 
[libvirt] 
use_virtio_for_bridges=True 
[wsgi] 
api_paste_config=/etc/nova/api-paste.ini 
[keystone_authtoken] 
auth_uri = http://controller:5000 
auth_url = http://controller:35357 
memcached_servers = controller:11211 
auth_type = password 
project_domain_name = default 
user_domain_name = default 
project_name = service 
username = nova 
password = lxcpassword 
[vnc] 
enabled = True 
vncserver_listen = $my_ip 
vncserver_proxyclient_address = $my_ip 
novncproxy_base_url = http://controller:6080/vnc_auto.html 
[glance] 
api_servers = http://controller:9292 
[libvirt] 
virt_type = lxc 
[neutron] 
url = http://controller:9696 
auth_url = http://controller:35357 
auth_type = password 
project_domain_name = default 
user_domain_name = default 
region_name = RegionOne 
project_name = service 
username = neutron 
password = lxcpassword 
service_metadata_proxy = True 
metadata_proxy_shared_secret = lxcpassword 
root@controller:~#

```

填充 `neutron` 数据库：

```
root@controller:~# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron 
INFO [alembic.runtime.migration] Context impl MySQLImpl. 
INFO [alembic.runtime.migration] Will assume non-transactional DDL. 
Running upgrade for neutron ... 
INFO [alembic.runtime.migration] Context impl MySQLImpl. 
INFO [alembic.runtime.migration] Will assume non-transactional DDL. 
... 
OK 
root@controller:~#

```

最后，启动所有网络服务并重新启动 `nova-compute`：

```
root@controller:~# service nova-api restart 
root@controller:~# service neutron-server restart 
root@controller:~# service neutron-linuxbridge-agent restart 
root@controller:~# service neutron-dhcp-agent restart 
root@controller:~# service neutron-metadata-agent restart 
root@controller:~# service neutron-l3-agent restart 
root@controller:~# service nova-compute restart

```

让我们验证 Neutron 服务是否正在运行：

```
root@controller:~# openstack network agent list 
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+ 
| ID | Agent Type | Host | Availability Zone | Alive | State | Binary | 
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+ 
| 2a715a0d-5593-4aba-966e-6ae3b2e02ba2 | L3 agent | controller | nova | True | UP | neutron-l3-agent | 
| 2ce176fb-dc2e-4416-bb47-1ae44e1f556f | Linux bridge agent | controller | None | True | UP | neutron-linuxbridge-agent | 
| 42067496-eaa3-42ef-bff9-bbbbcbf2e15a | DHCP agent | controller | nova | True | UP | neutron-dhcp-agent | 
| fad2b9bb-8ee7-468e-b69a-43129338cbaa | Metadata agent | controller | None | True | UP | neutron-metadata-agent | 
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+ 
root@controller:~#

```

## 定义 LXC 实例规格，生成密钥对，并创建安全组

在我们创建 LXC 实例之前，需要定义其规格 - CPU、内存和磁盘大小。以下命令创建一个名为 `lxc.medium` 的规格，包含一个虚拟 CPU、1 GB 内存和 5 GB 磁盘：

```
root@controller:~# openstack flavor create --id 0 --vcpus 1 --ram 1024 --disk 5 lxc.medium 
+----------------------------+------------+ 
| Field | Value | 
+----------------------------+------------+ 
| OS-FLV-DISABLED:disabled | False | 
| OS-FLV-EXT-DATA:ephemeral | 0 | 
| disk | 5 | 
| id | 0 | 
| name | lxc.medium | 
| os-flavor-access:is_public | True | 
| properties | | 
| ram | 1024 | 
| rxtx_factor | 1.0 | 
| swap | | 
| vcpus | 1 | 
+----------------------------+------------+ 
root@controller:~#

```

为了能够通过 SSH 访问 LXC 容器，我们可以在实例配置期间管理和安装 SSH 密钥，如果我们不希望它们被嵌入到实际镜像中。要生成 SSH 密钥对并将其添加到 OpenStack，请运行以下命令：

```
root@controller:~# ssh-keygen -q -N ""

```

输入要保存密钥的文件（`/root/.ssh/id_rsa`）：

```
root@controller:~# openstack keypair create --public-key ~/.ssh/id_rsa.pub lxckey 
+-------------+-------------------------------------------------+ 
| Field | Value | 
+-------------+-------------------------------------------------+ 
| fingerprint | 84:36:93:cc:2f:f0:f7:ba:d5:73:54:ca:2e:f0:02:6d | 
| name | lxckey | 
| user_id | 3a04d141c07541478ede7ea34f3e5c36 | 
+-------------+-------------------------------------------------+ 
root@controller:~#

```

要列出我们刚刚添加的新密钥对，请执行以下命令：

```
root@controller:~# openstack keypair list 
+--------+-------------------------------------------------+ 
| Name | Fingerprint | 
+--------+-------------------------------------------------+ 
| lxckey | 84:36:93:cc:2f:f0:f7:ba:d5:73:54:ca:2e:f0:02:6d | 
+--------+-------------------------------------------------+ 
root@controller:~#

```

默认情况下，一旦新的 LXC 容器被配置，iptables 会禁止访问它。我们将创建两个安全组，允许 ICMP 和 SSH，以便我们可以测试连接并连接到实例：

```
root@controller:~# openstack security group rule create --proto icmp default 
+-------------------+--------------------------------------+ 
| Field | Value | 
+-------------------+--------------------------------------+ 
| created_at | 2016-12-02T20:30:14Z | 
| description | | 
| direction | ingress | 
| ethertype | IPv4 | 
| headers | | 
| id | 0e17e0ab-4495-440a-b8b9-0a612f9eccae | 
| port_range_max | None | 
| port_range_min | None | 
| project_id | 488aecf07dcb4ae6bc1ebad5b76fbc20 | 
| project_id | 488aecf07dcb4ae6bc1ebad5b76fbc20 | 
| protocol | icmp | 
| remote_group_id | None | 
| remote_ip_prefix | 0.0.0.0/0 | 
| revision_number | 1 | 
| security_group_id | f21f3d3c-27fe-4668-bca4-6fc842dcb690 | 
| updated_at | 2016-12-02T20:30:14Z | 
+-------------------+--------------------------------------+ 
root@controller:~# 
root@controller:~# openstack security group rule create --proto tcp --dst-port 22 default 
... 
root@controller:~#

```

## 创建网络

让我们首先在 Neutron 中创建一个名为`nat`的新网络：

```
root@controller:~# openstack network create nat 
+---------------------------+--------------------------------------+ 
| Field | Value | 
+---------------------------+--------------------------------------+ 
| admin_state_up | UP | 
| availability_zone_hints | | 
| availability_zones | | 
| created_at | 2016-12-02T20:32:53Z | 
| description | | 
| headers | | 
| id | 66037974-d24b-4615-8b93-b0de18a4561b | 
| ipv4_address_scope | None | 
| ipv6_address_scope | None | 
| mtu | 1450 | 
| name | nat | 
| port_security_enabled | True | 
| project_id | 488aecf07dcb4ae6bc1ebad5b76fbc20 | 
| project_id | 488aecf07dcb4ae6bc1ebad5b76fbc20 | 
| provider:network_type | vxlan | 
| provider:physical_network | None | 
| provider:segmentation_id | 53 | 
| revision_number | 3 | 
| router:external | Internal | 
| shared | False | 
| status | ACTIVE | 
| subnets | | 
| tags | [] | 
| updated_at | 2016-12-02T20:32:53Z | 
+---------------------------+--------------------------------------+ 
root@controller:~#

```

接下来，定义将分配给 LXC 容器的 DNS 服务器、默认网关和子网范围：

```
root@controller:~# openstack subnet create --network nat --dns-nameserver 8.8.8.8 --gateway 192.168.0.1 --subnet-range 192.168.0.0/24 nat 
+-------------------+--------------------------------------+ 
| Field | Value | 
+-------------------+--------------------------------------+ 
| allocation_pools | 192.168.0.2-192.168.0.254 | 
| cidr | 192.168.0.0/24 | 
| created_at | 2016-12-02T20:36:14Z | 
| description | | 
| dns_nameservers | 8.8.8.8 | 
| enable_dhcp | True | 
| gateway_ip | 192.168.0.1 | 
| headers | | 
| host_routes | | 
| id | 0e65fa94-be69-4690-b3fe-406ea321dfb3 | 
| ip_version | 4 | 
| ipv6_address_mode | None | 
| ipv6_ra_mode | None | 
| name | nat | 
| network_id | 66037974-d24b-4615-8b93-b0de18a4561b | 
| project_id | 488aecf07dcb4ae6bc1ebad5b76fbc20 | 
| project_id | 488aecf07dcb4ae6bc1ebad5b76fbc20 | 
| revision_number | 2 | 
| service_types | [] | 
| subnetpool_id | None | 
| updated_at | 2016-12-02T20:36:14Z | 
+-------------------+--------------------------------------+ 
root@controller:~#

```

更新 Neutron 中子网的信息：

```
root@controller:~# neutron net-update nat --router:external 
Updated network: nat 
root@controller:~# 

```

作为 `lxc` 用户，创建一个新的软件路由器：

```
root@controller:~# . rc.lxc 
root@controller:~# openstack router create router 
+-------------------------+--------------------------------------+ 
| Field | Value | 
+-------------------------+--------------------------------------+ 
| admin_state_up | UP | 
| availability_zone_hints | | 
| availability_zones | | 
| created_at | 2016-12-02T20:38:08Z | 
| description | | 
| external_gateway_info | null | 
| flavor_id | None | 
| headers | | 
| id | 45557fac-f158-40ef-aeec-496de913d5a5 | 
| name | router | 
| project_id | b0cc5ccc12eb4d6b98aadd784540f575 | 
| project_id | b0cc5ccc12eb4d6b98aadd784540f575 | 
| revision_number | 2 | 
| routes | | 
| status | ACTIVE | 
| updated_at | 2016-12-02T20:38:08Z | 
+-------------------------+--------------------------------------+ 
root@controller:~#

```

作为管理员用户，将我们之前创建的子网添加为路由器的接口：

```
root@controller:~# . rc.admin 
root@controller:~# neutron router-interface-add router nat 
Added interface be0f1e65-f086-41fd-b8d0-45ebb865bf0f to router router. 
root@controller:~#

```

让我们列出创建的网络命名空间：

```
root@controller:~# ip netns 
qrouter-45557fac-f158-40ef-aeec-496de913d5a5 (id: 1) 
qdhcp-66037974-d24b-4615-8b93-b0de18a4561b (id: 0) 
root@controller:~#

```

要显示软件路由器上的端口以及 LXC 容器的默认网关，请运行以下命令：

```
root@controller:~# neutron router-port-list router 
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------------+ 
| id | name | mac_address | fixed_ips | 
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------------+ 
| be0f1e65-f086-41fd-b8d0-45ebb865bf0f | | fa:16:3e:a6:36:7c | {"subnet_id": "0e65fa94-be69-4690-b3fe-406ea321dfb3", "ip_address": "192.168.0.1"} | 
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------------+ 
root@controller:~#

```

## 使用 OpenStack 提供 LXC 容器

在启动我们的 LXC 容器之前，让我们再三检查一下是否满足所有要求。

首先列出可用的网络：

```
root@controller:~# openstack network list 
+--------------------------------------+------+--------------------------------------+ 
| ID | Name | Subnets | 
+--------------------------------------+------+--------------------------------------+ 
| 66037974-d24b-4615-8b93-b0de18a4561b | nat | 0e65fa94-be69-4690-b3fe-406ea321dfb3 | 
+--------------------------------------+------+--------------------------------------+ 
root@controller:~#

```

显示我们可以选择的计算实例规格：

```
root@controller:~# openstack flavor list 
+----+------------+------+------+-----------+-------+-----------+ 
| ID | Name | RAM | Disk | Ephemeral | VCPUs | Is Public | 
+----+------------+------+------+-----------+-------+-----------+ 
| 0 | lxc.medium | 1024 | 5 | 0 | 1 | True | 
+----+------------+------+------+-----------+-------+-----------+ 
root@controller:~#

```

接下来，列出可用的镜像：

```
root@controller:~# openstack image list 
+--------------------------------------+------------------+--------+ 
| ID | Name | Status | 
+--------------------------------------+------------------+--------+ 
| 417e72b5-7b85-4555-835d-ce442e21aa4f | lxc_ubuntu_12.04 | active | 
+--------------------------------------+------------------+--------+ 
root@controller:~#

```

同时，显示我们之前创建的默认安全组：

```
root@controller:~# openstack security group list 
+--------------------------------------+---------+------------------------+----------------------------------+ 
| ID | Name | Description | Project | 
+--------------------------------------+---------+------------------------+----------------------------------+ 
| f21f3d3c-27fe-4668-bca4-6fc842dcb690 | default | Default security group | 488aecf07dcb4ae6bc1ebad5b76fbc20 | 
+--------------------------------------+---------+------------------------+----------------------------------+ 
root@controller:~#

```

现在加载 **网络块设备**（**NBD**）内核模块，因为 Nova 期望它：

```
root@controller:~# modprobe nbd

```

最后，要使用 OpenStack 提供 LXC 容器，请执行以下命令：

```
root@controller:~# openstack server create --flavor lxc.medium --image lxc_ubuntu_12.04 --nic net-id=66037974-d24b-4615-8b93-b0de18a4561b --security-group default --key-name lxckey lxc_instance 
... 
root@controller:~#

```

注意我们如何指定实例规格、镜像名称、网络 ID、安全组、密钥对名称和实例名称。

确保用系统返回的输出替换这些 ID。

要列出 LXC 容器、其状态以及分配的 IP 地址，请运行以下命令：

```
root@controller:~# openstack server list 
+--------------------------------------+--------------+--------+-----------------+------------------+ 
| ID | Name | Status | Networks | Image Name | 
+--------------------------------------+--------------+--------+-----------------+------------------+ 
| a86f8f56-80d7-4d36-86c3-827679f21ec5 | lxc_instance | ACTIVE | nat=192.168.0.3 | lxc_ubuntu_12.04 | 
+--------------------------------------+--------------+--------+-----------------+------------------+ 
root@controller:~#

```

正如我们在本章前面看到的，OpenStack 使用 libvirt 驱动程序来提供 LXC 容器。我们可以使用在第二章*，安装和运行 LXC 在 Linux 系统上* 中使用过的 `virsh` 命令，列出主机上的 LXC 容器：

```
root@controller:~# virsh --connect lxc:// list --all 
Id Name State 
---------------------------------------------------- 
16225 instance-00000002 running 
root@controller:~#

```

如果我们列出主机上的进程，可以看到 `libvirt_lxc` 父进程启动了容器的初始化进程：

```
root@controller:~# ps axfw 
... 
16225 ? S 0:00 /usr/lib/libvirt/libvirt_lxc --name instance-00000002 --console 23 --security=apparmor --handshake 26 --veth vnet0 
16227 ? Ss 0:00 \_ /sbin/init 
16591 ? S 0:00 \_ upstart-socket-bridge --daemon 
16744 ? Ss 0:00 \_ dhclient3 -e IF_METRIC=100 -pf /var/run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases -1 eth0 
17692 ? S 0:00 \_ upstart-udev-bridge --daemon 
17696 ? Ss 0:00 \_ /sbin/udevd --daemon 
17819 ? S 0:00 | \_ /sbin/udevd --daemon 
18108 ? Ss 0:00 \_ /usr/sbin/sshd -D 
18116 ? Ss 0:00 \_ dbus-daemon --system --fork --activation=upstart 
18183 ? Ss 0:00 \_ cron 
18184 ? Ss 0:00 \_ atd 
18189 ? Ss 0:00 \_ /usr/sbin/irqbalance 
18193 ? Ss 0:00 \_ acpid -c /etc/acpi/events -s /var/run/acpid.socket 
18229 pts/0 Ss+ 0:00 \_ /sbin/getty -8 38400 tty1 
18230 ? Ssl 0:00 \_ whoopsie 
18317 ? Sl 0:00 \_ rsyslogd -c5 
19185 ? Ss 0:00 \_ /sbin/getty -8 38400 tty4 
19186 ? Ss 0:00 \_ /sbin/getty -8 38400 tty2 
19187 ? Ss 0:00 \_ /sbin/getty -8 38400 tty3 
root@controller:~#

```

容器的配置文件和磁盘的位置如下：

```
root@controller:~# cd /var/lib/nova/instances/ 
root@controller:/var/lib/nova/instances# ls -la a86f8f56-80d7-4d36-86c3-827679f21ec5/ 
total 12712 
drwxr-xr-x 3 nova nova 4096 Dec 2 20:52 . 
drwxr-xr-x 5 nova nova 4096 Dec 2 20:52 .. 
-rw-r--r-- 1 nova nova 0 Dec 2 20:52 console.log 
-rw-r--r-- 1 nova nova 13041664 Dec 2 20:57 disk 
-rw-r--r-- 1 nova nova 79 Dec 2 20:52 disk.info 
-rw-r--r-- 1 nova nova 1534 Dec 2 20:52 libvirt.xml 
drwxr-xr-x 2 nova nova 4096 Dec 2 20:52 rootfs 
root@controller:/var/lib/nova/instances#

```

让我们检查一下容器的配置文件：

```
root@controller:/var/lib/nova/instances# cat a86f8f56-80d7-4d36-86c3-827679f21ec5/libvirt.xml 
<domain type="lxc"> 
 <uuid>a86f8f56-80d7-4d36-86c3-827679f21ec5</uuid> 
 <name>instance-00000002</name> 
 <memory>1048576</memory> 
 <vcpu>1</vcpu> 
 <metadata> 
 <nova:instance 
      > 
 <nova:package version="14.0.1"/> 
 <nova:name>lxc_instance</nova:name> 
 <nova:creationTime>2016-12-02 20:52:52</nova:creationTime> 
 <nova:flavor name="lxc.medium"> 
 <nova:memory>1024</nova:memory> 
 <nova:disk>5</nova:disk> 
 <nova:swap>0</nova:swap> 
 <nova:ephemeral>0</nova:ephemeral> 
 <nova:vcpus>1</nova:vcpus> 
 </nova:flavor> 
 <nova:owner> 
 <nova:user uuid="3a04d141c07541478ede7ea34f3e5c36">
          admin
        </nova:user> 
 <nova:project uuid="488aecf07dcb4ae6bc1ebad5b76fbc20">
          admin
        </nova:project> 
 </nova:owner> 
 <nova:root type="image" uuid="417e72b5-7b85-4555-835d-ce442e21aa4f"/> 
 </nova:instance> 
 </metadata> 
 <os> 
 <type>exe</type> 
 <cmdline>console=tty0 console=ttyS0 console=ttyAMA0</cmdline> 
 <init>/sbin/init</init> 
 </os> 
 <cputune> 
 <shares>1024</shares> 
 </cputune> 
 <clock offset="utc"/> 
 <devices> 
 <filesystem type="mount"> 
 <source dir="/var/lib/nova/instances/
      a86f8f56-80d7-4d36-86c3-827679f21ec5/rootfs"/> 
 <target dir="/"/> 
 </filesystem> 
 <interface type="bridge"> 
 <mac address="fa:16:3e:4f:e5:b5"/> 
 <source bridge="brq66037974-d2"/> 
 <target dev="tapf2073410-64"/> 
 </interface> 
 <console type="pty"/> 
 </devices> 
</domain> 
root@controller:/var/lib/nova/instances#

```

我们已经看过之前章节中为 libvirt 容器构建的类似配置文件。

在 Neutron 管理的网络环境下，我们应该能够看到桥接和容器的接口作为端口被添加：

```
root@controller:/var/lib/nova/instances# brctl show 
bridge name      bridge id         STP enabled      interfaces 
brq66037974-d2   8000.02d65d01c617  no              tap4e3afc26-88 
 tapbe0f1e65-f0 
 tapf2073410-64 
 vxlan-53 
virbr0           8000.5254004e7712  yes             virbr0-nic 
root@controller:/var/lib/nova/instances#

```

让我们在桥接接口上配置一个 IP 地址，并允许容器使用 NAT 连接：

```
root@controller:/var/lib/nova/instances# ifconfig brq66037974-d2 192.168.0.1 
root@controller:~# iptables -A POSTROUTING -t nat -s 192.168.0.0/24 ! -d 192.168.0.0/24 -j MASQUERADE 
root@controller:~#

```

要使用 SSH 和我们之前生成的密钥对连接到 LXC 容器，请执行以下命令：

```
root@controller:/var/lib/nova/instances# ssh ubuntu@192.168.0.3
Welcome to Ubuntu 12.04.5 LTS (GNU/Linux 4.4.0-51-generic x86_64)
ubuntu@lxc-instance:~$ ifconfig
eth0 Link encap:Ethernet HWaddr fa:16:3e:4f:e5:b5
inet addr:192.168.0.3 Bcast:192.168.0.255 Mask:255.255.255.0
inet6 addr: fe80::f816:3eff:fe4f:e5b5/64 Scope:Link
UP BROADCAST RUNNING MULTICAST MTU:1450 Metric:1
RX packets:290 errors:0 dropped:0 overruns:0 frame:0
TX packets:340 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:35038 (35.0 KB) TX bytes:36830 (36.8 KB)
lo Link encap:Local Loopback
inet addr:127.0.0.1 Mask:255.0.0.0
inet6 addr: ::1/128 Scope:Host
UP LOOPBACK RUNNING MTU:65536 Metric:1
RX packets:0 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1
RX bytes:0 (0.0 B) TX bytes:0 (0.0 B)
ubuntu@lxc-instance:~$ exit
logout
Connection to 192.168.0.3 closed.
root@controller:/var/lib/nova/instances# cd
root@controller:~#

```

最后，要使用 OpenStack 删除 LXC 容器，请运行以下命令：

```
root@controller:~# openstack server delete lxc_instance 
root@controller:~# openstack server list 
root@controller:~#

```

# 摘要

在本章中，我们查看了一个基本的 OpenStack 部署示例，使用身份服务 Keystone 存储服务目录，以及进行身份验证和授权，使用 Nova 计算服务为 LXC 实例提供服务，使用 Glance 镜像服务存储 LXC 容器镜像，并通过 Neutron 网络服务创建桥接并为我们的容器分配 IP 地址。

完整的生产就绪部署将包括多个控制节点，这些节点运行前述服务，并配备一组计算服务器来提供容器。

使用 OpenStack 和 LXC 是创建和管理多租户云环境的好方法，可以以集中式和高度可扩展的方式运行各种软件应用。

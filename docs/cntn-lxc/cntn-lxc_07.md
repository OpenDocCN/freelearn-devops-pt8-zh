# 第七章。容器化世界中的监控与备份

在上一章中，我们看了一些如何通过创建多个实例并放置在代理服务（如 HAProxy）后面，来扩展在 LXC 容器中运行的应用程序的示例。这确保了应用程序有足够的资源并能承受故障，从而实现一定程度的高可用性。

对于运行在单个 LXC 实例中的应用程序，通常希望定期备份容器，包括根文件系统和容器的配置文件。根据后台存储的不同，有一些可用的选项，我们将在本章中探讨。

使用高度可用或共享的后台存储可以帮助快速从失败的主机恢复，或者在需要在服务器之间迁移 LXC 容器时提供帮助。我们将看看如何在**互联网小型计算机系统接口**（**iSCSI**）目标上创建容器，并在服务器之间迁移 LXC。我们还将给出一个示例，展示如何使用 GlusterFS 作为共享文件系统来托管容器的根文件系统，从而创建活动-被动 LXC 部署。

我们还将介绍如何使用各种工具（如 Monit 和 Sensu）监控 LXC 容器及其内部运行的服务；我们将在本章结束时给出一个基于监控触发器创建简单自动扩展解决方案的示例。

在本章中，我们将探讨以下主题：

+   使用 `tar` 和 `rsync` 备份 LXC 容器

+   使用 `lxc-copy` 工具备份 LXC 容器

+   使用 iSCSI 目标作为后台存储创建 LXC 容器，并演示当主服务器故障时如何迁移容器

+   演示如何使用 GlusterFS 作为容器根文件系统的共享文件系统，并部署活动-被动 LXC 节点

+   了解 LXC 暴露了哪些度量指标，以及如何监控、警报和处理这些指标

# 备份和迁移 LXC

创建 LXC 实例的备份可以确保我们在服务器崩溃或后台存储损坏等事件发生时能够恢复数据。备份还为在主机之间迁移实例或通过更改配置文件快速启动多个相似容器提供了一种便捷的方式。

## 使用 tar 和 rsync 创建 LXC 备份

在大多数使用场景中，我们从模板构建容器，或使用如 debootstrap 等工具，这些工具会为实例创建完整的根文件系统。在这种情况下，创建备份的方法是停止容器，归档其配置文件以及实际的根文件系统，然后将它们存储在远程服务器上。让我们通过一个简单的示例来演示这个概念：

1.  首先更新你的 Ubuntu 发行版并安装 LXC。在本示例中，我们将使用 Ubuntu 16.04，但该说明适用于任何其他 Linux 发行版：

    ```
     root@ubuntu:~# apt-get update && apt-get -y upgrade && reboot 
     root@ubuntu:~# lsb_release -cd 
     Description: Ubuntu 16.04.1 LTS 
     Codename: xenial 
     root@ubuntu:~# 
     root@ubuntu:~# apt-get install -y lxc

    ```

1.  接下来，使用默认目录后台存储创建一个容器，并启动它：

    ```
     root@ubuntu:~# lxc-create --name dir_container --template ubuntu 
     root@ubuntu:~# lxc-start --name dir_container

    ```

1.  在 LXC 内安装一个应用程序，本例中是 Nginx，并创建一个自定义的索引文件：

    ```
     root@ubuntu:~# lxc-attach --name dir_container 
     root@dir_container:~# apt-get install -y nginx 
     root@dir_container:~# echo "Original container" > 
          /var/www/html/index.nginx-debian.html 
     root@dir_container:~# exit 
     exit 
     root@ubuntu:~#

    ```

1.  在容器运行时，确保我们能够访问 HTTP 服务：

    ```
     root@ubuntu:~# lxc-ls -f 
     NAME STATE AUTOSTART GROUPS IPV4 IPV6 
     dir_container RUNNING 0 - 10.0.3.107 - 
     root@ubuntu:~# curl 10.0.3.107 
     Original container 
     root@ubuntu:~#

    ```

1.  请注意，容器的文件系统及其配置文件都自包含在 `/var/lib/lxc` 目录中：

    ```
     root@ubuntu:~# cd /var/lib/lxc 
     root@ubuntu:/var/lib/lxc# ls -lah 
     total 12K 
     drwx------ 3 root root 4.0K Nov 15 16:28 . 
     drwxr-xr-x 42 root root 4.0K Nov 15 16:17 .. 
     drwxrwx--- 3 root root 4.0K Nov 15 16:33 dir_container 
     root@ubuntu:/var/lib/lxc# ls -la dir_container/ 
     total 16 
     drwxrwx--- 3 root root 4096 Nov 15 16:33 . 
     drwx------ 3 root root 4096 Nov 15 16:28 .. 
     -rw-r--r-- 1 root root 712 Nov 15 16:33 config 
     drwxr-xr-x 21 root root 4096 Nov 15 16:38 rootfs 
     root@ubuntu:/var/lib/lxc#

    ```

1.  使用标准 Linux 工具（如 `tar` 和 `rsync`）创建备份非常方便。在备份之前，请先停止容器：

    ```
     root@ubuntu:/var/lib/lxc# lxc-stop --name dir_container 
     root@ubuntu:/var/lib/lxc# lxc-ls -f 
     NAME STATE AUTOSTART GROUPS IPV4 IPV6 
     dir_container STOPPED 0 - - - 
     root@ubuntu:/var/lib/lxc#

    ```

1.  创建根文件系统和配置文件的 `bzip` 归档文件，确保保留数字所有者 ID，这样我们就可以在不同的服务器上启动它而不会发生用户 ID 冲突：

    ```
     root@ubuntu:/var/lib/lxc# tar --numeric-owner -jcvf 
          dir_container.tar.bz2 dir_container/ 
     ... 
     root@ubuntu:/var/lib/lxc# file dir_container.tar.bz2 
     dir_container.tar.bz2: bzip2 compressed data, block size = 900k 
     root@ubuntu:/var/lib/lxc#

    ```

1.  接下来，将归档文件复制到另一台服务器，在本例中是 `ubuntu-backup`，并从原始服务器删除该归档文件：

    ```
     root@ubuntu:/var/lib/lxc# rsync -vaz dir_container.tar.bz2 
          ubuntu-backup:/tmp 
     sending incremental file list 
     dir_container.tar.bz2 
     sent 148,846,592 bytes received 35 bytes 9,603,008.19 bytes/sec 
     total size is 149,719,493 speedup is 1.01 
     root@ubuntu:/var/lib/lxc# rm -f dir_container.tar.bz2

    ```

在目标服务器上有了归档文件后，我们现在可以在需要时恢复它。

## 从归档备份恢复

要从 `bz2` 备份中恢复容器，在目标服务器上，提取 `/var/lib/lxc` 目录中的归档文件：

```
root@ubuntu-backup:~# cd /var/lib/lxc 
root@ubuntu-backup:/var/lib/lxc# ls -la 
total 8 
drwx------ 2 root root 4096 Oct 21 16:56 . 
drwxr-xr-x 42 root root 4096 Nov 15 16:48 .. 
root@ubuntu-backup:/var/lib/lxc# 
root@ubuntu-backup:/var/lib/lxc# tar jxfv /tmp/dir_container.tar.bz2 
... 
root@ubuntu-backup:/var/lib/lxc# ls -la 
total 12 
drwx------ 3 root root 4096 Nov 15 16:59 . 
drwxr-xr-x 42 root root 4096 Nov 15 16:48 .. 
drwxrwx--- 3 root root 4096 Nov 15 16:33 dir_container 
root@ubuntu-backup:/var/lib/lxc#

```

注意，在提取根文件系统和配置文件后，列出所有容器将显示我们刚刚恢复的容器，尽管它处于停止状态：

```
root@ubuntu-backup:/var/lib/lxc# lxc-ls -f 
NAME STATE AUTOSTART GROUPS IPV4 IPV6 
dir_container STOPPED 0 - - - 
root@ubuntu-backup:/var/lib/lxc#

```

我们开始并访问 HTTP 端点，确保我们得到与原始 LXC 实例相同的结果：

```
root@ubuntu-backup:/var/lib/lxc# lxc-start --name dir_container 
root@ubuntu-backup:/var/lib/lxc# lxc-ls -f 
NAME STATE AUTOSTART GROUPS IPV4 IPV6 
dir_container RUNNING 0 - 10.0.3.107 - 
root@ubuntu-backup:/var/lib/lxc# curl 10.0.3.107 
Original container 
root@ubuntu-backup:/var/lib/lxc#

```

要清理，请运行以下命令：

```
root@ubuntu-backup:/var/lib/lxc# lxc-stop --name dir_container 
root@ubuntu-backup:/var/lib/lxc# lxc-destroy --name dir_container 
Destroyed container dir_container 
root@ubuntu-backup:/var/lib/lxc# ls -la 
total 8 
drwx------ 2 root root 4096 Nov 15 17:01 . 
drwxr-xr-x 42 root root 4096 Nov 15 16:48 .. 
root@ubuntu-backup:/var/lib/lxc#

```

## 使用 lxc-copy 创建容器备份

无论容器的后端存储是什么，我们都可以使用 `lxc-copy` 工具来创建 LXC 实例的完整副本。按照以下步骤创建容器备份：

1.  我们首先指定要备份的原始主机上的容器名称，以及副本的名称：

    ```
     root@ubuntu:/var/lib/lxc# lxc-copy --name dir_container --newname 
          dir_container_backup 
     root@ubuntu:/var/lib/lxc#

    ```

    该命令在主机上创建了一个新的根文件系统和配置文件：

    ```
     root@ubuntu:/var/lib/lxc# ls -la 
     total 16 
     drwx------ 4 root root 4096 Nov 15 17:07 . 
     drwxr-xr-x 42 root root 4096 Nov 15 16:17 .. 
     drwxrwx--- 3 root root 4096 Nov 15 16:33 dir_container 
     drwxrwx--- 3 root root 4096 Nov 15 17:07 dir_container_backup 
     root@ubuntu:/var/lib/lxc# lxc-ls -f 
     NAME STATE AUTOSTART GROUPS IPV4 IPV6 
     dir_container STOPPED 0 - - - 
     dir_container_backup STOPPED 0 - - - 
     root@ubuntu:/var/lib/lxc#

    ```

1.  创建完整副本时，会更新新容器的配置文件，指定新的容器名称和 `rootfs` 的位置：

    ```
     root@ubuntu:/var/lib/lxc# cat dir_container_backup/config | egrep 
          "rootfs|utsname" 
     lxc.rootfs = /var/lib/lxc/dir_container_backup/rootfs 
     lxc.rootfs.backend = dir 
     lxc.utsname = dir_container_backup 
     root@ubuntu:/var/lib/lxc#

    ```

    请注意，容器的名称和目录已从原始实例更改。现在，我们可以像上一节中展示的那样使用 `tar` 和 `rsync` 归档并远程存储。如果我们需要确保容器的名称和 `rootfs` 位置不同于原始实例，这种方法非常方便，尤其是在我们希望将备份保存在同一台服务器上，或在具有相同 LXC 名称的主机上时。

1.  最后，要清理，请执行以下命令：

    ```
     root@ubuntu:/var/lib/lxc# lxc-destroy --name dir_container_backup 
     Destroyed container dir_container_backup 
     root@ubuntu:/var/lib/lxc# lxc-destroy --name dir_container 
     Destroyed container dir_container 
     root@ubuntu:/var/lib/lxc#

    ```

## 在 iSCSI 目标上迁移 LXC 容器

将容器从一台主机迁移到另一台主机在进行服务器维护或重新分配服务器负载时非常有用。像 OpenStack 这样的云平台利用调度程序来选择 LXC 实例创建的位置，并基于某些标准（如实例密度、资源利用率等）迁移它们，正如我们在下一章中将看到的那样。不过，了解如何在需要时手动迁移主机间的实例是很有帮助的。

当我们使用默认的 `dir` 存储后端与 LXC 时，创建容器的备份或副本更加容易；然而，当使用其他类型的存储（如 LVM、Btrfs 或 ZFS）时，除非使用共享存储（如 GlusterFS 或 iSCSI 块设备），否则这并不那么简单，我们将接下来探讨这一点。

iSCSI 协议已经存在了一段时间，围绕它有许多**存储区域网络**（**SAN**）解决方案。它是通过 TCP/IP 网络提供块设备访问的绝佳方式。

### 设置 iSCSI 目标

暴露块设备的端点被称为**目标**。在 Linux 上创建 iSCSI 目标是相当简单的。我们所需要的仅仅是一个块设备。在下面的示例中，我们将使用两台服务器；一台将作为 iSCSI 目标，暴露一个块设备，另一台将是发起者服务器，它将连接到目标并将提供的块设备作为 LXC 容器的根文件系统和配置文件的存储位置。

设置 iSCSI 目标的步骤如下：

1.  让我们先在 Ubuntu 上安装必要的软件包，然后在 CentOS 上演示相同的操作：

    ```
     root@ubuntu-backup:~# apt-get install -y iscsitarget 
     root@ubuntu-backup:~#

    ```

1.  安装软件包后，启用目标功能：

    ```
     root@ubuntu-backup:~# sed -i   
          's/ISCSITARGET_ENABLE=false/ISCSITARGET_ENABLE=true/g' 
          /etc/default/iscsitarget 
     root@ubuntu-backup:~#

    ```

1.  接下来，让我们创建目标配置文件。该文件有良好的注释说明；我们将使用一些可用配置选项中的一个小子集。我们从指定一个任意标识符开始，在此示例中为 `iqn.2001-04.com.example:lxc`，用户名和密码，最重要的是，我们将暴露的块设备 `/dev/xvdb`。iSCSI 标识符的格式如下：

    ```
          iqn.yyyy-mm.naming-authority:unique name
    ```

    该标识符的描述如下：

    +   `yyyy-mm` : 这是命名机构成立的年份和月份

    +   `naming-authority`：通常是命名机构的互联网域名的反向语法，或者是服务器的域名

    +   `unique name`：这是你希望使用的任何名称

    考虑到这些，最小的工作目标配置文件如下：

    ```
     root@ubuntu-backup:~# cat /etc/iet/ietd.conf 
     Target iqn.2001-04.com.example:lxc 
     IncomingUser lxc secret 
     OutgoingUser 
     Lun 0 Path=/dev/xvdb,Type=fileio 
     Alias lxc_lun 
     root@ubuntu-backup:~#

    ```

1.  接下来，指定哪些主机或发起者可以连接到 iSCSI 目标；将 IP 地址替换为将连接到目标服务器的主机的 IP 地址：

    ```
     root@ubuntu-backup:~# cat /etc/iet/initiators.allow 
     iqn.2001-04.com.example:lxc 10.208.129.253 
     root@ubuntu-backup:~#

    ```

1.  现在，启动 iSCSI 目标服务：

    ```
     root@ubuntu-backup:~# /etc/init.d/iscsitarget start 
     [ ok ] Starting iscsitarget (via systemctl): iscsitarget.service. 
     root@ubuntu-backup:~#

    ```

1.  在 CentOS 上，过程和配置文件稍有不同。要安装软件包，请执行以下命令：

    ```
     [root@centos ~]# yum install scsi-target-utils 
     [root@centos ~]#

    ```

1.  配置文件的格式如下：

    ```
     [root@centos ~]# cat /etc/tgt/targets.conf 
     default-driver iscsi 
     <target iqn.2001-04.com.example:lxc> 
     backing-store /dev/xvdb 
     initiator-address 10.208.5.176 
     incominguser lxc secret 
     </target> 
     [root@centos ~]#

    ```

1.  要启动服务，请运行以下命令：

    ```
     [root@centos ~]# service tgtd restart

    ```

1.  让我们通过运行以下命令列出暴露的目标：

    ```
     [root@centos ~]# tgt-admin --show 
     Target 1: iqn.2001-04.com.example:lxc 
     System information: 
     Driver: iscsi 
     State: ready 
     I_T nexus information: 
     LUN information: 
     ... 
     LUN: 1 
     Type: disk 
     SCSI ID: IET 00010001 
     SCSI SN: beaf11 
     Size: 80531 MB, Block size: 512 
     Online: Yes 
     Removable media: No 
     Prevent removal: No 
     Readonly: No 
     SWP: No 
     Thin-provisioning: No 
     Backing store type: rdwr 
     Backing store path: /dev/xvdb 
     Backing store flags: 
     Account information: 
     ACL information: 
     10.208.5.176 
     [root@centos ~]#

    ```

### 设置 iSCSI 发起者

发起者是将连接到目标并通过 iSCSI 协议访问暴露的块设备的服务器。

1.  要在 Ubuntu 上安装发起者工具，请运行以下命令：

    ```
     root@ubuntu:~# apt-get install -y open-iscsi

    ```

    在 CentOS 上，软件包名称有所不同：

    ```
     [root@centos ~]# yum install iscsi-initiator-utils

    ```

1.  接下来，启用服务并启动 iSCSI 守护进程：

    ```
     root@ubuntu:~# sed -i 's/node.startup = manual/node.startup = 
          automatic/g'   /etc/iscsi/iscsid.conf 
     root@ubuntu:~# /etc/init.d/open-iscsi restart 
     [ ok ] Restarting open-iscsi (via systemctl): open-iscsi.service. 
     root@ubuntu:~#

    ```

1.  `iscsiadm` 命令是我们可以从发起服务器使用的用户空间工具，用来请求目标提供可用的设备。从启动主机上，我们向之前配置的目标服务器询问可用的块设备：

    ```
     root@ubuntu:~# iscsiadm -m discovery -t sendtargets -p 10.208.129.201 
     10.208.129.201:3260,1 iqn.2001-04.com.example:lxc 
     192.237.179.19:3260,1 iqn.2001-04.com.example:lxc 
     10.0.3.1:3260,1 iqn.2001-04.com.example:lxc 
     root@ubuntu:~#

    ```

    从上面的输出中，我们可以看到目标服务器提供了一个目标，其 **iSCSI 合格名称**（**IQN**）为 `iqn.2001-04.com.example:lxc`。在这种情况下，目标服务器有三个 IP 地址，因此输出中有三行。

1.  之前，我们配置了目标使用用户名和密码。我们需要配置启动主机，以便向目标主机提供相同的凭证来访问资源：

    ```
     root@ubuntu:~# iscsiadm -m node -T iqn.2001-04.com.example:lxc -p 
          10.208.129.201:3260 --op=update --name node.session.auth.authmethod 
          --value=CHAP 
     root@ubuntu:~# iscsiadm -m node -T iqn.2001-04.com.example:lxc -p   
          10.208.129.201:3260 --op=update --name node.session.auth.username 
          --value=lxc 
     root@ubuntu:~# iscsiadm -m node -T iqn.2001-04.com.example:lxc -p
          10.208.129.201:3260 --op=update --name node.session.auth.password 
          --value=secret 
     root@ubuntu:~#

    ```

1.  更新启动器配置后，我们可以通过检查自动创建的配置文件来确认凭证是否已应用：

    ```
     root@ubuntu:~# cat 
          /etc/iscsi/nodes/iqn.2001-04.com.example\:lxc/10.208.129.201\,3260\,1
          /default | grep auth 
     node.session.auth.authmethod = CHAP 
     node.session.auth.username = lxc 
     node.session.auth.password = secret 
     node.conn[0].timeo.auth_timeout = 45 
     root@ubuntu:~#

    ```

### 使用提供的块设备登录到 iSCSI 目标，并将其作为 LXC 的 rootfs

配置好启动主机后，我们现在可以登录到 iSCSI 目标并使用提供的块设备：

1.  要登录，从启动主机运行以下命令：

    ```
     root@ubuntu:~# iscsiadm -m node -T iqn.2001-04.com.example:lxc -p  
          10.208.129.201:3260 --login 
     Logging in to [iface: default, target: iqn.2001-04.com.example:lxc, 
          portal:10.208.129.201,3260] (multiple) 
     Login to [iface: default, target: iqn.2001-04.com.example:lxc, 
          portal:10.208.129.201,3260] successful. 
     root@ubuntu:~#

    ```

1.  让我们验证启动主机现在是否包含 iSCSI 会话：

    ```
     root@ubuntu:~# iscsiadm -m session 
     tcp: [2] 10.208.129.201:3260,1 iqn.2001-04.com.example:lxc 
          (non-flash) 
     root@ubuntu:~#

    ```

1.  启动主机现在应该有一个新的块设备可以使用：

    ```
     root@ubuntu:~# ls -la /dev/disk/by-path/ 
     total 0 
     drwxr-xr-x 2 root root 60 Nov 15 20:23 . 
     drwxr-xr-x 6 root root 120 Nov 15 20:23 .. 
     lrwxrwxrwx 1 root root 9 Nov 15 20:23 
          ip-10.208.129.201:3260-iscsi-iqn.2001-04.com.example:lxc-lun-0 
          -> ../../sda 
     root@ubuntu:~#

    ```

1.  这个新的块设备可以作为常规存储设备使用。我们来在上面创建一个文件系统：

    ```
     root@ubuntu:~# mkfs.ext4 /dev/disk/by-path/ip-10.208.129.201\
          :3260-iscsi-iqn.2001-04.com.example\:lxc-lun-0 
     root@ubuntu:~#

    ```

1.  在文件系统存在的情况下，我们将块设备作为默认位置，挂载到 `/var/lib/lxc`，以便用于 LXC 文件系统：

    ```
     root@ubuntu:~# mount /dev/disk/by-path/ip-10.208.129.201\
          :3260-iscsi-iqn.2001-04.com.example\:lxc-lun-0 /var/lib/lxc 
     root@ubuntu:~# df -h | grep lxc 
     /dev/sda 74G 52M 70G 1% /var/lib/lxc 
     root@ubuntu:~#

    ```

由于我们将 iSCSI 设备挂载到 LXC 容器根文件系统的默认位置，下次我们创建容器时，其根文件系统将位于 iSCSI 设备上。正如我们稍后将看到的那样，这在启动主机需要维护或由于某种原因变得不可用时非常有用，因为我们可以将相同的 iSCSI 目标挂载到新主机上，并且只需在 LXC 端没有配置更改的情况下启动相同的容器。

### 构建 iSCSI 容器

按照惯例创建一个新的 LXC 容器，启动它，并确保它正在运行：

```
root@ubuntu:~# lxc-create --name iscsi_container --template ubuntu 
root@ubuntu:~# lxc-start --name iscsi_container 
root@ubuntu:~# lxc-ls -f 
NAME STATE AUTOSTART GROUPS IPV4 IPV6 
iscsi_container RUNNING 0 - 10.0.3.162 - 
root@ubuntu:~#

```

即使根文件系统现在位于新的 iSCSI 块设备上，从主机操作系统的角度来看，也没有任何变化：

```
root@ubuntu:~# ls -la /var/lib/lxc/iscsi_container/ 
total 16 
drwxrwx--- 3 root root 4096 Nov 15 21:01 . 
drwxr-xr-x 4 root root 4096 Nov 15 21:01 .. 
-rw-r--r-- 1 root root 716 Nov 15 21:01 config 
drwxr-xr-x 21 root root 4096 Nov 15 21:01 rootfs 
root@ubuntu:~#

```

由于容器的 `rootfs` 现在位于远程块设备上，取决于 LXC 启动主机与 iSCSI 目标主机之间的网络连接，可能会有一些延迟。在生产部署中，建议 iSCSI 目标和启动主机之间使用隔离的低延迟网络。

要将容器迁移到另一个主机，我们需要停止容器，卸载磁盘，然后从 iSCSI 目标注销块设备：

```
root@ubuntu:~# lxc-stop --name iscsi_container 
root@ubuntu:~# umount /var/lib/lxc 
root@ubuntu:~# iscsiadm -m node -T iqn.2001-04.com.example:lxc -p 10.208.129.201:3260 --logout 
Logging out of session [sid: 6, target: iqn.2001-04.com.example:lxc, portal: 10.208.129.201,3260] 
Logout of [sid: 6, target: iqn.2001-04.com.example:lxc, portal: 10.208.129.201,3260] successful. 
root@ubuntu:~#

```

### 恢复 iSCSI 容器

要恢复 iSCSI 容器，请按以下步骤操作：

1.  在配置为启动主机的新主机上，我们可以登录到与之前相同的目标：

    ```
     root@ubuntu-backup:~# iscsiadm -m node -T iqn.2001-04.com.example:lxc
          -p 10.208.129.201:3260 --login 
     Logging in to [iface: default, target: iqn.2001-04.com.example:lxc, 
          portal: 10.208.129.201,3260] (multiple) 
     Login to [iface: default, target: iqn.2001-04.com.example:lxc, 
          portal: 10.208.129.201,3260] successful. 
     root@ubuntu-backup:~#

    ```

1.  确保在登录到 iSCSI 目标后已提供新的块设备：

    ```
     root@ubuntu-backup:~# ls -la /dev/disk/by-path/ 
     total 0 
     drwxr-xr-x 2 root root 60 Nov 15 21:31 . 
     drwxr-xr-x 6 root root 120 Nov 15 21:31 .. 
     lrwxrwxrwx 1 root root 9 Nov 15 21:31 ip-10.208.129.201:
          3260-iscsi-iqn.2001-04.com.example:lxc-lun-0 -> ../../sda 
     root@ubuntu-backup:~#

    ```

    当在新主机上登录 iSCSI 目标时，请记住，呈现的块设备的名称可能与原始服务器上的不同。

1.  接下来，将块设备挂载到 LXC 根文件系统的默认位置：

    ```
     root@ubuntu-backup:~# mount /dev/disk/by-path/ip-10.208.129.201\
          :3260-iscsi-iqn.2001-04.com.example\:lxc-lun-0 /var/lib/lxc 
     root@ubuntu-backup:~#

    ```

1.  如果我们现在列出所有可用的容器，我们会看到之前在主机上创建的容器现在已经出现在新服务器上，仅通过挂载 iSCSI 目标即可实现：

    ```
     root@ubuntu-backup:~# lxc-ls -f 
     NAME STATE AUTOSTART GROUPS IPV4 IPV6 
     iscsi_container STOPPED 0 - - - 
     root@ubuntu-backup:~#

    ```

1.  最后，我们可以像往常一样启动容器：

    ```
     root@ubuntu-backup:~# lxc-start --name iscsi_container 
     root@ubuntu-backup:~# lxc-ls -f 
     NAME STATE AUTOSTART GROUPS IPV4 IPV6 
     iscsi_container RUNNING 0 - 10.0.3.162 - 
     root@ubuntu-backup:~#

    ```

如果容器配置了静态 IP 地址，则新主机上也会有相同的 IP；然而，如果容器通过动态方式获取网络配置，则其 IP 地址可能会发生变化。如果你的容器迁移时在 DNS 中有关联的 A 记录，请记住这一点。

## 使用复制 GlusterFS 存储的 LXC 活跃备份

在上一节中，我们展示了如何通过 iSCSI 导出远程块设备，将其用作 LXC 的本地存储。我们将块设备格式化为不允许多个服务器共享访问的文件系统。如果你在多个服务器上登录目标设备，并且它们同时尝试写入，数据将会损坏。使用单节点上的 iSCSI 设备托管 LXC 容器提供了一种出色的冗余解决方案，当 LXC 服务器宕机时，我们只需在新发起服务器上登录相同的块设备并启动容器。我们可以将这种方式视为冷备份，因为在将 iSCSI 块设备迁移到新主机、登录并启动容器时会有停机时间。

还有另一种方式，我们可以使用一个可以同时被多个服务器附加和访问的共享文件系统，在多个主机上运行相同的 LXC 容器，并且这些容器有不同的 IP 地址。让我们探索这种使用可扩展网络文件系统 GlusterFS 作为远程共享文件系统的场景。

### 创建共享存储

GlusterFS 具有两个主要组件——一个服务器组件，运行 GlusterFS 守护进程，导出称为 **砖块**（bricks）的本地块设备；一个客户端组件，通过 TCP/IP 网络与服务器连接，使用自定义协议，创建聚合的虚拟卷，然后可以将其挂载并用作常规文件系统。

有三种类型的卷：

+   **分布式**：这些是将文件分布到集群中的卷

+   **复制**：这些是将数据复制到存储集群中两个或更多节点的卷

+   **条带化**：这些条带文件分布在多个存储节点上

为了实现运行在这些共享卷上的 LXC 容器的高可用性，我们将使用两个 GlusterFS 服务器上的复制卷。在本例中，我们将在每个 GlusterFS 服务器上使用 LVM 上的块设备。

1.  在第一个存储节点上，我们将在 `/dev/xvdb` 块设备上创建 PV、VG 和 LV：

    ```
     root@node1:~# pvcreate /dev/xvdb 
     Physical volume "/dev/xvdb" successfully created 
     root@node1:~# vgcreate vg_node /dev/xvdb 
     Volume group "vg_node" successfully created 
     root@node1:~# lvcreate -n node1 -L 10g vg_node 
     Logical volume "node1" created. 
     root@node1:~#

    ```

1.  接下来，让我们在 LVM 设备上创建文件系统并挂载它。XFS 在 GlusterFS 中表现非常好：

    ```
     root@node1:~# mkfs.xfs -i size=512 /dev/vg_node/node1 
     meta-data=/dev/vg_node/node1   isize=512       agcount=4,  
          agsize=655360 blks 
     =                       sectsz=512      attr=2, 
          projid32bit=1 
     =                       crc=1           finobt=1, 
          sparse=0 
     data =                       bsize=4096      blocks=2621440, 
          imaxpct=25 
     =                       sunit=0         swidth=0 
          blks 
     naming =version 2              bsize=4096      ascii-ci=0 
          ftype=1 
     log    =internal log           bsize=4096      blocks=2560, 
          version=2 
     =                       sectsz=512      sunit=0 blks, 
          lazy-count=1 
     realtime =none                 extsz=4096      blocks=0, 
          rtextents=0 
     root@node1:~# mount /dev/vg_node/node1 /mnt/ 
     root@node1:~# mkdir /mnt/bricks

    ```

1.  最后，让我们安装 GlusterFS 服务：

    ```
     root@node1:~# apt-get install -y glusterfs-server

    ```

    在第二个 GlusterFS 节点上重复上述步骤，必要时将`node1`替换为`node2`。

1.  一旦在两个节点上运行了 GlusterFS 守护进程，就可以从`node1`进行探测，看看是否能看到任何对等节点：

    ```
     root@node1:~# gluster peer status 
     Number of Peers: 0 
     root@node1:~# gluster peer probe node2 
     peer probe: success. 
     root@node1:~# gluster peer status 
     Number of Peers: 1 
     Hostname: node2 
     Uuid: 65a480ba-e841-41a2-8f28-9d8f58f344ce 
     State: Peer in Cluster (Connected) 
     root@node1:~#

    ```

    通过从`node1`进行探测，我们现在可以看到`node1`和`node2`都属于同一个集群。

1.  下一步是创建复制卷，指定之前创建和挂载的 LV 的挂载路径。由于我们在存储集群中使用了两个节点，复制因子将为`2`。以下命令将创建复制卷，该卷将包含来自`node1`和`node2`的块设备，并列出创建的卷：

    ```
     root@node1:~# gluster volume create lxc_glusterfs replica 2 transport 
          tcp node1:/mnt/bricks/node1 node2:/mnt/bricks/node2 
     volume create: lxc_glusterfs: success: please start the volume to 
          access data 
     root@node1:~# gluster volume list 
     lxc_glusterfs 
     root@node1:~#

    ```

1.  要启动新创建的复制卷，运行以下命令：

    ```
     root@node1:~# gluster volume start lxc_glusterfs 
     volume start: lxc_glusterfs: success

    ```

    要创建并启动卷，我们只需要从其中一个存储节点运行上述命令。

1.  要获取新卷的信息，请在任何一个节点上运行此命令：

    ```
     root@node1:~# gluster volume info
     Volume Name: lxc_glusterfs
     Type: Replicate
     Volume ID: 9f11dc99-19d6-4644-87d2-fa6e983bcb83
     Status: Started
     Number of Bricks: 1 x 2 = 2
     Transport-type: tcp
     Bricks:
     Brick1: node1:/mnt/bricks/node1
     Brick2: node2:/mnt/bricks/node2
     Options Reconfigured:
     performance.readdir-ahead: on
     root@node1:~#

    ```

    从前面的输出中，注意到来自`node1`和`node2`的两个砖块。我们也可以在两个节点上看到它们：

    ```
     root@node1:~# ls -la /mnt/bricks/node1/ 
     total 0 
     drwxr-xr-x 4 root root 41 Nov 16 21:33 . 
     drwxr-xr-x 3 root root 19 Nov 16 21:33 .. 
     drw------- 6 root root 141 Nov 16 21:34 .glusterfs 
     drwxr-xr-x 3 root root 25 Nov 16 21:33 .trashcan 
     root@node1:~#

    ```

    我们在`node2`上看到以下内容：

    ```
     root@node2:~# ls -la /mnt/bricks/node1/ 
     total 0 
     drwxr-xr-x 4 root root 41 Nov 16 21:33 . 
     drwxr-xr-x 3 root root 19 Nov 16 21:33 .. 
     drw------- 6 root root 141 Nov 16 21:34 .glusterfs 
     drwxr-xr-x 3 root root 25 Nov 16 21:33 .trashcan 
     root@node2:~#

    ```

我们应该看到相同的文件，因为我们使用的是复制卷。在一个挂载的卷上创建的文件应会出现在另一个上，尽管与之前的 iSCSI 设置一样，网络延迟很重要，因为数据需要通过网络传输。

### 构建 GlusterFS LXC 容器

在 GlusterFS 集群准备好之后，让我们使用第三台服务器从存储集群挂载复制卷，并将其用作 LXC 根文件系统和配置位置：

1.  首先，让我们安装 GlusterFS 客户端：

    ```
     root@lxc1:~# apt-get install -y glusterfs-client attr

    ```

1.  接下来，将存储集群中的复制卷挂载到 LXC 主机上：

    ```
     root@lxc1:~# mount -t glusterfs node2:/lxc_glusterfs /var/lib/lxc

    ```

    在前面的命令中，我们使用`node2`作为目标进行挂载；不过，我们也可以以完全相同的方式使用`node1`。在`lxc_glusterfs`挂载命令中指定的目标设备名称就是我们之前指定的复制卷名称。

1.  现在，复制的 GlusterFS 卷已挂载到默认的 LXC 位置，让我们创建并启动一个容器：

    ```
     root@lxc1:~# lxc-create --name glusterfs_lxc --template ubuntu 
     root@lxc1:~# lxc-start --name glusterfs_lxc

    ```

1.  附加到容器并安装 Nginx，以便稍后我们可以从多个服务器测试连接性：

    ```
     root@lxc1:~# lxc-attach --name glusterfs_lxc 
     root@glusterfs_lxc:~# apt-get -y install nginx 
     ... 
     root@glusterfs_lxc:~# echo "Nginx on GlusterFS LXC" > 
          /var/www/html/index.nginx-debian.html 
     root@glusterfs_lxc:~# exit 
     exit 
     root@lxc1:~#

    ```

1.  获取容器的 IP 地址，并确保我们能从主机操作系统连接到 Nginx：

    ```
     root@lxc1:~# lxc-ls -f 
     NAME STATE AUTOSTART GROUPS IPV4 IPV6 
     glusterfs_lxc RUNNING 0 - 10.0.3.184 - 
     root@lxc1:~# curl 10.0.3.184 
     Nginx on GlusterFS LXC 
     root@lxc1:~#

    ```

1.  容器创建后，根文件系统和配置文件将在两个存储节点上可见：

    ```
     root@node1:~# ls -la /mnt/bricks/node1/ 
     total 12 
     drwxr-xr-x 5 root root 62 Nov 16 21:53 . 
     drwxr-xr-x 3 root root 19 Nov 16 21:33 .. 
     drw------- 261 root root 8192 Nov 16 21:54 .glusterfs 
     drwxrwx--- 3 root root 34 Nov 16 21:53 glusterfs_lxc 
     drwxr-xr-x 3 root root 25 Nov 16 21:33 .trashcan 
     root@node1:~#

    ```

1.  以下内容将在`node2`上可见：

    ```
     root@node2:~# ls -la /mnt/bricks/node2/ 
     total 12 
     drwxr-xr-x 5 root root 62 Nov 16 21:44 . 
     drwxr-xr-x 3 root root 19 Nov 16 21:33 .. 
     drw------- 261 root root 8192 Nov 16 21:44 .glusterfs 
     drwxrwx--- 3 root root 34 Nov 16 21:47 glusterfs_lxc 
     drwxr-xr-x 3 root root 25 Nov 16 21:33 .trashcan 
     root@node2:~#

    ```

根据您的网络带宽和延迟，复制数据到两个存储节点之间可能需要一些时间。这也会影响 LXC 容器的构建、启动和停止所需的时间。

### 恢复 GlusterFS 容器

让我们创建第二个 LXC 主机，安装 GlusterFS 客户端，并以与之前相同的方式挂载复制卷：

```
root@lxc2:~# apt-get install -y glusterfs-client attr 
root@lxc2:~# mount -t glusterfs node1:/lxc_glusterfs /var/lib/lxc 
root@lxc2:~# lxc-ls -f 
NAME STATE AUTOSTART GROUPS IPV4 IPV6 
glusterfs_lxc STOPPED 0 - - - 
root@lxc2:~#

```

请注意，仅通过挂载 GlusterFS 卷，主机现在看到容器处于停止状态。这与 `node1` 上运行的容器完全相同——即相同的配置和根文件系统。由于我们使用的是共享文件系统，因此可以在多个主机上启动容器，而不必担心数据损坏，这与使用 iSCSI 的情况不同：

```
root@lxc2:~# lxc-start --name glusterfs_lxc 
root@lxc2:~# lxc-ls -f 
NAME STATE AUTOSTART GROUPS IPV4 IPV6 
glusterfs_lxc RUNNING 0 - 10.0.3.184 - 
root@lxc2:~#

```

由于我们使用 DHCP 动态分配 IP 地址给容器，新的主机上的相同容器会获得一个新的 IP 地址。请注意，连接到容器中运行的 Nginx 仍然会返回相同的结果，因为容器在多个 LXC 节点之间共享其文件系统和配置文件：

```
root@lxc2:~# curl 10.0.3.184 
Nginx on GlusterFS LXC 
root@lxc2:~#

```

这种设置在某种程度上实现了热备份，在这种情况下，我们可以使用两个容器通过像 HAProxy 这样的代理服务，第二个节点仅在第一个节点宕机时才会使用，确保任何配置更改立即生效。作为替代方案，可以同时使用两个 LXC 容器，但请注意，它们会写入相同的文件系统，因此在这种情况下，Nginx 日志将由 `lxc1` 和 `lxc2` 节点上的两个 LXC 容器写入。

# 监控和告警 LXC 指标

监控 LXC 容器与监控虚拟机或服务器没有太大区别——我们可以在容器内部运行监控客户端，或者在运行 LXC 的实际主机上运行。由于容器的根文件系统在主机上可见，且 LXC 使用 cgroups 和命名空间，我们可以直接从主机操作系统收集各种信息，如果我们不想在容器中运行监控代理的话。在我们查看两个 LXC 监控示例之前，先来看看如何收集我们可以监控和告警的各种指标。

## 收集容器指标

LXC 提供了一些简单的工具，用于监控容器的状态和资源利用率。正如你接下来将看到的那样，这些工具提供的信息并不冗长；但是，我们可以利用 cgroup 文件系统从中收集更多的信息。让我们来探索一下这些选项。

### 使用 lxc-monitor 跟踪容器状态

`lxc-monitor` 工具可用于跟踪容器的状态变化——例如它们何时启动或停止。

为了演示这一点，打开两个终端；在一个终端创建一个新容器，在另一个终端运行 `lxc-monitor` 命令。启动容器并观察第二个终端中的输出：

```
root@ubuntu:~# lxc-create --name monitor_lxc --template ubuntu 
root@ubuntu:~# lxc-start --name monitor_lxc 
root@ubuntu:~# lxc-monitor --name monitor_lxc 
'monitor_lxc' changed state to [STARTING] 
'monitor_lxc' changed state to [RUNNING] 

```

停止容器并注意状态变化：

```
root@ubuntu:~# lxc-stop --name monitor_lxc 
root@ubuntu:~# 
'monitor_lxc' changed state to [STARTING] 
'monitor_lxc' changed state to [RUNNING] 
'monitor_lxc' exited with status [0] 
'monitor_lxc' changed state to [STOPPING] 
'monitor_lxc' changed state to [STOPPED] 
root@ubuntu:~# lxc-start --name monitor_lxc

```

### 使用 lxc-top 获取 CPU 和内存利用率

`lxc-top` 工具类似于标准的 `top` Linux 命令，它显示 CPU、内存和 I/O 的使用情况。要启动它，请执行以下命令：

```
root@ubuntu:~# lxc-top --name monitor_lxc 
Container  CPU    CPU     CPU    BlkIO    Mem 
Name               Used     Sys    User   
Total    Used 
monitor_lxc        0.52   0.26    0.16    0.00 
12.71 MB 
TOTAL 1 of 1       0.52   0.26    0.16    0.00 
12.71 MB

```

### 使用 lxc-info 收集容器信息

我们可以使用 `lxc-info` 工具定期轮询信息，例如 CPU、内存、I/O 和网络利用率：

```
root@ubuntu:~# lxc-info --name monitor_lxc 
Name:        monitor_lxc 
State:       RUNNING 
PID:         19967 
IP:          10.0.3.88 
CPU use:     0.53 seconds 
BlkIO use:   0 bytes 
Memory use:  12.74 MiB 
KMem use:    0 bytes 
Link:        veth8OX0PW 
TX bytes:    1.34 KiB 
RX bytes:    1.35 KiB 
Total bytes: 2.69 KiB 
root@ubuntu:~#

```

### 利用 cgroups 收集内存指标

在 *第一章，**Linux 容器介绍* 中，我们详细探讨了 cgroups，并了解了 LXC 如何在 cgroup 虚拟文件系统中为每个启动的容器创建目录层级。要查找我们之前构建的容器的 cgroup 层级，请运行以下命令：

```
root@ubuntu:~# find /sys/fs/ -type d -name monitor_lxc 
/sys/fs/cgroup/freezer/lxc/monitor_lxc 
/sys/fs/cgroup/cpuset/lxc/monitor_lxc 
/sys/fs/cgroup/net_cls,net_prio/lxc/monitor_lxc 
/sys/fs/cgroup/devices/lxc/monitor_lxc 
/sys/fs/cgroup/perf_event/lxc/monitor_lxc 
/sys/fs/cgroup/hugetlb/lxc/monitor_lxc 
/sys/fs/cgroup/blkio/lxc/monitor_lxc 
/sys/fs/cgroup/pids/lxc/monitor_lxc 
/sys/fs/cgroup/memory/lxc/monitor_lxc 
/sys/fs/cgroup/cpu,cpuacct/lxc/monitor_lxc 
/sys/fs/cgroup/systemd/lxc/monitor_lxc 
root@ubuntu:~#

```

我们可以使用内存组中的文件来设置和获取可以监控并告警的指标。例如，要将容器的内存设置为 512 MB，运行以下命令：

```
root@ubuntu:~# lxc-cgroup --name monitor_lxc memory.limit_in_bytes 536870912 
root@ubuntu:~# 

```

要读取容器当前的内存利用率，请执行以下命令：

```
root@ubuntu:~# cat /sys/fs/cgroup/memory/lxc/monitor_lxc/memory.usage_in_bytes 
13361152 
root@ubuntu:~#

```

要收集有关容器内存的更多信息，请从以下文件读取：

```
root@ubuntu:~# cat /sys/fs/cgroup/memory/lxc/monitor_lxc/memory.stat 
cache 8794112 
rss 4833280 
rss_huge 0 
mapped_file 520192 
dirty 0 
... 
root@ubuntu:~#

```

### 使用 cgroups 收集 CPU 统计数据

要收集 CPU 使用情况，我们可以从 `cpuacct` cgroup 子系统中读取：

```
root@ubuntu:~# cat /sys/fs/cgroup/cpu,cpuacct/lxc/monitor_lxc/cpuacct.usage 
627936278 
root@ubuntu:~#

```

### 收集网络指标

每个容器在主机上创建一个虚拟接口；之前从 `lxc-info` 命令显示的接口是 `veth8OX0PW`。我们可以通过运行以下命令来收集发送和接收的包、错误率等信息：

```
root@ubuntu:~# ifconfig veth8OX0PW 
veth8OX0PW Link encap:Ethernet HWaddr fe:4d:bf:3f:17:8f 
 inet6 addr: fe80::fc4d:bfff:fe3f:178f/64 Scope:Link 
 UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1 
 RX packets:125645 errors:0 dropped:0 overruns:0 frame:0 
 TX packets:129909 errors:0 dropped:0 overruns:0 carrier:0 
 collisions:0 txqueuelen:1000 
 RX bytes:20548005 (20.5 MB) TX bytes:116477293 (116.4 MB) 
root@ubuntu:~#

```

另外，我们可以连接到容器的网络命名空间并通过这种方式获取信息。以下三个命令演示了如何在容器的网络命名空间中执行命令。注意 `19967` 的 PID；它可以从 `lxc-info` 命令中获取：

```
root@ubuntu:~# mkdir -p /var/run/netns 
root@ubuntu:~# ln -sf /proc/19967/ns/net /var/run/netns/monitor_lxc 
root@ubuntu:~# ip netns exec monitor_lxc ip a s 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1 
 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 
 inet 127.0.0.1/8 scope host lo 
 valid_lft forever preferred_lft forever 
 inet6 ::1/128 scope host 
 valid_lft forever preferred_lft forever 
17: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000 
 link/ether 00:16:3e:01:b7:8b brd ff:ff:ff:ff:ff:ff link-netnsid 0 
 inet 10.0.3.88/24 brd 10.0.3.255 scope global eth0 
 valid_lft forever preferred_lft forever 
 inet6 fe80::216:3eff:fe01:b78b/64 scope link 
 valid_lft forever preferred_lft forever 
root@ubuntu:~#

```

请注意，我们可以看到 LXC 容器内部的网络接口，尽管我们是在主机上运行这些命令。

## 使用 Monit 进行简单的容器监控和告警

现在我们已经演示了如何收集各种监控数据，接下来我们将实际设置一个监控系统并对其进行告警。在本节中，我们将安装一个简单的监控解决方案，利用 `monit` 守护进程。Monit 是一个易于配置的服务，使用可以基于特定监控事件和阈值自动执行的脚本。

接下来，让我们看一些例子：

1.  要安装 `monit`，运行以下命令：

    ```
     root@ubuntu:~# apt-get install -y monit mailutils

    ```

1.  接下来，创建一个最小配置文件。随安装包提供的配置文档写得相当清楚：

    ```
     root@ubuntu:~# cd /etc/monit/ 
     root@ubuntu:/etc/monit# cat monitrc | grep -vi "#" 
     set daemon 120 
     set logfile /var/log/monit.log 
     set idfile /var/lib/monit/id 
     set statefile /var/lib/monit/state 
     set eventqueue 
     basedir /var/lib/monit/events 
     slots 100 
     set httpd port 2812 and 
     allow admin:monit 
     include /etc/monit/conf.d/* 
     include /etc/monit/conf-enabled/* 
     root@ubuntu:/etc/monit#

    ```

    前面的配置启动了一个可以通过指定凭证在端口 `2812` 访问的 Web 界面。它还定义了两个可以读取配置文件的目录。

1.  接下来，让我们创建一个监控配置，检查容器是否在运行。执行一个脚本，我们将写在下面，来执行实际检查：

    ```
     root@ubuntu:/etc/monit# cat conf.d/container_state.cfg 
     check program container_status with path 
          "/etc/monit/container_running.sh 
          monitor_lxc" 
     if status == 1 
     then exec "/bin/bash -c '/etc/monit/alert.sh'" 
     group lxc 
     root@ubuntu:/etc/monit#

    ```

    前面的配置指示 `monit` 定期运行 `container_running.sh` 脚本，如果退出状态为 `1`，则执行一个名为 `alert.sh` 的第二个脚本来发出警报。很简单。`container_running.sh` 脚本如下：

    ```
     root@ubuntu:/etc/monit# cat container_running.sh 
     #!/bin/bash 
     CONTAINER_NAME=$1 
     CONTAINER_STATE=$(lxc-info --state --name $CONTAINER_NAME 
          | awk '{print $2}') 
     if [ "$CONTAINER_STATE" != "RUNNING" ] 
     then 
     exit 1 
     else 
     exit 0 
     fi 
     root@ubuntu:/etc/monit#

    ```

    我们可以看到我们正在使用 `lxc-info` 命令来检查容器的状态。`alert.sh` 脚本更加简单：

    ```
     root@ubuntu:/etc/monit# cat alert.sh 
     #!/bin/bash 
     echo "LXC container down" | mail -s "LXC container Alert" 
          youremail@example.com 
     root@ubuntu:/etc/monit#

    ```

1.  重新加载`monit`服务并检查我们在配置文件中之前命名的新的监控服务`container_status`的状态：

    ```
     root@ubuntu:/etc/monit# monit reload 
     Reinitializing monit daemon 
     The Monit daemon 5.16 uptime: 15h 43m 
     root@ubuntu:/etc/monit# monit status container_status 
     The Monit daemon 5.16 uptime: 15h 47m 
     Program 'container_status' 
     status Status ok 
     monitoring status Monitored 
     last started Fri, 18 Nov 2016 15:03:24 
     last exit value 0 
     data collected Fri, 18 Nov 2016 15:03:24 
     root@ubuntu:/etc/monit#

    ```

    我们还可以通过端口 `2812` 连接到网页界面，并查看新定义的监控目标：

    ![使用 Monit 进行简单容器监控和警报](img/image_07_001.jpg)

1.  让我们停止容器并检查`monit`的状态：

    ```
     root@ubuntu:/etc/monit# lxc-stop --name monitor_lxc 
     root@ubuntu:/etc/monit# monit status container_status 
     The Monit daemon 5.16 uptime: 15h 53m 
     Program 'container_status' 
     status Status failed 
     monitoring status Monitored 
     last started Fri, 18 Nov 2016 15:09:24 
     last exit value 1 
     data collected Fri, 18 Nov 2016 15:09:24 
     root@ubuntu:/etc/monit#

    ```

    ![使用 Monit 进行简单容器监控和警报](img/image_07_002.jpg)

从命令输出和网页界面中注意到，`container_status`服务的状态现在是`failed`。由于我们设置了`monit`，当我们监控的服务失败时会发送电子邮件，检查邮件日志。你应该已经收到了来自`monit`的电子邮件，这封邮件很可能会被归入你的`垃圾邮件`文件夹：

```
root@ubuntu:/etc/monit# tail -5 /var/log/mail.log 
Nov 18 15:13:51 ubuntu postfix/pickup[26069]: 8AB30177CB3: uid=0 from=<root@ubuntu> 
Nov 18 15:13:51 ubuntu postfix/cleanup[31295]: 8AB30177CB3: message-id=<20161118151351.8AB30177CB3@ubuntu> 
Nov 18 15:13:51 ubuntu postfix/qmgr[5392]: 8AB30177CB3: from=<root@ubuntu>, size=340, nrcpt=1 (queue active) 
Nov 18 15:13:51 ubuntu postfix/smtp[31297]: 8AB30177CB3: to=< youremail@example.com >, relay=gmail-smtp-in.l.google.com[74.125.70.26]:25, delay=0.22, delays=0.01/0.01/0.08/0.13, dsn=2.0.0, status=sent (250 2.0.0 OK 1479482031 u74si2324555itu.40 - gsmtp) 
Nov 18 15:13:51 ubuntu postfix/qmgr[5392]: 8AB30177CB3: removed 
root@ubuntu:/etc/monit#

```

如需更多关于 Monit 的信息，请参见[`mmonit.com/monit/documentation/monit.html`](https://mmonit.com/monit/documentation/monit.html)。

Monit 是为每个服务器设置 LXC 容器监控的快速简便方法。它是无代理的，且由于 cgroup 层次结构中暴露的指标，可以轻松对各种数据点设置警报，而无需在容器中附加或运行任何额外的组件。

## 使用 Sensu 进行容器监控和警报触发

Monit 是一个非常适合在分散式环境中进行监控和警报的工具。然而，为了实现更强大且功能丰富的集中式监控部署，可以使用其他监控工具，如 Sensu。使用 Sensu 实现监控有两种主要方式——在每个容器中部署代理，或者在 LXC 主机上使用独立检查，类似于 Monit，从 cgroups 等数据源收集数据。

Sensu 使用客户端-服务器架构，服务器通过 RabbitMQ 提供的消息队列发布检查，客户端订阅该队列中的主题，并根据设定的阈值执行检查和警报。状态和历史数据存储在 Redis 服务器中。

让我们首先展示在 LXC 容器内部的 Sensu 部署，然后再进行无代理监控。

### 使用 Sensu 代理和服务器监控 LXC 容器

我们需要安装 Sensu 使用的必要服务，包括 Redis 和 RabbitMQ：

1.  让我们先安装 Redis 服务器，安装完成后，确保它正在运行：

    ```
     root@ubuntu:~# apt-get -y install redis-server 
     root@ubuntu:~# redis-cli ping 
     PONG 
     root@ubuntu:~#

    ```

1.  安装 RabbitMQ 也同样简单：

    ```
     root@ubuntu:~# apt-get install -y rabbitmq-server

    ```

1.  安装完成后，我们需要创建代理将订阅并读取消息的虚拟主机：

    ```
     root@ubuntu:~# rabbitmqctl add_vhost /sensu 
     Creating vhost "/sensu" ... 
     root@ubuntu:~#

    ```

1.  接下来，创建一个用户名和密码来连接到该主题：

    ```
     root@ubuntu:~# rabbitmqctl add_user sensu secret 
     Creating user "sensu" ... 
     root@ubuntu:~# rabbitmqctl set_permissions -p 
          /sensu sensu ".*" ".*" ".*" 
     Setting permissions for user "sensu" in vhost "/sensu" ... 
     root@ubuntu:~#

    ```

1.  现在是安装 Sensu 服务器和客户端的时候了：

    ```
     root@ubuntu:~# wget -q 
          https://sensu.global.ssl.fastly.net/apt/pubkey.gpg -O- 
          | sudo apt-key add - 
     OK 
     root@ubuntu:~# echo 
          "deb https://sensu.global.ssl.fastly.net/apt sensu main" 
          | sudo tee /etc/apt/sources.list.d/sensu.list 
     deb https://sensu.global.ssl.fastly.net/apt sensu main 
     root@ubuntu:~# apt-get update 
     root@ubuntu:~# apt-get install -y sensu 
     root@ubuntu:~# cd /etc/sensu/conf.d/

    ```

至少，我们需要五个配置文件，一个是 Sensu API 端点的配置文件，两个指定我们使用的传输方式——在本例中是 RabbitMQ——还有 Sensu 的 Redis 配置文件，以及在同一服务器上运行的 Sensu 客户端的客户端配置文件。以下配置文件基本上不言自明——我们指定 RabbitMQ 和 Redis 服务器的 IP 地址和端口，以及 API 服务：

```
root@ubuntu:/etc/sensu/conf.d# cat api.json 
{ 
 "api": { 
 "host": "localhost", 
 "bind": "0.0.0.0", 
 "port": 4567 
 } 
} 
root@ubuntu:/etc/sensu/conf.d# 
root@ubuntu:/etc/sensu/conf.d# cat transport.json 
{ 
 "transport": { 
 "name": "rabbitmq", 
 "reconnect_on_error": true 
 } 
} 
root@ubuntu:/etc/sensu/conf.d# 
root@ubuntu:/etc/sensu/conf.d# cat rabbitmq.json 
{ 
 "rabbitmq": { 
 "host": "10.208.129.253", 
 "port": 5672, 
 "vhost": "/sensu", 
 "user": "sensu", 
 "password": "secret" 
 } 
} 
root@ubuntu:/etc/sensu/conf.d# 
root@ubuntu:/etc/sensu/conf.d# cat redis.json 
{ 
 "redis": { 
 "host": "localhost", 
 "port": 6379 
 } 
} 
root@ubuntu:/etc/sensu/conf.d# cat client.json 
{ 
 "client": { 
 "name": "ubuntu", 
 "address": "127.0.0.1", 
 "subscriptions": [ 
 "base" 
 ], 
 "socket": { 
 "bind": "127.0.0.1", 
 "port": 3030 
 } 
 } 
} 
root@ubuntu:/etc/sensu/conf.d#

```

有关 Sensu 的更多信息，请参阅[`sensuapp.org/docs/`](https://sensuapp.org/docs/)。

在启动 Sensu 服务器之前，我们可以安装一个名为 Uchiwa 的基于 Web 的前端：

```
root@ubuntu:~# apt-get install -y uchiwa

```

在配置文件中，我们指定 Sensu API 服务的地址和端口——本地主机和端口`4567`——以及 Uchiwa 监听的端口——`3000`——在本例中：

```
root@ubuntu:/etc/sensu/conf.d# cat /etc/sensu/uchiwa.json 
{ 
 "sensu": [ 
 { 
 "name": "LXC Containers", 
 "host": "localhost", 
 "ssl": false, 
 "port": 4567, 
 "path": "", 
 "timeout": 5000 
 } 
 ], 
 "uchiwa": { 
 "port": 3000, 
 "stats": 10, 
 "refresh": 10000 
 } 
} 
root@ubuntu:/etc/sensu/conf.d#

```

配置完成后，我们来启动 Sensu 服务：

```
root@ubuntu:~# /etc/init.d/sensu-server start 
* Starting sensu-server [ OK ] 
root@ubuntu:~# /etc/init.d/sensu-api start 
* Starting sensu-api [ OK ] 
root@ubuntu:/etc/sensu/conf.d# /etc/init.d/sensu-client start 
* Starting sensu-client [ OK ] 
root@ubuntu:/etc/sensu/conf.d# 
root@ubuntu:~# /etc/init.d/uchiwa restart 
uchiwa stopped. 
uchiwa started. 
root@ubuntu:~#

```

现在 Sensu 服务器已经完全配置好，我们需要连接到主机上的一个容器并安装 Sensu 代理：

```
root@ubuntu:~# lxc-attach --name monitor_lxc 
root@monitor_lxc:~# apt-get install -y wget 
root@monitor_lxc:~# wget -q https://sensu.global.ssl.fastly.net/apt/pubkey.gpg -O- | sudo apt-key add - 
OK 
root@monitor_lxc:~# echo "deb https://sensu.global.ssl.fastly.net/apt sensu main" | sudo tee /etc/apt/sources.list.d/sensu.list 
deb https://sensu.global.ssl.fastly.net/apt sensu main 
root@monitor_lxc:~# apt-get update 
root@monitor_lxc:~# apt-get install -y sensu 
root@monitor_lxc:/# cd /etc/sensu/conf.d/

```

要配置 Sensu 代理，我们需要编辑客户端配置文件，在其中指定 IP 地址和容器的名称：

```
root@monitor_lxc:/etc/sensu/conf.d# cat client.json 
{ 
 "client": { 
 "name": "monitor_lxc", 
 "address": "10.0.3.2", 
 "subscriptions": ["base"] 
 } 
} 
root@monitor_lxc:/etc/sensu/conf.d#

```

我们通过提供之前创建的 IP 地址、端口和凭据，告诉代理如何连接到主机上的 RabbitMQ 服务器：

```
root@monitor_lxc:/etc/sensu/conf.d# cat rabbitmq.json 
{ 
 "rabbitmq": { 
 "host": "10.0.3.1", 
 "port": 5672, 
 "vhost": "/sensu", 
 "user": "sensu", 
 "password": "secret" 
 } 
} 
root@monitor_lxc:/etc/sensu/conf.d#

```

然后，指定传输机制：

```
root@monitor_lxc:/etc/sensu/conf.d# cat transport.json 
{ 
 "transport": { 
 "name": "rabbitmq", 
 "reconnect_on_error": true 
 } 
} 
root@monitor_lxc:/etc/sensu/conf.d#

```

在前面提到的三个文件配置好之后，我们来启动代理：

```
root@monitor_lxc:/etc/sensu/conf.d# /etc/init.d/sensu-client start 
* Starting sensu-client [ OK ] 
root@monitor_lxc:/etc/sensu/conf.d#

```

![使用 Sensu 代理和服务器监控 LXC 容器](img/image_07_003.jpg)

为了验证所有服务是否正常运行，并且 Sensu 代理能够连接到服务器，我们可以使用主机 IP 连接到 Uchiwa 界面，端口为 3000，如前所述。

仍然连接到 LXC 容器时，让我们安装一个 Sensu 检查。Sensu 检查可以作为 gem 使用，或者可以手动编写。我们来搜索`gem`库，看看是否有现成的内存检查，而不是自己编写：

```
root@monitor_lxc:/etc/sensu/conf.d# apt-get install -y rubygems 
root@monitor_lxc:/etc/sensu/conf.d# gem search sensu | grep plugins | grep memory 
sensu-plugins-memory (0.0.2) 
sensu-plugins-memory-checks (1.0.2) 
root@monitor_lxc:/etc/sensu/conf.d#

```

安装内存检查并重启代理：

```
root@monitor_lxc:/etc/sensu/conf.d# gem install sensu-plugins-memory-checks 
root@monitor_lxc:/etc/sensu/conf.d# /etc/init.d/sensu-client restart 
configuration is valid 
* Stopping sensu-client [ OK ] 
* Starting sensu-client [ OK ] 
root@monitor_lxc:/etc/sensu/conf.d#

```

在 Sensu 服务器主机上（而不是容器内），我们需要定义新的内存检查，以便 Sensu 服务器可以告诉代理执行它。我们通过创建一个新的检查文件来完成这一步：

```
root@ubuntu:/etc/sensu/conf.d# cat check_memory.json
{
  "checks": {
    "memory_check": {
    "command": "/usr/local/bin/check-memory-percent.rb -w 80 -c 90",
    "subscribers": ["base"],
    "handlers": ["default"],
    "interval": 300
   }
  }
}
root@ubuntu:/etc/sensu/conf.d#

```

我们指定需要从 Sensu 代理运行的检查的路径和名称，在本例中就是我们从`gem`安装到容器中的 Ruby 脚本。重启 Sensu 服务以使更改生效：

```
root@ubuntu:/etc/sensu/conf.d# /etc/init.d/sensu-server restart 
configuration is valid 
* Stopping sensu-server [ OK ] 
* Starting sensu-server [ OK ] 
root@ubuntu:/etc/sensu/conf.d# /etc/init.d/sensu-api restart 
configuration is valid 
* Stopping sensu-api [ OK ] 
* Starting sensu-api [ OK ] 
root@ubuntu:/etc/sensu/conf.d# /etc/init.d/uchiwa restart 
Killing uchiwa (pid 15299) with SIGTERM 
Waiting uchiwa (pid 15299) to die... 
Waiting uchiwa (pid 15299) to die... 
uchiwa stopped. 
uchiwa started. 
root@ubuntu:/etc/sensu/conf.d#

```

查看 Uchiwa 界面，我们可以看到内存检查现在已经激活：

![使用 Sensu 代理和服务器监控 LXC 容器](img/image_07_004.jpg)

我们可以在 LXC 容器内从 gem 安装多个 Sensu 检查脚本；像在普通服务器或虚拟机上一样，在 Sensu 服务器上定义它们，最终获得一个完整的监控解决方案。

在容器内运行代理时有一些注意事项：

+   代理会消耗资源；如果我们尝试运行内存占用较小的容器，这可能不是最好的解决方案。

+   在衡量 CPU 负载时，容器内的负载将反映宿主机本身的总体负载，因为容器并未通过虚拟化程序与宿主机隔离。衡量 CPU 利用率的最佳方法是从 cgroups 获取数据，或者使用宿主机上的`lxc-info`命令。

+   如果使用共享根文件系统（如我们在上一章中看到的那样），监控容器内的磁盘空间可能会反映服务器上的总空间。

如果不希望在 LXC 容器内运行 Sensu 代理，我们可以改为在同一主机上的 Sensu 服务器执行独立检查。接下来，我们来探索这种设置。

### 使用独立 Sensu 检查监控 LXC 容器

在运行 LXC 容器的服务器上，我们可以像本章前面所做的那样，直接在宿主操作系统上安装 Sensu 代理，而不是在每个容器内安装。我们还可以利用 Sensu 的独立检查，它提供了一种去中心化的监控方式，这意味着 Sensu 代理定义并调度检查，而不是由 Sensu 服务器来完成。这为我们提供了一个好处，即无需在容器内安装代理和监控脚本，Sensu 代理会在每个服务器上运行这些检查。

让我们展示如何在我们一直使用的 LXC 主机上创建一个独立的检查：

```
root@ubuntu:/etc/sensu/conf.d# cat check_memory_no_agent.json 
{ 
 "checks": { 
 "check_memory_no_agent": { 
 "command": "check_memory_no_agent.sh -n monitor_lxc -w 943718400 
      -e 1048576000", 
 "standalone": true, 
 "subscribers": [ 
 "base" 
 ], 
 "interval": 300 
 } 
 } 
} 
root@ubuntu:/etc/sensu/conf.d#

```

定义独立检查的主要区别在于存在`"standalone": true`的字段，正如前面的输出所示。在检查配置的命令部分，我们指定执行什么脚本来执行实际检查，以及它应当在什么阈值时发出警报。脚本可以是任何东西，只要它在出现关键警报时以错误代码`2`退出，警告时以错误代码`1`退出，所有正常时以`0`退出。

这是一个非常简单的 bash 脚本，它使用`memory.usage_in_bytes` cgroup 文件来收集内存使用指标，并在达到指定阈值时发出警报：

```
root@ubuntu:/etc/sensu/conf.d# cd ../plugins/ 
root@ubuntu:/etc/sensu/plugins# cat check_memory_no_agent.sh 
#!/bin/bash 
# Checks memory usage for an LXC container from cgroups 
 usage() 
{ 
 echo "Usage: `basename $0` -n|--name monitor_lxc -w|--warning 
  5000 -e|--error 10000" 
 exit 2 
} 
sanity_check() 
{ 
 if [ "$CONTAINER_NAME" == "" ] || [ "$WARNING" == "" ] || [ "$ERROR" 
  == "" ] 
 then 
 usage 
 fi 
} 
report_result() 
{ 
 if [ "$MEMORY_USAGE" -ge "$ERROR" ] 
 then 
 echo "CRITICAL - Memory usage too high at $MEMORY_USAGE" 
 exit 2 
 elif [ "$MEMORY_USAGE" -ge "$WARNING" ] 
 then 
 echo "WARNING - Memory usage at $MEMORY_USAGE" 
 exit 1 
 else 
 echo "Memory Usage OK at $MEMROY_USAGE" 
 exit 0 
 fi 
} 
get_memory_usage() 
{ 
 declare -g -i MEMORY_USAGE=0 
 MEMORY_USAGE=$(cat 
  /sys/fs/cgroup/memory/lxc/$CONTAINER_NAME/memory.usage_in_bytes) 
} 
main() 
{ 
 sanity_check 
 get_memory_usage 
 report_result 
} 
while [[ $# > 1 ]] 
do 
 key=$1 
 case $key in 
 -n|--name) 
 CONTAINER_NAME=$2 
 shift 
 ;; 
 -w|--warning) 
 WARNING=$2 
 shift 
 ;; 
 -e|--error) 
 ERROR=$2 
 shift 
 ;; 
 *) 
 usage 
 ;; 
 esac 
 shift 
done 
main 
root@ubuntu:/etc/sensu/plugins#

```

上述脚本虽然简单，但应该是读者开始编写更多有用的 Sensu 检查的良好起点，这些检查能够操作各种 LXC 指标。通过查看脚本，我们可以看到它有三个基本功能。首先，它在`sanitiy_check()`函数中检查提供的容器名称，以及警告和错误阈值。然后，它在`get_memory_usage()`函数中从 cgroup 文件获取内存使用情况（以字节为单位），最后，它在`report_result()`函数中报告结果，通过返回适当的错误代码，正如之前所描述的那样。

更改脚本的权限和执行标志，重新加载 Sensu 服务，并确保检查在 Uchiwa 中显示：

```
root@ubuntu:/etc/sensu/plugins# chown sensu:sensu check_memory_no_agent.sh 
root@ubuntu:/etc/sensu/plugins# chmod u+x check_memory_no_agent.sh 
root@ubuntu:/etc/sensu/plugins# echo "sensu ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/sensu 
root@ubuntu:/etc/sensu/plugins# /etc/init.d/sensu-client restart

```

![使用独立 Sensu 检查监控 LXC 容器](img/image_07_005.jpg)

与 Monit 类似，Sensu 提供了处理程序，当触发警报时将被触发。这可以是发送电子邮件的自定义脚本，向 PagerDuty 等外部服务打电话等。所有这些都提供了处理 LXC 警报的自动化和主动的方式。

有关 Sensu 处理程序的更多信息，请参阅 [`sensuapp.org/docs/latest/reference/handlers.html`](https://sensuapp.org/docs/latest/reference/handlers.html) 的文档。

# LXC、Jenkins 和 Sensu 的简单自动扩展模式

在 第六章 中，我们探讨了如何通过在多个主机上为 LXC 和 HAProxy 配置更多容器来实现水平扩展服务。在本章中，我们探讨了监控 LXC 容器资源利用率并根据警报触发操作的不同方法。有了这些知识基础，我们现在可以实现一种常用的自动扩展模式，如下图所示：

![LXC、Jenkins 和 Sensu 的简单自动扩展模式](img/image_07_006.jpg)

此模式使用 Jenkins 作为构建系统，由 Sensu 警报处理程序控制。例如，当运行在 LXC 容器内的 Sensu 代理接收来自 Sensu 服务器的预定检查（例如内存检查）时，它执行脚本并根据配置的警报阈值返回 `OK`、`Warning` 或 `Critical` 状态。如果返回 `Critical` 状态，则配置的 Sensu 处理程序（可以简单到一个 `curl` 命令）将 API 调用发送到 Jenkins 服务器，后者依次执行预配置的作业。Jenkins 作业可以是选择从主机列表中的主机的脚本，根据一组标准来构建新容器，或者尝试增加提醒的 LXC 容器的可用内存。这是一种最简单的自动扩展设计模式之一，利用监控系统和 Jenkins 等 RESTful 构建服务。

在接下来的章节中，我们将探讨一个完整的 OpenStack 部署，该部署利用智能调度程序在可用内存或已运行的容器数量基础上选择计算主机以部署新的 LXC 容器。

我们已经看过如何在前面的图表中实现大多数组件和交互的示例。让我们快速触及 Jenkins 并设置一个简单的作业，通过远程 REST API 调用时创建新的 LXC 容器。其余部分留给读者自行尝试。

要在 Ubuntu 上安装 Jenkins，请运行以下命令：

```
root@ubuntu: ~# wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add - 
root@ubuntu:~# sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list' 
root@ubuntu:~# apt-get update 
root@ubuntu:~# apt-get install jenkins

```

Jenkins 监听端口 `8080`。以下两条 iptables 规则将端口 `80` 转发到端口 `8080`，使连接更加便捷：

```
root@ubuntu:~# iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080 
root@ubuntu:~# iptables -t nat -I OUTPUT -p tcp -d 127.0.0.1 --dport 80 -j REDIRECT --to-ports 8080

```

确保将 `jenkins` 用户添加到 `sudoers` 文件中：

```
root@ubuntu:~# cat /etc/sudoers | grep jenkins 
jenkins ALL=(ALL) NOPASSWD:ALL 
root@ubuntu:~#

```

安装完成后，Jenkins 将启动并可通过 HTTP 访问。打开 Jenkins 网页并粘贴如下文件的内容：

```
root@ubuntu:~# cat /var/lib/jenkins/secrets/initialAdminPassword 
32702ecf076e439c8d58c51f1247776d 
root@ubuntu:~#

```

安装推荐的插件后，执行以下操作：

1.  创建一个名为`LXC Provision`的新作业。

1.  在**构建触发器**中，选择**远程触发构建（例如，从脚本中）**，并输入一个随机字符串，例如`somesupersecrettoken`。

1.  在**构建**部分，点击**添加构建步骤**并选择**执行 shell**。

1.  在**命令**窗口中，添加以下简单的 bash 脚本：

    ```
          set +x
          echo "Building new LXC container"
          sudo lxc-create --template ubuntu --name $(cat /dev/urandom | 
          tr -cd 'a-z' | head -c 10)
    ```

1.  最后，点击**保存**。

Jenkins 作业应该类似于以下屏幕截图：

![带有 LXC、Jenkins 和 Sensu 的简单自动扩展模式](img/image_07_007.jpg)

要远程触发该作业，运行以下命令，替换用户名、密码和 IP 地址为你主机上配置的：

```
root@ubuntu:~# curl http://user:password@192.237.167.103:8080/job/LXC%20Provision/build?token=somesupersecrettoken 
root@ubuntu:~#

```

这将触发 Jenkins 作业，进而创建一个随机名称的新 LXC 容器：

```
root@ubuntu:~# lxc-ls -f 
NAME STATE AUTOSTART GROUPS IPV4 IPV6 
ifjvdlpvnv STOPPED 0 - - - 
root@ubuntu:~#

```

现在，只需为检查或一组检查创建一个 Sensu 处理程序（或`monit`触发脚本），然后可以执行类似前面`curl`命令的操作。 当然，这个部署只是为了让我们略微了解通过结合 Sensu、Jenkins 和 LXC 实现容器内自动扩展服务的可能性。

# 总结

在本章中，我们学习了如何使用 Linux 原生工具如`tar`和`rsync`，以及 LXC 工具如`lxc-copy`来备份 LXC 容器。我们查看了如何使用 iSCSI 目标作为 LXC 根文件系统和配置文件存储，创建冷备份和热备份的示例。我们还学习了如何使用 GlusterFS 部署共享网络文件系统，以及在同一文件系统上运行多个容器，但在不同主机上的好处。

我们还简要讨论了如何使用 Monit 和 Sensu 等工具监控 LXC 容器的状态、健康状况和资源利用率，并如何触发操作，如运行脚本来响应这些警报。

最后，我们回顾了一种常见的自动扩展模式，通过结合多个工具，根据警报事件自动创建新容器。

在下一章中，我们将查看一个完整的 OpenStack 部署，这将使我们能够利用智能调度器创建 LXC 容器。

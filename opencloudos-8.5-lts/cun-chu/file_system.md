# 第 1 章 文件系统概述

因为有着大量的选项以及其中所涉及的权衡，因此选择适合应用的文件系统是一个重要的决定。本章节讲述了OpenCloud OS 附带的一些文件系统，并且提供了适合应用的文件系统的相关历史背景以及建议。

## 1.1 文件系统类型

OpenCloud OS 支持各种类型的文件系统（File System）。不同类型的文件系统可以解决不同类型的问题，并且在某些特定的应用程序中他们的也会有不同的应用。在大多数情况下，文件系统的主要类型可分为以下几种：

**表 1.1 文件系统类型及其用例**

| 类型                | 文件系统          | 属性和使用案例                                                                                                                |
| ----------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------- |
| 磁盘/本地文件系统         | XFS           | XFS 是 OpenCloudOS 中默认使用的文件系统。因为它将文件布局扩展为数据块，所以它不像 ext4 那样容易受到文件碎片的影响。除非有兼容性或性能方面的相关问题方可部署为其它的文件系统，一般建议使用 XFS 文件系统。     |
|                   | ext4          | ext4 的优势是在 Linux 上使用时间长。因此，几乎所有的 Linux 应用都支持它。在大多数情况下，它和 XFS 在性能上相互竞争。ext4 通常都用于主目录。                                   |
| 网络/客户端和服务器之间的文件系统 | NFS           | 使用NFS在同一份网络的多个系统之间共享文件。                                                                                                |
|                   | SMB           | 使用SMB可以与 Windows 系统的文件进行共享                                                                                             |
| 共享存储/共享磁盘文件系统     | GFS2          | GFS2 为计算集群的成员提供共享写入的访问权限。其重点在于稳定性和可靠性，可以获得类似本地文件系统的体验。                                                                 |
| 卷管理文件系统           | Stratis（技术预览） | Stratis 是基于 XFS 和 LVM 构建的卷管理器。Stratis 的目的是模拟卷管理文件系统（如 Btrfs 和 ZFS）所提供的功能。可以手动构建此堆栈，但 Stratis 可以减少配置的复杂度、实施最佳实践并整合错误信息。 |

## 1.2 本地文件系统

本地文件系统是在本地单一服务器中运行并直接附加到存储的文件系统。

例如，本地文件系统是内部 SATA 或 SAS 磁盘的唯一选择，可在当您的服务器具有内部硬件 RAID 控制器（带有本地驱动器）时使用。当 SAN 上导出的设备未共享时，本地文件系统也是 SAN 连接的存储上最常用的文件系统。

所有本地文件系统都与 POSIX 兼容，且与所有支持的 OpenCloud OS 版本完全兼容。与 POSIX 兼容的文件系统为一组定义良好的系统调用提供支持，如 `read()`、`write()` 和 `seek()`。

从应用程序员的角度来看，本地文件系统之间的差别相对较少。从用户的角度来看，最显著的差异与可扩展性和性能相关。在考虑如何选择文件系统时，请考虑文件系统需要多大、应该具有哪些独特的功能，以及它在您的工作负载下性能如何。

**可用的本地文件系统**

- XFS
- ext4

## 1.3 XFS 文件系统

XFS 是一个高度可扩展、高性能、健壮且成熟的 64 位日志文件系统，其支持单个主机上非常大的文件和文件系统。XFS 最初于 1990 年代由 SGI 早期开发，并在大型的服务器和存储阵列中运行有很长的历史记录。

XFS 的功能包括：

**可靠性**

元数据日志，其确保系统崩溃后文件系统的完整性，方法是保留系统重启和重新挂载文件系统时可以重新执行的文件系统操作的记录，

- 广泛的运行时元数据一致性检查
- 可扩展且快速修复工具
- 配额日志。这可避免在崩溃后进行冗长的配额一致性检查。

**可伸缩性和性能**

- 支持最多 1024 TiB 的文件系统大小
- 支持大量并发操作的能力
- B-tree 索引，用于空闲空间的可扩展性管理
- 复杂的元数据读头算法
- 优化流视频工作负载

**分配方案**

- 基于扩展数据块的分配
- 条带化分配策略
- 延迟分配
- 空间预分配
- 动态分配的 inode

**其他功能**

- 基于 Reflink 的文件副本
- 严格集成备份和恢复工具
- 在线清理
- 在线文件系统增大
- 全面的诊断功能
- 扩展属性(`xattr`)。这允许系统能够按文件关联多个额外的名称/值对
- 项目或目录配额。这允许对目录树的配额限制
- 小于秒的时间戳

**性能特性**

XFS 在具有企业级工作负载的大型系统上具有高性能。大型系统是一个有相对较多的 CPU 、多个 HBA 和连接外部磁盘阵列的系统。XFS 在具有多线程、并行 I/O 工作负载的较小系统上也表现良好。

对于单线程、元数据密集型的工作负载，XFS 的性能相对较低：例如，在单个线程中创建或删除大量小文件的工作负载。

## 1.4 ext4文件系统

ext4 文件系统是 ext 文件系统系列的第四代。

ext4 驱动程序可以对 ext2 和 ext3 文件系统进行读写，但 ext4 文件系统格式又与 ext2 和 ext3 驱动程序是不兼容的。

ext4 添加了几个新的改进的功能，例如：

- 支持的文件系统大小高达 50 TiB
- 基于扩展的元数据
- 延迟分配
- 日志的 checksum
- 大型存储支持

基于扩展数据块的元数据和延迟分配功能提供了一种更加紧凑和高效的方法来跟踪文件系统中的已用空间。这些功能提高了文件系统性能，并减少了元数据所占用的空间。延迟分配允许文件系统延迟选择新写入用户数据的永久位置，直到数据刷新到磁盘。这可以实现更高的性能，因为它允许更大的、连续的分配，允许文件系统根据更佳的信息做出相应决策。

在 ext4 中使用 fsck 工具对文件系统的修复时间比在 ext2 和 ext3 中要快得多。一些文件系统修复的性能会增加最多 6 倍。

## 1.5 XFS 和 ext4 的比较

XFS 是 OpenClous OS 中的默认文件系统。本节比较 XFS 和 ext4 的用法和功能。

**元数据错误行为**

在 ext4 中，当文件系统遇到元数据错误时您可以配置行为。默认的行为是继续操作。当 XFS 遇到不可恢复的元数据错误时，它会关闭文件系统，并返回 `EFSCORRUPTED` 错误。

**配额**

在 ext4 中，您可以在创建文件系统时启用配额，或稍后在现有文件系统上启用配额。然后您可以使用挂载选项配置配额强制。

XFS 配额不是一个可重新挂载的选项。您必须在初始挂载中激活配额。

在 XFS 文件系统上运行 `quotacheck` 命令没有效果。当您第一次打开配额记帐时，XFS 会自动检查配额。

**文件系统重新定义大小**

XFS 没有工具来缩小文件系统的大小。您只能增大 XFS 文件系统的大小。而 ext4 支持扩展和缩小文件系统大小。

**内节点（inode）号**

ext4 文件系统不支持超过 2<sup>32</sup> 内节点。

XFS 动态分配内节点。只要文件系统上存在空闲空间，XFS 文件系统就无法耗尽 inode 。

某些应用程序无法正确处理 XFS 文件系统上大于 2<sup>32</sup> 的 inode 数。这些应用程序可能会导致 32 位 stat 调用失败，返回值为 EOVERFLOW 。在以下情况下，inode 数超过 2<sup>32</sup>:

- 文件系统大于 1 TiB，其 inode 为 256 字节。
- 文件系统大于 2 TiB，其 inode 为 512 字节。

如果您的应用程序由于 inode 数太大而失败，请使用 `-o inode32` 选项挂载 XFS 文件系统，来强制inode 数低于 2<sup>32</sup>。请注意，使用 `inode32` 不会影响已分配了 64 位数的 inode。

> **重要**
> 
> 除非特定环境需要，否则请勿使用 inode32 选项。inode32 选项可改变分配行为。因此，如果没有可用空间在较低磁盘块中分配 inode ，则可能会出现 ENOSPC 错误。

## 1.6 选择文件系统

要选择一个满足应用要求的文件系统，您需要对部署文件系统的目标系统有一定的了解。您通过以下问题来决定文件系统的类型：

- 您有大型服务器吗？
- 您有大的存储需求或者在本地使用的、读取速度慢的 SATA 驱动器吗？
- 您希望应用程序在哪一种 I/O 工作负载中？
- 您对吞吐量和延迟有要求吗？
- 您的服务器和存储硬件稳定性怎么样？
- 您的文件和数据组的默认大小是多少？
- 如果系统失败，您能接受的关机时间是多少？

如果您的服务器和存储设备都很大，那么 XFS 无疑是最佳选择。即使在存储阵列比较小，平均文件大小较大（例如，几百兆字节）的情况下，XFS 也依旧表现良好。

如果您现有的工作负载在 ext4 上表现良好，那么继续使用 ext4 会为您和您的应用提供一个舒适的环境。

ext4 文件系统在 I/O 能力有限的系统上往往表现会更好。ext4 在有限带宽（小于 200MB/s）上使用性能也会更好，最高可达到 1000 IOPS 。在能力较高的任何事情上，XFS 往往会比 ext4 更快。

与 ext4 相比，XFS 消耗的CPU元数据是 ext4 的两倍 ，因此如果您用很少并发的 CPU 绑定工作负载，则 ext4 会更快。通常情况下，如果应用使用单个读/写线程和小文件，则 ext4 效果更佳；而当应用使用多个读/写线程和较大的文件时，XFS 会更出色。

您无法对 XFS 文件系统进行缩小操作。如果您想要缩小文件系统，请考虑使用 ext4 ，其支持离线缩小。

通常，建议您使用 XFS，除非您有 ext4 的特殊用例。为了确保您选择到合适的文件系统，您还需要考虑到服务器和存储系统上特定应用的性能问题。

**表 1.2 本地文件系统建议概述**

| 场景                      | 推荐的文件系统 |
| ----------------------- | ------- |
| 没有特殊用例                  | XFS     |
| 大型服务器                   | XFS     |
| 大型存储设备                  | XFS     |
| 大文件                     | XFS     |
| 多线程 I/O                 | XFS     |
| 单线程 I/O                 | ext4    |
| 有限 I/O 能力（1000 IOPS 以下） | ext4    |
| 有限带宽（200 MB/s 以下）       | ext4    |
| CPU 绑定工作负载              | ext4    |
| 支持离线缩小                  | ext4    |

## 1.7 网络文件系统

网络文件系统也被称为客户端/服务器文件系统，使客户端能够访问存储在共享服务器上的文件。这可以实现多个系统上的多个用户可以共享文件和存储资源。

此类文件系统由一个或多个服务器中构建，这些服务器将一组文件系统导出到一个或多个客户端。客户端节点无法访问底层的块存储，而是使用允许更好的访问控制协议与存储进行交互。

**可用网络文件系统**

- OpenCloud OS 客户最常用的客户端/服务器文件系统就是 NFS 文件系统。OpenCloud OS 提供了一个 NFS 服务器组件来通过网络导出本地文件系统，并提供 NFS 客户端来导入这些文件系统。
- Open Cloud OS 还包括 CIFS 客户端，该客户端支持流行的 Microsoft SMB 文件服务器以实现 Windows 互操性。用户空间 Samba 服务器为 Windows 客户端提供来自 Open Cloud OS 服务器的 Microsoft SMB 服务。

## 1.8 共享存储文件系统

共享存储文件系统有时被称为集群文件系统，允许集群中的每台服务器通过本地的存储区域网络(SAN)直接访问共享块设备。

**和网络文件系统的比较**

与客户端/服务器文件系统一样，共享存储文件系统在一组服务器上工作，这些服务器都是集群的成员。但与 NFS 不同，没有一个服务器向其他成员提供对数据或元数据的访问：即集群的每个成员都可以直接访问同一存储设备（共享存储），并且所有群集节点都可以访问同一组文件。

**并发**

缓存一致性是集群文件系统中确保数据一致性和完整性的关键。集群中所有文件的单个版本都必须对集群内的所有节点可见。为了防止数据损坏，文件系统必须阻止集群成员同时更新同一存储块。为此，共享存储文件系统使用集群范围的锁机制作为并发控制机制来仲裁对存储的访问。例如，在创建新文件或写入多个服务器上打开的文件之前，服务器上的文件系统组件必须获得正确的锁。

集群文件系统的要求是提供一种像 Apache Web 服务器那样高可用的服务。集群中的任何成员都将看到存储在共享磁盘文件系统中的数据的完全一致的视图，并且所有更新都会通过锁机制进行正确的仲裁。

**性能特性**

由于锁开销的计算成本，共享磁盘文件系统并不总像运行在同一个系统上的本地文件系统那样表现良好。如果每个节点几乎都以独占方式写入一组不与其他节点共享的特定文件或者一组文件在一组节点上以几乎独占的只读方式被共享，那么共享磁盘文件系统可以很好地执行这种工作负载。这将导致最小的跨节点缓存失效，并可以最大限度地提高性能。

设置共享磁盘文件系统非常复杂，调优应用以在共享磁盘文件系统上表现良好非常有挑战性。

**可用的共享存储文件系统**

- 提供 GFS2 文件系统。GFS2 与 OpenCloud OS High Availability Add-On 和 Resilient Storage 附加组件紧密整合。

- 支持集群中 GFS2 的节点大小为 2 到 16 个。

## 1.9 网络和共享存储文件系统之间的抉择

在网络和共享存储文件系统之间选择时，请考虑以下几点：

- 对于提供 NFS 服务器的环境来说，基于 NFS 的网络文件系统是非常常见和流行的选择。

- 网络文件系统可以使用超高性能的网络技术（如 Infiniband 或 10 GB 以太网卡）来进行部署。这意味着您不应该仅仅为了获得存储的原始带宽而转向使用共享存储文件系统。如果访问速度至关重要，则使用 NFS 导出类似 XFS 的本地文件系统。

- 共享存储文件系统的设置或维护并非易事，因此您应该在本地或网络文件系统无法提供所需的可用性时才部署它们。

- 集群环境中的共享存储文件系统可以通过消除涉及到重新分配高可用性服务的典型故障切换情景中所需要执行的卸载和挂载步骤来减少停机时间。

除非您有共享存储文件系统的特定用例，否则还是建议您使用网络文件系统。共享存储文件系统主要用于需要以最少的停机时间提供高可用性服务且具有严格的服务等级要求的部署。

## 1.10 卷管理文件系统

为了实现堆栈内优化和简洁，卷管理文件系统集成了整个存储堆栈。

**可用卷管理文件系统**

OpenCloud OS 作为技术预览提供 Stratis 卷管理器。Stratis 对文件系统层使用 XFS，并将其与 LVM、设备映射器和其他组件集成。

Stratis 1.0 是一个直观的、基于命令行的卷管理器，可以在隐藏用户复杂性的同时执行重要的存储管理操作：

- 卷管理

- 创建池

- 精简存储池

- 快照

- 自动读取缓存

Stratis 提供强大的功能，但目前缺乏其他产品（如 Btrfs 或 ZFS）的某些功能。最值得注意的是，它不支持带自我修复的 CRC。

# 第 2 章 使用系统角色管理本地存储

要使用 Ansible 管理 LVM 和本地文件系统(FS)，您可以使用 Storage 角色，这是 OpenCloud OS 中可用的系统角色之一。

使用存储角色可让您自动管理多台机器上的磁盘和逻辑卷上的文件系统，以及系统的版本。

## 2.1 Storage （存储）角色简介

Storage 角色可以管理：

- 磁盘上未被分区的文件系统
- 完整的 LVM 卷组，包括其逻辑卷和文件系统

使用 Storage 角色，您可以执行以下任务：

- 创建文件系统
- 删除文件系统
- 挂载文件系统
- 卸载文件系统
- 创建 LVM 卷组
- 删除 LVM 卷组
- 创建逻辑卷
- 删除逻辑卷
- 创建 RAID 卷
- 删除 RAID 卷
- 创建带有 RAID 的 LVM 池
- 删除带有 RAID 的 LVM 池

## 2.2 识别存储设备参数（存储系统角色）

您的存储角色配置仅影响您在以下变量中列出的文件系统、卷和池.

`storage_volumes`

在所有要管理的未分区磁盘中的文件系统列表。

当前不支持的分区。

`storage_pools`

要管理的池列表。

目前唯一支持的池类型是 LVM。使用 LVM 时，池代表卷组（VG）。每个池中都有一个要由角色管理的卷列表。对于 LVM，每个卷对应一个带文件系统的逻辑卷（LV）。

## 2.3 在块存储设备中创建XFS 文件系统的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色，来使用默认参数在块设备上创建 XFS 文件系统。

> **警告**
> 
> 存储角色只能在未分区、整个磁盘或逻辑卷（LV）上创建文件系统。它不能在分区中创建文件系统。

**例 2.1 在/dev/sdb上创建 XFS 的playbook**

```
---
- hosts: all
  vars:
    storage_volumes:
      - name: barefs
        type: disk
        disks:
          - sdb
        fs_type: xfs
  roles:
    - rhel-system-roles.storage
```

- 卷名称（示例中的 barefs ）目前是任意的。存储角色根据 disk: 属性下列出的磁盘设备来识别卷。
- 您可以省略 `fs_type: xfs` 行，因为 XFS 是 OpenCloud OS 中的默认文件系统。
- 要在 LV 上创建文件系统，请在 `disks: `属性下提供 LVM 设置，包括括起来的卷组。详情请参阅 管理逻辑卷 的 Ansible playbook 示例。

不要提供到 LV 设备的路径。

## 2.4 永久挂载系统文件的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色用来立即和永久挂载 XFS 文件系统。

**例 2.2 将 /dev/sdb 上的文件系统挂载到 /mnt/data 的 playbook**

```
---
- hosts: all
  vars:
    storage_volumes:
      - name: barefs
        type: disk
        disks:
          - sdb
        fs_type: xfs
        mount_point: /mnt/data
  roles:
    - rhel-system-roles.storage
```

- 此 playbook 将文件系统添加到 `/etc/fstab` 文件中，并立即挂载文件系统。
- 如果 `/dev/sdb` 设备上的文件系统或挂载点目录不存在，则 playbook 会创建它们。

## 2.5 管理逻辑卷的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色在卷组中创建 LVM 逻辑卷。

**例 2.3 在 myvg 卷组中创建 mylv 逻辑卷的 playbook**

```
- hosts: all
  vars:
    storage_pools:
      - name: myvg
        disks:
          - sda
          - sdb
          - sdc
        volumes:
          - name: mylv
            size: 2G
            fs_type: ext4
            mount_point: /mnt
  roles:
    - rhel-system-roles.storage
```

- `myvg` 卷组由以下磁盘组成：
  
  - `/dev/sda`
  - `/dev/sdb`
  - `/dev/sdc`

- 如果 `myvg` 卷组已存在，则 playbook 会将逻辑卷添加到卷组。

- 如果 `myvg` 卷组不存在，则 playbook 会创建它。

- playbook 在 `mylv` 逻辑卷上创建 Ext4 文件系统，并在 `/mnt` 上永久挂载文件系统。

## 2.5 启用在线快丢弃的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色用来挂载启用了在线块丢弃的 XFS 文件系统。

**例 2.4 一个 playbook，它在 `/mnt/data/` 上启用在线块丢弃功能**

```
---
- hosts: all
  vars:
    storage_volumes:
      - name: barefs
        type: disk
        disks:
          - sdb
        fs_type: xfs
        mount_point: /mnt/data
        mount_options: discard
  roles:
    - rhel-system-roles.storage
```

## 2.7 创建挂载 Ext4 文件系统的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色来创建和挂载 Ext4 文件系统。

**例 2.5 在 `/dev/sdb` 上创建 Ext4 并挂载到 `/mnt/data` 的 playbook**

```
---
- hosts: all
  vars:
    storage_volumes:
      - name: barefs
        type: disk
        disks:
          - sdb
        fs_type: ext4
        fs_label: label-name
        mount_point: /mnt/data
  roles:
    - rhel-system-roles.storage
```

- playbook 在 `/dev/sdb` 磁盘上创建文件系统。
- playbook 将文件系统永久挂载在 `/mnt/data` 目录。
- 文件系统的标签是 `label-name`。

## 2.8 创建和挂载 ext3 文件系统的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色来创建和挂载 Ext3 文件系统。

**例 2.6 在 `/dev/sdb` 上创建 Ext3 ，并将其挂载在 `/mnt/data` 的 playbook**

```
---
- hosts: all
  vars:
    storage_volumes:
      - name: barefs
        type: disk
        disks:
          - sdb
        fs_type: ext3
        fs_label: label-name
        mount_point: /mnt/data
  roles:
    - rhel-system-roles.storage
```

- playbook 在 `/dev/sdb` 磁盘上创建文件系统。
- playbook 将文件系统永久挂载在 `/mnt/data` 目录。
- 文件系统的标签是 `label-name`。

## 2.9 使用存储 OpenCloud OS系统角色对现有 Ext4 或 Ext3 文件系统调整大小的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色来调整块设备上现有的 Ext4 或 Ext3 文件系统的大小。

**例 2.7 在磁盘上设置单个卷的 playbook**

```
---
- name: Create a disk device mounted on /opt/barefs
- hosts: all
  vars:
    storage_volumes:
      - name: barefs
        type: disk
        disks:
          - /dev/sdb
    size: 12 GiB
        fs_type: ext4
        mount_point: /opt/barefs
  roles:
    - rhel-system-roles.storage
```

- 如果上例中的卷已存在，要调整卷的大小，您需要运行相同的 playbook，使用不同的 `size` 参数值。例如：

**例 2.8 在 `/dev/sdb`上调整 `ext4` 大小的 playbook**
opencloud

```
---
- name: Create a disk device mounted on /opt/barefs
- hosts: all
  vars:
    storage_volumes:
      - name: barefs
        type: disk
        disks:
          - /dev/sdb
    size: 10 GiB
        fs_type: ext4
        mount_point: /opt/barefs
  roles:
    - rhel-system-roles.storage
```

- 卷名称（示例中的 barefs）当前是任意的。Storage 角色根据 disk: 属性中列出的磁盘设备标识卷。

> **注意**
> 
> 在其他文件系统中使用 调整大小 操作可能会破坏您正在使用的设备上的数据

## 2.10 使用存储 OpenCloud OS 系统角色在 LVM 上对现有文件系统的大小进行调整的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储 OpenCloud OS 系统角色来使用文件系统重新定义 LVM 逻辑卷大小。

> **警告**
> 
> 在其他文件系统中使用 调整大小 操作可能会破坏您正在使用的设备上的数据。

**例 2.9 调整 myvg 卷组中现有 mylv1 和 myvl2 逻辑卷大小的 playbook**

```
---

- hosts: all
   vars:
    storage_pools:
      - name: myvg
        disks:
          - /dev/sda
          - /dev/sdb
          - /dev/sdc
        volumes:
            - name: mylv1
              size: 10 GiB
              fs_type: ext4
              mount_point: /opt/mount1
            - name: mylv2
              size: 50 GiB
              fs_type: ext4
              mount_point: /opt/mount2

- name: Create LVM pool over three disks
  incude_role:
    name: rhel-system-roles.storage
```

- 此 playbook 调整以下现有文件系统的大小：
  
  - `mylv1` 卷上的 Ext4 文件系统挂载在 `/opt/mount1`，大小调整为 10 GiB。
  - `mylv2` 卷上的 Ext4 文件系统挂载在 `/opt/mount2`，大小调整为 50 GiB。

## 2.11 使用存储 OpenCloud OS 系统角色创建交换分区的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储角色来创建交换分区（如果不存在的话），或者使用默认参数在块设备上修改交换分区（如果已存在的话）。

**例 2.10 创建或修改 `/dev/sdb` 上现有 XFS 的 playbook**

```
name: Create a disk device with swap
hosts: all
vars:
  storage_volumes:```
- name: swap_fs
  type: disk
  disks:
    - /dev/sdb
size: 15 GiB
 fs_type: swap
roles:
- rhel-system-roles.storage 
```

- 卷名称（示例中的 `swap_fs` ）目前是任意的。存储角色根据 `disk:` 属性下列出的磁盘设备来识别卷。

## 2.12 使用存储系统角色配置 RAID 卷

使用存储系统角色，您可以使用  Ansible Automation Platform 在 OpenCloud OS 上配置 RAID 卷。在本小节中，您将了解如何使用可用参数设置 Ansible playbook 来配置 RAID 卷以满足您的要求。

**前提条件**

- Ansible Core 软件包安装在控制机器上。
- 您已在要运行 playbook 的系统上安装了 `rhel-system-roles` 软件包。
- 您有一个清单文件详细描述了您要使用存储系统角色部署 RAID 卷的系统。

**流程**

1. 使用以下内容创建一个新的 *`playbook.yml`* 文件：
   
   ```
   - hosts: all
    vars:
        storage_safe_mode: false
        storage_volumes:
         - name: data
           type: raid
           disks: [sdd, sde, sdf, sdg]
           raid_level: raid0
           raid_chunk_size: 32 KiB
           mount_point: /mnt/data
           state: present
    roles:
        - name: rhel-system-roles.storage
   ```
   
   > **警告**
   > 
   > 设备名称在某些情况下可能会改变，例如：当您在系统中添加新磁盘时。因此，为了避免数据丢失,我们不建议在 playbook 中使用特定的磁盘名称。

2. 可选。验证 playbook 语法。
   
   ```
    # ansible-playbook --syntax-check playbook.yml*
   ```

3. 在清单文件上运行 playbook:
   
   ```
   # ansible-playbook -i inventory.file /path/to/file/playbook.yml
   ```

## 2.13 使用存储系统角色使用 RAID 配置 LVM 池

使用 Storage 系统角色，您可以使用 Ansible Automation Platform 在 OpenCloud OS 上使用 RAID 配置 LVM 池。在本小节中，您将了解如何使用可用参数设置 Ansible playbook 来配置使用 RAID 的 LVM 池。

**前提条件**

- Ansible Core 软件包安装在控制机器上。
- 您已在要运行 playbook 的系统上安装了 `rhel-system-roles` 软件包。
- 您有一个清单文件详细描述了您要使用存储系统角色在其上配置带有 RAID 的 LVM 池的系统。

**流程**

1. 使用以下内容创建一个新的 *`playbook.yml`* 文件：
   
   ```
   - hosts: all
   vars:
       storage_safe_mode: false
       storage_pools:
       - name: my_pool
           type: lvm
           disks: [sdh, sdi]
           raid_level: raid1
           volumes:
           - name: my_pool
               size: "1 GiB"
               mount_point: "/mnt/app/shared"
               fs_type: xfs
               state: present
   roles:
       - name: rhel-system-roles.storage
   ```

> **注意**
> 
> 要创建带有 RAID 的 LVM 池，您必须使用 raid_level 参数指定 RAID 类型。

2. 可选。验证 playbook 语法。
   
   ```
   # ansible-playbook --syntax-check playbook.yml
   ```

3. 在清单文件上运行 playbook:
   
   ```
   # ansible-playbook -i inventory.file /path/to/file/playbook.yml
   ```

## 2.14 使用存储 OpenClous OS 系统角色在 LVM 上压缩和删除数据重复的 VDO 卷的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储 OpenCloud OS 系统角色来使用虚拟数据优化(VDO)，让卷对逻辑管理器卷(LVM)的进行压缩和删除重复数据操作。

**例 2.11 在 `myvg` 卷组中创建 `mylv1` LVM VDO 卷的 playbook**

```
---
- name: Create LVM VDO volume under volume group 'myvg'
  hosts: all
  roles:
    -opencloudos-system-roles.storage
  vars:
    storage_pools:
     - name: myvg
       disks:
         - /dev/sdb
       volumes:
         - name: mylv1
           compression: true
           deduplication: true
           vdo_pool_size: 10 GiB
           size: 30 GiB
           mount_point: /mnt/app/shared
```

在本例中，`compression` 和 `deduplication` 都被设置为 true，这里指定使用 VDO。下面描述了这些参数的用法：

- `deduplication` 用于删除存储在存储卷上重复的数据。
- `compression` 用于压缩存储卷上存储的数据，从而增加存储容量。
- vdo_pool_size 指定卷在设备上的实际大小。VDO 卷的虚拟大小由 `size` 参数设置。注：由于 Storage 角色使用 LVM VDO，每个池只有一个卷可以使用压缩和删除重复数据。

## 2.15 使用使用存储系统角色创建 LUKS 加密卷

您可以使用存储角色运行 Ansible playbook 来创建和配置那些使用 LUKS 加密的卷。

**前提条件**

- 对一个或多个 *受管节点* 的访问和权限，*受管节点* 是您要使用加密策略系统角色配置的系统。

- 对控制节点的访问和权限，这是 Ansible Core 配置其他系统的系统。

- 在控制节点上：
  
  - `ansible-core` 和 `rhel-system-roles` 软件包已安装 。

> **重要**
> 
> OpenCloud OS 提供对基于 Ansible 的自动化需要 Ansible Engine 的独立 Ansible 存储库的访问权限。Ansible Engine 包含命令行实用程序，如 ansible、ansible-playbook、连接器（如 docker 和 podman ）以及许多插件和模块。
> 
> OpenCloud OS 引入了 Ansible Core（以 ansible-core 软件包的形式提供），其中包含 Ansible 命令行工具、命令以及小型内置 Ansible 插件。

- 列出受管节点的清单文件。

**流程**

1. 使用以下内容创建一个新的 *`playbook.yml`* 文件：
   
   ```
   - hosts: all
   vars:
       storage_volumes:
       - name: barefs
           type: disk
           disks:
           - sdb
           fs_type: xfs
           fs_label: label-name
           mount_point: /mnt/data
           encryption: true
           encryption_password: your-password
   roles:
   - rhel-system-roles.storage
   ```

2. 可选：验证 playbook 语法：
   
   ```
   # ansible-playbook --syntax-check playbook.yml
   ```

3. 在清单文件上运行 playbook:
   
   ```
   # ansible-playbook -i inventory.file /path/to/file/playbook.yml
   ```

## 2.16 使用存储 OpenCloud OS 系统角色以百分比形式表示池卷大小的 Ansible playbook 示例

本节提供了一个 Ansible playbook 示例。此 playbook 应用存储系统角色来通过存储池百分比的形式表示逻辑卷管理卷(LVM)的大小。

**例 2.12 将卷大小表示为池容量百分比的 playbook**

```
---
- name: Express volume sizes as a percentage of the pool's total size
  hosts: all
  roles
    - rhel-system-roles.storage
  vars:
    storage_pools:
    - name: myvg
      disks:
        - /dev/sdb
      volumes:
        - name: data
          size: 60%
          mount_point: /opt/mount/data
        - name: web
          size: 30%
          mount_point: /opt/mount/web
        - name: cache
          size: 10%
          mount_point: /opt/cache/mount
```

这个示例将 LVM 卷的大小以存储池容量的百分比表示，例如："60%"。另外，您还可以将 LVM 卷的大小以文件系统中人类可读的类型（如 "10g" 或 "50 GiB"）使用存储池容量的百分比表示。

# 第 3 章 挂载 NFS 共享

作为系统管理员，您可以在您的系统上挂载远程 NFS 共享来访问共享数据。

## 3.1 NFS 简介

这部分解释了 NFS 服务的基本概念。

网络文件系统(NFS)允许远程主机通过网络挂载文件系统，并像挂载在本地那样与文件系统进行交互。这可以让您将资源整合到网络的集中服务器中。

NFS 服务器参考 `/etc/exports` 配置文件来确定是否允许客户端访问任何导出的文件系统。一旦被验证，那么所有文件和目录操作都对用户有效。

## 3.2 支持的 NFS 版本

这部分列出了 OpenCloud OS 支持的 NFS 版本及其特性。

目前，OpenCloud OS 8 支持以下 NFS 主版本：

- 与 NFSv2 相比，NFS 版本 3（NFSv3）支持安全异步写入操作，并在处理错误时更加可靠。同时它也支持 64 位文件和偏移，允许客户端访问的文件数据超过 2 GB 。

- NFS 版本 4(NFSv4)可以越过防火墙在 Internet 上工作，这以为着不再需要 rpcbind 服务。同时 NFSv4 支持访问控制列表(ACL)，并且可以使用有状态操作。

不再支持 NFS 版本 2(NFSv2)。

### 默认 NFS 版本

OpenCloud OS 8 中默认 NFS 版本为 4.2。NFS 客户端默认尝试使用 NFSv4.2 挂载，并在服务器不支持 NFSv4.2 时回退到 NFSv4.1。挂载之后会返回到 NFSv4.0，然后回退到 NFSv3。

### 次要 NFS 版本的特性

以下是 OpenCloud OS 8 中的 NFSv4.2 的功能：

**服务器端复制**

使用 `copy_file_range()` 调用系统，可以使 NFS 客户端高效地复制数据，而不至于浪费网络资源。

**稀疏文件**

使文件存在一个或者多个空位，其中存在只由 0 组成的未分配或者未初始化的数据块。NFSv4.2 中的 `lseek()` 操作支持 `seek_hole()` 和 `seek_data()`，使应用程序能够在稀疏文件中映射出漏洞的位置。

**保留空间**

允许存储服务器保留空闲空间防止服务器耗尽空间。NFSv4.2 支持 `allocate()` 操作来保留空间，支持 `deallocate()` 操作来释放空间，支持 `fallocate()` 操作来预分配和释放文件中的空间。

**标记的 NFS**

强制实施数据访问权限，并在客户端和服务器之间 NFS 文件系统上的各个文件中启用 SELinux 标签。

**布局增强**

提供 `layoutstats()` 操作，它可以让使并行 NFS(pNFS)服务器收集到更好的性能统计数据。

 NFSv4.1 的功能如下：

- 增强性能和网络安全，同时支持 pNFS 的客户端。

- 对于回调不再需要单独的 TCP 连接，回调允许 NFS 服务器在无法联系客户端的情况下授予委托：例如， NAT 或防火墙干扰。

- 只提供一次命令（除重启操作外），防止出现先前的问题，即如果回复丢失且操作被发送了两次，则某些操作有时会返回不准确的结果。

## 3.3 NFS 所需的服务

这部分列出了运行 NFS 服务器或挂载 NFS 共享所需的系统服务。Opencloud OS 会自动启动这些服务。

Opencloud OS 使用内核级支持和服务流程组合提供 NFS 文件共享。所有 NFS 版本都依赖于客户端和服务器间的远程过程调用（RPC）。要共享或者挂载 NFS 文件系统，下列服务根据所使用的 NFS 版本而定：

**`nfsd`**

为共享 NFS 文件系统请求的 NFS 服务器内核模块。

**`rpcbind`**

接受本地 RPC 服务保留的端口。这些端口随后可用（或公布出去），以便相应的远程 RPC 服务访问。`rpcbind` 服务对 RPC 服务的请求进行响应，并与请求的 RPC 服务建立连接。这不能与 NFSv4 一起使用。

**`rpc.mountd`**

NFS 服务器使用这个进程处理来自 NFSv3 客户端的 `MOUNT` 请求。它检查请求的 NFS 共享是否由目前的 NFS 服务器导出，并且允许客户端访问它。如果允许挂载请求，`nfs-mountd` 服务会回复 Success 状态，并将此 NFS 共享的文件句柄返回给 NFS 客户端。

**`rpc.nfsd`**

这个过程允许定义服务器通告的显式 NFS 版本和协议。它与 Linux 内核一起工作来满足 NFS 客户端的动态需求。例如，在每次连接 NFS 客户端时提供服务器线程。这个进程对应于 `nfs-server` 服务。

**`lockd`**

这是一个在客户端和服务器中运行的内核线程。它实现了网络锁管理器(NLM)协议，该协议使 NFSv3 客户端能够锁住服务器上的文件。每当运行 NFS 服务器以及挂载 NFS 文件系统时，它会自动启动。

**`rpc.statd`**

这个进程实现网络状态监控器(NSM)RPC 协议，该协议可在 NFS 服务器没有正常关机而重启时通知 NFS 客户端。`rpc-statd` 服务由 `nfs-server` 服务自动启动，并且此服务不需要用户配置。这不能与 NFSv4 一起使用。

**`rpc.rquotad`**

这个过程为远程用户提供用户的配额信息。启动 `nfs-server` 时，用户也必须启动 `quota-rpc` 软件包提供的 `rpc-rquotad` 服务。

**`rpc.idmapd`**

此进程提供 NFSv4 客户端和服务器调用， 它们在线上 NFSv4 名称（`user@domain`形式的字符串）、本地 UID 、 GID 之间进行映射。要使 `idmapd` 与 NFSv4 正常工作，必须配置 `/etc/idmapd.conf` 文件。至少应该指定 `Domain` 参数，该参数定义了 NFSv4 的映射域。如果 NFSv4 的映射域与 DNS 域名相同，则可以跳过该参数。客户端和服务器就 NFSv4 的映射域达成一致，ID 映射才能正常工作。

只有 NFSv4 服务器使用 `rpc.idmapd`，它由 `nfs-idmapd` 服务启动。NFSv4 客户端使用基于密钥环的 `nfsidmap` 工具，内核按需调用它来执行 ID 映射。如果 `nfsidmap` 有问题，客户端将退回使用 `rpc.idmapd`。

### NFSv4 的 RPC 服务

挂载和锁定协议已合并到 NFSv4 协议中。该服务器还会侦听已知的 TCP 端口 2049。因此，NFSv4 不需要与 `rpcbind`、`lockd` 和 `rpc-statd` 服务进行交互。NFS 服务器上仍然需要 nfs-mountd 服务来设置导出，但不涉及任何线上操作。

## 3.4 NFS 主机名格式

这部分论述了在挂载或导出 NFS 共享时用来指定主机的不同格式。

您可以使用以下格式指定主机：

**单台机器**

  以下任意一种：

- 完全限定域名（可由服务器解析）
- 主机名（可由服务器解析）
- IP 地址。

**IP 网络**

  以下格式之一有效：

- `a.b.c.d/z` ，其中 `a.b.c.d` 是网络，`z` 是子网掩码中的位数，例如 `192.168.0.0/24`。
- `a.b.c.d/netmask`，其中 `a.b.c.d` 是网络，`netmask` 是子网掩码；例如 `192.168.100.8/255.255.255.0`。

**Netgroups**

`@group-name` 格式，其中 `group-name` 是 NIS netgroup 名称。

## 3.5 安装 NFS

这个过程安装挂载或导出 NFS 共享所需的所有软件包。

**流程**

- 安装 nfs-utils 软件包：
  
  ```
  # yum install nfs-utils
  ```

## 3.6  发现 NFS 导出

这个步骤发现给定 NFSv3 或者 NFSv4 服务器导出哪个文件系统。

**流程**

- 对于支持 NFSv3 的任何服务器，请使用 `showmount` 工具：
  
  ```
  $ showmount --exports my-server
  
  Export list for my-server
  /exports/foo
  /exports/bar
  ```

- 对于支持 NFSv4 的任何服务器，挂载根目录并查找：
  
  ```
  # mount my-server:/ /mnt/
  # ls /mnt/
  
  exports
  
  # ls /mnt/exports/
  
  foo
  bar
  ```

在同时支持 NFSv4 和 NFSv3 的服务器上，这两种方法都可以工作，并给出同样的结果。

## 3.7 使用 mount 挂载一个 NFS 共享

这个流程使用 mount 工具挂载从服务器导出的 NFS 共享。

**流程**

- 要挂载一个 NFS 共享，请使用以下命令：
  
  ```
  # mount -t nfs -o options host:/remote/export /local/directory
  ```

这个命令使用以下变量：

***选项***

以逗号分隔的挂载选项列表。

***host***

导出您要挂载的文件系统的服务器的主机名、IP 地址或者完全限定域名。

***/remote/export***

从服务器导出的文件系统或目录，即您要挂载的目录。

***/local/directory***

挂载 /remote/export 的客户端位置。

## 3.8 常用 NFS 挂载选项

这部分列出了在挂载 NFS 共享时常用的选项。这些选项可用于手动挂载命令、`/etc/fstab` 设置和 `autofs`。

**`lookupcache=mode`**

指定内核应该如何管理给定挂载点的目录条目缓存。*mode* 的有效参数为 `all`、`none` 或 `positive`。

**`nfsvers=version`**

指定要使用 NFS 协议的哪个版本，其中 *version* 为 `3`、`4`、`4.0`、`4.1` 或 `4.2`。这对于运行多个 NFS 服务器的主机很有用，或者禁止使用较低版本重试挂载。如果没有指定版本，NFS 将使用内核和 `mount` 工具支持的最高版本。

选项 `vers` 等同于 `nfsvers` ，出于兼容性的原因包含在此发行版本中。

**`noacl`**

关闭所有 ACL 处理。当与 Solaris 交互时，可能需要此功能，因为最新的 ACL 技术与较旧的系统不兼容。

**`nolock`**

禁用文件锁定。当连接到非常旧的 NFS 服务器时，有时需要这个设置。

**`noexec`**

防止在挂载的文件系统中执行二进制文件。这在系统挂载不兼容二进制文件的非 Linux 文件系统时有用。

**`nosuid`**

禁用 `set-user-identifier` 和 `set-group-identifier` 位。这可防止远程用户通过运行 `setuid` 程序获得更高的特权。

**`port=num`**

指定 NFS 服务器端口的数字值。如果 *num* 为 `0`（默认值），则 `mount` 查询远程主机上 `rpcbind` 服务，以获取要使用的端口号。如果远程主机上的 NFS 服务没有注册其 `rpcbind` 服务，则使用标准的 NFS 端口号 TCP 2049。

**`rsize=num` 和 `wsize=num`**

这些选项设定单一 NFS 读写操作传输的最大字节数。

`rsize` 和 `wsize` 没有固定的默认值。默认情况下，NFS 使用服务器和客户端都支持的最大的可能值。在 Opencloud OS 8 中，客户端和服务器最大为 1,048,576 字节。

**`sec=flavors`**

用于访问挂载导出文件的安全类别。*flavors* 值是一个冒号分隔的、由一个或多个安全类别组成的列表。

默认情况下，客户端会尝试查找客户端和服务器都支持的安全类别。如果服务器不支持任何选定的类别，挂载操作将失败。

可用类别：

- `sec=sys` 使用本地 UNIX UID 和 GID。它们使用 `AUTH_SYS` 验证 NFS 操作。
- `sec=krb5` 使用 Kerberos V5 ,而不是本地 UNIX UID 和 GID 来验证用户。
- `sec=krb5i` 使用 Kerberos V5 进行用户身份验证，并使用安全校验和执行 NFS 操作的完整性检查，以防止数据被篡改。
- `sec=krb5p` 使用 Kerberos V5 进行用户身份验证、完整性检查，并加密 NFS 流量以防止流量嗅探。这是最安全的设置，但它也会涉及最大的性能开销。

**`tcp`**

指示 NFS 挂载使用 TCP 协议。

# 第 4 章 导出 NFS 共享

作为系统管理员，您可以使用 NFS 服务器来通过网络共享系统上的目录。

## 4.1 NFS 简介

这部分解释了 NFS 服务的基本概念。

网络文件系统(NFS)允许远程主机通过网络挂载文件系统，并像它们挂载在本地那样与这些文件系统进行交互。这可让您将资源整合到网络的集中服务器中。

NFS 服务器参考 `/etc/exports` 配置文件来确定是否允许客户端访问任何导出的文件系统。一旦被验证，所有的文件和目录操作都对用户有效。

## 4.2 支持的 NFS 版本

这部分列出了 OpenCloud OS 支持的 NFS 版本及其特性。

目前，OpenCloud OS 8 支持以下 NFS 主版本：

- 与 NFSv2 相比，NFS 版本 3（NFSv3）支持安全异步写入操作，并在处理错误时更加可靠。同时它也支持 64 位文件和偏移，允许客户端访问的文件数据超过 2 GB 。

- NFS 版本 4(NFSv4)可以越过防火墙在 Internet 上工作，这以为着不再需要 rpcbind 服务。同时 NFSv4 支持访问控制列表(ACL)，并且可以使用有状态操作。

不再支持 NFS 版本 2(NFSv2)。

### 默认 NFS 版本

OpenCloud OS 8 中默认 NFS 版本为 4.2。NFS 客户端默认尝试使用 NFSv4.2 挂载，并在服务器不支持 NFSv4.2 时回退到 NFSv4.1。挂载之后会返回到 NFSv4.0，然后回退到 NFSv3。

### 次要 NFS 版本的特性

以下是 OpenCloud OS 8 中的 NFSv4.2 的功能：

**服务器端复制**

使用 `copy_file_range()` 调用系统，可以使 NFS 客户端高效地复制数据，而不至于浪费网络资源。

**稀疏文件**

使文件存在一个或者多个空位，其中存在只由 0 组成的未分配或者未初始化的数据块。NFSv4.2 中的 `lseek()` 操作支持 `seek_hole()` 和 `seek_data()`，使应用程序能够在稀疏文件中映射出漏洞的位置。

**保留空间**

允许存储服务器保留空闲空间防止服务器耗尽空间。NFSv4.2 支持 `allocate()` 操作来保留空间，支持 `deallocate()` 操作来释放空间，支持 `fallocate()` 操作来预分配和释放文件中的空间。

**标记的 NFS**

强制实施数据访问权限，并在客户端和服务器之间 NFS 文件系统上的各个文件中启用 SELinux 标签。

**布局增强**

提供 `layoutstats()` 操作，它可以让使并行 NFS(pNFS)服务器收集到更好的性能统计数据。

 NFSv4.1 的功能如下：

- 增强性能和网络安全，同时支持 pNFS 的客户端。

- 对于回调不再需要单独的 TCP 连接，回调允许 NFS 服务器在无法联系客户端的情况下授予委托：例如， NAT 或防火墙干扰。

- 只提供一次命令（除重启操作外），防止出现先前的问题，即如果回复丢失且操作被发送了两次，则某些操作有时会返回不准确的结果。

## 4.3 NFSv3 和 NFSv4 中的 TCP 和 UDP 协议

NFSv4 需要通过 IP 网络运行的传输控制协议（TCP）。

NFSv3 还可以使用早期 Linux 版本中的用户数据报协议(UDP)。在 Opencloud OS 8 中不再支持通过 UDP 的 NFS。默认情况下，UDP 在 NFS 服务器中被禁用。

## 4.4 NFS 所需的服务

这部分列出了运行 NFS 服务器或挂载 NFS 共享所需的系统服务。Opencloud OS 会自动启动这些服务。

Opencloud OS 使用内核级支持和服务流程组合提供 NFS 文件共享。所有 NFS 版本都依赖于客户端和服务器间的远程过程调用（RPC）。要共享或者挂载 NFS 文件系统，下列服务根据所使用的 NFS 版本而定：

**`nfsd`**

为共享 NFS 文件系统请求的 NFS 服务器内核模块。

**`rpcbind`**

接受本地 RPC 服务保留的端口。这些端口随后可用（或公布出去），以便相应的远程 RPC 服务访问。`rpcbind` 服务对 RPC 服务的请求进行响应，并与请求的 RPC 服务建立连接。这不能与 NFSv4 一起使用。

**`rpc.mountd`**

NFS 服务器使用这个进程处理来自 NFSv3 客户端的 `MOUNT` 请求。它检查请求的 NFS 共享是否由目前的 NFS 服务器导出，并且允许客户端访问它。如果允许挂载请求，`nfs-mountd` 服务会回复 Success 状态，并将此 NFS 共享的文件句柄返回给 NFS 客户端。

**`rpc.nfsd`**

这个过程允许定义服务器通告的显式 NFS 版本和协议。它与 Linux 内核一起工作来满足 NFS 客户端的动态需求。例如，在每次连接 NFS 客户端时提供服务器线程。这个进程对应于 `nfs-server` 服务。

**`lockd`**

这是一个在客户端和服务器中运行的内核线程。它实现了网络锁管理器(NLM)协议，该协议使 NFSv3 客户端能够锁住服务器上的文件。每当运行 NFS 服务器以及挂载 NFS 文件系统时，它会自动启动。

**`rpc.statd`**

这个进程实现网络状态监控器(NSM)RPC 协议，该协议可在 NFS 服务器没有正常关机而重启时通知 NFS 客户端。`rpc-statd` 服务由 `nfs-server` 服务自动启动，并且此服务不需要用户配置。这不能与 NFSv4 一起使用。

**`rpc.rquotad`**

这个过程为远程用户提供用户的配额信息。启动 `nfs-server` 时，用户也必须启动 `quota-rpc` 软件包提供的 `rpc-rquotad` 服务。

**`rpc.idmapd`**

此进程提供 NFSv4 客户端和服务器调用， 它们在线上 NFSv4 名称（`user@domain`形式的字符串）、本地 UID 、 GID 之间进行映射。要使 `idmapd` 与 NFSv4 正常工作，必须配置 `/etc/idmapd.conf` 文件。至少应该指定 `Domain` 参数，该参数定义了 NFSv4 的映射域。如果 NFSv4 的映射域与 DNS 域名相同，则可以跳过该参数。客户端和服务器就 NFSv4 的映射域达成一致，ID 映射才能正常工作。

只有 NFSv4 服务器使用 `rpc.idmapd`，它由 `nfs-idmapd` 服务启动。NFSv4 客户端使用基于密钥环的 `nfsidmap` 工具，内核按需调用它来执行 ID 映射。如果 `nfsidmap` 有问题，客户端将退回使用 `rpc.idmapd`。

### NFSv4 的 RPC 服务

挂载和锁定协议已合并到 NFSv4 协议中。该服务器还会侦听已知的 TCP 端口 2049。因此，NFSv4 不需要与 `rpcbind`、`lockd` 和 `rpc-statd` 服务进行交互。NFS 服务器上仍然需要 nfs-mountd 服务来设置导出，但不涉及任何线上操作。

## 4.5 NFS 主机名格式

这部分论述了在挂载或导出 NFS 共享时用来指定主机的不同格式。

您可以使用以下格式指定主机：

**单台机器**

  以下任意一种：

- 完全限定域名（可由服务器解析）
- 主机名（可由服务器解析）
- IP 地址。

**IP 网络**

  以下格式之一有效：

- `a.b.c.d/z` ，其中 `a.b.c.d` 是网络，`z` 是子网掩码中的位数，例如 `192.168.0.0/24`。
- `a.b.c.d/netmask`，其中 `a.b.c.d` 是网络，`netmask` 是子网掩码；例如 `192.168.100.8/255.255.255.0`。

**Netgroups**

  `@group-name` 格式，其中 `group-name` 是 NIS netgroup 名称。

## 4.6 NFS 服务器配置

这部分论述了在 NFS 服务器中配置导出的语法和选项：

- 手动编辑 /etc/exports 配置文件
- 在命令行上使用 exportfs 工具

### 4.6.1 /etc/exports 配置文件

`/etc/exports` 文件控制哪些文件系统被导出到远程主机，并指定选项。它遵循以下语法规则：

- 空白行将被忽略。
- 要添加注释一行，以井号(`#`)开头。
- 您可以使用反斜杠(`\`)换行长行。
- 每个导出的文件系统都应该独立。
- 所有在导出的文件系统后放置的授权主机列表都必须用空格分开。 
- 每个主机的选项必须在主机标识符后直接放在括号中，没有空格分离主机和第一个括号。

#### 导出条目

导出的文件系统的每个条目都有以下结构：

```
export host(options)
```

您还可以指定多个主机以及每个主机的特定选项。要做到这一点，在同一行中列出主机列表（以空格分隔），每个主机名带有其相关的选项（在括号中），如下所示：

```
export host1(options1) host2(options2) host3(options3)
```

在这个结构中：

***export***

  导出的目录

***host***

  导出要共享的主机或网络

***options***

  用于主机的选项

**例 4.1 一个简单的 /etc/exports 文件**

在最简单的形式中，`/etc/exports` 文件只指定导出的目录和允许访问它的主机：

```
/exported/directory bob.example.com
```

`bob.example.com` 可以挂载 NFS 服务器的 `/exported/directory/`。因为在这个示例中没有指定选项，所以 NFS 使用默认选项。

> **重要**
> 
> `/etc/exports` 文件的格式要求非常精确，特别是在空格字符的使用方面。需要将导出的文件系统与主机、不同主机间使用空格分隔。但是，除了注释行外，文件中不应该包括其他空格。
> 
> 例如，下面两行并不具有相同的意义：
> 
> ```
> /home bob.example.com(rw)
> /home bob.example.com (rw)
> ```
> 
> 第一行仅允许来自 `bob.example.com` 的用户读写 `/home` 目录。第二行允许来自 `bob.example.com` 的用户以只读方式挂载目录（默认），而其他用户可以将其挂载为读/写。

#### 默认选项

导出条目的默认选项有：

**`ro`**

  导出的文件系统是只读的。远程主机无法更改文件系统中共享的数据。要允许主机更改文件系统（即读写），指定 rw 选项。

**`sync`**

  在将之前的请求所做的更改写入磁盘前，NFS 服务器不会回复请求。要启用异步写，请指定 `async` 选项。

**`wdelay`**

  如果 NFS 服务器预期另外一个写入请求即将发出，则 NFS 服务器会延迟写入磁盘。这可以提高性能，因为它可减少不同的命令写入访问磁盘的次数，从而减少写入开销。要禁用此功能，请指定 `no_wdelay` 选项，该选项仅在指定了默认 sync 选项时才可用。

**`root_squash`**

  这可以防止远程连接的 root 用户（与本地连接相反）具有 root 特权；相反，NFS 服务器为他们分配用户 ID `nobody` 。这可以有效地将远程 root 用户的权限"挤压"成最低的本地用户，从而防止在远程服务器上可能的未经授权的写操作。要禁用 root 挤压，请指定 `no_root_squash` 选项。

  要挤压每个远程用户（包括 root 用户），请使用 `all_squash` 选项。要指定 NFS 服务器应该分配给来自特定主机的远程用户的用户和组 ID，请分别使用 `anonuid` 和 `anongid` 选项，如下所示：

```
export host(anonuid=uid,anongid=gid)
```

  这里，uid 和 gid 分别是用户 ID 号和组 ID 号。anonuid 和 anongid 选项允许您为要共享的远程 NFS 用户创建特殊的用户和组帐户。

默认情况下，Opencloud OS 下的 NFS 支持访问控制列表(ACL)。要禁用此功能，请在导出文件系统时指定 no_acl 选项。

#### 默认和覆盖选项

每个导出的文件系统的默认值都必须被显式覆盖。例如，如果没有指定 rw 选项，则导出的文件系统将以只读形式共享。以下是 /etc/exports 中的示例行，其覆盖两个默认选项：

```/another/exported/directory 192.168.0.3(rw,async)```

在此示例中，`192.168.0.3` 可以以读写的形式挂载 `/another/exported/directory/` ，并且所有对磁盘的写入都是异步的。

### 4.6.2 exportfs 工具

`exportfs` 工具使 root 用户能够有选择地导出或取消导出目录，而无需对 NFS 服务进行重启。给定合适的选项后，`exportfs` 工具将导出的文件系统写到 `/var/lib/nfs/xtab`。由于 `nfs-mountd` 服务在决定访问文件系统的特权时参考的是 `xtab` 文件，所以对导出的文件系统列表所做的更改会立即生效。

#### 常用的 exportfs 选项

以下是 `exportfs` 的常用选项：

**`-r`**

  通过在 `/var/lib/nfs/etab` 中构建出新的导出列表，可以将 `/etc/exports` 中列出的所有目录导出。如果对 `/etc/exports` 做了任何更改，那么这个选项可以有效地刷新导出列表。

**`-a`**

  根据传给了 `exportfs`的那些其他选项，将导出或取消导出所有目录。如果没有指定其他选项，`exportfs` 会导出 `/etc/exports` 中指定的所有文件系统。

**`-o file-systems`**

  指定没有在 `/etc/exports` 中列出的要导出的目录。使用要导出的额外文件系统替换 *file-systems*。这些文件系统的格式化方式必须与 `/etc/exports` 中指定的方式相同。此选项通常用于在将其永久添加到导出的文件系统列表之前测试导出的文件系统。

**`-i`**

  忽略 `/etc/exports` ；只有命令行上指定的选项才会用于定义导出的文件系统。

**`-u`**

  取消导出所有的共享目录。命令 exportfs -ua 可暂停 NFS 文件共享，同时保持所有 NFS 服务正常运行。要重新启用 NFS 共享，请使用 exportfs -r。

**`-v`**

  详细操作，当执行 exportfs 命令时，更详细地显示正在导出的或取消导出的文件系统。

如果没有选项传给 exportfs 工具，它将显示当前导出的文件系统列表。

## 4.7 NFS 和 rpcbind

本节解释 NFSv3 所需的 `rpcbind` 服务的用途。

`rpcbind` 服务将远程过程调用(RPC)服务映射到其侦听的端口。RPC 进程在启动时通知 `rpcbind`，注册它们正在侦听的端口以及它们期望提供服务的 RPC 程序号。然后，客户端系统会使用特定的 RPC 程序号联系服务器上的 `rpcbind`。`rpcbind` 服务将客户端重定向到正确的端口号，这样它就可以与请求的服务进行通信。

由于基于 RPC 的服务依赖 `rpcbind` 来与所有传入的客户端请求建立连接，因此 `rpcbind` 必须在这些服务启动之前可用。

`rpcbind` 的访问控制规则会影响所有基于 RPC 的服务。另外，也可以为每个 NFS 的 RPC 守护进程指定访问控制规则。

## 4.8 安装 NFS

这个过程中安装挂载/导出 NFS 共享所需的所有软件包。

**流程**

- 安装 nfs-utils 软件包：
  
  ```
  # yum install nfs-utils
  ```

## 4.9 启动 NFS 服务器

这个步骤描述了如何启动 NFS 服务器,这是导出 NFS 共享所必需的步骤。

**前提条件**

- 对于支持 NFSv3 连接的服务器，`rpcbind` 服务必须处于运行状态。要验证 `rpcbind` 是否处于活动状态，请使用以下命令：
  
  ```
  $ systemctl status rpcbind
  ```
  
  如果停止该服务，启动并启用该服务：
  
  ```
  $ systemctl enable --now rpcbind
  ```

**流程**

- 要启动 NFS 服务器，并使其在引导时自动启动，请使用以下命令：
  
  ```
  # systemctl enable --now nfs-server
  ```

## 4.10 NFS 和 rpcbind 故障排除

由于 `rpcbind` 服务在 RPC 服务和用于与之通信的端口号之间提供协调，因此在排除故障时，使用 `rpcbind` 查看当前 RPC 服务的状态非常有用。`rpcinfo` 工具显示每个基于 RPC 的服务，以及其端口号、RPC 程序号、版本号和 IP 协议类型（TCP 或 UDP）。

**流程**

1. 要确保为 rpcbind 启用了正确的基于 NFS RPC 的服务，请使用以下命令：
   
   ```
   # rpcinfo -p
   ```
   
   **例 4.2. rpcinfo -p 命令输出**
   
   下面是一个这个命令的输出示例：
   
   ```
   program vers proto   port  service
   100000    4   tcp    111  portmapper
   100000    3   tcp    111  portmapper
   100000    2   tcp    111  portmapper
   100000    4   udp    111  portmapper
   100000    3   udp    111  portmapper
   100000    2   udp    111  portmapper
   100005    1   udp  20048  mountd
   100005    1   tcp  20048  mountd
   100005    2   udp  20048  mountd
   100005    2   tcp  20048  mountd
   100005    3   udp  20048  mountd
   100005    3   tcp  20048  mountd
   100024    1   udp  37769  status
   100024    1   tcp  49349  status
   100003    3   tcp   2049  nfs
   100003    4   tcp   2049  nfs
   100227    3   tcp   2049  nfs_acl
   100021    1   udp  56691  nlockmgr
   100021    3   udp  56691  nlockmgr
   100021    4   udp  56691  nlockmgr
   100021    1   tcp  46193  nlockmgr
   100021    3   tcp  46193  nlockmgr
   100021    4   tcp  46193  nlockmgr
   ```
   
   如果其中一个 NFS 服务没有正确启动，`rpcbind` 将不能将来自客户端的对该服务的 RPC 请求映射到正确的端口。。

2. 在很多情况下，如果 NFS 没有出现在 rpcinfo 输出中，重启 NFS 会使服务正确使用 rpcbind 注册，并开始工作：
   
   ```
   # systemctl restart nfs-server
   ```

## 4.11 将 NFS 服务器配置在防火墙后运行

NFS 需要 `rpcbind` 服务，该服务为 RPC 服务动态分配端口，并可能导致配置防火墙规则时出现问题。下面的部分描述了如何在防火墙后配置 NFS 版本（如果要支持）：

- NFSv3
  
  这包括支持 NFSv3 的任何服务器：
  
  - NFSv3-only 服务器
  - 支持 NFSv3 和 NFSv4 的服务器

- 只使用 NFSv4

### 4.11.1 将 NFSv3-enabled 服务器配置在防火墙后运行

下面的步骤描述了如何将支持 NFSv3 的服务器配置为在防火墙后运行。这包括支持 NFSv3 和 NFSv4 的 NFSv3-only 服务器和服务器。

**流程**

1. 要允许客户端访问防火墙后面的 NFS 共享，请在 NFS 服务器上运行以下命令来配置防火墙：
   
   ```
   firewall-cmd --permanent --add-service mountd
   firewall-cmd --permanent --add-service rpc-bind
   firewall-cmd --permanent --add-service nfs
   ```

2. 指定 `/etc/nfs.conf` 文件中 RPC 服务 `nlockmgr` 使用的端口，如下所示：
   
   ```
   [lockd]
   
   port=tcp-port-number
   udp-port=udp-port-number
   ```
   
   或者，您可以在 `/etc/modprobe.d/lockd.conf` 文件中指定 `nlm_tcpport` 和 `nlm_udpport`。

3. 在 NFS 服务器中运行以下命令打开防火墙中指定的端口：
   
   ```
   firewall-cmd --permanent --add-port=<lockd-tcp-port>/tcp
   firewall-cmd --permanent --add-port=<lockd-udp-port>/udp
   ```

4. 通过编辑 `/etc/nfs.conf` 文件中的 `[statd]` 部分为 `rpc.statd` 添加静态端口，如下所示：
   
   ```
   [statd]
   
   port=port-number
   ```

5. 在 NFS 服务器中运行以下命令，在防火墙中打开添加的端口：
   
   ```
   firewall-cmd --permanent --add-port=<statd-tcp-port>/tcp
   firewall-cmd --permanent --add-port=<statd-udp-port>/udp
   ```

6. 重新载入防火墙配置：
   
   ```
   firewall-cmd --reload
   ```

7. 首先重启 `rpc-statd` 服务，然后重启 `nfs-server` 服务：
   
   ```
   # systemctl restart rpc-statd.service
   # systemctl restart nfs-server.service
   ```
   
   或者，如果您在 `/etc/modprobe.d/ lockd.conf` 文件中指定锁定端口：
   
   i.更新 `/proc/sys/fs/nfs/nlm_tcpport` 和 `/proc/sys/fs/nfs/nlm_udpport` 的当前值：
   
   ```
   # sysctl -w fs.nfs.nlm_tcpport=<tcp-port>
   # sysctl -w fs.nfs.nlm_udpport=<udp-port>
   ```
   
   ii. 重启 `rpc-statd` 和 `nfs-server` 服务：
   
   ```
   # systemctl restart rpc-statd.service
   # systemctl restart nfs-server.service 
   ```

### 4.11.2 将只使用 NFSv4 的服务器配置在防火墙后运行

下面的步骤描述了如何将只使用 NFSv4 的服务器配置为在防火墙后运行。

**流程**

1. 要允许客户端在防火墙后访问 NFS 共享，在 NFS 服务器中运行以下命令配置防火墙：
   
   ```
   firewall-cmd --permanent --add-service nfs
   ```

2. 重新载入防火墙配置：
   
   ```
   firewall-cmd --reload
   ```

3. 重启 nfs-server：
   
   ```
   # systemctl restart nfs-server
   ```

### 4.11.3 将 NFSv3 客户端配置为在防火墙后运行

将 NFSv3 客户端配置为在防火墙后运行的步骤类似于将 NFSv3 服务器配置为在防火墙后运行。

如果您要配置的机器既是 NFS 客户端也是服务器，请按照 [将 NFSv3-enabled 服务器配置在防火墙后运行](#4111-将-nfsv3-enabled-服务器配置在防火墙后运行) 中所述的步骤进行。

以下流程描述了如何配置只在防火墙后运行的 NFS 客户端的机器。

**流程**

1. 要在客户端位于防火墙后允许 NFS 客户端对 NFS 客户端的执行进行回调，请在 NFS 客户端上运行以下命令将 `rpc-bind` 服务添加到防火墙中：
   
   ```
   firewall-cmd --permanent --add-service rpc-bind
   ```

2. 指定 `/etc/nfs.conf` 文件中 RPC 服务 `nlockmgr` 使用的端口，如下所示：
   
   ```
   [lockd]
   
   port=port-number
   udp-port=upd-port-number
   ```
   
   或者，您可以在 `/etc/modprobe.d/lockd.conf` 文件中指定 `nlm_tcpport` 和 `nlm_udpport`。

3. 在 NFS 客户端中运行以下命令打开防火墙中指定的端口：
   
   ```
   firewall-cmd --permanent --add-port=<lockd-tcp-port>/tcp
   firewall-cmd --permanent --add-port=<lockd-udp-port>/udp
   ```

4. 通过编辑 `/etc/nfs.conf` 文件的 `[statd]` 部分为 `rpc.statd` 添加静态端口，如下所示：

```
[statd]

port=port-number
```

5. 在 NFS 客户端中运行以下命令，在防火墙中打开添加的端口：
   
   ```
   firewall-cmd --permanent --add-port=<statd-tcp-port>/tcp
   firewall-cmd --permanent --add-port=<statd-udp-port>/udp
   ```

6. 重新载入防火墙配置：
   
   ```
   firewall-cmd --reload
   ```

7. 重启 `rpc-statd` 服务：
   
   ```
   # systemctl restart rpc-statd.service
   ```
   
   或者，如果您在 `/etc/modprobe.d/ lockd.conf` 文件中指定锁定端口：
   
   i.更新 `/proc/sys/fs/nfs/nlm_tcpport` 和 `/proc/sys/fs/nfs/nlm_udpport` 的当前值：
   
   ```
   # sysctl -w fs.nfs.nlm_tcpport=<tcp-port>
   # sysctl -w fs.nfs.nlm_udpport=<udp-port>
   ```
   
   ii. 重启 rpc-statd 服务：
   
   ```
   # systemctl restart rpc-statd.service
   ```

### 4.11.4 将 NFSv4 客户端配置为在防火墙后运行

仅在客户端使用 NFSv4.0 时执行此步骤。在这种情况下，需要为 NFSv4.0 回调打开端口。

NFSv4.1 或更高版本不需要这个过程，因为在后续协议版本中，服务器在客户端发起的同一连接上执行回调。

**流程**

1. 要允许 NFSv4.0 通过防火墙回调，请设置 `/proc/sys/fs/nfs_callback_tcpport` 并允许服务器连接到客户端上的该端口，如下所示：
   
   ```
   # echo "fs.nfs.nfs_callback_tcpport = <callback-port>" >/etc/sysctl.d/90-nfs-callback-port.conf
   # sysctl -p /etc/sysctl.d/90-nfs-callback-port.conf
   ```

2. 在 NFS 客户端中运行以下命令打开防火墙中指定的端口：
   
   ```
   firewall-cmd --permanent --add-port=<callback-port>/tcp
   ```

3. 重新载入防火墙配置：
   
   ```
   firewall-cmd --reload
   ```

## 4.12 通过防火墙导出 RPC 配额

如果您导出使用磁盘配额的文件系统，那么您可以使用配额远程调用(RPC)服务来给 NFS 客户端提供磁盘的配额数据。

**流程**

1. 启用并启动 `rpc-rquotad` 服务：
   
   ```
   # systemctl enable --now rpc-rquotad
   ```
   
   > **注意**
   > 
   > 如果启用了 `rpc-rquotad` 服务，其会在启动 `nfs-server` 服务后自动启动。

2. 为了使配额 RPC 服务可在防火墙后访问，需要打开 TCP（如果启用了 UDP，则为 UDP）端口 875。默认端口号定义在 `/etc/services` 文件中。
   
   您可以通过将 `-p port-number` 附加到 `/etc/sysconfig/rpc-rquotad` 文件中的 `RPCRQUOTADOPTS` 变量来覆盖默认端口号。

3. 默认情况下，远程主机只能读取配额。如果允许客户端设置配额，请将 `-S` 选项附加到 `/etc/sysconfig/rpc-rquotad` 文件中的 `RPCRQUOTADOPTS` 变量中。

4. 重启 `rpc-rquotad` ，使 `/etc/sysconfig/rpc-rquotad` 文件中的更改生效：
   
   ```
   # systemctl restart rpc-rquotad
   ```

## 4.13 启用通过 RDMA(NFSoRDMA) 的 NFS

远程直接访问内存(RDMA)服务在 Opencloud OS 8 中启用了 RDMA 硬件实现自动工作。

**流程**

1. 安装 `rdma-core` 软件包：
   
   ```
   # yum install rdma-core
   ```

2. 在 `/etc/rdma/modules/rdma 文件中验证带有 xprtrdma 和 svcrdma` 的行已注释掉：
   
   ```
   # NFS over RDMA client support
   xprtrdma
   # NFS over RDMA server support
   svcrdma
   ```

3. 在 NFS 服务器中，创建目录 `/mnt/nfsordma` 并将其导出到 `/etc/exports` ：
   
   ```
   # mkdir /mnt/nfsordma
   # echo "/mnt/nfsordma *(fsid=0,rw,async,insecure,no_root_squash)" >> /etc/exports
   ```

4. 在 NFS 客户端上，使用服务器 IP 地址挂载 nfs-share，例如 `192.168.1.10` ：
   
   ```
   # mount -o rdma,port=20049 192.168.1.10:/mnt/nfs-share /mnt/nfs
   ```

5. 重启 `nfs-server` 服务：
   
   ```
   # systemctl restart nfs-server
   ```

# 第 5 章 配置只使用

作为 NFS 服务器管理员，您可以将 NFS 服务器配置为仅支持 NFSv4，从而最大限度地减少系统上打开的端口和运行服务的数量。

## 5.1 只使用 NFSv4 的服务的优劣

本节说明将 NFS 服务器配置为仅支持 NFSv4 的优点和缺点。

默认情况下，NFS 服务器在 Opencloud OS 8 中支持 NFSv3 和 NFSv4 连接。但是，您也可以将 NFS 配置为仅支持 NFS 4.0 和更高版本。这最大限度地减少了系统上打开的端口和运行服务的数量，因为 NFSv4 不需要 `rpcbind` 服务来监听网络。

当您的 NFS 服务器配置为仅 NFSv4 时，尝试使用 NFSv3 挂载共享的客户端会失败，并出现如下错误：

```
Requested NFS version or transport protocol is not supported.
```

或者，您还可以禁用对 `RPCBIND`、`MOUNT` 和 `NSM` 协议的监听，这在仅使用 NFSv4 的情况下不是必需的。

禁用这些附加选项的影响有：

- 尝试使用 NFSv3 从服务器挂载共享的客户端会无响应。
- NFS 服务器本身无法挂载 NFSv3 文件系统。

## 5.2 将 NFS 服务器配置为仅支持 NFSv4

此过程描述如何将 NFS 服务器配置为仅支持 NFS 4.0 版及更高版本。

**流程**

1. 通过将以下行添加到 `/etc/nfs.conf` 配置文件的 `[nfsd]` 部分来禁用 NFSv3：
   
   ```
   [nfsd]
   
   vers3=no
   ```

2. (可选)禁用对 `RPCBIND`、`MOUNT` 和 `NSM` 协议的监听，这在仅使用 NFSv4 的情况下不是必需的。 禁用相关服务：
   
   ```
   # systemctl mask --now rpc-statd.service rpcbind.service rpcbind.socket
   ```

3. 重启 NFS 服务器：
   
   ```
   # systemctl restart nfs-server
   ```

更改会在您启动或重新启动 NFS 服务器后立即生效。

## 5.3 验证只读 NFSv4 配置

此过程描述如何使用 netstat 工具验证您的 NFS 服务器是否配置为仅使用 NFSv4 模式。

**流程**

- 使用 `netstat` 实用程序列出侦听 TCP 和 UDP 协议的服务：
  
  ```
  # netstat --listening --tcp --udp
  ```
  
  **例 5.1 仅 NFSv4 服务器上的输出**
  
  以下是仅使用 NFSv4 服务器上的 `netstat` 输出示例；禁用对 `RPCBIND`、MOUNT 和 NSM 的监听。这里，`nfs` 是唯一一个监听 NFS 的服务：
  
  ```
  # netstat --listening --tcp --udp
  
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State
  tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
  tcp        0      0 0.0.0.0:nfs             0.0.0.0:*               LISTEN
  tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
  tcp6       0      0 [::]:nfs                [::]:*                  LISTEN
  udp        0      0 localhost.locald:bootpc 0.0.0.0:*
  ```
  
  **例 5.2 配置仅 NFSv4 服务器之前的输出**
  
  相比之下，配置仅 NFSv4 服务器之前的 `netstat` 输出包括 `sunrpc` 和 `mountd` 服务：
  
  ```
  # netstat --listening --tcp --udp
  
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address State
  tcp        0      0 0.0.0.0:ssh             0.0.0.0:*       LISTEN
  tcp        0      0 0.0.0.0:40189           0.0.0.0:*       LISTEN
  tcp        0      0 0.0.0.0:46813           0.0.0.0:*       LISTEN
  tcp        0      0 0.0.0.0:nfs             0.0.0.0:*       LISTEN
  tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*       LISTEN
  tcp        0      0 0.0.0.0:mountd          0.0.0.0:*       LISTEN
  tcp6       0      0 [::]:ssh                [::]:*          LISTEN
  tcp6       0      0 [::]:51227              [::]:*          LISTEN
  tcp6       0      0 [::]:nfs                [::]:*          LISTEN
  tcp6       0      0 [::]:sunrpc             [::]:*          LISTEN
  tcp6       0      0 [::]:mountd             [::]:*          LISTEN
  tcp6       0      0 [::]:45043              [::]:*          LISTEN
  udp        0      0 localhost:1018          0.0.0.0:*
  udp        0      0 localhost.locald:bootpc 0.0.0.0:*
  udp        0      0 0.0.0.0:mountd          0.0.0.0:*
  udp        0      0 0.0.0.0:46672           0.0.0.0:*
  udp        0      0 0.0.0.0:sunrpc          0.0.0.0:*
  udp        0      0 0.0.0.0:33494           0.0.0.0:*
  udp6       0      0 [::]:33734              [::]:*
  udp6       0      0 [::]:mountd             [::]:*
  udp6       0      0 [::]:sunrpc             [::]:*
  udp6       0      0 [::]:40243              [::]:*
  ```

# 第 6 章 保护 NFS

为了最大限度地降低 NFS 安全风险并保护服务器上的数据，在服务器上导出 NFS 文件系统或将它们挂载到客户端时，请考虑以下部分。

## 6.1 具有 AUTH_SYS 和导出控制的 NFS 安全性

NFS 提供以下传统选项以控制对导出文件的访问：

- 服务器限制哪些主机可以通过 IP 地址或主机名挂载哪些文件系统。

- 服务器对 NFS 客户端上的用户强制执行文件系统权限的方式与对本地用户强制执行权限的方式相同。传统上，NFS 使用 `AUTH_SYS` 调用消息（也称为 `AUTH_UNIX`）来执行此操作，该消息依赖于客户端来声明用户的 UID 和 GID。请注意，这意味着恶意或配置错误的客户端可能很容易出错，并允许用户访问不应访问的文件。

为了限制潜在风险，管理员通常将访问权限限制为只读或压缩用户权限为普通用户和组 ID。不幸的是，这些解决方案阻止了 NFS 共享以最初预期的方式使用。

此外，如果攻击者获得了导出 NFS 文件系统的系统所使用的 DNS 服务器的控制权，他们可以将与特定主机名或完全限定域名关联的系统指向未经授权的机器。此时，未经授权的机器是允许挂载 NFS 共享的系统，因为没有交换用户名或密码信息来为 NFS 挂载提供额外的安全性。

通过 NFS 导出目录时应谨慎使用通配符，因为通配符的范围可能包含比预期更多的系统。

## 6.2 带有 AUTH_GSS 的 NFS 安全性

NFS 的所有版本都支持 RPCSEC_GSS 和 Kerberos 机制。

与 AUTH_SYS 不同，使用 RPCSEC_GSS Kerberos 机制，服务器不依赖客户端来正确表示正在访问文件的用户。相反，加密用于向服务器验证用户身份，从而防止恶意客户端在没有该用户的 Kerberos 凭证的情况下模拟用户。使用 RPCSEC_GSS Kerberos 机制是保护挂载的最直接方法，因为在配置 Kerberos 之后，不需要额外的设置。

## 6.3 配置 NFS 服务器和客户端使用 Kerberos

Kerberos 是一种网络身份验证系统，它允许客户端和服务器通过使用对称加密和受信任的第三方 KDC 来相互进行身份验证。 建议使用身份管理 (IdM) 来设置 Kerberos。

**前提条件**

- 已安装并配置 Kerberos 密钥分发中心 (`KDC`)。

**流程**

1. - 在 NFS 服务器端创建 nfs/hostname.domain@REALM 主体。
     - 在服务器端和客户端创建 host/hostname.domain@REALM 主体。
     - 将相应的密钥添加到客户端和服务器的密钥表中。

2. 在服务器端，使用 `sec=` 选项启用所需的安全类别。启用所有安全类别以及非加密挂载：
   
   ```
   /export *(sec=sys:krb5:krb5i:krb5p)
   ```
   
   与 `sec=` 选项一起使用的有效安全类型是：
   
   - `sys`：无密码保护，默认
   
   - `krb5`：仅认证
   
   - `krb5i`：完整性保护
     
     - 使用 Kerberos V5 进行用户身份验证，并使用安全校验和执行 NFS 操作的完整性检查，以防止数据篡改。
   
   - `krb5p`：隐私保护
     
     - 使用 Kerberos V5 进行用户身份验证、完整性检查，并加密 NFS 流量以防止流量嗅探。这是最安全的设置，但也涉及最多的性能开销
       。

3. 在客户端，将 `sec=krb5`（或 `sec=krb5i`，或 `sec=krb5p`，取决于设置）添加到挂载选项：

```
# mount -o sec=krb5 server:/export /mnt
```

## 6.4 NFSv4 安全选项

NFSv4 包括基于 Microsoft Windows NT 模型而不是 POSIX 模型的 ACL 支持，因为 Microsoft Windows NT 模型的功能和广泛部署。

NFSv4 的另一个重要安全特性是不再使用 `MOUNT` 协议来挂载文件系统。 因为协议处理文件句柄的方式，`MOUNT` 协议存在安全风险。

## 6.5 挂载的 NFS 导出的文件权限

一旦 NFS 文件系统被远程主机以读取或读写方式挂载，保护每个共享文件的唯一方法就是权限。如果共享相同用户 ID 值的两个用户在不同的客户端系统上挂载相同的 NFS 文件系统，他们可以修改彼此的文件。此外，在客户端系统上以 root 身份登录的任何人都可以使用 `su -` 命令访问具有 NFS 共享的任何文件。

默认情况下，Opencloud OS 的 NFS 支持访问控制列表 (ACL)。一般建议保持启用此功能。

默认情况下，NFS 在导出文件系统时使用 *root squashing* 。这会将在其本地计算机上以 root 用户身份访问 NFS 共享的任何人的用户 ID 设置为 `nobody`。Root squashing 由默认选项 `root_squash` 控制；有关此选项的更多信息，请参阅 [NFS 服务器配置](#46-nfs-服务器配置)。

将 NFS 共享导出为只读时，请考虑使用 `all_squash` 选项。此选项使访问导出文件系统的每个用户都使用 `nobody` 用户的用户ID。

# 第 7 章 在 NFS 中启用 pNFS SCSI 布局

您可以将 NFS 服务器和客户端配置为使用 pNFS SCSI 布局来访问数据。pNFS SCSI 在涉及对文件进行更长时间的单客户端访问的用例中非常有用。

**前提条件**

- 客户端和服务器都必须能够向同一个块设备发送 SCSI 命令。也就是说，块设备必须在共享的 SCSI 总线上。
- 块设备必须包含 XFS 文件系统。
- SCSI 设备必须支持 SCSI-3 Primary Commands，比如规范中描述的 SCSI Persistent Reservations。

## 7.1 pNFS 技术

pNFS 架构提高了 NFS 的可扩展性。当服务器实现 pNFS 时，客户端能够同时通过多个服务器访问数据。这可以提高性能。

pNFS 在 OpenCloud OS 上支持以下存储协议或布局：

- 文件
- Flexfiles
- SCSI

## 7.2 pNFS SCSI 布局

SCSI 布局建立在 pNFS 布局的基础上。布局是依靠 SCSI 设备定义的。它包含一系列固定大小的块来作为逻辑单元 (LU)，这些块必须能够支持 SCSI 持久保留。 LU 设备由它们的 SCSI 设备来识别。

pNFS SCSI 在涉及对文件进行长时间的单客户端访问中表现良好。例如：邮件服务器或虚拟机。

### 客户端和服务器之间的操作

当 NFS 客户端读取或写入文件时，客户端执行 `LAYOUTGET` 操作。服务器以文件在 SCSI 设备上的位置进行响应。客户端可能需要执行 `GETDEVICEINFO` 的附加操作来确定要使用的 SCSI 设备。如果这些操作正常工作，客户端可以直接向 SCSI 设备发出 I/O 请求，而不是向服务器发送 `READ` 和 `WRITE` 操作。

客户端之间的错误或争用可能会导致服务器重新调用布局或不将它们发布给客户端。在这些情况下，客户端回退到向服务器发出 READ 和 WRITE 操作，而不是直接向 SCSI 设备发送 I/O 请求。

要监控操作，请参阅[监控 pNFS SCSI 布局功能](#第-8-章-监控-pnfs-scsi-布局功能)。

**预留设备**

pNFS SCSI 通过预留分配来处理防护。在服务器向客户端发布布局之前，它会保留 SCSI 设备以确保只有注册的客户端才能访问该设备。如果客户端可以向该 SCSI 设备发出命令但未向该设备注册，则该设备上客户端的许多操作都会失败。例如，如果服务器没有为客户端提供该设备的布局，则客户端上的 `blkid` 命令无法显示 XFS 文件系统的 UUID。

服务器不会删除它自己的持久保留。这可以在客户端和服务器重新启动时保护设备上文件系统中的数据。为了重新利用 SCSI 设备，您可能需要手动删除 NFS 服务器上的持久保留。

## 7.3 检查与 pNFS 兼容的 SCSI 设备

此过程检查 SCSI 设备是否支持 pNFS SCSI 布局。

**前提条件**

- 安装 `sg3_util`s 包：
  
  ```
  # yum install sg3_utils
  ```

**流程**

- 在服务器和客户端上，检查正确的 SCSI 设备支持：
  
  ```
  # sg_persist --in --report-capabilities --verbose path-to-scsi-device
  ```
  
  确保设置了 *Persist Through Power Loss Active* (`PTPL_A`) 位。
  
  **例 7.1 支持 pNFS SCSI 的 SCSI 设备**
  
  以下是支持 pNFS SCSI 的 SCSI 设备的 `sg_persist` 输出示例。 `PTPL_A` 位报告` 1`。
  
  ```
    inquiry cdb: 12 00 00 00 24 00
    Persistent Reservation In cmd: 5e 02 00 00 00 00 00 20 00 00
    LIO-ORG   block11           4.0
    Peripheral device type: disk
    Report capabilities response:
    Compatible Reservation Handling(CRH): 1
    Specify Initiator Ports Capable(SIP_C): 1
    All Target Ports Capable(ATP_C): 1
    Persist Through Power Loss Capable(PTPL_C): 1
    Type Mask Valid(TMV): 1
    Allow Commands: 1
    Persist Through Power Loss Active(PTPL_A): 1
      Support indicated in Type mask:
        Write Exclusive, all registrants: 1
        Exclusive Access, registrants only: 1
        Write Exclusive, registrants only: 1
        Exclusive Access: 1
        Write Exclusive: 1
        Exclusive Access, all registrants: 1
  ```

## 7.4 在服务器上设置 pNFS SCSI

此过程将 NFS 服务器配置为导出 pNFS SCSI 布局。

**流程**

1. 在服务器上，挂载在 SCSI 设备上创建的 XFS 文件系统。

2. 将 NFS 服务器配置为导出 NFS 4.1 或更高版本。在 `/etc/nfs.conf` 文件的 `[nfsd]` 部分中设置以下选项：
   
   ```
   [nfsd]
   
   vers4.1=y
   ```

3. 使用 pnfs 选项配置 NFS 服务器以通过 NFS 导出 XFS 文件系统：
   
   **例 7.2 /etc/exports 中用于导出 pNFS SCSI 的条目**
   
   `/etc/exports` 配置文件中的以下条目将挂载在 `/exported/directory/` 的文件系统导出到 `allowed.example.com` 客户端，来作为来作为 pNFS SCSI 布局：
   
   ```
   /exported/directory allowed.example.com(pnfs)
   ```
   
   > **注意**
   > 
   > 必须在整个块设备中创建导出的文件系统，而不仅仅在分区中创建。

## 7.5 在客户端上设置 pNFS SCSI

此过程将 NFS 客户端配置为挂载 pNFS SCSI 布局。

**前提条件**

- NFS 服务器配置为通过 pNFS SCSI 导出 XFS 文件系统。请参阅[在服务器上设置 pNFS SCSI](#74-在服务器上设置-pnfs-scsi)。

**流程**

- 在客户端上，使用 NFS 4.1 或更高版本挂载导出的 XFS 文件系统：
  
  ```
  # mount -t nfs -o nfsvers=4.1 host:/remote/export /local/directory
  ```
  
  不要在没有 NFS 的情况下直接挂载 XFS 文件系统。

## 7.6 释放服务器上的 pNFS SCSI 保留

此过程释放 NFS 服务器在 SCSI 设备上保留的持久保留。这使您可以在不再需要导出 pNFS SCSI 时重新调整 SCSI 设备的用途。

您必须从服务器中删除保留。它不能从不同的 IT Nexus 中删除。

**前提条件**

- 安装 `sg3_utils` 包：
  
  ```
  # yum install sg3_utils
  ```

**流程**

1. 查询服务器上的现有预留：
   
   ```
   # sg_persist --read-reservation path-to-scsi-device
   ```
   
   **例 7.3 在 /dev/sda 上查询预留**
   
   ```
   # *sg_persist --read-reservation /dev/sda*
   
    LIO-ORG   block_1           4.0
    Peripheral device type: disk
    PR generation=0x8, Reservation follows:
      Key=0x100000000000000
      scope: LU_SCOPE,  type: Exclusive Access, registrants only
   ```

2. 删除服务器上的现有注册：
   
   ```
   # sg_persist --out \
              --release \
              --param-rk=reservation-key \
              --prout-type=6 \
              path-to-scsi-device
   ```
   
   **例 7.4 删除 /dev/sda 上的保留**
   
   ```
   # sg_persist --out \
              --release \
              --param-rk=0x100000000000000 \
              --prout-type=6 \
              /dev/sda
   
    LIO-ORG   block_1           4.0
    Peripheral device type: disk
   ```

# 第 8 章 监控 pNFS SCSI 布局功能

您可以监视 pNFS 客户端和服务器是否交换正确的 pNFS SCSI 操作，或者它们是否回退到常规 NFS 操作。

**前提条件**

- 配置了 pNFS SCSI 客户端和服务器。

## 8.1 使用 nfsstat 从服务器检查 pNFS SCSI 操作

此过程使用 `nfsstat` 实用程序从服务器监视 pNFS SCSI 操作。

**流程**

1. 监控从服务器服务的操作：
   
   ```
   # watch --differences \
          "nfsstat --server | egrep --after-context=1 read\|write\|layout"
   
   Every 2.0s: nfsstat --server | egrep --after-context=1 read\|write\|layout
   
   putrootfh    read         readdir      readlink     remove     rename
   2         0% 0         0% 1         0% 0         0% 0         0% 0         0%
   --
   setcltidconf verify      write        rellockowner bc_ctl     bind_conn
   0         0% 0         0% 0         0% 0         0% 0         0% 0         0%
   --
   getdevlist   layoutcommit layoutget    layoutreturn secinfononam sequence
   0         0% 29        1% 49        1% 5         0% 0         0% 2435     86%
   ```

2. 客户端和服务器在以下情况下使用 pNFS SCSI 操作：
- `layoutget`、`layoutreturn` 和 `layoutcommit` 计数器递增。这意味着服务器正在提供布局。
- 服务器 `READ` 和 `WRITE` 计数器不会增加。这意味着客户端直接向 SCSI 设备执行 I/O 请求。

## 8.2 使用 mountstats 从客户端检查 pNFS SCSI 操作

此过程使用 `/proc/self/mountstats` 文件从客户端监视 pNFS SCSI 操作。

**流程**

1. 列出每个挂载操作计数器：
   
   ```
   # cat /proc/self/mountstats \
        | awk /scsi_lun_0/,/^$/ \
        | egrep device\|READ\|WRITE\|LAYOUT
   
   device 192.168.122.73:/exports/scsi_lun_0 mounted on /mnt/rhel7/scsi_lun_0 with fstype nfs4 statvers=1.1
      nfsv4:  bm0=0xfdffbfff,bm1=0x40f9be3e,bm2=0x803,acl=0x3,sessions,pnfs=LAYOUT_SCSI
              READ: 0 0 0 0 0 0 0 0
            WRITE: 0 0 0 0 0 0 0 0
          READLINK: 0 0 0 0 0 0 0 0
          READDIR: 0 0 0 0 0 0 0 0
        LAYOUTGET: 49 49 0 11172 9604 2 19448 19454
      LAYOUTCOMMIT: 28 28 0 7776 4808 0 24719 24722
      LAYOUTRETURN: 0 0 0 0 0 0 0 0
      LAYOUTSTATS: 0 0 0 0 0 0 0 0
   ```

2. 在结果中：
- `LAYOUT` 统计信息指示客户端和服务器使用 pNFS SCSI 操作的请求。
- `READ` 和 `WRITE` 统计信息指示客户端和服务器退回到 NFS 操作的请求。

# 9. FS-Cache 入门

FS-Cache 是一个持久的本地缓存，文件系统可以使用它来获取通过网络检索的数据并将其缓存在本地磁盘上。 这有助于最大限度地减少用户从通过网络安装的文件系统（例如 NFS）访问数据的网络流量。

## 9.1  FS-Cache 概述

下图是 FS-Cache 如何工作的高级说明：

**图 9.1 FS 缓存概述**

![缓存概述图](./images/9-1.png)

FS-Cache 旨在对系统的用户和管理员尽可能透明。与 Solaris 上的 `cachefs` 不同，FS-Cache 允许服务器上的文件系统直接与客户端的本地缓存交互，而无需创建过载的文件系统。对于 NFS，挂载选项指示客户端在启用 FS-cache 的情况下挂载 NFS 共享。挂载点将导致两个内核模块的自动上传：`fscache` 和 `cachefiles`。 cachefilesd 守护进程与内核模块通信以实现缓存。

FS-Cache 不会改变在网络上工作的文件系统的基本操作——它只是为该文件系统提供一个可以永久缓存数据的位置。例如，无论是否启用了 FS-Cache，客户端仍然可以挂载 NFS 共享。此外，缓存的 NFS 可以处理不能全部缓存的文件（无论是单独的还是集体的），因为文件可以被部分缓存，而不必预先完全读取。FS-Cache 还会隐藏发生在客户端文件系统驱动程序的缓存中的所有 I/O 错误。

为了提供缓存服务，FS-Cache 需要一个 *缓存后端* 。缓存后端是配置缓存服务的存储驱动程序，即 `cachefile` 。在这种情况下，FS-Cache 需要一个已挂载的基于块的文件系统，该文件系统支持 `bmap` 和扩展属性（例如 ext3）作为其缓存后端。

支持 FS-Cache 缓存后端所需功能的文件系统包括以下几种（文件系统由 OpenCloud OS 8 实现）：

- ext3（启用扩展属性）
- ext4
- XFS

FS-Cache 不能任意缓存任何文件系统，无论是通过网络还是其他方式：必须更改共享文件系统的驱动程序以允许与 FS-Cache、数据存储/检索以及元数据设置和验证进行交互。 FS-Cache 需要缓存的文件系统中 *索引密钥* 和 *一致性数据* 来支持持久性：索引密钥匹配文件系统对象缓存对象，一致性数据确定缓存对象是否仍然有效。

> **注意**
> 
> 在 OpenCloud OS 8 中，不会默认安装 **cachefilesd** 软件包，需要手动安装。

## 9.2 性能保证

FS-Cache 不保证提高性能。使用缓存会导致性能损失：例如，缓存的 NFS 共享会将磁盘访问添加到跨网络查找中。虽然 FS-Cache 尽可能尝试异步，但在同步路径（例如读取）中这是不可能的。

例如，使用 FS-Cache 在没有负载的 GigE 网络上的两台计算机之间缓存 NFS 共享，可能不会显示文件访问的任何性能改进。相反，从服务器内存而不是本地磁盘可以更快地满足 NFS 请求。

因此，FS-Cache 的使用是各种因素之间的 *折衷* 。例如，如果使用 FS-Cache 来缓存 NFS 流量，它可能会稍微降低客户端的速度，但会通过满足本地读取请求而不消耗网络带宽来大幅减少网络和服务器负载。

## 9.3 设置缓存

目前，OpenCloud OS 8 只提供缓存文件缓存后端。 cachefilesd 守护进程启动和管理缓存文件。 /etc/cachefilesd.conf 文件控制 cachefiles 如何提供缓存服务。

缓存后端通过在托管缓存的分区上维护一定数量的可用空间来工作。它会根据系统的其他元素用尽可用空间来增加和缩小缓存，使其在根文件系统（例如笔记本电脑）上安全使用。 FS-Cache 对此行为设置默认值，可以通过缓存剔除限制进行配置。有关配置缓存剔除限制的更多信息，请参阅缓存剔除限制配置。

此过程显示如何设置缓存。

**前提条件**

- 软件包已安装并且服务已成功启动。要确保服务正在运行，请使用以下命令：
  
  ```
  # systemctl start cachefilesd
  # systemctl status cachefilesd
  ```
  
  状态必须处于 *active（running）*。

**流程**

1. 在缓存后端配置哪个目录用作缓存，使用以下参数：
   
   ```
   $ dir /path/to/cache
   ```

2. 通常，缓存后端目录在 `/etc/cachefilesd.conf` 中设置为 `/var/cache/fscache`，如下所示：
   
   ```
   $ dir /var/cache/fscache
   ```

3. 如果要更改缓存后端目录，selinux 上下文必须与 `/var/cache/fscache` 相同：
   
   ```
   # semanage fcontext -a -e /var/cache/fscache /path/to/cache
   # restorecon -Rv /path/to/cache
   ```

4. 设置缓存时将 */path/to/cache* 替换为目录名称。

5. 如果用于设置 selinux 上下文的给定命令不起作用，请使用以下命令：
   
   ```
   # semanage permissive -a cachefilesd_t
   # semanage permissive -a cachefiles_kernel_t
   ```
   
   FS-Cache 将缓存存储在托管 *`/path/to/cache`* 的文件系统中。在笔记本电脑上，建议使用根文件系统 (`/`) 作为主机文件系统，但对于台式机，更谨慎的做法是安装专门用于缓存的磁盘分区。

6. 主机文件系统必须支持用户定义的扩展属性； FS-Cache 使用这些属性来存储保持一致性的维护信息。要为 ext3 文件系统（即`device`）启用用户定义的扩展属性，请使用：
   
   ```
   # tune2fs -o user_xattr /dev/device
   ```

7. 要在挂载时为文件系统启用扩展属性，也可以使用以下命令：
   
   ```
   # mount /dev/device /path/to/cache -o user_xattr
   ```

8. 配置文件到位后，启动 `cachefilesd` 服务：
   
   ```
   # systemctl start cachefilesd
   ```

9. 要将 cachefilesd 配置为在引导时启动，请以 root 身份执行以下命令：
   
   ```
   # systemctl enable cachefilesd
   ```

## 9.4 cache cull limits 配置

`cachefilesd` 守护进程工作原理：通过缓存来自共享文件系统的远程数据来释放磁盘空间。这可能会消耗所有的可用空间，如果磁盘还存在根分区，这可能会很糟糕。为了控制这一点，`cachefilesd` 尝试通过丢弃缓存中的旧对象（即最近访问较少的对象）来维持一定数量的可用空间。这种行为称为缓存筛选 （*cache culling*）。

缓存筛选是根据底层文件系统中块的百分比和可用文件的百分比来完成的。 `/etc/cachefilesd.conf` 中有控制六个限制的设置：

brun ***N%***（**块百分比**），frun ***N%***（**文件百分比**）

  如果缓存中的可用空间量和可用文件的数量超过这两个限制，则关闭筛选。

bcull ***N%***（**块百分比**），fcull ***N%***（**文件百分比**）

  如果缓存中的可用空间量或文件数低于这些限制中的任何一个，则开始筛选。

bstop ***N%***（**块百分比**），fstop ***N%***（**文件百分比**）

  如果缓存中的可用空间量或可用文件数低于这些限制中的任何一个，则不允许进一步分配磁盘空间或文件，直到筛选出再次引发超过这些限制的情况。

每个设置的默认 `N` 值如下：

- `brun`/`frun` - 10%
- `bcull`/`fcull` - 7%
- `bstop`/`fstop` - 3%

配置这些设置时，以下必须成立：

- bstop < bcull < brun < 100
- fstop < fcull < frun < 100

这些是可用空间和可用文件的百分比，不会显示为 100 减去 `df` 程序显示的百分比。

> **重要**
> 
> 筛选同时依赖于 bxxx 和 fxxx 对；用户不能单独处理它们。

## 9.5 从 fscache 内核模块检索统计信息

FS-Cache 还跟踪一般统计信息。此过程显示如何获取此信息。

**流程**

1. 要查看 FS-Cache 的统计信息，请使用以下命令：
   
   ```
   # cat /proc/fs/fscache/stats
   ```

FS-Cache 统计信息包括有关决策点和对象计数器的信息。有关详细信息，请参阅以下内核文档：

`/usr/share/doc/kernel-doc-4.18.0/Documentation/filesystems/caching/fscache.txt`

## 9.6 FS-Cache 参考

本节提供 FS-Cache 的参考信息。

1. 有关 `cachefilesd` 以及如何配置它的更多信息，请参阅 `man cachefilesd` 和 `man cachefilesd.conf`。以下内核文档还提供了其他信息：
- `/usr/share/doc/cachefilesd/README`

- `/usr/share/man/man5/cachefilesd.conf.5.gz`

- `/usr/share/man/man8/cachefilesd.8.gz`
2. 有关 FS-Cache 的常用信息，包括有关其设计约束、可用统计数据和功能的详细信息，请参阅以下内核文档：
   
   `/usr/share/doc/kernel-doc-4.18.0/Documentation/filesystems/caching/fscache.txt`

# 第 10 章 在 NFS 中使用缓存

除非明确指示，否则 NFS 不会使用缓存。本段说明如何使用 FS-Cache 配置 NFS 挂载。

**前提条件**

- **cachefilesd** 软件包已安装并正在运行。要确保它正在运行，请使用以下命令：
  
  ```
  # systemctl start cachefilesd
  # systemctl status cachefilesd
  ```
  
  状态处于 *active（running）*。

- 使用以下选项挂载 NFS 共享：
  
  ```
  # mount nfs-share:/ /mount/point -o fsc
  ```
  
  对 *`/mount/point`* 下文件的所有访问都将通过缓存，除非该文件是为直接 I/O 或写入而打开的。有关详细信息，请参阅 [NFS 的缓存限制](#102-nfs-的缓存限制)。

NFS 使用 NFS 文件句柄而不是文件名来索引缓存内容，这意味着硬链接文件正确共享缓存。

NFS 版本 3、4.0、4.1 和 4.2 支持缓存。但是，每个版本都使用不同的分支进行缓存。

## 10.1 配置 NFS 缓存共享

NFS 缓存共享存在几个潜在问题。因为缓存是持久的，所以缓存中的数据块会在四个键的序列上被索引：

- 第 1 级：服务器详情
- 第 2 级：一些挂载选项；安全类型；FSID；uniquifier
- 第 3 级：文件处理
- 第 4 级：文件中的页号

为了避免超级块之间的一致性管理问题，所有需要缓存数据的 NFS 超级块都具有唯一的 2 级键。通常，具有相同源卷和选项的两个 NFS 挂载共享一个超级块.因此，共享缓存在该卷中挂载不同的目录也是如此。

以下是如何使用不同选项配置缓存共享的示例。

**流程**

1. 使用以下命令挂载 NFS 共享：
   
   ```
   mount home0:/disk0/fred /home/fred -o fsc
   mount home0:/disk0/jim /home/jim -o fsc
   ```
   
   在这里，`/home/fred` 和 `/home/jim` 可能共享超级块，因为它们具有相同的选项，特别是如果它们来自 NFS 服务器 (`home0`) 上的相同卷/分区。

2. 要是不共享超级块，请使用带有以下选项的 `mount` 命令：
   
   ```
   mount home0:/disk0/fred /home/fred -o fsc,rsize=8192
   mount home0:/disk0/jim /home/jim -o fsc,rsize=65536
   ```
   
   在这种情况下，`/home/fred` 和 `/home/jim` 不会共享超级块，因为它们具有不同的网络访问参数，这些参数是 2 级键的一部分。

3. 要在不共享超级块的情况下将两个子树（`/home/fred1` 和 `/home/fred2`）的内容缓存两次，请使用以下命令：
   
   ```
   mount home0:/disk0/fred /home/fred1 -o fsc,rsize=8192
   mount home0:/disk0/fred /home/fred2 -o fsc,rsize=65536
   ```

4. 避免超级块共享的另一种方法是使用 `nosharecache` 参数显式抑制它。使用相同的示例：
   
   ```
   mount home0:/disk0/fred /home/fred -o nosharecache,fsc
   mount home0:/disk0/jim /home/jim -o nosharecache,fsc
   ```
   
   然而，在这种情况下，只有一个超级块被允许使用缓存，因为没有什么可以区分 `home0:/disk0/fred` 和 `home0:/disk0/jim` 的 2 级键。

5. 要指定超级块的寻址，请在至少一个挂载上添加 *unique-identifie* ，即 `fsc=unique-identifier`：
   
   ```
   mount home0:/disk0/fred /home/fred -o nosharecache,fsc
   mount home0:/disk0/jim /home/jim -o nosharecache,fsc=jim
   ```
   
   这里，唯一标识符 `jim` 被添加到 `/home/jim` 的缓存中使用的 2 级键中。

> **重要**
> 
> 用户不能在具有不同通信或协议参数的超级块之间共享缓存。例如，无法在 NFSv4.0 和 NFSv3 之间或 NFSv4.1 和 NFSv4.2 之间共享，因为它们强制使用不同的超级块。另外，设置读取大小(rsize)等参数可防止缓存共享，因为它也强制使用不同的超级块。

## 10.2  NFS 的缓存限制

NFS 有一些缓存限制：

- 通过直接 I/O 从共享文件系统打开文件会自动绕过缓存。这是因为这种类型的访问必须直接访问服务器。
- 从共享文件系统打开文件来进行直接 I/O 或写入会刷新文件的缓存副本。 FS-Cache 不会再次缓存文件，直到它不再为直接 I/O 或写操作而打开。
- 此外，此版本的 FS-Cache 仅缓存常规 NFS 文件。 FS-Cache 不会缓存目录、符号链接、设备文件、FIFO 和套接字。

# 第 11 章 挂载 SMB 共享

服务器消息块 (SMB) 协议实现了一个应用层网络协议，用于访问服务器上的资源，例如文件共享和共享打印机。

> **注意**
> 
> 在 SMB 的上下文中，您可以发现提到了通用 Internet 文件系统(CIFS)协议，它是 SMB 的一种方言。 SMB 和 CIFS 协议都受支持，并且挂载 SMB 和 CIFS 共享所涉及的内核模块和实用程序都使用名称 `cifs`。

本节介绍如何从 SMB 服务器挂载共享。

**前提条件**

在 Microsoft Windows 上，默认实现 SMB。在 OpenCloud OS 上，内核的 `cifs.ko` 文件系统模块支持挂载 SMB 共享。因此，安装 `cifs-utils` 包：

```
# yum install cifs-utils
```

`cifs-utils` 软件包提供了以下实用程序：

- 挂载 SMB 和 CIFS 共享
- 管理内核密钥环中的 NT Lan Manager (NTLM) 凭证
- 在 SMB 和 CIFS 共享的安全描述符中设置和显示访问控制列表 (ACL)

## 11.1 支持的 SMB 协议版本

`cifs.ko` 内核模块支持以下 SMB 协议版本：

- SMB 1

> **警告**
> 
> 由于已知的安全问题，SMB1 协议已被弃用，只能在**私有网络上安全使用**。 SMB1 仍然作为受支持的选项提供的主要原因是，它是目前唯一支持 UNIX 扩展的 SMB 协议版本。如果您不需要在 SMB 上使用 UNIX 扩展，强烈建议使用 SMB2 或更高版本。

- SMB 2.0
- SMB 2.1
- SMB 3.0
- SMB 3.1.1

> 注意
> 
> 根据协议版本，并非所有 SMB 功能都已实现。

## 11.2  UNIX 扩展支持

Samba 使用 SMB 协议中的 `CAP_UNIX` 功能位来提供 UNIX 扩展功能。 `cifs.ko` 内核模块也支持这些扩展。但是，Samba 和内核模块都只支持 SMB 1 协议中的 UNIX 扩展。

要使用 UNIX 扩展：

1. 将 `/etc/samba/smb.conf` 文件中 `[global]` 部分中的 `server min protocol` 设置为 `NT1`。

2. 通过向 mount 命令提供 `-o vers=1.0` 选项，使用 SMB 1 协议安装共享。例如：
   
   ```
   # mount -t cifs -o vers=1.0,username=user_name //server_name/share_name /mnt/
   ```
   
   默认情况下，内核模块使用 SMB 2 或服务器支持的更高版本的协议。将 `-o vers=1.0` 选项传递给 `mount` 命令会强制内核模块使用使用 UNIX 扩展所需的 SMB 1 协议。

要验证是否启用了 UNIX 扩展，请显示已安装共享的选项：

```
# mount
...
//server/share on /mnt type cifs (...,unix,...)
```

如果 `unix` 条目显示在挂载选项列表中，则表示启用了 UNIX 扩展。

## 11.3 手动挂载 SMB 共享

如果您只需要临时挂载 SMB 共享，则可以使用 `mount` 工具手动挂载它。

> 注意
> 
> 重新启动系统时，手动挂载的共享不会再次自动挂载。要配置 OpenCloud OS 在系统引导时自动挂载共享，请参阅在[系统启动时自动挂载 SMB 共享](#114-系统启动时自动挂载-smb-共享)。

**前提条件**

- `cifs-utils` 软件包已安装。

流程

要手动挂载 SMB 共享，请使用带有 `-t cifs` 参数的 `mount` 工具：

```
# mount -t cifs -o username=user_name //server_name/share_name /mnt/
Password for user_name@//server_name/share_name:  password
```

在 `-o` 参数中，您可以指定用于挂载共享的选项。有关详细信息，请参阅 `mount.cifs(8)` 手册页中的 `OPTIONS` 部分和[常用挂载选项](#116-常用挂载选项)。

**例 11.1 使用加密的 SMB 3.0 连接挂载共享**

要以 *`DOMAIN`*`\Administrator` 用户身份通过​​加密的 SMB 3.0 连接将` \\server\example\` 共享挂载到 /mnt/ 目录：

```
# mount -t cifs -o username=DOMAIN\Administrator,seal,vers=3.0 //server/example /mnt/
>Password for DOMAIN\Administrator@//server_name/share_name:  password
```

## 11.4 系统启动时自动挂载 SMB 共享

如果服务器上永久需要访问已安装的 SMB 共享，请在启动时自动挂载共享。

**前提条件**

- `cifs-utils` 软件包已安装。

**流程**

要在系统引导时自动挂载 SMB 共享，请将共享条目添加到 `/etc/fstab` 文件。例如：

```
//server_name/share_name /mnt cifs credentials=/root/smb.cred 0 0
```

> **重要**
> 
> 要使系统能够自动挂载共享，您必须将用户名、密码和域名存储在凭证文件中。有关详细信息，请参阅[使用凭证文件对 SMB 共享进行身份验证](#115-使用凭证文件对-smb-共享进行身份验证)。

在 `/etc/fstab` 中的第四个字段中，指定挂载选项，例如凭证文件的路径。有关详细信息，请参阅 `mount.cifs(8)` 手册页中的 `OPTIONS` 部分和[常用挂载选项](#116-常用挂载选项)。

要验证共享是否成功，请输入：

```
# mount /mnt/
```

## 11.5 使用凭证文件对 SMB 共享进行身份验证

在某些情况下，例如在引导时自动挂载共享时，应在不输入用户名和密码的情况下挂载共享。要实现这一点，请创建一个凭证文件。

**前提条件**

- `cifs-utils` 软件包已安装。

**流程**

1. 创建一个文件，例如 `/root/smb.cred`，并指定该文件的用户名、密码和域名：
   
   ```
   username=user_name
   password=password
   domain=domain_name
   ```

2. 将权限设置为只允许所有者访问文件：

```
# chown user_name /root/smb.cred
# chmod 600 /root/smb.cred
```

您现在可以将 `credentials=file_name` 挂载选项传递给 `mount` 工具，或在 `/etc/fstab` 文件中使用它来挂载共享，而不会提示您输入用户名和密码。

## 11.6 常用挂载选项

挂载 SMB 共享时，选项确定：

- 如何与服务器建立连接。例如，连接到服务器时使用哪个 SMB 协议版本。
- 如何将共享挂载到本地文件系统中。例如，如果系统覆盖远程文件和目录权限，使多个本地用户能够访问服务器上的内容。

要在 `/etc/fstab` 文件的第四个字段或 mount 命令的 `-o` 参数中设置多个选项，请用逗号分隔它们。例如，请参阅[使用 multiuser 选项挂载共享](#121-使用-multiuser-选项挂载共享)。

以下列表给出了常用的挂载选项：

| **选项**                      | **描述**                                                                                                                                                          |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| credentials=*file_name*     | 设置凭证文件的路径。请参阅[使用凭证文件认证到 SMB 共享](#115-使用凭证文件对-smb-共享进行身份验证)。                                                                                                     |
| dir_mode=*mode*             | 如果服务器不支持 CIFS UNIX 扩展，则设置目录模式。                                                                                                                                  |
| file_mode=*mode*            | 如果服务器不支持 CIFS UNIX 扩展，则设置文件模式。                                                                                                                                  |
| password=*password*         | 设置在 SMB 服务器中验证的密码。另外，也可使用 `credentials` 选项指定凭证文件。                                                                                                               |
| seal                        | 使用 SMB 3.0 或更高的协议版本启用对连接的加密支持。因此，使用 `seal` 和 `vers` 挂载选项来设置成 `3.0` 或更高版本。请参阅 [手动挂载 SMB 共享](#113-手动挂载-smb-共享) 中的示例。                                              |
| sec=*security_mode*         | 设置安全模式，如 `ntlmsspi`，来启用 NTLMv2 密码哈希和已启用的数据包签名。有关支持值的列表，请查看 `mount.cifs(8)` 手册页中的选项描述。如果服务器不支持 ntlmv2 安全模式，则使用 `sec=ntlmssp`，这是默认值。出于安全考虑，请不要使用不安全的 `ntlm` 安全模式。 |
| username=*user_name*        | 设置在 SMB 服务器中验证的用户名。另外，也可使用 `credentials` 选项指定凭证文件。                                                                                                              |
| vers=*SMB_protocol_version* | 设定用于与服务器通信的 SMB 协议版本。                                                                                                                                           |

有关完整列表，请参阅 `mount.cifs(8)` 手册页中的 `OPTIONS` 部分。

# 第 12 章 执行多用户 SMB 挂载

默认情况下，您提供的用于挂载共享的凭证决定了挂载点的访问权限。例如，如果您在挂载共享时使用 *`DOMAIN`*`\example` 用户，则该共享上的所有操作都将以该用户执行，而不管是哪个本地用户执行操作。

但是，在某些情况下，管理员希望在系统启动时自动挂载共享，但用户应使用自己的凭证对共享内容执行操作。 `multiuser` 挂载选项允许您配置此方案。

> **重要**
> 
> 要使用 `multiuser` 挂载选项，您必须另外将 `sec` 挂载选项设置为支持以非交互方式提供凭证的安全类型，例如 `krb5` 或带有凭证文件的 `ntlmssp` 选项。有关详细信息，请参阅[以用户身份访问共享](#123-以用户身份访问共享)。

`root` 用户使用多用户选项和对共享内容具有最小访问权限的帐户来安装共享。然后，普通用户可以使用 cifscreds 实用程序将他们的用户名和密码提供给当前会话的内核密钥环。如果用户访问已挂载共享的内容，内核将使用来自内核密钥环的凭证，而不是最初用于挂载共享的凭证。

使用此功能包括以下步骤：

- [使用 `multiuser` 选项安装共享](#121-使用-multiuser-选项挂载共享)。
- [（可选）验证是否使用 `multiuser` 选项成功安装了共享](#122-验证是否使用-multiuser-选项挂载了-smb-共享)。
- [以用户身份访问共享](#123-以用户身份访问共享)。

**前提条件**

- `cifs-utils` 软件包已安装。

## 12.1 使用 multiuser 选项挂载共享

在用户可以使用自己的凭证访问共享之前，请使用权限有限的帐户以 `root` 用户身份安装共享。

**流程**

在系统引导时使用 `multiuser` 选项自动挂载共享：

1. 在 /etc/fstab 文件中为共享创建条目。例如：
   
   ```
   //server_name/share_name /mnt cifs multiuser,sec=ntlmssp,credentials=/root/smb.cred 0 0
   ```

2. 挂载共享：
   
   ```
   # mount /mnt/
   ```

如果您不想在系统启动时自动挂载共享，请通过将 `-o multiuser,sec=security_type` 传递给 `mount` 命令来手动挂载它。有关手动挂载 SMB 共享的详细信息，请参阅[手动挂载 SMB 共享](#113-手动挂载-smb-共享)。

## 12.2 验证是否使用  multiuser 选项挂载了 SMB 共享

若要验证共享是否是通过 multiuser 选项挂载的，显示挂载选项如下。

**流程**

```
# mount
...
//server_name/share_name on /mnt type cifs (sec=ntlmssp,multiuser,...)
```

如果 multiuser 条目显示在挂载选项列表中，则该功能已启用。

## 12.3 以用户身份访问共享

如果使用 multiuser 选项挂载 SMB 共享，用户可以将其服务器凭证提供给内核的密钥环：

```
# cifscreds add -u SMB_user_name server_name
Password: password
```

当用户在包含挂载的 SMB 共享的目录中执行操作时，服务器会为该用户提供文件系统权限，而不是在挂载共享时最初使用的权限。

> **注意**
> 
> 多个用户可以同时在挂载的共享上使用他们自己的凭证执行操作。

# 安装指南

### 概述

作为企业级Linux服务器操作系统，OpenCloudOS基于Linux内核自主研发设计，OpenCloudOS 8.5 是稳定的企业级服务器Linux发行版，其稳定性、安全性、兼容性和性能等核心能力均已得到充分验证。

OpenCloudOS的ISO目前支持安装方式是自动探测模式，引导之后安装就会执行安装操作，并格式化磁盘进行安装操作。执行安装操作之前，请务必确保您磁盘的数据已经备份，以免造成数据丢失。

### 1.介绍

OpenCloudOS 是企业级社区研发的定制化服务器操作系统。该系统集成了众多服务器系列的优点，加入自主研发的软件，便于用户操作使用，提供全方位（内核及用户态）的操作系统支持。系统特点：安全、易用、稳定、快速、长久支持。安装镜像提供了服务器常用的各种软件支持，同时可以使用线上软件源安装及更新软件。此说明适用于OpenCloudOS 发行版的安装与使用。

### 2.安装前准备

安装OpenCloudOS 服务器操作系统前，您的服务器需要满足以下要求：

1. 服务器接入稳定电源
2. 确保服务器至少拥有50GB硬盘空间，4GB内存空间
3. 获取安装DVD光盘（需要服务器拥有DVD光驱）或USB安装（需要服务器拥有USB接口）
4. 安装前请备份您的硬盘数据，以防数据丢失
5. 镜像获取地址：

* [http://mirrors.opencloudos.org/opencloudos/8/isos/](http://mirrors.opencloudos.org/opencloudos/8/isos/)
* [https://mirrors.tencent.com/opencloudos/8/isos/](https://mirrors.tencent.com/opencloudos/8/isos/)

### 3.光盘安装说明

* 插入安装光盘，启动时进入BIOS选择从CDROM驱动器启动
* 进入系统安装选择目录，选择cdrom方式进行安装

### 4.USB安装说明

* 使用UltraISO烧录iso镜像到USB介质中
* 插入USB安装介质，启动时从BIOS选择USB启动
* 进入安装目录，选择从usb安装介质安装操作系统

### 5.调整根分区大小

OpenCloudOS 默认的分区大小。

/ 分区20G，SWAP分区2G，/usr/local分区20G，/data分区为该硬盘剩余空间大小。

KVM虚拟机安装不支持rootsize，只有一个分区。



修改根分区方法：

* 进入安装目录，输入tab键，进入cmd命令模式
* 在命令尾部追加 rootsize=40，回车键完成修改，即可调整根分区大小为40GB

### 6.选择安装路径

OpenCloudOS 支持选择安装路径，在服务器存在多个硬盘时，可以选择安装到制定硬盘中。

* 进入安装后，短暂等待，当出现提示时，按任意键即可进入安装选择
* 如果不进行操作，则会在10s后自动安装到sda盘中
* 按下任意键后，出现如图硬盘选择
* 输入sda，系统将会安装到指定sda盘中
* 选择指定硬盘后，系统将自动分区，安装系统镜像

{% embed url="https://docs.qq.com/doc/DZk1mUUFyQVdzWkVz" %}

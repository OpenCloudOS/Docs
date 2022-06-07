# 图形化安装

#### 概述

作为企业级Linux服务器操作系统，OpenCloudOS基于Linux内核自主研发设计，OpenCloudOS 8.5 是稳定的企业级服务器Linux发行版，其稳定性、安全性、兼容性和性能等核心能力均已得到充分验证。

OpenCloudOS的ISO支持两种安装方式：

图形安装模式，用户根据需要选择语言，软件源，安装磁盘，网络配置等操作。本文着重介绍图形安装的过程。

自动探测模式，引导之后安装就会执行安装操作。

**1.介绍**

OpenCloudOS 是企业级社区研发的定制化服务器操作系统。该系统集成了众多服务器系列的优点，加入自主研发的软件，便于用户操作使用，提供全方位（内核及用户态）的操作系统支持。系统特点：安全、易用、稳定、快速、长久支持。安装镜像提供了服务器常用的各种软件支持，同时可以使用线上软件源安装及更新软件。此说明适用于OpenCloudOS 发行版的安装与使用。

**2.安装前准备**

安装OpenCloudOS 服务器操作系统前，您的服务器需要满足以下要求：

* 服务器接入稳定电源
* 确保服务器至少拥有50GB硬盘空间，4GB内存空间
* 获取安装DVD光盘（需要服务器拥有DVD光驱）或USB安装（需要服务器拥有USB接口）
* 安装前请备份您的硬盘数据，以防数据丢失
* 镜像获取地址：
* http://mirrors.opencloudos.org/opencloudos/8/
* https://mirrors.tencent.com/opencloudos/8/

**3.光盘安装说明**

* 插入安装光盘，启动时进入BIOS选择从CDROM驱动器启动
* 进入系统安装引导选择，选择Install进行安装

**4.USB安装说明**

* 使用Rufus 或者 dd命令行烧录iso镜像到USB介质中
* 插入USB安装介质，启动时从BIOS选择USB启动
* 进入系统安装引导选择，选择Install进行安装

{% embed url="https://docs.qq.com/doc/DZmt0TXJDcndTVFlw" %}

# 第7章 Nmstate简介

nmstate 是一个声明式的网络管理工具。 nmstate 软件提供 libnmstate Python库，以及用于管理NetworkManager的命令行工具 nmstatectl 。使用 Nmstate 时，您可以使用 YAML 或 JSON 格式的说明描述预期的网络状态。

使用Nmstate的优点：
- 提供稳定且可扩展的接口来管理 OpenCLoudOS 网络功能
- 支持主机和集群级别的原子和事务操作
- 支持对大多数属性进行部分编辑，并保留未在说明中指定的现有设置
- 提供插件支持，使管理员能够使用自己的插件

## 7.1.在Python程序中使用libnmstate库

libnmstate Python 库可让开发人员在他们自己的应用程序中使用 Nmstate

要使用库，请在源代码中导入它：
 ```python
import libnmstate
```

请注意，您必须先安装 nmstate 软件包才能使用这个库。

案例：使用 libnmstate 库查询网络状态

下面的代码导入了 libnmstate 库，并显示可用的网络接口及其状态：

```python
import json
import libnmstate
from libnmstate.schema import Interface

net_state = libnmstate.show()
for iface_state in net_state[Interface.KEY]:
    print(iface_state[Interface.NAME] + ": "
        + iface_state[Interface.STATE])
```

## 7.2.使用nmstatectl更新网络配置

您可以使用 nmstatectl 工具将一个或多个接口的当前网络配置存储在一个文件中。然后您可以使用此文件：

- 修改配置并将其应用到同一系统。
- 将文件复制到其他主机上，并使用相同的或经过修改的设置配置主机。

下面介绍了如何将 ens3 接口的设置导出到文件中，修改配置，并在主机上应用设置。

**前提条件**

- nmstate 软件包已安装。

**流程**

1. 将 ens3 接口的设置导出到 ~/network-config.yml 文件：
    ```
    # nmstatectl show ens3 > ~/network-config.yml
    ```
    此命令会以 YAML 格式存储 ens3 的配置。要以 JSON 格式存储输出，请将 --json 选项传给命令。

    如果没有指定接口名称，nmstatectl 将导出所有接口的配置。
2. 使用文本编辑器修改 ~/network-config.yml 文件，以更新配置。
3. 应用 ~/network-config.yml 文件中的设置：
    ```
    # nmstatectl apply ~/network-config.yml
    ```
    如果您以 JSON 格式导出设置，请将 --json 选项传给命令。

## 7.3.其他
    /usr/share/doc/nmstate/README.md
    /usr/share/doc/nmstate/examples/

# 第8章 配置以太网连接

## 8.1.使用nmcli配置静态以太网连接

**流程**

1. 添加新的NetworkManager网络连接配置集：
    ```
    # nmcli connection add con-name test-con ifname ens4 type ethernet
    ```
    修改 test-con 为您需要的网络连接配置集。
2. 设置IPv4地址：
    ```
    # nmcli connection modify test-con ipv4.addresses 192.128.1.1/24
    ```
3. 设置IPv6地址：
    ```
    #  nmcli connection modify test-con ipv6.addresses AD80::ABAA:0000:00C2:0002/64
    ```
4. 将 IPv4 和 IPv6 连接方法设置为 manual：
    ```
    #  nmcli connection modify test-con ipv6.method manual
    #  nmcli connection modify test-con ipv4.method manual
    ```
5. 设置 IPv4 和 IPv6 默认网关：
    ```
    #  nmcli connection modify test-con ipv4.gateway 192.128.1.254
    #  nmcli connection modify test-con ipv6.gateway AD80::ABAA:0000:00C2:FFFE
    ```
6. 设置 IPv4 和 IPv6 DNS 服务器地址：
    ```
    #  nmcli connection modify test-con ipv6.dns "AD80::ABAA:0000:00C2:0001"
    #  nmcli connection modify test-con ipv4.dns "114.114.114.114"
    ```
7. 为 IPv4 和 IPv6 连接设置 DNS 搜索域：
    ```
    #  nmcli connection modify test-con ipv4.dns-search test.com
    #  nmcli connection modify test-con ipv6.dns-search test.com
    ```
8. 激活连接配置集：
    ```
    # nmcli connection up test-con
    ```

**验证**

1. 显示设备和连接的状态：
    ```
    # nmcli device status
    DEVICE      TYPE      STATE      CONNECTION
    ens4      ethernet  connected  test-con
    ```
2. 显示网络连接配置集所有设置：
    ```
    # nmcli connection show test-con
    connection.id:              test-con
    connection.uuid:            b6cdfa1c-e4ad-46e5-af8b-a75f06b79f76
    connection.stable-id:       --
    connection.type:            802-3-ethernet
    connection.interface-name:  ens4
    ...
    ```
3. 使用 ping 命令验证网络连通性：
   - 同一子网：
        IPv4:
        ```
        # ping 192.128.1.3
        ```
        IPv6:
        ```
        # ping AD80::ABAA:0000:00C2:0005
        ```
        如果命令失败，请检查IP和子网设置。
        
   - 远程子网：
        IPv4:
        ```
        # ping 192.168.1.3
        ```
        IPv6:
        ```
        # ping AD80::ABAA:0000:00C3:0005
        ```
        - 如果命令失败，先 ping 默认网关验证设置。
            IPv4:
            ```
            # ping 192.128.1.254
            ```
            IPv6:
            ```
            # ping AD80::ABAA:0000:00C2:FFFE
            ```
4. 使用 host 命令验证域名解析是否正常：
    ```
    # host client.test.com
    ```
    如果命令返回任何错误，如 connection timed out 或 no servers could be reached，请验证您的 DNS 设置。

**故障排除步骤**
如果连接失败，或者网络接口在上线和关闭状态间切换：

- 确保网络电缆插入到主机和交换机。
- 检查连接失败是否只存在于这个主机上，或者其他连接到该服务器连接的同一交换机的主机中。
- 验证网络电缆和网络接口是否如预期工作。执行硬件诊断步骤并替换有缺陷的电缆和网络接口卡。
- 如果磁盘中的配置与设备中的配置不匹配，则启动或重启 NetworkManager 会创建一个代表该设备的配置的内存连接。

## 8.2.使用nmcli互动编辑器配置静态以太网连接

**流程**

1. 以互动模式添加 NetworkManager 网络连接配置集：
    ```
    # nmcli connection edit type ethernet con-name test-con
    ```
2. 设置网络接口：
    ```
    nmcli> set connection.interface-name ens4
    ```
3. 设置IPv4地址：
    ```
    nmcli> set ipv4.addresses 192.128.1.1/24
    ```
4. 设置IPv6地址：
    ```
    nmcli> set ipv6.addresses AD80::ABAA:0000:00C2:0002/64
    ```
5. 将 IPv4 和 IPv6 连接方法设置为 manual：
    ```
    nmcli> set ipv4.method manual
    nmcli> set ipv6.method manual
    ```
6. 设置 IPv4 和 IPv6 默认网关：
    ```
    nmcli> set ipv4.gateway 192.128.1.254
    nmcli> set ipv6.gateway AD80::ABAA:0000:00C2:FFFE
    ```
7. 设置 IPv4 和 IPv6 DNS 服务器地址：
    ```
    nmcli> set ipv4.dns 114.114.114.114
    nmcli> set ipv6.dns AD80::ABAA:0000:00C2:0001
    ```
    要设置多个 DNS 服务器，以空格分隔并用引号括起来。
8. 为 IPv4 和 IPv6 连接设置 DNS 搜索域：
    ```
    nmcli> set ipv4.dns-search example.com
    nmcli> set ipv6.dns-search example.com
    ```
9. 保存并激活连接：
    ```
    nmcli> save persistent
    Saving the connection with 'autoconnect=yes'. That might result in an immediate activation of the connection.
    Do you still want to save? (yes/no) [yes] yes
    ```

10. 退出互动模式：
    ```
    nmcli> quit
    ```

**验证**

1. 显示设备和连接的状态：
    ```
    # nmcli device status
    DEVICE      TYPE      STATE      CONNECTION
    ens4      ethernet  connected  test-con
    ```
2. 显示网络连接配置集所有设置：
    ```
    # nmcli connection show test-con
    connection.id:              test-con
    connection.uuid:            b6cdfa1c-e4ad-46e5-af8b-a75f06b79f76
    connection.stable-id:       --
    connection.type:            802-3-ethernet
    connection.interface-name:  ens4
    ...
    ```
3. 使用 ping 命令验证网络连通性：
   - 同一子网：
        IPv4:
        ```
        # ping 192.128.1.3
        ```
        IPv6:
        ```
        # ping AD80::ABAA:0000:00C2:0005
        ```
        如果命令失败，请检查IP和子网设置。
        
   - 远程子网：
        IPv4:
        ```
        # ping 192.168.1.3
        ```
        IPv6:
        ```
        # ping AD80::ABAA:0000:00C3:0005
        ```
        - 如果命令失败，先 ping 默认网关验证设置。
            IPv4:
            ```
            # ping 192.128.1.254
            ```
            IPv6:
            ```
            # ping AD80::ABAA:0000:00C2:FFFE
            ```
4. 使用 host 命令验证域名解析是否正常：
    ```
    # host client.test.com
    ```
    如果命令返回任何错误，如 connection timed out 或 no servers could be reached，请验证您的 DNS 设置。

**故障排除步骤**
如果连接失败，或者网络接口在上线和关闭状态间切换：

- 确保网络电缆插入到主机和交换机。
- 检查连接失败是否只存在于这个主机上，或者其他连接到该服务器连接的同一交换机的主机中。
- 验证网络电缆和网络接口是否如预期工作。执行硬件诊断步骤并替换有缺陷的电缆和网络接口卡。
- 如果磁盘中的配置与设备中的配置不匹配，则启动或重启 NetworkManager 会创建一个代表该设备的配置的内存连接。

## 8.3.使用nmstatectl配置静态以太网连接

**前提条件**

- 已安装 nmstate 。

**流程**

1. 创建一个 YAML 文件 ~/create-ethernet-profile.yml 包含以下内容：
    ```
    ---
    dns-resolver:
    config:
        search: []
        server:
        - 114.114.114.114
    route-rules:
    config: []
    routes:
    config:
    - destination: 0.0.0.0/0
        metric: 100
        next-hop-address: 192.168.128.1
        next-hop-interface: ens3
        table-id: 254
    interfaces:
    - name: ens3
    type: ethernet
    state: up
    accept-all-mac-addresses: false
    ipv4:
        enabled: true
        address:
      - ip: 192.168.133.95
        prefix-length: 20
      dhcp: false
    ipv6:
      enabled: true
      address:
      - ip: fe80::f816:3eff:fec6:ce86
        prefix-length: 64
      auto-dns: true
      auto-gateway: true
      auto-route-table-id: 0
      auto-routes: true
      autoconf: true
      dhcp: true
    ```
2. 将配置应用到系统：
    ```
    # nmstatectl apply ~/create-ethernet-profile.yml
    ```

**验证**

1. 显示设备和连接的状态：
    ```
    # nmcli device status
    DEVICE      TYPE      STATE      CONNECTION
    ens3      ethernet  connected  ens3
    ```
2. 显示连接配置集的所有设置：
    ```
    # nmcli connection show enp7s0
    connection.id:              ens3
    connection.uuid:            b6cdfa1c-e4ad-46e5-af8b-a75f06b79f76
    connection.stable-id:       --
    connection.type:            802-3-ethernet
    connection.interface-name:  ens3
    ...
    ```
3. 以 YAML 格式显示连接设置：
    ```
    # nmstatectl show ens3
    ```

## 
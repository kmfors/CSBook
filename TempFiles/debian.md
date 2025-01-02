更改主机名：

```shell
/etc/hostname
```

解析 DNS ：`/etc/resolv.conf`

```shell
nameserver 192.168.11.1
```

IP地址分配到一个主机，那么这个主机一定不能够直接访问互联网，必须通过一个作为网关的代理服务或通过 [网络地址转换 Network Address Translation (NAT)](https://zh.wikipedia.org/wiki/Network_address_translation). 消费局域网环境，宽带路由器通常使用 NAT。



## 网络简介

`systemd` 是一个系统和服务管理器，`systemd` 使用易于理解和配置的单元文件（以 `.service`、`.socket`、`.path` 等结尾的文件），这些文件定义了服务的行为。`systemd` 提供了 `systemctl` 命令，用于控制和查询系统状态，如启动、停止、重启服务，以及查看服务状态。

对于使用 `systemd` 的现代 Debian 桌面系统，网络接口通常由两个服务进行初始化：`lo` 接口通常在“`networking.service`”处理，而其它接口则由“`NetworkManager.service`”处理。

Debian 可以通过[后台守护进程（daemon）](https://zh.wikipedia.org/wiki/Daemon_(computer_software))管理软件来管理网络连接，例如 [NetworkManager (NM)](https://zh.wikipedia.org/wiki/NetworkManager)（network-manager 和相关软件包）。这些现代的网络配置工具需要进行适当的配置，以避免与传统 `ifupdown` 软件包发生冲突，它的配置文件位于 “`/etc/network/interfaces`”。



### 没有图像界面的现代网络配置

使用 [systemd](https://zh.wikipedia.org/wiki/Systemd) 的系统中，可以在 `/etc/systemd/network/` 里配置网络

DHCP 客户端的配置可以通过创建 "`/etc/systemd/network/dhcp.network`" 文件来进行设置。例如：

```shell
[Match]
Name=en*

[Network]
DHCP=yes
```

一个静态网络配置能够通过创建 "`/etc/systemd/network/static.network`" 来设置.比如：

```shell
[Match]
Name=en*

[Network]
Address=192.168.0.15/24
Gateway=192.168.0.1
```



使用`/etc/network/interfaces`文件

```shell
# /etc/network/interfaces
sudo systemctl restart networking

# /etc/systemd/network/dhcp.network
sudo systemctl restart systemd-networkd
```

sudo systemctl enable systemd-networkd.service



https://www.debian.org/doc/manuals/debian-reference/ch05.zh-cn.html#_the_modern_network_configuration_for_cloud



---



### 方法1：查看 `init` 系统

你可以在终端中运行以下命令来查看当前使用的 `init` 系统：

bash

```bash
ps -p 1 -o comm=
```

这个命令会显示系统的第一个进程（即 `init` 进程），其命令名称通常会告诉你使用的是哪个 `init` 系统。如果输出是 `init`，那么可能是 `systemd`、`upstart` 或 `sysvinit` 之一。为了更准确地确定，你可以检查 `sbin` 目录下的 `init` 可执行文件：

bash

```bash
ls -l /sbin/init
```

如果指向 `systemd`，那么系统使用的就是 `systemd`。

---

sudo systemctl status systemd-networkd

查看 `systemd-networkd` 的状态：

```shell
sudo systemctl status systemd-networkd
```

输出如下：

```shell
○ systemd-networkd.service - Network Configuration
     Loaded: loaded (/lib/systemd/system/systemd-networkd.service; disabled; preset: enabled)
     Active: inactive (dead)
TriggeredBy: ○ systemd-networkd.socket
       Docs: man:systemd-networkd.service(8)
             man:org.freedesktop.network1(5)
```

注意：

- `disabled`：表示服务没有被设置为系统自启动。
- `preset: enabled`：服务预设状态为启用。
- `Active: inactive (dead)`：服务当前没有运行，且无与任何进程相关联。

```shell
# 确保服务在每次系统启动时都自动运行
sudo systemctl enable systemd-networkd.service

# 即启动服务而不关心它是否在下次启动时运行
sudo systemctl start systemd-networkd.service
```


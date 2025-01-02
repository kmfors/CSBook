# Debian12 配置静态 IP 地址

`/etc/network/interfaces` 文件是 Debian 及其衍生系统（如 Ubuntu）中网络配置的标准文件，它被 `ifup` 和 `ifdown` 命令用来控制网络接口的启动和关闭。这个文件通常用于静态配置网络接口，即手动指定 IP 地址、子网掩码、网关等信息。

---



编辑 `/etc/network/interfaces` 网络文件



```shell
#iface eno1 inet dhcp  # 配置动态
iface eno1 inet static # 配置静态

address 192.168.20.10  # 随意配置
netmask 255.255.255.0
gateway 192.168.20.1   # 网关地址末位最好写1
```

编辑 `/etc/resolv.conf` 解析配置文件

```shell
nameserver 8.8.8.8
nameserver 114.114.114.114
```

```text
systemctl restart networking
```

nmcli 是 NetworkManager 的命令行工具，用于管理和配置网络连接。它可以用于查看当前网络连接的状态、配置新的网络连接、启用和禁用网络连接等操作。nmcli 支持大多数常见的网络连接类型，如以太网、Wi-Fi、VPN 等。通过 nmcli 命令，用户可以方便地在命令行界面下管理网络连接，而无需依赖图形界面工具。




```shell
connected：已被NM托管，并且当前有活跃的connection
disconnected：已被NM托管，但是当前没有活跃的connection
unmanaged：未被NM托管，就是不让NM动这个设备相关的任何操作
unavailable：不可用，NM无法托管，通常出现于网卡设备未启用的时候
```


报错 1

```
(/org/freedesktop/NetworkManager/Devices/2) failed: Device is not activated
```
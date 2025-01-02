

**例 8.2. 静态配置**

auto enp0s31f6
iface enp0s31f6 inet static
  address 192.168.0.3/24
  broadcast 192.168.0.255
  network 192.168.0.0
  gateway 192.168.0.1

Netplan、NetworkManager、Systemd-networkd 关系
Netplan 支持调用 NetworkManager 和 Systemd-networkd；
NetworkManager和systemd-networked可以理解为相互替代关系。
如果要禁用NetworkManager，则应启用systemd-networkd，而在systemd-networkd运行时最好禁用networkmanager。

```shell
apt install network-manager
apt install netplan.io
```
————————————————

                            版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
                        
原文链接：https://blog.csdn.net/qq_38880380/article/details/135420863

https://blog.csdn.net/u012964600/article/details/137362011

查看[网络设备](https://so.csdn.net/so/search?q=%E7%BD%91%E7%BB%9C%E8%AE%BE%E5%A4%87&spm=1001.2101.3001.7020)列表

```shell
 sudo nmcli dev
```

注意，如果列出的设备状态全部是 unmanaged 的，说明这些网络设备不受NetworkManager管理，你需要清空 /etc/network/interfaces下的网络设置,然后重启.

wlp2s0

扫描可用的无线网络

```shell
sudo iwlist <WIRELESS_INTERFACE> scan | grep ESSID
# 将 `<WIRELESS_INTERFACE>` 替换为你的无线网卡接口，例如 `wlan0`,有用！
```

  https://blog.csdn.net/qq_62214908/article/details/134975199?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-134975199-blog-125116027.235^v43^pc_blog_bottom_relevance_base2&spm=1001.2101.3001.4242.1&utm_relevant_index=3

1. `nmcli device wifi list`
    
    这将列出附近可用的 WiFi 网络以及它们的相关信息，包括名称（SSID）、信号强度、加密类型等。
    
2. **连接到 WiFi 网络：** 选择你想要连接的 WiFi 网络，并使用以下命令连接（替换 `SSID` 和 `PASSWORD` 为你的 WiFi 网络的名称和密码）：
    
    bashCopy code
    
    `nmcli device wifi connect SSID password PASSWORD`
    
    如果 WiFi 网络是开放的（无密码），则无需提供密码参数。

重启网络服务： sudo systemctl restart NetworkManager

nmcli connection down wifi名

`nmcli connection down CONNECTION_NAME` 和 `nmcli device disconnect wlan0` 
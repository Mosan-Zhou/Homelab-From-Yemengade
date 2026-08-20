# Homelab-From-Yemengade

闲着无聊，搞了一台极夜的T2mini机，配置还不错。

<img width="333" height="133" alt="image" src="https://github.com/user-attachments/assets/4c3b5ad3-d818-4d61-916b-c5653ce3a050" />

系统选择了Proxmox-VE_9.2-1，个人homelab用着正合适。
安装系统比较简单，就是先这样再这样最后那样就可以了，安装好系统之后配置网络。
nano /etc/network/interfaces
auto lo
iface lo inet loopback

iface enp1s0 inet manual   # enp1s0是你的物理网口名，可能不同，保持原样

auto vmbr0
iface vmbr0 inet static
      address 192.168.1.0/24    # 给PVE设置一个固定IP，这里我直接设置了一整个网段
      gateway 192.168.1.0         # 网关随便填了，反正直连用不上
      bridge-ports enp1s0
      bridge-stp off
      bridge-fd 0

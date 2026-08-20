# Homelab-From-Yemengade

闲着无聊，搞了一台极夜的T2mini机，配置还不错。

<img width="333" height="133" alt="image" src="https://github.com/user-attachments/assets/4c3b5ad3-d818-4d61-916b-c5653ce3a050" />

## 2026.8.16
### 安装系统
系统选择了Proxmox-VE_9.2-1，个人homelab用着正合适。
安装系统比较简单，就是先这样再这样最后那样就可以了，安装好系统之后配置网络。

```
nano /etc/network/interfaces auto lo iface lo inet loopback
```
```
   iface enp1s0 inet manual   # enp1s0是你的物理网口名，可能不同，保持原样
   
   auto vmbr0 iface vmbr0 inet static
         address 192.168.1.10/24    # 给PVE设置一个固定IP
         gateway 192.168.1.0         # 网关随便填了，反正直连用不上
         bridge-ports enp1s0
         bridge-stp off
         bridge-fd 0
```
然后重启系统
```
reboot
```
### 访问PVE管理后台
由于手头没有交换机，所以直接网线连接主力机和极夜T2。
在浏览器里输入：`https://192.168.1.100:8006` （注意是https），就能打开PVE的Web管理界面了。
发现无法进入后台，也无法ping通，怀疑是pve防火墙的问题，于是关闭防火墙：
```
systemctl stop pve-firewall
systemctl stop pvefw-logger
```
修改防火墙配置
```
nano /etc/pve/firewall/cluster.fw
```
```                                                                                                                 
	[RULES]
	# 允许自己的 IP 访问 PVE Web 和 SSH ，为了方便不同设备调试，直接白名单整个网段
	IN ACCEPT -source 192.168.1.0/24 -destport 8006,22 -proto tcp

	# 因为需要 Ping，所以加上这条
	IN ACCEPT -source 192.168.1.0/24 -proto icmp

	# 下面这两条不是必须，但为了防止被锁，加上了
	IN ACCEPT -source 192.168.1.0/24 -destport 53 -proto udp # DNS
```
重启防火墙
```
pve-firewall compile     # 编译规则
pve-firewall restart     # 重启防火墙生效
```
成功登录后台
<img width="2559" height="1271" alt="image" src="https://github.com/user-attachments/assets/5c84982d-f017-49e9-b48f-1b5430ce5646" />


## 2026.8.20
### 通过MobaXterm SSH登录pve
<img width="2560" height="1392" alt="image" src="https://github.com/user-attachments/assets/37883579-7355-4e42-bcd0-2509d59fbb66" />

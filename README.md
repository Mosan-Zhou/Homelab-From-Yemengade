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
         address 10.10.10.10/24    # 给PVE设置一个固定IP
         gateway 10.10.10.10         # 网关随便填了，反正直连用不上
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
在浏览器里输入：`https://10.10.10.10:8006` （注意是https），就能打开PVE的Web管理界面了。
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
	IN ACCEPT -source 10.10.10.0/24 -destport 8006,22 -proto tcp

	# 因为需要 Ping，所以加上这条
	IN ACCEPT -source 10.10.10.0/24 -proto icmp

	# 下面这两条不是必须，但为了防止被锁，加上了
	IN ACCEPT -source 10.10.10.0/24 -destport 53 -proto udp # DNS
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

## 2026.8.26
### 通过LXC和Docker搭建一个下载姬
#### 构建集群A1
<img width="2250" height="960" alt="image" src="https://github.com/user-attachments/assets/1e7f6720-e9da-431a-a2cf-48050d9ceb97" />

#### 1.添加CT模板
<img width="2880" height="1704" alt="image" src="https://github.com/user-attachments/assets/a5c06933-01ea-47ca-a34a-df622d079c14" />

#### 2.创建LXC
<img width="1510" height="1140" alt="image" src="https://github.com/user-attachments/assets/8ce1e0fe-a5a3-43f6-bd76-0feb453ca309" />

```                                                                                                                 
	常规：设置主机名和密码，取消勾选“无特权的容器”。
	模板：选择刚下载的 Debian 12 模板。
	磁盘/CPU/内存： CPU 1核，内存 1GB，磁盘30G（由于SSD就256G，暂时分配30G，欧需根据需求可能添加新的SSD）。
	网络：设置静态IP 10.10.10.101，网关10.10.10.1。
	启用功能：创建后，在容器侧边栏的 选项 -> 功能 中，勾选“嵌套”。
```

修改配置文件
```                                                                                                                 
nano /etc/pve/lxc/100.conf
```

添加以下三行
```                                                                                                                 
	lxc.apparmor.profile: unconfined
	lxc.cgroup.devices.allow: a
	lxc.cap.drop:
```

#### 3.在 LXC 内安装 Docker 和 qBittorrent
更换软件源
通过脚本一步到位
```
bash <(curl -sSL https://linuxmirrors.cn/main.sh)
```

安装 Docker
```
bash <(curl -sSL https://linuxmirrors.cn/docker.sh)
```

安装 qBittorrent，使用 docker-compose 来安装和管理
```
apt install docker-compose-plugin
```

创建一个项目目录
```
	version: "3"
	services:
	  qbittorrent:
	    image: linuxserver/qbittorrent:latest
	    container_name: qbittorrent
	    environment:
	      - PUID=1000
	      - PGID=1000
	      - TZ=Asia/Shanghai
	      - WEBUI_PORT=8080
	    volumes:
	      - /path/to/config:/config
	      - /path/to/downloads:/downloads
	    ports:
	      - 8080:8080
	      - 6881:6881
	      - 6881:6881/udp
	    restart: unless-stopped
```

在 docker-compose.yml 文件所在目录执行
```
docker compose up -d
```

#### 4.登录qBittorrent Web
<img width="2878" height="1622" alt="image" src="https://github.com/user-attachments/assets/a735ac5a-0a36-477b-bce2-7c3a885b05e8" />
需要注意的是，qBittorrent Web默认用户是admin，但默认密码并非很多帖子中提到的adminadmin，而是在临时日志中显示，这个特性在很早之前就更新了。


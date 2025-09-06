# Linux 云平台服务配置

# 1、系统安装

#### **1. 通过 PC1 浏览器连接并安装 Rocky Linux ARM64**

1. **访问安装界面**：
   在 PC1 浏览器中输入 Server2 的 IP 地址（假设为初始 DHCP 分配的地址），通过 VNC 或 Web 管理界面访问安装程序。
2. **安装 Rocky Linux ARM64 CLI**：
   - 选择英文语言（English）。
   - 配置磁盘分区（推荐使用默认自动分区）。
   - 设置 root 密码：`Key-1122`。
   - 完成安装并重启。

# 2、登录server2后：

编辑网络配置文件 /etc/NetworkManager/system-connections/ens160.nmconnection

```bash
# 添加/修改以下内容
BOOTPROTO=static
IPADDR=10.10.40.100
PREFIX=24
GATEWAY=10.10.40.1  # 根据实际网关修改
DNS1=10.10.40.101    # DNS服务器（后续配置）
ONBOOT=yes
```

# 3、安装qemu、libvirt 和virt-install

```bash
# 安装qemu、libvirt和virt-install
dnf install -y qemu-kvm libvirt virt-install virt-viewer

# 启动并启用libvirtd服务
systemctl enable  libvirtd
```

关闭防火墙启动libvirt

systemctl stop firewalld.service

systemctl disable firewalld.service

systemctl restart libvirtd

 systemctl enable libvirtd

1、创建桥接网络

 nmcli connection add  type  bridge ifname br0d

桥接网卡绑定到物理网卡

nmcli connection add type bridge-slave ifname  ens160  master br0

重启网路服务

 systemctl restart NetworkManager

将网络设置为桥接网卡

2、创建磁盘文件

创建并配置Linux0虚拟机

sudo qemu-img create -f qcow2 /root/linux0.qcow2 80G
sudo qemu-img create -f qcow2 /root/linux1.qcow2 80G

以此类推，创建好所有文件。	

## 4、创建rocky-arm64 虚拟机

3、安装linux0（使用Rocky Linux 9 ARM64镜像）

创建虚拟机Linux0

| **命令部分**                                          | **解释**                                                     |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| `sudo virt-install`                                   | 以管理员权限运行 `virt-install` 命令，用于创建和启动虚拟机。 |
| `--name linux0`                                       | 指定虚拟机的名称为 `linux0`。                                |
| `--ram 4096`                                          | 分配 4096MB（4GB）内存给虚拟机。                             |
| `--disk path=/root/linux0.qcow2,size=80,format=qcow2` | 指定虚拟机的磁盘文件路径为 `/root/linux0.qcow2`，大小为 80GB，格式为 `qcow2`。 |
| `--vcpus 2`                                           | 分配 2 个虚拟 CPU 给虚拟机。                                 |
| `--cdrom /opt/Rocky-9.5-x86_64-boot.iso`              | 指定使用 `/opt/Rocky-9.5-x86_64-boot.iso` 作为虚拟机的安装介质。 |
| `--network bridge=br0`                                | 将虚拟机的网络接口桥接到主机的 `br0` 网络桥接。              |
| `--graphics vnc,port=5900`                            | 启用 VNC 图形界面，允许通过 VNC 客户端连接到虚拟机的图形控制台。`port=5900`：指定 VNC 服务的端口号为 `5900`。 |
| `--noautoconsole`                                     | 在虚拟机启动时不自动连接到控制台。                           |



sudo virt-install \
--name linux0 \
--ram 4096 \
--disk path=/root/linux0.qcow2,size=80,format=qcow2 \
--vcpus 2 \
--cdrom /opt/Rocky-9.5-x86_64-boot.iso \
--network bridge=br0 \
--graphics vnc,port=5900 \
--noautoconsole

![image-20250615100622501](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250615100622501.png)

为了让 `qemu` 用户能够访问 `/root` 目录中的文件，你需要调整 `/root` 目录的权限，使其对 `qemu` 用户可读。

sudo chmod 750 /root
sudo chown root:qemu /root


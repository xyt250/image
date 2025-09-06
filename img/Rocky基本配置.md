# **1、DNS服务器搭建**

## 编写配置文件时要仔细检查

### 1、安装DNS服务：

 yum install -y bind*

![image-20250525082334233](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250525082334233.png)

### 2、配置DNS文件

### 3、配置named.conf

vim /etc/named.conf

将第一行和最后一行括号内的内容全部替换为"any"

![image-20250525082649549](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250525082649549.png)

### 4、配置named.rfc1912.zones

vim /etc/named.rfc1912.zones

最上面那一段不用删除

将所有内容都删除只剩下最后两段，

第一行引号下填写域名 

 第二段的第一行填写反向IP反向IP只写到网络位，不写主机位，例如你的网络位是10.10.60，那么这里的反向IP就应该填写60.10.10

60.10.10.in-addr.arpa  正确的格式

第三行依次填写正向解析文件名和反向解析文件名

![image-20250525103822202](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250525103822202.png)

### 5、复制正反向解析文件

去到/var/named/目录内

随便复制这两个其中的一个文件就可以了，我选的是两个都复制

![image-20250525095752542](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250525095752542.png)

复制正向解析文件a

cp -p named.localhost ninework.com

复制反向解析文件b

cp -p named.localhost 60.10.10.zone

### 5.1 编辑DNS正向解析文件

![image-20250525084415156](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250525084415156.png)

左边填写域名	右边填写域名并在末尾加上英文. 

和对应的IP地址

![image-20250525111306846](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250525111306846.png)

### 5.2编辑DNS反向解析文件

左边填写主机位  右边填写域名并在末尾加上英文.

![image-20250525111330110](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250525111330110.png)

 named-checkzone ninework.com /var/named/ninework.com

named-checkzone 20.168.192.zone /var/named/20.168.192.zone

检查配置文件是否错误

![image-20250526174617816](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526174617816.png)

**重启Bind服务**

Bind的进程名叫named

 systemctl restart named

### 6、虚拟机DNS指向本机

网卡配置文件在/etc/NetworkManager/system-connections/enp1s0.nmconnection内

![image-20250525090308998](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250525090308998.png)

### 7、加载激活网卡配置文件

加载网卡配置文件

nmcli connection load /etc/NetworkManager/system-connections/enp1s0.nmconnection

激活网卡配置文件

 nmcli connection up /etc/NetworkManager/system-connections/enp1s0.nmconnection

### 能够解析到的DNS配置才算成功

![image-20250525110814553](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250525110814553.png)



### 以下是配置错误的情况

![image-20250525090656909](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250525090656909.png)

如果配置文件没有出现明显错误大概率是文件权限问题将权限给成755再试试

chmod 755 ninework.zone

chmod 755 20.168.192.zone

# **2、DNS主从服务器配置**

要求需要三台主机

| 主机名称     | IP            | 首选DNS指向   | 备选DNS指向   | 扮演角色      | 系统版本      |
| ------------ | ------------- | ------------- | ------------- | ------------- | ------------- |
| Rocky-Master | 192.168.20.20 | 192.168.20.20 | /             | DNS主服务器   | Rocky_dvd_9.5 |
| Rocky-Slave  | 192.168.20.21 | 192.168.20.20 | /             | DNS从服务器   | Rocky_dvd_9.5 |
| Rocky-Client | 192.168.10.22 | 192.168.20.20 | 192.168.20.21 | DNS验证客户端 | Rocky_dvd_9.5 |

**这里我使用的IP是10.10.60.20 10.10.60.21 10.10.60.22**

三台主机都要完成防火墙和  SElinux的关闭（核心防火墙） systemctl stop firewalld.service systemctl disable firewalld.service 

setenforce 0（注意此命令只是临时关闭核心防火墙）



## 1、配置Master

### 1、安装Bind

yum install -y bind*

![image-20250525143533203](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525143533203.png)

### 2、修改DNS指向

Master主机修改ens160文件：

 vim /etc/NetworkManager/system-connections/ens160.nmconnection

DNS指向本机

![image-20250525144228404](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525144228404.png)

重新激活网卡配置：

nmcli connection load /etc/NetworkManager/system-connections/ens160.nmconnection    

nmcliconnection up /etc/NetworkManager/system-connections/ens160.nmconnection

修改主机名

直接在hostname文件内写入主机名修改完成后，重启生效：init 6 或者 reboot

 vim /etc/hostname

![image-20250525145241560](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525145241560.png)

### 3、修改主配置文件

 vim /etc/named.conf

![image-20250525145411755](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525145411755.png)

将第一行和最后一行括号内容修改为any或本机IP地址

![image-20250525145500204](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525145500204.png)

### 4、修改区域配置文件

最好备份以下区域配置文件，如果写错了还可以重新写

备份文件： cp -p /etc/named.rfc1912.zones /etc/named.rfc1912.zones.bak

修改配置文件 ：vim /etc/named.rfc1912.zones

![image-20250525145848783](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525145848783.png)

1、第一行双引号内容替换为域名  第三行替换为正向解析文件名称

2、花括号内的none替换为DNS从服务器的IP地址

3、下一段的第一行把0替换为反向网络位IP地址

不清楚自己的IP可以通过 ifconfig 查看 前三段是网络位置 反向地址要从后往前填写

例如 192.168.10=10.168.192

4、下面的第三行引号中间填写反向解析文件的文件名称

5、最后一行花括号内填写DSN从服务器的IP地址

![image-20250525151124734](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525151124734.png)

### 5、编写正反向解析文件

先进入到/var/name目录下

cd /var/name

复制正反向解析文件模板

cp -p named.localhost a

cp -p named.loopback fx

![image-20250525151448904](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525151448904.png)

修改正向解析文件

vim a

![image-20250525151906387](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525151906387.png)

修改反向解析文件 vim fx

![image-20250525152122795](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525152122795.png)

6、重启Bind服务

重启Bind : systemctl restart named

查看Bind的状态： systemctl status named

![image-20250525152258379](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525152258379.png)

## 2、配置slave

### 1、安装Bind

yum install -y bind*

![image-20250525143533203](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525143533203.png)

### 2、修改DNS指向

由于slave是DNS辅助服务器，并不存储DNS记录，是从master主机获取DNS记录的，所以slaveDNS指向

为master

![image-20250525152646721](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525152646721.png)

重新激活网卡配置：

nmcli connection load /etc/NetworkManager/system-connections/ens160.nmconnection    

nmcliconnection up /etc/NetworkManager/system-connections/ens160.nmconnection

### 3、修改主机名

vim /etc/hostname

修改完成后重启 init 6 或者 reboot

### 4、修改主配置文件

vim /etc/named.conf

![image-20250525152854568](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525152854568.png)

将第一行和最后一行花括号内内容改为 any

![image-20250525153100400](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525153100400.png)

### 5、修改区域配置文件

vim /etc/named.rfc1912.zones

保留最后两段

![image-20250525153254727](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525153254727.png)

1、将第一行改为你的域名 type后面的master 修改为slave

2、由于是slave是从master主机获取DNS记录的，所以slave这边的正反向文件不参与可以注释掉

3、最后一行也需要注释掉

在下面添加一行 masters {10.10.60.20;};

4、下一段的第一行双引号内添加反向网络位IP

也把类型由master 修改为slave 注释掉下面的两句话 ，然后添加指向masters 的IP记录 masters {192.168.10.181;};



![image-20250525154151991](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525154151991.png)

修改完成后保存并退出 ，重启named

systemctl restart named

systemctl enable named

systemctl status named

![image-20250525154350882](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525154350882.png)

## 3、client验收环节

修改DNS指向master以及slave 

vim /etc/NetworkManager/system-connections/enp1s0.nmconnection

![image-20250525154536969](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525154536969.png)

首先在client输入nslookup 域名

也可以输入nslookup IP

![image-20250525154743363](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525154743363.png)

然后停掉master的DNS服务

systemctl stop named

然后再次在client中输入nslookup域名 输入nslookup IP也能解析到

![image-20250525154957499](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525154957499.png)

# **3、FTP服务配置**

**1-FTP服务概述：**

1. FTP是File Transfer Protocol（文件传输协议）的英文简称，它工作在OSI模型的第七层，TCP模型的第四层上，使用TCP传输而不是UDP，这样FTP客户在和服务器建立连接前就要经过一个被广为熟知的”三次握手”的过程，它带来的意义在于客户与服务器之间的连接是可靠的，而且是面向连接，为数据的传输提供了可靠的保证。
2. FTP服务使用FTP协议（文件传输协议）来进行文件的上传和下载，可以非常方便的进行远距离的文件传输，还支持断点续传功能，可以大幅度地减小CPU和网络带宽的开销，并实现相应的安全控制。
3. FTP协议中，控制连接均有客户端发起，而数据连接有两种工作方式：PORT(主动)方式和PASV(被动)方式

**2-FTP登入概述：**
FTP可以有三种登入方式，分别是：

1. 匿名登录方式：不需要用户密码
2. 本地用户登入：使用本地用户和密码登入
3. 虚拟用户方式：也是使用用户和密码登入，但是该用户不是linux中创建的用户



| 名称         | IP          | 扮演角色  |
| ------------ | ----------- | --------- |
| Rocky-Master | 10.10.60.20 | FTP服务器 |
| Win-Client   | 10.10.60.30 | FTP客户端 |

## 1、安装FTP服务

yum install -y vsftpd

![image-20250525160031765](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525160031765.png)

安装完成之后在/etc/vsftpd/路径下会存在三个配置文件:

![image-20250525160200538](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525160200538.png)

1. vsftpd.conf: vsftp主配置文件
2. ftpusers: 指定哪些用户不能访问FTP服务器,这里的用户包括root在内的一些重要用户。
3. user_list: 指定的用户是否可以访问ftp服务器，通过vsftpd.conf文件中的userlist_deny的配置来决定配置中的用户是否可以访问，userlist_enable=YES ，userlist_deny=YES ，userlist_file=/etc/vsftpd/user_list 这三个配置允许文件中的用户访问FTP

## 2、编写vsftpd,conf

查看主配置文件缺省配置：

cat /etc/vsftpd/vsftpd.conf | grep -v '^#';

备份主配置文件：

cp -p vsftpd.conf vsftpd.conf.bak

重建vsftpd.conf 文件：

重命名为vsftpd.old

mv vsftpd.conf vsftpd.old

新建ftp配置文件：

touch /etc/vsftpd/vsftpd.conf

编写ftp配置文件：

vim vsftpd.conf	

anonymous_enable=YES
write_enable=YES
local_umask=022
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
anon_root=/var/ftp/pub
ftp_username=ftp
ftpd_banner=Welcome To My FTP Site

## 3、设置映射文件夹权限

设置FTP所映射的文件夹/var/ftp/pub权限为755

chmod 755 /var/ftp/pub/

## 4、测试环节

重启vsftpd:

systemctl restart vsftpd

防火墙允许通过21端口：

 firewall-cmd  --zone=public --add-port=21/tcp --permanent

或者直接关闭防火墙

 systemctl stop firewalld.service

客户端关闭防火墙

![image-20250525162606449](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525162606449.png)



测试连通：

![image-20250525162540906](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525162540906.png)

使用命令进行连接FTP：

ftp 10.10.60.20 

输入匿名用户"ftp",密码为空，然后就可以连接成功了

![image-20250525162753058](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250525162753058.png)

**2):使用可视化客户端"FileZilla"测试：**
"主机"填写FTP服务器地址
"用户名"填写匿名用户名"ftp"
"密码"为空
"端口"填写"21"

# **4、DHCP服务器配置**

**1):什么是DHCP：**
动态主机配置协议DHCP（Dynamic Host Configuration Protocol）是一种网络管理协议，用于集中对用户IP地址进行动态管理和配置。
DHCP于1993年10月成为标准协议，其前身是BOOTP协议。DHCP协议由RFC 2131定义，采用客户端/服务器通信模式，由客户端（DHCP Client）向服务器（DHCP Server）提出配置申请，DHCP Server为网络上的每个设备动态分配IP地址、子网掩码、默认网关地址，域名服务器（DNS）地址和其他相关配置参数，以便可以与其他IP网络通信。

| 名称        | IP            | 扮演角色   | 系统版本      |
| ----------- | ------------- | ---------- | ------------- |
| DHCP-Master | 192.168.2.137 | DHCP服务器 | Rocky-DVD-9.5 |
| DHCP-Client | DHCP获取      | DHCP客户端 | Rocky-DVD-9.5 |
|             |               |            |               |

## 1、安装DHCP

yum install -y dhcp*

**![image-20250526084315253](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526084315253.png)** 		dfghj

## 2、编写DHCP配置文件

配置文件 /etc/dhcp/dhcpd.conf

vim /etc/dhcp/dhcpd.conf

打开这个文件会发现里面啥也没有但是会为你指向一个DHCP配置文件模板路径	

我们将这个模板复制过来，在它的模板基础上进行修改就可以了

![image-20250526111952091](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526111952091.png)

复制DHCP配置文件的模板

 cp -p /usr/share/doc/dhcp-server/dhcpd.conf.example /etc/dhcp/dhcpd.conf

输入y覆盖现有的DHCP配置文件

![image-20250526112323046](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526112323046.png)

编辑DHCP配置文件

vim /etc/dhcp/dhcpd.conf

![image-20250526112409861](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526112409861.png)

第十四行的注释取消掉 因为默认是取消DNS更新的所以需要删除

第二十七行开始配置网段

声明要分配的网段IP，主机位为0 子网掩码为3个255

网段IP填写自己的网段，主机位填0 子网掩码默认3个255即可

地址范围按照需求划分（起始地址之间有空格分隔）默认网关按照需求和情况填写

![image-20250526142130889](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526142130889.png)

填写完之后后面多余的内容就可以删掉了



![image-20250526142330483](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526142330483.png)

## 3、验收环节

1、重启DHCP服务：

 systemctl restart dhcpd

2、将DHCP服务加入开机自启动：

systemctl enable dhcpd

3、关闭防火墙：

systemctl stop firewalld

4、关闭防火墙开机自启动：

 systemctl disable firewalld

5、临时关闭SELinux（重启后失效）：

setenforce 0

6、查看DHCP运行状态：

systemctl status dhcpd

![image-20250526142905590](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526142905590.png)





















































# 5、sabma服务器配置

**1): Samba简介：**
Samba是基于SMB协议（Server Message Block 信息服务块）的开源软件。一种Linux、UNIX系统上可用于共享文件和打印机等资源的协议，这种协议基于Client/Server型协议
Samba服务目录主要是用于Linux系统和Windows系统之间共享文件的最佳选择





| 名称        | IP             | 扮演角色  | 系统版本                     |
| ----------- | -------------- | --------- | ---------------------------- |
| SMB-Server  | 192.168.10.181 | SMB服务器 | Rocky-DVD-9.5                |
| SMB-Client1 | 192.168.10.161 | SMB客户端 | Windows Server 2022 Standard |



## 1、安装samba

yum install -y samba



![image-20250526155935397](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526155935397.png)

## 2、创建用户

创建一个系统用户，为后续的samba需要的用户准备：useradd  username 创建完成后为该账户创建用户密码 ：passwd username 

![image-20250526160322950](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526160322950.png)

## 3、创建共享文件夹

mkdir /smbshare

然后在文件夹内创建一个txt文件，再随便往里面填写点内容

![image-20250526163628081](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526163628081.png)

![image-20250526160621797](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526160621797.png)

## 4、编写Samba配置文件

备份配置文件：

cp -p /etc/samba/smb.conf   /etc/samba/smb.conf.bak

编写配置文件

vim /etc/samba/smb.conf

然后在配置文件末尾添加以下内容：

 [test1]
        comment= '114514'
        path=/smbshare
        publicon= no
        writable= yes
        guest ok = yes



![image-20250526161142423](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526161142423.png)

## 5、验收环节

1、重启SMB:

systemctl restart smb

2、关闭防火墙：

systemctl stop firewalld

3、关闭SELinux：

setenforce 0

Windows客户端测试连通性：

![image-20250526161844725](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526161844725.png)

这时候访问会显示“无法访问”

![image-20250526162132009](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526162132009.png)

这是因为还没有向samba中添加本地用户

然后再次输入两次密码即可

 smbpasswd -a test1



![image-20250526162320086](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526162320086.png)

![image-20250526164009223](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526164009223.png)



![image-20250526164131485](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526164131485.png)



在Ubuntu"文件"-"其他位置"-"连接到服务器中输入"`smb://192.168.10.181`

选择"匿名"，然后点击"连接"

# 6、Apache+SSL服务器配置

Apache服务需要DNS支持，所以要在做Apache事先做好DNS。

配置完成DNS后再开始进行下面步骤



![image-20250526174507855](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250526174507855.png)

##  1、安装Apache

yum install -y httpd

![image-20250527082200174](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527082200174.png)

## 2、编辑Apache配置文件

Apache配置文件在于/etc/httpd/conf/httpd.conf

![image-20250527082634308](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527082634308.png)

### 124行

124行存放的是站点根目录，网站上的文件都存放在于该目录下

修改成自己创建的/www/web （没创建记得创建）

### 129行

129行也是改成/www/web

### 136行

136行也是改成/www/web

![image-20250527083059121](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527083059121.png)

下一部分

![image-20250527083259311](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527083259311.png)

### 169行

把169行改成最后改成web.html，这里也可以不改但是不改的话，创建网站根文件的时候也要写成index.html不改的话就还是按照web.html。

![image-20250527083554628](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527083554628.png)

## 3、创建站点根目录

由于在httpd.conf里面指定的都是/www/web目录，所以要添加-p选项。

mkdir -p /www/web

## 4、创建站点根文件

如果你之前在169行没有修改的情况下，那就按照index.html来创建。

如果修改了之后那就把 index.html 替换成 web.html ,这里我改过了名字，那我就创建

web.html 了

touch /www/web/web.html

![image-20250527084238519](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527084238519.png)



## 5、编写站点根文件

创建好站点根文件后，里面没有任何的内容，这时候去里面随便写点内容

vim /www/web/web.html

## 6、SSL证书

### 6.1 安装SSL证书

yum install mod_ssl

![image-20250527084636084](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527084636084.png)

### 6.2 生成SSL证书

一直回车即可

openssl req -newkey rsa:2048 -nodes -keyout server.key -x509 -days 365 -out serv
er.crt

![image-20250527085055768](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527085055768.png)

### 6.3 复制SSL证书及密钥

把这两份SSL证书及密钥复制到/etc/pki/tls/certs和/etc/pki/tls/private/目录内。

 cp server.crt /etc/pki/tls/certs/server.crt

cp server.key /etc/pki/tls/private/server.key

### 6.4 修改SSL配置文件

 vim /etc/httpd/conf.d/ssl.conf

![image-20250527085728643](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527085728643.png)

把“local”替换为“server”即可

![image-20250527085849666](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527085849666.png)

## 7、重启服务

重启Apache：

systemctl restart httpd

重启SSL：

 service sshd restart 或者 systemctl restart sshd

再次查看是否正确启动了：

 systemctl status sshd

![image-20250527090841900](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527090841900.png)

在 Linux 系统中， 和 本质上都是用于重启 SSH 服务，但它们的底层实现、适用场景和功能特性存在明显差异，具体区别如下：`service sshd restart``systemctl restart sshd.service`

### **一、命令本质与系统兼容性** 

| **维度**     | **`service sshd restart`**                                  | **`systemctl restart sshd.service`**                         |
| ------------ | ----------------------------------------------------------- | ------------------------------------------------------------ |
| **底层实现** | 基于 SysV init（传统初始化系统）的脚本命令                  | 基于 systemd（现代系统管理器）的原生命令                     |
| **适用系统** | 适用于 CentOS 6、Debian 7 及更早版本等使用 SysV init 的系统 | 适用于 CentOS 7+/RHEL 7+、Ubuntu 16.04+、Debian 8 + 等使用 systemd 的系统 |
| **命令本质** | 本质是调用脚本并传递参数`/etc/init.d/sshd``restart`         | 直接作 systemd 管理器，控制服务单元（.service 文件）         |

关闭防火墙及SELinux

关闭防火墙：

systemctl stop firewalld.service

关闭SElinux:

setenforce 0

## 9、验证

### 9.1 IP地址验证

打开浏览器输入虚拟机IP地址

192.168.20.162

![image-20250527091430151](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527091430151.png)

### 9.2 域名验证

输入域名后，可以看到并不是我们的HTML

![image-20250527091652323](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527091652323.png)

这是因为宿主机的DNS是233.6.6.6，电视跑到阿里的DNS服务器上找"www.ninework.com"这个网站去了。

​	找到了就会显示输入的域名，没有找到就会出现无法访问此网站。

这里建议，跟虚拟机有数据流通的网卡外全部都禁用掉，我用的是VM NAT模式 ，那我就把外网的网卡禁掉，打开VM的虚拟网卡 。  如果用的是桥接模式，那么就把VM的虚拟网卡禁了，只打开外网的网卡。

![image-20250527094250035](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527094250035.png)

把DNS指向虚拟机后就可以访问到我们的ninework.com站点了。

![image-20250527093624993](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527093624993.png)

### 9.3 原理

### 【10.1】DNS指向公网DNS服务器

DNS=223.6.6.6的时候，电脑会向阿里DNS服务器搜寻是否有"www.ninework.com"这个网站，事实上是有这个网站的，只不过不是我们刚才搭建的网站
![www.ninework.com-Erection-address.png](https://s1.imagehub.cc/images/2023/09/05/www.ninework.com-Erection-address.png)
经过查询后，这个站点是架设在美国亚马逊的服务器站点，并不是我们想要的内网站点
~~我们局域网内的网站也不可能出现在阿里的DNS服务器里，因为虚拟机并没有公网IP，公网访问不到我们局域网里的网站~~

# 7、Mail服务器搭建

## 开始之前先进行准备工作

配置DNS正反向配置文件时添加mail

## 1、修改主机名称

可以通过 mntui 或者直接修改	或者通过修改 /etc/hostname 文件来更改名称。 重启后生效。

![image-20250527095141899](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527095141899.png)

## 2、关闭防火墙

systemctl stop firewalld.service

## 3、配置DNS：

### 3.1 安装DNS：

 yum install bind*

![image-20250527095746320](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527095746320.png)

### 3.2 配置named.conf：

cd到etc:

cd /etc/

编辑named.conf

vim named.conf

![image-20250527100230641](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527100230641.png)

把第11行花括号内修改成"any" 把第19行的花括号内修改成“any”。

![image-20250527100419232](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527100419232.png)

### 3.3  配置named.rfc1912.zones文件

把16行到34行删掉：

![image-20250527100634507](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527100634507.png)

修改完成后

![image-20250527100732968](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527100732968.png)

第18行修改为域名，24行中的“0”修改为反向的网络位IPV4地址：

然后第20行和第26行分别修改为正向解析文件和反向解析文件。

![image-20250527101210848](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527101210848.png)

### 3.4 编写 a 文件以及 fx 文件：

首先cd到/var/named内：

然后复制两份文件：

cp -p named.localhost a

cp -p named.loopback fx

![image-20250527101421346](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527101421346.png)![image-20250527101421419](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527101421419.png)

编辑a文件：

vim a

修改前

![image-20250527101519926](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527101519926.png)

修改后

![image-20250527101707193](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527101707193.png)

然后修改 fx 文件 ：

vim fx

修改前

![image-20250527101812309](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527101812309.png)

修改后

![image-20250527102032673](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527102032673.png)

### 3.5 配置虚拟机DNS

虚拟机网卡的DNS必须要指向自己才可以，所以要修改虚拟机网卡DNS：

vim /etc/NetworkManager/system-connections/ens160.nmconnection

![image-20250527102456545](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527102456545.png)

### 3.6 重启网卡

systemctl restart NetworkManager

### 3.7 重启 Bind

systemctl restart named

### 3.8 验证 

输入nslookup验证：

![image-20250527102806792](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527102806792.png)

输入域名和IP后，不报错或者没有出现别的IP地址就是配置好了。

## 4、配置 postfix

### 4.1、安装 postfix

yum -y install postfix

![image-20250527103042584](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527103042584.png)

### 4.2、编辑 postfix 配置文件

postfix配置文件存放于vim /etc/postfix/main.cf 中

### 94行

打开main.cf文件，找到94行内容，取消注释

然后在后面跟上mail的域名

![image-20250527103515754](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527103515754.png)

### 102行

找到102行，取消注释 后边跟上域名

![image-20250527103830499](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527103830499.png)

### 118行

找到118行，取消注释

![image-20250527103929600](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527103929600.png)

### 132行 和135行

找到132行，取消注释。 找到135行，在首行添加注释。

![image-20250527104110230](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527104110230.png)

### 183 行 和 184行

找到183行，在首行加注释。找到184行，取消注释。

### ![image-20250527104147800](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527104147800.png)

## 5、配置dovecot

### 5.1、安装dovecot

yum -y  install dovecot

### 5.2、编辑dovecot配置文件

dovecot配置文件存放于 /etc/dovecot/dovecot.conf 中

### 5.3、24行

找到第24行，取消首行注释，并在25行添加一行。

 disable_plaintext_auth = no

允许用户使用明文进行密码验证

![image-20250527105154627](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527105154627.png)

### 5.4、48行

找到48行的login_trusted_networks ，根据需要来选择取消注释并配置允许登录的网段地址。

我们可以在这里限制只有来自于某个网段的用户才能使用电子邮件系统。如果想允许所有人都能使用，那就不用修改本参数。

我这里就限制只有20段用户才能使用mail

![image-20250527105518101](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527105518101.png)

### 6、配置10-ssl.conf

这步按需配置，如果需要SSL证书，那直接进行下一步即可，如果这步默认的话，Outlook无法接受邮件

10-SSL.conf配置文件存在于/etc/dovecot/conf.d/10-ssl.conf中

第八行修改成 no 

![image-20250527105945240](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527105945240.png)

### 7、配置10-mail.conf

在子配置文件/etc/dovecot/conf.d/10-mail.conf中，找到mail_location并且把注释去掉即可。

 vim /etc/dovecot/conf.d/10-mail.conf

![image-20250527110253032](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527110253032.png)

## 6、重启工作

重启bind

systemctl restart named

重启postfix

systemctl restart postfix

重启dovecot

systemctl restart dovecot

## 7、创建mail登录用户

### 7.1、创建用户以及设置密码

创建space7用户

useradd space7

为space7用户设置登录密码

 passwd space7

### 7.2、创建保存邮件目录

先切换到刚才创建的space7用户

su - space7

在space7家目录中创建用于保存邮件的目录

mkdir -p mail/.imap/INBOX

## 8、验收环节

### 8.1、安装s-nail

yum -y install s-nail

### 8.2 、使用root账户给其他用户发送邮件

先切换到root用户

su - root

echo "I am ikk " | mail -s "Hello" space7@ninework.com

然后切换到space7用户

su - space7

输入mail 查看收件箱

![image-20250527112242782](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527112242782.png)



# 8、keepAlive配置

## 配置环境

| 名称            | IP             | 优先级 |
| --------------- | -------------- | ------ |
| server1         | 192.168.20.130 | 100    |
| server2         | 192.168.20.131 | 99     |
| KeepAlive虚拟IP | 192.168.20.140 | /      |

## 1、KeepAlive安装

 yum -y install keepalived

![image-20250527145951493](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527145951493.png)

## 2、配置KeepAlive安装

KeepAlive配置文件路径：/etc/keepalived/keepalived.conf

vim /etc/keepalived/keepalived.conf

配置serve1

### 12行												

” router_id “后修改为自己需要的名称

### 14行

 vrrp_strict 将这条内容删除掉

### 20行

 “interface” 后边写上网卡名称：



![image-20250527151314652](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527151314652.png)

不知道网卡名称通过  ifconfig 查看

![image-20250527151429977](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527151429977.png)

### 29行

删除现有的IP地址，输入KeepAlive虚拟IP地址

### 19行

  state MASTER 是主KeepAlive，默认就行，不用修改

### 22行

 priority 是优先权，优先权值越大，优先级越高默认就是100。



## 3、配置server2

### 12行

修改名称为 server2

### 14行

删除“ vrrp_strict”

### 19行

更改服务器扮演角色为BACKUP

### 20行

修改网卡名称

### 22行

修改服务器优先级，server1优先级是100，server2优先级只要比100低就行

### 29行

![image-20250527152903390](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527152903390.png)

重启KeepAlive服务

systemctl restart keepalived.service

## 4、创建HTTP站点

### 1、创建web主页文件

​	创建http站点路径

mkdir -p /www/web

创建web主页文件

touch /www/web/web.html

![image-20250527153535754](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527153535754.png)



### 2、修改httpd配置文件

vim /etc/httpd/conf/httpd.conf 

### 124行

![image-20250527153919065](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527153919065.png)

### 129行

![image-20250527154000619](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527154000619.png)

### 136行

![image-20250527154018181](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527154018181.png)

### 169行

![image-20250527154046233](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527154046233.png)

重启httpd服务

 systemctl restart httpd

server2配置http

server2跟server1一样的配置，只不过为了区分server1和server2，web.html的内容是不一样的内容，但在实际生活中server2其实就是用来接替server1的http服务的，所以它们的内容是一致的。

![image-20250527154755924](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527154755924.png)

## 5、验收环节

关闭防火墙

systemctl stop firewalld.service

关闭SELinux

setenforce 0

浏览器验证

![image-20250527160817799](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527160817799.png)

模拟宕机

通过停止server1的keepalive服务，模拟server1宕机

 systemctl stop keepalived.service

查看keepalive是不是死掉了

systemctl status keepalived.service

![image-20250527160217672](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527160217672.png)

这时候刷新网页，可以看到server2已经接替了http服务

![image-20250527160934469](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527160934469.png)

重新接管

然后server1 keepalive服务重新启动

systemctl restart keepalived.service

keepalive活动

![image-20250527161043379](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527161043379.png)

server1重新接管http服务

![image-20250527161202284](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250527161202284.png)

# 9、chrony时间同步配置

| 名称         | IP             | 扮演角色      | 镜像版本            |
| ------------ | -------------- | ------------- | ------------------- |
| Rocky-server | 192.168.20.130 | chrony server | Rocky Linux DVD 9.5 |
| Rocky-client | 192.168.20.131 | chrony client | Rocky Linux DVD 9.5 |

## 1、server主机安装chrony

 yum -y install chrony

## 2、修改server主机中的chrony配置文件

vim /etc/chrony.conf

### 第三行

修改前：s

![image-20250530085236894](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530085236894.png)

把第3行的内容注释掉，该行用于填写外网服务器网址。

![image-20250530085331366](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530085331366.png)

### 第4行

格式：server IP iburst  修改前：

![image-20250530085623708](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530085623708.png)

添加 server IP iburst 填写本地chrony时间同步服务器

![image-20250530085710772](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530085710772.png)

### 第26行

修改前：

![image-20250530090006186](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530090006186.png)

取消allow前面的注释：

![image-20250530090133343](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530090133343.png)

### 第29行

修改前：

![image-20250530090353672](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530090353672.png)

取消local stratum 10前面的注释。否则NTP synchronized: 为no
取消掉后变为NTP synchronized:yes。

![image-20250530090540398](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530090540398.png)

## 3、重启chrony服务

systemctl restart chronyd

并且设置开机自启动

systemctl enable chronyd

![image-20250530090719909](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530090719909.png)

## 4、检查配置文件

 netstat -antulp|grep chronyd

 ss -antulp|grep chronyd

![image-20250530091205990](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530091205990.png)

## 5、client下载chrony

 yum install -y chrony

## 6、client修改chrony配置文件

 vim /etc/chrony.conf

### 第3行

![image-20250530091502558](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530091502558.png)

注释到第三行，该行为填写网络时间服务器网址

![image-20250530091629350](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530091629350.png)

### 第4行

![image-20250530091704075](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530091704075.png)

在第4行填写本地chrony服务器地址，指向chrony server

![image-20250530091807499](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530091807499.png)

## 7、重启chrony服务

重启chrony服务：

systemctl restart chronyd

将chrony加入开机自启：

systemctl enable chronyd

## 8、关闭防火墙以及SELinux

systemctl stop firewalld

setenforce 0

## 9、检查状态

timedatectl

![image-20250530092203003](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530092203003.png)

10、查看时间源信息

server：

chronyc sources -v

![image-20250530092403646](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530092403646.png)

client：

chronyc sources -v

![image-20250530092431008](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530092431008.png)

11、验收环节

查看当前server时间：

date

![image-20250530092557802](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530092557802.png)

查看当前client时间：

![image-20250530092626235](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530092626235.png)

修改server时间为8:00 

date -s "2025-5-30 08:00:00"

![image-20250530092826467](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530092826467.png)

等待客户端自动同步

同步时间较为缓慢，我这里也等了很长时间

![image-20250530093723611](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530093723611.png)

同步成功之后，System clock synchronized会变成“yes”。

![image-20250530094249187](C:\Users\xyt\AppData\Roaming\Typora\typora-user-images\image-20250530094249187.png)


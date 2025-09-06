# 配置本地yum源

# rocky

## 1、传送DVD镜像到本地目录下

![image-20250526150753135](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526150753135.png)

在本机的文件路径下查看有没有镜像‘

ls /opt/

![image-20250526151459859](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526151459859.png)

## 2、设置镜像开机自动挂载

vim /etc/fstab

修改前

![image-20250526154140825](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526154140825.png)

在最后一行添加一段内容 前边写镜像的绝对路径，然后写要挂载到的路径，然后再写上iso9660 loop,defaults（按一下Tab建）然后再输入 0 0 

![image-20250526154455273](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526154455273.png)

可以查看一下/mnt/有没有挂载成功	ls /mnt/![image-20250526154733177](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526154733177.png)

进入到yum.repos.d的目录下然后去到`，将除了`rocky.repo的所有文件全部删除或者是移动到其他目录去
在/etc/yum.repos.d下创建了一个bak文件夹，然后把rocky-开头的文件全部移动到了bak文件夹里我这里将文件全都删除了。

cd /etc/yum.repos.d/

查看目录下的文件

ls

![image-20250526151645486](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526151645486.png)

将rocky-开头的文件都删除掉 当文件就剩下这一个的时候就可以编写了

rm -rf rocky-*

![image-20250526151813610](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526151813610.png)

## 3、编辑rocky.repo文件

打开rocky.repo配置文件

 vim rocky.repo

![image-20250526151949002](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526151949002.png)

把除了第一段以外的全部删除了

![image-20250526152025051](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526152025051.png)

然后再把第一段复制一下，再去复制到下一行。

![image-20250526152434806](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526152434806.png)

第12行最后的一段后缀需要改成跟第11行样的 ，把第13行注释掉，在14行下面再添加一行

baseurl=file:///mnt/BaseOS

把第15行改成 gpgcheck=0前面添加一行之后就变成16行了 然后按照这样的格式 把下一段开头的内容改成AppStream然后后缀也改成这个就可以了

![image-20250526153610901](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526153610901.png)

[baseos]
name=Rocky Linux $releasever - BaseOS
#mirrorlist=https://mirrors.rockylinux.org/mirrorlist?arch=$basearch&repo=BaseOS-$releasever$rltype
#baseurl=http://dl.rockylinux.org/$contentdir/$releasever/BaseOS/$basearch/os/
baseurl=file:///mnt/BaseOS
gpgcheck=0
enabled=1
countme=1
metadata_expire=6h
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-Rocky-9

[AppStream]
name=Rocky Linux $releasever - AppStream
#mirrorlist=https://mirrors.rockylinux.org/mirrorlist?arch=$basearch&repo=BaseOS-$releasever$rltype
#baseurl=http://dl.rockylinux.org/$contentdir/$releasever/BaseOS/$basearch/os/
baseurl=file:///mnt/AppStream
gpgcheck=0
enabled=1
countme=1
metadata_expire=6h
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-Rocky-9

清除yum缓存并生成新缓存：

yum makecache		yum clean all

![image-20250526155147952](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250526155147952.png)

# 配置本地yum源

# CentOS

## 1、传送DVD镜像到本地目录下

查看本地目录下镜像是否传送过来了。

![image-20250530100818041](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530100818041.png)

## 2、设置镜像开机自动挂载

vim /etc/fstab

![image-20250530101256682](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530101256682.png)

在最后一行添加内容

填写镜像的绝对路径，要挂载到的目录 然后在后续添加iso9660 

loop,defaults 按下Tab建输入 0 0 

![image-20250530101622545](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530101622545.png)

看看挂载有没有成功

ls /mnt

![image-20250530102052542](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530102052542.png)

## 3、编辑centos.repo配置文件

cd 到 yum.repos.d/的目录下

cd /etc/yum.repos.d/

查看一下该目录之中的内容

ls 

![image-20250530102238921](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530102238921.png)

将多余的文件删除掉 

rm -rf centos-*![image-20250530102337090](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530102337090.png)

编辑centos.repo配置文件

vim centos.repo

修改前

![image-20250530102455993](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530102455993.png)

将除了第一段以外的内容全部删除掉

![image-20250530103235594](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530103235594.png)



将第3行的内容注释掉，在第4行的下面添加一项内容为第5行“baseurl=file:///mnt/BaseOS” 然后把第6行的gpgcheck修改为=0 ，在复制一行到下面 但是第一行标题的内容和跟标题一样的后缀都改为AppStream 第13行的后缀和第16行的后缀都需要修改。

![image-20250530104607524](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530104607524.png)

修改完成

![image-20250530104625597](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530104625597.png)

[baseos]
name=CentOS Stream $releasever - BaseOS
#metalink=https://mirrors.centos.org/metalink?repo=centos-baseos-$stream&arch=$basearch&protocol=https,http
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-SHA256
baseurl=file:///mnt/BaseOS
gpgcheck=0
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=1

[AppStream]
name=CentOS Stream $releasever - AppStream
#metalink=https://mirrors.centos.org/metalink?repo=centos-baseos-$stream&arch=$basearch&protocol    =https,http
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-SHA256
baseurl=file:///mnt/AppStream
gpgcheck=0
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=1

重建yum缓存并清除yum缓存：

 yum makecache

yum clean all

![image-20250530104932258](https://cdn.jsdelivr.net/gh/xyt250/image/img/image-20250530104932258.png)

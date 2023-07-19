# qemu配置和busybox根文件系统制作

## 虚机内文件情况

169.254.248.141的虚机文件情况

### filesystem：

rootfs.cpio.gz       未知

rootfs.ext3            arm32的busybox1.36.1指令集   format=ext3

test-en25.ext3      EN25QH64固件的指令集           format=ext3

initramfs                arm64的busybox1.34.1指令集   format=newc

test_gd25.ext4      GD25Q64C固件的指令集

gd25test.ext3	    GD25Q64C固件的指令集，可运行

gd25run.ext3		GD25Q64C固件的指令集，将mtdblock7内的文件移动到了/usr/

### linux-core：

linux-4.9.129       		 arm32编译-vexpress_defconfig

linux-4.20.9        		  arm32编译-vexpress

linux-4.4.282     		   arm32编译-vexpress_defconfig

Image                 		   linux4.19.183   arm64位编译-virt

palmetto-image  	    来自openBMC官网的内核镜像

ipczImage		 		   TL-IPC43AN摄像头固件的内核，用于测试dd命令

gd25image        		  dd裁剪的GD25摄像头内核

en25image				  dd裁剪的EN25摄像头内核

### run qemu：

qemu-64-run.sh         initramfs和Image的arm64启动脚本

cpio-start.sh                rootfs.cpio.gz和linux-4.20.9的arm32启动脚本

en25run.sh                 EN25QH64固件的启动脚本——可在linux-4.9.129上启动，已经可以进入界面

gd25run.sh				 GD25Q64C固件的启动脚本——可运行

ltest.sh						用于测试qemu相关命令

rootfs.sh					rootfs.ext3的启动脚本——vexpress-a9，busybox文件系统

TL-IPC43AN-run.sh	测试TL-IPC43AN裁剪内核的启动脚本



network-setup.sh		配置网络脚本



## 基本信息

静态IP

虚拟网关同前一台机子

##### VMware外部配置

NAT模式

子网IP：169.254.248.0

DHCP设置：起始段169.254.248.128，结束段169.254.248.254

子网掩码：255.255.255.0

记得给主机也配上VMnet8的适配（有个选项可以勾上），不然主机ping不到虚拟机

##### 虚拟机内地址：

IPv4：169.254.248.141

netmask：255.255.255.0

gateway：169.254.248.2



已经换源为中科大源，未作ubuntu源的备份，[参考](https://midoq.github.io/2022/05/30/Ubuntu20-04%E6%9B%B4%E6%8D%A2%E5%9B%BD%E5%86%85%E9%95%9C%E5%83%8F%E6%BA%90/)https://midoq.github.io/2022/05/30/Ubuntu20-04%E6%9B%B4%E6%8D%A2%E5%9B%BD%E5%86%85%E9%95%9C%E5%83%8F%E6%BA%90/



binwalk已安装

ubuntu内自带Python 3.8.10环境，要变化可参考[这个](https://blog.51cto.com/u_14900374/2523671)

已编译QEMU

已下载Debain镜像



## 用户权限管理

用户：

riria

密码：

riria



用户：

root

密码（已通过sudo passwd修改密码）：

riria







## VMware远程传输已配置

```
 apt install open-vm-tools

 apt install open-vm-tools-desktop
```

[参考](https://www.jianshu.com/p/687acbfd21a5)https://www.jianshu.com/p/687acbfd21a5

## VScode SSH远程调试已开启

apt-get install openssh-server -y

service ssh start

ps -e|grep ssh 							  查看ssh启动没，返回sshd就ok了

systemctl enable ssh					开机启动

ufw disable									关防火墙



###### 再改SSH配置

vim /etc/ssh/sshd_config

修改以下配置：

```
33 #LoginGraceTime 2m
34 #PermitRootLogin prohibit-password
35 #StrictModes yes
36 #MaxAuthTries 6
37 #MaxSessions 10
```

修改为：

```bash
 LoginGraceTime 2m
 PermitRootLogin yes
 StrictModes yes
 #MaxAuthTries 6
 #MaxSessions 10
```

service ssh restart

然后在VScode的.config中把用户名改成root，连接就好



## QEMU配置

版本：7.1.0

过高版本会导致ERROR: meson setup failed错误，最好的方法是降低版本



关闭qemu的方法为，Ctrl-A, X.

注意：是先按下Ctrl-A,松开后按下X

Ctrl-A，C 是在仿真中唤出QEMU控制台

------

### 安装qemu：

```
wget https://download.qemu.org/qemu-2.11.1.tar.xz

tar xvJf qemu-2.11.1.tar.xz

cd qemu-2.11.1

./configure

make
```

一般需要安装以下安装包：

```
apt-get install zlib1g zlib1g-dev

apt install libglib2.0-dev

apt-get install libpixman-1-dev
```

------

#### 报错

##### ERROR: Cannot find Ninja 

[参考该链接](https://www.cnblogs.com/from-zero/p/14327440.html)，使用以下命令安装Ninja：

```
apt install re2c
git clone git://github.com/ninja-build/ninja.git && cd ninja
./configure.py --bootstrap
sudo cp ninja /usr/bin/

ninja --version  # 查看安装版本
```

##### ERROR: meson setup failed

```
apt-get update libpixman-1-dev
```

如果弄完这一步还报错，降低qemu版本。

------

**`qemu` 支持 `virtfs`**

```cpp
./configure --enable-kvm --enable-virtfs --prefix=/opt/software/toolchain/qemu
```



安好后输入以下命令：

```
make install | tee make-install.log
```

验证安装（输入后按两次tab键）：

```
qemu-
```

这样就安好了大部分qemu组件，如果要比较全的还得安下面这些



------

### edu设备加入make编译：

在arm64 qemu运行，在启动命令行中加上-device edu


注：编译qemu，需要打开CONFIG_EDU=y，位于default-configs/aarch64-softmmu.mak 

```
default-configs/aarch64-softmmu.mak 



#Default configuration for aarch64-softmmu

#We support all the 32 bit boards so need all their config

include arm-softmmu.mak

CONFIG_XLNX_ZYNQMP_ARM=y
CONFIG_XLNX_VERSAL=y
CONFIG_SBSA_REF=y
CONFIG_EDU=y
```



#### hw/misc/Kconfig

添加如下内容

```
config EDU
    bool
    default y if TEST_DEVICES
    depends on PCI && MSI_NONBROKEN
```

#### hw/misc/meson.build

添加如下内容

```
softmmu_ss.add(when: 'CONFIG_EDU', if_true: files('edu.c'))
```

qemu启动时添加参数-device edu

在虚拟机启动后，lspci -n查看id ‘1234:11e8’即为仿真的设备

[参考](https://blog.csdn.net/qq_42138566/article/details/130953688)https://blog.csdn.net/qq_42138566/article/details/130953688

[参考](https://zhuanlan.zhihu.com/p/350947593)https://zhuanlan.zhihu.com/p/350947593

------

### make之后qemu如何删除：

qemu卸载根据安装方式的不同也会有响应的卸载方式：（1）源码编译安装需要手动卸载；（2）ubutnu pakage安装需要命令卸载


(1) 源码编译安装的qemu需要手动卸载：

```
可执行文件默认放在/usr/local/bin

库文件默认存放在/usr/local/libexec

配置文件默认存放在/usr/local/etc

共享文件默认存放在/usr/local/share
```

卸载源码只需将上面四个目录中相关文件或者目录删除

(2) pakage安装方式需命令卸载

删除包和相关依赖

```
sudo apt-get remove --auto-remove qemu-system-x86
```

删除配置文件和相关的数据文件

```
sudo apt-get purge --auto-remove qemu-system-x86
```

[参考](https://blog.csdn.net/Oliverlyn/article/details/56667222)https://blog.csdn.net/Oliverlyn/article/details/56667222



## QEMU内使用debain镜像（被毙掉了，但可选）

来源：https://people.debian.org/~gio/dqib/

这台机子使用三种镜像：

mips

mipsel

arm64

所有qemu启动命令都在readme.txt里边



## 内核编译（如果要自己弄内核那就做这个，最终是这个方案）

#### 交叉编译ARM架构内核

下载（root）交叉编译器（arm32位）

```
apt-get install gcc-arm-linux-gnueabi  
```

验证安装结果

```
dpkg -l gcc-arm-linux-gnueabi
```

#### 准备阶段

修改Makefile，把在300多行的ARCH和CROSS_COMPILE配置成arm编译器

```
ARCH ?= arm#$(SUBARCH)
CROSS_COMPILE = arm-linux-gnueabi-
```



在linux解压后的安装目录内使用以下命令把vexpress_defconfig作为配置文件保存为.config文件（以vexpress开发板的默认配置）下面在编译内核时就根据这个config中的配置进行编译。在下一条指令可以调整裁剪的范围以及相应功能

```
make vexpress_defconfig  #建议别选，因为没有PCI总线导致 -device 命令无效

make defconfig  #这个可以，但建议选对应指令集的开发板进行编译  

若此步报错 bison: not found：
sudo apt install bison flex
//一般缺少bison的时候也会缺少flex，所以也一起安装了吧。
error : openssl/bio.h :No such file or folder
apt install libssl-dev

make menuconfig

若此步报错  Unable to find the ncurses libraries
apt-get install ncurses-dev
```

#### 清空之前的编译信息

make mrproper



#### 内核裁剪配置（可选）

**在配置菜单中，启用内核debug，关闭地址随机化，不然断点处无法停止。**（找不到也可以不找）

```
Kernel hacking  ---> 
    [*] Kernel debugging
    Compile-time checks and compiler options  --->
        [*] Compile the kernel with debug info
        [*]   Provide GDB scripts for kernel debuggin


Processor type and features ---->
    [] Randomize the address of the kernel image (KASLR)
```

##### 配置E1000网卡（保证后边qemu虚拟机能连到主机传东西，只在linux5.0以上版本有，但是linux5.0版本以上启动dts需要用其他命令）

```
Device Drivers  --->
	[*] Network device support  --->
		[*]   Ethernet driver support  --->
			[*]   Intel devices
			<*>     Intel(R) PRO/1000 Gigabit Ethernet support
```

make config     要按回车巨久



#### 开始编译arm架构内核

```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j 8
```

所有默认开发板配置在`arch/arm/configs/`下边，可在defconfig上修改





## 用busybox制作根目录文件系统，参考[知乎](https://zhuanlan.zhihu.com/p/340362172)--[博客](https://www.renshengjihe.cn/1fe7a0c955f0),但是[这个](https://www.zhaixue.cc/qemu/qemu-build_busybox_rootfs.html)才靠谱

编译完成后的内核仅仅是内核，没有引导的文件系统可供操作。所以这个时候启动会有 *end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)* 的错误。



那么我们需要用busybox做一个根目录文件系统

##### 配置Makefile

```
gedit Makefile +191
```

和编译kernel的时候一样，ARCH和CROSS_COMPILE配置成arm编译器

```
ARCH ?= arm
CROSS_COMPILE = arm-linux-gnueabi-
```

和kernel一样，输入make menuconfig进行一些配置
(1) 设置一下编辑器环境，方便操作:
Settings —-> [*] vi-style line editing commands (New)

(2) 设置一下安装路径，避免错误安装在根目录
Settings —-> Destination path for ‘make install’

进入以后，输入自己的rootfs的路径(我这里的路径是: /home/nfs)



设置完毕，设置编译安装:

```
make install -j 2
没有指定目录的话，就默认busybox目录下的_install
make install
```

无需再输入单独的编译命令，没有编译的话默认是会编译好了再安装的
注意！！！！！这里不要以root用户编译也不要加sudo编译，防止安装到根目录里，破坏Ubuntu
我们通常设置的安装模式安装到的/home/tftpboot已经给了最高权限，是肯定可以安装的，如果错误设置不安装到这个地方就会因为没有权限导致安装不上，从而完全避免了安装到根目录导致Ubuntu崩溃的问题。
如下图，就是错误的将安装目录设置为根目录的时候，因为没有权限没有执行安装过程，从而避免了因为错误安装导致的系统崩溃。

正常安装以后:

基本系统的功能是有了，接下来的功能便是安装动态链接库、设置设备节点、设置初始化进程、设置文件系统、设置环境变量



##### 安装动态链接库

通过apt安装的arm编译器，动态链接库路径通常为:/usr/arm-linux-gnueabi/lib，复制到根文件系统的lib下

```
cd /home/nfs

mkdir lib

cd /usr/arm-linux-gnueabi/lib

cp *.so* /home/nfs/lib -d
```

若在运行linuxrc时失败，库的版本不兼容等，解决方法是编译busybox时，配置静态编译：

[*] Build static binary (no shared libs)
这样linuxrc运行时就不依赖动态库了，就可以正常运行了。

##### 创建设备结点

```
cd /home/nfs

mkdir dev

cd dev

sudo mknod -m 666 tty1 c 4 1

sudo mknod -m 666 tty2 c 4 2

sudo mknod -m 666 tty3 c 4 3

sudo mknod -m 666 tty4 c 4 4

sudo mknod -m 666 console c 5 1

sudo mknod -m 666 null c 1 3
```

##### 设置初始化进程/etc/rcS

```
cd /home/nfs

mkdir -p etc/init.d

cd etc/init.d

touch rcS

chmod 777 rcS

gedit rcS
```

将以下内容加入rcS中，这里会设置系统内的相关参数以及可用命令

```
#!/bin/sh
PATH=/bin:/sbin:/usr/bin:/usr/sbin 
export LD_LIBRARY_PATH=/lib:/usr/lib
/bin/mount -n -t ramfs ramfs /var
/bin/mount -n -t ramfs ramfs /tmp
/bin/mount -n -t sysfs none /sys
/bin/mount -n -t ramfs none /dev
/bin/mkdir /var/tmp
/bin/mkdir /var/modules
/bin/mkdir /var/run
/bin/mkdir /var/log
/bin/mkdir -p /dev/pts
/bin/mkdir -p /dev/shm
/sbin/mdev -s
/bin/mount -a
/sbin/ifconfig lo 127.0.0.1
/sbin/ifconfig eth0 192.168.4.10
echo "-----------------------------------"
echo "*****welcome to vexpress board*****"
echo "-----------------------------------"
```

##### 设置文件系统/etc/fstab

```
cd /home/nfs/etc

touch fstab

gedit fstab
```

输入以下内容：

```
proc    /proc           proc    defaults        0       0
none    /dev/pts        devpts  mode=0622       0       0
mdev    /dev            ramfs   defaults        0       0
sysfs   /sys            sysfs   defaults        0       0
tmpfs   /dev/shm        tmpfs   defaults        0       0
tmpfs   /dev            tmpfs   defaults        0       0
tmpfs   /mnt            tmpfs   defaults        0       0
var     /dev            tmpfs   defaults        0       0
ramfs   /dev            ramfs   defaults        0       0
```



##### 设置初始化脚本/etc/inittab

```
cd /home/nfs/etc

touch inittab

gedit inittab
```

输入以下内容:

```
::sysinit:/etc/init.d/rcS
::askfirst:-/bin/sh
::ctrlaltdel:/bin/umount -a -r
```

##### 设置环境变量/etc/profile

```
cd /home/nfs/etc

touch profile

gedit profile
```

输入以下内容:

```
USER="root"
LOGNAME=$USER
export HOSTNAME=`cat /etc/sysconfig/HOSTNAME`
export USER=root
export HOME=/root
export PS1="[$USER@$HOSTNAME \W]\# "
PATH=/bin:/sbin:/usr/bin:/usr/sbin
LD_LIBRARY_PATH=/lib:/usr/lib:$LD_LIBRARY_PATH
export PATH LD_LIBRARY_PATH
5.2.7 增加主机名/etc/sysconfig/HOSTNAME
```



```
cd /home/nfs/etc

mkdir sysconfig

cd sysconfig

touch HOSTNAME
```

输入以下内容:

```
vexpress
```

##### 创建剩下的文件夹

```
cd /home/nfs

mkdir mnt proc root sys tmp var
```

##### 封装构建好的根文件系统，并挂载

```
cd /home/

sudo mkdir temp

sudo dd if=/dev/zero of=rootfs.ext3 bs=1M count=32

sudo mkfs.ext3 rootfs.ext3

sudo mount -t ext3 rootfs.ext3 temp/ -o loop

sudo cp -r nfs/* temp/

sudo umount temp

sudo mv rootfs.ext3 tftpboot

cd /home/tftpboot

sudo gedit start.sh
```

更改启动脚本start.sh:
和前面一样，root登录才能启动，当然也可以用上边的启动命令

```
qemu-system-arm \
        -M vexpress-a9 \
        -m 512M \
        -kernel zImage \
        -dtb vexpress-v2p-ca9.dtb \
        -nographic \
        -append "root=/dev/mmcblk0 rw console=ttyAMA0" \
        -sd rootfs.ext3
```







## 打包为squashfs镜像的linux命令

这个命令通常不用，因为squashfs是只读的

```
mksquashfs source_dir dest_file
例：
mksquashfs /root/files myfiles.sqsh
```







## 网络--用的是tap的连接方案，其他方案[见此](https://blog.csdn.net/qq_34160841/article/details/104901127)

##### 1)创建网桥

```
sudo brctl addbr virbr0
```

如果创建失败请用root账户或者自己手动创建。

```
sudo echo 'allow virbr0' >> /etc/qemu/bridge.conf
```

##### 2)打开网桥stp

```
sudo brctl stp virbr0 on
```

##### 3）添加tap虚拟网卡

```
sudo ip tuntap add name virbr0-nic mode tap
```

##### 4）启动网卡

```
sudo ip link set dev virbr0-nic up
```

##### 5）虚拟网卡添加到网桥

```
sudo brctl addif virbr0 virbr0-nic
```

##### 6）开机启动网桥--可选

```
sudo systemctl enable libvirtd
sudo dhclient virbr0
```

##### 7）网桥自动获取IP

```
sudo dhclient virbr0
```

##### 8)查看网桥状态

```
brctl show 
```



#### qemu启动命令

```
-net nic,macaddr=<mac addr>,model=ftgmac100  \
-net bridge,id=net0,helper=/usr/lib/qemu-bridge-helper,br=virbr
```

这里<mac addr> 可以直接copy**网桥**的mac，然后修改后几位，model、以及后面的“-net bridge ...”参数可以不要，但是一般建议带此参数。另外如果在启动多个虚拟机时，需要设置不同mac，否则所有虚拟机使用同一个IP地址。



## tap接口方案——主要方案，每次启动ubuntu都得配置

### 主机

首先修改/etc/netplan/01-network-manager-all.yaml，添加以下内容，主要是添加br0

```
    ens34:
     dhcp4 : no
  bridges:
    br0:
            dhcp4 : yes
            interfaces:
                    - ens34
```

##### 添加网桥

```
brctl addbr br0  //在上边配置了就不用添加了

sudo brctl stp br0 on


ifconfig br0 192.168.4.1 netmask 255.255.255.0 up 

brctl show   ——要看到br0以及stp on
```



##### 添加接口

```
tunctl -t tap0    //注意这个tap0，如果启动命令中网络部分ifname=tap2，那添加的接口就是tap2
ifconfig tap0 192.168.4.2/24 up
brctl addif br0 tap0

brctl show 检查
```



于root下做



### qemu虚拟机

```
ifconfig eth0 192.168.4.12/24 up   配地址就行了

ifconfig eth0 192.168.4.25 netmask 255.255.255.0 up
```



### 启动命令

```
        -net tap,ifname=tap0,script=no \
        -net nic \
```

[参考](https://blog.csdn.net/qq_34160841/article/details/104901127)https://blog.csdn.net/qq_34160841/article/details/104901127





## 文件系统用initrd挂载

进入binwalk解压得到的文件系统目录

```
find . | cpio --create --format='newc' | gzip -k  > ../xxx.cpio.gz
```

然后可放到qemu中使用`-initrd`挂载。注意`-initrd` 选项适用于 initrd 和 initramfs 两种类型的初始化根文件系统。这是因为 QEMU 的命令行选项在这方面是通用的，不管使用哪种类型的初始化根文件系统，都可以使用 `-initrd` 选项来指定要加载的映像文件。

可[参考](https://osh-2020.github.io/lab-1/initrd/)https://osh-2020.github.io/lab-1/initrd/

------

#### initrd和"initramfs区别

`initramfs`和`initrd`都是用于引导Linux操作系统的初始化根文件系统。

1. **initrd**（Initial RAM Disk）：initrd是早期Linux系统中使用的一种初始化根文件系统的方法。它是一个被加载到内存中的临时文件系统，通常包含用于启动系统的必要工具和驱动程序。initrd是一个压缩的文件系统映像，它在启动时被加载到内存中，并用于提供必要的资源来挂载真正的根文件系统。一旦真正的根文件系统被挂载，initrd就会被卸载。
2. **initramfs**（Initial RAM Filesystem）：initramfs是较新的一种初始化根文件系统的方法，它是在Linux内核中广泛使用的。与initrd不同，initramfs不是一个压缩的文件系统映像，而是一个由内核动态生成的文件系统。它由一组初始文件和目录组成，这些文件和目录在系统启动时被加载到内存中。initramfs在启动过程中提供了必要的工具、驱动程序和库文件，以便挂载和准备真正的根文件系统。与initrd相比，initramfs更加灵活，允许动态加载和卸载模块，以及支持更复杂的文件系统结构。



`format=raw`似乎只能用-sd挂载，`format=raw`参数在某些情况下，特别是在一些特殊用途的场景中，可以被用于指定将磁盘格式化为原始格式，而不是使用任何特定的文件系统。这**意味着磁盘上的数据将被视为连续的字节流，而不会被解释为文件系统的组织结构**。







## 以-sd方式挂载的EN25与GD25固件

##### GD25

```
qemu-system-arm \
        -M vexpress-a9 \
        -m 1024M \
        -kernel /home/riria/Desktop/linux-core/linux-4.4.282/arch/arm/boot/zImage  \
        -dtb /home/riria/Desktop/linux-core/linux-4.4.282/arch/arm/boot/dts/vexpress-v2p-ca9.dtb \
        -nographic \
        -append "root=/dev/mmcblk0 rw console=ttyAMA0" \
        -sd  /home/riria/Desktop/file-system/gd25test.ext3 \
        -net tap,ifname=tap0,script=no \
        -net nic \

```

##### EN25

```
qemu-system-arm \
        -M vexpress-a9 \
        -m 1024M \
        -kernel /home/riria/Desktop/linux-core/linux-4.9.129/arch/arm/boot/zImage \
        -dtb /home/riria/Desktop/linux-core/linux-4.9.129/arch/arm/boot/dts/vexpress-v2p-ca9.dtb \
        -nographic \
        -append "root=/dev/mmcblk0 rw console=ttyAMA0" \
        -sd /home/riria/Desktop/file-system/test-en25.ext3 \
```




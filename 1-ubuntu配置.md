# ubuntu20.04配置

------

## 基本信息

静态IP

##### 虚拟机内地址：

IPv4：169.254.248.148

netmask：255.255.255.0

gateway：169.254.248.2

##### VMware外部配置

NAT模式

子网IP：169.254.248.0

DHCP设置：起始段169.254.248.128，结束段169.254.248.254

子网掩码：255.255.255.0

记得给主机也配上VMnet8的适配（有个选项可以勾上），不然主机ping不到虚拟机



已换源为中科大，在/etc/apt/sources.list文件中修改，换完后要sudo apt update一遍

ubuntu root密码为riria，远程连接同

## 开机没网

[这个，20版本后都得改netplan](https://www.jianshu.com/p/dbbfe02745fa)

把networkManager删掉，在/etc/netplan里边

这里配置需要注意的时你需要在dhcp: 4这种写法后边加空格，要不然在文本中不会高亮显示（编译时会语法错误）

```bash
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd#把NetworkManager删掉
  ethernets:
    ens33:#网卡名称
     dhcp4: no
     addresses: [192.168.31.2/24]#输DHCP给你配的ip，自己弄的静态ip也行
     gateway4: 192.168.1.1#网关
     nameservers:
       addresses: [114.114.114.114,8.8.8.8]
```

记得配完输入netplan apply

配完了nslookup看看，或者service network-manager restart一下（虽然好像和netplan无关）

------

基本上这一步配完就有网了，可以ping到百度。但是只有开了梯子才能下东西，国内如果要换源的话看[这个](https://midoq.github.io/2022/05/30/Ubuntu20-04%E6%9B%B4%E6%8D%A2%E5%9B%BD%E5%86%85%E9%95%9C%E5%83%8F%E6%BA%90/)，修改/etc/apt/sources.list文件（把里边的删了然后替换下边的，最好备份一个sources.list），共有四个源：

#### 阿里云——我用了下感觉缺些东西

```
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

# deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

## Pre-released source, not recommended.
# deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
```

#### 清华

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

## Pre-released source, not recommended.
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

#### 中科大——在用的

```
deb https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse

## Pre-released source, not recommended.
# deb https://mirrors.ustc.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

#### 网易163

```
deb http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse

# deb-src http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse

## Pre-released source, not recommended.
# deb http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
```

换完 sudo apt update

## VMware远程传输

从Ubuntu14.04开始open-vm-tools 代替了官方 VMware Tools

安装步骤：

1 更新下系统源

*sudo apt update*

2 安装open-vm-tools

*sudo apt install open-vm-tools*

3 如果要实现文件夹共享，需要安装 open-vm-tools-dkms

*sudo apt install open-vm-tools-dkms*

4 桌面环境还需要安装 open-vm-tools-desktop 以支持双向拖放文件

*sudo apt install open-vm-tools-desktop*

其实一般安桌面环境这个就行了



记得装完后Reboot



作者：yancolin
链接：https://www.jianshu.com/p/687acbfd21a5
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



## DNS

改DNS[链接](http://www.tlcement.com/359166.html)，ping外边时域名无法解析问题

检查 DNS 设置。可以使用 `nslookup` 命令来测试域名解析是否正常：

```
nslookup www.google.com
```

------

或者这个[链接](https://codeantenna.com/a/JeShlbAtkl)

修改 /etc/netplan/01-netcfg.yaml的文件中nameservers部分，如下：

```
# Let NetworkManager manage all devices on this systemnetwork:
network:
  ethernets:
    eth0 : #配置的网卡的名称
      addresses: [192.168.0.103/24]    #配置的静态ip地址和掩码
      dhcp4: no    #关闭DHCP，如果需要打开DHCP则写yes
      optional: true
      gateway4: 192.168.0.1    #网关地址
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]    #DNS服务器地址，多个DNS服务器地址需要用英文逗号分隔开
  version: 2
  renderer: networkd    #指定后端采用systemd-networkd或者Network Manager，可不填写则默认使用systemd-workd
```

 改完后，保存退出，重新启动netplan：sudo netplan apply

或者

```
service network-manager restart
```

注意，ubuntu20后上边这个指令会让resolv.conf复原，所以如果要改resolv.conf的话需要解除resolv.conf的依赖关系，然后新建一个resolv.conf保存自己要的静态ip以及nameserver。具体参考[这个](https://blog.51cto.com/renzailvtu/764913)静态链接配置。

如果是 Kali Linux（Debian），则需要用以下命令：

```
service networking restart 
```

如果是Centos，则需要用以下命令：

```
nmcli c reload
```





## 配置SSH做VScode远程调试

[参考](https://blog.51cto.com/zhangsz0516/6103251)https://blog.51cto.com/zhangsz0516/6103251

##### 主机端：

VScode上在商店里下载Remote-SSH组件，左下角有小点就证明安好了



##### Ubuntu端：

apt-get install openssh-server -y

service ssh start

ps -e|grep ssh 							  查看ssh启动没，返回sshd就ok了

systemctl enable ssh					开机启动

ufw disable									关防火墙



##### 连接

在VScode的Remote-SSH里边选择Connect to Host--->add New SSH Host--->输入SSH用户名（主机上的）@ubuntu主机ip -A（e.g ssh riria@192.168.149.128 -A）

设置之后，如果在左边没有出现可供连接的ip，点左下角的小点选择Connet current windows to host，然后就自己在连接了。如果要改相关的参数就去.config里改，一样点下边小点会弹参数



然后需要有网给VScode安装VScode-ssh-server组件，要有网才能装



##### 但上面只能做到普通用户连接，想要root用户的话得做以下修改

在ubuntu中

###### 现设置Root密码，把随机的密码变得固定

sudo passwd

用

su root

验证密码

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

------



### C语言Debug调试配置

打开项目，发现居然用命令行编译文件。这不行，得在本机有点自动化的东西。

[参考](https://kcjia.cn/2021/05/22/vscode-remote-c/)https://kcjia.cn/2021/05/22/vscode-remote-c/

在VScode中的.vscode文件夹下配置项目信息

```
tasks.json

{
    "tasks": [
        {
            "type": "process",
            "label": "CFileBuild",
            "command": "/usr/bin/gcc",//编译器位置，改成ubuntu里边的
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",//要编译的源文件
                "-o",
                "${fileDirname}/out/${fileBasenameNoExtension}.out"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "presentation": {
                "echo": true,
                "focus": false,  
                "panel": "shared"   // 不同的文件的编译信息共享一个终端面板
            },
            "group": {
                "kind": "build",
                "isDefault": true // 不为true时ctrl shift B就要手动选择了
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}


```

------



```
launch.json//没有创建一个

   {
 // https://go.microsoft.com/fwlink/?linkid=830387
 "version": "0.2.0",
 "configurations": [
     {
         "name": "(gdb) Launch",
         "type": "cppdbg",
         "request": "launch",
         //这里要改成task中配置的输出的文件地址
         "program": "${fileDirname}/out/${fileBasenameNoExtension}.out",
         "args": [],
         "stopAtEntry": false,
         "cwd": "${workspaceFolder}",
         "environment": [],
         "externalConsole": false,
         "MIMode": "gdb",
         "setupCommands": [
             {
                 "description": "Enable pretty-printing for gdb",
                 "text": "-enable-pretty-printing",
                 "ignoreFailures": true
             }
         ],
         "preLaunchTask": "buildC" //这里表示debug开启之前执行的任务，对应task中的label字段
     }
 ]
   }
```



## 解决问题

**解决Ubuntu系统“无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系”的有效方法**

用aptitude来帮助降级，aptitude install <target>




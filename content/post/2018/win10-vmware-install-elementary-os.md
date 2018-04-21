+++
title = 'Win10 VMware 安装 Elementary OS'
date = 2018-04-21T10:59:40+08:00
categories = ["Linux"]
tags = ["elementary", "vmware"]
+++

![Elementary OS Poster](/images/post/win10-vmware-install-elementary-os/elementary-os-poster.jpg)

<!--more-->

> elementary OS是一个基于Ubuntu的Linux发行版。它使用一个自己开发的基于GNOME的名为Pantheon的桌面环境。 这个桌面环境出众的原因是它深度集成了其他elementary OS应用程序，如Plank（一个基于Docky的Dock）、Midori（默认的网页浏览器）或Scratch（一个简单的文本编辑器）。该发行版使用基于Mutter的Gala作为其窗口管理器。
>
> 这个发行版是从为Ubuntu设计的一套主题和应用程序发展而来的。由于是基于Ubuntu的，因此与Ubuntu的仓库和包完全兼容。它使用Ubuntu自己的软件中心来处理软件的安装和卸载。其类似于Mac OS X的界面致力于使新用户不需要费太大力气就可以根据直觉使用。
> —— [elementary OS wiki](https://zh.wikipedia.org/wiki/Elementary_OS)

2018年，**elementary OS** 有文章评价为“**最美的Linux桌面发行版**”，其桌面环境和使用跟 Mac OS X 很相似；如果你想在 Windows 下拥有 Mac 系统的操作体验，那么在虚拟机中安装 elementary os 是个最好的选择，可以随时在 Windows 办公娱乐、elementary OS 开发环境之间切换。

本文将一步步介绍在 Win10 VMware 中安装 elementary OS 的步骤。

### 下载 Elementary OS 镜像
1. 进入官网 https://elementary.io/zh_CN/
1. 输入金额 0，点击下载 iso 镜像
![download-elementary-os-iso](/images/post/win10-vmware-install-elementary-os/download-elementary-os-iso.png)

### 下载安装 VMware Workstation Pro
1. 进入 VMware 官网下载页面下载 https://my.vmware.com/en/web/vmware/info/slug/desktop_end_user_computing/vmware_workstation_pro/14_0
1. 安装
1. 输入序列号激活 `CV7T2-6WY5Q-48EWP-ZXY7X-QGUWD`、`FF31K-AHZD1-H8ETZ-8WWEZ-WUUVA`

### 创建 elementary OS 虚拟机
1. 启动 VMware

1. 创建新的虚拟机，选择**自定义**，下一步
<center>
![new-vm](/images/post/win10-vmware-install-elementary-os/new-vm.png)
</center>

1. 选择虚拟机兼容性，默认下一步

1. 选择**安装程序光盘映像文件**，选中刚才下载的 Elementary OS iso镜像文件，下一步
<center>
![select-iso](/images/post/win10-vmware-install-elementary-os/select-iso.png)
</center>

1. 操作系统版本选择 **Linux**、**其他 Linux 4.x 或更高版本内核 64位**，下一步
<center>
![select-os](/images/post/win10-vmware-install-elementary-os/select-os.png)
</center>

1. 修改默认的虚拟机名称和存储位置，下一步
<center>
![vm-location](/images/post/win10-vmware-install-elementary-os/vm-location.png)
</center>

1. 处理器配置2核（当前操作系统内核数的一半），下一步
<center>
![vm-cpu](/images/post/win10-vmware-install-elementary-os/vm-cpu.png)
</center>

1. 内存看个人选择，这里选择3G，下一步
<center>
![vm-memory](/images/post/win10-vmware-install-elementary-os/vm-memory.png)
</center>

1. 网络类型选择 **桥接网络**，下一步
<center>
![vm-network](/images/post/win10-vmware-install-elementary-os/vm-network.png)
</center>

1. 选择I/O控制器类型，默认下一步

1. 选择硬盘类型，默认下一步

1. 创建新虚拟磁盘，（使用物理磁盘可能会影响到主机系统的正常使用），下一步
<center>
![vm-disk](/images/post/win10-vmware-install-elementary-os/vm-disk.png)
</center>

1. 最大磁盘大小根据个人选择，这里选择40G，选择**将虚拟磁盘存储为单个文件**，下一步
<center>
![vm-disk-size](/images/post/win10-vmware-install-elementary-os/vm-disk-size.png)
</center>

1. 完成创建虚拟机
<center>
![vm-done](/images/post/win10-vmware-install-elementary-os/vm-done.png)
</center>


### 安装 elementary OS
1. 开启此虚拟机

1. 语言选择 **English**（或者中文简体），点击**Install Elementary**
<center>
![os-language](/images/post/win10-vmware-install-elementary-os/os-language.png)
</center>

1. 选择 **Install third-party software**，continue
<center>
![os-install-third-party-software](/images/post/win10-vmware-install-elementary-os/os-install-third-party-software.png)
</center>

1. Install Type 默认选择**擦除磁盘安装Elementary OS**，点击**Install Now**，**Continue**
<center>
![os-install-type](/images/post/win10-vmware-install-elementary-os/os-install-type.png)
</center>

1. 选择城市，**Contiune**

1. 键盘布局选择**Chiese**，**Continue**

1. 设置个人用户名、密码

1. 安装过程等待5分钟左右，完成后重启
<center>
![os-done](/images/post/win10-vmware-install-elementary-os/os-done.jpg)
</center>


### 在虚拟机中安装 VMTools
1. 选择 **虚拟机 > 安装 VMware Tools**

1. 在虚拟机中会挂载 **VMware Tools** CD

1. 进入挂载的 **VMware Tools**，拷贝**VMwareTools**文件到**Documents**目录下（在CDROM中无法写入）

1. 进入**Documents**目录，用`tar`命令解压**VMwareTools**文件
    ```shell
    tar zxf VMwareTools
    ```

1. 进入解压出的**VMwareTools**，执行**vmware-install.pl**文件
    ```shell
    cd VMwareTools
    sudo vmware-install.pl
    ```

1. 一路默认回车，完成安装

1. 要在主机和虚拟机之间拖拽文件和复制粘贴文本，需要执行
    ```shell
    /usr/bin/vmware-user
    ```

1. 注销，重新登录


## 虚拟机无法连接网络
1. 虚拟机 > 设置 > 选择桥接模式
<center>
![network-bridge](/images/post/win10-vmware-install-elementary-os/network-bridge.png)
</center>

1. 编辑 > 虚拟网络编辑器 > 桥接模式桥接到你的物理网卡
<center>
![network-edit](/images/post/win10-vmware-install-elementary-os/network-edit.png)
</center>

### 测试主机和虚拟机之间能够互相访问
- 主机ping虚拟机IP
- 虚拟机访问主机开启的http服务（可能无法ping通主机IP）

---

到这里，elementary OS 在 VMware 中就安装完成了，后面还会介绍 elementary OS 的初始化和使用
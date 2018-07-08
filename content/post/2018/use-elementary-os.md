+++
title = '使用 Elementary OS'
date = 2018-07-08T18:16:57+08:00
categories = ["Linux"]
tags = ["elementary"]
+++

[上一篇](https://jancat.github.io/post/2018/win10-vmware-install-elementary-os/) 介绍了在 Win10 上安装 Elementary OS，这篇介绍下 Elementary OS 的简单使用。

![elementary-desktop](/images/post/use-elementary-os/elementary-desktop.png)
<!--more-->

## 包管理 apt-get
因为 Elementary OS 是基于 Ubuntu 的 Linux 发行版，而 Ubuntu 又是基于 Debian，所有基于 Debian 的发行版都使用 `apt-get` 这个包管理系统。deb 包可以把一个应用的文件包在一起，大体就如同 Windows 上的安装文件。

使用 `apt-get` 命令的第一步就是引入必需的软件库，Debian 的软件库也就是 Debian 软件包的集合，它们存在互联网上的一些公共站点上。把它们的地址引入后，`apt-get` 就能搜索到我们想要的软件。`/etc/apt/sources.list` 是存放这些地址列表的配置文件，其格式如下：

```shell
deb [web或ftp地址] [发行版名字] [main/contrib/non-free]
```

在修改 `/etc/apt/sources.list` 之后需要执行 `sudo apt-get update` 来更新本地包列表。

### 替换为 阿里云源
Elementary OS 的软件源默认为国外的源，速度较慢，可以更换为国内速度较快的阿里云软件源：

1. 在终端下修改 `/etc/apt/sources.list` 文件的可写权限：
    
    ```shell
    sudo chmod a+w /etc/apt/source.list
    ```
1. 备份 **source.list**

    ```shell
    sudo cp /etc/apt/source.list /etc/apt/source.list.bak
    ```
1. 在 **Scratch** 文本编辑器中打开 `/etc/apt/source.list`
1. 删除原有内容，替换为 **阿里云源**

    ```txt
    deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
    ```
1. 执行更新
    ```shell
    sudo apt-get update
    ```

## 常用软件安装
### 搜狗拼音输入法
Linux 系统跟应用程序打交道的是输入法框架，其中 IBus 是目前主流发行版的默认配置。各种输入法要有相对应的输入法框架支持，中文输入法是基于 **fcitx** 的，而系统默认的是 iBus，所以要先安装 **fcitx 中文输入法框架**。

#### 安装 fcitx
1. 安装第三方软件源 `software-properties-common` 和 `python-software-properties`
    
    ```shell
    sudo apt-get install software-properties-common python-software-properties
    ```
1. 添加 **ppa** 源
    
    ```shell
    sudo add-apt-repository ppa:fcitx-team/nightly
    ```
1. 更新软件
    
    ```shell
    sudo apt-get update
    ```
1. 安装 fcitx
    
    ```shell
    sudo apt-get install fcitx 
    ```
1. 安装 fcitx 的配置工具、table-all 软件包、fcitx-table-all、im-config 切换工具
    
    ```shell
    sudo apt-get install fcitx-config-gtk fcitx-table-all im-config
    ```
1. 修复软件依赖
    
    ```shell
    sudo apt-get -f install
    ```
1. 验证 fcitx 是否安装成功，在 **Applications** 中查看是否有 fcitx 的应用图标

#### 安装搜狗拼音输入法
1. 在 [搜狗官网](https://pinyin.sogou.com/linux/?r=pinyin) 下载输入法 Linux 版 64位
1. 在下载目录中使用 `dpkg` 命令安装 deb 包
    ```shell
    sudo dpkg -i sougoupinyin-xxx.deb
    ```
1. 更新包依赖
    ```shell
    sudo apt -f install -y
    ```
1. 重启系统
1. 在终端中输入 `im-config`，打开 **Input Method Configuration** 输入法配置界面
1. 确定默认输入法为 **搜狗拼音**
1. 在桌面右上角中启用搜狗输入法
    ![sougou-pinyin](/images/post/use-elementary-os/sougou-pinyin.jpg)

### Chrome
```shell
sudo apt-get install google-chrome-stable
```

### Monitor
进程管理工具，在 App Center 中搜索 **Monitor**

### Albert
Alfred in Elementary os 
快速搜索、启动应用神器
```shell
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt update
sudo apt install albert

sudo apt -f install
```

## 系统设置
#### Dock 设置（位置、图标）
终端执行 `plank --preferences`

## 常见问题
### dpkg 文件被锁，其它进程正在占用
```shell
E: Could not open lock file /var/lib/dpkg/lock - open (13: Permission denied)
E: Unable to lock the administration directory (/var/lib/dpkg/), are you root?
```
**解决：** 删除文件锁
```shell
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock
```
---

### Unable to locate package
执行 `sudo apt-get update`

--- 

### Could not get lock 无法锁文件
```shell
E: Could not get lock /var/lib/apt/lists/lock - open (11: Resource temporarily unavailable)
E: Unable to lock directory /var/lib/apt/lists/
```
**原因：** 文件锁冲突
**解决：**
```shell
sudo rm /var/lib/apt/lists/lock
sudo apt-get update
```
---

### W: 无法下载 http://ppa.launchpad.net/fcitx-team/nightly/ubuntu/dists/jessie/main/binary-amd64/Packages
**解决：** 执行：
```shell
cd /etc/apt/sources.list.d
sudo mv xx.list xx.list.bak
```
---
title: DL Machine系列-00 環境建置
date: 2019-12-17 15:30:26
tags: [ubuntu, ubuntu-18.04, xrdp, static ip, nvidia-gpu, docker]
categories: DL Machine
thumbnail: /gallery/thumbnails/nvidia_docker-cover.png
toc: true
---

## 前言

有鑒於每次實驗架設環境都花很多時間，也常常遇到版本更新後某些套件不相容的問題，所以才打算使用Docker來架設深度學習的虛擬環境~~(絕對不是因為覺得用Docker很帥才用的)~~，說到深度學習就不得不使用GPU，使用GPU就不得不用`nvidia-docker`來架設環境，剛剛好本人的實驗室目前有一台電腦正空閒下來，也剛剛好多出一張RTX 2070S，想說利用Docker來一勞永逸這個問題，也順便試試看多GPU的環境是什麼樣的感覺。

<!--more-->

## 電腦配置

CPU： AMD Ryen Threadripper 1900X 8-core
MotherBoard：ROG Strix X399-E Gamming
GPU：RTX 2070 super, GTX 1050
RAM：Kingston 16Gx8 2933MHz
Storage：2TB SSD*2, 1\*1TB m.2 SSD
Power：750w金牌電源

## Ubuntu 18.04安裝

在安裝Ubuntu的時候，會顯示出

`install ubuntu/ try ubuntu without installation`
`install ubuntu`

等等的選項，但是選擇後螢幕變黑屏沒反應。

經過查找，應該是因為Ubuntu對於RTX顯示卡沒有對應的Driver，所以導致這個問題。

**我這邊使用另一張顯卡安裝，再去更新Nvidia-driver來避免這個問題。**

安裝的時候選擇：

`在新安裝的Ubuntu上使用LVM`

這是因為之後新增硬碟用LVM來管理。

## 確認GPU狀態

執行`ubuntu-drivers devices`去確認

如果你只有看到這一項：`nvidia-driver-390 - distro non-free`，那你必須去將NVIDIA repository加入到你的apt庫。

可以用`dpkg -l 'nvidia*'`去看電腦上安裝的Nvidia Driver

執行`sudo ubuntu-drivers autoinstall`安裝driver，完成之後執行`nvidia-smi`就可以看到：

![Nvidia SMI](nvidia-smi.png)

## 安裝基本工具

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install vim #好用的編輯器
sudo apt-get install net-tools curl #用來看網路介面卡
sudo apt-get install gparted #硬碟管理工具
```
## 設定ip(區域網路)

打開`/etc/netplan/01-network-manager-all.yaml`
更改成這樣：
```yaml
# Let NetworkManager manage all devices on this system
network:
    version: 2
    # renderer: NetworkManager
    eno1:
        addresses: [192.168.123.111/24]
        gateway4: 192.168.111.1
        nameservers:
            addresses: [8.8.8.8,8.8.4.4]
```
>參數說明：
>eno1： 網卡名稱(可以透過`ifconfig`查看)
>addresses： 要指定的ip
>gateway4： 閘道 ip4(gateway6 閘道 ip6)
>nameservers： dns 以逗號階隔
>註：
>/32 指的是 network mask of 255.255.255.255
>/24 指的是 network mask of 255.255.255.0

依照個人網路調整即可，用`sudo netplan apply`就可以套用剛剛的設定了。

## 安裝vnc遠端操控

安裝 xfce4 與 xrdp
```bash
sudo apt-get install xfce4
sudo apt-get install xrdp
```
配置登入環境

```bash
echo xfce4-session > ~/.xsession
sudo vim /etc/xrdp/startwm.sh
```

將`stratum.sh`更改：

```diff
if test -r /etc/profile; then
        . /etc/profile
fi

test -x /etc/X11/Xsession && exec /etc/X11/Xsession
-exec /bin/sh /etc/X11/Xsession
+startxfce4
```
啟動 xrdp 服務：

```bash
sudo service xrdp restart
```

確認服務正常運行：

```bash
netstat -na | grep 3389
```

這時就可以透過Windows的遠端桌面連線到你的Linux主機了：

![xrdp-Remote](xrdp-remote.png)

## 補充

可以透過：

```bash
sudo lshw -html > ~/hardware.html
```

來看這台電腦的硬體配備，用瀏覽器打開即可。
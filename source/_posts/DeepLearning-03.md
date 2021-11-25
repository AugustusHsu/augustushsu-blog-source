---
title: DL Machine系列-03 Docker
date: 2019-12-23 12:03:26
tags: [docker, Dockerfile, portainer, xrdp, anaconda, cuda, cudnn]
categories: DL Machine
thumbnail: /uploads/docker.jpg
toc: true
---

## 前言

網路上關於Docker的資訊已經有很多了，這邊就不多作介紹了，只針對幾個常用和在我的實作上有用到的指令和套件去做介紹。

用一張圖來簡單的說明Docker的架構：

<!--more-->

![Architecture](docker-containerized&vm.png)

## Portainer

Portainer是一個用來管理Docker的工具，他可以透過網頁來查看或管理目前執行的container等等，也可以很快速地進入一個正在執行的container，簡而言之就是一種用來管理Docker的圖形化介面。

### 安裝

可以用`docker search portainer`來查看目前有哪些可以用的資源：

```bash
sudo docker search portainer
# output
NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
portainer/portainer              Making Docker management easy. https://porta…   1439
portainer/agent                  An agent used to manage all the resources in…   50
portainer/templates              App Templates for Portainer http://portainer…   18
```

下載：

```bash
docker pull portainer/portainer
```

依據官方的文件啟動container，當然你可以自訂你想要的port (第一個是host port，第二個是container port)：

```bash
docker volume create portainer_data
docker run -d -p 9000:9000 \
--name portainer \
--restart always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data \
portainer/portainer
```

打開瀏覽器就能看到：

![Portainer](portainer.png)

這樣就可以管理docker的`image`、`container`跟`volume`，還可以看到其他的資源。

### 常用Docker指令

記錄一些常用的Docker指令，以備不時之需，不過我想`protainer`應該可以取代大部分功能：

```bash
# 查看目前的image
docker images
# 刪除image
docker rmi [OPTIONS] IMAGE [IMAGE...]
# 查看目前運行的 container
docker ps
# 查看目前全部的 container（ 包含停止狀態的 container ）
docker ps -a
# 停止 Container
docker stop [OPTIONS] CONTAINER [CONTAINER...]
# 删除 Container
docker rm [OPTIONS] CONTAINER [CONTAINER...]
# 查看 Container 詳細資料
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
# 查看 log
docker logs [OPTIONS] CONTAINER
# 顯示容器資源 ( CPU , I/O ...... )
docker stats [OPTIONS] [CONTAINER...]
# 停止指定的 CONTAINER 中全部的 processes
docker pause CONTAINER [CONTAINER...]
# 恢復指定暫停的 CONTAINER 中全部的 processes
docker unpause CONTAINER [CONTAINER...]
```

> `docker stop` : process 級別。
>
> `docker pause`: container 級別。

## 實作Deep Learning環境

我這裡選擇的image是`ubuntu18.04`，然後透過`ARG`來新增使用者：

```dockerfile
FROM ubuntu:18.04
MAINTAINER jim jimhsu11@gmail.com

ARG USERNAME
ARG USERPWD
```

### DEBIAN_FRONTEND noninteractive

接下來這個步驟很重要：

```dockerfile
# debconf to be non-interactive
ENV DEBIAN_FRONTEND noninteractive
```

因為`ubuntu`在安裝的時候，某些套件會需要輸入指令，這邊將它設定成**沒有交互介面**的模式來安裝。

### Add User

接著就是新增使用者，這邊新增使用者主要是為了之後的`xrdp`套件，在run docker的時候可以不用再去建立使用者，不過在build的時候要記得加上`ARG`參數:

```dockerfile
# Update and Add User
RUN apt-get update \
    && apt-get install -y vim sudo wget \
    && useradd -ms /bin/bash ${USERNAME}\
    && sudo adduser ${USERNAME} sudo\
    && echo ${USERNAME}:${USERPWD} | chpasswd
```

### xrdp

這邊安裝`xrdp`套件，讓Windows系統可以透過`遠端桌面連線`連線到Container：

```dockerfile
# xrdp
RUN apt-get update \
    && apt-get install -y xfce4 xfce4-goodies xorg dbus-x11 x11-xserver-utils xrdp \
    && echo xfce4-session > /home/${USERNAME}/.xsession \
    && sed -i "s/^exec.*Xsession$/startxfce4/g" "/etc/xrdp/startwm.sh" \
    && service xrdp restart
```

這邊跟一般安裝`xrdp`的過程一樣，其中`sed`是將`/etc/xrdp/startwm.sh`最後一行替換成`startxfce4`。

### Anaconda

這邊使用[Anaconda](https://www.anaconda.com/)來管理`python`的套件，雖然已經使用Docker來隔離系統了，不過還是習慣用Anaconda來建立python環境：

```dockerfile
# Install Anaconda
RUN wget --quiet https://repo.continuum.io/archive/Anaconda3-5.0.1-Linux-x86_64.sh -O ~/anaconda.sh \
    && /bin/bash ~/anaconda.sh -b -p /opt/conda \
    && rm ~/anaconda.sh \
    && echo "export PATH=/opt/conda/bin:$PATH" >> /home/${USERNAME}/.bashrc \
    && sudo chown -R ${USERNAME}:${USERNAME} /opt/conda
```

這邊`wget`後面的網址可以自己去更改，找符合自己[需求的版本](https://repo.continuum.io/archive/)來安裝。

然後安裝Anaconda的時候會需要輸入一些指令，所以用`-b`使用預設值安裝。

`-p`後面接的是安裝位置，這邊也可以自己去調整。

### cuda & cudnn

我這邊cuda使用的版本是10.0，雖然在系統上是安裝的版本是10.1，不過經過測試，是不影響使用的。

我是使用Nvidia/cuda的Dockerfile指令來安裝`cuda`和`cudnn`，分別將[nvidia/cuda:10.0-base-ubuntu18.04](https://gitlab.com/nvidia/container-images/cuda/blob/ubuntu18.04/10.0/base/Dockerfile)、[nvidia/cuda:10.0-runtime-ubuntu18.04](https://gitlab.com/nvidia/container-images/cuda/blob/ubuntu18.04/10.0/runtime/Dockerfile)和[nvidia/cuda:10.0-cudnn7-runtime-ubuntu18.04](https://gitlab.com/nvidia/container-images/cuda/blob/ubuntu18.04/10.0/runtime/cudnn7/Dockerfile)上需要的Dockerfile指令，加到自己的Dockerfile：

```dockerfile
#-------------------From Nvidia-------------------
# Nvidia install list
RUN apt-get update \
    && apt-get install -y --no-install-recommends gnupg2 curl ca-certificates \
    && curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub | apt-key add - \
    && echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list \
    && echo "deb https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/nvidia-ml.list \
    && apt-get purge --autoremove -y curl \
    && rm -rf /var/lib/apt/lists/*

ENV CUDA_VERSION 10.0.130
ENV CUDA_PKG_VERSION 10-0=$CUDA_VERSION-1

# For libraries in the cuda-compat-* package: https://docs.nvidia.com/cuda/eula/index.html#attachment-a
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        cuda-cudart-$CUDA_PKG_VERSION \
        cuda-compat-10-0 \
    && ln -s cuda-10.0 /usr/local/cuda \
    && rm -rf /var/lib/apt/lists/*

ENV PATH /usr/local/nvidia/bin:/usr/local/cuda/bin:${PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64
ENV NCCL_VERSION 2.4.2

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        cuda-libraries-$CUDA_PKG_VERSION \
        cuda-nvtx-$CUDA_PKG_VERSION \
        libnccl2=$NCCL_VERSION-1+cuda10.0 \
    && apt-mark hold libnccl2 \
    && rm -rf /var/lib/apt/lists/*

ENV CUDNN_VERSION 7.6.0.64

LABEL com.nvidia.cudnn.version="${CUDNN_VERSION}"

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
            libcudnn7=$CUDNN_VERSION-1+cuda10.0 \
    && apt-mark hold libcudnn7 \
    && rm -rf /var/lib/apt/lists/*
#-------------------Nvidia End-------------------
```

這邊有試過用nvidia本身的image來建立，不過失敗了，後來將nvidia上Dockerfile複製自己需要的部分卻成功了，原因沒有深究，如果有人知道的話，歡迎留言告訴我。

以上就是我的Dockerfile所有的內容。

## Build & Run & Upload

上面可以看到我的功能是一層一層添加的，實際上實作完一層，我就會build和run一次那個Dockerfile，以確保我的Dockerfile沒有寫錯，所以接下來就是將剛剛寫完的Dockerfile建立起來。

### Build Dockerfile

進入`Dockerfile`所在的資料夾，執行以下指令：

```bash
sudo docker build -t image_name:tag \
--build-arg USERNAME=username \
--build-arg USERPWD=yourpassword .
```

`-t`後面接的是image的名字跟`tag`，`USERNAME`跟`USERPWD`就是登入系統時要輸入的帳密，這樣image就建立好了。

### Run Image

執行下列命令就可以進入到創立的`container`了：

```bash
sudo docker run --gpus device=1 -it \
-p 33890:3389 \
-v /mnt/SSD:/data/SSD \
-v /mnt/HDD:/data/HDD \
-v /docker_config/config:/config \
image_name:tag
```

`--gpus`:可以指令你要用的gpu，當然是要你電腦上有安裝複數的gpu才能指令，不然可以直接用`--gpus all`來使用全部的gpu。

`-p`:因為有使用`xrdp`套件，而這個套件使用的port是3389，所以要將container的port映射到主機上的port，這邊選擇加上一個0。

`-v`:可以將主機上的資料夾位置映射到container上面，也可以是docker的volume映射到container上。這邊要注意的是冒號，冒號前是主機上的位置；後面是container上的位置。

`-it`：建立好後會直接進入container。

接下來要啟動`xrdp`套件才可以連線進去，執行：

```bash
service xrdp restart
```

啟動**遠端桌面連線**程式，輸入你主機的ip：

![Remote Desktop](remote-desktop.png)

再輸入前面設定的帳號密碼：

![Login](login.png)

進入到你的環境後，要記得進入`Setting Manager`：

![Settings Manager](settings-manager.png)

打開`Preferred Applications`：

![Preferred App](preferred-app.png)

選擇`Utilities`，將預設的Terminal Emulator改成Xfce Terminal：

![Xfce Terminal](xfce-terminal.png)

這樣就可以用Terminal了。

### Docker Hub

實作完自己的Dockerfile之後，用[Docker Hub](https://hub.docker.com/)備份或是分享到網上。

第一步要當然是註冊一個帳號，再利用docker login來登入：

```bash
docker login
# output
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: username
Password: yourpassword
```

登入完，直接用docker push來上傳就行了:

```bash
docker push yourusername/image_name:tag
```

## 連結

Docker Hub：https://hub.docker.com/repository/docker/augustushsu/ubuntu18.04-xrdp

Github：https://github.com/AugustusHsu/Docker-DL


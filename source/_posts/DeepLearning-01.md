---
title: DL Machine系列-01 安裝Docker-19.03+Nvidia-docker
date: 2019-12-18 17:32:56
tags: [docker-19.03, nvidia-docker, native gpu, nvidia-container-toolkit]
categories: DL Machine
thumbnail: /gallery/thumbnails/nvidia_docker-cover.png
toc: true
---

## docker安裝

`sudo apt-get remove docker docker-engine docker.io`
用來確保你的環境中沒有docker免得版本衝突，可以點[這個網站](https://download.docker.com)下載和你作業系統相符的docker安裝檔案。
我這邊下載的是[19.03.5版本](https://download.docker.com/linux/static/stable/x86_84/)。

<!--more-->

接著解壓縮跟copy到`bin`目錄：

```bash
tar xzvf docker-19.03.5.tgz
sudo cp -rf docker/* /usr/local/bin/
```

可以透過執行：

```bash
docker --version
sudo docker run hello-world
```

來確定版本跟能否順利執行。

## Nvidia Docker

- [x] 確認docker版本在19.03以上
- [x] linux kernel版本大於3.10(可以透過`uname -r`確認)
- [x] 你主機板上裝的GPU架構要在`Fermi(2.1)`以上(可以上[Wiki](https://en.wikipedia.org/wiki/List_of_Nvidia_graphics_processing_units)查看)
- [x] 還有GPU的Driver要361.93以上(可以用`nvidia-smi`在終端機查看)

將nvidia的資料庫加到電腦中：

```bash
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
```

安裝nvidia-container-toolkit：

```bash
sudo apt-get install -y nvidia-container-toolkit
```

參考資料：[nvidia-docker的wiki](https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(Native-GPU-Support))

## bug1-docker路徑問題

因為前面安裝docker是用手動安裝的，所以docker的位置跟用`sudo apt-get install docker`的位置不一樣，所以有以下錯誤碼：

```bash
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

接著我嘗試重新啟動docker：

```bash
sudo systemctl restart docker
# 以下為output
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details
```

要你執行`systemctl status docker.service`和`journalctl -xe`找詳細資料:

```bash
sudo systemctl status docker.service
# 以下為output
docker.service - LSB: Create lightweight, portable, self-sufficient containers.
   Loaded: loaded (/etc/init.d/docker; generated)
   Active: failed (Result: exit-code) since Mon 2019-12-16 23:47:44 CST; 20min ago
     Docs: man:systemd-sysv-generator(8)

12月 16 23:47:44 mars systemd[1]: Starting LSB: Create lightweight, portable, self-sufficient containers....
12月 16 23:47:44 mars docker[7032]:  * /usr/bin/dockerd not present or not executable
12月 16 23:47:44 mars systemd[1]: docker.service: Control process exited, code=exited status=1
12月 16 23:47:44 mars systemd[1]: docker.service: Failed with result 'exit-code'.
12月 16 23:47:44 mars systemd[1]: Failed to start LSB: Create lightweight, portable, self-sufficient containers..
```

看到`/usr/bin/dockerd`，因為安裝的時候dockerd是放在`/usr/local/bin/`裡面，因此要更改`docker.service`中的設定，前往`/etc/init.d/`，編輯`docker`:

```bash
cd /etc/init.d/
vim docker
```

將檔案中`DOCKERD`的位置改成上面手動安裝的位置：

```diff
-DOCKERD=/usr/bin/dockerd
+DOCKERD=/usr/local/bin/dockerd
```

接著重啟daemon和docker.service，然後查看docker.service：

```bash
systemctl daemon-reload && systemctl restart docker.service
sudo systemctl status docker.service
# 以下為output
docker.service - LSB: Create lightweight, portable, self-sufficient containers.
   Loaded: loaded (/etc/init.d/docker; generated)
   Active: active (running) since Tue 2019-12-17 00:08:53 CST; 2s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 7724 ExecStart=/etc/init.d/docker start (code=exited, status=0/SUCCESS)
    Tasks: 23 (limit: 19660)
   CGroup: /system.slice/docker.service
           └─7736 /usr/local/bin/dockerd -p /var/run/docker.pid

12月 17 00:08:53 mars systemd[1]: Starting LSB: Create lightweight, portable, self-sufficient containers....
12月 17 00:08:53 mars docker[7724]:  * Starting Docker: docker
12月 17 00:08:53 mars docker[7724]:    ...done.
12月 17 00:08:53 mars systemd[1]: Started LSB: Create lightweight, portable, self-sufficient containers..
```

可以看到可以成功執行了～～

接著試試看nvidia-docker：

```bash
# Starting a GPU enabled container
$ docker run --gpus all nvidia/cuda nvidia-smi

# Start a GPU enabled container on two GPUs
$ docker run --gpus 2 nvidia/cuda nvidia-smi

# Starting a GPU enabled container on specific GPUs
$ docker run --gpus device=1,2 nvidia/cuda nvidia-smi
$ docker run --gpus device=UUID-ABCDEF,1 nvidia/cuda nvidia-smi

# Specifying a capability (graphics, compute, ...) for my container
# Note this is rarely if ever used this way
$ docker run --gpus all,capabilities=utilities nvidia/cuda nvidia-smi
```

如果成功會跟你在電腦中執行`nvidia-smi`的結果一樣。

因為`docker-19.03`已經支援使用`NVIDIA GPUs`作為運行中的設備了。

上面指令有加上`--gpu`的選項，如果你不要加，可以在`Dockerfile`上加上：

```dockerfile
# 用來指定使用的gpu，和上面的--gpu相同功能
ENV NVIDIA_VISIBLE_DEVICES all
# 用來指定計算資源，跟上面寫的一樣，這個功能很少用到
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
```

## bug2-container無法stop, kill

只能使用`sudo systemctl restart docker`來重新啟動`docker`，這是因為原本我安裝的是19.03.1版本，這版本有無法刪除container的問題，在網路上搜索一段時間後，有一些人也有遇到這個問題，不過是在windows版本上。

簡而言之就是因為一些deadlock導致容器的API沒有任何回應，也就是無法stop,kill的問題。

後來[在github上看到有人說](https://github.com/moby/moby/issues/22357#issuecomment-559555618)19.03.5也就是最新版本，解決了deadlocks的問題，果斷更新，更換/usr/local/內所有從docker-19.03.1.tgz解壓縮的檔案，再重開機就沒有這個問題了。
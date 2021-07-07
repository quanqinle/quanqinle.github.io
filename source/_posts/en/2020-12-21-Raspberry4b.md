---
layout:       post
title:        "Raspberry | How to install and set up Raspberry 4b"
subtitle:     "A new Raspberry 4b with 4G RAM, to install OS and set up step by step"
date:         2020-12-21
updated:      2021-07-06
author:       "Quan Qinle"
header-img:   "img/in-post/raspberry-pi-4b.webp"
lang:         en
catalog:      true
tags:
    - Raspberry 4b

---

# Why to buy Raspberry

* I don't have NAS at home, it's not convenient to access my hard drives at any time
* Want to add an additional input source to the Mi TV
* Want to connect to the home network from the outside
* Want to download something into hard drives by 通过树莓派下载视频到移动硬盘

<!-- more -->

# About Hardware

* Raspberry 4b produced in 2020, 4G of RAM
* 32G TF flashcard
* During setting up process, there aren't any keyboard, mouse, monitor with Raspberry
* Using a laptop to read/write the flashcard
* Raspberry connects to a wireless hotspot provided by the laptop
* SSH to the Raspberry to complete all the operations on the laptop

# Choose the OS

I read some articles online that said the Raspberry with 4G RAM runs very well on 64-bit system, so I decided to choose the 64-bit OS to install.

I tried first to install `Ubuntu 64-bit LST`（[download-url]）, it did not go well, and I finally gave up.

[download-url]:https://ubuntu.com/download/raspberry-pi

Eventually I installed the `Offical 64-bit OS`.

So, download the [raspios_arm64 zip][1] on your laptop first.

[1]:https://downloads.raspberrypi.org/raspios_arm64/images/

# Install the OS

The tool [Raspberry Pi Imager][2] provided by offical for installing the OS is easy to use very much, however, it is not applicable for me, there are two reasons here, the one is it doesn't the offical 64-bit OS image in it, the other is the external keyboard and mouse are necessary when using this tool but I didn't have.

[2]:https://www.raspberrypi.org/software/

The tool [Win32DiskImager][3] is another tool I usually uesed, it can burn the system OS zip into a flashcard.

[3]:https://www.raspberrypi.org/documentation/installation/installing-images/windows.md

# Pre-configurate

Open the falshcard on the laptop, go into /boot directory
* Enable SSH access: to create a new empty file `ssh`
* Enable access to the network through hotspot mode: to create a new file `wpa_supplicant.conf`, add the following in it

```txt
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=CN
ap_scan=1
fast_reauth=1

network={
  ssid="hotspot-name-on-your-pc"
  psk="hotspot-password"
  priority=1
  id_str="hotspot"
}
network={
  ssid="wifi-name"
  psk="wifi-password"
  priority=2
  id_str="home5g"
}
```

On how to set up wireless from offical:  
https://www.raspberrypi.org/documentation/configuration/wireless/headless.md  
https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md

Note: for `psk`, use the plain text password with the double quotes, or use a pre-encrypted 32 byte password without the double quotes. You can get the pre-encrypted 32 byte password through the command line `wpa_passphrase "your pasword"`.

# Turn on the Pi

Insert the flashcard into the Pi, then plug it in the power, the Raspberry will start automatically.

# Log in the Pi

Find out the IP of the Raspberry on the hotspot management page of your laptop or the IP manangement page of your router.

Access the Raspberry with `SSH`：
```sh
ssh pi@IP-of-raspberry
```
The default user name and password are `pi / raspberry` when you use the offical OS.

# Change apt source to mirror site in China

I wanted to do so, but I can't find a mirror site which supported the arm64, such as Ali and Tsinghua. So I did NOT change the apt source and used the default.

If it was really slow for you, turn on the VPN before using apt.

# Change pip/pip3 to mirror site in China

```sh
mkdir ~/.pip
vi ~/.pip/pip.conf

[global]
timeout=100
index-url=http://mirrors.aliyun.com/pypi/simple/
extra-index-url=https://pypi.tuna.tsinghua.edu.cn/simple/
[install]
trusted-host=
    mirrors.aliyun.com
    pypi.tuna.tsinghua.edu.cn
```

# Set up: Enable VNC, change resolution, etc

```sh
sudo raspi-config
```
Use the arrow keys to move around the menus. Press the Enter key to select the menu or finish:

* In `Interfacing Options`, find `VNC`，select `Yes`
* In `Boot Options`, find `Desktop/CLI`，select `Desktop Autologin`
* In `Advanced Options`, find `Resolution`，select `1920*1080`
* In `Advanced Options`, select `Expand Filesystem`
* Use `Tab` key，move to `Finish`，press `Enter` to reboot

By now, the Raspberry Pi is configured. After reboot, it is ready to be accessed by VNC or connected to your TV.

# [Optional] Install Aria
Install the following:
+ aria
+ AriaNg: a front-end web management page, I recommend [All in One](https://github.com/mayswind/AriaNg/releases)

At the beginning, I used `sudo apt install aria2` to install, and modified the configuration by myself, unfortunately, the download speed was not ideal. Afterwards, I switched to a hot project on GitHub [“Aria2 一键安装管理脚本（增强版）”](https://github.com/P3TERX/aria2.sh)

After finishing the installment, rerun the script, then achieve configurating the following:
+ Select `修改 配置`, then `修改 Aria2 下载目录`
+ Enable `自动更新 BT-Tracker`

At this point, open the AriaNg webpage, `Aria2 status` always displays "连接中", and an error popup prompt: 认证失败!
> Solution:  
> 1. Open the configuration file `/root/.aria2c/aria2.conf`, find `rpc-secret=`, then copy its value
> 2. Open `AriaNg 设置——RPC (localhost:6800)` on browser, paste the above value into “Aria2 RPC 密钥”. The webpage url should be like this: http://loclhost/AriaNg/index.html#!/settings/ariang

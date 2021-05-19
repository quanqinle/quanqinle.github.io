---
layout:       post
title:        "小米 6 刷机"
subtitle:     "小米刷欧版 miui"
date:         2020-07-05
updated:      2020-07-05
author:       "权芹乐"
catalog:      true
tags:
    - 刷机
    - Android
---

[toc]

# 刷官方 ROM

## 下载官方 ROM
+ 下载所需官方版 ROM
+ 复制到手机存储，路径任意，根目录即可

## 开启隐藏功能【手动选择安装包】
+ MIUI 版本 界面，连续点击中间“11”图标
+ 点击右上角【...】，手动选择安装包、重启到 Recovery 出现

<!-- more -->

# 刷第三方 ROM

第三方 ROM，例如，[xda](https://forum.xda-developers.com/mi-6/development/rom-evolution-x-4-20-2-sagit-t4089445) 或 [eu](https://xiaomi.eu/community/threads/miui-11-0-stable-release.52628/)

## 1. 开启`USB调试`
开启`开发者模式`：我的设备 → 全部参数 → MIUI 版本 连续点击 7 次

开启`USB调试`：更多设置 → 开发人员选项 → USB 调试模式。

手机 USB 插到电脑上。

## 2. 解锁 BL（Bootloader）
http://www.miui.com/unlock/download.html

## 3. 安装 TWRP

> What is TWRP? TWRP is a Custom Recovery.
>
> 存在多种操作方式，其中，手机上安装 app 的方式需要 root 手机，所以，改用命令行`adb+fastboot`的方式

+ 进入 Fastboot

`adb reboot bootloader`
如果未按预期进入 Fastboot，则，先手动关机，再长按 `音量下`+`电源` 开机

+ 下载合适的 `twrp.img`

https://twrp.me/xiaomi/xiaomimi6.html

+ 刷 twrp.img
```cmd
fastboot flash recovery twrp.img
```

+ 进入 Recovery TWRP
```cmd
fastboot boot twrp.img
// OR
fastboot reboot
```
如果未按预期进入 TWRP，则，在 Fastboot 模式下，长按`音量上`+`电源`，等看到小米 log 时，只松开`电源`键，直到出现 TWRP，再松开`音量上`。

清数据，FORMAT /data partition (NEVER wipe System or Persist!)

+ 刷 ROOM 镜像

把 ROOM 镜像放到手机存储

+ 完成


# 其他

## 新系统引导界面，卡在 Google 账号验证
Tip: 刷机前，应在老系统中退出 Google 账号登陆，否则就会遇到这个问题。  
如果你的网络可以穿墙，在手机尚无法安装设置其他 app 的情况下，其实这也不算个事儿。

解决办法：关闭新系统新机引导界面
1. 进入 Recovery TWRP
2. 在菜单中完成/system 分区。以防万一，adb 中再次挂载`adb remount /system`
3. 进入 shell，关闭引导
```cmd
adb shell
echo "ro.setupwizard.mode=DISABLED" >> /system/build.prop
```
4. 系统重启

---
title: linux 扩展boot分区
date: 2019-05-04 12:06:18
tags:
 - linux
categories: linux
---

扩展linux的/boot分区

## 使用gpared或者fdisk创建一个新的partition

## find the uuid of the new partition
使用命令
~$:ls -l /dev/disk/by-uuid/
获得分区的uuid 
    lrwxrwxrwx 1 root root 10 11月 24 14:34 19d6c114-8859-4209-aef9-60ee3cc108c1 -> ../../sda9
    lrwxrwxrwx 1 root root 10 11月 24 14:34 1C48-1828 -> ../../sda2
    lrwxrwxrwx 1 root root 10 11月 24 14:34 2840620D4061E254 -> ../../sda4
    lrwxrwxrwx 1 root root 11 11月 24 14:34 66ab484d-0bbc-41cb-b2ca-8f436a330e2b -> ../../sda10
    lrwxrwxrwx 1 root root 10 11月 24 14:34 71640978-4b7b-49aa-9a3e-ef22c994a183 -> ../../sda6
    lrwxrwxrwx 1 root root 10 11月 24 14:34 8856E16256E1518C -> ../../sdb1
    lrwxrwxrwx 1 root root 10 11月 24 14:34 99f1b75b-eb7b-41bb-9aa8-3c5ab2446f01 -> ../../sda7
    lrwxrwxrwx 1 root root 10 11月 24 14:34 B4CEF361CEF31A76 -> ../../sdb2
    lrwxrwxrwx 1 root root 10 11月 24 14:34 B836469636465592 -> ../../sda1
    lrwxrwxrwx 1 root root 10 11月 24 14:34 C14D581BDA18EBFA -> ../../sda5
    lrwxrwxrwx 1 root root 10 11月 24 14:34 e9b32a21-5e8a-4c53-9982-a31cd67c464e -> ../../sda8

## 更新配置文件/etc/fstab
通过改变uuid将/boot目录挂在到新的挂载点上
    from 
       UUID=99f1b75b-eb7b-41bb-9aa8-3c5ab2446f01 /boot           ext4    defaults        0       2
    to 
       UUID=66ab484d-0bbc-41cb-b2ca-8f436a330e2b /boot           ext4    defaults        0       2
    here we can use the device name /dev/sda10 but it may change if we add some other devices, uuid is unique so that it won't change.


## 重启

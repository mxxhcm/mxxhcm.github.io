---
title: linux-启动流程
date: 2019-05-07 16:41:27
tags:
categories:
---

## Linux的启动流程
BIOS   MBR    boot loader    boot sector
## BIOS
BIOS(Basic Input Ouput System)是一套程序,它被写死到主板上面的一个内存芯片，这个内存芯片没有电时也能将数据记录下来，那就是一个ROM(Read Only Memory)。BIOS是系统开机时首先会去读取的一个小程序，它控制着开机时候的各项硬件参数的取得，它掌握了系统硬件的详细信息以及开机设备的选择，BIOS程序代码也会被适度修改，但是如果写在ROM中是无法修改的，现在多把BIOS写入Flash Memory 或者EEPROM中。

BIOS通过硬件的INT13中断功能来读取MBR的，所以只要BIOS能检测到磁盘那么他就能够通过INT13这条信道来读取该磁盘的第一个扇区内的MBR，这样就能够执行boot loader

## Boot Loader
boot loader的最主要功能就是认识操作系统的文件格式并且加载该操作系统的内核到内存中执行。不同操作系统的文件类型不同，所以boot loader也是不同的，那么如果通过一个MBR来安装多操作系统呢。

## Boot Sector
对于文件系统来说，每个文件系统都会有保留一个引导扇区(boot sector)提供给操作系统来安装boot loader。
每个操作系统默认会安装一个boot loader到它的文件系统中。对于Linux来说，我们可以选择将boot loader安装到MBR，也可以不选择，那样boot loader只会安装在它自己的文件系统中的即是(boot sector)。但是Windows操作系统会默认直接将boot loader安装在MBR以及boot sector中，所以说安装双系统时，最好先装Windows，再装Linux，否则反过来的话，那么Windows的boot loader可能就会覆盖掉Linux的boot loader.
但是，系统的MBR只有一个，所以，如何执行boot sector中的boot loader呢，那就需要谈到boot loader的功能了。
boot loader的功能
		提供菜单
		加载内核文件
		转交其他loader
我们可以通过MBR中的boot loader选择其他的loader，这样就可以选择其他的操作系统运行了。
通过boot loader的管理读取了内核文件之后，那么就要进行工作了，重新检测硬件等。
但是从某些版本之后，内核是可以动态加载内核模块的，这些模块被放在/lib/modules/目录内，模块放置到磁盘根目录内，因此，启动过程中内核必须要挂载根目录，这样才能动态读取内核模块提供加载驱动程序的功能。
一般来说，非必要的功能可以编译成模块的内核功能，许多Linux会将内核编译成模块。USB，SATA,SCSI等设备的驱动程序都是通过模块的方式存在的。
那么问题来了，内核是不认识SATA硬盘的，所以根目录无法挂载，更无法通过根目录下的/lib/modules来驱动SATA硬盘了。这时候，就用到了虚拟文件系统(/boot/initrd)来管理。
虚拟文件系统能够通过boot loader加载到内存中，解压缩被当成一个根目录，从而通过该程序加载启动过程中所最需要的内核模块,通常是USB,RAID,LVM,SCSI等文件系统以及硬盘的驱动程序。
需要initrd的原因是因为启动时无法挂载根目录，如果根目录能被挂载，那么就不需要了，根目录再USB,SATA,SCSI等磁盘，或者文件系统比较特殊，为LVM，RAID等，那么是需要
载入这些模块之后，initd就会帮助内核重新调用/sbin/init进行后续正常的启动
	



---
title: linux file system
date: 2019-05-07 17:02:37
tags:
 - linux
categories: linux
---

## 碎片整理
文件写入的block太碎了，文件的读写性能太差，所以可以通过碎片整理将一个文件的block回合在一起FAT文件系统经常需要碎片整理,但是Ext2文件类型是索引式文件系统，所以不太需要经常碎片整理的。

## dumpe2fs [-bh]	
查询每个区段以及superblock的信息
### 参数介绍
dumpe2fs [-bh]
    -b
    -h 仅列出superblock的数据

## df 
查询挂载的设备
### 参数介绍
df [-haiT]　[dir/file]	显示文件系统的整体磁盘用量
    -a 列出所有的文件
    -h 显示文件系统的大写，自行显示格式
    -i 可用的inode
    -T 联通分区文件系统的名称

## du 
目录或者文件所占的容量
### 参数介绍
du [-ashkm] [dir/filename] 默认显示的是目录的容量，不包含文件
    -s 该目录下所有文件的容量，不细列出来
    -a 显示所有的目录与文件的容量
    -h 以人们熟悉的大小方式显示出来
    -k 以kb列出容量
    -m 以mb列出容量:

## ln [-s] 链接文件
hard-link	硬链接，将某个目录下的block多写入一个数据,磁盘的inode与b			lock数量一般不会改变，磁盘容量也不会改变，而且删除一个文			件并不会响另一个文件的读写，但是其对于目录是没有作用的，			对于不同的文件系统也是没有用的。  
sybomlic	

## 磁盘分区，格式化，检验与挂载
df + 目录  查看某个目录挂载的磁盘位置
eg: df /

sudo fdisk [-l]　+ 设备　输出后面设备所有的分区内容　如不加设备名称，列					　出整个系统。

## 新增或者删除分区
sudo fdisk + 设备   对设备进行操作  
partprobe


sudo mkfs [-t ext2/ext2/vfat] + 设备名　 将某个设备格式化为某种文件系统

sudo mke2fs [-b block_size] [-i inode_size]  [-L 卷标] [-cj -c 检查磁盘				错误　-j 加入日志文件] + 设备名

sudo fsck [-CAay] [-t filesystem] + 设备
    -C  使用直方图显示进度
    -A  依据/etc/fstab内容，扫描设备
    -a  自动修复检查到的有问题的扇区
    -y  与-a 类似
    ext2 ext3 支持额外参数　　[-fD] -f 强制进入设备进行检查
				-D 对文件系统下的目录进行优化配								置
sudo badblocks [-sv] + 设备  -s 在屏幕上列出进度 -v 在屏幕上看到进度

## mount
挂载文件系统与磁盘 P227
### 参数介绍
mount [-aoltnL] 
    -a　按照/etc/fstab的配置信息将所有未挂载的磁盘挂载上来
    -l　可增加label名称
    -t　加上文件类型
    -n　默认情况下会将挂载情况写入/etc/mtab，加入-n可以不写入
    -L　可以利用卷标名来挂载
    -o 加一些挂载时的额外参数　
        ro(只读)　rw(可写)
        async sync 此文件系统是否使用同步写入或者异步的内存机制
        auto noauto 允许此分区以mount -a自动挂载(auto)
        dev nodev 是否运行在此分区创建设备文件
        suid nosuid 
        exec noexec 是否可拥有可执行binary文件
        user,nouer 设置user参数可以让一般user能对此分区挂载
        defaults　默认为rw,suid,dev,exec,auto,nouser,async
        remount 重新挂载，在系统出错，或者重新更新参数时

### 示例
mount 设备文件名　挂载点
        
用卷标名挂载设备
~$:mount -L mxx_logical /medic/mxx	

用磁盘设备名挂载
~$:mount /dev/sdb1 /mnt/usb
~$:mount -t iso9660 /dev/cdrom /media/cdrom
~$:mount -o remount,rw,auto /dir
~$:mount -o loop ~/my.iso/myfile.iso /mnt/iso

## 磁盘参数修改
设备用文件来代表通过文件的major与minor数值来替代 
Major 主设备代码，Minor　次设备代码
/dev/hd\*  major = 3				
/dev/sd\*  minor = 8	

### mknod
mknod [bcp]
    b   设置设备名称成为一个外部存储文件，如硬盘 
    c   设置设备名称成为一个外部输入文件，如鼠标/键盘
    p   设置设备名称成为一个FIFO文件

### e2label
e2label /dev/sdb5 + 新的label名称

### tune2fs
tune2fs [-jlL]
    -l  类似 dupme2fs -h 
    -j  将ext2转换为ext3
    -L  类似于　e2label 

### hdparm
hdparm -Tt /dev/sd\*  测试SATA硬盘的读取性能

## 挂载.iso文件
mount -o loop /home/mxx/ubuntu16.04 /mnt/ubuntu16.04

dd命令　创建一个大文件 
dd if=/dev/zero of=/home/mxx/filename bs=1M count=512
    if--input file	/dev/zero 一直输出0
    of--output file	将if中的内容加入到of接的文件名中
    bs--block size	
    count	共有多少个bs

## 构建swap空间
例如将第二快硬盘的第五个分区改为swap分区
~$:sudo fdisk -l /dev/sdb
    p
    t 5
    82
    w
    partprobe
将/dev/sdb5更改为swap类型的文件系统
~$:mkswap /dev/sdb5
~$:free 查看memory以及swap分区的使用情况
~$:swapon /dev/sdb5 使用/deb/sdb5的swap分区	
~$:swapon -s 查看目前使用的swap设备有哪些
~$:swapoff /dev/sdb5

## boost sector与superblock 的关系
1. superblock的大小为1024b
boost sector与superblock 各占一个block ,可以查看/boot的挂载目录
0号block给boost ，1号block给superblock
2. superblock的大小大于1024b,如为4096b
superblock在0号blok ,但是superblock 只有1024b,所以为了防止空间浪费，于是		在0号block内，superblock(1024-2047),boost sector(0-1023),2048后			面的空间保留。
实际情况中，由于在比较大的block中，我们能将引导装载程序安装到superblock所在的0号block,但事实上还是安装到启动扇区的保留区域。
比较正确的说法是，安装到文件系统最前面的1024b内的区域，就是启动扇区

## 查看文件的inode编号
~$:ls -i 
目录并不一定只占一个block，当目录内的文件数太多时，会增加该目录的block

## 参考文献
1.《鸟哥的LINUX私房菜》


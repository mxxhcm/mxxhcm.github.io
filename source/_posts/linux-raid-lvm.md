---
title: linux raid lvm
date: 2019-05-07 16:52:12
tags:
 - linux
categories: linux
---

## 磁盘阵列

### mdadm新建raid
#### 参数介绍
mdadm --detail  后面接的那个磁盘阵列设备的具体信息
madam --create 为新建raid的参数
    --auto=yes /dev/md[01...]
    --raid-device=N 使用几个raid作为磁盘阵列的设备
    --spare-device=N　使用几个磁盘作为备用
    --level=[0125]  这组磁盘阵列的等级是0,1还是5之类的

#### 示例
~#:mdadm --create --auto=yes /dev/md0 --level=5 --raid-device=3 --spare-device=1 /dev/sdb{5,6,7,8}
创建raid需要时间，创建好之后
~#:mdadm --detail /dev/md0 
查看建好的RAID
~#:cat /proc/mdstat
~#:mkfs -t ext4 /dev/md0
~#:mkdir /mnt/raid
~#:mount /dev/md0 /mnt/raid
~#:df

### mdadm管理raid
#### 参数介绍
mdadm --manage /dev/md[0-9] [--add 设备] [--remove 设备] [--fail 设备]
    ~#:mdadm --manage /dev/md0 --fail /dev/sdb6
    ~#:mdadm --detail /dev/md0
    ~#:cat /proc/mdstat
    过一段时间在执行会发现以及将坏的设备更新了
    ~#:mdadm --detail /dev/md0
    ~#:mdadm --manage /dev/md0 --add /dev/sdb9 --remove /dev/sdb10
    

### 开机自动加载raid
~#:mdadm --detail /dev/md0 | grep -i 'uuid'
~#:vim /etc/mdadm/mdadm.conf
    ARRAY /dev/md0 UUID=.....
~#:vim /etc/fstab
    /dev/md0 /mnt/raid ext4 defaults 1 2
~#:umount /dev/md0 
~#:mount -a
~#:df


### 关闭raid
~#:vim /etc/fstab
    # /dev/md0 ..
~#:mdadm --stop /dev/md0
~#:cat /proc/mdstat
~#:vim /etc/mdadm/mdadm.conf

## LVM的制作
LVM Logical Volume Manager
PV physical volume
VG volume group
PE physical extend
LV logical volume

### LV的写入机制
- 线性机制
若有两个设备/dev/sda1,/dev/sdb1,他们都在一个VG中，并且只有一个LV，线性机制就是在一个设备完全写满之后，再向另一个设备写入
- 交错模式

### 新建分区
~#:sudo fdisk /dev/sdb
    new /dev/sdb{5,6,7,8,9,10}
    t 8e(Linux LVM)
    w
~#:partprobe
### 安装应用
sudo apt-get install lvm2
### PV物理卷的新建
pvcreate 将物理分区新建为PV分区
pvscan 查询目前系统里具有PV的磁盘
pvdisplay 显示目前系统上面的PV状态
pvremove 将PV属性删除

pvmove 将某个设备内的pe给移动到另一个设备
    pvmove /dev/sdb5 /dev/sdb9

~#:pvscan
~#:pvcreate /dev/sdb{5,6,7,8}
~#:pvscan 
~#:pvdisplay

### VG卷用户组的新建
vgcreate 新建VG
vgscan 查找目前系统上的VG
vgdisplay 显示目前系统上的VG状态
vgextend 在VG内新增额外的VG
vgreduce 在VG内删除PV
vgchange 设置VG是否启动
vgremove 删除一个VG
VG名称是自己定义的。而PV名称实际上是分区的设备文件名

vgcreate [-s] VG名称 PV名称
    -s　后面接PE的大写,单位可以是m,g,t (支持大小写)
~#:vgcreate -s 16M mxxvg /dev/sdb{5,6,7}
~#:vgscan
~#:pvscan
~#:vgdisplay
vgextend VG名称　PV名称
~#:vgextend mxxvg /dev/sdb8
~#:vgdisplay
  
### LV逻辑卷的新建
lvcreate 新建lv
lvscan 查询系统上的lv
lvdisplay 展示系统上的lv
lvextend 在lv里增加容量
lvreduce 在lv里减少容量
lvremove 删除一个lv
lvresise 对lv的大小进行重新调整

lvcreate [-lLs] [-n lv名称]　vg名称
    -l 后接的是PE的个数
    -L 后接的是vg的容量
    -n 后接lv的名称
    -s snapshot 快照
~#:lvcreate -l 252 -n mxxlv mxxvg
~#:ls -l /dev/mxxvg/mxxlv
~#:lvscan
~#:lvdisplay

### 文件系统新建
~#:mkfs -t ext4 /dev/mxxvg/mxxlv
~#:mkdir /mnt/lvm
~#:mount /dev/mxxvg/mxxlv /mnt/lvm
~#:df -h .
  
### 增加lv容量
~#:sudo fdisk /dev/sdb
    new /dev/sdb9
~#:pvcreate /dev/sdb9
~#:pvscan
~#:vgextend mxxvg /dev/sdb9
~#:vgdisplay
增加lv的容量
~#:lvresize -l +63 /dev/mxxvg/mxxlv 
~#:lvdisplay
~#:df -h /mnt/lvm

此时虽然lv显示的容量增大，但是对应的/dev/mxxvg/mxxlv文件系统还没有改变
~#:dumpe2fs /dev/mxxvg/mxxlv
重新计算文件系统
resizefs [-f] [device] [size]
    -f 强制进行resize
    device 后接的文件系统或者是设备名
    size 如果没有size默认为整个文件系统，如果有size的话，必须给一个             单位
~#:resize2fs /dev/mxxvg/mxxlv   //可在线进行resize

### 缩小lv容量
　　　先计算需要缩小多少
~#:pvscan
~#:pvdisplay
　　　缩小文件系统容量
放大可以直接进行，但是缩小需要先卸载
~#:umount /dev/mxxvg/mxxlv
~#:resize2fs /dev/mxxvg/mxxlv 3900M
报错需要用e2fsck
~#:e2fsckk -f /dev/mxxvg/mxxlv 
~#:resize2fs /dev/mxxvg/mxxlv 3900M
~#:mount /dev/mxxvg/mxxlv /mnt/lvm
~#:df -h /mnt/lvm
　　　降低lv容量
~#:lvresize -l -63 /dev/mxxvg/mxxlv
~#:lvdisplay
      转移pv
~#:pvdisplay
~#:pvmove /dev/sdb5 /dev/sdb9
　　　删除vg
~#:vgreduce mxxvg /dev/sdb5
      删除pv    
~#:pvscan
~#:pvremove /dev/sdb5
     

## LVM的快照
需要有未使用的PE块
所以需要新加入一个PV块

~#:vgdisplay
~#:pvcreate /dev/sdb5
~#:vgextend mxxvg /dev/sdb5
~#:vgdisplay
~#:lvcreate -l 40 -s -n mxxlv_ss /dev/mxxvg/mxxlv
    -s snapshot
~#:lvdisplay
复原的数据是不能比快照区的大小大的，此处不能大于40个PE

接下来改变LVM中的数据，会发现lvm与快照区是不同的
~#:cd /mnt/lvm
~#:cp -a /home/mxx/my.iso /mnt/lvm
~#:lvdisplay 会发现lv的快照区已经被使用了
~#:df 会发现原始文件与快照区文件系统也是不同的

### 利用快照区进行备份
~#:tar -cvj -f /home/mxx/my.bak/lvm.bak.tar.bz2 *
~#:umount /mnt/snapshot

将快照区进行删除，因为已经被备份
~#:lvremove /dev/mxxvg/mxxlv_ss 

~#:umount /mnt/lvm
~#:mkfs -t ext4 /mnt/lvm
~#:mount /dev/mxxvg/mxxlv /mnt/lvm

将备份的数据还原，那么这个文件系统就会和原来一样了
~#:tar -xvj -f /home/mxx/my.bak/lvm.bak.tar.bz2 /mnt/lvm
~#:ls -l /mnt/lvm

### LVM的关闭
先卸载lvm系统，包括快照与原系统
再使用lvremove删除LV
使用vgchange -a n VG 名称 让其不再为active
使用vgremove删除VG
使用pvremove删除PV
最后使用sudo fdisk 修改System ID

~#:umount /mnt/lvm
~#:lvremove /dev/mxxvg/mxxlv
~#:vgchage -a n mxxvg
~#:vgremove mxxvg
~#:pvremover /dev/sdb{5,6,7,8}
~#:sudo fdisk -l 

## 参考文献
1.《鸟哥的LINUX私房菜》 


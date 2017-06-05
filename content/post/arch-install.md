+++
date = "2017-06-05T20:39:10+08:00"
draft = true
title = "arch 安装"

+++

## Boot Arch Linux 启动 安装

```bash
efivar -l #查看是不是 UEFI启动 还是 BIOS
```

## 连接网络 如果是wifi的话

```bash
cp /etc/netctl/examples/wireless-wpa-config /etc/netctl/homewifi  #homewifi 随便起的名字
ip link show # 显示 wifi 设备 一般为 wlo1
```
我的是 wlo1,打开wifi设备

```bash
ip link set wlo1 up # 关闭 ip link set wlo1 down
```

修改 /etc/netctl/homewifi

```bash
netctl start homwwifi
ping -c 3 www.baidu.com #测试网络
```

## 分区

```bash
lsblk #显示硬盘设备
cfdisk #分区  
```
### 固态硬盘

> 分3个区
> swap分区 我是没分，主要我内存6G也够了，而且swap分区读写频繁会影响ssd寿命
> swap 没有听说会影响休眠功能，而且内存不够时会直接kill掉进程
> 不过swap 可以向window那样 挂在文件作为swap

/boot  # 建议500M，我个人分了200M 主要我ssd只有120G 安装linux内核太多时可能会满
/mnt   # 系统而软件主分区 15-35G 都可以，我分了30G，我安装软件还是挺多的
/home  # 剩下的 分给它

```bash
mkfs.vfat -F32 /dev/sda1
mkfs.btrfs /dev/sda2
mkfs.btrfs /dev/sda3
```
ssd /boot 格式话未 vfat -F32 兼容性好，其他的格式 btrfs

### 机械硬盘

分4个区
/boot
/swap # 内存4G以下建议分内存的2倍,4G以上的分2G或者部分也可以
/mnt
/home

```bash
mkfs.vfat -F32 /dev/sda1
mkswap /dev/sda2
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda4
```

### 挂在分区

挂在根分区 其他分区都挂在在根后面

```bash
mount  /dev/sda3 /mnt
mkdir /mnt/home
mount /dev/sda3 /mnt/home
mkdir -p /mnt/boot/EFI # BOIS 启动 mkdir /mnt/boot
mount /dev/sda1 /mnt/boot/EFI # BOIS 启动 mount /dev/sda1 /mnt/boot
```

有 swap 分区的也启动 swap

```bash
swapon /dev/sda2
```
待续。。。

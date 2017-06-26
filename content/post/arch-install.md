+++
date = "2017-06-26T11:34:45+08:00"
draft = false
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

## 开始安装Arch

### 修改源成中国源

> aliyun 163 的源挺快的,我的放前面

```bash
mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
cat /etc/pacman.d/mirrorlist.bak | grep -e .cn/ > /etc/pacman.d/mirrorlist
```
刷新系统包,开始真的安装
```
pacman -Syy
pacstrap -i /mnt base base-devel
```

生成 fstab 文件

> fstab文件是 系统的分区挂载情况描述文件,告诉文件系统挂载情况

之前的操作 都是基于U盘启动环境的，需要在真是的电脑arch生成fstab
真是的arch被挂载为 /mnt

```
genfstab -U -p /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab #看看fstab文件正常不
```

到此我们之前所有的操作都为U盘启动环境，现在需要进入真是arch
```
arch-chroot /mnt /bin/bash
```

现在/mnt 只有系统 没有启动文件，所有不能重启
```
passwd #修改root密码
echo 你主机名 > /etc/hostname
```
修改 */etc/hostname*,添加 *127.0.0.1 localhost.localdomain localhost 你主机名 *

安装 grub 启动引导
UEFI需要*efibootmgr*,有多个系统需要*os-prober*
根据需要安装
```
pacman  -S grub efibootmgr
```
将引导信息写入扇区,以后启动不了系统，也可以进入安装环境，挂载硬盘系统为*/mnt*,重新安装grub
```
grub-install --recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```
执行`pacman -S vim`安装vim。
执行`ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime` 设置时区。
执行`hwclock --systohc --utc`设置时间标准 为 UTC。
`timedatectl set-ntp true` 开启网络对时，与window共享时，不要开启网络对时，否则两时间不一致而是设置为localtime `hwclock --systohc --localitme`
`vim /etc/locale.gen`移除*en_US.UTF-8 UTF-8、zh_CN.UTF-8 UTF-8*
`locale-gen` 生成本地化信息
`echo LANG=en_US.UTF-8 > /etc/locale.conf`将系统 locale 设置为en_US.UTF-8.
>这里可以设中文，但是这里设的是全局,如果设成非英文环境，桌面显示正常，但是terminal则乱码

还需要在实际电脑配置网络

有线网络
```
systemctl enable dhcpcd.service
```
无线网络
安装iw wpa_supplicant dialog

```
cp /etc/netctl/examples/wireless-wpa /etc/netctl/your_profile
#修改 你的 *your_profile*
vim /etc/netctl/your_profile
netctl start your_profile
exit
umount -R /mnt
root
```
启动

如果启动后wifi没连上
```
iwconfig #查看wifi 硬件接口 或ip link查看
#假设为wlo1
ip link set wlo1 up
netctl status your_profile #查看状态
```
创建用户加入wheel
```
useradd -m -G wheel -s /bin/bash johnson
passwd johnson
#修改whell ALL=(ALL)ALL 去掉 ＊#*
vim /etc/sudoers
```

## 安装桌面
修改源,按照上面方法

```
pacman -S xorg #安装基础
pacman -S lxdm #启动管理器，连接桌面程序
pacman -S xfce4
pacman -S xfce4-goodies # 包含很多界面插件，一下常用软件
systemctl enable lxdm #开机启动
```
lxdm 默认 lxde 登录
修改为xfce4
`vim /etc/lxdm/lxdm.conf` *session=/usr/bin/startlxde* 修改为 *session=/usr/bin/startxfce4*
实现不需要密码登录用户 lxdm.conf 修改 autologin 为指定用户 johnson

重启进入 *xfce4*  进入 *jonson* 用户
配置 *jonson*登录语言环境
```
pacman -S wqy-microhei #中文字体
# 输入法
pacman -S fcitx-im fcitx-configtool
pacman -S ibus ibus-rime
touch ~/.xprofile
vim ~/.xprofile
#加入
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
export LC_CTYPE=en_US.UTF-8
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```
以后谁需要就 cp .xprofile /home/用户名

 声音
```
pacman -S pulseaudio pavucontrol #有关 pacman -S alsa-utils  启动alsamixer
```
安装yaourt
```
sudo vim /etc/pacman.conf
#加入
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server   = https://mirrors.163.ccom/archlinuxcn/$arch
sudo pacman -Syy #更新数据库
sudo pacman -S archlinuxcn-keyring #安装中文源密钥
sudo pacman -S yaourt
```

常用软件
`zsh git tmux unrar zip unzip openssh glances htop iftop screenfetch tree vlc wget net-tools 网易云音乐 chromum-dev proxychains4 dnsmasq ss atom npm plantsuml`
安装 oh-my-zsh
安装 guake 开机启动  sudo ln -s /usr/share/applications/guake.desktop /etc/xdg/autostart/.

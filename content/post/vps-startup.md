+++
categories = ["vps"]
date = "2017-08-01T14:48:47+08:00"
tags = ["vps","linux"]
title = "vps 基本设置"

+++

## 虚拟主机、vps、实体服务器的对比小结
![](/vps-start/diff.png)


## vps的重要参数

我们选购时，要重点考察以下几个技术参数：虚拟化技术，操作系统，内存大小，硬盘容量，每月流量，独立IP个数，服务器所在地，Ping值等。

## vps的虚拟化技术

vps根本上就是虚拟机，都是在一定的虚拟化技术上构建的。目前用得最多的虚拟化技术是Xen, OpenVZ, Hyper-V, vmware. 下面针对vps有用的部分粗略的讲一下，了解更多可以参看这里 xen、kvm、vmware、hyper-v等虚拟化技术的比较

其中Hyper-V是微软自家的虚拟化技术，只能在windows上运行，也就是一般买windows系统的vps时，很可能是Hyper-V的。vmware国内的一些较小主机商会用，跑windows或linux的都有，用过虚拟机的朋友应该知道它。

另外两种都是主要跑linux的虚拟化技术。

其中OpenVZ是基于操作系统的虚拟化技术，它运行效率跟真机(实体服务器)几乎一样。不过也别高兴过早了，vps的性能都是来自于宿主机的，因为宿主机上有很多vps，每个vps可以获得的资源事实上并不很高，具体这要看宿主机本身硬件性能如何、上面运行了多少vps。

OpenVZ有几个显著特点：没有交换分区swap（虚拟内存），不能运行pptp协议的vpn，容易被超售。

>关于超售：假设宿主机有16G内存，但开出20台1G内存的vps，都卖出去了；而这20台vps里都显示1G内存，这就是超售。事实上OpenVZ通常超售得更厉害！

Xen，是一种称为半虚似化的技术，性能比真机有所损失，但虚拟出来的系统跟真机相似度极高，有swap，可以运行pptp的vpn，不容易超售。在xen的linux上，可以更换或升级内核；据说甚至可以再装个虚拟机环境虚拟出vps（没有亲眼见过，不过即使成功，性能也是极其低下，没有实用性的）。

一般来说，大家都认同以下说法：

>购买同等配置的vps，xen的性能要明显优于OpenVZ. 最主要的原因就是超售问题。
OpenVZ没有swap，通过free命令查出的内存，其中一部分事实上是宿主机的swap的，只是被vps当成物理内存；
没有不超售的OpenVZ vps.
512M的Xen，其内存性能比1G OpenVZ vps的好，甚至是远超。
OpenVZ内存用完时，系统就差不多只能重启了，因为这时远程ssh连接也无法建立的。而xen的，还有swap可用，通常不至于要重启。
看上去，xen几乎是完胜于openvz，那价格呢，也一样，xen远远高于openvz. 毕竟一分价钱一分货。

关于xen与openvz的了解更多，请参阅这里 <http://www.path8.net/tn/archives/5383>


使用32位还是64位的操作系统？ 十分负责任的告诉你，毫不犹豫的选择32位！除非以下两种情况：你的vps内存在超过4G，或者你要运行某的软件只能在64位下运行。 选用32位原因：运行同样的程序，32位占用内存小；

## startup
* apt-get update && apt-get upgrade
* apt-get install vim  创建.vimrc `wget -c https://raw.githubusercontent.com/johnsonwjx/vim-for-server/master/vimrc -O .vimrc`
* 更新内核

>下载内核地址<http://kernel.ubuntu.com/~kernel-ppa/mainline>
```
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10.17/linux-image-4.10.17-041017-generic_4.10.17-041017.201705201051_amd64.deb
#安装内核
dpkg -i linux-image-4.*.deb
#删除旧内核
dpkg -l|grep linux-image
apt-get purge 旧内核
#更新 grub 系统引导文件并重启
update-grub
reboot
```

* 开启bbr

>开机后 uname -r 看看是不是内核 >= 4.9
执行 lsmod | grep bbr，如果结果中没有 tcp_bbr 的话就先执行
```
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
```
开启bbar
```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
#保存生效
sysctl -p
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
#如果结果都有bbr, 则证明你的内核已开启bbr
#看到有 tcp_bbr 模块即说明bbr已启动
```
* 修改root密码，创建新用户，给 sudo

> apt-get install sudo 查看 */etc/sudoers* 是否*%sudo*用户组启用了
```
passwd #修改root
useradd $username
usermod -G sudo $username
```
* 修改ssh端口，改用密钥登录,禁用root 和 密码登录

>
```
su $username
mkdir .ssh
touch .ssh/authorized_keys
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
```
复制公钥
mac cat ~/.ssh/id_rsa.pub | pbcopy
linux cat ~/.ssh/id_rsa.pub | xclip -selection cllipboard
粘贴到 vps的 .ssh/authorized_keys
先测试密钥登录再禁用密码登录
本地的.ssh/config
```
### default for all ##
Host *
     ForwardAgent no
     ForwardX11 no
     ForwardX11Trusted yes
     Protocol 2
     ServerAliveInterval 60
     ServerAliveCountMax 30
    IdentityFile ~/.ssh/id_rsa

## override as per host ##
Host la
     HostName $ip
     User $username
     Port $port
```

* apt-get install fail2ban

> 配置 /etc/fail2ban/jail.conf 主要 maxretry bantime

* ufw 防火墙管理

>
```
apt-get install ufw
#查看防火墙状态
sudo ufw status
ufw status
#启用
ufw enable
sudo ufw default deny
#开启/禁用
sudo ufw allow|deny [service]
打开或关闭某个端口，例如：
sudo ufw allow smtp　允许所有的外部IP访问本机的25/tcp (smtp)端口
sudo ufw allow 22/tcp 允许所有的外部IP访问本机的22/tcp (ssh)端口
sudo ufw allow 53 允许外部访问53端口(tcp/udp)
sudo ufw allow from 192.168.1.100 允许此IP访问所有的本机端口
sudo ufw allow proto udp 192.168.0.1 port 53 to 192.168.0.2 port 53
sudo ufw deny smtp 禁止外部访问smtp服务
sudo ufw delete allow smtp 删除上面建立的某条规则
```

##参考/转载与

<https://www.path8.net/tn/archives/5370>
<http://blog.leanote.com/post/shaodeyang@hotmail.com/VPS%E5%9F%BA%E7%A1%80%E5%AE%89%E5%85%A8%E8%AE%BE%E7%BD%AE>
